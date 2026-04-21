---
name: tadawul-sector-rotation
description: Use this skill when the user asks about Saudi (Tadawul) sector performance, leadership, rotation, or how to align sector exposure with the market cycle — e.g., "what sectors are leading on Tadawul?", "is sector rotation happening?", "what's the market cycle phase for Saudi?", "should I rotate from petrochem to healthcare?". Pulls live sector performance from Argaam or Saudi Exchange across 1D/1W/1M/3M/YTD timeframes, identifies leaders, laggards, and active rotation, and overlays Saudi-specific drivers (oil price, Vision 2030 contracts, Hajj/Umrah season, MSCI/FTSE rebalances). Filters banking analysis to Islamic banks only for Shariah-aware investors.
---

# Tadawul Sector Rotation Analyzer

## Overview

Tracks relative performance across Tadawul sectors to identify which are leading or lagging, detect rotation, and provide market-cycle context. Built for Shariah-aware investors — banking sector analysis filters to Islamic banks only.

## Required live data

```
web_fetch: https://www.argaam.com/en/sector
   (sector performance table, 1D/1W/1M/YTD)
   
web_fetch: https://www.tradingview.com/symbols/TADAWUL-TASI/
   (TASI level, recent moves)
   
web_search query: "tadawul sector performance this week"
   (recent commentary on which sectors are leading)
   
web_search query: "saudi oil price brent today"
   (key driver for energy/petrochem sectors)
```

If sector data is not freshly available from web sources, tell the user that the rotation analysis is based on stale data and suggest they check Argaam directly before acting.

## When to Use

- User asks "which Saudi sectors are hot/cold?"
- User wants to rotate from one sector to another
- User asks about market cycle stage (early/mid/late/recession)
- User shares portfolio and asks if sector exposure aligns with current trends
- Periodic (weekly/monthly) sector breadth check

## Workflow

### Step 1 — Gather Sector Performance Data

Sources (in order of preference):
- **Argaam sector performance:** `argaam.com/en/sector` — gives 1D, 1W, 1M, YTD per sector
- **Saudi Exchange sector indices:** Official sector sub-indices on saudiexchange.sa (22 sectors)
- **TradingView:** TADAWUL sector indices, charts available

