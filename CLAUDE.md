# Claude Code Integration Guide

This repository provides a protocol package under `.comedy-forge/`.
When running tasks in Claude Code, use this file as the operational entrypoint.

## Task Trigger

Use `forge` intent with required `theme`, for example:

```text
forge type=auto theme="职场汇报焦虑" min-rounds=3 max-rounds=5 survival-rate=0.3 batch-size=50 audience-size=30 absurd-booster=5
```

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
- Enforce wildcard A/B mechanism and candidate pool cap.
