# Half-Kelly evaluation addendum (pre-registered BEFORE any constructor code)

Committed 2026-07-22, before any half-Kelly constructor code exists in
this repository. Evaluation-only: replays the frozen Stage-2 v2 dataset
(`backtest_runs/prod/`, committed at cc0901f) and cached PIT prices
through the existing `evaluate.py` machinery. NO scoring change, NO
rescoring, NO modification of any existing verdict artifact
(`backtest_runs/stage1_eval/`, `backtest_runs/stage2_eval/`). Once
results exist, nothing in this file may change; a change restarts this
evaluation.

## Construction (owner-approved text, verbatim)

At each scoring date, eligible names = those passing the existing
rank_pct entry threshold in their sleeve. Expected excess return per
name = expanding-window realized mean forward-12m return of its sleeve
score-decile, using only months fully completed before that scoring
date, minimum 24 months of completed history; before that burn-in,
weights are equal. Risk = trailing 12-month realized monthly return
variance from cached PIT prices. Raw weight = 0.5 × (expected excess
return ÷ variance); normalize; cap 10% per name; min 5 / max 50
holdings (if fewer than 5 eligible, hold eligible names EW; if more
than 50, take top 50 by rank_pct). Monthly rebalance, 0.1%/side costs,
delisting rules unchanged from v1 §5.

## Binding criteria

Identical to `docs/BACKTEST_PREREGISTRATION_V2.md` §1 (36m/60m alpha
shares, maxDD limit, CAGR margin ≥ +2.0pp, both calibrations already
passed and unchanged).

## Decision rule

Half-Kelly is selected for paper trading iff it passes all binding
criteria AND beats top20 EW on ≥2 of {CAGR margin, Sharpe, maxDD}.
Otherwise top20 EW is selected. No other outcome, no post-hoc
adjustment.

## Implementation micro-decisions (frozen pre-results) — appendix

Copied verbatim from the halfkelly_eval.py module docstring (commit
49f6874), where they were frozen at implementation time, before any
results existed:

```
  M1 eligibility pool = scored RECORDS with rank_pct >= V2_BUY_RANK_PCT
     (the only rank_pct entry threshold that exists; both sleeves).
  M2 sleeve score = conviction_adjusted (value) / garq_adjusted (growth) —
     the same selectors the binding calibrations use.
  M3 per-month sleeve deciles: calibration-style contiguous buckets over
     that month's sleeve population, ascending score (divmod sizing).
  M4 forward-12m return convention = evaluate.calibration verbatim:
     closeadj(t_{i+12})/closeadj(t_i)-1; delisted -> terminal last closeadj;
     pair skipped only if no price at t_i.
  M5 expected-return fallback: empty (sleeve, decile) history -> sleeve-wide
     expanding mean; E <= 0 -> name dropped from the Kelly book (long-only,
     mirrors evaluate.select's sw50 non-positive-weight rule); every E <= 0
     -> EW over eligible.
  M6 variance validity: >= 6 trailing monthly grid returns and var > 0;
     invalid -> cross-sectional median variance of the month's valid
     eligible names; no valid names at all -> EW month.
  M7 "normalize; cap 10%": normalize positive raw weights to sum 1, then
     cap at 10% with the excess left UNINVESTED (cash, 0%) — not
     redistributed. Burn-in / fewer-than-5 EW branches are sum-1 EW
     (the addendum's explicit overrides).
  M8 entry-price guard: a selected name with no closeadj at t0 is dropped
     and its weight stays in cash (no renormalization), consistent with M7.
  M9 burn-in: month i has completed history j <= i-12; count = i-11;
     Kelly activates at the first month with >= 24 completed (index 35).
```

Note: M1 resolved the addendum's "existing rank_pct entry threshold"
to the BUY rank_pct threshold (V2_BUY_RANK_PCT = 1 - 486/37777)
because no other rank_pct threshold exists anywhere in v2.
