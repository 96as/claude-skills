# Tadawul Shariah Trading Skills for Claude (v2)

A toolkit of **10 Claude Skills** for **Shariah-compliant short-term trading and investing on the Saudi Tadawul exchange**, built around a **paper-trading experimental loop**.

This v2 bundle splits into two groups:

- **Analytical skills** (6) — research, screening, analysis. Useful for any Tadawul investor.
- **Operational skills** (4, new in v2) — manage the paper-trading experiment: portfolio state, decision journal, risk gate, daily loop.

Adapted from the open-source [tradermonty/claude-trading-skills](https://github.com/tradermonty/claude-trading-skills) project and rebuilt around:
- **AAOIFI Shariah Standard No. 21** (Saudi/Tadawul convention)
- **Saudi market mechanics** — current tick-size matrix (effective June 29, 2025), T+2 settlement, ±10% Main Market / ±30% Nomu daily limits, 10:00-15:10 Riyadh trading hours, Sun-Thu week
- **SAR currency** throughout
- **Argaam / Almaqasid / Saudi Exchange** as primary data sources
- **Live web fetching** — every factual claim about prices, Shariah status, or market data is fetched fresh, never recited from training-data memory

---

## 📦 The 10 Skills

### Analytical (research, screening, analysis)

| # | Skill | What it does |
|---|---|---|
| 1 | **shariah-compliance-screener** | Verifies Shariah compliance under AAOIFI SS21 via live Argaam/Almaqasid fetch. Calculates dividend purification. Always run first. |
| 2 | **tadawul-stock-analyzer** | End-to-end stock research: Shariah check → fundamentals → news → bull/bear cases. Refuses non-compliant stocks. |
| 3 | **tadawul-technical-analyst** | Chart patterns, indicators, probability-weighted scenarios with Saudi-specific considerations. |
| 4 | **tadawul-position-sizer** | Fixed-risk share-count calculation in SAR with current tick bands, daily limits, and concentration caps. |
| 5 | **tadawul-portfolio-manager** | Holistic review: concentrations, TASI comparison, purification ledger, quarterly Shariah re-verification. |
| 6 | **tadawul-sector-rotation** | Sector leadership, cycle phase, Vision 2030 catalysts. |

### Operational (paper trading loop — new in v2)

| # | Skill | What it does |
|---|---|---|
| 7 | **tadawul-paper-portfolio** | Manages a strict-schema JSON portfolio: simulated cash, positions, trade history, realized P/L, T+2 settlement. **Paper only** — cannot place real orders. |
| 8 | **tadawul-trade-journal** | Append-only JSONL log of every decision (including SKIPs). Captures thesis at decision time, then outcome at close. Enables calibration over time. |
| 9 | **tadawul-risk-gate** | Hard pre-trade checks: Shariah, cash, concentration, sector caps, daily loss limits, tick alignment, liquidity. Returns PASS or REFUSE with full reasoning. |
| 10 | **tadawul-daily-loop** | Orchestrates the others into morning / mid-session / end-of-day runs. Degrades gracefully when data is unavailable. |

Each skill has its own `SKILL.md` plus reference files.

---

## 🎯 Intended Workflow (Phase 1 Paper Trading)

The skills are designed to run a week-long paper-trading experiment. Suggested flow:

**Day 0 — Setup**
1. Decide starting capital (recommend SAR 500-1000 for the experiment; fees make smaller amounts uneconomic even on paper)
2. Call `tadawul-paper-portfolio` with `init` to create `paper_portfolio.json`
3. Optionally customize `risk_rules.json` (defaults are conservative)

**Day 1+ — Daily rhythm (Sun-Thu)**
- **Before 10:00 Riyadh:** "run the morning brief" → `tadawul-daily-loop` mode `morning`
- **During 10:00-15:10:** "run mid-session" (once or twice) → mode `mid_session`
- **After 15:10:** "run end of day" → mode `end_of_day`

**Weekly (Thursdays after close)**
- Full `tadawul-portfolio-manager` review
- Journal stats via `tadawul-trade-journal.compute_stats`

**What you're learning**
- Whether Claude's analytical judgment actually predicts outcomes (journal calibration)
- What the real cost of running Claude is per trading day (API usage)
- Which skills trigger when they should and which don't
- What's missing from the toolkit for your style

The paper trading result itself (gain or loss over a week at SAR 1K) is noise — that's not the signal. The signal is in the journal.

---

## 🚀 How to Deploy

### Option A — Claude.ai Web App (recommended)

1. Go to **claude.ai** in your browser
2. Click **Settings** → **Capabilities** / **Skills** (depending on your plan)
3. Click **Upload skill** — upload each `.skill` file in this bundle
4. Enable each skill
5. Start a chat and use natural prompts; skills auto-invoke based on your phrasing

**Note on auto-invocation:** Skills trigger based on how close your prompt matches their description. "Is 2222 halal?" reliably triggers the Shariah screener; "check this stock" might not. When in doubt, name the skill directly: *"Use the tadawul-daily-loop skill to run my morning brief."*

### Option B — Claude Code (CLI/desktop)

1. Open Claude Code
2. **Settings → Skills → Open Skills Folder**
3. Copy each skill folder from this bundle into the Skills folder
4. Restart / reload

### Option C — Claude Cowork (limited)

Cowork does not load `.skill` files the same way claude.ai does. For this use case, prefer claude.ai. Cowork can still read the SKILL.md files as context prompts if you want, but you won't get auto-invocation.

### Option D — Future autonomous setup (OpenClaw)

If you get to Phase 3 and want autonomous execution (not recommended before running Phase 1 for a week), OpenClaw can drive the broker's web UI. These skills work with OpenClaw as the "brain" — they're model-agnostic instructions. Phase 3 also needs:
- Your broker's terms of service reviewed for auto-trading allowance
- A plan for 2FA / Nafath prompts (there is no fully hands-off solution today)
- A supervisor prompt (e.g., Telegram bot) before every order placement

None of that is needed for Phase 1.

---

## 🎯 Example Prompts

### Research

> "Is Almarai (2280) shariah compliant right now?"  
> "Analyze Sulaiman Al Habib as a short-term opportunity"  
> "Where are key support/resistance for Al Rajhi?"  
> "I have SAR 80,000, want to buy STC at 40 with stop at 38. How many shares?"

### Operational (paper trading)

> "Initialize my paper portfolio with SAR 1,000 starting capital"  
> "Run today's morning brief"  
> "I want to paper-buy 10 shares of 2222 at SAR 28.50 — check the risk gate first"  
> "Run end-of-day"  
> "Show my journal stats for this week"  
> "What trades did I skip and why?"

### Combined

> "I want to add a healthcare position. Find me a Shariah-compliant name with a clean setup, size it for my SAR 1K paper account at 1% risk, and record it if the risk gate passes."

This one prompt invokes 5-6 skills.

---

## ⚠️ Important Disclaimers

1. **Not financial advice.** Claude analyzes; you decide.
2. **Paper portfolio only.** The `tadawul-paper-portfolio` skill cannot place real orders. Real trades require you to execute in your broker app.
3. **Shariah lists change quarterly.** Every screening in this bundle fetches live from Argaam or Almaqasid. Still, the definitive source is your Shariah board.
4. **Past performance ≠ future results.** Especially true with ~30 trades of evidence.
5. **Saudi market has limited liquidity in some names.** Mid/small-caps can be hard to exit. Nomu especially.
6. **Purification ≠ Zakat.** Separate Islamic obligations.
7. **Margin / short-selling not supported.** Most margin is riba-based; shorts are generally not compliant. Skills are long-only.
8. **Tick-size matrix changed June 29, 2025.** The skills use the current schedule. If trading beyond mid-2025, verify no further changes.

---

## 🔗 Key Saudi Resources

| Resource | URL | Use |
|---|---|---|
| Argaam | argaam.com | Daily fundamentals + Shariah list |
| Almaqasid | almaqasid.com | Shariah verification |
| Saudi Exchange | saudiexchange.sa | Official prices + disclosures |
| TradingView | tradingview.com/symbols/TADAWUL-XXXX/ | Charts |
| Yahoo Finance | finance.yahoo.com (XXXX.SR) | Historical data |

---

## 🔄 Updating / Extending

Each skill is self-contained in its folder. To update:
1. Edit the `SKILL.md` or reference files
2. Re-zip: `cd /path/to/bundle && zip -r ../<skill-name>.skill <skill-name>/`
3. Re-upload to claude.ai

Suggested updates as you use the toolkit:
- Your actual broker's commission rates → update `metadata` in paper portfolio
- Your preferred Shariah board if it differs from AAOIFI
- Sector caps if you're consistently bumping into them
- Risk rules as you learn from journal data

---

## 📝 Changes from v1

| Change | Reason |
|---|---|
| Tightened SKILL.md descriptions | Better auto-invocation; concrete trigger phrases |
| Added explicit `web_search` / `web_fetch` instructions | Prevents fallback to stale training-data memory |
| Updated tick-size matrix | Tadawul reformed tick sizes June 29, 2025 (v1 used old bands) |
| Corrected trading hours | 10:00-15:10 (was 10:00-15:00) |
| Added QFI framework abolishment note | As of Feb 1, 2026 |
| **Added tadawul-paper-portfolio** | Manages simulated portfolio state |
| **Added tadawul-trade-journal** | Captures decision rationale for calibration |
| **Added tadawul-risk-gate** | Hard pre-trade checks |
| **Added tadawul-daily-loop** | Orchestrates the daily experiment cycle |

---

**Built April 2026** | Always verify current Shariah status and market data before any trade.
