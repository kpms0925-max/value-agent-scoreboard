# Backtest Pre-Registration — Stage-2 (v2 engine)

**Committed BEFORE any historical scoring run under the v2 engine.
Nothing in this file may change once results exist. A change to this
file restarts the Stage-2 backtest.**

**Governs the v2 engine only. Does not revise, supersede, or reopen
docs/BACKTEST_PREREGISTRATION.md (Stage-1/v1), which remains the
permanent, unaltered record of the v1 verdict.**

Approved by kpms0925: 2026-07-20 (methodology, all binding criteria, and
signal PIT-verification results).
Engine: frozen at tag `backtest-v2` (bf2a5d8), branch `v2-scoring` off
`backtest-infra`. Scoring changes per docs/V2_SCORING_DESIGN.md (a7da187)
§1–§5; implementation verified in docs/V2_BUILD_REPORT.md. Parity gate
passed 2026-07-19 (byte-identical agent.py/backtest.py replay).
Evaluation module: evaluate.py extended to add the CAGR-margin criterion
and Sharpe/Sortino informational reporting (§1/§2 below); all v1-era
calculations byte-identical and regression-tested (tests/eval_verify.py
unchanged, 34/34 passing) — commit 44e66ed.
Claude's ±10 qualitative adjustment: excluded entirely, unchanged from v1.
Signal layers: v1-era classifications frozen per
docs/BACKTEST_SIGNAL_INVENTORY.md. Form 4 insider and 8-K guidance become
live for the first time under v2 — PIT-verified 2026-07-20 (commit
6c17e6b, docs/V2_SIGNAL_PIT_VERIFICATION.md):
- Form 4: CLEAN across the full 2013–2025 window (54-filing sample +
  6,945-filing population scan; three schema versions all parse without
  modification).
- 8-K guidance: PIT-consistent but PARTIAL coverage — the regex-matched
  EX-99 exhibit mechanism catches roughly half of exhibit-bearing
  earnings 8-Ks in sample (stable ~33–56% across the window, no era
  cliff, no lookahead). v2 tests "guidance detectable via this consistent
  instrument," not full guidance detection. Accepted as a documented
  limitation, same treatment as v1's 13F pre-2013 gap and ADR/reorg
  skips.
Any signal-layer classification change restarts Stage-2.

---

## 1. Pass/fail criteria

- Rolling 3-year alpha vs. SPY positive in ≥65% of windows; rolling
  5-year alpha positive in ≥75% of windows.
- Max drawdown no more than 10 points worse than SPY over the same window.
- **Full-period magnitude (NEW, binding):** portfolio CAGR must exceed
  SPY CAGR by at least 2.0 percentage points, annualized, over the full
  evaluation period.
- Returns net of modeled costs (0.1% slippage/side, commission-free).
- Value-sleeve calibration: conviction-score deciles at least weakly
  monotonic in forward 12-month return (unchanged from v1).
- **GARQ-sleeve calibration — BINDING (v1 treated this as informational
  only):** GARQ-adjusted-score deciles at least weakly monotonic in
  forward 12-month return, top-decile mean > bottom-decile mean.

Failure stops the process — no gate discovery, no paper trading, until
diagnosed and fixed, same discipline as v1.

## 2. Operational definitions

Alpha, max drawdown, costs, benchmark, portfolio-returns: unchanged from
v1 §2, verbatim.

- **CAGR margin (NEW, binding):** compound annual growth rate computed
  from the full monthly net-of-cost equity curve (Feb 2013 – Dec 2025);
  PASS iff `portfolio_CAGR − SPY_CAGR ≥ 2.0 percentage points`.
- **Sharpe / Sortino vs. SPY (NEW, informational only, not pass/fail):**
  computed over the same full monthly net-of-cost return series;
  reported per portfolio alongside CAGR/total-return context, same
  non-binding treatment as v1's CAGR/total-return table.
- Calibration — value sleeve: unchanged from v1.
- Calibration — GARQ sleeve (NEW, binding): pool all (scoring date,
  growth candidate) pairs with a garq-adjusted score and a measurable
  forward 12-month return; deciles by score; PASS iff positive Spearman
  (decile rank, decile mean forward return) AND top-decile mean >
  bottom-decile mean.
- BUY threshold: `sell_score == 0 AND rank_pct >= V2_BUY_RANK_PCT
  (0.9871350292506023) AND news >= -1` — both sleeves.
- rank_pct: inclusive percentile within one sleeve's scored population
  on one scoring date only.

## 3. Window and cadence

Unchanged from v1 (Jan 2013 – Dec 2025, 156 dates), applied to both
sleeves. PIT discipline unchanged from v1's bullets, plus: rank_pct
computed within each scoring date's own sleeve population — same
pattern already used for Greenblatt rank/MoS percentile in v1.

## 4. Portfolios evaluated

1. Top-10 EW — 10 highest rank_pct holdings, equal weight.
2. Top-20 EW — 20 highest rank_pct holdings, equal weight.
3. Score-weighted 50 (REDEFINED) — weights proportional to rank_pct, all
   watchlist holdings.
4. Value-only EW — value-sleeve holdings, equal weight.
5. GARQ-only EW — growth-sleeve holdings, equal weight.
6. BUY-only EW — recommendation == BUY, equal weight. Pre-registered
   expectation: tight rank_pct threshold (top 1.29% of sleeve) on a
   small growth population (~22 candidates/month historically) is
   expected to produce more frequent zero-BUY growth months than v1 —
   not an anomaly.

Tie-breaks: rank_pct, then raw adjusted score, then watchlist position.
Fewer names than target → equal weight over what exists. Empty set →
100% cash, 0% return. No look-ahead.

## 5. Delistings

Unchanged from v1, verbatim.

## 6. No-peeking protocol

Unchanged in principle. Criteria evaluated only by the automated report
at the end of the full 156-month v2 run. Every decision written in the
v2 schema (design §5) from day one.

---

## Erratum (2026-07-22, post-verdict — documentation only, gate unchanged)

Section 2's BUY-threshold sentence lists three conditions
(`sell_score == 0 AND rank_pct >= V2_BUY_RANK_PCT AND news >= -1`).
The implemented gate in `scoring.py compute_recommendation` — frozen at
`backtest-v2` (bf2a5d8) BEFORE this pre-registration was locked, and the
code that actually produced the Stage-2 verdict — additionally requires
`rr_label != "Poor"` (the v1 "FIX 3" poor-risk/reward HOLD cap, carried
into v2 unchanged and never exempted). The prereg text omitted that
fourth condition; the code never changed after the lock. Funnel impact
across the full run: 369 -> 360 value threshold-passers (9
holding-month exclusions); growth unaffected (128 -> 128). Recorded for
completeness per the Stage-2 diagnosis (docs/STAGE2_DIAGNOSIS.md §1.3).
The gate itself is NOT modified by this note.
