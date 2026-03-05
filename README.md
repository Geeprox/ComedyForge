# Comedy Forge

[English](./README.en.md)

一个面向脱口秀/漫才创作的协议化 Skill，支持“人格生成 -> 观众模拟 -> 评分进化 -> 评语驱动改写 -> 最终演出组装”的完整流程。

论文风格运行机制详解：  
[《Comedy Forge 运行机制论文式说明（中文）》](./docs/comedy-forge-mechanism-paper.md)  
[English Paper Version](./docs/comedy-forge-mechanism-paper.en.md)

角度预设选择手册：  
[《Angle Preset 选择手册（中文）》](./docs/angle-preset-playbook.md)  
[Angle Preset Playbook (English)](./docs/angle-preset-playbook.en.md)

## 兼容性

本项目不是 Codex 专用，当前同时面向：

- Codex（Skill/Agent 工作流）
- Claude Code（`SKILL.md` 作为安装后入口，`CLAUDE.md`/`.claude/commands/forge.md` 作为仓库模式辅助）

## Claude Code 安装

### 快速安装（推荐）

```bash
npx skill add https://github.com/Geeprox/ComedyForge
```

如果你的环境里 `skill` 命令不可用，再使用下面的兼容别名（功能相同）：

```bash
npx skills add https://github.com/Geeprox/ComedyForge
```

### 手动安装

1. 下载仓库 zip 并解压。
2. 复制到 `~/.claude/skills/comedy-forge`。
3. 确认以下文件存在：
   - `~/.claude/skills/comedy-forge/SKILL.md`
   - `~/.claude/skills/comedy-forge/.comedy-forge/skill.mdc`

### 安装后验证

在 Claude Code 中发起类似请求，若命中本 Skill 即安装成功：

```text
帮我基于“职场汇报焦虑”生成一轮 standup 段子，并给出下一轮改写建议
```

## 项目结构

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

## 核心能力

- `forge` 参数化生成：`type/theme/min-rounds/max-rounds/survival-rate/batch-size/audience-size/absurd-booster`
- 创作者与评审上下文隔离
- Manzai 双人气场优先评分
- Seed 驱动观众情绪分布与评分松紧联动
- 遗传进化 + 外卡扩容 + A/B 版本机制
- 评语驱动改写（结构化信号，不透传原始评语）
- Set List 组合评分 + 二次精修回路

## 使用方式

### Codex

1. 加载 `.comedy-forge/skill.mdc` 及关联协议文件。
2. 触发 `forge` 任务并传入主题与参数。
3. 按 `templates/output-format.md` 输出最终演出稿。

### Claude Code

1. 安装后 Claude Code 会读取根目录 `SKILL.md` 作为技能入口。
2. 运行时按照 `SKILL.md` 指向的 `.comedy-forge/*` 协议执行。
3. 可选使用 `.claude/commands/forge.md` 作为命令模板。

示例意图：

```text
forge type=auto theme="职场汇报焦虑" min-rounds=3 max-rounds=5 survival-rate=0.3 batch-size=50 audience-size=30 absurd-booster=5
```

## 贡献

欢迎提交 Issue / PR。
涉及协议修改时，请同步检查以下文件一致性：

- `system/*`
- `workflows/*`
- `templates/*`
- `.comedy-forge/skill.mdc`
- `CLAUDE.md`（若涉及运行入口变化）

## 许可证

本项目使用 MIT License，详见 [LICENSE](./LICENSE)。
