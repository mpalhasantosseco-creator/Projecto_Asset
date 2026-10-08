# Asset Pricing & Portfolio Management — Group Project (2026/27)

NOVA IMS · Postgraduate Programme in Data Science for Finance

Goal: study the empirical properties of stock returns (stylised facts) and compare
8 portfolio construction strategies with a *walk-forward* backtest of 100 experiments.

## Repository structure

| Folder / file | Contents |
|---|---|
| `notebooks/01_data_collection.ipynb` | Universe (23 S&P 500 stocks + SPY + ^GSPC), download from Yahoo Finance, trading calendar, missing values, daily/weekly/monthly log returns |
| `notebooks/02_walmart_eda.ipynb` | Exploratory analysis of Walmart (WMT): prices, histograms, Q-Q plots, quantiles, VaR, ACF, comparison with SPY/^GSPC. Also available as `.html` |
| `notebooks/03_stylised_facts_garch.ipynb` | The 6 stylised facts of WMT at daily, weekly and monthly frequency (Ljung-Box, Jarque-Bera, skewness with bootstrap, ARCH-LM, GJR-GARCH, standardised residuals), with SPY as a comparison. Also available as `.html` |
| `notebooks/04_strategies_backtest.ipynb` | The 8 strategies, illustration with 23 stocks, backtest of 100 experiments, metrics, robustness (costs, quarterly rebalancing, Markowitz λ, periods, stocks) |
| `notebooks/data/` | Prices and returns (CSV) produced by notebook 01 (`clean_prices.csv`, `daily_log_returns.csv`, `weekly_log_returns.csv`, `monthly_log_returns.csv`); the daily WMT history from Yahoo Finance used by notebook 02 (`walmart_history.csv`: open, high, low, close, adjusted close, volume, dividends and splits; it was saved during the original extraction and no notebook re-creates it, so do not delete it); and the risk-free rate (FRED DGS3, `risk_free_DGS3.csv`, created by notebook 04) |
| `figures/` | All charts (PNG). Those from notebook 02 have no prefix, those from notebook 03 start with `03_`, those from notebook 04 with `04_` |
| `results/` | All tables as CSV and as Excel (`03_stylised_facts.xlsx`, `04_strategies_backtest.xlsx`) |
| `reports/` | Report |

## How to reproduce

```bash
pip install -r requirements.txt
```

You also need Jupyter to run the notebooks, for example Anaconda or `pip install notebook` (this also installs `nbformat`
and `nbconvert`, which the first cell of notebook 02 uses to export the notebook to HTML). If you use VS Code instead,
also run `pip install ipykernel nbconvert`.

Run the notebooks **in order**, from the `notebooks/` folder (the paths in the code are relative to it:
`data/`, `../figures/` and `../results/`):

1. `01` — downloads the prices from Yahoo Finance (needs internet). So that the data used in the report do not change,
   the cell that saves the CSV files is disabled; the CSV files in `notebooks/data/` are the ones from the original extraction.
2. `02` and `03` — only read the CSV files.
3. `04` — on the first run it downloads the **DGS3** series from FRED and saves `data/risk_free_DGS3.csv`
   (later runs read the local file). It takes about 2–3 minutes.
   - **If the download fails** (university or company proxy/firewall): open
     <https://fred.stlouisfed.org/graph/fredgraph.csv?id=DGS3> in the browser, save the CSV as
     `notebooks/data/risk_free_DGS3.csv` and run the cell again. The file must contain the full series (starting before 2010).
   - Before running, **close in Excel** the files in `results/` (on Windows, Excel locks them → `PermissionError`).
   - At the end, section 13 prints an automatic summary with the numbers from your run; section 14 has the conclusions
     and the recommendation (check the sentences about the Sharpe ratio against those numbers).
   - To generate the HTML: `jupyter nbconvert --to html 04_strategies_backtest.ipynb`.

To copy tables into the report, use the **.xlsx** files (the CSV files are a plain-text copy).

## Reproducibility

