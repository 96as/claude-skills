---
name: tadawul-daily-loop
description: Use this skill when the user wants to run a full paper-trading day on Tadawul — typical phrasings: "run today's loop", "do my morning check", "what's the trading plan for today?", "end of day review", "run the Tadawul loop for me". Orchestrates the existing skills in sequence: settle overnight cash, check TASI and sector rotation, review open positions against stops/targets, propose new trades from compliant candidates, run each proposed trade through the risk gate, and record approved trades to the paper portfolio with full journal entries. Produces a single daily report showing what was done and why. This is the skill that makes the Phase-1 experiment reproducible.
---

# Tadawul Daily Loop

## Overview

The orchestration skill for the Phase-1 paper trading experiment. Runs a complete daily cycle using the other skills. Produces one report per run so the user can audit what happened and why.

There are three call modes: **morning**, **mid-session**, and **end-of-day**. The user picks one, or runs all three on a trading day. Outside Sun-Thu 10:00-15:10 Riyadh time, the loop runs in "review only" mode (no new trades proposed).

## Required skills

This skill calls (and therefore depends on):
- `shariah-compliance-screener` — re-verify compliance
- `tadawul-stock-analyzer` — fundamental + news check on candidates
- `tadawul-technical-analyst` — chart read on candidates and open positions
- `tadawul-sector-rotation` — weekly leader identification
- `tadawul-position-sizer` — share count calculations
- `tadawul-risk-gate` — pre-trade checks
- `tadawul-paper-portfolio` — state management
- `tadawul-trade-journal` — decision logging

If any of these are missing, the loop degrades gracefully: it reports which checks it could not run and refuses to propose new trades until they're available.

## Modes

### Mode: `morning` (pre-session, before 10:00 Riyadh time)

**Purpose:** set up the day's plan.

Sequence:
1. **Settle pending cash** — call `tadawul-paper-portfolio.settle_pending_cash`. Any T+2 cash from sales two days ago becomes available.
2. **TASI snapshot** — fetch current TASI level and overnight/weekend move.
3. **Weekend gap check** (on Sundays only) — scan for news events between Thu close and Sun open that might affect open positions.
4. **Review open positions** — for each position:
   - Fetch current price
   - Check against journal's stop and targets
   - Flag any that would hit stop at the open, hit target 1, or have news since entry
5. **Sector rotation snapshot** — call `tadawul-sector-rotation` for a quick 1W/1M leader view.
6. **Candidate list** — propose 1-3 compliant names from leading sectors that have clean technical setups. Use the stock analyzer and technical analyst skills. Do NOT propose trades yet — just flag candidates.
7. **Output the morning brief.**

**Morning brief format:**

```
# 📝 PAPER — Morning Brief, [Date]

## TASI Context
- TASI: X (±Y% overnight, ±Z% 1W)
- Regime: [uptrend / sideways / downtrend]
- Weekend news affecting positions: [none / list]

## Open Positions — Status
| Ticker | Shares | Entry | Current | Unrealized | Stop | Target 1 | Status |
|---|---|---|---|---|---|---|---|
| 2222 | 10 | 28.50 | 30.10 | +SAR 16 | 27.30 | 30.00 | 🟢 Near target 1 |
| ... |

## Actions Recommended (for user decision)
- 🟢 2222: Consider trimming into target 1 at 30.00; journal note J0042
- 🔴 None hitting stops

## Sector Leaders (1W)
1. Energy +X%
2. Healthcare +Y%
3. Telecom +Z%

## Candidates for Today (no trades recorded yet)
1. **4013 Sulaiman Al Habib** — healthcare leader; daily flag on rising volume; proposed entry SAR X, stop SAR Y
2. **2222 Aramco** — already held; potential add on pullback to 28.80

## What I would NOT do today
- Any petchem entry — sector is weakening
- Any Nomu name — too illiquid for current capital

## Cash
- Available: SAR X
- Settling (T+2): SAR Y, available [date]
```

### Mode: `mid_session` (during 10:00-15:10)

**Purpose:** decide on actual entries and exits based on real price action.

Sequence:
1. Re-check TASI and sector direction (have things changed since morning?)
2. For each candidate from the morning brief:
   - Fetch current price
   - Check if the technical trigger has fired (e.g., breakout above flag line on volume)
   - If yes → compute position size via `tadawul-position-sizer`
   - Run the proposed trade through `tadawul-risk-gate`
   - If PASS → call `tadawul-paper-portfolio.record_buy` with a full journal entry
   - If REFUSE → note the reason, skip
