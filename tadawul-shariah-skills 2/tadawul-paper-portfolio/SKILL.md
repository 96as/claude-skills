---
name: tadawul-paper-portfolio
description: Use this skill whenever the user wants to track a simulated (paper) Tadawul portfolio — recording simulated buys and sells, viewing current cash and positions, computing realized and unrealized P/L. Triggers on "buy X shares of Y at price Z (paper)", "sell my Aramco position (paper)", "show my paper portfolio", "what's my cash balance", "what's my P/L on the simulation". Manages a strict-schema JSON file (paper_portfolio.json) with cash balance, open positions (with avg cost), trade history, and simulated commissions. Designed for the Phase-1 experimental loop where Claude proposes trades and records them without touching a real broker. NEVER for real-money execution — that is a separate concern.
---

# Tadawul Paper Portfolio

## Overview

Manages a simulated portfolio backed by a single JSON file. Every "buy" or "sell" is an append to that file's trade history plus a recompute of cash and positions. No broker integration; no real money moves. This is the core data substrate for paper trading.

## CRITICAL: This is paper trading only

Every output of this skill should make the simulation framing visible:
- Reports start with the header "📝 PAPER PORTFOLIO" 
- Trades are described as "simulated" or "recorded"
- Never use language like "I placed an order" or "your trade is filled" without the word "simulated" or "paper" attached

If the user asks to use this skill against a real account, refuse and tell them this skill cannot place real orders — they need to execute in their broker app themselves.

## File location

The portfolio lives at: `paper_portfolio.json` in the conversation working directory (or wherever the user uploads it).

**Schema** (strict — do not invent additional fields):

```json
{
  "metadata": {
    "currency": "SAR",
    "created_date": "YYYY-MM-DD",
    "last_updated": "YYYY-MM-DD HH:MM",
    "starting_capital": 1000.00,
    "broker_commission_rate": 0.00155,
    "vat_rate": 0.15,
    "min_commission_sar": 12.00
  },
  "cash": {
    "available_sar": 1000.00,
    "settling_sar": 0.00,
    "settling_until": null
  },
  "positions": [
    {
      "ticker": "2222",
      "name": "Saudi Aramco",
      "shares": 10,
      "avg_cost_sar": 28.50,
      "first_buy_date": "2026-04-21",
      "shariah_status_at_entry": "compliant_clean",
      "purification_rate_pct": 0.0
    }
  ],
  "trade_history": [
    {
      "trade_id": "T0001",
      "timestamp": "2026-04-21T10:30:00+03:00",
      "action": "BUY",
      "ticker": "2222",
      "shares": 10,
      "price_sar": 28.50,
      "gross_sar": 285.00,
      "commission_sar": 12.00,
      "vat_sar": 1.80,
      "net_sar": 298.80,
      "rationale_journal_id": "J0001",
      "shariah_verified": true,
      "risk_gate_passed": true
    }
  ],
  "realized_pnl_sar": 0.00,
  "purification_owed_sar": 0.00
}
```

## Required workflow — read before write

Every modification follows this sequence:

1. **Read** the current `paper_portfolio.json` (use the `view` tool)
2. **Validate** the proposed change against the schema and current state
3. **Compute** the new state in memory
4. **Re-verify** before write (cash sufficient? position exists? schema intact?)
5. **Write** the entire updated file (use `create_file` to overwrite)
6. **Confirm** to the user with a brief summary of what changed

If the file does not exist yet, the very first action is to create it from the schema with the user's stated starting capital. Do not write any trades until the file exists.

## Operations

### `init` — create a new portfolio

Inputs: starting capital in SAR, broker commission rate (default 0.155%), min commission (default 12 SAR).

Action: write a fresh `paper_portfolio.json` with empty positions and trade_history.

### `view` — show current state

Action: read the file, format a report:

