---
name: tadawul-stock-analyzer
description: Use this skill when the user asks for analysis of any Saudi (Tadawul) listed stock — e.g., "analyze Aramco", "give me a report on STC", "what do you think about ticker 2222", "research Almarai for me". Runs end-to-end research: mandatory Shariah compliance check first (refuses to fully analyze non-compliant stocks unless explicitly overridden), then fundamentals from Argaam, then recent news and catalysts, then bull/bear case. Always fetches live price and recent news rather than relying on memorized data. Outputs a structured report with watch levels but never buy/sell recommendations.
---

# Tadawul Stock Analyzer

## Overview

End-to-end research workflow for Saudi Exchange (Tadawul) stocks. Designed for an investor who requires Shariah compliance as a non-negotiable filter. The skill always runs the compliance check FIRST and only proceeds with full analysis if the stock passes (or if the user explicitly overrides).

## Required live data (fetch every time)

Stock prices, ratios, and news change daily. Never report from training-data memory. For every analysis, run at minimum:

```
web_search query: "<ticker> tadawul argaam"
web_fetch: https://www.argaam.com/en/tadawul/tasi/<company-slug>
web_search query: "<company name> news <current month> <current year>"
```

For real-time price, also try:
```
web_fetch: https://www.tradingview.com/symbols/TADAWUL-<TICKER>/
web_search query: "<ticker>.SR yahoo finance" (then fetch the Yahoo URL)
```

If Argaam's English page returns mostly navigation chrome (it sometimes does), try the Saudi Exchange directly:
```
web_fetch: https://www.saudiexchange.sa/wps/portal/saudiexchange/ourmarkets/main-market-watch/main-market-overview/main-market-detailed
```

If after three searches you cannot get a current price or recent earnings, tell the user explicitly and offer to proceed with structural analysis only.

## Workflow

### Step 0 — Identify the Ticker
- Saudi tickers are 4-digit numbers (e.g., 2222 = Aramco, 1120 = Al Rajhi).
- If the user gives a name only ("Aramco"), confirm the ticker.
- TradingView format: `TADAWUL:2222`. Yahoo Finance format: `2222.SR`.

### Step 1 — Shariah Compliance Check (MANDATORY)

**Use the `shariah-compliance-screener` skill or apply its logic.** Sequence:
1. web_fetch the Argaam Shariah list.
2. Cross-check Almaqasid if borderline.
3. Report status: ✅ Compliant / ✅ Compliant w/ purification / ⚠️ Doubtful / ❌ Non-compliant.

**If the verdict is ❌ NON-COMPLIANT:**
> Stop here. Inform the user clearly:
> "This stock is not Shariah-compliant under [board name]. I won't proceed with a full buy-side analysis. If you'd like, I can: (a) explain *why* it's non-compliant, (b) suggest compliant alternatives in the same sector, or (c) give a brief structural overview without buy-side framing. Let me know which."

**If verdict is ⚠️ DOUBTFUL:** Inform the user, give the specific borderline ratio, and ask if they want to proceed.

**If verdict is ✅ COMPLIANT:** Proceed to Step 2.

### Step 2 — Gather Fundamental Data

Search and fetch from these sources (in priority order):

1. **Argaam company profile** — `argaam.com/en/tadawul/tasi/[company-slug]` (financials, news, profile)
2. **Saudi Exchange company page** — `saudiexchange.sa` (official disclosures, IPO docs, dividend announcements)
3. **TradingView** — `tradingview.com/symbols/TADAWUL-[TICKER]/` (chart, key metrics)
4. **Company investor relations page** — for annual reports, earnings presentations
5. **Reuters / Bloomberg** — for latest news and analyst sentiment

**Key data to collect:**

| Category | Data Points |
|---|---|
| **Identity** | Sector, sub-sector, market cap, free float, FOL (foreign ownership limit) |
| **Price** | Current, 52-week range, YTD performance, vs TASI |
| **Valuation** | P/E (TTM and forward), P/B, EV/EBITDA, dividend yield |
| **Growth** | Revenue growth (YoY, 3-yr CAGR), EPS growth, ROE |
| **Quality** | Net margin, ROIC, debt/equity, current ratio |
| **Distribution** | Dividend history, payout ratio, last ex-date |
| **Recent** | Latest earnings vs estimates, upcoming events |

### Step 3 — Apply Saudi-Specific Context

Saudi market has unique dynamics. Always consider:

- **TASI correlation** — How does the stock move with TASI overall?
- **Oil correlation** — Energy/petrochem stocks especially track oil prices
- **Vision 2030 exposure** — Companies aligned with NEOM, Red Sea, tourism, mining benefit from sovereign spending
- **Government ownership** — PIF stake, GOSI stake, government direct holdings (often supports prices)
- **SAR/USD peg** — Reduces FX risk for foreign earnings (riyal pegged to dollar at 3.75)
- **Trading days** — Tadawul trades **Sunday–Thursday, 10:00–15:10 Riyadh time** (closed Fri/Sat). News from Thursday close to Sunday open can create gaps.
- **Settlement** — T+2 settlement (in effect since April 2017)
- **Daily price limits** — ±10% on Main Market stocks (±20% on first day for IPOs); ±30% on Nomu Parallel Market
- **Foreign access** — As of February 1, 2026, the QFI framework was abolished; all foreign investors can now invest directly on the Main Market

