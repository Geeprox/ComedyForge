# Comedy Forge Angle Preset Playbook

This playbook helps you choose between `balanced / conservative / exploratory` and keep angle distribution aligned with your goal.

## 1. Preset Quick View

| Preset | Goal | Core/Long-tail Bias | Best For |
|---|---|---|---|
| `balanced` | Stability + exploration | ~75 / 25 | Default choice, daily iteration |
| `conservative` | Reliable output, lower risk | ~85 / 15 | Show-ready runs, delivery mode, tight timeline |
| `exploratory` | Higher novelty and experiments | ~65 / 35 | Workshops, idea expansion, R&D rounds |

## 2. Fast Decision Rule

1. Not sure: start with `balanced`.  
2. Need stable output now: use `conservative`.  
3. Need fresh material now: use `exploratory`.  
4. If `quota_summary=FAIL` for 2 consecutive rounds: return to `balanced`.  

## 3. Batch-Size Guidance

| batch-size | Recommendation |
|---|---|
| `10-29` | Prefer `balanced` or `conservative`; use `exploratory` only for experiments |
| `30-79` | Start with `balanced`; switch to `exploratory` for creative pushes |
| `80-200` | All three are viable; switch by objective and health panel signals |
| `201-500` | Prefer `balanced` or `exploratory`; monitor long-tail share and hit rate |

## 4. Suggested Operating Flow

1. Round 1: run `balanced` as baseline.  
2. Check the “Angle Quota Execution Panel”:  
   - If `angle_quota_hit_rate >= 80%` and `quota_summary=PASS`, keep preset.  
   - If hit rate drops or `WARN/FAIL`, go back to `balanced` first.  
3. For performance stability, move toward `conservative`.  
4. For material expansion, move toward `exploratory`.  

## 5. Command Examples

```text
forge type=auto theme="Workplace reporting anxiety" min-rounds=3 max-rounds=5 survival-rate=0.3 batch-size=50 audience-size=30 absurd-booster=5 angle-preset=balanced
```

```text
forge type=standup theme="AI meeting jargon" min-rounds=3 max-rounds=6 survival-rate=0.28 batch-size=80 audience-size=40 absurd-booster=6 angle-preset=conservative
```

```text
forge type=auto theme="Rental-era emotional bills" min-rounds=4 max-rounds=7 survival-rate=0.25 batch-size=120 audience-size=60 absurd-booster=8 angle-preset=exploratory
```

## 6. Metrics To Watch Before Switching

Prioritize:

1. `angle_quota_hit_rate`: target `>= 80%`.  
2. `long_tail_share_target/actual`: actual should not significantly lag target.  
3. `quota_summary`: `PASS / WARN / FAIL`.  
4. `top1_share`: avoid one dominant angle taking over.  

## 7. Common Strategies

1. Too safe (“works but not fresh”): `balanced -> exploratory`.  
2. Too volatile (score variance too high): `exploratory -> balanced` or `conservative`.  
3. Frequent overlap: keep preset, prioritize `REPEAT_WITHIN_SET` rewrite actions.  
4. Deadline mode: switch to `conservative` and optimize for stability.  

