# Tadawul Data Sources — Complete Guide

## Free Sources (No Account Needed)

### Saudi Exchange Official (saudiexchange.sa)
- **What:** Official prices, announcements, IPO docs, disclosures, indices
- **Pros:** Authoritative, free, real-time delayed quotes
- **Cons:** Heavy site, sometimes slow, less analytical depth
- **Use for:** Official announcements, dividend declarations, board changes
- **URL pattern:** `https://www.saudiexchange.sa/wps/portal/saudiexchange/ourmarkets/main-market-watch/companies-details/...`

### Argaam (argaam.com)
- **What:** News, financials, fundamentals, Shariah list, analyst views
- **Pros:** Free, English+Arabic, great for daily analysis, has Shariah list
- **Cons:** Sometimes ads-heavy, occasional outdated data
- **Use for:** Daily research, Shariah verification, financial summaries
- **Key URLs:**
  - Company profiles: `argaam.com/en/tadawul/tasi/[ticker]`
  - Shariah list: `argaam.com/en/company/shariahcompanies`
  - Sector data: `argaam.com/en/sector`

### Mubasher (mubasher.info)
- **What:** Live prices, news, charts
- **Pros:** Free, Arabic and English
- **Use for:** Real-time price tracking
- **URL:** `english.mubasher.info/markets/TADAWUL`

### Yahoo Finance (with .SR suffix)
- **What:** Price history, basic financials
- **Pros:** Free, programmatic access via yfinance Python lib
- **Cons:** Limited fundamental data for Saudi stocks
- **Use for:** Historical price downloads
- **Format:** `2222.SR` (Aramco), `1120.SR` (Al Rajhi)

### TradingView (tradingview.com)
- **What:** Excellent charts with all indicators, social analysis
- **Pros:** Best charting, free tier sufficient
- **Cons:** Some features paid; no Saudi fundamentals
- **Use for:** Technical analysis, chart screenshots
- **Format:** `TADAWUL:2222`

### Investing.com
- **What:** Prices, news, analyst views
- **Pros:** Free, multilingual
- **Use for:** Cross-verification

## Paid / Premium Sources

### Bloomberg / Refinitiv
- Institutional pricing; expensive
- Best data quality for serious analysis

### TASI Pro (tasipro.com)
- Saudi-focused premium data and signals
- Reasonable subscription fee

### TradingView Premium
- Real-time Tadawul data, more chart features
- ~$15-60/month

### Argaam Pro
- Professional data, models, alerts
- ~SAR 500-1000/year

## Broker Platforms (For Customers Only)

These give the best Saudi data but require an account:

| Broker | Strengths |
|---|---|
| **Al Rajhi Tadawul** | Embedded Shariah list, deep fundamentals, popular |
| **SNB Capital** | Strong research reports |
| **Riyad Capital** | Good charting tools |
| **AlJazira Capital** | Strong Islamic-focused research |
| **AlInma Investment** | Shariah-focused interface |
| **AlBilad Capital** | Islamic-focused |
| **Derayah Financial** | Modern interface, lower fees |

## Data Source Decision Matrix

| Need | Best Source |
|---|---|
| Live price | Saudi Exchange or your broker |
| Shariah status | Argaam list → Almaqasid for verification |
| Fundamentals | Argaam → Annual Report on company IR page |
| News | Argaam + Saudi Exchange announcements |
| Charts | TradingView |
| Historical price download | Yahoo Finance (.SR) |
| Analyst targets | Argaam + Reuters |
| IPO documents | Saudi Exchange |
| Dividend history | Argaam company profile |
| Earnings calendar | Saudi Exchange "Disclosures" |

## Limitations to Know

1. **Tadawul data is often delayed 15 minutes** on free sources. Real-time requires a paid feed or broker.
2. **Foreign ownership data** is hard to get freely; brokers usually have it.
3. **English coverage is limited** for smaller/Nomu stocks — Arabic sources fill the gap.
4. **Not all companies file English annual reports.** Use Arabic versions if needed (browser translation works).
5. **Insider trading data** (board member buys/sells) is published on Saudi Exchange under disclosures but is not always centralized.

## Workflow Suggestion

For a typical Tadawul research session:
1. Check **Argaam Shariah list** for status
2. Open **Argaam company page** for snapshot + financials
3. Open **TradingView** for chart
4. Check **Saudi Exchange disclosures** for latest announcements
5. Cross-verify with one news source (Argaam News, Reuters, Bloomberg)

Total time per stock: 10-15 minutes for a quick read; 45-60 minutes for a deep dive.
