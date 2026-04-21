---
name: tadawul-risk-gate
description: Use this skill BEFORE recording any paper buy or sell to enforce hard pre-trade safety checks — Shariah compliance, position sizing limits, sector concentration, daily loss limit, cash availability, tick alignment, liquidity. Triggers automatically from tadawul-paper-portfolio.record_buy and record_sell, and on user requests like "is this trade safe to take?", "would this trade pass my rules?", "check this order before I record it". Returns either PASS (with all check results) or REFUSE (with the specific rules that blocked it). Hard rules are non-negotiable — this skill never approves a trade that breaks them, even if the user insists.
---

# Tadawul Risk Gate

## Overview

A pre-trade safety layer. Every proposed paper trade goes through this gate. The skill returns `PASS` or `REFUSE` with explicit reasons. Hard rules cannot be overridden mid-experiment — if you want to change them, you change the rules file deliberately, not in the moment when emotion or excitement might be driving.

This is the most important "boring" skill in the bundle. Most blow-ups come from breaking rules in the moment, not from analytical mistakes.

## Inputs

```json
{
  "action": "BUY | SELL",
  "ticker": "2222",
  "shares": 10,
  "price_sar": 28.50,
  "stop_sar": 27.30,           // for BUY only
  "current_portfolio_state": { ... },  // from tadawul-paper-portfolio
  "rules_override_path": null  // optional, default uses default rules
}
```

## Default rule set

These are the defaults; users can override via a `risk_rules.json` file in the working directory. The skill reads that file if present, otherwise uses these:

```json
{
  "max_risk_per_trade_pct": 1.0,
  "max_position_pct_blue_chip": 15.0,
  "max_position_pct_mid_cap": 8.0,
  "max_position_pct_small_cap": 5.0,
  "max_position_pct_nomu": 3.0,
  "max_sector_pct": 30.0,
  "max_combined_energy_petchem_pct": 40.0,
  "max_top_5_concentration_pct": 50.0,
  "min_cash_floor_pct": 10.0,
  "max_daily_loss_pct": 3.0,
  "max_position_count": 15,
  "min_position_count_warning": 5,
  "max_pct_of_avg_daily_volume": 10.0,
  "stop_must_be_inside_daily_limit": true,
  "shariah_status_max_age_days": 7,
  "non_compliant_block": true,
  "doubtful_warn_only": true
}
```

## Check sequence

Run these in order. Stop at first hard failure. Soft warnings accumulate but don't block.

### Hard checks (any failure → REFUSE)

**H1. Shariah compliance**
- For BUY: status must be `compliant_clean` or `compliant_purify` and verified within `shariah_status_max_age_days`.
- If status is `non_compliant`: REFUSE.
- If status is `doubtful`: WARN (proceeds unless `doubtful_warn_only` is false).
- If status is unknown or stale: REFUSE — user must run `shariah-compliance-screener` first.

**H2. Cash availability** (BUY only)
- Available cash >= gross + commission + VAT.
- If insufficient: REFUSE with the exact shortfall.

**H3. Position exists** (SELL only)
- Open position with at least the requested shares must exist.
- If not: REFUSE.

**H4. Tick alignment**
- Both price and stop must align with current Tadawul tick bands (see `tadawul-position-sizer` for the matrix).
- If not aligned: REFUSE and suggest the nearest valid tick.

**H5. Risk per trade** (BUY only)
- Risk amount = `shares × (price - stop)`
- `risk_pct = risk / total_account_value × 100`
- If `risk_pct > max_risk_per_trade_pct`: REFUSE.

**H6. Position concentration** (BUY only)
- Determine the cap from cap-class (blue-chip / mid-cap / small-cap / Nomu) — based on TASI top-30 status, market cap.
- New position value = `shares × price` (plus existing position if averaging up)
- New position pct = position_value / total_account_value × 100
- If exceeds the cap: REFUSE with the max-allowed share count.

**H7. Sector concentration** (BUY only)
- Compute new sector weight after this trade.
- If `> max_sector_pct`: REFUSE.
- For energy/petchem combined: check `max_combined_energy_petchem_pct` separately.

**H8. Daily loss limit**
- Compute today's realized + unrealized loss vs start-of-day account value.
- If `loss_pct >= max_daily_loss_pct`: REFUSE all new BUYs for the rest of the session.
- SELLs are still allowed (closing positions to limit further damage).

