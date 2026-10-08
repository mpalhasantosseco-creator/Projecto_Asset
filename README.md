# Asset Pricing & Portfolio Management — Group Project (2026/27)

NOVA IMS · Postgraduate Programme in Data Science for Finance

Goal: study the empirical properties of stock returns (stylised facts) and compare
8 portfolio construction strategies with a *walk-forward* backtest of 100 experiments.

## Repository structure

| Folder / file | Contents |
|---|---|
| `notebooks/01_data_collection.ipynb` | Universe (23 S&P 500 stocks + SPY + ^GSPC), download from Yahoo Finance, data treatment (dividends, splits, currency, calendar, missing data), daily/weekly/monthly log returns. Also as `.html` |
| `notebooks/02_walmart_eda.ipynb` | Exploratory analysis of Walmart (WMT): prices, histograms, Q-Q plots, quantiles, VaR, ACF, comparison with SPY/^GSPC. Also as `.html` |
| `notebooks/03_stylised_facts_garch.ipynb` | The 6 stylised facts of WMT at daily, weekly and monthly frequency (Ljung-Box, Jarque-Bera, skewness with bootstrap, ARCH-LM, GJR-GARCH, standardised residuals), with SPY as a comparison; appendix (section 10) with descriptive statistics and Jarque-Bera p-values of all 25 series at the 3 frequencies. Also as `.html` |
| `notebooks/04_strategies_backtest.ipynb` | The 8 strategies, illustration with 23 stocks, backtest of 100 experiments, metrics, robustness (costs, quarterly rebalancing, Markowitz λ, periods, stocks), conclusions. **Committed without outputs**: it must be run locally (see below) |
| `notebooks/data/` | Prices and returns produced by notebook 01 (`clean_prices.csv`, `daily_log_returns.csv`, `weekly_log_returns.csv`, `monthly_log_returns.csv`) and the daily WMT history used by notebook 02 (`walmart_history.csv`: open, high, low, close, adjusted close, volume, dividends and splits). These are the frozen data of the report: do not delete them. The risk-free rate (`risk_free_DGS3.csv`) is created by notebook 04 |
| `figures/` | All charts (PNG). Those from notebook 02 have no prefix, those from notebook 03 start with `03_`, those from notebook 04 with `04_` |
| `results/` | All tables as CSV and as Excel (`03_stylised_facts.xlsx`, `04_strategies_backtest.xlsx`) |
| `reports/` | Report |

## How to reproduce

### 1. Python and the libraries

Use **Python 3.11, 3.12 or 3.13** (not 3.14 or newer: the pinned library versions have no installers for it). On a
Windows laptop with an ARM processor (e.g. Snapdragon), install the normal x64 Python. Create a separate environment so
that the exact versions in `requirements.txt` are used:

```bash
# Windows (PowerShell or cmd), inside the project folder
py -3.11 -m venv .venv
.venv\Scripts\activate

# macOS / Linux
python3.11 -m venv .venv
source .venv/bin/activate

# both
pip install -r requirements.txt
jupyter notebook
```

`requirements.txt` already installs Jupyter (`notebook`, `ipykernel`) and `nbformat`/`nbconvert`, which the first cell of
notebook 02 uses to export the notebook to HTML. In VS Code, select the `.venv` interpreter as the kernel. With Anaconda,
do not install into the `base` environment; create one first: `conda create -n appm python=3.11`, `conda activate appm`,
then `pip install -r requirements.txt`.

### 2. Run the notebooks in order, from the `notebooks/` folder

The paths in the code are relative to that folder (`data/`, `../figures/`, `../results/`).

1. `01` — downloads the prices from Yahoo Finance (needs internet). So that the data used in the report do not change,
   the cell that saves the CSV files is disabled; the CSV files in `notebooks/data/` are the ones from the original extraction.
2. `02` (about 20 seconds) and `03` (about 30 seconds) — only read the CSV files.
3. `04` (about 5 minutes) — on the first run it downloads the **DGS3** series from FRED and saves
   `data/risk_free_DGS3.csv` (later runs read the local file).
   - **If the download fails** (university or company proxy/firewall, or on macOS a `CERTIFICATE_VERIFY_FAILED` error —
     run *Install Certificates.command* in `/Applications/Python 3.x/` or download by hand): open
     <https://fred.stlouisfed.org/graph/fredgraph.csv?id=DGS3> in the browser, save the CSV as
     `notebooks/data/risk_free_DGS3.csv` (on Windows check that it is not saved as `risk_free_DGS3.csv.csv`) and run the cell
     again. The file must contain the full series (starting before 2010); the notebook stops with a clear message if the
     series does not cover the sample or is constant. Write down the download date: it is the extraction date of the rate.
   - Before running, **close in Excel** the files in `results/` (on Windows, Excel locks them → `PermissionError`).
   - At the end, section 13 prints an automatic summary with the numbers of your run; section 14 has the conclusions and
     the recommendation (check the sentences about the Sharpe ratio against those numbers).
   - Then export the HTML (`jupyter nbconvert --to html 04_strategies_backtest.ipynb`) and **commit** the executed
     notebook, its HTML, `figures/04_*`, `results/04_*`, `data/risk_free_DGS3.csv` and `data/risk_free_DGS3_info.txt`.
     That run is the reference for every backtest number in the report.

