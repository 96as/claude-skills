# Technical Analysis Framework — Adapted for Tadawul

## Core Principle

Markets reflect everything known. Price action and volume reveal the balance of supply and demand. Patterns repeat because human psychology repeats. This is true on any market — but **execution constraints differ**, and Saudi has unique ones.

## The Three Pillars

### 1. Trend
"The trend is your friend until it bends."

**How to define a trend:**
- Visual: connect highs and lows
- Mathematical: 50/200 day MAs
- Rule: HH+HL = uptrend; LH+LL = downtrend

**Saudi note:** TASI tends to trend more than US indices because of less arbitrage and more retail participation. Trends, when they form, can persist longer.

### 2. Momentum
"Momentum precedes price."

- RSI divergences often precede reversals by days/weeks
- MACD histogram momentum is a leading indicator for the underlying MACD crossover
- Volume surges precede price acceleration

### 3. Volume
"Price tells you what; volume tells you why."

- Breakout without volume = suspicious
- Trend with declining volume = exhausting
- Reversal with massive volume = institutional involvement (or capitulation)

## Patterns — Detailed Reference

### Continuation Patterns

**Bull Flag**
- Setup: Sharp up move ("flagpole") → tight downward consolidation ("flag")
- Trigger: Break above flag's upper trendline
- Target: Flagpole height projected from breakout
- Stop: Below flag low
- Time: Flag should resolve within ~1-3 weeks

**Cup and Handle**
- Setup: Rounded bottom (cup) over months → small pullback (handle)
- Trigger: Break above cup's left rim
- Target: Cup depth projected upward
- Saudi adaptation: Common in mid-cap Tadawul names that recover from crisis lows

**Triangle (Symmetrical, Ascending, Descending)**
- Ascending = bullish bias (flat top, rising bottoms)
- Descending = bearish bias (flat bottom, falling tops)
- Symmetrical = neutral until breakout
- Trade the breakout direction, not the pattern

### Reversal Patterns

**Head & Shoulders (top)**
- Three peaks: middle is highest
- Neckline: connect the two valleys between peaks
- Trigger: Close below neckline
- Target: Distance from head to neckline, projected down
- Confirmation: Volume should be lower on right shoulder than left

**Double Bottom**
- Two distinct lows at roughly same price
- Trigger: Close above the high between the two lows
- Target: Distance from bottoms to high, projected up

### Saudi-Specific Pattern Notes

- **The Sunday Gap Pattern:** Watch for stocks that gap on Sunday open due to weekend news — often see "gap fill" within 2-3 days, especially on retail-heavy names
- **The Limit-Up Reversal:** A stock that hits +10% limit on light volume often reverses next day; on heavy volume often continues
- **Earnings Season Tightening:** Many Saudi stocks tighten into earnings (consolidate) and break out post-earnings on Sunday open

## Indicators — When to Use Each

### Trend Indicators
- **Moving Averages (20, 50, 200)** — Always show
- **ADX** — Use to assess trend strength (>25 = trending; <20 = ranging)

### Momentum Indicators
- **RSI** — Default. Look for divergences and 70/30 extremes.
- **Stochastic** — Better than RSI in ranging markets
- **MACD** — Best for trending markets

### Volume Indicators
- **OBV (On-Balance Volume)** — Cumulative; divergence from price is meaningful
- **VWAP** — Especially useful intraday
- **Volume Profile** — Shows where most trading happened

### Volatility Indicators
- **Bollinger Bands** — Squeeze (bands narrow) → breakout coming
- **ATR** — Use for stop-loss sizing (e.g., stop = 2×ATR below entry)

### Avoid Overload
Pick **3-4 indicators max**. Common useful combo:
1. 20/50/200 MAs (trend)
2. RSI (momentum)
3. Volume bars (confirmation)
4. ATR (sizing)

## Confluence — The Key Concept

A signal is stronger when MULTIPLE independent factors agree.

**Strong signal example:**
- Price breaking above a 4-month consolidation (level)
- On 2x average volume (volume)
- With RSI breaking out of 50 (momentum)
- While TASI is also trending up (market context)
- And the 50-day MA is rising (trend)

**Weak signal example:**
- Price tags resistance once on low volume

## Risk-Reward & Position Sizing

Always before entering:
1. Define **entry**, **stop**, and **target** in advance
2. Calculate R/R ratio: should be at least **1.5:1**, preferably **2:1+**
3. Use the `tadawul-position-sizer` skill to size

## Multi-Timeframe Approach

A simple, effective routine:
1. **Weekly chart** — Define the primary trend
2. **Daily chart** — Find the setup
3. **Hourly chart** — Time the entry

Trade in the direction of the higher timeframe. Counter-trend trades are valid but should be smaller and have tighter stops.

## Common Mistakes

1. **Overfitting** — Drawing patterns to fit your bias. Be skeptical of yourself.
2. **Indicator stacking** — More indicators ≠ more accuracy
3. **Ignoring volume** — Price moves without volume are weak
4. **Counter-trend trades without sizing down**
5. **No stop-loss** — Always know your invalidation level BEFORE entry
6. **Moving stops away from price** — Stops are your discipline; honor them
7. **Chasing breakouts already 5%+ extended** — Wait for pullback or retest

## Saudi-Specific Mistakes to Avoid

1. **Buying limit-up close on retail euphoria** — Often gives back next day
2. **Ignoring TASI direction** — TASI sets the tone for 60%+ of stocks
3. **Trading thin small-caps with market orders** — Slippage destroys edge
4. **Holding leveraged positions over weekend** — Gap risk is real
5. **Trading right at open Sunday before price discovery completes** — Wait 15-30 min for spread to settle

## Resources Worth Knowing

- **Investopedia** — Free, comprehensive TA encyclopedia
- **Murphy's "Technical Analysis of the Financial Markets"** — Bible of TA
- **Edwards & Magee "Technical Analysis of Stock Trends"** — Classic patterns
- **TradingView education** — Free video lessons
- **StockCharts ChartSchool** — Free, well-organized
