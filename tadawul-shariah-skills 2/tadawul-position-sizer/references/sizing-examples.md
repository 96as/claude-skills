# Position Sizing — Worked Examples for Tadawul

## Example 1 — Conservative Beginner, Blue-Chip

**Setup:**
- Account: SAR 50,000
- Risk per trade: 0.5% (= SAR 250)
- Stock: 1120 Al Rajhi Bank
- Entry: SAR 95.00
- Stop: SAR 92.00
- Risk per share: SAR 3.00

**Calculation:**
- Raw shares: 250 / 3.00 = 83 shares
- Position value: 83 × 95 = SAR 7,885 (15.8% of account — at cap)
- Cap check (15% blue-chip = SAR 7,500): need to reduce
- Final: 78 shares × 95 = SAR 7,410
- Actual risk: 78 × 3 = SAR 234

✅ Under risk budget, under concentration cap. Good.

---

## Example 2 — Moderate Risk, Mid-Cap

**Setup:**
- Account: SAR 200,000
- Risk per trade: 1% (= SAR 2,000)
- Stock: 4002 Mouwasat Healthcare
- Entry: SAR 92.00
- Stop: SAR 87.00
- Risk per share: SAR 5.00

**Calculation:**
- Raw shares: 2,000 / 5.00 = 400 shares
- Position value: 400 × 92 = SAR 36,800 (18.4% of account)
- Cap check (8% mid-cap = SAR 16,000): MUST reduce
- Final: 173 shares × 92 = SAR 15,916
- Actual risk: 173 × 5 = SAR 865 (0.43% of account)

✅ Position cap dominates. Use less of the risk budget but stay within concentration rules.

---

## Example 3 — Wide Stop on Volatile Stock

**Setup:**
- Account: SAR 100,000
- Risk per trade: 1% (= SAR 1,000)
- Stock: 4321 Real Estate small-cap
- Entry: SAR 35.00
- Stop: SAR 31.50 (10% wide — at the daily limit!)
- Risk per share: SAR 3.50

**Calculation:**
- Raw shares: 1,000 / 3.50 = 285 shares
- Position value: 285 × 35 = SAR 9,975 (10% of account)
- Cap check (5% small-cap = SAR 5,000): MUST reduce significantly
- Final: 142 shares × 35 = SAR 4,970
- Actual risk: 142 × 3.50 = SAR 497

⚠️ **Warning:** Stop at 10% means a "limit-down" day could blow through your stop entirely with no fill. Consider:
- Smaller position (further reduced)
- Tighter stop (but be careful of being stopped on noise)
- Skip the trade entirely if liquidity is a concern

---

## Example 4 — Very Tight Stop, Liquid Name

**Setup:**
- Account: SAR 500,000
- Risk per trade: 1% (= SAR 5,000)
- Stock: 2222 Aramco
- Entry: SAR 28.50
- Stop: SAR 28.00
- Risk per share: SAR 0.50

**Calculation:**
- Raw shares: 5,000 / 0.50 = 10,000 shares
- Position value: 10,000 × 28.50 = SAR 285,000 (57% of account!)
- Cap check (15% blue-chip = SAR 75,000): cap kicks in HARD
- Final: 2,631 shares × 28.50 = SAR 74,983
- Actual risk: 2,631 × 0.50 = SAR 1,316

✅ Tight stops trigger position cap, not risk cap. Use 15% concentration.

⚠️ **Trade-off:** Tight stop = high R/R if it works, but high probability of stop-out on noise. Aramco at SAR 28.50 has typical ATR of SAR 0.40-0.60 — a SAR 0.50 stop is INSIDE one day's normal range. Likely to whip out.

**Better alternative:** Stop at SAR 27.30 (1.5×ATR). Risk per share = SAR 1.20. Risk budget = SAR 5,000. Raw = 4,166 shares = SAR 118,731 (still cap-bound to SAR 75,000 = 2,631 shares). Actual risk = SAR 3,157 (0.63%).

