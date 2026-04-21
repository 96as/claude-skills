# Cap-Class Criteria for Position Sizing

The risk gate applies different position caps based on stock classification. Here's how to classify.

## Blue-chip (15% max position)

A stock is blue-chip for this skill's purposes if **all** of these hold:

- Member of the TASI top 30 by market cap, OR member of the MSCI Tadawul 30 (MT30) index
- Average daily traded value > SAR 50M over the past 90 days
- Free float > 20%
- Listed for > 2 years

Examples as of 2026 (verify current status before relying): 2222 (Aramco), 1120 (Al Rajhi), 2010 (SABIC), 7010 (STC), 1180 (SNB), 1211 (Maaden), 2280 (Almarai), 4013 (Sulaiman Al Habib), 2082 (ACWA Power), 1140 (AlBilad), 1150 (AlInma).

## Mid-cap (8% max position)

- Market cap SAR 5B – 25B, AND
- Average daily traded value between SAR 10M and SAR 50M
- Listed on Main Market (not Nomu)

Many second-tier healthcare, petrochem, retail, and telecom names fall here.

## Small-cap (5% max position)

- Market cap SAR 1B – 5B, AND
- Average daily traded value < SAR 10M
- Listed on Main Market

Harder to exit cleanly. Spreads widen.

## Nomu / Parallel Market (3% max position)

- Any security listed on Nomu regardless of market cap
- Daily price limit is ±30% (vs ±10% on Main Market) — wider gaps
- Typically much lower liquidity

Nomu names carry meaningfully higher risk, which is why the cap is strict.

## Edge cases

**Thinly traded main-market name:** If the stock is Main Market but avg daily value < SAR 2M, treat it as small-cap regardless of market cap. Liquidity is the binding constraint.

**Recent IPO:** If listed < 6 months ago, apply mid-cap caps at most, regardless of market cap. Price discovery still in progress.

**Company under CMA suspension or investigation:** The risk gate should refuse new positions entirely until status clears.

## How the skill decides

In order:
1. If Nomu → small-cap Nomu cap (3%).
2. Else if avg daily value < SAR 2M → small-cap cap (5%) regardless.
3. Else if market cap < SAR 1B → small-cap (5%).
4. Else if market cap < SAR 5B OR daily value < SAR 10M → small-cap (5%).
5. Else if market cap < SAR 25B OR daily value < SAR 50M → mid-cap (8%).
6. Else → blue-chip (15%).

When in doubt, the skill picks the *more restrictive* cap. Being safe costs you opportunity; being unsafe costs you the account.

## Fetching this data

```
web_fetch: https://www.argaam.com/en/tadawul/tasi/<company-slug>
```

Argaam company pages show market cap, avg daily volume, and listing market (Main / Nomu). Use these to classify before any BUY gets proposed.
