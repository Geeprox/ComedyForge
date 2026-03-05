# Comedy Forge：协议化多轮喜剧生成系统实现论文（对齐最新 Skill）

**版本**：v1.1（与当前 `skill.mdc` 同步）  
**状态**：实现级规范文档（Implementation-Level Spec）  
**对应仓库**：`Geeprox/ComedyForge`

---

## 摘要

本文给出 Comedy Forge 的实现级机制说明。系统面向 stand-up 与 manzai 生成，采用“协议约束 + 反馈闭环 + 组合优化”的多轮流程：先在人格空间中生成批量候选，再通过观众模拟与盲评形成评分分布，经遗传进化和评语驱动改写持续迭代，最终通过 Set List 组合评分与二次精修输出演出级文本。相较单次采样方案，本文系统新增并形式化了以下关键机制：

1. **多样性主动子集采样**（active subset then sample）与 core/long-tail 动态调权；
2. **`angle-preset` 驱动的 10 类角度配额计划**与 `angle_quota_plan/report` 双向校验；
3. **L0-L5 预期违背层级**及冲突优先级；
4. **seed 驱动情绪组分布**（含 7 类评分偏移）与模板池亚组；
5. **外卡 A/B + 条件 C 变体**与 500 上限池管理；
6. **vA/vB + 条件 vC 精修回路**与整场级组合目标函数；
7. **评语驱动改写信号**的结构化诊断与收益回写。

本文提供可直接落地的参数约束、数据契约、阈值、伪代码、复杂度与验证策略，作为 Codex/Claude Code 共用实现参考。

**关键词**：喜剧生成、协议化智能体、观众模拟、遗传进化、评语驱动改写、组合优化

---

## 1. 引言

### 1.1 问题

喜剧文本生成并非“语义通顺”问题，而是“结构可演”问题。系统需要同时满足：

- 单段内部结构（setup -> punchline -> 节奏 -> 回收）；
- 段间组合结构（热场 -> 推进 -> 高潮 -> 收场）；
- 反馈可解释性（知道为什么低分、怎么改）；
- 风格边界可控（人格一致且不过度同质）。

### 1.2 目标

在给定主题与参数下，输出可演出、可追踪、可迭代优化的整场喜剧稿，且每轮有可审计日志（评分、改写、配额、分布体检）。

---

## 2. 系统结构

### 2.1 分层

- 入口与契约：`.comedy-forge/skill.mdc`
- 核心协议：`system/*.md`
- 工作流：`workflows/*.md`
- 模板：`templates/*.md`
- 运行兼容：`compatibility.targets = [codex, claude-code]`

### 2.2 执行链路

```mermaid
flowchart TD
  A[参数校验] --> B[人格子集激活与段子初稿]
  B --> C[多样性/配额/去重体检]
  C --> D[观众模板采样与盲评]
  D --> E[评分聚合与Polarizer判定]
  E --> F[评语信号化重写]
  F --> G[精英保留+外卡扩容+A/B/C变体]
  G --> H[蒸馏归档与收益回写]
  H --> I[Set List组合搜索]
  I --> J[vA/vB(/vC)精修]
  J --> K[标准输出]
```

### 2.3 隔离原则

- 创作者侧不可见评审原始评分与原始评语。
- 评审侧不可见创作者隐藏人格规则。
- 创作者仅接收结构化改写信号（tag/action/priority/confidence）。

### 2.4 双端执行模板（强制）

创作阶段必须实例化独立创作者上下文：

```xml
<persona_creator>
你是一位具备专业基线的喜剧创作者。
今晚你戴上面具：[8维人格变量]
任务：围绕主题 {{theme}} 写段子。
约束：必须在 {{lens}} 认知滤镜内表达，不能跳出角色。
</persona_creator>
```

评审阶段必须切换到独立盲评上下文：

```xml
<persona_judge>
你现在是一位盲评观众，拥有随机 demographic 特征。
请对段子给出 1-5 星与一句评语。
[禁止参考创作者人格设定或历史版本信息]
</persona_judge>
```

约束：每位观众评分前重置一次`persona_judge`；禁止将`persona_creator`字段透传给评审提示词。

---

## 3. 参数空间与全局目标

### 3.1 输入参数

\[
\Theta = \{type, theme, min\_rounds, max\_rounds, survival\_rate, batch\_size, audience\_size, absurd\_booster, angle\_preset\}
\]

约束：

