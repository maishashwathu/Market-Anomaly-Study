# Indian Market Anomaly Study

A statistical research pipeline for detecting unusual market behavior across **Nifty 50, Nifty Midcap 150, and Nifty Smallcap 250**. It combines rolling feature Z-scores, multivariate Mahalanobis distance, cross-index return divergence, and lead–lag regime analysis.

**Research manuscript:** [A Multivariate Approach to Market Anomaly Detection Using Nifty Indices](paper/market_anomaly_manuscript.pdf), by **Shashwat Jain and Kartik Soni**. This is an **unpublished manuscript**, not a peer-reviewed publication.

## Project highlights

- Fetches daily index OHLCV data through the FYERS API.
- Engineers daily returns, intraday high–low spread, and 21-day realized volatility.
- Detects unusual individual features and joint multivariate deviations.
- Compares large-, mid-, and small-cap return spreads on aligned dates.
- Explores small-cap versus large-cap leadership and a heuristic warning signal.
- Includes **14 figures extracted from the notebook's saved outputs** and the complete manuscript.

## Recorded results

The following counts come from the saved analysis notebook. They have not been independently reproduced; the original market CSV files are not supplied in the source folder.

| Index | Rows after feature NaN removal | Recorded Mahalanobis flags |
|---|---:|---:|
| Nifty 50 | 2,464 | 115 |
| Nifty Midcap 150 | 1,667 | 66 |
| Nifty Smallcap 250 | 1,667 | 77 |

The detector uses a 126-observation trailing baseline and the threshold **sqrt(chi-square quantile at 0.999, df=3) = 4.0331**. The first 126 rows have no distance score, and singular covariance windows can also be skipped. Counts therefore do not share identical sample coverage and should not be compared as rates.

![Saved Mahalanobis anomaly scores for the three Nifty indices](assets/mahalanobis_overview.png)

The original figure labels threshold exceedances “Confirmed Anomaly.” This means a detector flag, not an externally validated financial event.

## How the pipeline works

| Stage | Method in the notebook |
|---|---|
| Acquisition | FYERS daily history requests in date chunks, starting 1 April 2016 and ending on the execution date |
| Features | Close-to-close return, high minus low, 21-observation rolling return standard deviation |
| Univariate anomalies | Absolute rolling Z-score above 3; 21-observation baselines for returns/spread, 36 for volatility |
| Multivariate anomalies | Mahalanobis distance using the prior 126 observations of the three raw features |
| Cross-index divergence | Pairwise daily return differences, 21-observation Z-scores, absolute threshold above 3 |
| Lead–lag analysis | Prior 21 observations; lags −5 to +5; choose maximum signed correlation; smooth dominant lag over 10 observations |
| Warning heuristic | Positive smoothed lag and small-cap 21-observation cumulative return greater than large-cap return |

Z-score baselines include the current observation. Mahalanobis baselines and lead–lag estimation use trailing observations excluding the current day. The warning rule also uses cumulative returns through the current day, so it is an end-of-day descriptive signal.

This project does not train a neural network or save model weights. Its outputs are engineered features, anomaly scores, plots, and exploratory signals.

## Selected visual results

### Cross-capitalization divergence

![Large-cap versus small-cap return spread and rolling Z-score](assets/divergence_large_small.png)

The spread is large-cap return minus small-cap return. Positive values indicate relative large-cap outperformance; negative values indicate relative small-cap outperformance.

### Lead–lag regime and warning signal

![Saved lead–lag estimate and heuristic warning signals](assets/lead_lag_signals.png)

This is the original exploratory visualization. “Bubble Warning” and “Healthy” are labels used by the implementation, not validated diagnoses. No predictive accuracy, trading backtest, or independent bubble ground truth is supplied.

See the [full gallery of all 14 figures](docs/figures.md) and [saved execution log](results/saved_execution_log.txt).

## Research paper

[Read or download the unpublished manuscript (PDF)](paper/market_anomaly_manuscript.pdf).

The original PDF is included unchanged with both authors credited. [Research notes](docs/research_notes.md) explain differences between manuscript descriptions and notebook implementation, including sample coverage, rolling windows, and signed versus absolute correlation selection.

## Run the project

Use Python 3.10 or 3.11. Dependencies are suggested ranges, not a locked reconstruction of the original environment.

```powershell
git clone https://github.com/maishashwathu/Market-Anomaly-Study.git
cd Market-Anomaly-Study
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
jupyter lab
```

### 1. Acquire data

Supply your FYERS application credentials as environment variables in the terminal that launches Jupyter:

```powershell
$env:FYERS_CLIENT_ID = "your-app-id"
$env:FYERS_SECRET_KEY = "your-app-secret"
$env:FYERS_REDIRECT_URI = "your-registered-redirect-uri"
```

Run `Get_Data_Fyers.ipynb` and complete its interactive authorization flow. Tokens stay in memory and are not printed. The notebook writes these files in the repository root:

- `NIFTY_50_EOD_Features.csv`
- `NIFTY_MIDCAP_150_EOD_Features.csv`
- `NIFTY_SMALLCAP_250_EOD_Features.csv`

A FYERS account, application, and appropriate data access are required. The acquisition code is provided from the original project and has not been tested against the current API. Failed chunks are reported and skipped; check coverage before analysis.

### 2. Analyze

Run `Anomaly.ipynb` in order from the repository root. It writes analyzed features, divergence scores, lead–lag signals, and the combined Mahalanobis plot into `Market_Anomaly_Data_Updated/`.

Machine-specific paths were replaced with paths relative to that output directory, and input dates are sorted. Statistical formulas are preserved. Saved notebook outputs and extracted figures remain from the original execution, not a rerun of the updated source.

The date configuration cell in the analysis notebook does not filter the input CSVs. To reproduce a fixed study period, filter all three inputs explicitly and record their actual date ranges.

### 3. Extract saved figures without data access

```powershell
python scripts/extract_notebook_outputs.py
```

This standard-library utility extracts embedded PNGs and text output without executing either notebook, downloading data, or authorizing FYERS.

## Repository structure

| Path | Purpose |
|---|---|
| `Get_Data_Fyers.ipynb` | Interactive FYERS daily OHLCV acquisition |
| `Anomaly.ipynb` | Full statistical analysis pipeline with saved outputs |
| `assets/` | Fourteen original embedded figures |
| `paper/market_anomaly_manuscript.pdf` | Unpublished coauthored research manuscript |
| `docs/figures.md` | Full visual results gallery |
| `docs/research_notes.md` | Implementation details and manuscript discrepancies |
| `results/` | Extracted logs and recorded anomaly-count table |
| `scripts/extract_notebook_outputs.py` | Reproducible saved-output extraction |
| `requirements.txt` | Suggested Python dependencies |

## Limitations and next steps

- Original CSVs are absent, so dates, event counts, and signal episodes cannot be independently recomputed here.
- Market observations are dependent and often heavy-tailed. Nominal Z-score/chi-square thresholds are heuristic cutoffs, not calibrated empirical false-alarm probabilities.
- Lead–lag selection tests several shifts in short overlapping windows without a significance test. It does not establish causality.
- High–low index spread is a price-range proxy, not a measured bid–ask spread or direct liquidity estimate. Index volume is downloaded but not used in the detector.
- Future work: consistent sample coverage, lagged-only Z-score baselines, robust covariance, sensitivity analysis, grouped event validation, and walk-forward evaluation of any predictive claim.

## Attribution and licensing

Research by **Shashwat Jain and Kartik Soni**. No open-source code or manuscript license has been selected. Raw market data is excluded; obtain it through your own authorized provider access and follow its terms.
