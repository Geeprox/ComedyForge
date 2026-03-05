# Claude Code Integration Guide

This repository provides a protocol package under `.comedy-forge/`.
When running tasks in Claude Code, use this file as the operational entrypoint.

## Task Trigger

Use `forge` intent with required `theme`, for example:

```text
forge type=auto theme="职场汇报焦虑" min-rounds=3 max-rounds=5 survival-rate=0.3 batch-size=50 audience-size=30 absurd-booster=5 angle-preset=balanced
```

Parameters:

- `type`: `standup | manzai | auto` (default: `auto`)
- `theme`: required
- `min-rounds`: `3-20` (default: `3`)
- `max-rounds`: `5-30` (default: `5`)
- `survival-rate`: `0.1-0.5` (default: `0.3`)
- `batch-size`: `10-500` (default: `50`)
- `audience-size`: `10-200` (default: `30`)
- `absurd-booster`: `3-50` (default: `3`)
- `angle-preset`: `balanced | conservative | exploratory` (default: `balanced`)

## Execution Contract

1. Read `.comedy-forge/skill.mdc` for command schema and hard constraints.
2. Apply `system/*` protocols in this order:
   - `persona-generator.md`
   - `scoring-protocol.md`
   - `comment-rewrite-rules.md`
   - `evolution-rules.md`
   - `batch-diversity.md`
3. Execute `workflows/*` in order:
   - `batch-creator.md`
   - `audience-simulator.md`
   - `distillation-engine.md`
   - `final-assembly.md`
4. Render output by `templates/output-format.md`.

## Guardrails

- Keep creator and judge contexts isolated.
- Do not expose raw judge comments to creator side.
- For manzai, prioritize duo chemistry over single punch density.
- Enforce wildcard A/B mechanism (with conditional C) and candidate pool cap.
