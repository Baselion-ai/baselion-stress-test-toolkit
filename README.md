# baselion-stress-test-toolkit

> **12-category stress testing framework for algorithmic trading strategies.**

`baselion-stress-test-toolkit` is an opinionated, open-source framework for pressure-testing quantitative trading strategies before they see real money. It codifies the 12 stress categories that the [FOX OMEGA Engine](https://baselion.ai) runs internally before any signal reaches a production tier.

## Why another backtest library?

Most open-source backtesting tools stop at in-sample performance and a naïve walk-forward. That gets you to *"looks good on the training set"* — it does not get you to *"still good when the market regime shifts, the venue degrades, or a whale moves the book."*

This toolkit focuses on the **12 failure modes** that kill otherwise-profitable strategies in production:

| # | Category | Checks against |
|---|----------|----------------|
| 1 | Walk-forward (expanding & rolling) | lookahead bias, overfit parameters |
| 2 | Combinatorial Purged Cross-Validation (CPCV) | information leakage across folds |
| 3 | Monte Carlo bootstrap | path-dependent fragility |
| 4 | Regime stress (HMM / GMM splits) | strategy dies in LATERAL / STRONG_TREND / VOLATILE |
| 5 | Whale order-flow imbalance stress | price impact from single large actors |
| 6 | Latency degradation | +50 ms / +200 ms / +1 s inference delay |
| 7 | Fee & slippage sensitivity | taker fee changes, adverse selection |
| 8 | Book-depth degradation | thin-book execution |
| 9 | Funding rate regime | perpetual funding flips |
| 10 | Correlated venue outage | one exchange drops for N minutes |
| 11 | Data gap / flash crash injection | missing candles, tick storms |
| 12 | Deflated Sharpe Ratio (DSR) vs # of trials | honest significance after parameter search |

## Install

```bash
pip install baselion-stress-test-toolkit
```

## 30-second example

```python
from baselion_stress import StressSuite, load_ohlcv

prices = load_ohlcv("BTCUSDT", "2022-01-01", "2026-01-01", "1h")

suite = StressSuite(
    strategy=my_strategy_fn,           # callable(prices) -> signal series
    categories="all",                   # or a subset: ["walk_forward", "cpcv", "dsr"]
    n_bootstrap=1000,
    regime_detector="gmm",
    seed=42,
)
report = suite.run(prices)

print(report.summary())        # PASS/WARN/FAIL per category
report.to_html("stress.html")  # full drill-down
```

## What a passing report looks like

```
Category                         Result   Metric
walk_forward                     PASS     median_sharpe=1.43 (N=12 folds)
cpcv                             PASS     oos_sharpe=1.21  (purge=20, embargo=5)
monte_carlo_bootstrap            PASS     p5_sharpe=0.68
regime_gmm                       WARN     volatile_sharpe=0.42  (< 1.0 threshold)
whale_ofi_stress                 PASS     max_impact=-12bps @ p99
latency_degradation              PASS     sharpe_at_+200ms=1.11
fee_slippage                     PASS     breakeven_fee=18bps (current=5bps)
book_depth                       PASS     sharpe_at_30pct_depth=0.91
funding_regime                   PASS     funding_flip_sharpe=0.88
venue_outage                     PASS     max_drawdown_10min_gap=-3.2%
data_gap                         PASS     robustness=0.94
dsr                              PASS     dsr=1.08 (trials=240, alpha=0.05)
```

A single **FAIL** or multiple **WARNs** should stop deployment until the authors understand why.

## Notebooks

* [`notebooks/01_quickstart.ipynb`](notebooks/01_quickstart.ipynb) — run the full suite on a baseline SMA crossover
* [`notebooks/02_cpcv_deep_dive.ipynb`](notebooks/02_cpcv_deep_dive.ipynb) — Lopez de Prado's CPCV, step by step *(coming soon)*
* [`notebooks/03_dsr_worked_example.ipynb`](notebooks/03_dsr_worked_example.ipynb) — Deflated Sharpe vs plain Sharpe *(coming soon)*

## Methodology references

* López de Prado, M. (2018). *Advances in Financial Machine Learning.* Wiley. (CPCV, DSR)
* Bailey, D. H., & López de Prado, M. (2014). *The Deflated Sharpe Ratio.* SSRN.
* Harvey, C. R., Liu, Y., & Zhu, H. (2016). *…and the Cross-Section of Expected Returns.* Review of Financial Studies.
* Bailey, D. H., Borwein, J., López de Prado, M., & Zhu, Q. J. (2014). *Pseudo-mathematics and financial charlatanism.* Notices of the AMS.

## Status

Alpha. API will change. Pin the version. Issues and PRs welcome — especially additional stress categories.

## License

MIT.

---

## About Baselion

Baselion is an institutional-grade quantitative trading intelligence platform, powered by the proprietary FOX OMEGA Engine. Learn more at [baselion.ai](https://baselion.ai).
