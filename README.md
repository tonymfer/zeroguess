# ZeroGuess

**End the "just do it" guessing forever.**

ZeroGuess finds every real implementation fork silently, forces explicit answers with concrete options + Surprise Me, and stops after max 5 questions.
Pure clarification. Zero guesses. Done in 2-5 minutes.

## Why ZeroGuess

Every AI coding agent has the same failure mode: you say "build me X" and it guesses wrong on 3 decisions you never specified. Then you waste 20 minutes undoing bad assumptions.

ZeroGuess fixes this by:

- **Silent Fork Analysis** — internally brainstorms 3-5 interpretations, finds where they diverge, only asks about real forks
- **Concrete Options** — never "monolithic vs microservices", always "single Express server with all routes, or separate Lambda per endpoint"
- **Surprise Me** — every question includes a best-guess option so you can skip if you trust the agent
- **Hard 5-question limit** — zero fatigue, self-terminates and summarizes
- **Auto language detection** — replies in whatever language you're using
- **Anti-pattern guards** — won't ask about things it can read from your project context (tsconfig.json exists? it won't ask "TypeScript or JavaScript?")

## Install

```bash
# For GSD / pi users
mkdir -p ~/.agents/skills/zeroguess

# For Claude Code users
mkdir -p ~/.claude/skills/zeroguess
```

Copy [`SKILL.md`](./SKILL.md) into the directory. Restart your agent. Done.

## Usage

Type any ambiguous request, then invoke:

```
/zeroguess
```

Or with GSD:

```
/gsd-zeroguess
```

### Example

**You:** "Build me a chat app"

**ZeroGuess internally:** detects 3 forks (who's chatting, real-time vs polling, single vs multi-user)

**ZeroGuess asks:** "Who's chatting with whom?"
- People with each other (like Slack)
- User with an AI (like ChatGPT)
- Surprise me — real-time group chat with rooms and WebSocket

**After 2-4 questions:** summary + handoff to brainstorm, plan, or build.

### Clear requests get no questions

**You:** "Add dark mode toggle to the settings page"

**ZeroGuess:** "Nothing to clarify. Ready to implement."

## How It Works

```
Request → Silent Fork Analysis → Ask One Fork at a Time → Summary → Handoff
              (internal)           (max 5 questions)       (user picks next step)
```

1. **Fork Analysis** — reads your request + project context, identifies where different interpretations lead to completely different code
2. **Kill Non-Forks** — cosmetic choices, trivial defaults, things already decided by your project = skipped
3. **Ask** — one question per message, 2-3 concrete options, always includes Surprise Me
4. **Stop** — when no forks remain, user says "go", or 5 questions reached
5. **Handoff** — summary of decisions + choice of brainstorm / plan / build

## What Makes It Different

| Feature | Other clarifiers | ZeroGuess |
|---------|-----------------|-----------|
| Shows internal analysis | Often | Never |
| Question limit | None (asks forever) | Hard cap at 5 |
| Option style | Abstract jargon | Concrete builds |
| "You decide" escape | Ignored | Respected immediately |
| Project context awareness | Rare | Reads files first, skips known answers |
| Surprise Me option | No | Every question |
| Language | English only | Auto-detects user's language |

## License

MIT

## Credits

Built by [@toismfer](https://github.com/toismfer)

⭐ Star if it saves you from bad guesses
