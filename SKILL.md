---
name: zeroguess
description: "Finds every implementation-forking decision, scores them by impact/reversibility, forces explicit answers, and writes a machine-readable .zeroguess contract that enforces decisions during implementation. Zero guesses. Works standalone or with GSD (/gsd-zeroguess)."
---

# ZeroGuess

## Purpose

Always respond in the exact language the user is using (auto-detect).
Find every decision point where guessing wrong would waste work.
Score each fork. Ask about real ones. Auto-resolve obvious ones. Write a contract. Enforce it.

This is NOT brainstorming. This is a 2-5 minute interrogation that produces a **machine-readable decision contract** — not prose that gets forgotten.

<HARD-GATE>
Do NOT write code, create files (except .zeroguess), generate designs, invoke other skills, or take ANY implementation action.
This skill does exactly one thing: clarify intent → write contract → stop.
</HARD-GATE>

## When to Use

- Request has ambiguous forks
- Before brainstorming, before planning, before coding
- Explicitly invoked with `/zeroguess` or `/gsd-zeroguess`
- Resuming work on a project (load existing `.zeroguess` contract)

## When NOT to Use

- The request is already crystal-clear with no forks (say "Nothing to clarify. Ready." and get out)
- The user says "just do it" or "you decide" — respect that, exit immediately
- The user is ideating, not building ("I want to build something cool", "give me side project ideas") — they need brainstorming, not decision contracts
- Mid-implementation detail questions — those belong in the implementation phase

## Process

### Step 1: Silent Fork Analysis (never shown to user)

Before asking anything, do this internally (do NOT show this to the user):

1. **Read the project** — check files, configs, dependencies, existing patterns
2. **List 3-5 plausible interpretations** of what the user wants
3. **Identify forks** — where different interpretations lead to completely different code
4. **Score each fork** using the heuristic below
5. **Auto-resolve** forks that can be answered from project context
6. **Rank remaining forks** by score — ask highest first

#### Fork Scoring Heuristic

For each fork, assess four dimensions:

| Dimension | High | Medium | Low |
|-----------|------|--------|-----|
| **Downstream Impact** | Changes DB schema, API, and frontend | Changes one layer | Cosmetic only |
| **Reversibility** | Migration/rewrite needed | Refactor needed | Swap anytime |
| **Context Available** | Nothing in project hints at answer | Partial clues | Config/file already decides it |
| **Intent Clarity** | Purpose/motivation is unknown ("why?") | Partially clear | Obviously understood |