- \(type \in \{standup, manzai, auto\}\)
- \(3 \le min\_rounds \le 20\)
- \(5 \le max\_rounds \le 30\)
- \(0.1 \le survival\_rate \le 0.5\)
- \(10 \le batch\_size \le 500\)
- \(10 \le audience\_size \le 200\)
- \(3 \le absurd\_booster \le 50\)
- \(angle\_preset \in \{balanced, conservative, exploratory\}\)
- \(min\_rounds \le max\_rounds\)

### 3.2 输出路径

\[
output/comedy\text{-}forge\text{-}\{theme\}\text{-}\{YYYYMMDD\text{-}HHMMSS\}.md
\]

### 3.3 优化目标

\[
\max \mathcal{L}=\alpha Q_{single}+\beta Q_{set}+\gamma Q_{diversity}+\delta Q_{rewrite}-\lambda R_{overlap}
\]

其中：

- \(Q_{single}\)：单段评分质量
- \(Q_{set}\)：整场组合质量
- \(Q_{diversity}\)：分布与覆盖健康度
- \(Q_{rewrite}\)：改写净收益
- \(R_{overlap}\)：同构重复风险

---

## 4. 运行时契约（来自 `skill.mdc`）

### 4.1 scoring_contract

- `manzai_duo_chemistry_first = true`
- `seed_mood_shift_enabled = true`
- 负向情绪组占比：默认 40%，负向 seed 60%，正向 seed 25%-30%
- 评分偏移枚举：`疲惫/暴躁/乐观/焦虑/无聊/释然/厌世`
- Polarizer 双阈值：`sigma_gt` + 大样本结构化方差条件

### 4.2 evolution_contract

- `survival_rate_applied = true`
- `wildcard_mode = expand_slots`
- 外卡每轮最多 5 个 origin
- 基础变体数 2（A/B），条件下可到 3（C）
- `candidate_pool_hard_cap = 500`

### 4.3 assembly_contract

- 二次精修启用
- 基础精修变体 2（vA/vB），条件下可到 3（vC）
- 精修模式池：`compress/callback_boost/clarity_boost/persona_lock/risk_soften`
- Set List 目标函数权重：

\[
w = \{0.55, 0.20, 0.15, 0.10, -0.15\}
\]

### 4.4 diversity_contract

- `sampling_mode = active_subset_then_sample`
- `weighting_mode = core_long_tail_dynamic`
- `angle_preset_default = balanced`
- `angle_preset_values = [balanced, conservative, exploratory]`
- 10类角度权重采用预设档位（core/long-tail）
- `core_weight_total_range = [0.65, 0.85]`
- `long_tail_weight_total_range = [0.15, 0.35]`
- 覆盖率、熵、Top1占比、pair重复率、JSD、漂移等体检指标

---

## 5. 数据契约

### 5.1 Joke

```yaml
id: "R2-17"
round_created: 2
persona:
  baseline_professional: true
  identity: "中层"
  generation: "Millennials"
  tone: "社恐"
  lens: "法律合规"
  language_style: "学术"
  secret: "副业亏损"
  current_state: "刚被HR约谈"
  theme_relation: "旁观者"
content:
  setup: "..."
  punchline: "..."
  tags: [观察错位, L3]
style:
  angle: "观察错位"
  expectation_level: "L3"
  attack_level: 7
  intellectual: 4
scores:
  overall: 4.35
  subgroup_scores: {A: 4.8, B: 4.2, C: 3.1, D: 4.5}
  std_dev: 0.8
metadata:
  is_polarizer: false
  is_wildcard: false
  callback_potential: [R1-05]
  rewrite_applied_actions: ["补前提桥接句"]
```

### 5.2 Audience

```yaml
id: "A-17"
dimensions:
  generation: "Millennials"
  profession: "Tech"
  experience: "Veteran"
  mood: "焦虑"
vote: 3
comment: "结构有效但结尾可预测"
```

### 5.3 Rewrite Signal

```yaml
joke_id: R2-15
round: 2
rewrite_signals:
  - tag: ENDING_PREDICTABLE
    priority: P2
    action: 后移关键反转词
    confidence: 0.81
keep_elements:
  - 中段冲突句有效
```

---

## 6. 人格与多样性采样

### 6.1 两层人格

- 常量基线：行业敬畏、观众尊重、方法论内化。
- 变量层：身份、代际、性格、认知滤镜、语言风格、隐藏秘密、当下状态、主题关系。

### 6.2 维度边界（防串味）

