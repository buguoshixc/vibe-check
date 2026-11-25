# vibe-check

中文 | [English](README.en.md)

一个遵循开放 **Agent Skills** 格式（`SKILL.md`）的技能，适用于支持该格式的主流 AI 编程助手（Claude Code、Qoder CLI、GitHub Copilot agent 模式等），**不绑定单一产品**。面向不懂编程的同学：vibe coding 时替你**独立评审** AI 写的代码和提的方案，用大白话给结论。

## 它解决什么

同一个 AI 既写代码又自夸，你没法判断好坏。vibe-check 当"独立评审员＋大白话翻译官"：

- 实际运行取证，不信 AI 自称"已测试"；大改动派全新上下文的子代理对抗评审
- 红绿灯结论（🟢 放心用 / 🟡 有隐患 / 🔴 建议返工）＋可直接复制发给 AI 的修复指令
- 方案对比给一个明确推荐＋成本／依赖轻重／能否回滚／维护难度／风险五维利弊表

## 安装

把 `vibe-check` 文件夹放进你所用产品的 skills 目录即可。常见位置：

| 产品 | 项目级（仅当前项目） | 用户级（所有项目） |
|---|---|---|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Qoder CLI | `.qoder/skills/` | `~/.qoder-cn/skills/`（国际版 `~/.qoder/skills/`） |
| GitHub Copilot（agent 模式） | `.github/skills/` 或 `.claude/skills/` | `~/.copilot/skills/` |
| 其他支持 Agent Skills 的产品 | 各自的 skills 目录 | 同左 |

以用户级安装为例（按你的产品替换目标目录）：

```bash
git clone https://github.com/buguoshixc/vibe-check.git
mkdir -p ~/.claude/skills          # Claude Code；Qoder 用 ~/.qoder-cn/skills
cp -r vibe-check/vibe-check ~/.claude/skills/
```

装好后重启会话（Qoder 可执行 `/skills reload`），用产品的技能列表确认出现 `vibe-check`。

## 用法

- 手动：输入 `/vibe-check`（多数产品支持斜杠调用技能，也可在对话里直接点名让它评审）。可留空自动评最近的代码／方案，或指定对象。
- 自动：对 AI 产出表达疑虑时触发，比如"这样行吗""靠谱吗""帮我看看""该选哪个""会不会翻车"。

## 示例

- **代码评审**：AI 写完一个网页，你问"这样行吗" → 它实际运行、指出"刷新就丢数据"等隐患、给修复指令。
- **方案选型**："React＋数据库 vs 单文件 HTML，选哪个？" → 五维对比表＋明确推荐。

## 结构

```
vibe-check/
├── SKILL.md                    # 主流程
└── references/
    ├── review-checklist.md     # 7 项评审清单
    └── plain-language.md       # 术语比方对照＋五维度定义
```

## 兼容性说明

本技能只依赖通用能力：读文件、用 shell 运行代码、（可选）派独立子代理。没有子代理能力的产品会自动降级为自审，并在报告里注明"未派独立评审"。
