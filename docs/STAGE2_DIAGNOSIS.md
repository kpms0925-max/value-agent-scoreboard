# Stage-2 Diagnosis — GARQ-only and BUY-only failures (v2 verdict)

Read-only investigation against the frozen v2 dataset
(`backtest_runs/prod/`, commit cc0901f) and the Stage-2 verdict artifacts
(`backtest_runs/stage2_eval/`). No scoring changes, no re-run of the
evaluation, no criterion touched. Companion to the v2 verdict in
`backtest_runs/stage2_eval/report.md`: top10/top20/sw50/value PASS;
garq-only and buy-only FAIL. All figures below computed from the frozen
records/watchlists, the verdict run's own `series.json`/`decisions.jsonl`,
and cached SEP closeadj prices — the same price source the verdict used.

BUY threshold referenced throughout: `rank_pct >= 0.9871350292506023`
(= 1 − 486/37,777, the v1-anchored 1.286% per-sleeve percentile,
prereg §BUY). Gates (scoring.py `compute_recommendation`, v2 symmetric
block): BUY requires rank_pct ≥ threshold AND sell_score == 0 AND
rr_label ≠ "Poor" AND news ≥ −1.

---

## 1. BUY-only: structural concentration, not bad luck

### 1.1 The portfolio is usually one stock

Holding counts across all 156 months (watchlist holdings with
recommendation BUY — the portfolio rule):

| holdings/month | months |
|---|---|
| 0 | 38 |
| 1 | 68 |
| 2 | 32 |
| 3 | 10 |
| 4 | 8 |

Median month: **one name at 100% weight**. Zero-BUY months cluster in
2013 (12/12 — news layer effectively silent early) and recur through
2021-2023. 194 holding-months total: 89 value, 105 growth — the growth
sleeve supplies **54% of BUY exposure from 9% of scored records**
(3,650 of 41,426). Distinct tickers: 36 value, 25 growth. Top repeats:
ZD 15 months, NVDA 11, IX 11, AGCO 9, IPAR 8.

Single-name months contribute **78% of the full series' monthly return
variance**. Mean return in single-name months +0.55%/mo vs +0.69%/mo in
multi-name months — concentration adds variance without adding return.

### 1.2 maxDD −76.3% is a chain of single-stock crashes

Peak 2021-12-31, trough 2025-10-31. The worst months are almost all
one-name months:

| month | net ret | book |
|---|---|---|
| 2023-12 | −32.2% | DQ 100% |
| 2024-03 | −31.8% | DQ 100% |
| 2025-08 | −25.4% | HRMY 100% |
| 2022-08 | −19.7% | NVDA 100% |
| 2022-01 | −17.2% | BIGGQ 100% |
| 2021-12 | −17.1% | BIGGQ/NVDA/TECH ⅓ each |
| 2022-07 | −14.1% | INTC/NVDA 50/50 |
| 2022-11 | −13.7% | NVDA 100% |

Three episodes do most of the damage:

- **NVDA 2022**: the gate set kept re-selecting NVDA through its 2022
  collapse — 13 NVDA-holding months compound to ×0.727. Calendar 2022
  alone: buy-only −59.6%.
- **DQ whipsaw 2023-11..2024-03**: −32.2%, +17.9%, +32.3%, −31.8% at
  100% weight — five months compound to ×0.766.
- **HRMY 2025**: five months, ×0.797, including −25.4% at 100% weight.

A 1-2 name book has no diversification to absorb any of these; the same
names inside top10/top20 cost ≤10% weight each and those portfolios
passed (maxDD −28.7% / −30.5%).

### 1.3 Compound-gate hypothesis: CONFIRMED, two distinct mechanisms

Funnel over all 156 months (records → threshold → +sell_score==0 →
+rr≠Poor → +news gate (=BUY) → ∩watchlist):

| sleeve | records | ≥thr | +sell0 | +rrOK | BUY | on watchlist |
|---|---|---|---|---|---|---|
| value | 37,776 | 642 (1.70%) | 369 | 360 | **87** | 87 |
| growth | 3,650 | 167 (4.58%) | 128 | 128 | **105** | 105 |

- The anchor implies ~3.1 value BUYs/month (1.286% × ~242
  candidates/month). Actual: **0.56/month** — the stacked gates keep only
  14% of value threshold-passers. The dominant cut is the **news gate**:
  of 360 value names passing threshold+sell+R/R, 271 carry news score
  −2 and are blocked by `news >= −1`; 95 of 156 months end with zero
  value BUYs.
- Mechanism behind that cut: the N8 8-K exhibit expansion made the same
  single-headline months score **−2 in v2 where v1 scored −1** (visible
  in matched run logs: `NEWS LRCX/SWBI/DECK/X/...: score=-1` v1 →
  `score=-2` v2). v1's realized BUY frequency — the source of the 1.286%
  anchor — was generated under the weaker news scores, so the anchor
  over-promises what the v2 gate stack can deliver. Not a bug: each
  piece behaves as specified; the *composition* starves the portfolio.
- Growth sleeve is the opposite failure: ~23 gate-eligible names/month
  means the 98.71st percentile is effectively "the #1-ranked name only"
  (top-1 = 4.3% > 1.29% granularity floor; realized 4.58% of records).
  News is never scored on growth records (no news key exported; gate
  auto-passes at 0), so 63% of growth threshold-passers become BUYs.
  Result: growth supplies most BUY months as **single-name bets on the
  one top-ranked growth stock** — NVDA/ZD/DQ/HRMY above.

### 1.4 Verdict on buy-only