Never open and save the CSV files of `notebooks/data/` in Excel (with a Portuguese locale Excel rewrites them with `;` and
decimal commas, and the notebooks can no longer read them). To copy tables into the report, use the **.xlsx** files in
`results/`.

## Reproducibility

- **The data are frozen.** The CSV files in `notebooks/data/` are committed and are the source of truth: notebooks 02, 03
  and 04 always read them. Re-running notebook 01 would download the prices again from Yahoo Finance, whose adjusted prices
  (*Adj Close*) change over time (they are recalculated after every new dividend or split); that is why its save cell is disabled.
- **Random seeds** (every random draw has a fixed seed): notebook 02 `np.random.seed(123)` (simulated normal returns);
  notebook 03 `np.random.seed(2026)` (bootstrap of the skewness); notebook 04 `np.random.seed(2026)` (`SEED`: the periods
  and stocks of the 100 experiments) and `SEED + 1` = 2027 (only the new draw of stocks in section 10.6). They use NumPy's
  legacy generator (`np.random.seed` + `randint` / `choice` / `normal`), whose sequence NumPy keeps frozen: the periods, the
  stocks and the bootstrap samples are **identical on any NumPy version and any computer** (tested with NumPy 1.26, 2.2 and
  2.5). The simulated normal series in notebook 02 is identical up to the last floating-point digit.
- **Library versions** — what was tested (clean copies of the project, notebooks 02-04):
  - with the versions **pinned in `requirements.txt`**, Python 3.11 and Python 3.13 gave **bit-for-bit identical** results
    (all 33 tables, every Excel value, every printed number and all 29 figures; 0 optimisation failures);
  - with much newer or older libraries, notebook 04 gave the same tables at the precision shown, and in notebook 03 only a few
    GJR-GARCH estimates of the (unreliable) weekly-normal and monthly models changed in the 4th decimal place.

  So: install the pinned versions, and take every number in the report from **one** run (the committed one). A different
  operating system or processor can still change the last decimals of the GARCH estimates; that does not change any conclusion.
- **Risk-free rate:** after the first local run of notebook 04, commit `notebooks/data/risk_free_DGS3.csv` (and
  `risk_free_DGS3_info.txt`, which records the extraction date) so that everyone uses the same series (FRED adds new days to
  the series, so two downloads on different days are not identical files).

## Getting these changes into the group repository (Troniospt/Projecto_Asset)

This repository (`mpalhasantosseco-creator/Projecto_Asset`) is a GitHub fork of the group repository
`Troniospt/Projecto_Asset`; both have the same Git history, so the changes can be merged into **your own branch** of the
group repository (e.g. `Miguel_seco`). **Never merge into, or push to, `main` of the group repository.**

In your local clone of the **group** repository (if you do not have one: `git clone https://github.com/Troniospt/Projecto_Asset`
and `cd Projecto_Asset`):

```bash
git remote add fork https://github.com/mpalhasantosseco-creator/Projecto_Asset   # only the first time
git fetch fork
git switch Miguel_seco               # your own branch
git branch --show-current            # must print your branch - NEVER main
git pull origin Miguel_seco          # make sure your branch is up to date
git merge --ff-only fork/main        # before the fork's pull request is merged, use fork/claude/gallant-euler-k99apo
git push origin Miguel_seco          # push only your own branch
```

- `--ff-only` makes Git refuse instead of creating a merge if your branch has commits that the fork does not have. In that
  case run `git merge fork/main` (without `--ff-only`) and resolve the conflicts.
- Files were renamed to English. Git recognises most renames, but **not** notebooks 02 and 03 or the charts: if you edited
  `02_analise_walmart.ipynb`, `03_modelacao_garch_volatilidade.ipynb` or a chart in `graficos/` on your branch, Git reports a
  *modify/delete* conflict, and you have to copy your edits into `02_walmart_eda.ipynb` / `03_stylised_facts_garch.ipynb` by hand.
