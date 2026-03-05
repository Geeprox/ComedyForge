# Comedy Forge Output Format

输出文件名：`output/comedy-forge-[theme]-[YYYYMMDD-HHMMSS].md`

```markdown
# Comedy Forge 演出稿 - {theme}
- 生成时间：{YYYY-MM-DD HH:MM:SS}
- 模式：{standup|manzai|auto}
- 角度预设：{balanced|conservative|exploratory}
- 轮次：{actual_rounds}
- Batch / Audience：{batch_size} / {audience_size}
- 二次精修回路：{enabled}
- Set List组合评分：{set_score}

## 1. 今夜人格画像
- 风格配方比例（实际）：
  - 自我贬损: {x}%
  - 观察错位: {x}%
  - 荒诞推演: {x}%
  - 攻击外物: {x}%
  - 文字游戏: {x}%
  - 情境模拟: {x}%
  - 角色反差: {x}%
  - 规则漏洞: {x}%
  - 元叙事自嘲: {x}%
  - 情绪反噬: {x}%
- 认知滤镜分布：{top_filters}
- 代际分布：{generation_mix}

### 1.1 多样性体检面板
- audience_subgroup_template：{T1|T2|T3}
- coverage_ratio(身份处境/认知滤镜/当下状态/语言风格)：{...}
- normalized_entropy(身份处境/认知滤镜/当下状态/语言风格)：{...}
- top1_share(关键维度)：{...}
- pair_repeat_rate(secret,current_state)：{...}
- elite_filter_drift：{...}
- distribution_jsd(关键维度)：{...}
- 健康结论：{PASS|WARN|FAIL}
- 自动处置动作：{actions}

### 1.2 角度配额执行面板（目标 vs 实际）
- angle_preset_used：{balanced|conservative|exploratory}
| 角度 | Target% | Actual% | Delta% | Target Count | Actual Count | Status |
|---|---:|---:|---:|---:|---:|---|
| 自我贬损 | {t}% | {a}% | {d}% | {tc} | {ac} | {OK/WARN} |
| 观察错位 | {t}% | {a}% | {d}% | {tc} | {ac} | {OK/WARN} |
| 荒诞推演 | {t}% | {a}% | {d}% | {tc} | {ac} | {OK/WARN} |
| 攻击外物 | {t}% | {a}% | {d}% | {tc} | {ac} | {OK/WARN} |
| 文字游戏 | {t}% | {a}% | {d}% | {tc} | {ac} | {OK/WARN} |
| 情境模拟 | {t}% | {a}% | {d}% | {tc} | {ac} | {OK/WARN} |
| 角色反差 | {t}% | {a}% | {d}% | {tc} | {ac} | {OK/WARN} |
| 规则漏洞 | {t}% | {a}% | {d}% | {tc} | {ac} | {OK/WARN} |
| 元叙事自嘲 | {t}% | {a}% | {d}% | {tc} | {ac} | {OK/WARN} |
| 情绪反噬 | {t}% | {a}% | {d}% | {tc} | {ac} | {OK/WARN} |
- angle_quota_hit_rate：{x}%
- long_tail_share_target/actual：{t}% / {a}%
- quota_summary：{PASS|WARN|FAIL}

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
- 精修轨迹：{draft -> vA/vB(/vC) -> final}
- 最终采用版本：{draft|vA|vB|vC}
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
- 必须出现“角度配额执行面板（目标 vs 实际）”并给出quota_summary。
- 演出清单数量必须在6-8之间。
- X区数量必须在2-3之间。
