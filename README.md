# Does cointegration survive out of sample? A pairs-trading study

**Short answer: mostly, no.** I screened 45 pairs of US energy stocks for cointegration on 2015–2020 data, froze each pair's hedge ratio, and tested everything on unseen 2021–2025 data, both statistically and by trading it.

## Question
If two stocks look cointegrated in past data, does the relationship actually hold in the future, and could you profit from it?

## Data and setup
- Daily adjusted closes from Yahoo Finance (`yfinance`) for 10 energy stocks: XOM, CVX, COP, EOG, OXY, MPC, PSX, VLO, SLB, HAL (45 pairs).
- 2,765 trading days, split **by date**: train 2015–2020 (1,511 days), test 2021–2025 (1,254 days). The test period was not touched until the pairs and hedge ratios were fixed.

## Method
1. **Screen** all 45 pairs on the training data with the Engle–Granger cointegration test (`statsmodels`).
2. **Freeze** each pair's hedge ratio (OLS on training data) and build the spread `y − β·x` on the test data.
3. **Test** whether the frozen spread stays stationary out of sample (ADF test), and whether training p-values predict test p-values.
4. **Trade it**: a z-score rule using the training mean and standard deviation. Enter beyond ±2, exit when z crosses 0, optional stop at |z| = 4. Daily P&L = yesterday's position × today's change in spread, so there is no look-ahead.
5. **Stress it**: stop-loss variants, trading costs (5–10 bps), a per-trade breakdown, and the same rule across all 45 pairs.

## Key results
| Check | Result |
|---|---|
| Pairs passing cointegration at 0.05 in training | 0 of 45 |
| Pairs passing the stationarity test out of sample | 1 of 45 (chance would give about 2.25) |
| Correlation of training vs test p-values | −0.008 |
| PSX–VLO, simple rule, no costs | +28.66 per unit of PSX (four closed trades about +41, one open trade about −12.4) |
| PSX–VLO max drawdown, no stop / stop at 4 | 28.3 / 15.6 |
| PSX–VLO net P&L at 5 bps / 10 bps costs | 27.72 / 26.78 |
| All 45 pairs, no stop | 11 profitable, median −18% of price level |
| All 45 pairs, stop at 4 | 15 profitable, median −8% |

**Takeaways**
- A pair's training p-value told me nothing about how it would behave out of sample.
- PSX–VLO made money, but it sits in the right tail of the 45 pairs, so it reads as a favourable draw rather than proof that the relationship held.
- A stop-loss cuts drawdown and helps the typical pair, but costs return on a pair that eventually reverts. It is insurance, not a source of return.

## What this project shows
- Strict out-of-sample methodology: date-based split, frozen parameters, no look-ahead.
- Awareness of multiple-testing and selection effects (45 pairs, best-of-45 results treated with suspicion).
- Time-series statistics with `statsmodels` (cointegration, ADF), and data work in `pandas` and `NumPy`.
- A trading backtest built as a position state machine, with trade-level attribution, drawdown, Sharpe and costs.
- Results cross-checked by recomputing individual pairs independently, and a negative result reported plainly.

## Run it
```bash
pip install -r requirements.txt
jupyter notebook cointegration_project.ipynb
```
Run all cells from the top. Prices are downloaded live from Yahoo Finance, so figures may shift slightly if Yahoo revises adjusted prices.

## Files
- `cointegration_project.ipynb`: the full analysis with notes at every step
- `requirements.txt`: Python dependencies

*Scope note: one sector, one test period, a handful of trades per pair, and a simple cost model, so the results are indicative rather than conclusive.*
