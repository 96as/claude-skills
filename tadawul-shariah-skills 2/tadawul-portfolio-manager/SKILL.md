---
name: tadawul-portfolio-manager
description: Use this skill when the user shares a Tadawul portfolio (list of holdings) and asks for a review, or asks about concentration, sector exposure, performance vs TASI, rebalancing, or running purification ledger. Triggers on "review my portfolio", "am I overconcentrated in X", "track my purification owed", "should I rebalance", "compare my returns to TASI". Re-verifies Shariah status of every holding against current Argaam list (positions can become non-compliant between quarters), checks concentration caps (15% single position, 30% single sector), and computes purification owed on dividends received.
---

# Tadawul Portfolio Manager

## Overview

Helps an investor monitor a Tadawul portfolio holistically — concentrations, drift, performance, Shariah status changes, running purification obligation. Unlike single-stock analysis, this skill looks at the WHOLE picture.

## Required live data

For every portfolio review, fetch:
1. Current price for each holding (via Argaam, TradingView, or Saudi Exchange)
2. Current Shariah status for each holding (via Argaam Shariah list — re-verify; a holding may have moved off the list since last review)
3. TASI level and 1W/1M/YTD performance for benchmarking

```
For each ticker:
  web_search query: "<ticker> tadawul price"
  
For Shariah:
  web_fetch: https://www.argaam.com/en/company/shariahcompanies
  
For TASI:
  web_fetch: https://www.tradingview.com/symbols/TADAWUL-TASI/
```

If you cannot fetch a current price for a holding, mark it explicitly as "stale" in the report rather than guessing.

## When to Use

- User shares a portfolio (list of tickers + quantities) and asks for review
- User asks "am I too concentrated in X sector?"
- User asks about portfolio performance, P/L, or comparison to TASI
- User asks "how much purification do I owe?"
- User asks "should I rebalance?"
- Periodic (monthly/quarterly) portfolio health check

## Inputs Needed

| Required | Optional |
|---|---|
| List of holdings: ticker, shares, avg cost | Cash balance |
| Account size or total invested capital | Date of last review |
| | Dividends received (with dates) for purification calc |
| | Any closed trades since last review |

If the user only gives tickers + share counts, you can still do most of the work — just note what's missing.

## Workflow

### Step 1 — Build the Position Table

For each holding, fetch current price. Compute:

```
Position Value = Shares × Current Price
Cost Basis = Shares × Avg Cost
Unrealized P/L (SAR) = Position Value − Cost Basis
Unrealized P/L (%) = (Current − Avg Cost) / Avg Cost
% of Portfolio = Position Value / Total Portfolio Value
```

### Step 2 — Shariah Compliance Re-Verification

