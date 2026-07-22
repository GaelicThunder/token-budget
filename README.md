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
/token-budget <level> [5h=<n>] [weekly=<n>]

/token-budget ultra                   # no cap → default: wind down at 5% of each window, stop at 10%
/token-budget ultra 5h=20             # ≤20% of the 5-hour window; weekly stays on the default
/token-budget medium 5h=20 weekly=60  # both windows capped explicitly
/token-budget                         # asks the level only — caps are never interrogated
```

Caps are **optional** — one window is enough, both never required. When given, they are
**always labeled by window** (`5h=` / `weekly=`): a bare number like `20` is ambiguous
and gets rejected — Claude will ask which limit you meant.

| Level  | Savings | Behavior |
|--------|---------|----------|
| low    | light   | concise output, ≤1 Sonnet subagent, no speculative work |
| medium | strong  | telegraphic output, subagents only when blocked |
| ultra  | max     | shortest possible output, no subagents, fragment-only reads |

Web search is **never blocked at any level** — most tasks need it. It's just kept lean
(targeted queries, no re-fetching).

Each cap is the share of that Claude usage window (5-hour or weekly) the session is
allowed to consume. Claude warns before expensive actions and asks you to check
`/usage` when in doubt. Per window:

- **explicit cap** → winds down at 80% of the cap, hard stop at 100%;
- **no cap (default)** → winds down at 5% of the window used, hard stop at 10%.

If an explicit cap is ≤10%, the skill automatically applies one level stricter than
requested.

## Why it works

- Every tool call resends the whole conversation → fewer, batched calls.
- Output tokens cost ~5× input → short answers.
- Subagents duplicate context → avoided; Sonnet when unavoidable.

## License

MIT
