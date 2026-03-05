# Workflow: Distillation Engine

## 目标
压缩淘汰信息、保留精英演化细节，并维护完整进化树。

## 数据蒸馏规则

### 1. 淘汰段子（轻存储）
仅保留：
- `id`
- `concept`（一句话概念）
- `final_score`
- `槽点摘要`（1-2条）

不保留完整正文，避免上下文污染。

## 淘汰者压缩格式（强制）
每轮结束后，对淘汰段子必须严格按以下格式输出，禁止超量信息：
`{{id}} | {{concept(≤10字)}} | {{score}} | {{reason(≤15字)}}`

示例：
- `R2-15 | 外卖员穿越设定 | 3.2 | 逻辑断裂难共鸣`
- `R2-18 | 老板像AI算法 | 2.9 | 攻击对象模糊`

执行约束：
- `concept`超长时必须截断或重写到10字内。
- `reason`超长时必须压缩到15字内。
- 禁止附加原文台词、隐藏人格、历史版本细节。

### 2. 精英段子（重存储）
保留：
- 完整`setup/punchline`
- 版本历史（R1->R2->...）
- 每轮得分与亚组分
- callback引用关系
- `rewrite_signals`与`rewrite_applied_actions`
- 改写前后效果差值（`score_delta`, `clarity_delta`, `callback_delta`）

### 3. 进化树
每轮记录节点与边：
- 节点：段子版本ID
- 边：`rewrite_from`, `variant_of`, `wildcard_promoted`
- 标签：`is_polarizer`, `survived_rounds`, `final_status`

## 输出结构建议
```yaml
round: 3
elite:
  - id: R3-07
    parent: R2-11
    scores: [4.1, 4.3, 4.5]
eliminated:
  - id: R3-22
    concept: "婚礼像公司团建"
    score: 3.2
    issues: ["冲突不足", "反转过早"]
```

## 护栏
- 淘汰段子严禁在下一轮直接复活。
- 仅可使用“概念残片”作为灵感，且必须重建人格与叙述角度。

## 评语驱动改写归档（MVP强制）
- 每轮必须生成`rewrite_feedback_log`：
  - `joke_id`
  - `applied_actions`
  - `before_score`
  - `after_score`
  - `net_delta`
- 若`net_delta < 0`连续2轮，自动降低该动作在后续轮次的优先级。
