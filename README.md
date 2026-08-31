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