**Structural concentration risk, not a bad-luck cluster.** The v1-anchored
percentile threshold assumes a population it no longer gets: value BUYs
are starved by the news gate interacting with N8's stronger news scores,
and growth "BUYs" collapse to top-1-of-23 picks with no news gate at
all. The specific 2022-2025 losses (NVDA, DQ, HRMY) are the expected
outcome of a 1-name, 100%-weight book in the highest-volatility sleeve —
different names would have produced a similar drawdown profile
eventually. Worth noting: the identical signal set diversified to 10-50
names (top10/top20/sw50) passed every binding criterion.

---

## 2. GARQ-only: pool risk dominant, ranking is not the problem

### 2.1 The gate population itself is higher-risk — CONFIRMED

Equal-weight forward-1-month returns of the **full gate-eligible pools**
(every scored record, before any ranking/selection; gross, no costs):

| pool | avg names/mo | ann. vol | Sharpe | maxDD | CAGR |
|---|---|---|---|---|---|
| value (all candidates) | 242 | 19.7% | 0.72 | −34.1% | +12.9% |
| growth (all GARQ-gated) | 23 | 20.9% | 0.56 | −39.8% | +10.0% |

Independent of anything GARQ's score does, the growth gate population
as a group runs +1.2pp higher volatility, −0.16 lower Sharpe, −5.8pp
deeper maxDD, and −2.9pp lower CAGR than the value candidate pool. Its
average cross-sectional monthly dispersion is also wider (11.7% vs
11.1%). The gap is broad, not era-specific: the growth pool's EW mean
monthly return trails the value pool's in 9 of 13 calendar years
(worst: 2021 −1.63pp/mo, 2022 −1.45pp/mo; only 2017 and 2024
meaningfully favor growth).

### 2.2 Decomposition of the garq-only shortfall

Portfolios as evaluated (net of costs): value port 21.8% vol / 0.82
Sharpe / −31.8% maxDD; garq port 26.7% vol / 0.67 Sharpe / −43.5% maxDD
(median 5 names, range 1-9).

| metric | garq shortfall vs value port | from pool (growth pool − value pool) | from selection/concentration (garq port − growth pool) |
|---|---|---|---|
| Sharpe | −0.15 | **−0.16** | +0.11 (ranking *helps*) |
| maxDD | −11.7pp | **−5.8pp** | −3.6pp (5-name book vs 23-name pool) |
| ann. vol | +4.9pp | +1.2pp | +5.8pp |

(Rows don't sum exactly — value-port-vs-value-pool selection effects
absorb the remainder; direction and dominance are the point.)

- **Ranking works.** GARQ's selection *improves* Sharpe over its own
  pool (0.67 vs 0.56) and lifts CAGR from 10.0% (pool, gross) to 15.3%
  (portfolio, net). The binding GARQ calibration passed in the verdict
  (calG ok) — deciles are monotone. The remaining "ranking weakness"
  from Stage-1 (v1's inverted GARQ cohorts) is not visible in v2.
- **The pool breaches the criterion on its own.** The maxDD limit was
  −33.9% (SPY −23.9% + 10pp). The full growth pool EW — zero selection
  involved — already draws down −39.8%. No ranking of a 23-name
  higher-beta pool was likely to pass; concentration (median 5 names)
  adds ~3.6pp more.
- Attribution: roughly **half to two-thirds** of the Sharpe/maxDD
  shortfall traces to the gate population's structural risk profile;
  the rest to small-book concentration; approximately none to the GARQ
  score's ordering.

### 2.3 Verdict on garq-only

**Confirmed: the growth-sleeve gate population is structurally
higher-risk/higher-volatility as a group, independent of GARQ's ranking
of it.** garq-only fails on pool composition + book size, while the
score itself adds value within the pool. Any v3 fix aims at the gate
definition/pool size (or at diversification constraints), not at the
ranking.

---

## 3. Implications (hypotheses only — nothing changed in this round)

1. The passing constructions (top10/top20/sw50/value) are the same
   signals with diversification; nothing in this diagnosis casts doubt
   on them. The two failures are portfolio-construction failures, not
   signal failures.
2. v3 candidates for buy-only: re-anchor the percentile on the v2 news
   distribution, or make the BUY population a floor-count construction
   (e.g. top-N-with-gates), or subject BUY to a minimum-breadth rule
   before it is publishable as a standalone portfolio. All require the
   before/after change protocol.
3. v3 candidates for garq-only: widen/retighten the GARQ gate set to
   grow the pool, and/or cap single-name weight. Ranking untouched.
4. The live product's BUY label inherits mechanism 1.3 today (same
   engine): single-headline −2 news blocks value BUYs; growth BUYs are
   top-1 picks. Worth an owner decision on how BUY is surfaced to
   subscribers until a v3 fix lands (product/format decision, outside
   the scoring firewall).

## Appendix — method

- Portfolio composition/counts: `recommendation == "BUY"` holdings in
  each `backtest_runs/prod/<date>/watchlist.json`, all 156 months.
- Drawdown attribution: verdict run's own `series.json` (buy returns,
  net) + `decisions.jsonl` (entries/weights); per-name gross monthly
  contribution from cached SEP closeadj (`PriceBook`, exact scoring-date
  bars, last-on-or-before for delist months) — same price source as the
  verdict run.
- Funnel: record-level fields (`rank_pct`, `sell_score`, `rr_label`,
  `recommendation`, `news_informational`) from `records.jsonl`; gate
  semantics per scoring.py `compute_recommendation` (v2 block).
- Pools: per month, all records of a sleeve, forward 1-month EW return
  to the next scoring date; stats over the 155 forward months.
  Population stdev, rf=0 Sharpe, ×√12 annualization — matching the
  verdict report's informational-ratio conventions.
