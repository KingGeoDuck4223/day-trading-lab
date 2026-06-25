# Setting up the automated cron workflow

This makes the day-trading lab run unattended, on GitHub's own infrastructure, via
`.github/workflows/day-trading-lab.yml`. Read this whole file before turning it on —
there are two real uncertainties below that are worth resolving with a manual test
run before you trust it with the schedule.

## What you need to set up

1. **Files in place**: this `day-trading-lab/` folder (with `routine-prompt.md`,
   `state.json`, `log.md`, `memory.md`, `mcp-config.json`) and `.github/workflows/`
   both need to be at the root of your repo (or adjust the paths in the workflow
   YAML to match wherever you put them).
2. **A GitHub secret**: `ANTHROPIC_API_KEY` — repo Settings → Secrets and variables
   → Actions → New repository secret. Get a key from console.anthropic.com.
3. **Robinhood connector auth**: this is the part I'm genuinely unsure will work
   without changes — see the caveat below before relying on it.
4. **Branch protection**: if your repo's main branch requires pull requests, the
   workflow's direct push will fail. Either relax that for this branch, or change
   the workflow to push to a dedicated branch (e.g. `trading-log`) instead.

## Two things to verify with a manual run before trusting the schedule

**Test it yourself first.** Go to the Actions tab → "Day Trading Lab" →
"Run workflow" (the `workflow_dispatch` trigger exists specifically for this).
Watch it run during market hours and check the actual commit it produces before
letting the cron schedule run unsupervised.

1. **Robinhood authentication in a headless run.** Connecting Robinhood through
   the chat interface involves an interactive OAuth step. I don't have confirmed
   details on how (or whether) that authentication carries over to a non-interactive
   GitHub Actions run with no browser. If the first test run fails to reach the
   Robinhood tools, this is almost certainly why — you may need a stored token
   passed in as another secret and added to `mcp-config.json`, and the exact
   mechanism isn't something I could verify from here.
2. **Permission mode.** The workflow uses `--permission-mode acceptEdits` rather
   than full `--dangerously-skip-permissions`, to keep some guardrails active in
   an unattended run. I'm not fully certain that's sufficient to avoid the run
   hanging on an unanswered prompt for a Bash or MCP tool call — if a test run
   times out or stalls, that's the likely cause. The `--allowedTools` list is
   scoped tightly on purpose (only git, file read/write, and the Robinhood MCP
   tools) specifically so that even a broader permission mode has a small blast
   radius if it ever needs loosening.

## The cost angle — read this before scheduling it for real

This is separate from the $50 trading capital and easy to overlook: **running
Claude Code via API key is not free**, and as of June 15, 2026 it draws from its
own metered usage rather than a flat subscription. Three runs a day, five days a
week, each reading several files and making a handful of tool calls, will accrue
real token costs — quite possibly more per month than the $50 this whole
experiment is trading with. Check current pricing at console.anthropic.com before
turning the schedule on, and treat the three-runs-a-day default in the workflow as
a cost-conscious starting point, not a floor — drop to one run a day if you want
to keep this cheap, or add more only once you've confirmed the cost per run.

## Daylight saving time

The cron times in the workflow assume EDT (UTC-4), which is correct right now.
When the US falls back to EST (UTC-5) in early November, every hour value in the
workflow needs to shift forward by one to keep hitting the same actual times in
US Eastern — that's a real, recurring maintenance task, not a one-time setup step.
