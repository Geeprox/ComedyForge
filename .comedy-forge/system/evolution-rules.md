# Evolution Rules (Genetic Loop)

## 1. 每轮主循环
1. 从上一轮结果中按`survival-rate`计算精英池：
   - `elite_count = ceil(batch_size * survival-rate)`
2. 生成新血段子，数量 = `batch_size - elite_count`（基础池）。
3. 新血必须绑定全新人格，并满足与旧精英认知滤镜差异度 > 0.5。
4. 合并后形成基础候选池，规模固定为`batch_size`（10-500）。
5. 送入观众评分（见`scoring-protocol.md`）。
6. 基于评语运行改写诊断（见`comment-rewrite-rules.md`），生成`rewrite_signals_map`。
7. 选出新精英池（数量=`elite_count`）进入下一轮。
8. 选出外卡origin（最多5个，来自Polarizer池且不在精英池）。
9. 下一轮为每个外卡origin生成A/B双版本；命中条件时追加C版本，作为扩容槽位加入候选池。

## 2. 外卡与AB版本策略
- 对于进入外卡的段子：下一轮强制创建`variant-A`与`variant-B`。
- 当满足以下任一条件时，额外创建`variant-C`（条件式）：
  - 该段子命中`REPEAT_WITHIN_SET`或`ENDING_PREDICTABLE`或`PERSONA_COLLAPSE`
  - 或该段子`std_dev > 1.2`且处于高争议高潜力区
- A/B/C版本共享同一`origin_id`，并独立评分。
- 若A/B均失败（低于本轮中位数），该路线进入X区归档。
- 外卡为扩容机制，不挤占精英名额。
- 扩容后池规模：`batch_size + 2 * wildcard_origin_count + variant_c_count`。
- 若扩容后超过硬上限500，先削减`variant-C`，再削减新血数量，不削减精英与外卡A/B版本。
- 若该origin存在改写信号：
  - A版优先执行P1动作（修致命问题）
  - B版优先执行P2/P3动作（保留冒险性）
  - C版优先执行“换叙述人称 + 换节奏编排”，保持核心冲突不变

## 3. 提前终止条件（必须同时满足）
- `current_round >= min-rounds`
- `elite_mean >= 4.2`
- `elite_std_dev < 0.6`
- `polarizer_rate < 10%`

## 4. 强制继续条件
只要任一终止条件不满足，且`current_round < max-rounds`，继续迭代。

## 5. 偏见注入（压力测试）
仅当以下条件成立时可启用：
- `max-rounds > 5`
- `current_round >= 6`

启用方式：
- 将30%观众倾斜到某个极端维度（如`mood=暴躁`）。
- 记录偏见注入标签，防止与正常轮次混淆。
- 对比偏见轮与正常轮的精英留存差异，评估段子鲁棒性。

## 6. 防近亲繁殖
- 若新血段子与任一精英段子的“认知滤镜 + 角度 + 核心冲突”三元组相似度 > 0.7，则强制突变：
  - 优先改`认知滤镜`
  - 其次改`攻击对象`
  - 最后改`叙述视角`

## 7. 进化日志要求
每轮记录：
- 精英ID列表
- 外卡ID列表
- 淘汰ID列表
- 轮均分、精英均分、争议率
- 是否触发偏见注入
- 是否满足提前终止
