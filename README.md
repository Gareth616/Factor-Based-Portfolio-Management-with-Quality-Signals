# Factor-Based-Portfolio-Management-with-Quality-Signals
This project built and backtested a 130/30 long-short equity strategy based on the AQR "Quality" factor (Asness et al., 2019) on a U.S. equity universe from 2005 to 2022, growing cumulative returns roughly 6x net of 0.2% round-trip transaction costs. Conducted Information Coefficient (Spearman) analysis on 1980–2022 monthly data to quantify the predictive power of the Quality factor and its three sub-components (Profitability, Growth, Safety), finding Profitability to be the strongest and most persistent predictor while documenting a long-run decline in the forecasting power of Quality and Safety. Optimized the strategy's portfolio breadth (100/250/400 stocks) and rebalancing frequency (1/3/6 months) through a Python grid search, identifying a 100-stock, monthly-rebalanced configuration as optimal with a Sharpe ratio of 0.83, and further evaluated how the factor's predictive power responds to the macro regime — inflation, interest rates, unemployment, and consumer sentiment — using rolling correlations and regime-conditional IC analysis. Finally, designed and backtested two original factor-combination methods, an exponentially IC-weighted blend and a momentum-weighted blend of the three sub-factors, both of which outperformed AQR's equal-weighted benchmark on cumulative return.

## Data

All data is monthly and covers a sample of U.S. stocks. **Files are not included in this repository** and must be placed in the project root before running the notebook.

| File | Description |
|---|---|
| `QUALITY.zip` | Monthly **Quality** factor values (composite of the three sub-components) |
| `PROFIT.zip` | Monthly **Profitability** sub-factor |
| `GROWTH.zip` | Monthly **Growth** sub-factor |
| `SAFETY.zip` | Monthly **Safety** sub-factor |
| `Prices.zip` | Monthly adjusted stock prices, from 1980 |
| `Market_cap.zip` | Monthly market capitalization, from 1980 |
| `Macro.csv` | Monthly macroeconomic indicators |

All factor values (**Quality**, **Profit**, **Growth**, **Safety**) are **already cross-sectionally normalized and winsorized**.

`Macro.csv` contains the following indicators:

| Column | Description |
|---|---|
| `INFL` | Annual U.S. inflation (trailing 12 months) |
| `10YTR` | Yield to maturity, **10-year** U.S. Government Bond |
| `1YTR` | Yield to maturity, **1-year** U.S. Government Bond |
| `UNRATE` | U.S. unemployment rate |
| `UMCSENT` | University of Michigan Consumer Sentiment Index |

## `apmodule` Reference

Custom course library providing the portfolio construction, backtesting, and factor-analysis functions used throughout the notebook. Only `ls_backtesting()` and `diagnostics()` are called directly in this project, but the module includes several related tools.

### Portfolio Strategies

| Function | Description |
|---|---|
| `rank_strategy(factor_data, market_cap, N=100)` | Market-cap-weighted, **long-only** portfolio in the top `N` stocks by factor rank; also returns the full-market benchmark. |
| `mn_strategy(factor_data, market_cap, N=100)` | **Market-neutral** portfolio: long the top `N` stocks, short the bottom `N`, each leg separately cap-weighted to sum to its own 100%. |
| `ls_strategy(factor_data, market_cap, N=100, active=0.3)` | **Long-short (130/30-style)** portfolio: market-cap weight plus an active over/underweight of `active/N` in the top/bottom `N` stocks. This is what powers `ls_backtesting()`. |

### Performance Diagnostics

| Function | Description |
|---|---|
| `diagnostics(port_ret)` | Given monthly return series, returns annualized **Mean Return**, **St. Dev.**, **RR Ratio** (return/risk), **% Positive** months, **Worst/Best Month**, and **Max Drawdown**. |

### Backtesting Functions

| Function | Description |
|---|---|
| `backtesting(...)` | Backtests the long-only `rank_strategy`. Returns `(monthly_returns, turnover, composition, performance)`. |
| `mn_backtesting(...)` | Backtests the market-neutral `mn_strategy`. |
| `ls_backtesting(...)` | Backtests the long-short `ls_strategy` (the **130/30** strategy used in this project). Handles rebalancing frequency, transaction costs, and drift of weights between rebalances. |

All three share the same signature: `info_signal`, `prices`, `market_cap`, `start`, `end`, `frequency` (rebalance interval in months), `t_cost` (round-trip cost), `N` (stocks per leg), and — for `ls_backtesting` — `active` (the active weight parameter, e.g. `0.3` for a 130/30 strategy).

### Optimal Portfolio Weights

| Function | Description |
|---|---|
| `portfolio_performance(weights, mean_returns, cov_matrix)` | Computes portfolio return and standard deviation from weights, expected returns, and a covariance matrix. |
| `neg_sharpe_ratio(...)` | Negative Sharpe ratio, used as the objective function for optimization (`scipy.optimize` minimizes). |
| `max_sharpe_ratio(mean_returns, cov_matrix, risk_free_rate)` | Solves for the weights (bounded 0–1, summing to 1) that maximize the Sharpe ratio via `SLSQP`. |
| `optimized_alpha_model(active_returns)` | Wraps `max_sharpe_ratio` with annualized mean/covariance to find optimal combination weights across a set of active-return streams (e.g. multiple factors). |
| `walkforward_alpha_model(active, T)` | Rolls `optimized_alpha_model` forward over a `T`-period window, producing a time series of optimal weights (out-of-sample-style, walk-forward optimization). |

### Predictability Analysis

| Function | Description |
|---|---|
| `ic_analysis(signal, prices, frequency='monthly')` | Computes the **Information Coefficient** (Spearman correlation with future returns) at monthly/quarterly/annual frequency, and prints average IC, % positive/negative periods, and a t-test on the IC series. |
| `quantile_analysis(signal, prices, n_bins=4)` | Buckets stocks into `n_bins` quantile portfolios by signal each period and reports the return (and diagnostics) of each bucket plus an active/neutral spread — **not used in this project**, since the assignment specifies IC-based analysis instead. |

### Machine Learning Analysis

| Function | Description |
|---|---|
| `ml_analysis(prediction, prices)` | Analogous to `quantile_analysis`, but groups by discrete ML-model prediction labels rather than quantile bins, comparing the best- vs. worst-predicted groups. |