- **The data CSV files are committed and are the source of truth.** Notebooks 02, 03 and 04 always read the files in
  `notebooks/data/`. Re-running notebook 01 downloads the prices again from Yahoo Finance, whose adjusted prices
  (*Adj Close*) change over time (they are recalculated after every new dividend or split). That is why the cell that
  saves the CSV files in notebook 01 is disabled (kept as text): re-running it does not overwrite the data.
- **Random seeds** (every random draw in the project has a fixed seed):
  - notebook 02: `np.random.seed(123)` (simulated Gaussian white noise compared with the WMT returns);
  - notebook 03: `np.random.seed(2026)` (bootstrap of the skewness);
  - notebook 04: `np.random.seed(2026)` (`SEED`, the periods and stocks of the 100 experiments) and `SEED + 1` = 2027
    (only for the new draw of 12 stocks in section 10.6).

  All of them use the legacy NumPy generator (`np.random.seed` + `np.random.randint` / `choice` / `normal`, i.e.
  `RandomState`). NumPy keeps this stream frozen, so the same seed gives the same random draws (the same periods, the
  same stocks and the same bootstrap samples) on any NumPy version and on any computer (Windows, macOS, Linux).
- **Risk-free rate:** after the first local run of notebook 04 (the one that downloads DGS3 from FRED), commit
  `notebooks/data/risk_free_DGS3.csv` (and `risk_free_DGS3_info.txt`, which records the extraction date) so that everyone
  in the group uses exactly the same risk-free series. FRED adds new days to the series, so two downloads made on different
  days are not identical. Before committing, check the line that notebook 04 prints
  (`Average rate over the period: ... | minimum: ... | maximum: ...`): with the real DGS3 series the minimum is below 0.5%
  (2020–2021) and the maximum is above 4% (2023). If the minimum and the maximum are equal, the file is not the real
  series: delete it and run the cell again.
- **Library versions:** the random draws above are identical everywhere, but the numerical optimisers (the GJR-GARCH
  fits with `arch` in notebook 03 and the SLSQP optimisations with `scipy` in notebook 04) can stop at slightly different
  points with different versions of NumPy, SciPy, pandas or arch. In a test with older versions (NumPy 1.26, SciPy 1.13,
  pandas 2.2, arch 7.0) and newer ones (NumPy 2.5, SciPy 1.18, pandas 3.0, arch 8.0), some GJR-GARCH estimates and p-values changed
  in the 3rd or 4th decimal place, and one Max Sharpe optimisation failed in one environment but not in the other, which
  changed some Max Sharpe figures slightly (turnover and effective N in the 2nd decimal place, and one value of the
  "% beats EW" table by 1 percentage point). So, to get exactly the same numbers on every computer, everyone in the
  group must install the same library versions, and the numbers in the report must all come from one single run (the one
  whose `results/` and `figures/` files are committed).

## Getting these changes into the group repository (Troniospt/Projecto_Asset)

This repository (`mpalhasantosseco-creator/Projecto_Asset`) is a GitHub fork of the group repository
`Troniospt/Projecto_Asset`, so both have the same Git history and these changes can be merged into any branch of the
group repository. Work in your local clone of the **group** repository, where `origin` points to
`Troniospt/Projecto_Asset` (if you do not have one yet: `git clone https://github.com/Troniospt/Projecto_Asset` and then
`cd Projecto_Asset`):

```bash
git remote add fork https://github.com/mpalhasantosseco-creator/Projecto_Asset   # only the first time
git fetch fork
git checkout <your branch>          # e.g. Miguel_seco
git pull origin <your branch>       # make sure your local branch is up to date first
git merge fork/main                 # fast-forward if nobody changed your branch in the meantime
git push origin <your branch>
```

- `fork/main` has these changes once the fork's pull request has been merged into its `main`
  (before that, merge `fork/claude/gallant-euler-k99apo` instead).
- Push only to your own branch, never to `main` of the group repository.
- Commit (or stash) your own local changes before the merge. Files and folders were renamed to English (the charts are
  now in `figures/`, the tables in `results/`, and every notebook and data file has an English name). Git normally
  detects the renames and carries your edits over to the new files; if it cannot, it reports a conflict and you have to
  copy your edits into the new file by hand.
