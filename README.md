# Markov Regime-Switching Model for Market Returns

This project fits a two-regime Markov switching model to daily S&P 500 log returns. The model assumes that the market alternates between two hidden states, each with its own mean return and volatility, and that the market moves between these states according to estimated transition probabilities. The notebook downloads 20 years of daily index data from Yahoo Finance, fits the model, and classifies each trading day as belonging to a calm or high-volatility regime.

The analysis is designed as a Jupyter notebook of ten cells that run top to bottom.

## Table of Contents

- [Background](#background)
- [What the Notebook Does](#what-the-notebook-does)
- [Requirements](#requirements)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the Notebook](#running-the-notebook)
- [Cell-by-Cell Walkthrough](#cell-by-cell-walkthrough)
- [Interpreting the Output](#interpreting-the-output)
- [Methodology](#methodology)
- [Known Limitations](#known-limitations)
- [Troubleshooting](#troubleshooting)
- [Extending the Project](#extending-the-project)
- [References](#references)
- [Disclaimer](#disclaimer)

## Background

Financial returns are not drawn from a single stable distribution. Periods of calm, steady gains alternate with periods of turbulence, sharp declines, and elevated volatility. Simple models that assume constant mean and variance miss this pattern.

A Markov regime-switching model addresses this by assuming an unobserved state variable, the regime, that follows a first-order Markov chain. Each regime has its own distribution of returns. In this notebook:

- **Regime 0 and Regime 1** are the two hidden states. Which one is "calm" and which is "volatile" is determined from the estimates, not assumed in advance.
- **Transition probabilities** describe how likely the market is to remain in its current regime from one day to the next. A probability near 1 means the regime is persistent.
- **Smoothed probabilities** give the probability that each day belongs to each regime, using the full sample.

The model is:

$$r_t = \mu_{S_t} + \sigma_{S_t}\,\varepsilon_t, \qquad \varepsilon_t \sim N(0,1)$$

where $r_t$ is the scaled daily return, $S_t \in \{0, 1\}$ is the unobserved regime, $\mu_{S_t}$ and $\sigma_{S_t}$ are the regime-specific mean and standard deviation, and regime transitions follow a Markov chain with transition matrix $P$.

## What the Notebook Does

1. Installs and imports the required Python libraries.
2. Defines the ticker (`^GSPC`) and a 20-year date window ending today.
3. Downloads daily adjusted closing prices from Yahoo Finance.
4. Computes daily log returns.
5. Plots the index level and return series to check for volatility clustering.
6. Scales returns to percent and fits a two-regime Markov switching model with regime-specific means and variances.
7. Reports regime parameters, annualized means and volatilities, and the transition matrix.
8. Computes smoothed regime probabilities and expected regime durations.
9. Classifies each day into a regime and reports the share of time spent in each.
10. Plots the index on a log scale with high-volatility periods shaded.

## Requirements

- Python 3.9 or newer (tested on Python 3.13)
- Jupyter Notebook, JupyterLab, or VS Code with Jupyter support
- An internet connection, since data is downloaded at runtime
- Modest memory requirements, typically under 500 MB

Python packages:

| Package | Purpose |
|---|---|
| `yfinance` | Downloads S&P 500 price history from Yahoo Finance |
| `statsmodels` | Estimates the Markov switching model via `MarkovRegression` |
| `pandas` | Data handling and alignment |
| `numpy` | Numerical operations and log returns |
| `matplotlib` | Plotting |

## Installation

Open a terminal in the project folder and create a virtual environment (recommended):

```bash
python -m venv regime-env
source regime-env/bin/activate        # macOS or Linux
regime-env\Scripts\activate           # Windows
```

Install the dependencies:

```bash
pip install yfinance statsmodels pandas numpy matplotlib
```

Cell 1 of the notebook also installs these packages, so this step is optional if you run the notebook directly.

## Configuration

Settings are defined in Cell 2.

| Setting | Default | Description |
|---|---|---|
| `ticker` | `^GSPC` | Yahoo Finance symbol. Any liquid index or stock works, such as `^IXIC`, `^FTSE`, or `SPY` |
| `start_date` | 20 years before today | Start of the sample |
| `end_date` | Today | End of the sample |

Other parameters you may adjust:

- **`k_regimes`** in Cell 6 (default `2`). Three regimes can capture a calm state, a moderate state, and a crisis state, but they require more data and are harder to interpret.
- **`switching_variance`** in Cell 6 (default `True`). Set to `False` to allow only the mean to switch, which is a weaker model.
- **`search_reps`** in Cell 6 (default `20`). Higher values make the optimizer more likely to find the global optimum, at the cost of runtime.
- **Scaling factor** in Cell 6 (`returns * 100`). Keeps the optimizer numerically stable.

## Running the Notebook

1. Launch Jupyter from the project folder:

   ```bash
   jupyter lab
   ```

2. Run the cells in order from top to bottom. Later cells depend on variables from earlier ones, so skipping cells will cause errors.

3. The model fit in Cell 6 is the slowest step. With `search_reps=20`, expect anywhere from several seconds to a few minutes depending on your hardware.

4. After Cell 6, confirm that the regime parameters look sensible before moving on. Cells 7 through 10 depend on them.

## Cell-by-Cell Walkthrough

| Cell | Action | Output |
|---|---|---|
| 1 | Install and import libraries | None |
| 2 | Set ticker and date range | Printed ticker and period |
| 3 | Download daily adjusted closes | Shape of the price series |
| 4 | Compute log returns | Summary statistics |
| 5 | Plot prices and returns | Two-panel figure |
| 6 | Fit the 2-regime Markov switching model | Full statsmodels summary |
| 7 | Build regime parameter table and transition matrix | Mean, volatility, and persistence for each regime |
| 8 | Compute smoothed probabilities and expected durations | Expected regime durations and latest probabilities |
| 9 | Classify days and count time in each regime | Share of days in each regime |
| 10 | Plot index with shaded high-volatility periods | Log-scale price chart with shaded regions |

## Interpreting the Output

**Regime labels.** Regimes are arbitrary until you inspect them. The notebook labels the higher-variance regime as "Bear/Volatile," but always confirm this against Cell 7. A regime with a higher mean and higher variance can also exist, typically a strong rebound period.

**Mean and volatility.** The annualized mean shows the typical return in each regime, and annualized volatility shows the typical risk. A well-behaved fit usually shows a low-volatility regime with positive mean returns and a high-volatility regime with lower or negative mean returns and much larger variance.

**Transition probabilities.** The diagonal of the transition matrix gives the probability of staying in the same regime tomorrow. Values near 0.98 or higher mean long-lived regimes. Expected duration is `1 / (1 - p)` days. For example, `p = 0.98` implies an average stay of about 50 trading days.

**Smoothed probabilities.** These are the probabilities that each day belongs to each regime, using the full sample. Values near 0 or 1 indicate confident classification. Values near 0.5 indicate ambiguous periods, often around transitions.

**Share of time in each regime.** This summarizes how often the market spends time in the high-volatility state. Most long-run samples show the calm regime dominating, but the exact share depends on the sample period.

**Log-scale plot.** Shaded areas mark days classified as high-volatility. Clusters of shading around major drawdowns are a good sign that the model is capturing real market stress. Shading scattered randomly across calm periods suggests the fit is picking up noise.

## Methodology

**Returns.** Daily log returns are computed as `ln(P_t / P_{t-1})` from adjusted closing prices. Log returns are additive over time, which makes them convenient for this model.

**Scaling.** Returns are multiplied by 100 before fitting. Parameters are therefore in percent per day, not decimals, and annualized figures multiply means by 252 and volatilities by the square root of 252.

**Model.** The `MarkovRegression` class from `statsmodels` estimates a two-regime model with regime-specific constants and variances. Estimation uses maximum likelihood, with a numerical optimizer started from multiple random initial values (`search_reps`) to reduce the risk of a local optimum.

**Transition matrix.** The estimated matrix `P` gives the probability of moving from each regime to each other regime. The notebook reads it from `result.regime_transition`, which works across statsmodels versions, rather than relying on parameter names.

**Smoothed versus filtered probabilities.** Smoothed probabilities use the entire sample, including future observations, to classify each day. This is appropriate for historical analysis. Filtered probabilities use only information available up to each date and are the correct choice for any real-time or forecasting use.

## Known Limitations

- **Look-ahead bias in smoothed probabilities.** Smoothed regimes use future data, so they should not be used to generate trading signals.
- **Normality assumption.** Each regime is modeled as Gaussian. Equity returns have fat tails, so extreme days may be misclassified or distort the variance estimates. A Student-t extension is a common remedy.
- **Two regimes may be too coarse.** Real markets can move through more than two states, and a two-regime model may blend a crisis state with a moderate correction.
- **Constant transition probabilities.** The model assumes transition probabilities do not depend on macroeconomic conditions, valuations, or other variables.
- **Index composition.** The S&P 500 changes its constituents over time, so the index is not a fixed portfolio.
- **Local optima.** Even with multiple starts, the optimizer may settle on a suboptimal solution. Check that regime parameters are economically sensible.
- **Yahoo Finance reliability.** The `yfinance` library uses unofficial endpoints that can change, and data may occasionally contain errors.

## Troubleshooting

**`KeyError` on `p[1->1]` or other transition parameter names.** Use `result.regime_transition` to read the transition matrix, as Cell 7 does. Parameter naming differs across statsmodels versions.

**`KeyError` on `const[0]` or `sigma2[0]`.** Run `print(params.index.tolist())` to see the names your version uses, then update the keys in Cells 7 and 8.

**Optimizer warnings or convergence failures.** Increase `search_reps`, confirm that returns are scaled to percent, and check for missing or extreme values in the return series.

**Regimes look unrealistic, such as one regime with a tiny sample.** The optimizer may have found a degenerate solution. Try a longer sample, more `search_reps`, or a different ticker with more history.

**`yf.download` returns an empty DataFrame.** Confirm the ticker symbol on finance.yahoo.com and check your internet connection. Some index tickers require the caret prefix, as in `^GSPC`.

**`squeeze` or shape errors in Cell 3.** Recent `yfinance` versions return a multi-level column index. Use `raw["Close"].squeeze()` as shown, or select with `raw.xs("Close", axis=1, level=0)`.

**Plot in Cell 10 looks empty or misaligned.** The regime series index may not match the price index. Confirm both are `DatetimeIndex` objects, and check that the lengths align after the log-return calculation, which drops the first observation.

## Extending the Project

- **Three regimes:** Set `k_regimes=3` in Cell 6 and adjust the labeling logic in Cell 8 to handle three states.
- **Student-t innovations:** Explore alternative distributions or custom likelihood functions for fat-tailed returns.
- **Filtered probabilities:** Replace `smoothed_marginal_probabilities` with `filtered_marginal_probabilities` in Cell 8 for a real-time-compatible classification.
- **Time-varying transitions:** Use `exog_tvtp` in `MarkovRegression` to let transition probabilities depend on a predictor such as VIX changes or the yield curve.
- **Asset comparison:** Run the same model on other indices, sectors, or bond returns and compare regime persistence.
- **Export results:** Save the regime series to CSV with `regime.to_csv("regimes.csv")`.

## References

- Hamilton, J. D. (1989). A new approach to the economic analysis of nonstationary time series and the business cycle. *Econometrica*, 57(2), 357–384.
- Kim, C.-J., & Nelson, C. R. (1999). *State-Space Models with Regime Switching*. MIT Press.
- `statsmodels` Markov switching documentation: https://www.statsmodels.org/stable/markov_switching.html
- `yfinance` documentation: https://github.com/ranaroussi/yfinance

## Disclaimer

This project is for educational and research purposes. It is not investment advice, and regime classifications from historical data do not guarantee future market behavior. Smoothed probabilities in particular use future information and should not be used for live trading decisions.
