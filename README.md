# token-budget

Claude Code skill: hard token-saving mode for a whole session.

Tells Claude: *we're on a token budget* — minimal output, no speculative work, few or no
subagents (Sonnet-only for research), and a user-defined cap on how much of the 5-hour /
weekly usage limits the session may burn.

## Install

Clone straight into your Claude Code skills folder, then restart Claude Code.

**macOS / Linux**

```bash
git clone https://github.com/GaelicThunder/token-budget ~/.claude/skills/token-budget
```

**Windows (PowerShell)**

```powershell
git clone https://github.com/GaelicThunder/token-budget "$env:USERPROFILE\.claude\skills\token-budget"
```

Prefer keeping the repo elsewhere? Clone it wherever you like and link it:

```bash
# macOS / Linux
ln -s /path/to/token-budget ~/.claude/skills/token-budget
```

```powershell
# Windows (junction, no admin needed)
New-Item -ItemType Junction -Path "$env:USERPROFILE\.claude\skills\token-budget" -Target "C:\path\to\token-budget"
```

Per-project instead of global: clone into `<project>/.claude/skills/token-budget`.

## Usage

```
/token-budget <level> [5h=<n>] [weekly=<n>]

/token-budget ultra                   # no cap → default: wind down at 5% of the 5-hour window, stop at 10%
/token-budget ultra 5h=20             # ≤20% of the 5-hour window; weekly untracked
/token-budget medium 5h=20 weekly=60  # both windows capped explicitly
/token-budget                         # asks the level only — caps are never interrogated
```

Caps are **optional** — one window is enough, both never required. When given, they are
**always labeled by window** (`5h=` / `weekly=`): a bare number like `20` is ambiguous
and gets rejected — Claude will ask which limit you meant.

| Level  | Savings | Behavior |
|--------|---------|----------|
| low    | light   | concise output, web delegate + ≤1 extra Sonnet subagent, no speculative work |
| medium | strong  | telegraphic output, only the web delegate unless blocked |
| ultra  | max     | shortest possible output, web delegate is the only subagent, fragment-only reads |

Web search is **never blocked at any level** — most tasks need it. But it is always
**delegated**: a cheap Sonnet subagent does the searching/reading and returns a short
digest, so raw web pages never enter the expensive main context.

Each cap is the share of that Claude usage window (5-hour or weekly) the session is
allowed to consume. Claude warns before expensive actions and asks you to check
`/usage` when in doubt. Per window:

- **explicit cap** → winds down at 80% of the cap, hard stop at 100%;
- **no cap (default)** → winds down at 5% of the **5-hour window** used, hard stop at
  10% of it; the weekly limit is only tracked if you cap it explicitly.

If an explicit cap is ≤10%, the skill automatically applies one level stricter than
requested.

## Why it works

- Every tool call resends the whole conversation → fewer, batched calls.
- Output tokens cost ~5× input → short answers.
- Code subagents duplicate context → avoided. Web research is the exception: a cheap
  Sonnet delegate reads the pages and returns a digest — cheap-model tokens weigh far
  less on the 5h/weekly limits, and the main context stays small (it is resent on
  every call).

## License

MIT
