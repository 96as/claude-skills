---
name: tadawul-technical-analyst
description: Use this skill when the user asks for technical analysis of any Saudi (Tadawul) stock or the TASI index — including chart pattern questions, support/resistance levels, indicator readings, breakout assessments, or short-term entry/stop/target levels. Triggers on uploaded chart screenshots, "is X breaking out?", "where's support for Y?", "what does this chart show?". Adapts universal TA principles to Saudi-specific mechanics: Sun-Thu trading week (10:00-15:10 Riyadh time), T+2 settlement, ±10% Main Market daily limit, ±30% Nomu limit, weekend gap risk. Always presents probability-weighted scenarios with explicit triggers and invalidations — never single-point predictions.
---

# Tadawul Technical Analyst

## Overview

Pure technical analysis (price + volume) for Saudi-listed equities, ETFs, and the TASI index. Designed to work with chart screenshots from TradingView, broker apps, or Saudi Exchange. Generates probability-weighted scenarios with specific trigger levels — never single-point predictions.

## Required live data

If the user provides a chart screenshot, work from it. If not, fetch a current chart:

```
web_fetch: https://www.tradingview.com/symbols/TADAWUL-<TICKER>/
```

For the TASI index:
```
web_fetch: https://www.tradingview.com/symbols/TADAWUL-TASI/
```

For current price and volume context:
```
web_search query: "<ticker> tadawul price today"
```

If only stale charts are available and the user has not provided a screenshot, tell them so and ask whether to proceed with structural levels (which change slowly) versus tactical levels (which require fresh data).

## Workflow

### Step 1 — Confirm What You're Looking At
- Ticker, name, timeframe (daily / weekly / monthly)
- If image provided: identify the chart elements visible (price action, indicators, volume)
- If no image: search for the latest TradingView snapshot (`TADAWUL:[ticker]`)

### Step 2 — Multi-Timeframe Context

Always assess at least two timeframes:
- **Weekly** — Primary trend (the "tide")
- **Daily** — Tactical (the "waves")
- For short-term traders: add **4-hour or hourly** if needed (the "ripples")

**Trend hierarchy rule:** Trade WITH the higher timeframe trend; counter-trend trades are higher risk and should be smaller.

### Step 3 — Identify the Trend

**1. Moving Averages**
- 20-day MA = short-term trend
- 50-day MA = intermediate trend
- 200-day MA = long-term trend
- Stack alignment: 20 > 50 > 200 (all rising) = strong uptrend; reverse = strong downtrend

**2. Higher Highs / Higher Lows**
- Uptrend: HH + HL pattern
- Downtrend: LH + LL pattern
- Sideways: roughly equal highs/lows

**3. Trendlines**
- Connect at least 3 touchpoints (2 is not enough)
- Steeper trendlines break sooner

### Step 4 — Identify Key Levels

**Support & Resistance:**
- Recent swing highs/lows (most relevant)
- Round numbers (SAR 50, 100 — psychological)
- Volume nodes (price levels with high volume bars)
- 52-week high/low
- All-time high/low
- Key MAs as dynamic support/resistance

**Saudi-specific levels:**
- IPO price (often acts as long-term support/resistance)
- Recent gap fills (Sunday weekend gaps especially)
- ±10% daily limit levels (only relevant intra-day)

### Step 5 — Patterns

**Continuation Patterns** (trend likely to continue):
- Bull/Bear Flag, Pennant, Triangle (Symmetrical, Ascending, Descending)
- Cup and Handle, Rectangle/Consolidation

**Reversal Patterns:**
- Head and Shoulders / Inverse H&S
- Double Top / Double Bottom, Triple Top / Triple Bottom
- Rounding Top / Bottom
- Wedge (Rising = bearish; Falling = bullish in downtrend)

**Candlestick Patterns** (especially at key levels):
- Doji, Hammer, Shooting Star
- Engulfing (Bullish/Bearish), Morning Star / Evening Star

### Step 6 — Indicators

Pick complementary ones; don't pile on:

**Momentum:**
- **RSI(14):** Overbought > 70, Oversold < 30. Look for divergences.
- **MACD:** Crossovers, divergences, histogram momentum

**Volume:**
- **Volume bars** — Confirmation of moves (breakout = volume should expand)
- **OBV** (On-Balance Volume) — Cumulative; confirms or diverges from price
- **Volume Profile** — Where price has spent most time

