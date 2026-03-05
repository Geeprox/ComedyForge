# Standup Schema

```yaml
id: "R2-17"
persona_mask: "Millennials中层，社恐，医学滤镜，刚分手"
setup: |
  ...
punchline: |
  ...
rhythm:
  - "[停顿]"
  - "[加速]"
  - "[收]"
callback_potential:
  - "R1-05: 电梯焦虑梗"
style:
  angle: "观察错位"
  expectation_level: "L2"
  attack_level: 7
  intellectual: 4
scores:
  overall: 4.35
  subgroup_scores:
    A: 4.8
    B: 4.2
    C: 3.1
    D: 4.5
  std_dev: 0.8
metadata:
  is_polarizer: false
  is_wildcard: false
```

## 规则
- `setup`必须提供真实可感情境。
- `punchline`必须体现预期违背。
- `expectation_level`必须使用`L0-L5`枚举。
- `style.angle`应来自最新10类角度配额池。
- 必须出现至少一个可回收callback锚点。
