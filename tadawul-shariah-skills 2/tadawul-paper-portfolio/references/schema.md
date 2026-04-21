# Paper Portfolio Schema

Full field definitions for `paper_portfolio.json`.

## Top-level fields

- `metadata` — immutable setup: currency, starting capital, broker cost assumptions. Set at init, rarely changed.
- `cash` — liquid cash state: available (usable now) and settling (T+2 pending).
- `positions` — array of currently-held positions. Empty array if none.
- `trade_history` — append-only record of every BUY, SELL, DIVIDEND, PURIFICATION event.
- `realized_pnl_sar` — running total of realized gains/losses, net of commissions and VAT.
- `purification_owed_sar` — running total of charity owed on dividends, not yet paid.

## Metadata fields

| Field | Type | Description |
|---|---|---|
| `currency` | string | Always "SAR" for this skill |
| `created_date` | date | YYYY-MM-DD when portfolio was initialized |
| `last_updated` | datetime | YYYY-MM-DD HH:MM of last state change |
| `starting_capital` | number | Initial cash balance; never changes |
| `broker_commission_rate` | number | e.g., 0.00155 for 0.155%. Multiplied by gross value. |
| `vat_rate` | number | 0.15 for 15% — applied to the commission, not the gross |
| `min_commission_sar` | number | e.g., 12.00 — floor per trade regardless of size |

## Position fields

| Field | Type | Description |
|---|---|---|
| `ticker` | string | 4-digit Tadawul ticker |
| `name` | string | Human-readable company name |
| `shares` | integer | Whole shares held; decreases to 0 on full exit |
| `avg_cost_sar` | number | Weighted-average cost including commissions from all buys into this lot |
| `first_buy_date` | date | Date of initial entry; not updated on averages |
| `shariah_status_at_entry` | string | Snapshot at entry: compliant_clean / compliant_purify / doubtful |
| `purification_rate_pct` | number | Per-dividend purification %, captured at entry |

## Trade history fields

| Field | Type | Description |
|---|---|---|
| `trade_id` | string | T0001, T0002, ... auto-incremented |
| `timestamp` | datetime | ISO 8601 with Riyadh timezone |
| `action` | string | BUY, SELL, DIVIDEND, PURIFICATION |
| `ticker` | string | |
| `shares` | integer | For BUY/SELL. For DIVIDEND: shares held at ex-date. |
| `price_sar` | number | Fill price per share |
| `gross_sar` | number | shares × price |
| `commission_sar` | number | Calculated per metadata rates |
| `vat_sar` | number | VAT on commission |
| `net_sar` | number | Cost out (BUY) or proceeds in (SELL) |
| `rationale_journal_id` | string | Link to the journal entry |
| `shariah_verified` | bool | Was status verified in the last 7 days before this trade? |
| `risk_gate_passed` | bool | Did the risk gate approve this? |

## Invariants

These must always hold after any operation:

- `available_sar + settling_sar + sum(positions.value_at_last_known_price) + realized_pnl_sar ≈ starting_capital + (sum of dividends) - (sum of purifications paid)` — the books must balance
- `available_sar >= 0` — never negative
- `all positions.shares > 0` — zero-share positions should be removed
- `trade_history` is sorted by timestamp ascending and never modified in place (append-only; updates go to separate purification ledger or journal)