**H9. Stop inside daily limit** (BUY only, if rule enabled)
- Check `(price - stop) / price <= 0.10` (Main Market) or `<= 0.30` (Nomu).
- If outside: REFUSE — the stop could be skipped by a limit-down session.

### Soft warnings (accumulate; don't block)

**S1. Liquidity**
- Position value > `max_pct_of_avg_daily_volume` × ADV.
- WARN: "Position is X% of average daily volume — slippage risk."

**S2. Position count**
- Already at `max_position_count`: WARN "Adding this would exceed the manageable count."
- Below `min_position_count_warning` and selling out the last position in a sector: WARN.

**S3. Top-5 concentration**
- After this trade, top 5 = > `max_top_5_concentration_pct`: WARN.

**S4. Cash floor**
- After this trade, cash% < `min_cash_floor_pct`: WARN "No dry powder for opportunities."

**S5. Weekend gap risk**
- If trade would be entered Wed close → Thu and held over weekend, and stock has high oil/macro sensitivity: WARN.

## Output format

```json
{
  "verdict": "PASS | REFUSE | PASS_WITH_WARNINGS",
  "trade_summary": { ... echo of input ... },
  "hard_check_results": {
    "H1_shariah": "PASS | REFUSE: <reason>",
    "H2_cash": "PASS | REFUSE: <reason>",
    "H3_position": "PASS | REFUSE: <reason>",
    "H4_tick": "PASS | REFUSE: <reason>",
    "H5_risk_per_trade": "PASS | REFUSE: <reason>",
    "H6_position_cap": "PASS | REFUSE: <reason>",
    "H7_sector_cap": "PASS | REFUSE: <reason>",
    "H8_daily_loss": "PASS | REFUSE: <reason>",
    "H9_stop_in_limit": "PASS | REFUSE: <reason>"
  },
  "soft_warnings": [
    "S1: Position is 15% of ADV — slippage risk",
    "S4: Cash drops to 8% — below recommended floor"
  ],
  "max_allowed_shares": 526,
  "rationale": "Brief plain-language summary"
}
```

For human-readable report:

```
# 🛡️ RISK GATE — [PASS / PASS WITH WARNINGS / REFUSE]

**Trade:** BUY 833 shares of 2222 at SAR 28.50 (stop SAR 27.30)

## Hard Checks
✅ Shariah: Compliant (Argaam, verified today)
✅ Cash: Sufficient (SAR 23,742 needed, SAR 100,000 available)
✅ Position exists: N/A (BUY)
✅ Tick alignment: 28.50 and 27.30 are valid in 0.02 band
✅ Risk per trade: SAR 1,000 (1.0% of account, at limit)
❌ Position cap: 23.7% of account exceeds 15% blue-chip cap
✅ Sector cap: Energy would be 23% (under 30%)
✅ Daily loss: Within limit
✅ Stop in daily limit: Yes (-4.2% from current)

## Soft Warnings
None

## Verdict: ❌ REFUSE
**Reason:** Position size exceeds 15% concentration cap.
**Max allowed shares:** 526 (= SAR 14,991 = 15.0% of account)

## Suggested fix
Reduce share count to 526. Risk would drop to SAR 631 (0.63% of account) — under the risk budget. The cap binds, not the risk budget.
```

## Operations

### `check_trade` — main entry point

Runs all hard and soft checks in order. Returns the structured output above.

### `read_rules` — show current rules

Returns the active rules — either from `risk_rules.json` or defaults.

### `update_rules` — modify rules deliberately

Inputs: rule key, new value, justification.

This is intentionally separate from the trade flow. Changing rules requires the user to explicitly call this skill, not happen as a side effect of "approving" a trade. The justification is logged.

## Anti-patterns

- ❌ Never approve a trade that fails a hard check, even if the user says "just this once"
- ❌ Never silently relax a rule because the trade looks good — change the rule file deliberately
- ❌ Never run only some checks — always run the full sequence
- ❌ Never report PASS without listing each check's result — opacity defeats the safety purpose
- ❌ Never override the daily loss limit "because the next trade will be a winner" — that's exactly when you most need the limit

## References

- `references/default-rules.json` — the canonical default rule set
- `references/cap-class-criteria.md` — how to classify a stock as blue-chip / mid-cap / small-cap / Nomu