**Volatility:**
- **Bollinger Bands** (20, 2σ): Squeeze = low vol = breakout pending; Walk the band = strong trend
- **ATR(14):** For stop-loss sizing

### Step 7 — Generate Scenarios

Always present **at least 2 scenarios with probabilities** — never a single forecast.

```
SCENARIO A — [Bullish/Bearish/Continuation]: Probability ~X%
  Trigger: Close above/below SAR Y on volume > Z%
  Target 1: SAR A
  Target 2: SAR B
  Invalidation: Close below/above SAR C
  
SCENARIO B — [Alternative]: Probability ~X%
  Trigger: ...
  Target: ...
  Invalidation: ...
```

Probabilities should sum to ~100%. Adjust based on confluence of signals.

### Step 8 — Saudi-Specific Considerations

**1. ±10% Daily Limit (Main Market)**
- A breakout that hits the +10% limit is paused. The breakout may be artificial if forced by retail buying.
- A "limit-up" close on heavy volume could continue next session — but also could gap down on profit-taking.
- Nomu Parallel Market uses ±30% limits.

**2. Weekend Gap Risk (Thursday close → Sunday open)**
- Tadawul is closed Friday and Saturday.
- Major news on Fri/Sat (oil moves, US/China events, geopolitical) can cause Sunday gaps.
- Avoid full position size right before close on Thursday if known weekend risk.
- Watch oil prices over the weekend if holding energy stocks.

**3. T+2 Settlement**
- Cash from a sale is not available for a new buy until T+2.
- Avoid being capital-locked if planning rapid rotation.

**4. Liquidity / Bid-Ask**
- For mid/small-caps with thin volume, the bid-ask spread can be 1-2% — eats your edge.
- Use **limit orders** in low-liquidity names; never market orders unless full-blown blue-chip.

**5. Sector Beta to TASI**
- TASI is bank/petchem heavy. If TASI is breaking down, sector ETFs and even uncorrelated names can be dragged.
- Always glance at TASI before entering an individual stock.

## Reporting Template

```markdown
# Technical Analysis: [Ticker] [Name] — [Timeframe]

**Date:** YYYY-MM-DD | **Current Price:** SAR X.XX

## Trend
- **Weekly:** [Up / Down / Sideways] — describe
- **Daily:** [Up / Down / Sideways] — describe
- **MA Stack:** [20/50/200 alignment]

## Key Levels
- **Resistance:** SAR X (description), SAR Y (description)
- **Support:** SAR A (description), SAR B (description)

## Patterns Identified
- [Pattern name] — [stage of pattern, neckline/trigger]

## Indicators
- **RSI(14):** X — [overbought/neutral/oversold + any divergence]
- **MACD:** [bullish/bearish + crossover status]
- **Volume:** [expanding / contracting / accumulation / distribution]
- **Bollinger:** [squeeze / walking band / mean reversion zone]

## Scenarios

**Scenario A — [name] (~X% probability)**
- Trigger: [specific level + condition]
- Target 1: SAR X
- Target 2: SAR Y
- Invalidation: [specific level]

**Scenario B — [name] (~Y% probability)**
- Trigger: ...
- Target: ...
- Invalidation: ...

## Saudi Context
- TASI direction: [aligned / divergent]
- Sector strength: [...]
- Liquidity check: [avg daily volume sufficient for your position?]
- Weekend / event risk: [...]

## Risk Notes
- ATR(14): SAR X (suggests stop ~X% below entry)
- Daily limit awareness: ±10%
- Suggested position sizing reference: see `tadawul-position-sizer` skill

---
**Disclaimer:** Technical analysis estimates probability, not certainty. Combine with fundamentals and Shariah screening before any decision.
```

## Anti-Patterns

- ❌ Single-point price predictions ("it will go to SAR 50") — always probability-weighted scenarios
- ❌ Cherry-picking one indicator that "agrees" with you
- ❌ Ignoring volume on a "breakout"
- ❌ Trading against the higher timeframe trend without sizing down
- ❌ Using too many indicators (analysis paralysis)
- ❌ Forgetting the weekend gap risk in Saudi stocks
- ❌ Drawing trendlines with only 2 touchpoints (need 3+)

## References

- `references/technical-framework-tadawul.md` — full TA framework adapted for Saudi market
- `assets/saudi-analysis-template.md` — fillable analysis template
