# Saudi Broker Cost Model

Typical retail brokerage fee structure on Tadawul. Users should verify with their specific broker; these are common industry rates.

## Per-side fees (applied on BUY and again on SELL)

| Component | Typical Rate | Notes |
|---|---|---|
| Brokerage commission | 0.105% – 0.155% of gross | Varies by broker and account tier |
| Minimum commission | SAR 10–15 per trade | Usually around SAR 12 |
| Tadawul trading fee | 0.00005 of gross | Exchange fee, very small |
| CMA fee | Small fixed or minimal % | Regulatory levy |
| Edaa (depository) fee | Small | Custody and settlement |
| VAT | 15% | Applied to the *commission* portion only, not gross |

## Simplified calculation used by this skill

```
commission = MAX(min_commission_sar, gross × commission_rate)
vat = commission × vat_rate         # 15% typically
total_fees = commission + vat
```

Round-trip (buy + sell combined) on a trade that breaks even on price:
- Two commissions + two VATs
- Roughly 0.30% – 0.40% of position value

This is why a 100 SAR test account cannot be profitable on single-name trades — the minimum commission alone is 12 SAR × 2 sides = 24 SAR = 24% of capital gone in fees before any price movement.

## Scale effects

| Position size | Commission as % of position |
|---|---|
| SAR 1,000 | 1.55% (hits minimum) |
| SAR 10,000 | 0.155% (at standard rate) |
| SAR 100,000 | 0.105% – 0.155% depending on broker tier |

A round-trip on SAR 1,000 position is ~3.5% in fees; the same trip on SAR 100,000 is ~0.35%. Position size matters for fee efficiency.

## What this model does NOT include

- Overnight financing (not applicable for cash accounts)
- Margin interest (not Shariah-compliant; not part of this skill's scope)
- Currency conversion (not applicable; SAR-only)
- Withdrawal/transfer fees (outside the trade itself)
- Tax — no personal income tax in Saudi Arabia on equity gains for residents; non-residents should check their home rules

## For real-world verification

Users should ask their broker for:
1. Exact commission percentage and minimum
2. Breakdown of other fees (some bundle, some itemize)
3. Whether their tier gives them negotiated rates

Update the `metadata.broker_commission_rate` and `metadata.min_commission_sar` in `paper_portfolio.json` to match your broker's actual rates.