### Step 4 — Recent News & Catalysts

Search for recent developments (last 30-60 days):
- Earnings releases
- Dividend declarations
- M&A activity
- Regulatory changes
- Sector-specific news (oil prices for energy, government spending plans, etc.)
- Major contracts or project wins

### Step 5 — Generate the Report

Use this structured template:

```markdown
# 📊 [Company Name] ([Ticker]) — Tadawul Analysis

**Date:** YYYY-MM-DD | **Sector:** [...] | **Market Cap:** SAR X bn

---

## ✅ Shariah Status: [Compliant / Compliant w/ Purification / etc.]
- **Source:** [Argaam list date / Almaqasid]
- **Purification rate:** X% of dividends → SAR X per SAR 1,000 dividend
- **Last verified:** YYYY-MM-DD (from web fetch this session)

---

## 1. Snapshot
| Metric | Value |
|---|---|
| Current Price | SAR X.XX |
| 52-Week Range | SAR X – SAR Y |
| YTD Performance | ±X% (vs TASI: ±Y%) |
| P/E (TTM) | X.X |
| P/B | X.X |
| Dividend Yield | X.X% |
| Avg Daily Volume | X shares |
| Free Float | X% |

## 2. Business Overview
[2-3 sentence description: what they do, market position, key segments]

## 3. Financial Health
- **Revenue trend:** [X-yr direction, latest YoY]
- **Profitability:** [Net margin trend, ROE, ROIC]
- **Balance sheet:** [Debt/equity, cash position, working capital]
- **Cash flow:** [Operating CF, capex, free cash flow trend]

## 4. Recent Performance & Catalysts
- **Latest earnings:** [Beat/miss, key takeaways]
- **Recent news:** [Bullet 2-3 items from last 60 days]
- **Upcoming events:** [Earnings date, dividend, AGM]

## 5. Saudi-Specific Context
- **TASI correlation:** [High/Medium/Low]
- **Vision 2030 angle:** [Direct beneficiary / Neutral / N/A]
- **Government / PIF stake:** [If significant]
- **Sector positioning:** [Leader / Mid-tier / Challenger]

## 6. Bull Case (3 reasons to be positive)
1. ...
2. ...
3. ...

## 7. Bear Case (3 reasons to be cautious)
1. ...
2. ...
3. ...

## 8. Suggested Watch Levels (for short-term consideration)
> ⚠️ Not a buy/sell recommendation. For your own decision-making.

- **Support:** SAR X (recent low / 200-day MA / etc.)
- **Resistance:** SAR Y (recent high / round number / 52-wk high)
- **Stop-loss reference:** SAR Z (X% below entry, depending on your risk model)

## 9. Open Questions / Things to Research Further
- ...

---

**Sources:** [list URLs actually fetched this session]
**Disclaimer:** This is research, not financial advice. Verify Shariah status with your preferred board before investing. Past performance does not guarantee future results.
```

## Quick Tickers Reference

Frequently-asked Tadawul tickers (verify Shariah status fresh each time):

| Ticker | Name | Sector |
|---|---|---|
| 2222 | Saudi Aramco | Energy |
| 1120 | Al Rajhi Bank | Banking (Islamic) |
| 1140 | AlBilad Bank | Banking (Islamic) |
| 1150 | AlInma Bank | Banking (Islamic) |
| 2010 | SABIC | Petrochemicals |
| 7010 | STC | Telecom |
| 7020 | Mobily (Etihad Etisalat) | Telecom |
| 7030 | Zain KSA | Telecom |
| 2280 | Almarai | Food/Dairy |
| 2050 | Savola | Food |
| 2350 | Saudi Kayan | Petrochemicals |
| 2380 | Petro Rabigh | Petrochemicals |
| 4002 | Mouwasat | Healthcare |
| 4013 | Dr. Sulaiman Al Habib | Healthcare |
| 4004 | Dallah Healthcare | Healthcare |
| 4321 | Cenomi Centers | Real Estate |
| 4300 | Dar Al Arkan | Real Estate |
| 1211 | Maaden | Mining |
| 2030 | SARCO | Refining |
| 2082 | ACWA Power | Utilities/Renewables |
| 8230 | Al Rajhi Takaful | Insurance (Takaful) |
| 4190 | Jarir Marketing | Retail |
| 7203 | ELM | Software & Services |

## Anti-Patterns

- ❌ Do NOT skip the Shariah check, even if the stock is "obviously" compliant. Status changes quarterly.
- ❌ Do NOT use US conventions (no FOMC, no SEC 10-K filings — use Tadawul disclosures).
- ❌ Do NOT cite stale price data — always fetch live or near-live.
- ❌ Do NOT make buy/sell recommendations. Provide analysis; let the user decide.
- ❌ Do NOT ignore the daily ±10% price limit when discussing risk.
- ❌ Do NOT use US-listed Saudi proxies (KSA ETF, FLSA) when the user wants to trade on Tadawul directly — they're different products.
- ❌ Do NOT assume foreign-investor restrictions still apply — QFI framework was abolished Feb 2026.

## References

- `references/tadawul-data-sources.md` — detailed guide to each Saudi data source
- `references/saudi-market-conventions.md` — trading hours, settlement, price limits
- `references/saudi-fundamental-metrics.md` — what each ratio means in Saudi context
