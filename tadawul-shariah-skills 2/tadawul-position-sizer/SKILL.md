---
name: tadawul-position-sizer
description: Use this skill when the user asks how many shares to buy of a Tadawul stock, what their position size should be, whether a position is too big, or how to size a trade given an account, risk percentage, entry, and stop. E.g., "how many shares of 2222 should I buy?", "I have SAR 80K, want to buy STC at 40 with stop at 38, how much?", "is this position too big for my account?". Uses fixed-risk model adapted for Saudi market — SAR currency, current Tadawul tick-size bands (effective June 29, 2025), ±10% daily price limit, liquidity check against average daily traded value, broker commission and VAT. Hard-caps positions by % of account regardless of risk-budget allowance.
---

# Tadawul Position Sizer

## Overview

Computes position size using the **fixed-risk model** — risk a fixed SAR amount per trade regardless of conviction. Adapted for SAR amounts and current Saudi market microstructure.

## Core Formula

```
Position Size (shares) = (Account × Risk %) / (Entry − Stop)
Position Value (SAR) = Position Size × Entry Price
```

**Risk % per trade** — conservative norms:
- Beginner / testing: **0.5% – 1%** of account
- Experienced: **1% – 2%** of account
- Aggressive: 2%+ (not recommended)

## Inputs Required

Always confirm these with the user before calculating:

| Input | Example |
|---|---|
| Account size (SAR) | 100,000 |
| Risk per trade (% of account) | 1% |
| Entry price (SAR) | 30.00 |
| Stop-loss price (SAR) | 28.50 |
| Ticker (for liquidity + Shariah check) | 2222 |

If any input is missing, ask once. Don't guess.

## Calculation Workflow

### Step 1 — Compute Risk per Share
```
Risk per share = Entry − Stop  (for long position)
```
(Short selling is not Shariah-compliant; this skill assumes long-only.)

### Step 2 — Compute Risk Budget (SAR)
```
Risk budget = Account × Risk %
```

### Step 3 — Compute Raw Position Size
```
Raw shares = Risk budget / Risk per share
```

### Step 4 — Apply Tadawul Adjustments

**a) Lot size:** Tadawul Main Market trades in single-share lots. Round DOWN to whole shares.

**b) Tick size compliance** — current tick-size matrix effective June 29, 2025:

| Price range (SAR) | Tick (SAR) |
|---|---|
| 0.01 – 24.99 | 0.01 |
| 25.00 – 49.98 | 0.02 |
| 50.00 – 99.95 | 0.05 |
| 100.00 – 249.90 | 0.10 |
| 250.00 – 499.80 | 0.20 |
| 500.00 and above | 0.50 |

Source: Saudi Exchange announcement, applies to Main Market and Nomu, excludes debt instruments. Adjust entry and stop to nearest valid tick before placing the order.

**c) Position cap by % of account** — no single position should exceed:
- **15% of account** for blue-chips (TASI top 30)
- **8% of account** for mid-caps
- **5% of account** for small/Nomu stocks
- **3% of account** for thinly-traded names

If the calculated position exceeds the cap, **the cap binds** (the cap wins, not the risk budget). On low-volatility blue-chips with tight stops the cap will frequently bind — this is the system working correctly, not a problem.

**d) Liquidity check:**
- Position should not exceed **10% of average daily traded value** (otherwise you'll move the price)
- Avg daily value = (Avg daily volume × Avg price) — fetch from Argaam or TradingView
- Example: A stock trading SAR 5M/day → max position SAR 500K

**e) Daily price limit context (±10%):**
- Your stop should logically be inside the daily limit, OR you accept the gap risk
- If your stop is below -10% from current, the stock could "limit-down" through your stop without filling
- Flag this to the user explicitly when it occurs

### Step 5 — Compute Final Position
```
Final shares = MIN(
    raw_shares (after tick rounding),
    position_cap_shares,
    liquidity_cap_shares
)

Final position value = Final shares × Entry
Actual risk = Final shares × Risk per share
```

### Step 6 — Add Cost Adjustments