- `身份处境`必须是长期结构角色。
- `刚离婚/刚失业/刚结婚`等短时事件必须放入`当下状态`。
- `隐藏秘密`与`当下状态`禁止语义重复。

### 6.3 主动子集采样

不是全量等概率抽样，而是两步：

1. 先激活子集 `active_values(dim)`
2. 再在子集内按动态权重逐条采样

激活规模（受维度总库上限约束）：

- batch 10-29：4-6
- batch 30-79：6-10
- batch 80-200：8-14
- batch 201-500：10-16

### 6.4 单值样本数护栏

\[
expected\_count\_per\_value = \frac{batch\_size}{active\_count(dim)}
\]

- `<30`: [2,6]
- `30-79`: [3,10]
- `80-200`: [4,18]
- `201-500`: 关键维度不少于8

### 6.5 覆盖底线

- 身份处境：>=4（batch>=50 时 >=5）
- 认知滤镜：>=5（batch>=50 时 >=6）
- 当下状态：>=4（batch>=50 时 >=5）
- 语言风格：>=3（batch>=50 时 >=4）
- 单一代际 <= 50%
- 相邻语言风格冲突率 = 0

---

## 7. 角度配额、层级配额与分布体检

### 7.1 `angle-preset` 驱动的10类角度权重

系统支持三档预设：

- `balanced`（默认，Core/Long-tail = 0.75/0.25）
- `conservative`（更稳，Core/Long-tail = 0.85/0.15）
- `exploratory`（更冒险，Core/Long-tail = 0.65/0.35）

选择规则：

- 若输入提供`angle-preset`且合法，按输入执行
- 未提供时使用`balanced`
- 非法值回退`balanced`并记录`preset_fallback=true`

`balanced`权重：

- 自我贬损 0.14，观察错位 0.16，荒诞推演 0.13，攻击外物 0.11，文字游戏 0.10，情境模拟 0.11
- 角色反差 0.08，规则漏洞 0.07，元叙事自嘲 0.06，情绪反噬 0.04

`conservative`权重：

- 自我贬损 0.16，观察错位 0.18，荒诞推演 0.15，攻击外物 0.13，文字游戏 0.11，情境模拟 0.12
- 角色反差 0.05，规则漏洞 0.04，元叙事自嘲 0.03，情绪反噬 0.03

`exploratory`权重：

- 自我贬损 0.12，观察错位 0.14，荒诞推演 0.11，攻击外物 0.10，文字游戏 0.09，情境模拟 0.09
- 角色反差 0.12，规则漏洞 0.09，元叙事自嘲 0.08，情绪反噬 0.06

### 7.2 配额换算

\[
raw_i=batch\_size\cdot weight_i,\quad base_i=\lfloor raw_i \rfloor
\]

剩余名额采用最大余数法分配。

### 7.3 预设分档配额下限

`balanced`：

- 10-29：long-tail>=15%，active>=2，angle_top1_share_max=0.35
- 30-79：long-tail>=20%，active>=3，angle_top1_share_max=0.35
- 80-200：long-tail>=22%，active>=4，angle_top1_share_max=0.35
- 201-500：long-tail>=25%，active>=4，angle_top1_share_max=0.35

`conservative`：

- 10-29：long-tail>=10%，active>=1，angle_top1_share_max=0.40
- 30-79：long-tail>=12%，active>=2，angle_top1_share_max=0.40
- 80-200：long-tail>=14%，active>=3，angle_top1_share_max=0.40
- 201-500：long-tail>=15%，active>=4，angle_top1_share_max=0.40

`exploratory`：

- 10-29：long-tail>=20%，active>=2，angle_top1_share_max=0.33
- 30-79：long-tail>=28%，active>=3，angle_top1_share_max=0.33
- 80-200：long-tail>=32%，active>=4，angle_top1_share_max=0.33
- 201-500：long-tail>=35%，active>=4，angle_top1_share_max=0.33

### 7.4 预期违背层级（L0-L5）

- \(L0 \le 10\%\)
- \(L1 \le 15\%\)
- \(L2 \ge 30\%\)
- \(L3 \ge 25\%\)
- \(L4 \ge 10\%\)
- \(L5 \ge 5\%\)

冲突优先级：`L2/L3 > L4/L5 > L1/L0`。

### 7.5 Diversity Health Report

每轮输出并校验：

