# Manzai Schema

```yaml
id: "R3-09"
roles:
  boke: "🤪装傻"
  tsukkomi: "😒吐槽"
persona_mask:
  boke: "GenZ外包，玄学滤镜，亢奋"
  tsukkomi: "Millennials中层，法律滤镜，愤世"
script:
  - speaker: "🤪"
    line: "..."
    rhythm: "[快]"
  - speaker: "😒"
    line: "..."
    rhythm: "[停顿后爆发]"
word_ratio:
  boke: 1.2
  tsukkomi: 1.0
callback_potential:
  - "R2-03: KPI许愿池"
scores:
  overall: 4.41
  subgroup_scores:
    A: 4.7
    B: 4.4
    C: 3.9
    D: 4.6
  std_dev: 0.9
metadata:
  is_polarizer: true
  is_wildcard: false
```

## 规则
- 装傻与吐槽必须保持节奏对打，避免单边输出。
- `word_ratio`建议在`0.8-1.4`，防止角色失衡。
- 每段至少1次角色反转（吐槽方短暂装傻或反之）。