**Saudi brokerage fees** (verify with user's specific broker; typical retail rates):
- Brokerage commission: ~0.10% – 0.155% per side
- Tadawul trading fee: 0.00005 of transaction value
- VAT on commission: 15% on the commission portion
- CMA fee: small fixed levy
- Edaa (depository) fee: small custody charge
- **Round-trip cost estimate:** ~0.30% – 0.40% of trade value

**Adjusted breakeven:** Your target needs to clear costs to be profitable. For a 0.35% round-trip, you need >0.35% price move just to break even.

## Output Template

```
# Position Sizing Calculation

**Trade:** Long [Ticker] [Name]
**Date:** YYYY-MM-DD

## Inputs
- Account size: SAR X
- Risk per trade: X% (= SAR X risk budget)
- Entry: SAR X (tick-aligned: ✅/⚠️ adjusted to SAR Y)
- Stop-loss: SAR X (tick-aligned: ✅/⚠️ adjusted to SAR Y)
- Risk per share: SAR X (X% of entry)

## Position Calculation
- Raw shares (risk-based): X shares
- Position cap (X% of account): Y shares
- Liquidity cap (10% of ADV): Z shares
- **Final position: MIN of above = N shares**

## Trade Details
- Position value: SAR X (X% of account)
- Actual risk: SAR X (X% of account — within budget)
- Round-trip cost estimate (~0.35%): SAR X
- Breakeven move needed: ~0.X% above entry

## Risk-Reward Check
If targeting SAR T:
- Reward per share: SAR (T − Entry)
- Total reward: SAR X
- R/R ratio: X.X : 1

## Tadawul-Specific Flags
- ⚠️ / ✅ Stop is inside daily ±10% limit
- ⚠️ / ✅ Position size < 10% of avg daily value
- ⚠️ / ✅ Position size within concentration cap
- ⚠️ / ✅ Entry and stop are valid ticks

## Suggested Adjustments
[If any flags triggered, suggest fix]
```

## Worked Example

**User scenario:**
- Account: SAR 100,000
- Risk tolerance: 1% per trade (SAR 1,000)
- Stock: 2222 Saudi Aramco
- Entry: SAR 28.50 (tick-aligned ✅, 0.02 tick band)
- Stop: SAR 27.30 (tick-aligned ✅)
- Avg daily value: ~SAR 100M (very liquid)

**Calculation:**
- Risk per share: 28.50 − 27.30 = SAR 1.20
- Raw shares (risk-based): 1,000 / 1.20 = 833 shares
- Raw position value: 833 × 28.50 = SAR 23,740 (23.7% of account)
- ⚠️ Position cap for blue-chip = 15% = SAR 15,000 → 526 shares
- Liquidity cap: 10% of SAR 100M = SAR 10M → not binding

**Final answer:** 526 shares
- Position value: 526 × 28.50 = SAR 14,991 (15.0% of account)
- Actual risk: 526 × 1.20 = SAR 631 (0.63% of account)
- Note: The 15% concentration cap reduced the position below the full risk budget. This is correct behavior — concentration matters more than risk-budget utilization.

## Position Sizing Philosophies

### Fixed-Risk (default — what this skill uses)
Risk same SAR amount per trade regardless of conviction. Most disciplined for beginners and short-term traders.

### Volatility-Adjusted (ATR-based)
Use stop = K × ATR (e.g., 2 × ATR). More volatile stocks → wider stop → smaller position. Same risk regardless of stock volatility.

### Kelly Criterion
Mathematically optimal but assumes you know your win-rate and edge — usually you don't with enough confidence. Use **half-Kelly** at most if applying. Don't use Kelly until you have 100+ trades of evidence.

### Equal-Weight
Every position is X% of account. Simpler but doesn't account for trade-specific risk.

**Recommendation:** Use **fixed-risk** for short-term trading. Add the **position cap** as a concentration limit.

## Concentration Rules

Beyond per-trade sizing, manage portfolio-level concentration:

| Rule | Limit |
|---|---|
| Single position max | 15% of portfolio (blue-chip), less for smaller names |
| Single sector max | 30% of portfolio |
| Top 5 positions max | 50% of portfolio |
| Cash floor | 10–20% reserve recommended for opportunities |
| Number of positions | 5–15 (any more is hard to monitor) |

For Shariah-compliant Saudi portfolios:
- Banks are restricted to ~3 names (Al Rajhi, AlBilad, AlInma) — natural sector cap
- Petrochem/energy can dominate Tadawul exposure — actively diversify

## Anti-Patterns

- ❌ "I'll just buy a round number of shares" — undisciplined
- ❌ Sizing up because "I'm sure about this one" — every trade is uncertain
- ❌ Ignoring the position cap because risk budget allows more
- ❌ Sizing without a defined stop (you can't size if you don't know the risk)
- ❌ Using market orders on illiquid Saudi names — slippage destroys your math
- ❌ Forgetting transaction costs (round-trip ~0.3-0.4%)
- ❌ Holding leveraged or margined positions if Shariah compliance matters (most margin is interest-based / riba)
- ❌ Using stale tick-size bands (the matrix changed June 29, 2025)

## References

- `references/risk-management-saudi.md` — broader risk management for Saudi investors
- `references/sizing-examples.md` — worked examples for different scenarios
