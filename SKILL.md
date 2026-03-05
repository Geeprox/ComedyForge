---
name: comedy-forge
description: End-to-end stand-up/manzai generation with persona synthesis, audience simulation, evolutionary scoring, comment-driven rewrites, and final set assembly. Use when users ask to generate comedy bits, iterate jokes based on audience feedback, build stage-ready set lists, or refine stand-up/manzai material across rounds.
---

# Comedy Forge Skill Entry

Use this file as the primary entrypoint for skill-based agents (including Claude Code skill installs).

## Execution Order

1. Read `.comedy-forge/skill.mdc` for command schema and hard constraints.
2. Apply system protocols:
   - `.comedy-forge/system/persona-generator.md`
   - `.comedy-forge/system/scoring-protocol.md`
   - `.comedy-forge/system/comment-rewrite-rules.md`
   - `.comedy-forge/system/evolution-rules.md`
   - `.comedy-forge/system/batch-diversity.md`
3. Run workflows:
   - `.comedy-forge/workflows/batch-creator.md`
   - `.comedy-forge/workflows/audience-simulator.md`
   - `.comedy-forge/workflows/distillation-engine.md`
   - `.comedy-forge/workflows/final-assembly.md`
4. Render final output with `.comedy-forge/templates/output-format.md`.

## Command Shape

`forge type=<standup|manzai|auto> theme=<required> min-rounds=<3..20> max-rounds=<5..30> survival-rate=<0.1..0.5> batch-size=<10..500> audience-size=<10..200> absurd-booster=<3..50>`

## Non-Negotiable Rules

- Keep creator and judge contexts isolated.
- Never expose raw judge comments to creator-side prompts.
- For manzai, prioritize duo chemistry over single punch density.
- Enforce wildcard A/B variant flow and candidate pool cap.
- Apply comment-driven rewrite signals only as structured actions.
