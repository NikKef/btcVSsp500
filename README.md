# BTC versus SPY historical performance

This project compares Bitcoin with SPY, an ETF tracking the S&P 500, using shared trading dates. It studies investment growth, periodic returns, volatility, risk-adjusted performance, correlation, contribution timing, and sensitivity to the starting date.

The notebooks explain the calculations and can be run cell by cell. The comprehensive report combines the figures, formulas, results, and limitations. This is a descriptive historical study, not a forecast.

## Data snapshot

The reported results use the following snapshot.

| Property | Value |
| --- | --- |
| First shared observation | December 27, 2013 |
| Last shared observation | November 12, 2026 |
| Price observations | 3,239 |
| Paired return observations | 3,238 |
| Elapsed years | 12.876112 |
| Complete months used in monthly statistics | January 2014 through October 2026, 154 months |
| Currency | USD |
| BTC input | Daily closing price from Alpha Vantage, filtered to positive-volume observations |
| SPY input | Adjusted closing price downloaded through yfinance |

The assets are joined on shared dates before calculating returns. BTC weekend movement is included in the next shared-date return. The providers' closing timestamps may differ, so matching dates does not guarantee precisely synchronized prices.

SPY adjusted prices proxy dividend reinvestment. The resulting units in investment simulations are synthetic adjusted-price units rather than literal historical share quantities.

## Read the completed analysis

These links work when the generated report package is present locally.

- [Comprehensive Markdown report](reports/btc_spy_analysis/report.md) for people and LLMs
- [Browser report](reports/btc_spy_analysis/report.html) with figures and tables
- [Structured results](reports/btc_spy_analysis/analysis_data.json) with units and numeric records
- [Portable report archive](reports/btc_spy_analysis.zip)

The package contains 16 embedded figures, 27 numeric tables, the aligned input snapshot, source notebooks, assumptions, and a file-hash manifest. Its relative links remain usable when the complete folder is moved.

The repository's current `.gitignore` excludes `data/raw/`, `data/processed/`, `results/`, `scripts/`, and `reports/`. A clone alone may therefore lack the dataset, generated reports, and report compiler. Obtain the report archive or those files separately to reproduce this exact snapshot. Do not assume ignored local artifacts are published with the repository.

## 1. Set up the Python environment

Run commands from the project root. Notebook metadata records Python 3.14.7. Use the same Python series and the pinned dependencies for the closest match to the original environment.

```bash
python3.14 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m ipykernel install --user --name btc-spy --display-name "BTC vs SPY"
```

If your Python executable has a different name, substitute its path in the first command. Check its version before creating the environment.

```bash
python --version
python -m pip check
```

If dependency installation fails, inspect the reported package and Python compatibility issue. Do not ignore an unsuccessful install and continue with an unknown environment.

## 2. Choose the reproduction path

### Path A: reproduce the saved results offline

Use this path to compare your calculations with the numbers in this README. No API credentials or new downloads are needed after dependencies are installed.

The expected input is `data/processed/btc_spy_daily.csv`. If that file is already present with the saved snapshot, leave it in place. Otherwise, extract the report archive and copy its bundled input into the project.

If the extracted package is under `reports/btc_spy_analysis`, run:

```bash
mkdir -p data/processed results
touch .env
cp reports/btc_spy_analysis/data/btc_spy_daily.csv data/processed/btc_spy_daily.csv
```

The copy command replaces an existing input file, so preserve a different local dataset before using it. Adjust the source path if you extracted the package elsewhere.

The empty `.env` file is required because notebooks 03 through 05 locate the project root by searching for that file. It does not need an API key for offline analysis. Notebooks 06 through 10 locate the root using the processed input file instead.

Check the restored dataset:

