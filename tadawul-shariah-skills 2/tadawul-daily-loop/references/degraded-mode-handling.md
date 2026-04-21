# Degraded Mode Handling

What to do when something upstream breaks. The loop should degrade gracefully, not pretend everything is fine.

## If web search or web_fetch fails

**Scenario:** Can't reach Argaam / TradingView / Saudi Exchange.

**Behavior:**
- Tell the user explicitly which source(s) failed.
- For open positions: report last known price with a "STALE" tag and the date of the last known data.
- For new trades: **refuse to propose any.** Entry decisions without current data are guesses.
- For journal outcome updates: skip until data is available.

**Never:** fabricate prices, estimate from "recent trend," or fall back to training-data memory. The loss of live data is itself information — the user should know.

## If Shariah status can't be verified

**Scenario:** Argaam Shariah list URL fails; no clear status returned.

**Behavior:**
- For existing positions: keep them but flag Shariah status as "STALE, needs verification."
- For candidates: drop from the list entirely. Never propose a BUY without a fresh Shariah confirmation.
- Report to the user: "I couldn't confirm Shariah status for X this session. Not proposing trades in that name until verified."

## If the risk gate skill is unavailable

**Scenario:** Skill file missing or errors out.

**Behavior:**
- **Refuse all new BUY trades.** The risk gate is the safety layer; there's no correct way to proceed without it.
- SELLs are still allowed (closing risk is always acceptable).
- Tell the user: "Risk gate unavailable. Only exits are being processed."

## If the portfolio file is corrupted or missing

**Scenario:** `paper_portfolio.json` doesn't parse as valid JSON, or the schema is broken.

**Behavior:**
- Do not attempt to "fix" the file silently. The file is state of record; unilateral repairs lose history.
- Tell the user what's wrong: "The portfolio file has an invalid structure at [location]. I won't modify it. Options: (a) restore from a backup, (b) show me the raw file for manual correction, (c) start a fresh portfolio (which would discard history)."
- Suggest the user keep regular backups of the file.

## If the market is closed

**Scenario:** Running the loop on Friday, Saturday, a holiday, or outside 10:00-15:10 Riyadh time.

**Behavior:**
- Mode auto-switches to "review only":
  - No new trade proposals
  - No risk-gate calls (there's nothing to gate)
  - Mark-to-market using last close
  - Journal entries for "reflection" decisions (e.g., "considered X, decided to pass because...") are still allowed and valuable
- Tell the user: "Market is closed. Running review-only."

## If a proposed trade's stock has been suspended or halted

**Scenario:** Live data shows suspension status.

**Behavior:**
- Refuse the trade.
- For existing positions in that name: can't trade, can't mark-to-market accurately. Flag explicitly.
- Do not enter new positions in suspended names even if they resume during the session.

## If TASI data is fresh but individual stock data is stale

**Scenario:** TASI level available but a specific ticker hasn't traded recently enough.

**Behavior:**
- Proceed with everything that doesn't depend on that ticker.
- For that ticker specifically, flag and refuse to record entries.
- Check whether the stock is in a low-liquidity state that suggests dropping it from the watch list entirely.

## General principle

**"Abstain when uncertain" beats "act on guess"** for this experiment. The cost of missing a trade is one missed trade. The cost of acting on bad data is recorded state that misleads every future decision.

The user should see a clear signal whenever the loop is running in degraded mode. Never hide degradation behind a normal-looking report.
