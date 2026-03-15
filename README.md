# ZeroGuess

**Stop guessing. Start contracting.**

ZeroGuess finds every implementation fork, scores them by impact and reversibility, auto-resolves what your project already decides, asks about the rest, and writes a **machine-readable `.zeroguess` contract** that enforces decisions during implementation.

Not a chatty Q&A session. A 2-5 minute interrogation that produces an enforceable contract.

## The Problem

Every clarification tool — including good ones — produces prose summaries that get forgotten the moment implementation starts. The agent asks 4 great questions, gets answers, writes a paragraph, then silently makes 20 more decisions on its own.

ZeroGuess fixes the full pipeline: **clarify → score → contract → enforce.**

## How It Works

```
Request
  ↓
Silent Fork Analysis (internal)
  - Score each fork: impact × reversibility × context available
  - Auto-resolve: project already has Tailwind? Lock it, don't ask.
  - Kill: low-impact, easily reversible? Skip.
  ↓
Declare Auto-Resolved (show what you already know)
  "Stack: Next.js, Prisma, Tailwind ✓ — correct me if wrong"
  ↓
Ask Remaining Forks (max 5, one at a time)
  Each with Surprise Me option
  ↓
Write .zeroguess Contract (machine-readable YAML)
  ↓
Implementation: agent checks contract before every decision
  - In contract → use locked value
  - In open_questions → STOP and ask
  - Not mentioned + high impact → STOP and ask
```

## The `.zeroguess` Contract

```yaml
# .zeroguess — Decision Contract
project: crypto-dashboard
created: 2026-03-15

decisions:
  platform:
    value: web
    source: user
    impact: high
    reversibility: hard
    locked: true

  database:
    value: PostgreSQL via Prisma
    source: auto-resolved   # found prisma/schema.prisma
    impact: high
    reversibility: hard
    locked: true

open_questions:
  - "Caching strategy"
  - "Error monitoring"
```

Agents read this file. Locked decisions are respected. Open questions trigger a stop-and-ask. No more silent assumptions.

## Fork Scoring

Not fake math. Categorical heuristics:

| Dimension | High | Medium | Low |
|-----------|------|--------|-----|
| **Impact** | Changes DB + API + frontend | Changes one layer | Cosmetic |
| **Reversibility** | Migration needed | Refactor needed | Swap anytime |
| **Context** | Nothing hints at answer | Partial clues | Already decided |

- HIGH impact + HARD to reverse + NO context → **must ask**
- HIGH impact + context available → **auto-resolve** (declare, don't ask)
- LOW impact + easy to reverse → **skip**

## What's Different

| Feature | Typical clarifier | ZeroGuess |
|---------|------------------|-----------|
| Output | Prose summary (forgotten) | `.zeroguess` contract (enforced) |
| Project awareness | Sometimes reads files | Scores forks against project context |
| Auto-resolve | No | Yes — declares known answers, skips questions |
| Implementation enforcement | None | Contract checked during coding |
| Cross-session memory | None | `.zeroguess` persists in repo |
| Surprise Me | No | Every question |
| Question limit | None | Hard cap at 5 |

## Install

```bash
# For GSD / pi users
mkdir -p ~/.agents/skills/zeroguess

# For Claude Code users  
mkdir -p ~/.claude/skills/zeroguess
```

Copy [`SKILL.md`](./SKILL.md) into the directory. Restart your agent.

## Usage

```
/zeroguess        # standalone
/gsd-zeroguess    # with GSD
```

The `.zeroguess` file is committed to your repo. Any agent that reads it will respect your decisions.

## License

MIT

## Credits

Built by [@toismfer](https://github.com/toismfer)

⭐ Star if it saves you from silent assumptions
