# token-budget

Claude Code skill: hard token-saving mode for a whole session.

Tells Claude: *we're on a token budget* — minimal output, no speculative work, few or no
subagents (Sonnet-only for research), and a user-defined cap on how much of the 5-hour /
weekly usage limits the session may burn.

## Install

```bash
git clone https://github.com/GaelicThunder/token-budget ~/Applications/token-budget
ln -s ~/Applications/token-budget ~/.claude/skills/token-budget
```

## Usage

```
/token-budget <level> <max-%>

/token-budget ultra 20    # max savings, burn at most 20% of the 5h/weekly limits
/token-budget medium 50
/token-budget             # asks level + cap interactively
```

| Level  | Savings | Behavior |
|--------|---------|----------|
| low    | light   | concise output, ≤1 Sonnet subagent, no speculative work |
| medium | strong  | telegraphic output, subagents only when blocked, no web |
| ultra  | max     | shortest possible output, no subagents, no web, fragment-only reads |

The cap (`max-%`) is the share of your Claude 5-hour **and** weekly limits the session is
allowed to consume. Claude warns before expensive actions, asks you to check `/usage`
when in doubt, winds down at 80% of the cap and stops at 100%.

If the cap is ≤10%, the skill automatically applies one level stricter than requested.

## Why it works

- Every tool call resends the whole conversation → fewer, batched calls.
- Output tokens cost ~5× input → short answers.
- Subagents duplicate context → avoided; Sonnet when unavoidable.

## License

MIT
