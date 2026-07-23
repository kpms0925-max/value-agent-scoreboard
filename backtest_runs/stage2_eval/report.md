# Stage-2 Backtest — Phase 5 Verdict (pre-registered criteria)

Window: 2013-01-31 .. 2025-12-31 | monthly returns: 155 | 36m windows: 120 | 60m windows: 96
Costs: 0.1%/side modeled on all portfolio trades. Benchmark: SPY dividend-adjusted (gross). SPY maxDD: -23.9%

## 1. Verdict matrix

| Portfolio | 36m alpha>0 share (need >=65%) | 60m alpha>0 share (need >=75%) | maxDD (limit -33.9%) | CAGR margin (need >= +2.0pp) | Verdict |
|---|---|---|---|---|---|
| top10 | 87.5% (105/120) PASS | 95.8% (92/96) PASS | -28.7% PASS | +4.3% PASS | **PASS** |
| top20 | 85.8% (103/120) PASS | 99.0% (95/96) PASS | -30.5% PASS | +3.4% PASS | **PASS** |
| sw50 | 79.2% (95/120) PASS | 94.8% (91/96) PASS | -31.3% PASS | +2.4% PASS | **PASS** |
| value | 83.3% (100/120) PASS | 95.8% (92/96) PASS | -31.8% PASS | +2.2% PASS | **PASS** |
| garq | 41.7% (50/120) FAIL | 41.7% (40/96) FAIL | -43.5% FAIL | +0.9% FAIL | **FAIL** |
| buy | 39.2% (47/120) FAIL | 49.0% (47/96) FAIL | -76.3% FAIL | -14.2% FAIL | **FAIL** |

## 2. Calibration (criterion 4)

**Value sleeve (conviction score) — the pass/fail population (design I-3):** n=34,748, excluded=0

| Decile | n | score range | mean fwd 12m |
|---|---|---|---|
| 1 | 3,475 | 0–12 | +11.3% |
| 2 | 3,475 | 12–17 | +10.3% |
| 3 | 3,475 | 17–20 | +11.6% |
| 4 | 3,475 | 20–23 | +13.8% |
| 5 | 3,475 | 23–26 | +14.2% |
| 6 | 3,475 | 26–30 | +14.8% |
| 7 | 3,475 | 30–33 | +18.5% |
| 8 | 3,475 | 33–37 | +18.8% |
| 9 | 3,474 | 37–43 | +21.8% |
| 10 | 3,474 | 43–83 | +25.9% |

Spearman(decile, mean fwd) = 0.988 (>0 PASS); top>bottom: YES -> **calibration PASS**

**GARQ sleeve (garq_adjusted) — BINDING under the v2 pre-registration (prereg §1):** n=3,396, excluded=0

| GARQ decile | n | mean fwd 12m |
|---|---|---|
| 1 | 340 | +11.4% |
| 2 | 340 | -0.2% |
| 3 | 340 | +4.3% |
| 4 | 340 | +10.9% |
| 5 | 340 | +10.8% |
| 6 | 340 | +12.1% |
| 7 | 339 | +14.1% |
| 8 | 339 | +16.7% |
| 9 | 339 | +17.4% |
| 10 | 339 | +18.3% |

Spearman(decile, mean fwd) = 0.867 (>0 PASS); top>bottom: YES -> **GARQ calibration PASS**

## 3. Context (informational, not criteria)

SPY total return +470.6% (CAGR +14.4%, Sharpe 1.03, Sortino 1.67)

| Portfolio | total return | CAGR | Sharpe | Sortino | cost drag (cum) | empty months | delist exits | skipped entries |
|---|---|---|---|---|---|---|---|---|
| top10 | +817.6% | +18.7% | 0.86 | 1.53 | +11.6% | 0 | 1 | 0 |
| top20 | +730.4% | +17.8% | 0.85 | 1.52 | +11.2% | 0 | 6 | 0 |
| sw50 | +647.3% | +16.8% | 0.83 | 1.50 | +9.0% | 0 | 10 | 0 |
| value | +629.0% | +16.6% | 0.82 | 1.47 | +9.2% | 0 | 7 | 0 |
| garq | +531.8% | +15.3% | 0.67 | 1.12 | +9.9% | 0 | 3 | 0 |
| buy | +3.7% | +0.3% | 0.17 | 0.24 | +12.9% | 38 | 1 | 0 |

## 4. Final verdicts (portfolio criteria AND calibration)

- **top10: PASS** (36m ok, 60m ok, maxDD ok, CAGRmargin ok, calV ok, calG ok)
- **top20: PASS** (36m ok, 60m ok, maxDD ok, CAGRmargin ok, calV ok, calG ok)
- **sw50: PASS** (36m ok, 60m ok, maxDD ok, CAGRmargin ok, calV ok, calG ok)
- **value: PASS** (36m ok, 60m ok, maxDD ok, CAGRmargin ok, calV ok)
- **garq: FAIL** (36m X, 60m X, maxDD X, CAGRmargin X, calG ok)
- **buy: FAIL** (36m X, 60m X, maxDD X, CAGRmargin X, calV ok, calG ok)

*Criteria and definitions exactly as locked in docs/BACKTEST_PREREGISTRATION_V2.md (c8b3a31) and docs/PHASE5_EVAL_DESIGN.md. No parameter was chosen or adjusted after results existed.*
