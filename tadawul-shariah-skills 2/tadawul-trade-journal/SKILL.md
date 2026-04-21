---
name: tadawul-trade-journal
description: Use this skill whenever Claude is about to record a paper trade or wants to retrospectively review past trading decisions. Triggers automatically as part of every record_buy and record_sell call from tadawul-paper-portfolio, and on user requests like "review my last 10 trades", "what was my reasoning on the Aramco buy?", "what's my win rate?", "show my journal". Maintains an append-only JSON log (paper_trade_journal.jsonl) where every entry captures the decision rationale, market context at the time, expected outcome, actual outcome (when known), and post-mortem notes. Critical for evaluating whether Claude's analytical judgment is actually any good — without this, you cannot distinguish skill from luck.
---

# Tadawul Trade Journal

## Overview

Append-only log of every trading decision. Captures *why* a trade was made at the moment of decision, separate from the trade record itself. Without this, after-the-fact analysis is just storytelling — you can't tell if Claude's reasoning was sound but the market was unfriendly, or if the reasoning was wrong from the start.

## File location

The journal lives at: `paper_trade_journal.jsonl` (one JSON object per line, append-only).

**Schema per entry:**

```json
{
  "journal_id": "J0001",
  "timestamp": "2026-04-21T10:25:00+03:00",
  "linked_trade_id": "T0001",
  "decision_type": "BUY | SELL | HOLD | SKIP | EXIT_PLAN",
  "ticker": "2222",
  "name": "Saudi Aramco",
  
  "context": {
    "tasi_level": 11270.62,
    "tasi_1d_pct": 0.02,
    "tasi_1w_pct": 2.94,
    "ticker_price_at_decision": 28.50,
    "ticker_1d_pct": 0.5,
    "ticker_volume_vs_avg_pct": 110,
    "sector_tier": "leading",
    "market_session": "open | pre | post | closed_weekend"
  },
  
  "shariah": {
    "status": "compliant_clean | compliant_purify | doubtful | non_compliant",
    "source": "argaam | almaqasid | calculated",
    "verified_date": "2026-04-21",
    "purification_rate_pct": 0.0
  },
  
  "thesis": {
    "summary_one_line": "Energy leadership + ascending triangle breakout on rising oil",
    "drivers": [
      "Brent +3% on OPEC+ news",
      "Daily ascending triangle since March, neckline at 28.40",
      "Sector tier: leading"
    ],
    "timeframe": "swing | day | position",
    "expected_holding_period_days": 5
  },
  
  "trade_plan": {
    "entry_sar": 28.50,
    "stop_sar": 27.30,
    "target_1_sar": 30.00,
    "target_2_sar": 31.50,
    "shares": 10,
    "position_value_sar": 285.00,
    "risk_sar": 12.00,
    "risk_pct_of_account": 1.2,
    "rr_to_target_1": 1.25,
    "rr_to_target_2": 2.5
  },
  
  "invalidation_conditions": [
    "Daily close < 27.30",
    "Brent oil drops > 5% in a session",
    "Aramco loses key contract or major guidance miss"
  ],
  
  "what_could_go_wrong": [
    "Weekend gap risk on oil headlines",
    "Daily limit ±10% caps upside if move accelerates",
    "Position sizing already at sector cap"
  ],
  
  "outcome": {
    "status": "open | closed_winner | closed_loser | closed_breakeven | invalidated",
    "exit_date": null,
    "exit_price_sar": null,
    "realized_pnl_sar": null,
    "realized_pnl_pct": null,
    "actual_holding_days": null,
    "hit_stop": null,
    "hit_target_1": null,
    "hit_target_2": null
  },
  
  "post_mortem": {
    "thesis_held_up": null,
    "what_was_right": null,
    "what_was_wrong": null,
    "lessons": null
  }
}
```

## Operations

### `append_decision` — log a new decision

Called automatically by `tadawul-paper-portfolio` for every buy/sell. Can also be called for HOLD or SKIP decisions (e.g., "I considered buying X but didn't because Y").

Action: append a new line to the JSONL with auto-incremented journal_id. Set outcome.status = "open" for buys; for sells, outcome fields are filled at append time.

### `update_outcome` — close out an entry

Called when a position is closed. Inputs: journal_id, exit_price, exit_date.

Action: read all lines, find the matching journal_id, update outcome fields, rewrite the file (yes, this technically violates append-only — that's a deliberate trade-off for a learning system; alternative is to append a new "outcome" entry and link it).

### `view_journal` — render journal entries

Inputs: filter (open / closed / all / by date range / by ticker). Default: last 20 entries.

Output:
```
# Trade Journal — [filter]

## J0042 — BUY 2222 (Aramco) — 2026-04-21
**Status:** OPEN | Entered at 28.50 | Stop 27.30 | Target 30.00
**Thesis:** Energy leadership + triangle breakout
**Risk:** SAR 12 (1.2% of account)
**Shariah:** ✅ Compliant (Argaam, 2026-04-21)

## J0041 — SELL 2010 (SABIC) — 2026-04-19
**Status:** CLOSED — LOSER | Exit 102.50 | P/L: -SAR 45 (-2.1%)
**Thesis at entry:** Petchem rotation, daily breakout
**What went wrong:** Breakout failed; closed back below trigger same session
**Lesson:** Required volume confirmation; volume was actually below avg

...
```

### `compute_stats` — aggregate journal performance

Inputs: optional date range.

Output:
```
# Journal Performance Stats

**Period:** YYYY-MM-DD to YYYY-MM-DD
**Closed trades:** N (W winners, L losers, B breakeven)
**Win rate:** X%
**Avg win:** SAR X (+X%)
**Avg loss:** SAR X (-X%)
**Win/Loss ratio:** X.X
**Profit factor:** X.X (gross wins / gross losses)
**Largest winner:** [ticker] +SAR X
**Largest loser:** [ticker] -SAR X
**Avg holding days (winners vs losers):** X vs Y

## Thesis Quality
**Theses that held up:** X / N closed
**Common "what was wrong" themes:** [pattern]
**Common "what was right" themes:** [pattern]
```

The thesis-quality section is the **whole point** of this skill. Win rate alone tells you whether the trades made money; thesis quality tells you whether your reasoning is actually predictive.

### `weekly_review` — generate the end-of-week journal review

Inputs: none (uses last 7 days).

Output: a structured review covering all decisions in the week, hit rate, what worked, what didn't, calibration check (did "high probability" trades actually win more often?).

## Anti-patterns

- ❌ Filling the journal AFTER the fact "with the benefit of hindsight" — defeats the purpose. Must be at decision time.
- ❌ Writing vague theses ("looks good") — be specific about drivers
- ❌ Skipping the "what could go wrong" section — pre-mortem is a key calibration tool
- ❌ Updating outcome with rationalizations ("market was wrong") rather than honest assessment
- ❌ Using the journal to back-justify trades — it's for learning, not vindication
- ❌ Skipping the journal because "this trade is small" — every decision teaches you something

## References

- `references/journal-schema.md` — full schema with field definitions and examples
- `references/calibration-guide.md` — how to use journal stats to improve calibration