- `coverage_ratio`（关键维度>=0.80）
- `coverage_ratio`（其他维度>=0.70）
- `normalized_entropy`（关键维度>=0.72）
- `normalized_entropy`（其他维度>=0.65）
- `top1_share`（关键维度<=0.35）
- `top1_share`（其他维度<=0.40）
- `pair_repeat_rate(secret,current_state) <= 0.03`
- `angle_quota_hit_rate >= 0.80`
- `long_tail_share_actual >= long_tail_share_min(angle_preset, batch_size)`
- `angle_preset in {balanced, conservative, exploratory}`
- `elite_filter_drift in [0.30, 0.70]`
- `distribution_jsd(key_dim) in [0.08, 0.35]`

---

## 8. 观众模拟、模板池与评分偏移

### 8.1 观众权重层

- 极高：代际/职业/阅历
- 高：性格/地域/禁忌
- 中：职级/遭遇/文化倾向/心情

### 8.2 相关性采样

- GenZ ↔ Student +0.8
- 暴躁 ↔ 被裁 +0.7
- 同行阅历 ↔ 批判 +0.6

### 8.3 Seed 联动（负向情绪组）

负向组定义：`疲惫/暴躁/焦虑/厌世`

- 默认占比：40%
- 负向 seed：60%
- 正向 seed：25%-30%

显式示例：`seed="雨夜沮丧"`时，`疲惫/暴躁/焦虑/厌世`合计占比从40%提升到60%，本轮整体评分宽松度下降（手更紧）。

评分偏移：

- 疲惫 -0.15
- 暴躁 -0.25
- 乐观 +0.10
- 焦虑 -0.10
- 无聊 -0.12
- 释然 +0.05
- 厌世 -0.18

### 8.4 Audience>100 模板池

不再固定单一 A/B/C/D 矩阵，而是模板池轮换（T1/T2/T3），并限制连续轮重复模板。

---

## 9. 评分协议与 Polarizer

### 9.1 评分维度

Standup：30/25/25/20。  
Manzai：35/25/25/15（双人气场优先）。

实现约束：`type=manzai`时，观众应优先评估“双人气场配合度/接梗反应速度/节奏对打完整性”，而非单人笑点密度。

### 9.2 聚合流程

1. `raw_score = weighted_sum(dim_scores)`
2. `biased_score = clamp(raw_score + mood_bias, 1, 5)`
3. 取整数票
4. 去2高2低
5. 计算均值与标准差

### 9.3 Polarizer 判定

小样本（n<=100）：尾部对冲 + sigma>1.15。  
大样本（n>100）：额外看同群方差、跨组标准差、sigma>1.2。

---

## 10. 进化机制：精英、外卡、A/B/C 变体

### 10.1 精英保留

\[
elite\_count = \lceil batch\_size\cdot survival\_rate \rceil
\]

### 10.2 外卡扩容

每轮最多 5 个外卡 origin。下一轮基础变体 A/B；命中条件时追加 C。

触发 C 条件：

- `REPEAT_WITHIN_SET` 或 `ENDING_PREDICTABLE` 或 `PERSONA_COLLAPSE`
- 或 `std_dev > 1.2` 且为高潜争议段

池大小：

\[
pool_{next}=batch\_size + 2\cdot wildcard\_origins + variant\_c\_count
\]

超 500 时：先削减 C，再削减新血，不削减精英与 A/B。

### 10.3 变体分工

- A：优先 P1（修致命）
- B：优先 P2/P3（保探索）
- C：换人称 + 换节奏，核心冲突不变

---

## 11. 评语驱动改写器

### 11.1 切片权重

- `vote<=2`：1.0
- `vote==3`：0.5
- `vote>=4`：仅提炼 keep_elements

### 11.2 标签体系（含新增）

结构/攻击/语言/节奏外，新增：

- `REPEAT_WITHIN_SET`
- `ENDING_PREDICTABLE`
- `PERSONA_COLLAPSE`

### 11.3 诊断分数

\[
tag\_score = \sum(comment\_weight\cdot confidence)
\]

取 Top1-3，优先级 `P1>P2>P3`，每段最多 3 动作。

### 11.4 安全与作用域

强制作用：外卡/边缘淘汰/低分段（overall<3.4）。

创作者侧只能读取结构化信号，不可读取原始评语。

---

## 12. 蒸馏、归档与收益回写

