# Value Agent

A fully automated US-equity research engine. Every weekday it screens
~6,000 companies across 12 classic value and growth frameworks (Graham,
Buffett, Lynch, and others) plus regulatory-filing signals, publishes a
ranked daily watchlist, and — since 2026-07-22 — runs a **paper-trading
portfolio** (top 20 names, equal weight, monthly rebalance) with every
order recorded in a public, append-only ledger.

**No human stock-picking. No overrides. No deleted history.**

## Follow along

- 📊 **Live paper-trading scoreboard:**
  [kpms0925-max.github.io/value-agent-scoreboard](https://kpms0925-max.github.io/value-agent-scoreboard/)
  — equity curve vs SPY, current holdings, monthly returns. Paper trading
  only; no real capital.
- 📣 **Public Telegram channel:** daily top-20 value / top-5 growth lists
  *(link added once the channel is live)*.
- ✉️ **Email report:** deeper daily write-up — signup via the scoreboard page.

## Evidence discipline

Before any paper trading began, the engine was backtested over 2013–2025
under **pre-registered, locked pass/fail criteria** — rules committed before
results existed, failing variants published alongside the winner:

- [Pre-registered criteria (v2)](docs/BACKTEST_PREREGISTRATION_V2.md)
- [Backtest verdict report](backtest_runs/stage2_eval/report.md)
- [Analysis of the failing constructions](docs/STAGE2_DIAGNOSIS.md)
- [Paper-trading ledger](paper_ledger/) (append-only)

Backtest results are simulated history and are never blended into the live
paper-trading curve.

## Disclaimer

**Not investment advice.** This is an educational and informational project.
Simulated (paper) trading does not represent real trading; no real capital
is at risk; nothing here is a recommendation to buy or sell any security.
Do your own research.
