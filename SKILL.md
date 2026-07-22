---
name: token-budget
description: "Hard token-saving mode for the whole session: minimal output, few or no subagents (Sonnet-only for research), and optional per-window caps on the 5-hour / weekly usage limits (no cap given → default: wind down at 5% of the window, stop at 10%). Levels: low, medium, ultra (ultra = max savings). Use when the user invokes /token-budget, mentions token budget, saving tokens, risparmiare token, or wants to limit Claude usage. Args: [low|medium|ultra] [5h=<n>] [weekly=<n>] — caps labeled by window, one is enough."
argument-hint: "[low|medium|ultra] [5h=<n>] [weekly=<n>]"
---

# Token Budget Mode

We are on a token budget. Spend as few tokens as possible while still completing the task correctly.
Physics to respect: every tool call resends the whole conversation (few large calls beat many small ones), output tokens cost ~5× input (keep replies short), subagents multiply context (avoid them).

## Activate

1. Parse args: `level` ∈ `low|medium|ultra`; optional caps **labeled by window**: `5h=<n>` and/or `weekly=<n>` (also accept `20%5h`, `w=10`). **One window is enough — never require both.** A bare number (e.g. `20`) is ambiguous: ask which window it refers to. Never assume.
2. No caps given → NO questions: the **default** applies to both windows — wind down at 5% of the window used, hard stop at 10%. An explicit cap overrides the default for its window only.
3. Level missing → ONE AskUserQuestion call (low / medium / ultra). Never ask for caps.
4. If any explicit cap ≤ 10%, apply one level stricter than requested (low→medium, medium→ultra).
5. Confirm in ONE line naming windows explicitly, e.g.: `Token budget ON — ultra, 5h: max 20%, weekly: default (stop 10%).` Never state a % without naming its window — here or anywhere later in the session.
6. Rules hold for the entire session, until the user says "stop token budget".

## All levels

- **Subagents: avoid.** If truly unavoidable, ONE at a time, `model: sonnet` for research/search (haiku for trivial lookups). Never opus/inherit for delegated work. No parallel fan-outs, no Workflow tool.
- Answer first. No preamble, no recap, no options you won't take, no "next steps" section. Headers/tables only if asked.
- Read fragments (`offset`/`limit`), not whole files. Never re-read what is already in context. Grep with narrow paths and `head_limit`.
- No speculative work: no drive-by refactors, no extra docs, nothing not asked.
- Batch shell commands (`&&`) when safe; minimize total tool-call count.
- **Web search: NEVER blocked, at any level** — most tasks need it. Keep it lean: targeted queries, no re-fetching the same page, extract what you need instead of quoting walls of text.

## Levels

### low — light savings
- Concise but complete answers; explanations only when asked.
- Subagents: max 1, Sonnet, only if clearly cheaper than doing it inline.
- Doc files (README/TODO/TO_FIX): still updated, but terse, one batch at task end.

### medium — strong savings
- Telegraphic style: short sentences, bullets, no filler. 1-line task summaries.
- No subagents unless blocked without one (then 1× Sonnet, tight prompt, minimal output requested).
- Read ≤200 lines per call. Prefer asking the user for exact paths over broad searching.
- Commit messages: subject line only.

### ultra — max savings
- NO subagents. Web still allowed: one targeted search/fetch at a time, no broad fan-out.
- Shortest correct answer: one line when possible; code = diff only, zero commentary.
- Read ≤100 lines per call; max 1 exploratory search per request — if not found, ask the user for the exact path.
- Skip README/TODO/TO_FIX maintenance this session (overrides the global docs rule). Auto-commit stays, subject-only message.
- Suppress everything optional: no alternatives, no background, no caveats unless safety-relevant.

## Enforcing the cap

- You cannot read usage directly. When in doubt, ask the user to check `/usage` (or `npx ccusage@latest blocks` for the live 5h block) and report the %.
- Before any single action likely to burn a noticeable slice of the budget (subagent, >500-line read, long generation), state the cost in one line first.
- Thresholds, per window:
  - **Explicit cap** (`5h=N` / `weekly=N`): wind down at 80% of that cap, hard stop at 100%.
  - **Default** (window with no explicit cap): wind down at 5% of the window used, hard stop at 10%.
- Wind down = finish the current step only, report state in 2 lines, start nothing new.
- Hard stop = stop. Say what remains undone.