3. For each open position:
   - Check stops and targets against current price
   - If stop hit → propose SELL, run risk gate, record if approved
   - If target 1 hit → propose partial SELL (50%) and trail stop on remainder
4. Output the mid-session log.

**Mid-session log format:**

```
# 📝 PAPER — Mid-Session Log, [Date] [Time]

## Decisions Made This Session

### BUY — 4013 Sulaiman Al Habib
- Shares: 5 at SAR 220.00
- Stop: SAR 212.00
- Target 1: SAR 236.00
- Risk gate: PASS
- Rationale: Healthcare leader; breakout above SAR 219 on 150% avg volume
- Journal: J0043

### SELL (partial) — 2222 Aramco
- Shares: 5 of 10 at SAR 30.10 (target 1)
- Trailing stop moved to SAR 28.80 on remaining 5
- Risk gate: PASS
- Rationale: Target 1 hit; lock in partial gain
- Journal: J0042-exit

## Candidates Skipped
- 2222 additional: price moved past proposed entry before trigger confirmed; stood down
- 1211 Maaden: risk gate REFUSE — sector concentration exceeds 30%

## Current Cash: SAR X
## Current Open Positions: N
```

### Mode: `end_of_day` (after 15:10 close)

**Purpose:** close the books on the day, update journal outcomes, compute daily stats.

Sequence:
1. Fetch end-of-session closing prices for all open positions.
2. Mark-to-market each position.
3. For positions closed today, update journal outcomes (did they hit stop/target as planned?).
4. Compute daily P/L (realized + change in unrealized).
5. Check against `max_daily_loss_pct` — if breached, flag for user review of rules.
6. Generate end-of-day summary.

**End-of-day summary format:**

```
# 📝 PAPER — End of Day, [Date]

## Daily P/L
- Realized today: SAR ±X (from Y closed trades)
- Unrealized change: SAR ±Y
- **Net day: SAR ±Z (±W% of account)**

## Closed Trades Today
| Journal | Ticker | Shares | Entry | Exit | P/L | Thesis held? |
|---|---|---|---|---|---|---|
| J0038 | 2222 (partial) | 5 | 28.50 | 30.10 | +SAR 8 | Yes |
| J0039 | 2010 | 3 | 103.00 | 101.20 | -SAR 5.4 | No — breakout failed |

## Open Positions — EOD Snapshot
| Ticker | Shares | Cost | EOD Price | Unrealized | Stop | Days Held |
|---|---|---|---|---|---|---|
| ... |

## Journal Outcomes Updated
- J0038: closed_winner, hit target 1
- J0039: closed_loser, thesis invalidated (volume didn't confirm)

## Week-to-date
- Trades: N (W / L / BE)
- Win rate: X%
- Net P/L: SAR ±X (±Y%)

## Notes for tomorrow
- Watch: [any open positions near stops/targets]
- Sector shifts: [any noticeable leadership change]
- Calendar: [earnings, dividends, Fed, OPEC on the horizon]
```

## Week-close and month-close

On Thursdays (last trading day of the Saudi week), run the `end_of_day` mode and *additionally* call `tadawul-portfolio-manager` for a full weekly review.

On the last trading day of the month, also run the purification ledger update and verify Shariah status for every held position (quarterly re-verification can be surfaced here).

## Degraded modes

If web access fails:
- Tell the user explicitly; do not fabricate prices.
- The loop switches to "review only" — reports current portfolio state from the file, but proposes no trades.

If `shariah-compliance-screener` returns unclear status for a candidate:
- Drop that candidate from the list. Do not propose it.

If `tadawul-risk-gate` REFUSEs a trade:
- Record the refusal in the log. Do not try to "reshape" the trade to pass — the refusal is itself data.

## Anti-patterns

- ❌ Running `mid_session` on a closed day — skip to `review only`
- ❌ Proposing trades when TASI data is stale — fetch fresh or abstain
- ❌ Skipping the morning brief and going straight to entries — the brief frames risk appetite for the day
- ❌ Recording a trade without a journal entry — they must always be paired
- ❌ Letting a "hot" candidate bypass the risk gate — the gate exists for when temptation is highest
- ❌ "Revenge trades" after a loss in the same session — if `max_daily_loss_pct` is approaching, the loop stops proposing
- ❌ Treating the loop as a recommendation engine for real-money action — it is a simulation. Real trades require the user's own decision in their broker

## References

- `references/loop-phases.md` — detailed explanation of each phase
- `references/degraded-mode-handling.md` — what to do when upstream skills/data fail