- Alternative on the GitHub website (pull request from the fork): GitHub **pre-selects `Troniospt/Projecto_Asset` : `main`**
  as the base. Always change **base** to your own branch (never `main`); head repository =
  `mpalhasantosseco-creator/Projecto_Asset`, compare = `main` (or `claude/gallant-euler-k99apo` before the fork's pull request is merged).

## Data

- **Prices:** Yahoo Finance via `yfinance` (`auto_adjust=False`, `actions=True`), adjusted close price (*Adj Close*, includes
  dividends and splits), from 2010-01-04 to 2026-09-18 (4,203 trading days). Extracted in September 2026, between the close of
  18 September 2026 (last price in the files) and 29 September 2026 (first commit of the files); `end_date = 2026-09-21` in
  notebook 01. Calendar = SPY trading days. TSLA only has data from 2010-06-29 onwards (122 missing days at the start).
- **Risk-free rate:** 3-year US Treasury yield, FRED series `DGS3` (% per year); the extraction date is recorded in
  `notebooks/data/risk_free_DGS3_info.txt` (or it is the date of the manual download).
- **SPY vs ^GSPC:** SPY is the investable ETF (market benchmark in the backtest); ^GSPC is the index, not investable and
  without dividends.

## Backtest conventions (notebook 04)

| Item | Choice |
|---|---|
| Experiments | 100, seed `2026` (`np.random.seed`, the same draws on any NumPy version) |
| Each experiment | contiguous 3-year period (756 trading days) + 12 stocks drawn from the eligible ones (23, or 22 when the period starts on or before 2010-06-29, before TSLA's first return) |
| Estimation / evaluation | years 1–2 / year 3 (out of sample) |
| Estimation window | rolling 1 year (252 days), past data only |
| Rebalancing | semi-annual (quarterly as a robustness test); weights drift between dates |
| Constraints | long-only, fully invested, no leverage |
| Optimiser | SLSQP (`ftol` 1e-10; one retry with 1e-8; if it still fails, equal weights and the failure is recorded) |
| Costs | 10 bps on Σ\|Δw\| (robustness: 0, 25, 50 bps) |
| Risk-free rate | FRED DGS3 (3-year Treasury), daily rate = annual / 252 |
| Benchmarks | Equally weighted and SPY buy & hold |
| Markowitz | risk aversion λ = 3 (sensitivity with λ = 1 and 5) |

## Mapping to the report

| Report section | Where |
|---|---|
| 1. Universe and portfolio construction | notebook 01; notebook 04, section 2 — `04_universe_summary.csv`, `04_universe_correlations.png` |
| 1. Returns of every asset (daily, weekly, monthly) | notebook 03, section 10 (appendix) — `03_descriptive_stats_all_assets.csv` |
| 2.1.1 Autocorrelation | notebook 03, section 3 — `03_acf_returns_wmt.png`, `03_ljung_box_returns.csv` |
| 2.1.2 Non-normality / fat tails | notebook 03, section 4 — `03_qqplot_normal_wmt_3freq.png`, `03_normality.csv` |
| 2.1.3 Asymmetry (skewness) | notebook 03, section 5 — `03_skewness.csv` |
| 2.1.4 Volatility clustering | notebook 03, section 6 — `03_acf_abs_squared_wmt.png`, `03_rolling_volatility_wmt.png`, `03_vol_clustering.csv` |
| 2.1.5 Leverage effect | notebook 03, section 7 — `03_news_impact_curve.png`, `03_leverage_correlation.png`, `03_conditional_volatility_wmt.png`, `03_leverage_correlation.csv`, `03_gjr_garch.csv` |
| 2.1.6 Conditional non-normality | notebook 03, section 8 — `03_qqplot_std_residuals_wmt.png`, `03_std_residuals.csv` |
| Summary of the stylised facts | notebook 03, section 9 — `03_facts_summary_wmt.csv` |
| 3. Strategies (objectives and constraints) | notebook 04, sections 3–4 — `04_weights_illustration.png`, `04_efficient_frontier.png`, `04_risk_contributions_iv_vs_erc.png` |
| 3. Backtest design | notebook 04, sections 5–7 — `04_experiment_design.png`, `04_experiment_design.csv` |
| 3. Backtest results | notebook 04, section 9 — `04_median_metrics.csv`, `04_dispersion.csv`, `04_beats_ew.csv`, `04_rankings.csv`, `04_boxplots_metrics.png` |
| Robustness (costs, quarterly, λ, period, stocks) | notebook 04, section 10 — `04_sharpe_by_cost.csv`, `04_rebalance_semiann_vs_quarterly.csv`, `04_lambda_sensitivity.csv`, `04_sharpe_by_regime.csv`, `04_subset_comparison.csv` |
| Optimisation failures | notebook 04 — `04_optimisation_failures.csv` (equal weights are used at those rebalancing dates) |
| Limitations, conclusions and recommendation | notebook 04, sections 12–14 |
