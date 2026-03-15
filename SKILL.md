---
name: zeroguess
description: "Finds every implementation-forking decision and forces explicit answers via ask_user_questions. Zero guesses. Not brainstorming, not planning — pure clarification only. Perfect before any build. Works standalone or with GSD (/gsd-zeroguess)."
---

# ZeroGuess

## Purpose

Always respond in the exact language the user is using (auto-detect).
Find every decision point where guessing wrong would waste work. Ask about each one. Never guess.

This is NOT a design process. This is NOT brainstorming. This is a 2-5 minute interrogation that ensures the LLM and the user agree on what's being built before anyone writes a single line.

<HARD-GATE>
Do NOT write code, create files, generate designs, invoke other skills, or take ANY implementation action.
This skill does exactly one thing: clarify intent. When intent is clear, it outputs a summary and stops.
</HARD-GATE>

## When to Use

- Request has ambiguous forks
- Before brainstorming, before planning, before coding
- Explicitly invoked with `/zeroguess` or `/gsd-zeroguess`

## When NOT to Use

- The request is already crystal-clear with no forks (say "Nothing to clarify. Ready." and get out)
- The user says "just do it" or "you decide" — respect that, exit immediately
- Mid-implementation detail questions — those belong in the implementation phase

## Process

### Step 1: Silent Fork Analysis (never shown to user)

Before asking anything, do this internally (do NOT show this to the user):

1. **List 3-5 plausible interpretations** of what the user wants
2. **Compare them**: where do they diverge? What decisions would lead to fundamentally different code?
3. **Identify real forks**: a fork is a decision where answer A leads to completely different implementation than answer B
4. **Kill non-forks**: if all interpretations agree on something, it's not a fork — don't ask about it
5. **Rank forks by impact**: which fork affects the most downstream decisions?

A fork is NOT:
- A cosmetic choice (colors, fonts, labels) — these don't change architecture
- A trivial default (indentation, naming convention) — use the project's existing patterns
- An implementation detail the user shouldn't need to care about

A fork IS:
- Platform (web vs CLI vs mobile) — completely different stack
- Scope (single-user vs multi-user) — 10x complexity difference
- Persistence (in-memory vs file vs database) — different architecture
- Framework choice (when multiple are viable) — different patterns and dependencies

### Step 2: Ask One Fork at a Time

For each fork, starting with the highest-impact one:

1. Use `ask_user_questions` with 2-3 options **in the user's language**
2. Each option must be **concrete and specific** — describe a real thing you'd build
3. The last option should be **"Surprise Me"** — your best guess with a 1-sentence preview of what you'd build
4. Wait for the answer before asking the next question

**Rules for questions:**
- **ONE question per message.** Never batch.
- **2-3 options max.** The user picks, not types.
- **No abstract options.** Bad: "Monolithic vs Microservices." Good: "Single Express server with all routes, or separate Lambda functions per endpoint?"
- **No jargon unless the user used it first.** Match their vocabulary.
- **Every option = a real thing.** The user should be able to picture what they'd get.

**The "Surprise Me" option:**
- Always the last option
- Label: "Surprise me — [1-sentence description of what you'd build]"
- This is your honest best guess based on everything you know so far
- Description should be specific enough that the user can say "yes, that" or "no, not that"

### Step 3: Stop When Done

After each answer, reassess:
- Are there remaining forks? → Ask the next one
- No more forks? → Go to Step 4
- User says "enough" or "just go"? → Go to Step 4 immediately

**Self-termination rule:** If you've asked 5 questions and still have forks, stop anyway. Present your best-guess summary and let the user correct it. More than 5 questions means you're over-clarifying.

### Step 4: Final Summary & Handoff

Output a brief summary, then immediately call `ask_user_questions` for the next step:

```
## What I Understand

- [Decision 1]: [What the user chose]
- [Decision 2]: [What the user chose]
- [Decision 3]: [What the user chose]

We're aligned. Ready to move forward.
```

Then call `ask_user_questions` with these options (in the user's language):

- **Brainstorm** — Explore ideas and design possibilities together
- **Detailed Plan** — Build a step-by-step execution plan based on what we locked in
- **Surprise Me — Start building the best version right now**

Do not proceed further until the user picks. Do not invoke another skill on your own.

## Anti-Patterns (Do NOT Do These)

1. **Asking questions you can answer from context.** If there's a `tsconfig.json`, don't ask "TypeScript or JavaScript?" Read the project first.
2. **Asking questions where you'll ignore the answer.** If you'd use React either way, don't ask about frameworks.
3. **Padding with safe questions.** "What should we name the project?" is not a fork. Skip it.
4. **Asking permission to ask.** Don't say "I have a few questions, is that okay?" Just ask.
5. **Showing your analysis.** The fork analysis is internal. The user sees questions and a summary, nothing else.
6. **Making up numbers.** Don't pretend to calculate entropy, EIG, or probabilities. You're an LLM, not a calculator. Just identify forks and ask about them.

## Examples

### Example 1: Ambiguous Request

User: "Build me a chat app"

**Internal fork analysis (NOT shown to user):**
- Interpretation A: Real-time chat (WebSocket, rooms, presence)
- Interpretation B: Simple messaging (REST API, polling)
- Interpretation C: AI chatbot interface (single user ↔ LLM)
- Forks: purpose (human↔human vs human↔AI), real-time vs not, single vs multi-user

**Question 1:** "Who's chatting with whom?"
- Option 1: People chatting with each other (like Slack/Discord)
- Option 2: User chatting with an AI (like ChatGPT)
- Option 3: Surprise me — a real-time group chat with rooms, WebSocket, and a clean React UI

### Example 2: Clear Request

User: "Add a dark mode toggle to the settings page"

**Internal fork analysis:** No forks. The project has an existing settings page, existing theme system. There's only one reasonable interpretation.

**Response:** "Nothing to clarify — I know what you need. Ready to implement."

### Example 3: Partially Clear

User: "Build an API for our inventory system"

**Internal fork analysis:**
- Framework is a fork (project has no existing backend)
- Auth is a fork (internal tool vs public API)
- Database is NOT a fork (project already uses PostgreSQL)

**Question 1:** Only asks about the actual forks. Skips database since it's already decided by the project context.
