---
name: shariah-compliance-screener
description: Use this skill when the user asks whether a Saudi (Tadawul) or global stock is Shariah-compliant, halal, or permissible to trade — including questions like "is X halal?", "is ticker XXXX shariah compliant?", "should I avoid this from an Islamic perspective?", "how much purification do I owe on dividends from Y?", or any request to screen a stock, watchlist, or portfolio against Islamic finance rules. Applies AAOIFI Shariah Standard No. 21 (the Saudi/Tadawul convention) with two-tier screening: business activity prohibitions plus three financial ratios using 12-month average market cap as denominator. Calculates dividend purification owed. Always fetches the live status from Argaam or Almaqasid before answering — does not rely on memorized status.
---

# Shariah Compliance Screener (Tadawul + Global)

## Overview

Determines whether a listed equity is Shariah-compliant under **AAOIFI Shariah Standard No. 21** — the standard adopted by the Saudi Capital Market Authority (CMA), Tadawul, and most Gulf Shariah boards. Also calculates the amount to be purified (donated) when a stock passes screening but has minor non-permissible income.

**This skill does NOT replace a certified Shariah supervisory board.** Always cross-check with an authoritative live list before trading.

## CRITICAL: Always fetch live data

Shariah lists update **quarterly** (after each earnings season). A stock's status this quarter does not reliably predict next quarter. Memorized lists from training data are unsafe — they may name a stock as compliant when it has since been removed.

**Mandatory first step for every screening request:**

```
web_search query: "argaam shariah list <ticker>"
   OR "argaam shariah compliant companies <year>"
web_fetch: https://www.argaam.com/en/company/shariahcompanies
```

If the user asks about a specific ticker and the Argaam result does not clearly show its status, also try:
```
web_search query: "almaqasid shariah <ticker>"
web_fetch: https://www.almaqasid.com (if returned in results)
```

If neither source returns a clear status within two searches, tell the user: "I could not confirm a current Shariah status from Argaam or Almaqasid for [ticker]. You should check directly with your Shariah board before trading. I can run the AAOIFI ratio calculation on the latest financials if you'd like, but that is a calculation, not a board ruling."

Never declare a stock compliant or non-compliant from training-data memory alone.

## The Two-Tier Screening Process

A stock is compliant ONLY if it passes BOTH tiers.

### Tier 1 — Business Activity Screening (Absolute Prohibitions)

A company is **automatically NON-compliant** if its core business is any of the following:

1. **Conventional banking** (interest-based lending) — excludes most traditional banks
2. **Conventional insurance** — except Takaful (Islamic cooperative insurance)
3. **Alcohol** — production, distribution, or sale
4. **Pork-related products**
5. **Gambling, casinos, lotteries**
6. **Adult entertainment / pornography**
7. **Tobacco** — most Shariah boards exclude; some tolerate if < 5%
8. **Weapons for mass destruction / offensive arms**
9. **Conventional bonds / Riba-based financial services** as primary business
10. **Hotels with significant alcohol or gambling revenue** (case-by-case)

**Mixed-business rule:** If the above activities are NOT the core business but generate *some* revenue, the company may still pass IF non-permissible income < 5% of total revenue (see Tier 2).

### Tier 2 — Financial Ratio Screening (AAOIFI SS21)

All three ratios must pass. **Denominator = 12-month average market capitalization** (AAOIFI / Tadawul convention — NOT total assets).

| Ratio | Formula | Threshold |
|---|---|---|
| **Debt Ratio** | Interest-bearing debt ÷ 12-month avg market cap | **< 30%** |
| **Cash + Interest-Bearing Securities Ratio** | (Cash + Interest-bearing deposits + conventional bonds) ÷ 12-month avg market cap | **< 30%** |
| **Non-Permissible Income Ratio** | Revenue from prohibited activities ÷ Total revenue | **< 5%** |

> **Methodology variants — apply the strictest (AAOIFI 30% / market cap):**
> - AAOIFI & Tadawul: 30% / 12-month avg market cap (use this for Saudi stocks)
> - S&P/Dow Jones Islamic: 33% / market cap
> - MSCI Islamic: 33% / total assets

## Screening Workflow

### Step 1 — Fetch Authoritative Live List (mandatory)

See "CRITICAL: Always fetch live data" above. Run web_search and web_fetch before anything else.

### Step 2 — If Not on a List, Compute the Ratios

Fetch the latest financials:
```
web_fetch: https://www.argaam.com/en/tadawul/tasi/<company-slug>
   (e.g., /saudi-aramco for ticker 2222)
```

