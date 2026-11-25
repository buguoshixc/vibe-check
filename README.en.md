# vibe-check

[中文](README.md) | English

> An "AI code quality inspector" for non-programmers: while you vibe-code, it **independently reviews** the code AI writes and the plans AI proposes, and tells you in plain language whether it's usable, what the pitfalls are, and what to do next.

Built on the open **Agent Skills** format (`SKILL.md`), compatible with mainstream AI coding assistants that support it — Claude Code, Qoder CLI, GitHub Copilot (agent mode), etc. — **not tied to any single product**.

---

## Table of contents

- [What problem it solves](#what-problem-it-solves)
- [Quick start](#quick-start)
- [Core features](#core-features)
- [How it works](#how-it-works)
- [What the output looks like](#what-the-output-looks-like)
- [The five comparison dimensions](#the-five-comparison-dimensions)
- [Install](#install)
- [Usage](#usage)
- [Design principles](#design-principles)
- [Compatibility & limitations](#compatibility--limitations)
- [Project structure](#project-structure)
- [FAQ](#faq)
- [Iteration & contributing](#iteration--contributing)

---

## What problem it solves

When non-programmers build software with AI (vibe coding), there's an unavoidable trap:

**The same AI both writes the code and judges it.** It says "already tested, fully working" and you can't verify; it offers three or four technical options and you can't tell them apart; it produces a pile of code and you don't know what's lurking inside. So you're left "trusting it" — and it's often overconfident.

vibe-check breaks that trap by playing two roles:

1. **Independent reviewer**: takes a "find problems" stance rather than defending the author, **gathers evidence by actually running the code**, never trusts the AI's self-report, and gives a clear "usable or not" verdict.
2. **Plain-language translator**: turns jargon into everyday analogies and technical trade-offs into dimensions you understand (does it cost money, will you be locked in, can you undo it if it breaks) — so you're **able to make the call**.

It doesn't write code for you; it helps you **understand and vet** the code and plans AI produces.

## Quick start

```bash
git clone https://github.com/buguoshixc/vibe-check.git
mkdir -p ~/.claude/skills          # replace with your product's directory (see Install)
cp -r vibe-check/vibe-check ~/.claude/skills/
```

Restart the session, then ask "is this okay?" about AI's output or type `/vibe-check`, and it starts reviewing.

## Core features

### 1. Independent reviewer

- **Code review**: read the changes → actually run them (run scripts / start a server / run tests) → check against a 7-item checklist with evidence → give a verdict.
- **Plan review**: check whether the plan matches your real goal (coursework? personal use? for others?), spot over-engineering, and compare trade-offs across five dimensions.
- **Adversarial independent review**: for large changes or anything touching data deletion, secrets, or payments, it spawns a **fresh-context subagent** to re-review, avoiding "grading your own homework".
- **Traffic-light verdict**: 🟢 safe to use / 🟡 has risks / 🔴 rework recommended, with a one-line reason.
- **Copy-paste fix instruction**: generates a message you can send straight to the coding AI, specific down to files and behavior — you don't have to phrase it yourself.

### 2. Plain-language translator (active for the whole session once triggered)

- Every term gets an analogy on first use (API = waiter, database = ledger, rollback = undo button…).
- For plans it gives **one clear recommendation + a pros/cons table**, not a dump of options for you to choose from.
- Review reports stay ≤400 words: verdict first, details on request — no wall of text to intimidate you.
- Language follows yours.

## How it works

Four steps; the first two are mandatory:

```
① Pick the target → ② Gather evidence (actually run) → ③ Spawn independent subagent for big changes → ④ Output in a fixed template
```

**① Pick the target**: if you specify one, review it; otherwise look at the recent conversation — just wrote code → review code, just proposed a plan → review the plan. Pure concept questions ("what is X") get a plain-language explanation, not a review.

**② Gather evidence (non-skippable)**: run scripts with normal and boundary inputs; for webpages start a local server to confirm they open, then verify key behavior; run tests if present. **If it can't run, it says so ("not actually run, based on reading only") and downgrades the verdict.** Operations that cost money, delete data, or send external requests ask you first.

**③ Independent subagent for big changes**: over ~100 lines / across multiple files / touching data deletion, secrets, or payments → spawn a fresh-context subagent to find faults, **without showing it prior conclusions or the AI's self-report**. Small changes are self-reviewed to save cost.

**④ Fixed-template output**: traffic light → issue list (≤5, by severity) → comparison table (plan reviews) → copy-paste fix instruction → evidence footnote (what ran, what wasn't verified).

### Traffic-light criteria

| Verdict | Code review | Plan review |
|---|---|---|
| 🟢 safe to use | Verified, no high/medium-severity issues | Matches goal, no high risk, costs stated |
| 🟡 has risks | Usable but risky, or partly unverified | Right direction, but costs to guard against |
| 🔴 rework recommended | Loses data / leaks / crashes, or doesn't do what you asked | Directionally wrong (heavy over-engineering / dangerous ops) |

**When unsure, prefer 🟡 over 🟢.**

### Honesty principle

- Only report run results it actually saw; never fabricate.
- Clearly mark unverified parts; never dress uncertainty up as certainty.
- Don't make excuses for the author when review fails, but keep fix instructions concrete and actionable — not just "rewrite it".

## What the output looks like

**Code review example** (a todo webpage that "claimed to be tested and cloud-synced", after actually running it):

```
🔴 Rework recommended
Core problem: todos vanish on refresh, and "cloud sync" is a shell — claimed features aren't delivered.

Issues (by severity)
1. Todos live only in "memory", never written to "disk": gone on refresh, like writing in sand (high)
2. "Cloud sync" is fake: zero network calls in testing, claim is false (medium)
3. Key taped to the door: secret hardcoded in the page; a real key would be exposed to anyone (medium)
...

Next step (copy-paste to the coding AI)
> Edit todo.html: persist todos with localStorage; remove the unused secret;
> render with textContent, never innerHTML-concat input; reject empty input; self-test that refresh keeps data.

---
Evidence: server returned HTTP 200; simulation showed list cleared on refresh, 0 network calls.
Not verified: no real browser, click experience untested.
```

**Plan comparison example** ("React + database vs a single HTML file for a budget page"): see the table under [the five dimensions](#the-five-comparison-dimensions).

## The five comparison dimensions

No jargon — only what you actually care about:

| Dimension | Plain meaning | Question to ask |
|---|---|---|
| Cost | Money and time | Need a paid server/service? How long to set up? (student budget ≈ 0) |
| Dependency weight | How much extra stuff, and lock-in | How many packages? How much rewriting to switch later? |
| Rollbackability | Can you undo a mess | Is there a save point? Does this step delete data / spend money? |
| Maintainability | How many places to touch for a small change | Can you read and change it yourself? |
| Risk | Worst case | Lose data? Leak? Waste a week? How likely? |

Example table (personal budget page):

| | Plan A: React + database | Plan B: single HTML file |
|---|---|---|
| Cost | Paid server, days to build | Free, usable in minutes |
| Dependency weight | Three big pieces, rewrite to switch | No parts, double-click to open |
| Rollbackability | Many moving parts, hard to undo | Copy the file = save point |
| Maintainability | Change one thing, touch three; you can't fix it | One file, AI can help edit |
| Risk | Waste money/effort, no one to fix it | Clearing browser loses data; backup prevents it |

> I recommend B. What matters most for you is personal use, zero budget, and not coding — B wins on all three. A's upside is handling many users and lots of data, but the price is ongoing cost and maintenance you can't manage.

## Install

Drop the `vibe-check` folder into your product's skills directory. Common locations:

| Product | Project-level (this project only) | User-level (all projects) |
|---|---|---|
| Claude Code | `.claude/skills/` | `~/.claude/skills/` |
| Qoder CLI | `.qoder/skills/` | `~/.qoder-cn/skills/` (international: `~/.qoder/skills/`) |
| GitHub Copilot (agent mode) | `.github/skills/` or `.claude/skills/` | `~/.copilot/skills/` |
| Other Agent Skills-compatible products | their skills directory | same |

**Step-by-step (Claude Code user-level example):**

```bash
# 1. Clone
git clone https://github.com/buguoshixc/vibe-check.git
cd vibe-check

# 2. Copy into the skills directory (for Qoder use ~/.qoder-cn/skills)
mkdir -p ~/.claude/skills
cp -r vibe-check ~/.claude/skills/

# 3. Restart the session; Qoder can run /skills reload
```

**Verify**: confirm `vibe-check` appears in your product's skill list (Qoder: `/skills list`). Once installed it auto-triggers when you express doubt, or invoke `/vibe-check` manually.

> Project-level vs user-level: install user-level to use it across all projects; install project-level (in that project's root folder) to scope it to one project.

## Usage

**Manual**: type `/vibe-check`.

- Empty: auto-review the latest code/plan.
- With a target: `/vibe-check this code` or `/vibe-check which plan should I pick` to review what you specify.
- Most products support slash-invoking skills; if yours doesn't, just name it in chat ("use vibe-check to take a look").

**Automatic**: triggers when you doubt AI's output, e.g.:

- "Is this okay?" / "Is this reliable?" / "Will this break?"
- "Take a look, any problems?" / "How's this code quality?"
- "Which one should I pick?" / "What's the difference between these two plans?"

**After triggering**: the plain-language rules stay active for the whole session; deep reviews happen on demand, not on every message.

## Design principles

- **Evidence over claims**: "already tested" doesn't count; run results do. Concluding without running is the cardinal violation.
- **Independent from the author**: the stance is finding problems, not defending; big changes use a fresh-context subagent to avoid self-endorsement.
- **Plain language over jargon**: if the user can't understand the term, you've said nothing.
- **Give judgment, not options**: a clear recommendation + trade-offs, not handing the choice back to someone who can't choose.
- **Honest degradation**: if it can't verify, it says so and lowers confidence — never packages uncertainty as certainty.

## Compatibility & limitations

**Required capabilities** (all universal, no product dependency):

- Reading files and running code via a shell (evidence gathering).
- (Optional) spawning an independent subagent. Products without subagent support **automatically fall back to self-review** and note "no independent review" in the report.

**Known limitations**:

- Without a real browser, webpages can only be verified for "opens + logic check"; click experience / appearance is marked "unverified".
- Independent subagents add tokens and latency, so they're only used for big changes / high risk.
- Review is judgment work: evidence-based but still blind spots — it helps you vet, not replace your final decision.

## Project structure

```
vibe-check/
├── SKILL.md                    # main flow: role, 4-step review, output template, honesty
└── references/
    ├── review-checklist.md     # 7-item checklist: delivery/correctness/security/data-loss/over-engineering/dependencies/rollback
    └── plain-language.md       # term-analogy table + five-dimension definitions + recommendation phrasing
```

`SKILL.md` keeps the main flow lean; details live in `references/` and load on demand to save context.

## FAQ

**Q: How is this different from just asking the AI to "check the code"?**
A: Asking the coding AI to self-check is letting it grade itself. vibe-check forces evidence by running, uses an independent adversarial subagent, checks a fixed checklist item by item, and gives plain-language verdicts — targeting exactly the "both athlete and referee" problem.

**Q: I don't code at all — can I understand the report?**
A: Yes. It leads with a traffic-light verdict, writes every issue as "what this means for you" in plain language, gives analogies for terms, and keeps the body ≤400 words. Ask follow-ups for anything unclear.

**Q: Will it change my code?**
A: No. It only reviews and gives verdicts and fix instructions. Whether and how to change is up to you (or passed to the coding AI).

**Q: Which products are supported?**
A: Any product supporting the open Agent Skills (`SKILL.md`) format — see the [Install table](#install). It relies only on universal capabilities: reading files, running code, (optional) subagents.

**Q: Does reviewing cost money?**
A: The skill itself is free. Reviewing uses your AI product's normal quota; independent subagents use a bit more, so they're reserved for big changes.

## Iteration & contributing

To improve it: edit `vibe-check/SKILL.md` or files under `references/`, re-copy into the skills directory, and restart the session (Qoder: `/skills reload`). Issues and PRs welcome — especially more term analogies, checklist items, and real-world failure cases.