```python
import pandas as pd
import numpy as np

prices = pd.read_csv(
    "data/processed/btc_spy_daily.csv",
    index_col="date",
    parse_dates=True,
)
assert prices.columns.tolist() == ["price_btc", "price_spy"]
assert prices.index.is_monotonic_increasing
assert prices.index.is_unique
assert np.isfinite(prices.to_numpy()).all()
assert prices.gt(0).all().all()
assert len(prices) == 3239
assert prices.index[0] == pd.Timestamp("2013-12-27")
assert prices.index[-1] == pd.Timestamp("2026-11-12")
```

Skip notebooks 01 and 02 for this path. They acquire and rebuild inputs and are unnecessary for reproducing calculations from the aligned snapshot.

### Path B: acquire and prepare a fresh dataset

Use this path to refresh the analysis. New downloads can change the date range and adjusted prices, so they are not expected to reproduce the saved values exactly.

1. Create a local `.env` file in the project root.
2. Set `ALPHA_VINTAGE_API_KEY` to your Alpha Vantage key. The spelling `VINTAGE` matches the existing notebook variable and must be retained unless you also change the notebook.
3. Run notebook 01 to download BTC and SPY.
4. Normalize the BTC column names before saving its processed CSV.
5. Run notebook 02 to filter BTC observations, select closing prices, and join shared dates.
6. Inspect the resulting range, schema, and missing values before continuing.

The local `.env` entry has this form:

```text
ALPHA_VINTAGE_API_KEY=your_key_here
```

Do not include credentials in notebooks, reports, or commits.

**BTC schema detail:** notebook 01 preserves provider names such as `4. close` and `5. volume`, while notebook 02 expects `close` and `volume`. The existing processed snapshot already has normalized names. For a fresh download, add the following after numeric conversion in notebook 01 and before writing `btc_usd_daily.csv`:

```python
btc = btc.rename(columns={
    "1. open": "open",
    "2. high": "high",
    "3. low": "low",
    "4. close": "close",
    "5. volume": "volume",
})
assert {"close", "volume"}.issubset(btc.columns)
```

If the provider returns a different schema or an API error, inspect the response and adapt explicitly. Do not replace missing prices with zero or suppress the error.

For a new date range, update the fixed 2026 YTD labels in notebook 05 and the snapshot-specific descriptions in the report compiler. Notebook 05 excludes the final observed month from complete-month summaries. That is appropriate for the saved mid-November snapshot, but requires review if your new data ends at a completed month boundary.

## 3. Run the analysis notebooks

Start Jupyter from the project root:

```bash
jupyter lab
```

Select the **BTC vs SPY** kernel. Restart the kernel for each notebook and run its cells from top to bottom. Do not depend on variables left over from another notebook or an earlier execution.

For the saved snapshot, run notebooks 03 through 10 in this order:

| Notebook | Calculations and outputs |
| --- | --- |
| [03 Investment growth](notebooks/03_investment_growth.ipynb) | Growth of $100, profit, total return, linear and logarithmic charts |
| [04 Annualised returns](notebooks/04_annualised_returns.ipynb) | CAGR using actual elapsed years |
| [05 Periodic returns](notebooks/05_periodic_returns.ipynb) | Calendar-year and monthly returns, positive-month rates, monthly comparisons |
| [06 Volatility](notebooks/06_volatility.ipynb) | Annualized volatility and rolling 63-return volatility |
| [07 Risk-adjusted returns](notebooks/07_risk_adjusted_returns.ipynb) | Sharpe, Sortino, full-period Calmar, and drawdowns |
| [08 Correlation](notebooks/08_correlation.ipynb) | Full-period Pearson correlation and rolling 63-return and 252-return correlations |
| [09 Contribution strategies](notebooks/09_contribution_strategies.ipynb) | Equal-budget lump sum versus monthly deployment, XIRR, drawdowns, fixed-horizon scenarios |
| [10 Start-date sensitivity](notebooks/10_start_date_sensitivity.ipynb) | Later entries with a common endpoint and summaries excluding earlier starts |

Notebook 03 creates `results/`. Creating that directory during setup also allows notebooks 04 and 05 to save figures when run separately.