**Decision rules:**
- Intent Clarity LOW → **MUST ASK FIRST** (always priority 0 — before any technical fork)
- Impact HIGH + Reversibility HARD + Context NO → **MUST ASK** (priority 1)
- Impact HIGH + Context YES → **AUTO-RESOLVE** (declare, don't ask)
- Impact LOW + Reversibility EASY → **SKIP** (not a real fork)

**Greenfield rule:** When there is no project context (empty directory, no configs), intent forks outrank technical forks. Ask "what and why" before "how."
- Impact MEDIUM + Context PARTIAL → **ASK WITH DEFAULT** (Surprise Me = the default)

A fork is NOT:
- A cosmetic choice (colors, fonts, labels) — Impact LOW, skip
- A trivial default (naming, formatting) — use project patterns, skip
- Something the project already decided — auto-resolve

A fork IS:
- Platform (web vs CLI vs mobile) — different stack entirely
- Scope (single-user vs multi-user) — 10x complexity difference
- Persistence (in-memory vs file vs database) — different architecture
- Integration method (REST vs GraphQL vs RPC) — different patterns

### Step 2: Declare Auto-Resolved Decisions

Before asking any questions, show what you already know:

```
## Already Decided (from your project)
- Stack: Next.js 16, TypeScript, Tailwind ✓
- Database: PostgreSQL via Prisma ✓
- Auth: Supabase Auth ✓

Correct me if any are wrong. Otherwise I'll lock these in.
```

This eliminates 2-3 questions that other skills would waste time asking.

### Step 3: Ask One Fork at a Time

For each remaining MUST-ASK fork, starting with the highest-impact one:

1. Use `ask_user_questions` with 2-3 options **in the user's language**
2. Each option must be **concrete and specific** — describe a real thing you'd build
3. The last option should be **"Surprise Me"** — your best guess with a 1-sentence preview
4. Wait for the answer before asking the next question

**Rules for questions:**
- **ONE question per message.** Never batch.
- **2-3 options max.** The user picks, not types.
- **No abstract options.** Bad: "Monolithic vs Microservices." Good: "Single Express server with all routes, or separate Lambda per endpoint?"
- **No jargon unless the user used it first.** Match their vocabulary.
- **Every option = a real thing.** The user should be able to picture what they'd get.

### Step 4: Stop When Done

- No forks left → Go to Step 5
- User says "enough" or "just go" → Go to Step 5
- 5 questions reached → hard stop, go to Step 5

### Step 5: Write Contract + Handoff

**Write the `.zeroguess` file** to the project root:

```yaml
# .zeroguess — Decision Contract
# Auto-generated by ZeroGuess. Source of truth for this project's decisions.
# Agents: check this file before making architectural choices.
# Humans: edit freely. Agents will respect your changes.

project: my-project
created: 2026-03-15
updated: 2026-03-15

decisions:
  platform:
    value: web
    source: user          # user | auto-resolved | default
    impact: high
    reversibility: hard
    locked: true

  data_source:
    value: paste addresses, auto-fetch via public APIs
    source: user
    impact: high
    reversibility: hard
    locked: true

  database:
    value: PostgreSQL via Prisma
    source: auto-resolved  # found prisma/schema.prisma
    impact: high
    reversibility: hard
    locked: true

  styling:
    value: Tailwind CSS
    source: auto-resolved  # found tailwind.config.ts
    impact: low
    reversibility: easy
    locked: true

# Decisions NOT YET MADE — agents must ask before deciding these:
open_questions:
  - "Caching strategy (Redis vs in-memory vs none)"
  - "Error monitoring (Sentry vs custom vs none)"
```

**Then show summary and handoff:**

```
## Contract Written → .zeroguess

Locked: 4 decisions (2 from you, 2 auto-resolved)
Open: 2 questions (will be asked during implementation)

We're aligned. Ready to move forward.
```

Then call `ask_user_questions` with these options (in the user's language):

- **Brainstorm** — Explore ideas and design possibilities together
- **Detailed Plan** — Build a step-by-step execution plan
- **Surprise Me — Start building the best version right now**

### Step 6: Contract Enforcement (for implementing agents)

**Any agent working on this project SHOULD read `.zeroguess` before starting.**

When an agent is about to make a decision:
1. Check if it's in `decisions:` → use the locked value
2. Check if it's in `open_questions:` → **stop and ask the user**
3. Not mentioned at all → if impact is HIGH or reversibility is HARD, **stop and ask**. Otherwise, proceed and note the assumption.

After asking, update `.zeroguess` with the new decision.

## Anti-Patterns (Do NOT Do These)

1. **Asking questions you can answer from context.** If there's a `tsconfig.json`, auto-resolve TypeScript. Don't ask.
2. **Asking questions where you'll ignore the answer.** If you'd use React either way, don't ask.
3. **Padding with safe questions.** "What should we name the project?" is not a fork. Skip it.
4. **Asking permission to ask.** Don't say "I have a few questions, is that okay?" Just ask.
5. **Showing your fork analysis.** The scoring is internal. The user sees declarations, questions, and a contract.
6. **Making up numbers.** Don't pretend to calculate entropy, EIG, or probabilities. The heuristic uses categorical assessment (high/medium/low), not fake math.
7. **Writing prose summaries instead of contracts.** The `.zeroguess` file IS the output. Not a paragraph.

## Examples

### Example 1: Ambiguous Request (Empty Project)

User: "Build me a chat app"

**Internal fork analysis (NOT shown to user):**
- 4 interpretations → 3 real forks
- Platform: HIGH impact, HARD to reverse, NO context → MUST ASK
- Purpose: HIGH impact, HARD to reverse, NO context → MUST ASK
- Styling: LOW impact, EASY to reverse → SKIP

**Step 2:** "No existing project context found."

**Question 1:** "Who's chatting with whom?"
- Option 1: People chatting with each other (like Slack/Discord)
- Option 2: User chatting with an AI (like ChatGPT)
- Option 3: Surprise me — a real-time group chat with rooms, WebSocket, and a clean React UI

### Example 2: Clear Request (Existing Project)

User: "Add a dark mode toggle to the settings page"

**Internal:** No forks. Project has settings page + theme system.

**Response:** "Nothing to clarify — I know what you need. Ready to implement."

### Example 3: Existing Project with Context

User: "Build a notification system" (Next.js + Prisma + Supabase project)

**Step 2 output:**
```
## Already Decided (from your project)
- Stack: Next.js, TypeScript, Tailwind ✓
- Database: PostgreSQL via Prisma ✓
- Auth: Supabase Auth ✓
- Real-time: Supabase Realtime available ✓
```

**Only asks:** delivery method (in-app vs email vs push) and trigger model (event-driven vs scheduled).
Skips stack, DB, auth, and real-time questions entirely.

### Example 4: Resuming a Project

User opens project that has `.zeroguess` file.

**Response:**
```
Found existing contract (.zeroguess):
- Platform: web ✓
- Data source: auto-fetch ✓
- Chains: EVM + Solana ✓

2 open questions remaining:
- Caching strategy
- Error monitoring

Continue with these, or re-clarify?
```
