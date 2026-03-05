# Comedy Forge

[中文](./README.md)

A protocol-driven skill for stand-up/manzai generation, covering the full loop from persona creation to audience simulation, evolutionary scoring, comment-driven rewriting, and final set assembly.

Paper-style mechanism walkthrough:  
[Comedy Forge Mechanism Paper (English)](./docs/comedy-forge-mechanism-paper.en.md)  
[中文论文版本](./docs/comedy-forge-mechanism-paper.md)

Angle preset playbook:  
[Angle Preset Playbook (English)](./docs/angle-preset-playbook.en.md)  
[中文版手册](./docs/angle-preset-playbook.md)

## Compatibility

This repository is not Codex-only. It is designed to work with:

- Codex (skill/agent workflows)
- Claude Code (`SKILL.md` as installed entrypoint, with `CLAUDE.md` / `.claude/commands/forge.md` for repo mode)

## Claude Code Installation

### Quick Install (Recommended)

```bash
npx skill add https://github.com/Geeprox/ComedyForge
```

If `skill` is not available in your environment, use this compatibility alias instead (same behavior):

```bash
npx skills add https://github.com/Geeprox/ComedyForge
```

### Manual Install

1. Download and extract this repository.
2. Copy it to `~/.claude/skills/comedy-forge`.
3. Verify these files exist:
   - `~/.claude/skills/comedy-forge/SKILL.md`
   - `~/.claude/skills/comedy-forge/.comedy-forge/skill.mdc`

### Post-Install Check

In Claude Code, run a request like:

```text
Generate one stand-up round on \"workplace reporting anxiety\" and propose rewrite actions for the next round.
```

If the skill is triggered, installation is successful.

## Repository Structure

```text
.comedy-forge/
├── skill.mdc
├── system/
│   ├── persona-generator.md
│   ├── scoring-protocol.md
│   ├── comment-rewrite-rules.md
│   ├── evolution-rules.md
│   └── batch-diversity.md
├── workflows/
│   ├── batch-creator.md
│   ├── audience-simulator.md
│   ├── distillation-engine.md
│   └── final-assembly.md
├── templates/
│   ├── standup-schema.md
│   ├── manzai-schema.md
│   └── output-format.md
└── context/
    └── hot-topics.md
```

## Core Capabilities

- Parameterized `forge` generation with `type/theme/min-rounds/max-rounds/survival-rate/batch-size/audience-size/absurd-booster/angle-preset`
- Strict creator/judge context isolation
- Duo-chemistry-first scoring for manzai
- Seed-driven audience mood shift and scoring strictness
- Evolution loop with wildcard expansion and A/B (conditional C) variants
- Comment-driven rewrite pipeline (structured signals only)
- Composite set-list scoring and second-pass polishing

## Usage

### Codex

1. Load `.comedy-forge/skill.mdc` and linked protocol files.
2. Trigger a `forge` task with a required theme.
3. Produce final output following `templates/output-format.md`.

### Claude Code

1. After installation, Claude Code uses root `SKILL.md` as the skill entrypoint.
2. Execution follows the protocol files under `.comedy-forge/*`.
3. Optionally use `.claude/commands/forge.md` as a command template.

Example intent:

```text
forge type=auto theme="Workplace reporting anxiety" min-rounds=3 max-rounds=5 survival-rate=0.3 batch-size=50 audience-size=30 absurd-booster=5 angle-preset=balanced
```

## Contributing

Issues and PRs are welcome.
If you change protocols, keep these in sync:

- `system/*`
- `workflows/*`
- `templates/*`
- `.comedy-forge/skill.mdc`
- `CLAUDE.md` (if runtime entry changes)

## License

This project is licensed under MIT. See [LICENSE](./LICENSE).