CSV tables, PNG figures, assumptions files, and individual HTML reports are written to `results/`. Rerunning cells overwrites matching output filenames. Preserve a copy if you want to compare different settings.

### Settings that affect results

| Setting | Saved baseline |
| --- | --- |
| Initial investment in notebook 03 | $100 |
| Annualization frequency | 252 shared return observations |
| Risk-free rate for Sharpe | 0% per year |
| Minimum acceptable return for Sortino | 0% per year |
| Full-period Calmar window | Entire sample |
| Budget per contribution strategy | $12,000 available upfront |
| Interest on idle cash | 0% |
| Fees, taxes, and spreads | Excluded |
| Fractional units | Allowed |
| Purchase timing | First available shared date of each month, at its recorded closing price |
| Notebook 09 rolling horizons | 1, 3, and 5 calendar years |
| Notebook 10 endpoint | Last available observation, November 12, 2026 in the snapshot |
| Notebook 10 minimum remaining history | One calendar year |

Changing a setting requires rerunning its dependent cells. Mixing exports from different datasets or assumptions can make the compiled report inconsistent.

## 4. Compile the comprehensive report

After running notebooks 03 through 10, run:

```bash
python scripts/build_analysis_report.py
```

If the compiler is absent from a clone, the report package includes a copy at `source/build_analysis_report.py`. Restore it to the project's `scripts/` directory before running it:

```bash
mkdir -p scripts
cp reports/btc_spy_analysis/source/build_analysis_report.py scripts/build_analysis_report.py
python scripts/build_analysis_report.py
```

The compiler expects the original project layout, the processed input, the notebook files, `requirements.txt`, and the complete `results/` exports. It does not download data or run notebooks automatically.

It writes the following:

```text
reports/
  btc_spy_analysis.zip
  btc_spy_analysis/
    report.md
    report.html
    analysis_data.json
    manifest.json
    data/
    figures/
    source/
```

The compiler recalculates core metrics from the input and checks their agreement with saved exports. It also verifies that report links resolve. A failed comparison usually means the dataset, assumptions, or saved results are inconsistent. Investigate and rerun the relevant notebooks instead of bypassing the check.

The report compiler currently includes narrative tied to the saved snapshot. Rebuilding it with new market data requires updating those dates, counts, example values, and interpretations as well as regenerating tables.

For LLM use, provide `report.md` together with `analysis_data.json`. Include the complete archive when the model also needs figures and source files. The manifest records hashes of bundled files. It identifies the snapshot but does not independently validate provider prices.

## 5. Calculations and units

Let `P[t]` be the observed price and `T` be elapsed calendar days divided by 365.25.

| Metric | Definition | Meaning |
| --- | --- | --- |
| Simple return | `P[t] / P[t-1] - 1` | Change between shared observations |
| Investment value | `initial_budget * P[t] / P[0]` | Buy-and-hold wealth |
| CAGR | `(P[end] / P[start]) ** (1/T) - 1` | Constant annual compounded rate matching the endpoints |
| Annualized volatility | `sample_std(returns) * sqrt(252)` | Fluctuation of returns, including gains and losses |
| Sharpe | `mean(excess_returns) / sample_std(excess_returns) * sqrt(252)` | Return above the risk-free benchmark per unit of volatility |
| Downside deviation | `sqrt(mean(min(return - target, 0) ** 2))` | Root-mean-square shortfall across all observations |
| Sortino | `mean(return - target) / downside_deviation * sqrt(252)` | Return above the target per unit of downside deviation |
| Drawdown | `wealth / cumulative_max(wealth) - 1` | Loss relative to the running peak |
| Full-period Calmar | `CAGR / abs(maximum_drawdown)` | Compounded growth relative to the worst observed drawdown |
| Pearson correlation | `cov(BTC_returns, SPY_returns) / (std(BTC_returns) * std(SPY_returns))` | Linear co-movement of returns |
| Monthly advantage | `monthly_final_wealth / lump_sum_final_wealth - 1` | Relative ending-wealth difference between funding strategies |