- 淘汰段：强制压缩格式 `id|concept|score|reason`
- 字段上限：`concept <= 10字`，`reason <= 15字`
- 禁止附加原文台词、隐藏人格、历史版本细节
- 精英段：完整文本 + 版本链 + 改写动作 + delta
- 每轮生成 `rewrite_feedback_log`
- 连续两轮 `net_delta<0` 的动作自动降权

---

## 13. Final Assembly：组合搜索与 vC 精修

### 13.1 单段预评分

\[
individual = 0.45\cdot overall + 0.20\cdot callback + 0.20\cdot clarity + 0.15\cdot freshness
\]

### 13.2 组合评分

\[
set\_score = 0.55\cdot avg(individual) + 0.20\cdot flow + 0.15\cdot callback\_chain + 0.10\cdot diversity - 0.15\cdot overlap
\]

### 13.3 搜索策略

- 候选<=14：全组合
- 候选>14：剪枝后 beam（建议 8）

### 13.4 二次精修

基础：vA 压缩版 + vB callback 强化版。  
条件触发 vC（clarity/persona/risk 三类）。

精修模式池：`compress/callback_boost/clarity_boost/persona_lock/risk_soften`。

---

## 14. 终止条件与异常处理

提前终止需同时满足：

- `round >= min_rounds`
- `elite_mean >= 4.2`
- `elite_std < 0.6`
- `polarizer_rate < 10%`

异常：

1. 去极值后样本 <6 -> 重采样观众
2. 亚组偏差 >±10% -> 重平衡重算
3. 动作冲突 -> 按优先级与人格一致性裁决
4. 池超限 -> 按 C -> 新血 顺序削减

---

## 15. 复杂度与工程开销

设轮次 \(R\)、批量 \(B\)、观众 \(A\)：

- 主评分成本：\(O(R\cdot B\cdot A)\)
- 去重最坏：\(O(B^2)\)（仅 B>100）
- 组合搜索：近似 \(O(k\cdot beam)\)

开销控制：

- 去重按阈值触发
- 候选上限 14
- 日志增量化

---

## 16. 验证协议

### 16.1 模式回归

- 极简：10/10/3
- 深度：100/200/5

### 16.2 约束回归

- 覆盖率、熵、Top1、pair重复、quota命中率
- L0-L5 分层是否达标
- Manzai 双人维度是否生效

### 16.3 改写收益

\[
mean(net\_delta) > 0
\]

并校验负收益动作是否降权。

### 16.4 组装收益

- `set_score`趋势
- `callback_chain`提升
- `overlap_penalty`下降

---

## 17. 局限与后续

- 模板池与权重仍是先验工程参数，需线上校准。
- 评语标签映射存在语义歧义边界。
- 当前以协议为主，统一 CLI runtime 仍可进一步产品化。

---

## 18. 结论

Comedy Forge 的本质是把“灵感型写作”变为“协议型优化”：用多样性约束保证输入空间质量，用盲评与改写信号形成反馈闭环，用组合目标函数实现整场最优。本文已与最新 skill 机制逐项对齐，可直接指导实现、审稿与复现。

---

## 附录 A：参数表

| 参数 | 范围 | 说明 |
|---|---|---|
| type | standup/manzai/auto | 体裁模式 |
| min-rounds | 3-20 | 最少轮次 |
| max-rounds | 5-30 | 最多轮次 |
| survival-rate | 0.1-0.5 | 精英生存率 |
| batch-size | 10-500 | 基础候选规模 |
| audience-size | 10-200 | 评审规模 |
| absurd-booster | 3-50 | 荒诞增幅 |
| angle-preset | balanced/conservative/exploratory | 角度配额预设 |

## 附录 B：协议映射

| 机制 | 文件 |
|---|---|
| 入口契约 | `.comedy-forge/skill.mdc` |
| 人格与观众生成 | `.comedy-forge/system/persona-generator.md` |
| 评分与争议判定 | `.comedy-forge/system/scoring-protocol.md` |
| 评语改写 | `.comedy-forge/system/comment-rewrite-rules.md` |
| 进化与外卡变体 | `.comedy-forge/system/evolution-rules.md` |
| 多样性与配额计划 | `.comedy-forge/system/batch-diversity.md` |
| 批量创作 | `.comedy-forge/workflows/batch-creator.md` |
| 评审流程 | `.comedy-forge/workflows/audience-simulator.md` |
| 蒸馏归档 | `.comedy-forge/workflows/distillation-engine.md` |
| 组装与精修 | `.comedy-forge/workflows/final-assembly.md` |
| 输出模板 | `.comedy-forge/templates/output-format.md` |
