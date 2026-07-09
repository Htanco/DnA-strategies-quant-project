# Bollinger Bands Mean Reversion Strategy — S&P 500

## Key finding

The notebook backtests a classic Bollinger Bands mean-reversion strategy on the S&P 500 (2020–2026) two ways, and both point the same direction: **the strategy did not come close to matching a passive buy-and-hold position over this window.**

- A lightweight pandas-based backtest shows the strategy compounding to roughly **+45.7%** cumulative return, versus **~+121.3%** for simply holding the index (SPX) over the same period.
- A full **Backtrader/Cerebro** engine run of the same logic — with position sizing and broker-level execution — turned a $100,000 starting portfolio into **$169,121.49**, a **52.54% total return**, with a **Sharpe ratio of 0.79** and a **16.18% max drawdown**.

The two implementations produce different absolute numbers because they handle position sizing and compounding differently, but the conclusion is consistent: this mean-reversion approach left significant return on the table relative to just holding the S&P 500. That's expected given the backdrop — 2020–2026 was a strongly trending bull market for US equities, and mean-reversion strategies are structurally built to fade moves, not ride them. A strategy that repeatedly shorts "overbought" markets and buys "oversold" dips will fight a persistent uptrend rather than benefit from it.

## Contents

- `bollinger_bands_strategy.ipynb` — full notebook: data acquisition, Bollinger Band feature engineering, mean-reversion signal generation, a pandas-based backtest, and a second backtest using the Backtrader/Cerebro engine, with charts and performance metrics.
- `spx_data.parquet` — cached daily OHLCV S&P 500 (^SPX) price data pulled via `yfinance`, saved locally so the notebook can be re-run without re-hitting the network.
- `requirements.txt` — Python dependencies for this project.

## Stack

Python, pandas, NumPy, matplotlib, yfinance, Backtrader.

## Method

1. **Data.** Daily S&P 500 (^SPX) OHLCV data (2020-01-01 to 2026-05-01) via `yfinance`, cached to `spx_data.parquet`.
2. **Feature engineering.** Bollinger Bands computed on a 20-day moving average with a 2-standard-deviation band width (middle band = MA, upper = MA + 2σ, lower = MA − 2σ).
3. **Signal generation.** Mean-reversion logic: go long when price falls below the lower band (considered oversold), go short when price rises above the upper band (considered overbought), and exit when price reverts to the moving average.
4. **Backtesting — pass 1 (pandas).** Daily strategy returns computed directly from the signal and prior-day return, compared cumulatively against a buy-and-hold benchmark.
5. **Backtesting — pass 2 (Backtrader).** The same signal logic re-implemented as a `bt.Strategy` subclass and run through Backtrader's Cerebro engine for more realistic, broker-level execution, with Sharpe ratio, max drawdown, and total return reported via Backtrader's built-in analyzers.
6. **Evaluation.** Strategy performance compared against passive buy-and-hold on the same underlying data to assess whether the mean-reversion signal adds value.
