# /forge

Run Comedy Forge protocol to generate stand-up/manzai material.

## Parameters

- `type`: `standup | manzai | auto` (default: `auto`)
- `theme`: required
- `min-rounds`: `3-20` (default: `3`)
- `max-rounds`: `5-30` (default: `5`)
- `survival-rate`: `0.1-0.5` (default: `0.3`)
- `batch-size`: `10-500` (default: `50`)
- `audience-size`: `10-200` (default: `30`)
- `absurd-booster`: `3-50` (default: `3`)
- `angle-preset`: `balanced | conservative | exploratory` (default: `balanced`)

## Procedure

1. Parse params and validate with `.comedy-forge/skill.mdc`.
2. Execute system/workflow protocols under `.comedy-forge/`.
3. Generate final markdown using `.comedy-forge/templates/output-format.md`.

## Example

```text
/forge type=auto theme="职场汇报焦虑" min-rounds=3 max-rounds=5 survival-rate=0.3 batch-size=50 audience-size=30 absurd-booster=5 angle-preset=balanced
```
