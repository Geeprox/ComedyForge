# Comedy Forge 角度预设选择手册

本手册用于帮助你在 `balanced / conservative / exploratory` 三种 `angle-preset` 里快速选择，并稳定跑出符合目标的段子分布。

## 1. 预设速览

| 预设 | 目标 | Core/Long-tail 倾向 | 适合场景 |
|---|---|---|---|
| `balanced` | 稳定与探索平衡 | 约 75 / 25 | 默认首选、日常迭代 |
| `conservative` | 稳定出活、降低风险 | 约 85 / 15 | 首演排练、交付导向、时间紧 |
| `exploratory` | 提高新鲜度与实验性 | 约 65 / 35 | 工作坊、素材池扩展、试验场 |

## 2. 快速决策

1. 不确定选哪个：先用 `balanced`。  
2. 本轮目标是“稳”：用 `conservative`。  
3. 本轮目标是“找新角度”：用 `exploratory`。  
4. 如果 `quota_summary=FAIL` 连续两轮：回到 `balanced`。  

## 3. 按 Batch 的建议

| batch-size | 建议 |
|---|---|
| `10-29` | 以 `balanced` 或 `conservative` 为主；`exploratory` 仅用于试验 |
| `30-79` | 优先 `balanced`；阶段性冲刺可切 `exploratory` |
| `80-200` | 三种都可，按目标切换；建议结合体检面板动态调整 |
| `201-500` | 推荐 `balanced` 或 `exploratory`；注意 long-tail 占比与命中率 |

## 4. 推荐运行流程

1. 第 1 轮用 `balanced` 建立基线。  
2. 看输出中的“角度配额执行面板”：  
   - `angle_quota_hit_rate >= 80%` 且 `quota_summary=PASS`：可保持预设。  
   - 命中率低或 `WARN/FAIL`：先回 `balanced`，再调。  
3. 目标是上台稳定性：向 `conservative` 收敛。  
4. 目标是扩素材池：向 `exploratory` 倾斜。  

## 5. 命令示例

```text
forge type=auto theme="职场汇报焦虑" min-rounds=3 max-rounds=5 survival-rate=0.3 batch-size=50 audience-size=30 absurd-booster=5 angle-preset=balanced
```

```text
forge type=standup theme="AI开会黑话" min-rounds=3 max-rounds=6 survival-rate=0.28 batch-size=80 audience-size=40 absurd-booster=6 angle-preset=conservative
```

```text
forge type=auto theme="租房时代情绪账单" min-rounds=4 max-rounds=7 survival-rate=0.25 batch-size=120 audience-size=60 absurd-booster=8 angle-preset=exploratory
```

## 6. 看哪些指标决定要不要换预设

优先看以下字段：

1. `angle_quota_hit_rate`：是否达到 `>= 80%`。  
2. `long_tail_share_target/actual`：实际值是否明显低于目标值。  
3. `quota_summary`：`PASS / WARN / FAIL`。  
4. `top1_share`：是否被单一角度“吃满”。  

## 7. 常见策略

1. 太保守（观众反馈“像但不新”）：`balanced -> exploratory`。  
2. 太跳（得分波动大）：`exploratory -> balanced` 或 `conservative`。  
3. 撞梗频繁：保持预设不变，优先处理 `REPEAT_WITHIN_SET` 改写信号。  
4. 截止时间近：直接切 `conservative`，优先稳态输出。  

