# Workflow: Audience Simulator

## 目标
对候选段子池进行“创作者不可见细节”的盲评，并产出可进化的评分结果。

## 步骤
1. 读取候选Joke池与`audience-size`。
2. 调用`system/persona-generator.md`生成观众12维画像。
3. 执行盲评：
   - 每位观众对每个段子给1-5星整数分
   - 生成一句话评语
   - `type=manzai`时，观众必须优先评估“双人气场配合度/接梗反应速度/节奏对打完整性”，而非只看单人笑点密度
4. 调用`system/scoring-protocol.md`：
   - 去极值
   - 均分/标准差
   - 亚组分数（若audience>100，按当轮A/B/C/D模板）
   - Polarizer判定
5. 应用Seed情绪联动：
   - 若本轮Seed命中负向氛围（如`雨夜沮丧`），将负向情绪组采样占比从默认40%提升到60%
   - 负向情绪组定义为：`疲惫/暴躁/焦虑/厌世`
   - 评分松紧偏移与裁剪逻辑严格执行`system/scoring-protocol.md`第4节
6. 生成本轮结果：Top池、外卡池、淘汰池。
7. 调用`system/comment-rewrite-rules.md`生成改写信号：
   - 输入每条段子的`trimmed_votes + trimmed_comments + metadata`
   - 仅对`wildcard/边缘淘汰/低分段`强制生成Top1-3改写动作
   - 产出`rewrite_signals`与`keep_elements`，禁止透传原始评语
8. 仅向创作者侧暴露摘要：
   - 每条段子的整体分、争议标记、方向建议
   - 每条段子的结构化改写信号（tag/action/priority/confidence）
   - 禁止下发观众个体明细评分

## 上下文隔离执行协议
评审阶段必须切换为：
```xml
<persona_judge>
你现在是一位盲评观众，拥有以下 demographic 特征：[随机特征]
你对以下段子进行1-5星评分...
[绝对禁止参考创作者的人格设定或历史版本信息]
</persona_judge>
```

隔离要求：
- 每位观众评分前都要重置到独立`persona_judge`上下文。
- 禁止将`persona_creator`字段透传到评审提示词。
- 输出到创作者侧前必须做脱敏聚合（只保留统计结果与方向建议）。

## 输出
- `round_scoreboard`
- `wildcard_candidates`（<=5）
- `eliminated_ids`
- `audience_snapshot`（仅统计摘要）
- `rewrite_signals_map`（按`joke_id`索引的结构化改写动作）