```
# 📝 PAPER PORTFOLIO — As of [date]

**Starting capital:** SAR X
**Current value:** SAR Y (cash + positions at last known price)
**Realized P/L:** SAR ±Z
**Open P/L (mark-to-market):** SAR ±W (only if recent prices fetched)

## Cash
- Available: SAR X
- Settling (T+2): SAR Y, available [date]

## Positions

| Ticker | Name | Shares | Avg Cost | Current* | Value | Unrealized P/L | Shariah |
|---|---|---|---|---|---|---|---|

*Current prices fetched live; mark stale if web fetch failed*

## Recent Trades (last 10)

| Date | Action | Ticker | Shares | Price | Net SAR |
|---|---|---|---|---|---|

## Purification Owed
- SAR X (from dividend events)
```

To get current prices, use `tadawul-stock-analyzer`'s live-data fetch logic.

### `record_buy` — simulate a buy

Inputs from user (ask if missing): ticker, shares, fill price (SAR), rationale.

**Pre-checks (in order):**
1. Shariah status — call `shariah-compliance-screener` if status is unknown or > 7 days old. Refuse if non-compliant.
2. Risk-gate — call `tadawul-risk-gate` to check position sizing, sector concentration, cash availability.
3. Cash sufficiency — gross + commission + VAT must be <= available cash.
4. Tick alignment — fill price must match current Tadawul tick bands (see `tadawul-position-sizer`).

If any pre-check fails, refuse the trade with a specific reason. Do not write to the file.

**On success:**
- Compute commission: `MAX(min_commission_sar, gross × commission_rate)`
- Compute VAT: `commission × vat_rate`
- Net cost: `gross + commission + vat`
- Update cash: `available_sar -= net_cost`
- Update positions: if ticker exists, weighted-average the cost; else add new entry
- Append to trade_history with auto-incremented trade_id
- Append to journal (call `tadawul-trade-journal`) and store the journal_id in the trade

### `record_sell` — simulate a sell

Inputs: ticker, shares, fill price.

**Pre-checks:**
1. Position exists with at least the requested shares.
2. Tick alignment.

**On success:**
- Compute commission and VAT (same formula)
- Gross proceeds: `shares × price`
- Net proceeds: `gross - commission - vat`
- Compute realized P/L for this lot: `(price - avg_cost) × shares - commission - vat`
- Update cash: settled cash goes to `settling_sar` with `settling_until = trade_date + 2 business days`. Available_sar increases only after T+2 simulated.
- Update positions: reduce shares; remove entry if shares == 0
- Update `realized_pnl_sar`
- Append to trade_history
- Append to journal

### `mark_dividend` — record a dividend received

Inputs: ticker, dividend amount per share, ex-date.

Action:
- Compute total dividend: `shares × per_share`
- Add to cash (immediately, simplified — real dividends settle later)
- Compute purification: `total × position.purification_rate_pct / 100`
- Add to `purification_owed_sar`
- Append to trade_history with action="DIVIDEND"

### `settle_pending_cash` — move settling cash to available

Run this at the start of each day to simulate T+2 settlement. Any `settling_until` <= today moves to `available_sar`.

### `record_purification_payment` — log a charity donation

Inputs: amount paid, recipient (free text).

Action: subtract from `purification_owed_sar`. Append to a separate purification ledger entry in trade_history with action="PURIFICATION".

## Anti-patterns

- ❌ Never modify the JSON without reading it first — concurrent edits will corrupt state
- ❌ Never invent fields not in the schema — if you need new state, propose a schema update to the user first
- ❌ Never skip the Shariah/risk pre-checks "for testing" — that defeats the experiment's purpose
- ❌ Never round monetary values silently — keep two decimals for SAR
- ❌ Never let cash go negative — refuse the trade instead
- ❌ Never report the simulated portfolio without the "📝 PAPER PORTFOLIO" header
- ❌ Never use the word "executed" — use "recorded" or "simulated"
- ❌ Never call this skill for real-money trades — refuse and direct the user to their broker

## References

- `references/schema.md` — full JSON schema with field definitions
- `references/cost-model.md` — Saudi broker commission and VAT calculation details
