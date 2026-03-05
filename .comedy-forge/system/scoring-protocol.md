# Scoring Protocol

## 1. 评分输入
- Joke池：10-500个
- Audience：10-200人（由`persona-generator.md`生成）
- 评分范围：1-5星，整数
- 每位观众必须提交：`vote` + `one-line comment`
- `type`：`standup | manzai | auto`（影响评分维度权重）

## 2. 核心评分流程
1. 盲评：观众仅看到段子内容，不看到作者人格细节和历史得分。
2. 维度评分：先算“原始维度分”，再映射到1-5星整数。
3. 情绪松紧联动：按观众`mood`施加评分偏移（见第4节）。
4. 去极值：每个段子去掉2个最高星与2个最低星。
5. 聚合均分：对剩余评分计算平均值，保留2位小数。
6. 离散度：计算标准差 `sigma`。
7. 亚组评分（Audience > 100）：按当轮模板计算A/B/C/D四组均分与跨组标准差。
8. 标记争议段（Polarizer）。
9. 生成晋级名单 + 外卡名单。

## 3. 评分维度与权重

### 3.1 Standup 默认维度
- 真实感与观察深度：30%
- 预期违背强度：25%
- 结构与节奏完成度：25%
- callback潜力：20%

### 3.2 Manzai 默认维度（强制强调双人配合）
- 双人气场配合度：35%
- 接梗反应速度：25%
- 节奏对打完整性：25%
- 单点笑点强度：15%

约束：
- `type=manzai`时，禁止只按单人笑点密度打分。
- `type=auto`时，若内容识别为双人对打结构，则自动切换到Manzai权重。

## 4. Seed情绪联动与评分松紧

### 4.1 情绪采样联动（由Audience Seed触发）
- 默认基线：`疲惫/暴躁/焦虑/厌世`合计占比40%。
- 若seed命中负向氛围（如`雨夜沮丧`）：`疲惫/暴躁/焦虑/厌世`合计占比提升到60%。
- 若seed命中正向氛围（如`节日庆典`）：`疲惫/暴躁/焦虑/厌世`合计占比下调到25%-30%。

### 4.2 评分偏移
- `疲惫`：`-0.15`
- `暴躁`：`-0.25`
- `乐观`：`+0.10`
- `焦虑`：`-0.10`
- `无聊`：`-0.12`
- `释然`：`+0.05`
- `厌世`：`-0.18`
- 其他情绪：`0`

执行顺序（必须遵守）：
1. 先按维度权重得到`raw_score`（连续值）。
2. 加上`mood_bias`得到`biased_score`。
3. 裁剪到`[1.0, 5.0]`。
4. 四舍五入为1-5星整数，作为最终投票`vote`。

## 5. Polarizer判定条件

### 5.1 Audience <= 100（抗噪口径）
满足任一条即判定Polarizer：
- `tail_1 >= max(2/n, 0.08)` 且 `tail_5 >= max(2/n, 0.08)`
- `tail_low(<=2星) >= 0.25` 且 `tail_high(>=4星) >= 0.25`
- `sigma > 1.15`

### 5.2 Audience > 100（结构化口径）
在5.1基础上，额外任一条成立也判定Polarizer：
- 同行组内方差 `> 1.5`
- 跨组标准差 `> 0.75`
- `sigma > 1.2`

## 6. 外卡机制
- Polarizer段子即使不在当轮精英池，仍可进入外卡池。
- 每轮外卡最多5个。
- 外卡段子在下一轮必须生成A/B双版本；满足条件时追加C版本：
  - A版：保持核心观点，仅优化结构与节奏。
  - B版：保留核心笑点，切换角度/叙述人称/攻击对象。
  - C版（条件式）：保留核心冲突，重排叙述视角与节奏组织。

## 7. 评分计算公式
```text
raw_score = weighted_sum(dim_scores)
biased_score = clamp(raw_score + mood_bias(audience.mood), 1.0, 5.0)
vote = round_to_int_1_5(biased_score)

trimmed_scores = sort(votes)[2:-2]
overall = round(mean(trimmed_scores), 2)
std_dev = round(stdev(trimmed_scores), 2)

if audience_size > 100:
  subgroup_scores = {A, B, C, D}
  cross_group_std = stdev([A, B, C, D])

tail_1 = count(vote==1)/n
tail_5 = count(vote==5)/n
tail_low = count(vote<=2)/n
tail_high = count(vote>=4)/n
```

## 8. 输出字段映射
写入Joke对象：
- `scores.overall`
- `scores.subgroup_scores`（若启用）
- `scores.std_dev`
- `metadata.is_polarizer`
- `metadata.is_wildcard`

## 9. 审核护栏
- 若去极值后样本数 < 6，标记该轮评分无效并要求重采样观众。
- 若任一亚组人数偏差超过±10%，触发亚组重平衡后重算。