Compute:
```
Business Activity Check:
  → Review revenue breakdown (Argaam company profile or annual report)
  → If >5% revenue from prohibited activities → FAIL Tier 1

Financial Ratios (using 12-month avg market cap):
  Debt Ratio           = Total Interest-Bearing Debt / Avg Market Cap
  Cash Ratio           = (Cash + Interest Deposits + Conv. Bonds) / Avg Market Cap
  Non-Permissible Inc. = Non-Halal Revenue / Total Revenue
  
All three must pass their thresholds.
```

### Step 3 — Report the Verdict

Use one of four clear labels:

| Label | Meaning |
|---|---|
| ✅ **COMPLIANT (Clean)** | Passes all screens; 0% non-permissible income |
| ✅ **COMPLIANT (Requires Purification)** | Passes all screens but has non-zero (< 5%) non-permissible income — purify dividends |
| ⚠️ **DOUBTFUL / BORDERLINE** | One ratio is within 2% of its threshold — risk of being dropped next quarter |
| ❌ **NON-COMPLIANT** | Fails any screen |

### Step 4 — Calculate Purification (if applicable)

When a stock is "Compliant — Requires Purification":

```
Purification Amount = (Non-Permissible Income % / 100) × Dividends Received
```

Some scholars also apply purification to capital gains (a minority view). Conservative approach:
```
Capital Gain Purification = (Non-Permissible Income % / 100) × Capital Gain
```

**Example:**
- You receive SAR 1,000 in dividends from stock X.
- Company X has 3% non-permissible income.
- Purification = 0.03 × 1,000 = **SAR 30** to be donated to charity.

**Important:** Purification is NOT zakat. It cannot benefit the donor (cannot be given to family members, cannot be claimed for religious reward — scholars treat it as removing impurity, not earning thawab).

## Reporting Template

```
# Shariah Compliance Report: [Company Name] ([Ticker])

## Verdict: [✅ / ⚠️ / ❌] [Label]

## Source & Date
- Primary source: [Argaam list / Almaqasid / calculated from financials]
- List date / fiscal data as of: [YYYY-MM-DD]
- URLs fetched: [list]

## Tier 1 — Business Activity
- Core business: [description]
- Any prohibited revenue streams? [Yes/No + details]

## Tier 2 — Financial Ratios (AAOIFI SS21)
| Ratio | Value | Threshold | Pass? |
|---|---|---|---|
| Debt / Avg Market Cap | X% | <30% | ✅/❌ |
| Cash+IBS / Avg Market Cap | X% | <30% | ✅/❌ |
| Non-Permissible Income / Revenue | X% | <5% | ✅/❌ |

## Purification
- Dividend purification rate: X%
- Per SAR 1,000 dividend → SAR X to charity

## Notes & Caveats
- [Any borderline ratios, upcoming concerns, scholar disagreements]
- Status can change quarterly — reverify before each purchase.

## Sources
- [Actual URLs fetched]
```

## Special Saudi Market Considerations

1. **Banks:** Most conventional Saudi banks (e.g., SNB 1180, SABB 1060, Riyad Bank 1010, ANB 1080, BSF 1050, BJAZ 1020, SAIB 1030) are **NON-compliant** under AAOIFI. Only **Al Rajhi Bank (1120)**, **AlBilad Bank (1140)**, and **AlInma Bank (1150)** operate on fully Islamic principles. Some boards also clear AlJazira (1020) — verify quarterly.
2. **Insurance:** Conventional insurance is non-compliant. Only **Takaful** companies pass (e.g., Al Rajhi Takaful 8230, Jazira Takaful 8012).
3. **REITs:** Most Saudi REITs are Shariah-compliant by design (structured under CMA's Islamic guidelines) but verify — some have conventional leverage.
4. **IPOs:** When a new company lists, it may take a quarter before being added to the Shariah list. If Shariah status matters, wait for the next list update.
5. **Sukuk vs. Bonds:** Sukuk are Shariah-compliant fixed-income; conventional bonds are not.
6. **Argaam list updates** ~mid-month after each quarter-end (mid-May, mid-August, mid-November, mid-March).

## Anti-Patterns (Things to NEVER Do)

- ❌ Do not declare a stock "halal" from training-data memory — always fetch live list first.
- ❌ Do not declare a stock "halal forever" — status is reviewed quarterly.
- ❌ Do not use total-assets denominator for Saudi stocks (that's MSCI convention, not Tadawul's).
- ❌ Do not ignore the business-activity screen just because the ratios pass — a casino with zero debt is still haram.
- ❌ Do not issue personal fatwa — cite the Shariah board whose methodology is being applied (AAOIFI, Al Rajhi, Almaqasid).
- ❌ Do not confuse purification with zakat — they are separate obligations.

## References

- `references/aaoifi-ss21-summary.md` — detailed AAOIFI SS21 methodology
- `references/purification-guide.md` — purification rules, scholar differences, examples
- `references/saudi-shariah-lists.md` — where to find each authoritative list
- `references/prohibited-sectors.md` — guidance on borderline sectors
