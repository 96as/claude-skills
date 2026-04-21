# Using the Journal to Improve Calibration

## What "calibration" means here

If you say a trade has a 70% chance of hitting target 1, and you look back at 20 trades where you said 70%, and 14 of them hit target 1 — that's 70%. You're calibrated.

If only 8 of them hit — 40% — you're systematically overconfident. That gap is information. It tells you your confident theses are less reliable than you believe, and you should either size those trades smaller or be more skeptical of your own conviction.

## The core practice

Every trade records a probability in the thesis. Every closed trade records an outcome. Periodically (every 10-20 closed trades), you bucket by probability band and check hit rates.

## Calibration table to build

After enough trades:

| Stated probability | # of trades | Actual hit rate | Gap |
|---|---|---|---|
| >70% confident | 15 | 47% | -23 pp (overconfident) |
| 50-70% confident | 22 | 55% | +5 pp (well calibrated) |
| <50% confident | 8 | 38% | -12 pp (slightly overconfident) |

Gaps > 10 percentage points are worth acting on.

## Common calibration failures

### 1. "High conviction" bias
Trades you're excited about hit less often than you think. The excitement itself is not predictive information — it might even be anti-predictive because it causes you to skip critical checks.

Fix: treat excitement as a *warning sign* to re-run the risk gate with stricter thresholds, not as validation.

### 2. Recency weight
Yesterday's winner's style bleeds into today's analysis. If a breakout worked last week, you see breakouts everywhere this week.

Fix: in the thesis drivers, separate "pattern I see" from "pattern I'd see if this were cold data."

### 3. Rationalization in post-mortem
"The thesis held up but the market was wrong" is a red flag. The market is never wrong; your thesis predicted it would do X and it did Y, so the thesis was wrong about something.

Fix: require the post-mortem to name one specific thing you got wrong, even on winners. Winners can teach you too — you might have been right for the wrong reason.

### 4. Confirmation search in the journal
When reviewing, don't filter for the trades that support the lesson you want to learn. Sample randomly.

## When to change your rules based on journal data

If after 30+ trades:

- Your win rate on "breakout + volume confirmation" trades is >55% → keep that pattern, maybe size those up slightly
- Your win rate on "breakout on average volume" is <40% → stop taking those trades
- Your Shariah-doubtful trades have worse outcomes → make the rule `doubtful_warn_only: false` (block them)
- Your holding period for winners is much longer than for losers → you're cutting winners early; trail stops more

Change the risk rules file deliberately, not trade by trade.

## What 30 trades actually tells you

Statistically, 30 trades is not a lot. A 50% win-rate trader could have a 30-trade run of 60% or 40% by chance. Use journal data directionally, not as proof.

The thesis quality review is more reliable than the win-rate review at small sample sizes. You can often tell after 10 trades whether your reasoning is rigorous or hand-wavy, even if the market hasn't given you enough data to confirm whether rigorous reasoning wins.
