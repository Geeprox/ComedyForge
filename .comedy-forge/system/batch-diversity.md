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

## 2. 喜剧角度权重与配额计划（强制）

### 2.1 角度预设（三档）
支持预设：
- `balanced`：默认，稳定与探索平衡
- `conservative`：更稳，偏向Core角度
- `exploratory`：更冒险，提升Long-tail探索

选择规则：
1. 若输入参数提供`angle-preset`，按输入预设执行。
2. 若未提供，默认使用`balanced`。
3. 若输入非法值，回退`balanced`并记录`preset_fallback=true`。

### 2.2 各预设权重表
`balanced`（Core 0.75 / Long-tail 0.25）：
- 自我贬损：`0.14`（Core）
- 观察错位：`0.16`（Core）
- 荒诞推演：`0.13`（Core）
- 攻击外物：`0.11`（Core）
- 文字游戏：`0.10`（Core）
- 情境模拟：`0.11`（Core）
- 角色反差：`0.08`（Long-tail）
- 规则漏洞：`0.07`（Long-tail）
- 元叙事自嘲：`0.06`（Long-tail）
- 情绪反噬：`0.04`（Long-tail）

`conservative`（Core 0.85 / Long-tail 0.15）：
- 自我贬损：`0.16`（Core）
- 观察错位：`0.18`（Core）
- 荒诞推演：`0.15`（Core）
- 攻击外物：`0.13`（Core）
- 文字游戏：`0.11`（Core）
- 情境模拟：`0.12`（Core）
- 角色反差：`0.05`（Long-tail）
- 规则漏洞：`0.04`（Long-tail）
- 元叙事自嘲：`0.03`（Long-tail）
- 情绪反噬：`0.03`（Long-tail）

`exploratory`（Core 0.65 / Long-tail 0.35）：
- 自我贬损：`0.12`（Core）
- 观察错位：`0.14`（Core）
- 荒诞推演：`0.11`（Core）
- 攻击外物：`0.10`（Core）
- 文字游戏：`0.09`（Core）
- 情境模拟：`0.09`（Core）
- 角色反差：`0.12`（Long-tail）
- 规则漏洞：`0.09`（Long-tail）
- 元叙事自嘲：`0.08`（Long-tail）
- 情绪反噬：`0.06`（Long-tail）

### 2.3 预设对应四档Batch配额模板
`balanced`：
- `batch-size 10-29`：`long_tail_share_min >= 0.15`，`long_tail_active_min >= 2`
- `batch-size 30-79`：`long_tail_share_min >= 0.20`，`long_tail_active_min >= 3`
- `batch-size 80-200`：`long_tail_share_min >= 0.22`，`long_tail_active_min >= 4`
- `batch-size 201-500`：`long_tail_share_min >= 0.25`，`long_tail_active_min >= 4`
- `top1_share_max`：`0.35`

`conservative`：
- `batch-size 10-29`：`long_tail_share_min >= 0.10`，`long_tail_active_min >= 1`
- `batch-size 30-79`：`long_tail_share_min >= 0.12`，`long_tail_active_min >= 2`
- `batch-size 80-200`：`long_tail_share_min >= 0.14`，`long_tail_active_min >= 3`
- `batch-size 201-500`：`long_tail_share_min >= 0.15`，`long_tail_active_min >= 4`
- `top1_share_max`：`0.40`

`exploratory`：
- `batch-size 10-29`：`long_tail_share_min >= 0.20`，`long_tail_active_min >= 2`
- `batch-size 30-79`：`long_tail_share_min >= 0.28`，`long_tail_active_min >= 3`
- `batch-size 80-200`：`long_tail_share_min >= 0.32`，`long_tail_active_min >= 4`
- `batch-size 201-500`：`long_tail_share_min >= 0.35`，`long_tail_active_min >= 4`
- `top1_share_max`：`0.33`

### 2.4 权重转配额算法（强制）
```text
raw_i = batch_size * weight_i
base_i = floor(raw_i)
remain = batch_size - sum(base_i)
按小数余数从高到低给前remain个角度 +1（最大余数法）
```

校正规则：
1. 若`long_tail_share_actual < long_tail_share_min(preset, batch-size)`，从Core高占比角度转移配额到Long-tail低占比角度。
2. 若`long_tail_active_count < long_tail_active_min(preset, batch-size)`，强制激活Long-tail角度并至少分配1条。
3. 若单角度占比过高（`top1_share > top1_share_max(preset)`），将超额配额优先分流到同层未达标角度。

输出要求：
- 每轮先产出`angle_quota_plan`（目标配额），再生成段子。
- 轮末产出`angle_quota_report`（目标/实际/偏差/预设信息）。

## 3. 预期违背层级配额
- L0 <= 10%（纯共鸣铺垫）
- L1 <= 15%
- L2 >= 30%
- L3 >= 25%
- L4 >= 10%
- L5 >= 5%（高风险多段反转）

若比例冲突，优先保证L2和L3，再补L4与L5，最后压缩L1与L0。

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
1. 先生成`active_values(dim)`快照与`angle_quota_plan`
2. 生成初稿池
3. 跑配额检测（覆盖率 + 单值样本数 + 维度边界 + 角度目标/实际偏差）
4. 对超标类别做定向重采样
5. 跑去重（若需要）
6. 复检配额并锁定Batch
7. 输出`diversity_health_report`与`angle_quota_report`

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
- `angle_quota_delta(angle) = actual_count - target_count`  
  目标：`|delta| <= max(1, round(batch_size * 0.06))`
- `angle_quota_hit_rate`（满足delta阈值的角度占比）  
  目标：`>= 0.80`
- `long_tail_share_actual`  
  目标：满足`long_tail_share_min(preset, batch-size)`
- `angle_preset`  
  目标：`balanced|conservative|exploratory`其一

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
6. 若`angle_quota_hit_rate < 0.80`：提高`angle_quota_plan`优先级并锁定未达标角度最小样本数。
7. 若`long_tail_share_actual`低于下限：从Core Top1角度转移5%-10%配额到Long-tail。
8. 若`quota_summary=FAIL`连续2轮：
   - `conservative` -> 自动回退到`balanced`
   - `exploratory` -> 自动回退到`balanced`
   - `balanced` -> 降低long-tail探索权重`-0.05`并重算
