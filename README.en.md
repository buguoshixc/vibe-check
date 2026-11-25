# vibe-check

[中文](README.md) | English

A skill built on the open **Agent Skills** format (`SKILL.md`), compatible with mainstream AI coding assistants that support it (Claude Code, Qoder CLI, GitHub Copilot agent mode, etc.) — **not tied to any single product**. For non-programmers: while you vibe-code, it **independently reviews** the code AI writes and the plans AI proposes, and gives verdicts in plain language.

## What it solves

The same AI both writes the code and praises it, so you can't tell good from bad. vibe-check acts as an "independent reviewer + plain-language translator":

- Gathers evidence by actually running the code; never trusts AI's "already tested" claim. Large changes get an adversarial review from a fresh-context subagent.
- Traffic-light verdict (🟢 safe to use / 🟡 has risks / 🔴 rework recommended) + a copy-paste fix instruction you can send straight to the coding AI.
- For plan comparisons: one clear recommendation + a pros/cons table across five plain dimensions — cost / dependency weight / rollbackability / maintainability / risk.

## Install

Drop the `vibe-check` folder into your product's skills directory. Common locations:

| Product | Project-level (this project only) | User-level (all projects) |
|---|---|---|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Qoder CLI | `.qoder/skills/` | `~/.qoder-cn/skills/` (international: `~/.qoder/skills/`) |
| GitHub Copilot (agent mode) | `.github/skills/` or `.claude/skills/` | `~/.copilot/skills/` |
| Other Agent Skills-compatible products | their skills directory | same |

User-level install example (replace the target directory for your product):

```bash
git clone https://github.com/buguoshixc/vibe-check.git
mkdir -p ~/.claude/skills          # Claude Code; for Qoder use ~/.qoder-cn/skills
cp -r vibe-check/vibe-check ~/.claude/skills/
```

Then restart the session (Qoder: run `/skills reload`) and confirm `vibe-check` appears in your product's skill list.

## Usage

- Manual: type `/vibe-check` (most products support slash-invoking skills; you can also just name it in chat). Leave it empty to auto-review the latest code/plan, or pass a target.
- Automatic: triggers when you doubt AI's output, e.g. "is this okay?", "is this reliable?", "take a look", "which one should I pick?", "will this break?".

## Examples

- **Code review**: AI finishes a webpage, you ask "is this okay?" → it runs the code, flags risks like "data disappears on refresh", and gives a fix instruction.
- **Plan choice**: "React + database vs a single HTML file, which one?" → five-dimension table + a clear recommendation.

## Structure

```
vibe-check/
├── SKILL.md                    # main workflow
└── references/
    ├── review-checklist.md     # 7-item review checklist
    └── plain-language.md       # term analogies + five-dimension definitions
```

## Compatibility

The skill relies only on universal capabilities: reading files, running code via a shell, and (optionally) spawning an independent subagent. Products without subagent support automatically fall back to self-review and note "no independent review" in the report.
