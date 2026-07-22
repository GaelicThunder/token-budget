---
name: token-budget
description: "Hard token-saving mode for the whole session: minimal output, few or no subagents (Sonnet-only for research), and a user-defined cap on the 5-hour and weekly usage limits. Levels: low, medium, ultra (ultra = max savings). Use when the user invokes /token-budget, mentions token budget, saving tokens, risparmiare token, or wants to limit Claude usage. Args: [low|medium|ultra] [max-% of limits]."
argument-hint: "[low|medium|ultra] [max-%]"
---

# Token Budget Mode

We are on a token budget. Spend as few tokens as possible while still completing the task correctly.
Physics to respect: every tool call resends the whole conversation (few large calls beat many small ones), output tokens cost ~5× input (keep replies short), subagents multiply context (avoid them).

## Activate

1. Parse args: `level` ∈ `low|medium|ultra`, `cap` = max % of BOTH the 5-hour window and the weekly limit this session may consume.
2. Anything missing → ONE AskUserQuestion call asking for it. Level options: low / medium / ultra. Cap options: 10% / 25% / 50%.
3. If cap ≤ 10%, apply one level stricter than requested (low→medium, medium→ultra).
4. Confirm in ONE line: `Token budget ON — <level>, max <cap>% of 5h/weekly.` Nothing else.
5. Rules hold for the entire session, until the user says "stop token budget".

## All levels

- **Subagents: avoid.** If truly unavoidable, ONE at a time, `model: sonnet` for research/search (haiku for trivial lookups). Never opus/inherit for delegated work. No parallel fan-outs, no Workflow tool.
- Answer first. No preamble, no recap, no options you won't take, no "next steps" section. Headers/tables only if asked.
- Read fragments (`offset`/`limit`), not whole files. Never re-read what is already in context. Grep with narrow paths and `head_limit`.
- No speculative work: no drive-by refactors, no extra docs, nothing not asked.
- Batch shell commands (`&&`) when safe; minimize total tool-call count.
- Web/MCP: only when the task requires it, max 1–2 fetches.

## Levels

### low — light savings
- Concise but complete answers; explanations only when asked.
- Subagents: max 1, Sonnet, only if clearly cheaper than doing it inline.
- Doc files (README/TODO/TO_FIX): still updated, but terse, one batch at task end.

### medium — strong savings
- Telegraphic style: short sentences, bullets, no filler. 1-line task summaries.
- No subagents unless blocked without one (then 1× Sonnet, tight prompt, minimal output requested).
- Read ≤200 lines per call. Prefer asking the user for exact paths over broad searching.
- No web unless the user asks. Commit messages: subject line only.

### ultra — max savings
- NO subagents. NO web/MCP unless the user explicitly asks.
- Shortest correct answer: one line when possible; code = diff only, zero commentary.
- Read ≤100 lines per call; max 1 exploratory search per request — if not found, ask the user for the exact path.
- Skip README/TODO/TO_FIX maintenance this session (overrides the global docs rule). Auto-commit stays, subject-only message.
- Suppress everything optional: no alternatives, no background, no caveats unless safety-relevant.

## Enforcing the cap

- You cannot read usage directly. When in doubt, ask the user to check `/usage` (or `npx ccusage@latest blocks` for the live 5h block) and report the %.
- Before any single action likely to burn a noticeable slice of the cap (subagent, >500-line read, long generation), state the cost in one line first.
- If reported usage ≥ 80% of the cap: finish the current step only, report state in 2 lines, start nothing new.
- At 100% of the cap: stop. Say what remains undone.