- Alternative without the command line: on GitHub, open a pull request with **base repository**
  `Troniospt/Projecto_Asset` and **base** = your own branch (not `main`), and **head repository**
  `mpalhasantosseco-creator/Projecto_Asset` and **compare** = `main`.

## Data

- **Prices:** Yahoo Finance via `yfinance`, adjusted close price (*Adj Close*, includes dividends and *splits*),
  from 2010-01-04 to 2026-09-18 (extracted in September 2026; `end_date = 2026-09-21` in notebook 01). Calendar = SPY trading days.
  TSLA only has data from 2010-06-29 onwards (122 missing days at the start).
- **Risk-free rate:** 3-year US Treasury *yield*, FRED series `DGS3` (% per year); the extraction date is recorded in
  `notebooks/data/risk_free_DGS3_info.txt` (or it is the date of the manual download, if the automatic one fails).
- **SPY vs ^GSPC:** SPY is the investable ETF (market benchmark in the backtest); ^GSPC is the index, not investable and without dividends.

## Backtest conventions (notebook 04)

| Item | Choice |
|---|---|
| Experiments | 100, seed `2026` (`np.random.seed`, the same sequence on any NumPy version) |
| Each experiment | contiguous 3-year period (756 trading days) + 12 stocks drawn from the 23 eligible ones |
| Estimation / evaluation | years 1–2 / year 3 (out of sample) |
| Estimation window | rolling 1 year (252 days), past data only |
| Rebalancing | semi-annual (quarterly as a robustness test); weights drift between dates |
| Constraints | long-only, fully invested, no leverage |
| Costs | 10 bps on Σ\|Δw\| (robustness: 0, 25, 50 bps) |
| Benchmarks | Equally weighted and SPY buy & hold |
| Markowitz | risk aversion λ = 3 (sensitivity with λ = 1 and 5) |

## Mapping to the report

| Report section | Where |
|---|---|
| 1. Universe and portfolio construction | notebook 01 |
| 2.1.1 Autocorrelation | notebook 03, section 3 — `03_acf_returns_wmt.png`, `03_ljung_box_returns.csv` |
| 2.1.2 Non-normality / fat tails | notebook 03, section 4 — `03_qqplot_normal_wmt_3freq.png`, `03_normality.csv` |
| 2.1.3 Asymmetry (skewness) | notebook 03, section 5 — `03_skewness.csv` |
| 2.1.4 Volatility clustering | notebook 03, section 6 — `03_acf_abs_squared_wmt.png`, `03_rolling_volatility_wmt.png`, `03_vol_clustering.csv` |
| 2.1.5 Leverage effect | notebook 03, section 7 — `03_news_impact_curve.png`, `03_leverage_correlation.png`, `03_conditional_volatility_wmt.png`, `03_leverage_correlation.csv`, `03_gjr_garch.csv` |
| 2.1.6 Conditional non-normality | notebook 03, section 8 — `03_qqplot_std_residuals_wmt.png`, `03_std_residuals.csv` |
| Summary of the stylised facts | notebook 03, section 9 — `03_facts_summary_wmt.csv` |
| 3. Strategies (objectives and constraints) | notebook 04, sections 3–4 — `04_weights_illustration.png`, `04_efficient_frontier.png`, `04_risk_contributions_iv_vs_erc.png` |
| 1. Universe (statistics and correlations) | notebook 04, section 2 — `04_universe_summary.csv`, `04_universe_correlations.png` |
| 3. Backtest results | notebook 04, section 9 — `04_median_metrics.csv`, `04_dispersion.csv`, `04_beats_ew.csv`, `04_rankings.csv`, `04_boxplots_metrics.png` |
| Robustness (costs, quarterly, λ, period, stocks) | notebook 04, section 10 — `04_sharpe_by_cost.csv`, `04_rebalance_semiann_vs_quarterly.csv`, `04_lambda_sensitivity.csv`, `04_sharpe_by_regime.csv`, `04_subset_comparison.csv` |
| Optimisation failures | notebook 04 — `04_optimisation_failures.csv` (equal weights are used at those rebalancing dates) |
| Limitations, conclusions and recommendation | notebook 04, sections 12–14 |