Pull data for these timeframes:
- 1 Day (today's leadership)
- 1 Week (short-term momentum)
- 1 Month (medium-term trend)
- 3 Months (rotation timeframe)
- YTD (year leadership)

### Step 2 — Identify Leaders & Laggards

Rank all sectors by 1-month and 3-month returns. Categorize:

| Tier | Definition |
|---|---|
| 🔥 Leading | Top 25% over both 1M and 3M |
| 📈 Strengthening | Bottom in 3M but top in 1M (rotating IN) |
| 📉 Weakening | Top in 3M but bottom in 1M (rotating OUT) |
| ❄️ Lagging | Bottom 25% over both 1M and 3M |

This is the **Relative Rotation Graph (RRG)** concept simplified.

### Step 3 — Cross-Reference with Market Cycle

| Cycle Phase | Outperforming Sectors |
|---|---|
| **Early Cycle** (recovery starts) | Cyclicals: Materials, Consumer Discretionary, Real Estate, Industrials |
| **Mid Cycle** (expansion) | Tech / Communication, Industrials, Healthcare |
| **Late Cycle** (peak nearing) | Energy, Materials, Healthcare (defensives gaining) |
| **Recession** | Defensives: Consumer Staples, Healthcare, Utilities, Telecom |

For **Saudi specifically**, also consider:
- **Oil price level** — high oil → energy leadership
- **Saudi government spending pace** — Vision 2030 contracts → infrastructure, mining, healthcare, tourism
- **Interest rate cycle (US Fed → SAMA mirror)** — rates down → REITs, growth; rates up → conventional banks (but Islamic banks less so), defensives

### Step 4 — Identify Saudi-Specific Drivers

| Driver | Effect |
|---|---|
| Oil price spike | Energy ✅ Petchem ✅ |
| Hajj/Umrah season | Hotels ✅ Transport ✅ Retail ✅ Telecom ✅ |
| Major NEOM/Red Sea contract | Construction ✅ Mining ✅ Cement ✅ |
| New Saudi PMI release | Services if positive |
| US Fed rate decision | Banks reaction (Islamic less direct) |
| Quarterly earnings season | Whichever sector reports strong/weak |
| Geopolitical risk in region | Defense (excluded for Shariah), Gold, Energy |
| MSCI/FTSE EM rebalance | Forced flows into Saudi large-caps |

### Step 5 — Generate Sector Report

```markdown
# Tadawul Sector Rotation — [Date]

## TASI Snapshot
- TASI level: X
- 1W: ±X% | 1M: ±X% | 3M: ±X% | YTD: ±X%
- Trend: [Up / Down / Sideways]

## Sector Performance Table

| Sector | 1D % | 1W % | 1M % | 3M % | YTD % | Tier |
|---|---|---|---|---|---|---|
| Energy | | | | | | 🔥 Leading |
| Petrochem | | | | | | 📈 Strengthening |
| Islamic Banks | | | | | | ❄️ Lagging |
| Telecom | | | | | | 📉 Weakening |
| Healthcare | | | | | | 🔥 Leading |
| ...[all sectors]... | | | | | | |

## Leaders (Top 3)
1. **[Sector]** — [why it's leading; specific catalyst]
2. **[Sector]** — [...]
3. **[Sector]** — [...]

## Laggards (Bottom 3)
1. **[Sector]** — [why it's lagging; what could change]
2. ...

## Active Rotation Detected
- **Money flowing IN:** [sectors strengthening from below]
- **Money flowing OUT:** [sectors weakening from above]

## Market Cycle Read
- **Estimated phase:** [Early / Mid / Late / Recession]
- **Key evidence:** [oil price, interest rates, PMI, sector behavior]
- **Sector implications:** [what should outperform from here]

## Saudi-Specific Catalysts (next 30 days)
- [Earnings season for which sectors]
- [Vision 2030 announcements expected]
- [Hajj / Ramadan / Eid impact if applicable]
- [Oil-related events: OPEC+, Aramco specific]

## Suggested Sector Tilts (for review)
| Sector | Current Weight in Portfolio | Suggested | Rationale |
|---|---|---|---|
| ... | | | |

## Names to Watch (Shariah-Compliant)
For each leading sector, 2-3 strong names:
- **[Leading sector]:** Tickers — note any new highs, breakouts, fundamentals
- ...

## Risks to This Read
- [What would invalidate this analysis?]
- [Counter-trend evidence if any]

---
**Note:** Sector rotation is probabilistic. Leadership can shift quickly with news/catalysts. This is a framework for tilt, not absolute prediction.
```

## Saudi-Specific Considerations

### 1. Limited Sector Universe vs US
Tadawul has fewer sectors and fewer names per sector than US markets. Some "sectors" have only 3-5 stocks. Concentration is a feature, not a flaw.

### 2. Banking Sector — Shariah Filter
For Shariah-compliant investors, "Banking" reduces to 3-4 names (Islamic banks). When the sector index moves, your investable subset may behave differently:
- Islamic banks tend to be **less** sensitive to interest rate moves than conventional banks
- They are more influenced by Saudi consumer/SME credit demand

### 3. Petchem ↔ Energy Linkage
On Tadawul, Petrochem and Energy effectively move together. Treating them as separate sectors for rotation is misleading — view them as a combined block.

### 4. Vision 2030 Mega-Cycle
Beyond traditional business cycle, Saudi has a sovereign mega-cycle:
- Tourism build-out (NEOM, Red Sea, Diriyah Gate) → multi-year tailwind for hospitality, construction, services
- Mining diversification → Maaden and related
- Renewables → ACWA Power and related

These can override traditional cyclical patterns for select sub-sectors.

### 5. Foreign Flow Influence
MSCI EM and FTSE EM index rebalancing → forced flows into Saudi large-caps (mostly banks + petchem). Watch quarterly review dates for short-term moves. Note also: as of February 2026, the QFI framework was abolished — direct foreign access could increase flow volatility.

## Anti-Patterns

- ❌ Chasing the hottest sector after it's already run 30%
- ❌ Rotating too frequently (transaction costs eat the edge)
- ❌ Ignoring Shariah filter when looking at "Banking sector strength" — your investable banks may not match
- ❌ Confusing sector noise (1-2 day moves) with rotation (weeks-months)
- ❌ Forgetting that some sectors have only 3-5 names (low diversification within sector)
- ❌ Using US sector cycle map without adjusting for Saudi specifics (oil cycle, Vision 2030 policy)

## References

- `references/sector-cycle-map.md` — detailed cycle-to-sector mapping for Saudi context
- `references/sector-comparison-template.md` — fillable template for periodic review
