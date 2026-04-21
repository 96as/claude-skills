# Daily Loop — Phase Details

The daily loop has three run modes. This file explains exactly what each one does and why the sequence matters.

## Why three modes, not one?

Conflating everything into a single "run the loop" command creates decision bleed — you end up evaluating new trades while still processing yesterday's P/L, which degrades both. Splitting into morning / mid-session / end-of-day matches the actual rhythm of a trading day and keeps each decision crisp.

## Morning (pre-session)

**Objective:** decide what *might* be done today; don't commit yet.

**Why this order:**

1. Settle pending cash first — you need accurate available cash before any other check. Anything else leads to cash-math errors.
2. TASI snapshot before anything else — the market regime frames every individual stock decision. A weak TASI means shrink risk appetite across the board.
3. Weekend gap check on Sundays — two non-trading days is enough for news to change a thesis. Always look.
4. Position review before candidate generation — existing positions at risk (hitting stops, thesis broken) should be handled before you add more exposure.
5. Sector rotation before candidates — candidates should come from leading sectors; proposing trades in lagging sectors is swimming against current.
6. Candidates last, and *only as candidates* — committing to entries in the morning brief leads to forcing trades at the open rather than waiting for the setup to confirm.

**What the morning brief is NOT for:**
- It's not a trade list. It's a *watch* list.
- No risk-gate calls here. That's mid-session.
- No trade records written. Portfolio state is unchanged.

## Mid-session

**Objective:** act on real price action, not on the morning plan.

**Why this order:**

1. Re-check TASI first — if regime has flipped since morning, adjust. A TASI reversal can invalidate the whole candidate list.
2. Candidate evaluation: has the technical trigger fired? If no, the candidate stays a candidate. No chasing.
3. For each fired candidate: size → risk gate → record (only if PASS).
4. Open position review: stops and targets checked against live prices. Mechanical exits.

**Critical rule:** mid-session decisions should be as mechanical as possible. The analytical work was done in the morning; mid-session is mostly execution. If you find yourself re-analyzing from scratch, something is wrong — either the morning brief was incomplete, or you're letting noise pull you off plan.

**What the mid-session log is NOT for:**
- New idea generation — if a brand-new setup appears mid-session and wasn't on the morning list, the default should be skip unless conditions are exceptionally clear. FOMO fills are usually losers.
- Rule overrides — if the risk gate REFUSEs, respect it. Don't "adjust" the trade to just barely pass.

## End of day

**Objective:** close the books honestly.

**Why this order:**

1. Fetch closing prices first — you need accurate mark-to-market for everything else.
2. Mark-to-market — now every open position has a current value.
3. Update journal outcomes for closed trades — do this before the daily P/L so you're looking at the raw record, not a summary.
4. Compute daily P/L — now you have the real number.
5. Check daily loss limit — if breached, flag for rule review (deliberate, not reactive).
6. Generate summary.

**Critical rule:** the journal outcome updates must be honest. If a trade hit the stop, mark it as `closed_loser` with thesis invalidated, even if the stock recovered after hours. The trade was a loss at the rules you set.

**What the end-of-day summary is NOT for:**
- Deciding tomorrow's trades — that's tomorrow's morning brief, with tomorrow's fresh data.
- Rationalizing losing trades — the post-mortem section is for learning, not excuses.

## Week-close (Thursdays)

Additionally run after the normal EOD:
- Call `tadawul-portfolio-manager` for a full review
- Compute weekly journal stats
- Re-examine rules against the week's outcomes (deliberate, not reactive)

## Month-close

Additionally:
- Purification ledger update
- Re-verify Shariah status for every held position (quarterly rhythm falls monthly here as a hedge)
- Full performance attribution vs TASI

## Dependencies between phases

```
Morning → Mid-session → EOD
   ↓           ↓          ↓
   journal    journal   journal updates
   (SKIP     (BUY/SELL  (outcome)
    records  decisions)
    for
    candidates
    you chose
    not to buy)
```

Every phase writes to the journal. The journal is the audit trail.
