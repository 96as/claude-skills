# QUICKSTART — Phase 1 Paper Trading in One Page

One-page setup and Day-1 walkthrough for the Tadawul Shariah Trading skill bundle.

## Before you start (one-time, ~15 minutes)

**1. Install all 10 skills on claude.ai**
- Settings → Capabilities / Skills → Upload skill
- Upload each of the 10 `.skill` files from this bundle
- Toggle each one ON

**2. Pick your starting capital**
- Recommended: **SAR 500–1,000**
- Why not less: Tadawul minimum commissions (~SAR 12) make <500 SAR uneconomic even on paper
- Why not more for Phase 1: keeps the simulation tight; you're testing the *loop*, not your portfolio allocation

**3. Decide your broker assumptions**
- Default commission: 0.155%
- Default minimum: SAR 12 per trade
- Default VAT: 15%
- If you already have a broker, use their actual rates

## Day 0 — Initialize the portfolio

Open a new claude.ai chat and say:

> "Initialize my Tadawul paper portfolio with SAR 1,000 starting capital, 0.155% commission, SAR 12 minimum, 15% VAT."

Claude should invoke the `tadawul-paper-portfolio` skill with the `init` command and create a `paper_portfolio.json` file.

Verify by asking:

> "Show my paper portfolio."

You should see a clean portfolio: SAR 1,000 cash, no positions, empty trade history.

## Day 1 morning — Sunday before 10:00 Riyadh time

Say:

> "Run the morning brief for my Tadawul paper portfolio."

Claude should invoke `tadawul-daily-loop` in `morning` mode and:
1. Fetch current TASI level and recent moves
2. Check for any weekend news
3. Report any open positions (none yet on Day 1)
4. Identify leading sectors (1W/1M)
5. Propose 1-3 candidate names from leading sectors — with Shariah status pre-checked

Read the brief. **Don't commit to any trades yet.** The morning brief is a watch list.

## Day 1 mid-session — sometime 10:00–15:10 Riyadh

Say:

> "Run mid-session. If any candidates from the morning brief have triggered, size them and check the risk gate."

Claude should:
1. Re-check TASI direction (has it held?)
2. For each morning candidate: check if the technical trigger has fired on current price
3. For any fired candidates: call `tadawul-position-sizer` → `tadawul-risk-gate` → if PASS, `tadawul-paper-portfolio.record_buy` with a journal entry

Expect that many sessions will have no new entries. That's normal and correct. A setup that doesn't trigger is not a missed trade; it's a disciplined pass.

## Day 1 end-of-day — after 15:10 Riyadh

Say:

> "Run end-of-day."

Claude should:
1. Fetch closing prices
2. Mark-to-market all open positions
3. Update any closed-trade journal outcomes
4. Report daily P/L, realized + unrealized
5. Flag anything needing attention tomorrow

## Daily rhythm from there

| Time | Command | Takes |
|---|---|---|
| Before 10:00 (Riyadh) | "Morning brief" | ~5 min |
| Once or twice between 10:00–15:10 | "Mid-session" | ~3 min each |
| After 15:10 | "End of day" | ~3 min |

**Closed days (Fri/Sat):** you can still ask for review-only mode ("show me where my positions stand") but no new trades.

## Week-close (Thursday after 15:10)

Say:

> "Run end-of-day, then give me a full weekly portfolio review and journal stats."

Claude invokes `tadawul-portfolio-manager` for holistic view and `tadawul-trade-journal.compute_stats` for calibration data.

## What you're tracking

The experiment is not really about the SAR P/L at the end of a week. At 1,000 SAR, the noise overwhelms any signal. The data you're actually collecting:

1. **How many times did Claude correctly identify a Shariah status vs get it wrong / stale?**
2. **Did the risk gate catch bad trades? Did it refuse trades that later would have been winners (false negatives)?**
3. **How many of Claude's "high-conviction" theses actually held up?** (See `references/calibration-guide.md` in the trade journal skill.)
4. **What did it cost?** — your API usage / Cowork usage per day
5. **Where did the loop break?** — data source failures, skills that didn't trigger, edge cases

Keep a side-note somewhere (a plain text file, a notes app) recording these observations as you go. After a week, you'll have enough to decide whether Phase 2 (supervised real trading) is worth the effort.

## Troubleshooting

**Skills don't auto-invoke from my prompt**
→ Name the skill explicitly: *"Use the tadawul-daily-loop skill to run my morning brief."*

**Claude claims it can't access a website**
→ That's fine. The `degraded-mode-handling.md` reference covers this. Expect review-only mode when data sources are down.

**The risk gate refuses every trade**
→ Check the default rules in `references/default-rules.json`. At SAR 1,000 account size, a 15% blue-chip cap = SAR 150 max position. Minimum commission SAR 12 = 8% of that. Position sizes this small hit every cap at once. Either (a) raise capital, (b) relax caps deliberately by editing `risk_rules.json`, or (c) accept that most candidates will be refused — the refusals are themselves informative.

**The portfolio file gets corrupted**
→ Don't let Claude "fix" it silently. Ask: "Show me the raw paper_portfolio.json contents." Fix it manually or start fresh.

## One rule above all

**This is paper trading.** The `tadawul-paper-portfolio` skill cannot place real orders. If you want to transition to real trades, you execute them yourself in your broker's app. The skills can research and draft; you click submit.

When in doubt, abstain. A missed trade is one missed trade. An undisciplined trade corrupts your journal for the rest of the experiment.
