# Day-Trading Lab

A small, real-money ($50) experiment in automated, self-revising intraday trading, run by
a Claude Code Routine against a Robinhood cash account. There's no fixed strategy — the
agent invents its own trading rules, tests them, keeps what's working, retires what isn't,
and keeps periodically testing new ideas even when something is currently winning, because
markets change and a rule that works now may stop working later.

This is genuinely experimental. It is not a statistical machine-learning model with weights
being optimized — it's an agent reasoning from a written history of its own results each
run. Treat any returns as evidence about a tiny live experiment, not a track record.

## Files

- **memory.md** — the agent's compressed current beliefs: what's working, what's been
  discredited, the current market read, what's being tested next. Fully rewritten every
  Friday. This is the main file to read for the weekly check-in.
- **log.md** — the full day-by-day journal: what strategy ran, why, what happened, what
  went wrong, what's next. Older entries get archived into log-archive.md periodically.
- **state.json** — hard numbers: account safety state, the approved trading universe, the
  exact strategy currently in force, and the track record of every strategy ever tried.
- **routine-prompt.md** — the instructions pasted into the Claude Code Routine config.

## How it works

1. **Once per trading day**, the agent reads memory.md + recent log.md + state.json and
   decides: continue the current strategy, revise it, retire it for two losing days in a
   row, or deliberately test something new (forced at least every 5 trading days even if
   the current approach is winning).
2. **During the day** (hourly checks, market hours only), it watches a fixed, vetted
   universe of ~30 liquid large-cap stocks for its current entry rule, trades on it, and
   manages the exit.
3. **Every position is force-closed by 3:45pm ET** — nothing held overnight.
4. **At day's end**, that strategy's track record updates and a journal entry is committed.
5. **Every Friday**, memory.md gets rewritten from scratch based on the week's evidence,
   and that's what gets reviewed together, in chat, each week.

## Hard safety rules (never violated regardless of what strategy the agent invents)

- Never exceed actual settled cash. No margin, options, leverage, or short selling.
- Only symbols in `approvedUniverse` (state.json) — a fixed list of ~30 liquid, well-known
  large caps. The agent can't wander into illiquid or speculative names.
- One entry and one exit per symbol per day. No re-entering after an exit.
- Never reuse same-day sale proceeds before they settle (T+1).
- Only ever touches account `762268225` (the dedicated $50 Agentic account).
- **Circuit breaker**: account value below $25 (half of starting capital) stops all
  trading immediately until a human resets it.
- Closed markets (weekend/holiday) or outside 9:30am–4:00pm ET → no action.
- Every run commits something, even a "no trade today" entry. The log never goes silent.

## Status

🟢 Not yet started — set this routine up in claude.ai/code/routines pointing at this repo,
with the Robinhood connector attached, before it will do anything.
