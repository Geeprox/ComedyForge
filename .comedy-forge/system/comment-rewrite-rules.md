# Comment-Driven Rewrite Rules

## 1. 目标
将低分评语自动压缩为“可执行改写动作”，用于下一轮重写与变体生成，避免人工逐条解读评语。

## 2. 输入
- `joke_id`
- `round`
- `trimmed_votes`（去极值后票）
- `trimmed_comments`（去极值后评语）
- `type`（standup/manzai/auto）
- `metadata`（angle, lens, attack_level, callback_potential, is_wildcard）

## 3. 评语切片与权重
1. 主诊断切片：`vote <= 2` 的评语（高权重，1.0）。
2. 次诊断切片：`vote == 3` 的评语（中权重，0.5）。
3. 对`vote >= 4`的正向评语只提取“应保留要素”，不参与问题判定。

## 4. 标签字典（问题 -> 动作）

### 4.1 结构与逻辑
- `LOGIC_GAP`（逻辑断裂/因果跳步）
  - 动作：`补前提桥接句`、`延后反转1拍`
- `SETUP_TOO_LONG`（铺垫过长）
  - 动作：`压缩setup 10%-20%`、`提前冲突点`
- `PUNCH_WEAK`（结尾不够爆）
  - 动作：`提升预期违背层级`、`增加对照反差句`

### 4.2 攻击对象与立场
- `TARGET_BLUR`（攻击对象模糊）
  - 动作：`改攻击对象为单一可识别实体`
- `STANCE_DRIFT`（观点摇摆）
  - 动作：`固定叙述立场`、`删除自我否定句`
- `TOO_OFFENSIVE`（冒犯过界）
  - 动作：`降低attack_level 1-2级`、`改为自嘲或制度嘲讽`

### 4.3 可理解性与语言
- `JARGON_OVERLOAD`（术语过载）
  - 动作：`术语降噪`、`替换为生活化比喻`
- `CONTEXT_HEAVY`（背景依赖过高）
  - 动作：`补一行上下文`、`去掉圈内梗专有名词`
- `LENS_DRIFT`（偏离人格滤镜）
  - 动作：`强制回到{{lens}}视角`、`重写首句为人格口吻`

### 4.4 节奏与互动
- `RHYTHM_FLAT`（节奏平）
  - 动作：`加入[停顿]-[加速]-[收]节奏锚点`
- `CALLBACK_WEAK`（回收弱）
  - 动作：`插入前文锚点`、`收尾回收母题`
- `MANZAI_CHEMISTRY_WEAK`（仅manzai）
  - 动作：`增加装傻-吐槽来回次数`、`强化接梗反应速度`

### 4.5 场内重复与收束质量
- `REPEAT_WITHIN_SET`（与同场段子重复）
  - 动作：`切换角度`、`替换冲突对象`
- `ENDING_PREDICTABLE`（结尾可预测）
  - 动作：`增加假设误导句`、`后移关键反转词`
- `PERSONA_COLLAPSE`（人格口吻崩坏）
  - 动作：`重写首句人格锚点`、`统一叙述人称`

## 5. 诊断算法
1. 对评语做关键词与语义匹配，映射到标签。
2. 计算标签得分：`tag_score = sum(comment_weight * confidence)`。
3. 选Top 1-3个标签作为该段子改写信号。
4. 对每个标签分配优先级：
   - P1：结构致命问题（LOGIC_GAP, TARGET_BLUR, MANZAI_CHEMISTRY_WEAK, TOO_OFFENSIVE）
   - P2：显著影响笑点（PUNCH_WEAK, RHYTHM_FLAT, CALLBACK_WEAK, ENDING_PREDICTABLE, REPEAT_WITHIN_SET）
   - P3：风格优化（JARGON_OVERLOAD, CONTEXT_HEAVY, LENS_DRIFT, PERSONA_COLLAPSE）

## 6. 输出结构（强制）
```yaml
joke_id: R2-15
round: 2
rewrite_signals:
  - tag: LOGIC_GAP
    priority: P1
    action: 补前提桥接句
    confidence: 0.86
  - tag: TARGET_BLUR
    priority: P1
    action: 改攻击对象为单一可识别实体
    confidence: 0.78
keep_elements:
  - 结尾“反转方向”受欢迎，保留
```

## 7. 作用域
- 默认对以下对象启用：
  - 当轮`wildcard`段子
  - 当轮“边缘淘汰段”（接近晋级线但未晋级）
  - 低分段（overall < 3.4）
- 不对“稳定高分段”强制重写，只给可选优化信号。

## 8. 隔离与安全
- 创作者侧只可接收`rewrite_signals`与`keep_elements`，不得看到原始观众评语文本。
- 改写动作不得突破人格约束与主题边界。
- 若动作冲突，按优先级执行：P1 > P2 > P3。
