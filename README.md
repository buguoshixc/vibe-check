# vibe-check

中文 | [English](README.en.md)

面向不懂编程的同学的 Qoder CLI skill：vibe coding 时替你**独立评审** AI 写的代码和提的方案，用大白话给结论。

## 它解决什么

同一个 AI 既写代码又自夸，你没法判断好坏。vibe-check 当"独立评审员＋大白话翻译官"：

- 实际运行取证，不信 AI 自称"已测试"；大改动派全新上下文的子代理对抗评审
- 红绿灯结论（🟢 放心用 / 🟡 有隐患 / 🔴 建议返工）＋可直接复制发给 AI 的修复指令
- 方案对比给一个明确推荐＋成本／依赖轻重／能否回滚／维护难度／风险五维利弊表

## 安装

需要 Qoder CLI。把 `vibe-check` 文件夹复制到用户级 skills 目录：

```bash
git clone https://github.com/buguoshixc/vibe-check.git
mkdir -p ~/.qoder-cn/skills
cp -r vibe-check/vibe-check ~/.qoder-cn/skills/
```

然后在 Qoder 里执行 `/skills reload`（或重启会话），用 `/skills list` 确认出现 `vibe-check`。

## 用法

- 手动：输入 `/vibe-check`。可留空（自动评最近的代码／方案），也可指定对象。
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
