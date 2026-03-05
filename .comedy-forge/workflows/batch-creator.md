# Workflow: Batch Creator

## 目标
在单轮内生成满足多样性约束的段子池，并输出Joke对象草案。

## 步骤
1. 读取输入：`theme/type/batch-size/round/history/absurd-booster/rewrite_signals_map`。
2. 读取上一轮结构化改写信号（若存在）：
   - 对`wildcard`与“边缘淘汰段”优先分配改写动作
   - 将`P1动作`先执行，`P2/P3`作为候补
3. 调用`system/persona-generator.md`构建本轮8维`active_values`（总库 -> 激活子集）。
4. 在`active_values`内采样人格池，并执行维度边界校验（身份处境=长期角色；短时事件进入当下状态）。
5. 给每个人格绑定创作任务：
   - 生成`setup`
   - 生成`punchline`
   - 标注`angle`与`预期违背层级`
6. 初稿完成后执行`system/batch-diversity.md`：
   - 人格配额检查
   - 激活值样本数检查
   - 维度边界检查
   - 角度配额检查
   - 层级配额检查
   - 去重（batch-size > 100）
7. 对不合格段子进行定向重写（角度/滤镜/当下状态/叙述视角）。
8. 输出合格Joke池，写入轮次缓存（含`active_values_snapshot`与`diversity_health_report`）。

## 创作硬规则
- 保持“说真话 + 预期违背 + 结构收束 + callback潜力”。
- 不允许纯梗堆砌，setup需提供可验证的生活逻辑。
- `type=auto`时，按主题语义自动选择`standup`或`manzai`模板。

## Prompt 模板（创作者侧）
```xml
<persona_creator>
你是一位具备以下专业基线的喜剧创作者：
[插入专业基线要求...]

今晚你戴上面具：[具体8维人格描述...]

任务：创作关于"{{theme}}"的段子。
约束：你必须以这个人格的认知滤镜({{lens}})来看待主题，不能跳出角色。
</persona_creator>
```

执行要求：
- 每条段子都必须单独实例化一个`persona_creator`上下文。
- 同轮中不同段子不得复用同一“隐藏秘密 + 当下状态”组合。
- `身份处境`字段禁止出现“刚X”类短时事件标签。
- 若人格字段冲突，按优先级修正：`身份处境稳定角色` > `当下状态短时事件` > `隐藏秘密长期隐患`。
- 创作阶段禁止读取评审团个体分数与评语明细（仅可读上一轮摘要）。
- 创作阶段可读取`rewrite_signals_map`，但不得读取原始评语文本。

## 评语驱动改写执行规则（MVP）
- `LOGIC_GAP` -> 补前提桥接句 + 延后反转1拍
- `TARGET_BLUR` -> 改攻击对象为单一可识别实体
- `PUNCH_WEAK` -> 提升预期违背层级或增加对照反差句
- `CALLBACK_WEAK` -> 插入前文锚点并在收尾回收
- `MANZAI_CHEMISTRY_WEAK` -> 增加装傻/吐槽来回次数

执行策略：
- 每条待改写段子最多应用3个动作，按`P1 > P2 > P3`排序。
- 若动作冲突，优先保留不破坏人格一致性的动作。
- 每次改写必须记录`rewrite_applied_actions`写入metadata。

## 输出字段
每条段子至少包含：
- `id`, `round_created`
- `persona`（专业基线标记 + 8维变量）
- `content.setup`, `content.punchline`, `content.tags`
- `style.angle`, `style.attack_level`, `style.intellectual`
- `metadata.callback_potential`
- `metadata.rewrite_applied_actions`（若由评语驱动改写触发）
- `metadata.active_values_snapshot`
- `metadata.diversity_health_report`
