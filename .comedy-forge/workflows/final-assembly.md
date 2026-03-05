# Workflow: Final Assembly

## 目标
将最终精英段子组装成可演出的Set List，并输出标准Markdown文件。

## 1. 选段规则
- 从最终精英池选Top 6-8。
- 至少包含2种以上喜剧角度。
- 至少包含1个高争议高回报段子（若存在Polarizer胜出者）。

预选池构建：
- 先按`individual_score`取Top 10-14进入装配候选。
- `individual_score`由以下加权计算：
  - `overall_score_norm`：45%
  - `callback_potential_norm`：20%
  - `clarity_norm`（理解门槛越低越高）：20%
  - `freshness_norm`（与同场语义重叠越低越高）：15%

## 2. 编排顺序
1. 热场：1个低理解门槛、高共鸣段子。
2. 中场：3-4个主题推进段子，做callback预埋。
3. 高潮：1-2个强冲突/高爆发段子。
4. 收场：1个价值回收型段子，完成主题闭环。

## 2.1 Set List组合评分（强制）
从预选池中搜索6-8条组合，目标最大化：

```text
set_score =
  0.55 * avg(individual_score)
+ 0.20 * flow_score
+ 0.15 * callback_chain_score
+ 0.10 * diversity_score
- 0.15 * adjacency_overlap_penalty
```

维度定义：
- `flow_score`：是否形成“热场 -> 推进 -> 高潮 -> 收场”的能量曲线。
- `callback_chain_score`：前文梗在后文被自然回收的次数与质量。
- `diversity_score`：角度/认知滤镜/语言风格的场内覆盖度。
- `adjacency_overlap_penalty`：相邻段子在同一攻击对象或同构结构上的重复惩罚。

搜索策略：
- 候选数<=14时，允许全组合搜索。
- 候选数>14时，先剪枝到14再做束搜索（beam size建议8）。

## 3. 精修动作
- 加入节奏标注：`[停顿] [加速] [收]`
- 给每个段子附`callback关联建议`
- 对台词长度做口语化压缩，避免书面腔
- 精修模式池：`compress/callback_boost/clarity_boost/persona_lock/risk_soften`

## 3.1 二次精修回路（强制）
对已选Set List中的每条段子执行1轮定向精修：
1. 生成`vA-压缩版`：压缩冗余铺垫，目标台词长度下降10%-20%。
2. 生成`vB-callback强化版`：保留核心笑点，增强与前后段落的梗回收接口。
3. 若触发以下任一条件，追加`vC-稳定性版`（从`clarity_boost/persona_lock/risk_soften`三选一）：
   - `clarity`低于阈值
   - 出现人格口吻漂移
   - 冒犯风险标记升高
4. 在不破坏人格一致性的前提下，比较`vA/vB(/vC)`的快速复评得分，选择更优版本入演出稿。

精修验收阈值：
- `clarity`不得下降超过0.1。
- `callback_chain_score`相对初稿至少提升0.05，或`punch_density`提升0.08。
- 若两版均未达阈值，保留初稿并记录`polish_failed_reason`。

上下文隔离：
- 精修器仅可读取该段子的文本、节奏标签与Set List邻接信息。
- 禁止读取创作者隐藏人格秘密字段与评审个体评论。

## 4. X区编排
- 输出2-3个高风险高回报段子
- 标注“失败风险”与“试演建议场景”

## 5. 文件输出
输出到：`output/comedy-forge-{theme}-{YYYYMMDD-HHMMSS}.md`
格式遵循：`templates/output-format.md`

额外输出要求：
- 必须附`set_score`与四个子维度分解。
- 每条入选段子必须附`polish_track`（初稿/vA/vB(/vC)/最终采用版本）。
