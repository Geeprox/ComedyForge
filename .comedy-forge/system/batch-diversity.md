# Batch Diversity Guardrails

## 1. 人格维度配额（强制）
每个Batch（10-500个）必须满足：
- 身份处境至少4种（`batch-size >= 50`时至少5种）
- 认知滤镜至少5种（`batch-size >= 50`时至少6种）
- 当下状态至少4种（`batch-size >= 50`时至少5种）
- 语言风格至少3种（`batch-size >= 50`时至少4种）
- 单一代际占比20%-50%
- 相邻段子语言风格必须不同
- 已激活值的最小样本数：
  - `batch-size < 30`：每个激活值至少2条样本
  - `30 <= batch-size < 80`：每个激活值至少3条样本
  - `80 <= batch-size <= 200`：每个激活值至少4条样本
  - `200 < batch-size <= 500`：关键维度（身份处境/认知滤镜/当下状态）每个激活值至少8条样本，非关键维度至少5条样本

## 1.1 激活子集校准（强制）
每轮先生成维度激活子集，再生成段子：
- 10-29：每维激活4-6个值
- 30-79：每维激活6-10个值
- 80-200：每维激活8-14个值
- 201-500：每维激活10-16个值

说明：以上区间受“该维总库规模上限”约束，不得超过该维可用枚举总数。

校准公式：
```text
expected_count_per_value = batch_size / active_count(dim)
batch_size < 30          -> expected_count_per_value in [2, 6]
30 <= batch_size < 80    -> expected_count_per_value in [3, 10]
80 <= batch_size <= 200  -> expected_count_per_value in [4, 18]
200 < batch_size <= 500  -> expected_count_per_value >= 8（不设硬上限，受维度总库约束）
```

若不满足范围：
1. 先调`active_count(dim)`；
2. 再调采样权重；
3. 最后才触发整池重采样。

## 1.2 维度边界检查（防串味）
- “身份处境”只允许长期结构角色；不得含“刚X”短时事件。
- `刚离婚/刚结婚/刚失业`等短时事件必须进入“当下状态”。
- “隐藏秘密”与“当下状态”不得同义重复（如同时写“刚失业”）。

## 2. 喜剧角度配额（每50个段子）
- 自我贬损：10
- 观察错位：12
- 荒诞推演：10
- 攻击外物：8
- 文字游戏：5
- 情境模拟：5

当`batch-size`非50整数倍时，按比例缩放并四舍五入，最后用“最大余数法”补齐总数。

## 3. 预期违背层级配额
- L1 <= 20%
- L2 >= 40%
- L3 >= 30%
- L4 >= 10%

若比例冲突，优先保证L2和L3，再补L4，最后压缩L1。

## 4. 去重策略（仅当batch-size > 100）

### 4.1 检测口径
- 对每个段子提取概念集合：`{主题词, 冲突词, 反转词, 目标对象}`
- 对认知滤镜先做标签归一化（如`法学/律师视角 -> 法律合规`）再比较
- 计算两两Jaccard相似度
- 若`Jaccard > 0.7`视为高重叠

### 4.2 处置顺序
1. 先尝试“转角度”重写（保持主题，改变喜剧角度）
2. 再尝试“换滤镜”重写（保持冲突，改变认知镜头）
3. 再尝试“换当下状态”重写（保持观点，改变时间压力源）
4. 仍高重叠则送入X区待实验

## 5. 批内自检流程
1. 先生成`active_values(dim)`快照
2. 生成初稿池
3. 跑配额检测（覆盖率 + 单值样本数 + 维度边界）
4. 对超标类别做定向重采样
5. 跑去重（若需要）
6. 复检配额并锁定Batch
7. 输出`diversity_health_report`

## 6. 失败回退
若连续3次仍无法满足配额：
- 放宽单一角度上限+5%
- 放宽“非关键维度”激活值数量下限-1
- 但不得破坏以下底线：
  - 认知滤镜覆盖 >= 5
  - 身份处境覆盖 >= 4
  - 当下状态覆盖 >= 4
  - 相邻语言风格不同
  - 单一代际 <= 50%
  - 激活值最小样本数 >= 2

## 7. 分布体检清单（Diversity Health Report）
每轮必须输出以下指标，用于判断“扩枚举是否有效”：

### 7.1 关键指标
- `coverage_ratio(dim) = used_count(dim) / active_count(dim)`  
  目标：关键维度（身份处境/认知滤镜/当下状态）`>= 0.80`，其他维度`>= 0.70`
- `normalized_entropy(dim) = entropy(used_dist) / log(active_count(dim))`  
  目标：关键维度`>= 0.72`，其他维度`>= 0.65`
- `top1_share(dim)`  
  目标：关键维度`<= 0.35`，其他维度`<= 0.40`
- `pair_repeat_rate(secret,current_state)`  
  目标：`<= 0.03`（同轮重复组合占比）
- `adjacent_style_violation_rate`  
  目标：`= 0`

### 7.2 跨轮漂移指标
- `elite_filter_drift = 1 - Jaccard(curr_elite_filters, prev_elite_filters)`  
  目标：`0.30 ~ 0.70`
- `distribution_jsd(dim)`（与上一轮该维分布的Jensen-Shannon Divergence）  
  目标：关键维度`0.08 ~ 0.35`

### 7.3 自动处置策略
1. 若`coverage_ratio`不足：减少该维`active_count`并重采样。
2. 若`normalized_entropy`过低：提高long-tail权重`+0.05`。
3. 若`top1_share`过高：对Top1值施加`repetition_penalty +0.10`。
4. 若`elite_filter_drift < 0.30`：对认知滤镜提高探索权重，且外卡优先“换滤镜”重写。
5. 若`elite_filter_drift > 0.70`：降低探索权重，回补core值稳定性。
