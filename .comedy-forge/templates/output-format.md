# Comedy Forge Output Format

输出文件名：`output/comedy-forge-[theme]-[YYYYMMDD-HHMMSS].md`

```markdown
# Comedy Forge 演出稿 - {theme}
- 生成时间：{YYYY-MM-DD HH:MM:SS}
- 模式：{standup|manzai|auto}
- 轮次：{actual_rounds}
- Batch / Audience：{batch_size} / {audience_size}
- 二次精修回路：{enabled}
- Set List组合评分：{set_score}

## 1. 今夜人格画像
- 风格配方比例：
  - 自我贬损: {x}%
  - 观察错位: {x}%
  - 荒诞推演: {x}%
  - 攻击外物: {x}%
  - 文字游戏: {x}%
  - 情境模拟: {x}%
- 认知滤镜分布：{top_filters}
- 代际分布：{generation_mix}

### 1.1 多样性体检面板
- coverage_ratio(身份处境/认知滤镜/当下状态/语言风格)：{...}
- normalized_entropy(身份处境/认知滤镜/当下状态/语言风格)：{...}
- top1_share(关键维度)：{...}
- pair_repeat_rate(secret,current_state)：{...}
- elite_filter_drift：{...}
- distribution_jsd(关键维度)：{...}
- 健康结论：{PASS|WARN|FAIL}
- 自动处置动作：{actions}

## 2. 进化树日志
| Joke ID | 存活轮次 | 最终得分 | Polarizer | Wildcard |
|---|---:|---:|---|---|
| R1-05 | R1-R4 | 4.42 | 否 | 否 |

### 2.1 评语驱动改写日志（MVP）
| Joke ID | Actions | Before | After | Net Delta |
|---|---|---:|---:|---:|
| R2-15 | LOGIC_GAP+TARGET_BLUR | 3.2 | 3.9 | +0.7 |

## 3. 演出清单（Top 6-8）
### 3.0 Set List组合评分面板
- set_score：{set_score}
- flow_score：{flow_score}
- callback_chain_score：{callback_chain_score}
- diversity_score：{diversity_score}
- adjacency_overlap_penalty：{adjacency_overlap_penalty}

### #1 {joke_id} - {title}
- 人格面具：{persona_mask}
- Setup：{setup}
- Punchline：{punchline}
- 节奏：{[停顿]/[加速]/[收]}
- Callback关联建议：{callback}
- 精修轨迹：{draft -> vA/vB -> final}
- 最终采用版本：{draft|vA|vB}
- 精修收益：{clarity_delta / callback_delta / punch_density_delta}
- 四亚组评分：A {a} / B {b} / C {c} / D {d}

## 4. X区荒诞实验
### X-1 {title}
- 高风险点：{risk}
- 潜在收益：{upside}
- 建议试演场：{scene}
- 实验版台词：{line}
```

## 必填校验
- 必须出现“今夜人格画像 / 进化树日志 / 演出清单 / X区荒诞实验”四大节。
- 必须出现“多样性体检面板”并给出健康结论。
- 演出清单数量必须在6-8之间。
- X区数量必须在2-3之间。