For each holding:
- Verify still on the latest Argaam (or chosen board's) Shariah list
- Note any holdings that have moved off the list
- Flag any in "Doubtful" status

**Critical:** If a stock has become non-compliant, surface this prominently. The user should consider exit within 90 days (most scholars' grace period) and purify any gains accumulated during the non-compliant period.

### Step 3 — Sector & Concentration Analysis

Group holdings by sector. Compute:

| Check | Threshold | Flag if breached |
|---|---|---|
| Largest single position | 15% | ⚠️ Concentrated |
| Largest sector | 30% | ⚠️ Sector-heavy |
| Top 5 positions | 50% | ⚠️ Very concentrated |
| Number of positions | 5-15 ideal | < 5 = under-diversified; > 15 = hard to monitor |
| Cash % | 10-20% target | < 10% = no dry powder |

For Shariah-compliant Tadawul portfolios, watch especially:
- **Petrochemical / Energy combined** — easy to over-concentrate
- **Islamic banks** (only 3 names: Al Rajhi, AlBilad, AlInma — natural concentration)
- **Healthcare** — recent Vision 2030 darlings, easy to overweight

### Step 4 — Performance Review

```
Period Return (SAR) = Current Value − Cost Basis
Period Return (%) = Period Return / Cost Basis × 100
```

If user provides date range, compare:
- vs TASI total return for same period
- vs MSCI Tadawul 30 (MT30) if available
- vs Saudi Shariah index if known

**Win/Loss breakdown:**
- # of winners vs losers
- Avg gain on winners
- Avg loss on losers
- Win rate (closed trades)

### Step 5 — Purification Ledger

If user provides dividend history:

```
For each dividend received:
  Purification = Dividend Amount × (Non-Permissible % / 100)
  
Total Purification Owed = Sum of all unpaid purifications
```

Default purification rates by stock if not provided:
- Fully Islamic banks (Al Rajhi, AlBilad, AlInma): typically 0%
- Saudi Aramco, SABIC, STC, Almarai: estimate 1-3% (verify with Argaam)
- General Tadawul large-caps: 1-3% typical
- ⚠️ When in doubt, ASK the user to confirm the rate from their Shariah board

### Step 6 — Action Recommendations

Based on the analysis, surface concrete actions:

1. **Compliance fixes:** Stocks that became non-compliant → exit timeline
2. **Concentration trims:** Positions exceeding caps → suggest trim amounts (in SAR)
3. **Diversification gaps:** Sectors entirely missing
4. **Stop-loss reviews:** Positions deeply in loss without active management
5. **Profit-taking:** Big winners that have run far → consider trim/trail
6. **Purification due:** Total SAR amount owed

### Step 7 — Format the Report

```markdown
# Portfolio Review — [User Name or "My Portfolio"]
**Date:** YYYY-MM-DD | **Total Value:** SAR X | **Total Cost:** SAR Y
**Total Unrealized P/L:** SAR ±Z (±X%)

## 1. Holdings Table

| Ticker | Name | Shares | Avg Cost | Current | Value (SAR) | P/L (SAR) | P/L % | % Port | Shariah |
|---|---|---|---|---|---|---|---|---|---|
| 2222 | Aramco | 1000 | 28.00 | 30.50 | 30,500 | +2,500 | +8.9% | 25.4% | ✅ Pur 1.5% |
| 1120 | Al Rajhi | 200 | 90.00 | 95.20 | 19,040 | +1,040 | +5.8% | 15.9% | ✅ Clean |
| ... |

## 2. Cash & Reserves
- Cash balance: SAR X (X% of portfolio)
- Status: [Adequate / Low / High]

## 3. Sector Allocation

| Sector | SAR | % | Cap | Status |
|---|---|---|---|---|
| Energy | X | 25% | 30% | ✅ |
| Banking (Islamic) | X | 16% | 30% | ✅ |
| Petrochem | X | 18% | 30% | ✅ |
| ... |

**Combined Energy + Petrochem:** X% [⚠️ if > 40%]

## 4. Concentration Check
- Largest position: [Ticker] at X% [✅/⚠️]
- Top 5 concentration: X% [✅/⚠️]
- Number of positions: X [✅/⚠️]

## 5. Shariah Status
- ✅ Compliant: X positions
- ✅ Compliant w/ purification: X positions
- ⚠️ Doubtful: X positions [list]
- ❌ Became non-compliant: X positions [list — exit plan needed]

## 6. Performance
- Portfolio return (since cost basis): ±X%
- TASI same period: ±Y%
- Relative: ±(X − Y) percentage points
- Win rate (closed trades, if data): X%

## 7. Purification Ledger (if provided)
- Total dividends received this period: SAR X
- Total purification owed: SAR Y
- Already paid: SAR Z
- **Outstanding to charity: SAR (Y − Z)**

## 8. Action Items (Prioritized)
1. 🔴 [Compliance / Risk] - [specific action]
2. 🟡 [Concentration] - [specific action]
3. 🟢 [Optimization] - [specific action]

## 9. Suggested Next Review Date
[Typically 4-6 weeks for active short-term traders, monthly for longer-term]
```

## Saudi-Specific Considerations

### 1. Limited Shariah Banking Universe
Only 3-4 Islamic banks on Tadawul (Al Rajhi, AlBilad, AlInma, sometimes AlJazira). If you want banking exposure as a Shariah-compliant investor, you're choosing among these 3-4 — don't expect 10+ options.

### 2. Heavy Petrochem/Energy Tilt
Tadawul's largest names are oil-linked. A "Saudi blue-chip portfolio" can easily be 40-50% oil-correlated. Actively diversify into healthcare, telecom, retail, food.

### 3. Dividend-Heavy Market
Saudi market pays generous dividends (4-7% yields common). Distinguish:
- **Total return** (price change + dividends)
- **Dividend yield received this year**
- **Purification owed on those dividends**

### 4. Government-Linked Names
PIF stake provides downside cushion but also caps explosive upside. Know your gov't-linked exposure (Aramco, SABIC, STC, Maaden, ACWA, etc.).

### 5. Quarterly Shariah Re-screen
Lists update after each earnings season. Re-verify every quarter:
- Mid-May (Q1 results in)
- Mid-August (Q2 results in)
- Mid-November (Q3 results in)
- Mid-March (Q4/annual results in)

### 6. Zakat (Separate from Purification)
For Saudi/GCC nationals, zakat on tradable shares = 2.5% of market value annually if held for trading. Different rules for long-term holdings. Track zakatable wealth separately — this skill does NOT calculate zakat (consult a scholar).

## Anti-Patterns

- ❌ Reviewing P/L without re-verifying Shariah status
- ❌ Calculating concentration only by # of positions, not by sector exposure
- ❌ Forgetting purification on dividends received
- ❌ Holding non-compliant stocks indefinitely "until they recover"
- ❌ Comparing only to TASI without considering you have a Shariah-filtered universe
- ❌ Adding to losing positions without thesis review (averaging down without a reason)

## References

- `references/portfolio-frameworks.md` — different portfolio management styles
- `references/purification-ledger-template.md` — fillable purification tracker
- `references/saudi-sectors-map.md` — sector classifications used on Tadawul
