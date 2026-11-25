# vibe-check

[中文](README.md) | English

A Qoder CLI skill for non-programmers: while you vibe-code, it **independently reviews** the code AI writes and the plans AI proposes, and gives verdicts in plain language.

## What it solves

The same AI both writes the code and praises it, so you can't tell good from bad. vibe-check acts as an "independent reviewer + plain-language translator":

- Gathers evidence by actually running the code; never trusts AI's "already tested" claim. Large changes get an adversarial review from a fresh-context subagent.
- Traffic-light verdict (🟢 safe to use / 🟡 has risks / 🔴 rework recommended) + a copy-paste fix instruction you can send straight to the coding AI.
- For plan comparisons: one clear recommendation + a pros/cons table across five plain dimensions — cost / dependency weight / rollbackability / maintainability / risk.

## Install

Requires Qoder CLI. Copy the `vibe-check` folder into your user-level skills directory:

```bash
git clone https://github.com/buguoshixc/vibe-check.git
mkdir -p ~/.qoder-cn/skills
cp -r vibe-check/vibe-check ~/.qoder-cn/skills/
```

Then run `/skills reload` in Qoder (or restart the session) and confirm `vibe-check` appears via `/skills list`.

## Usage

- Manual: type `/vibe-check`. Leave it empty (auto-reviews the latest code/plan) or pass a target.
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