---

## Example 5 — Multi-Position Account Build

**Setup:**
- Account: SAR 100,000
- Strategy: 8 positions, equal-weight 12.5% each
- Cash buffer: 0%

**Position layout (illustrative):**
| Stock | Sector | % | SAR |
|---|---|---|---|
| 1120 Al Rajhi | Banking (Islamic) | 12.5% | 12,500 |
| 2222 Aramco | Energy | 12.5% | 12,500 |
| 7010 STC | Telecom | 12.5% | 12,500 |
| 2280 Almarai | Food | 12.5% | 12,500 |
| 4002 Mouwasat | Healthcare | 12.5% | 12,500 |
| 1211 Maaden | Mining | 12.5% | 12,500 |
| 4190 Jarir | Retail | 12.5% | 12,500 |
| 2010 SABIC | Petrochem | 12.5% | 12,500 |

**Sector concentration check:**
- Energy + Petrochem = 25% ✅ (under 30% cap)
- Banking = 12.5% ✅
- All sectors diversified ✅

**Risk note:** This is equal-weight allocation (passive). For active management, use weights informed by conviction + technical setup quality, but keep concentration caps.

---

## Example 6 — Adding to a Winner (Pyramiding)

**Setup:**
- Original position: 200 shares of 4013 Al Habib at SAR 220
- Stock has run to SAR 245 (+11%)
- You want to add

**Decision framework:**
- Original position size: 200 × 220 = SAR 44,000
- If account is SAR 300,000: original = 14.7% (already near 15% cap for blue-chip)
- Adding more would breach concentration → DON'T add to a position already at cap
- Alternative: trail stop on existing position to lock in profit; deploy capital elsewhere

**General pyramiding rule:** You can add IF:
1. Original thesis still valid
2. You're under concentration cap
3. New stop covers entire (combined) position
4. You're not chasing — entry is at a new logical setup, not just "it's going up"

---

## Example 7 — Stop-Loss Math by ATR

**Setup:**
- Stock: 7010 STC at SAR 40.00
- ATR(14) daily = SAR 0.80
- Choose stop = 2 × ATR = SAR 1.60

So:
- Stop level: 40.00 − 1.60 = SAR 38.40
- This stop is OUTSIDE typical noise (less likely to be whipped out)
- Risk per share: SAR 1.60
- Account SAR 100K, 1% risk = SAR 1,000 budget
- Raw shares: 1,000 / 1.60 = 625 shares
- Position value: 625 × 40 = SAR 25,000 (25%, but blue-chip cap is 15% = SAR 15,000 → 375 shares)
- Final: 375 shares, SAR 15,000 position, SAR 600 actual risk

ATR-based stops adapt to each stock's natural volatility. Highly recommended for short-term trading.

---

## Example 8 — Cost-Aware Sizing

**Setup:**
- Trade size: SAR 10,000
- Broker commission: 0.155% per side
- VAT 15% on commission
- Round-trip cost: 2 × 0.155% × 1.15 = ~0.36%

So:
- Round-trip cost in SAR: 36
- Breakeven: stock must rise 0.36% from entry just to break even
- For a target of 5% gain: net = 4.64%
- For a target of 2% gain: net = 1.64% (significantly eroded)

**Lesson:** For very short-term trades (1-3% targets), costs matter A LOT. Make sure your edge is bigger than your cost drag.

---

## Quick Reference: Per-Trade Risk Budget Table

| Account (SAR) | 0.5% risk | 1% risk | 2% risk |
|---|---|---|---|
| 25,000 | 125 | 250 | 500 |
| 50,000 | 250 | 500 | 1,000 |
| 100,000 | 500 | 1,000 | 2,000 |
| 200,000 | 1,000 | 2,000 | 4,000 |
| 500,000 | 2,500 | 5,000 | 10,000 |
| 1,000,000 | 5,000 | 10,000 | 20,000 |