Sample standard deviation uses `ddof=1`. Annual risk-free and target rates are converted to interval rates using `(1 + annual_rate) ** (1/252) - 1`.

XIRR solves for an annual rate that grows each dated contribution to the ending invested value. It measures the invested sleeve. Whole-account CAGR includes the cash that was available but not yet invested. These measures answer different questions.

Watch the storage conventions:

- Notebooks 03 through 05 commonly store returns as percentage numbers. A value of `5` means 5%.
- Later notebooks generally store returns, volatility, drawdowns, and win rates as decimals. A value of `0.05` means 5%.
- Sharpe, Sortino, Calmar, and correlation are dimensionless.
- A return difference of 8% minus 3% is 5 percentage points.
- Structured report tables include unit notes. Unavailable values are represented as null rather than zero.

## 6. Expected results and interpretation

These rounded values are checks for the saved snapshot and baseline assumptions, not targets for a fresh download.

| Metric | BTC | SPY |
| --- | ---: | ---: |
| Final value of $100 | $10,013.70 | $510.92 |
| CAGR | 43.01% | 13.50% |
| Annualized volatility | 66.25% | 17.13% |
| Maximum observed drawdown | -82.90% | -33.72% |
| Sharpe | 0.875 | 0.827 |
| Sortino | 1.307 | 1.165 |
| Full-period Calmar | 0.519 | 0.401 |
| Mean complete-month return | 5.10% | 1.15% |
| Positive complete months | 54.55% | 68.18% |

BTC had much higher compounded growth over the full period, accompanied by substantially greater volatility and deeper losses from peaks. The gap in Sharpe ratios is far smaller than the gap in raw growth. No statistical significance test was performed on those ratio differences.

SPY rose in a larger fraction of months even though BTC had a higher average monthly return. How often an asset gains and how large its gains and losses are must be considered separately.

Full-period return correlation was **0.221854**. The latest 63-return and 252-return correlations were about **0.481** and **0.465**. The relationship changed over time. A low full-period correlation does not establish that BTC protects against equity-market losses.

### Contribution timing

For $12,000 available at the beginning, lump sum invests immediately. Monthly deployment uses 156 equal purchases of about $76.923077 over the full sample.

| Asset | Lump-sum ending wealth | Monthly ending wealth |
| --- | ---: | ---: |
| BTC | $1,201,645 | $599,190 |
| SPY | $61,310 | $33,075 |

Monthly deployment produced smaller observed whole-account drawdowns in this full-period comparison but lower ending wealth. Its invested-sleeve XIRR can still be higher because that calculation excludes the opportunity cost of idle cash.

Across 96 overlapping five-year starting-date scenarios per asset, lump sum won 85.42% for BTC and 100% for SPY. Those frequencies describe this historical sample. They are not independent probabilities of future success.

Investing an available lump sum is a different decision from investing monthly income as it arrives. The latter does not make future income available on the initial date.

### Starting-date sensitivity

Notebook 10 tests 144 monthly starts with a common endpoint, producing 576 asset and strategy combinations. The budget remains fixed, so later starts have fewer and larger installments.

| Start | BTC lump sum | BTC monthly | SPY lump sum | SPY monthly |
| --- | ---: | ---: | ---: | ---: |
| January 2018 | $59,130 | $53,687 | $38,703 | $24,467 |
| January 2022 | $19,027 | $20,506 | $20,317 | $18,325 |
| January 2024 | $19,647 | $11,794 | $19,904 | $15,187 |
| January 2025 | $9,115 | $10,370 | $15,888 | $13,919 |

Actual entries use the first shared trading date of the stated month. BTC's full-period advantage does not persist for every later start or contribution strategy.

Common-endpoint comparisons mix entry-date effects with different holding durations. Notebook 09's fixed-horizon comparisons provide a complementary view. Neither design alone identifies a causal effect of choosing a particular starting date.
