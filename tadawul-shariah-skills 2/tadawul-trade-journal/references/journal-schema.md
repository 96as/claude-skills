# Trade Journal Schema & Examples

## File format

`paper_trade_journal.jsonl` — JSON Lines format. One complete JSON object per line. Easier to append to than a single JSON array and more resilient to partial writes.

## Why separate from the portfolio?

- Portfolio answers "what do I own?" — it's state.
- Journal answers "why did I make this decision?" — it's history-of-thought.

Conflating them would cause the reasoning to get lost when positions close.

## Complete example entry (BUY)

```json
{
  "journal_id": "J0001",
  "timestamp": "2026-04-21T10:30:15+03:00",
  "linked_trade_id": "T0001",
  "decision_type": "BUY",
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
    "market_session": "open"
  },
  "shariah": {
    "status": "compliant_purify",
    "source": "argaam",
    "verified_date": "2026-04-21",
    "purification_rate_pct": 1.5
  },
  "thesis": {
    "summary_one_line": "Energy leadership + ascending triangle breakout on rising oil",
    "drivers": [
      "Brent +3% on OPEC+ extension news",
      "Daily ascending triangle since March, neckline at 28.40",
      "Energy sector tier: leading on 1W and 1M",
      "Aramco Q1 dividend ex-date next month; often supports price"
    ],
    "timeframe": "swing",
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
    "Brent drops > 5% in a session",
    "Major adverse Aramco-specific news (guidance cut, contract loss)"
  ],
  "what_could_go_wrong": [
    "Weekend oil gap against us",
    "Breakout is retail-driven and fails on volume fade",
    "OPEC+ reverses stance within the week"
  ],
  "outcome": {
    "status": "open"
  },
  "post_mortem": {}
}
```

## Example entry — closed winner

Same fields through `what_could_go_wrong`, then:

```json
  "outcome": {
    "status": "closed_winner",
    "exit_date": "2026-04-24",
    "exit_price_sar": 30.00,
    "realized_pnl_sar": 13.40,
    "realized_pnl_pct": 4.7,
    "actual_holding_days": 3,
    "hit_stop": false,
    "hit_target_1": true,
    "hit_target_2": false
  },
  "post_mortem": {
    "thesis_held_up": true,
    "what_was_right": "Energy leadership held; oil up another 1.5% over holding period; volume confirmed the breakout",
    "what_was_wrong": "Target 1 was hit but I should have trailed the stop tighter to catch more of the move — stock continued to 30.80 before pulling back",
    "lessons": "Use 2x ATR trailing stop after target 1 hits on trending names"
  }
```

## Example entry — SKIP (no trade taken)

Valuable for evaluating whether your filter works. Track what you almost bought and didn't:

```json
{
  "journal_id": "J0015",
  "timestamp": "2026-04-22T11:15:00+03:00",
  "linked_trade_id": null,
  "decision_type": "SKIP",
  "ticker": "4013",
  "name": "Sulaiman Al Habib",
  "context": { ... },
  "shariah": { "status": "compliant_clean", ... },
  "thesis": {
    "summary_one_line": "Healthcare leader; considered on breakout from flag",
    "drivers": [...]
  },
  "skip_reason": "Risk gate REFUSED — sector concentration would have hit 32% (cap 30%)",
  "outcome": {
    "status": "skipped",
    "what_happened_next": "Stock ran +4.2% over next 3 days — would have been a winner"
  },
  "post_mortem": {
    "lessons": "Consider raising sector cap to 35% for healthcare given Vision 2030 overweight, or trim existing healthcare position to make room"
  }
}
```

## Calibration over time

After 30+ entries, the journal stats start telling you:

- **Is "high probability" actually high probability?** — Bucket your entries by the probability assigned in the scenarios and check actual hit rate. If "70% probability" hits 40% of the time, your calibration is off.
- **Which drivers actually predict outcomes?** — Group winners/losers by driver keywords. "Sector leadership" might be a reliable winner; "breakout on average volume" might be a reliable loser.
- **When does your thesis most often fail?** — Read the "what_was_wrong" notes in aggregate. Patterns emerge.

This is where the real value of the experiment lives. The trades themselves barely move the needle on 1,000 SAR — but the learning loop can be significant.
