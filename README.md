# League of Legends Esports Analytics

A four-part statistical analysis of professional League of Legends match data, covering predictive modeling, time series forecasting, competitive rating systems, and hypothesis testing. The project is deliberately non-clinical, built to demonstrate breadth across core applied statistics methodologies using a single, richly documented dataset.

## Objective

Each part of this analysis addresses a distinct statistical problem, so that predictive modeling, forecasting, ranking under uncertainty, and formal hypothesis testing are all represented within a single coherent project. Professional esports match data was selected specifically for its richness: a well-documented schema of over 160 columns per game, more than a decade of match history, and no authentication or scraping required to access it.

## Data Source

Match data is provided by Oracle's Elixir (oracleselixir.com), a free, community-maintained dataset of professional League of Legends matches, updated daily and spanning every major competitive league. Data for 2024, 2025, and 2026 (year to date) was used for this analysis. Game statistics are the property of Riot Games; use of this data is subject to Riot Games' terms and policies.

Raw files are not included in this repository due to size and are not redistributed here. To reproduce this analysis, download the relevant year files from oracleselixir.com/tools/downloads and place them in `data/raw/`.

## Methodology

### Part 1: Win Prediction

A logistic regression and a gradient boosting model were trained to predict match outcome from early-game state (gold, experience, and creep score differentials at 15 minutes, plus first-objective indicators), using a chronological train/test split (2024-2025 for training, 2026 held out). Coefficient stability was checked across training years before pooling them, addressing a legitimate concern that patch changes could alter the underlying relationship between early-game state and winning.

### Part 2: Meta-Trend Forecasting

Weekly mean absolute gold differential at 15 minutes was constructed as a proxy for how decisively games are shaped by the early game, then forecast using ARIMA, exponential smoothing (ETS), and gradient boosting, compared via rolling-origin backtesting. Extreme weeks in the series were investigated directly instead of being dismissed as noise, tracing high weeks to smaller regional leagues and low weeks to major international events.

### Part 3: Cross-Region Team Rating

A sequentially updated Elo rating system was built spanning all professional matches, including domestic league play and international events, which serve as the bridge connecting regions that otherwise rarely compete against one another. The K-factor governing rating sensitivity was selected via empirical backtesting instead of a default assumption, and regional comparisons were evaluated using bootstrapped confidence intervals instead of relying on point estimates alone.

### Part 4: Patch-Impact Analysis

An anomaly identified in Part 1, a shift in the predictive importance of specific in-game objectives between training years, was traced to a specific patch boundary and formally tested using likelihood-ratio tests for objective-by-period interaction terms, with false discovery rate correction applied across the objectives tested.

## Key Findings

| Part | Method | Result |
|---|---|---|
| 1: Win Prediction | Logistic regression, XGBoost | AUC of 0.842 and 0.840 respectively on held-out 2026 data; first blood found non-predictive by both models once actual resource lead is known |
| 2: Forecasting | ARIMA(3,0,1), ETS, XGBoost | XGBoost achieved a lower RMSE (274.7 vs. 311.7 for ARIMA); the advantage concentrated in weeks surrounding major tournaments and season transitions |
| 3: Team Rating | Sequential Elo (K = 24) | The elite competitive tier was stable across a range of K-factors; apparent regional differences in raw mean rating were not statistically supported once sampling uncertainty was quantified |
| 4: Patch Impact | Logistic regression with interaction terms | A season-over-season shift in objective importance was confirmed statistically significant for both objectives tested, following FDR correction |

## Data Quality Notes

Several data-integrity issues were identified and resolved over the course of this analysis. Each is documented explicitly below:

- A duplicated column ("dragons (type unknown)"), an artifact specific to partial-completeness games, was identified and removed.
- The dataset's "partial" versus "complete" completeness flag was found to refer specifically to the presence of minute-by-minute timeline statistics; final results, per-player stats, and team objective counts are unaffected and fully present in both groups.
- A restructuring of the Americas competitive region (LCS, LLA, and CBLOL consolidating into a unified LTA system) was identified through team roster overlap and accounted for in the team rating model.
- A potential floating-point collision in patch version numbers (for example, 14.1 and 14.10 representing the same numeric value if stored without zero-padding) was checked directly against the raw data and confirmed not to affect this dataset, which uses zero-padded patch notation.

## Repository Structure

```
league-of-legends-modeling/
├── data/
│   └── raw/                  # gitignored — see Data Source above
├── lol_analysis.Rmd
└── README.md
```

## Usage

1. Download the relevant year files from Oracle's Elixir into `data/raw/`.
2. Open `lol_analysis.Rmd` in RStudio.
3. Install required packages: `install.packages(c("readr", "purrr", "tidyr", "dplyr", "xgboost", "pROC", "lubridate", "tsibble", "zoo", "ggplot2", "fable", "feasts"))`
4. Knit the document.

## Limitations

- The analysis pools all competitive leagues together; smaller regional leagues and elite international leagues are not distinguished except where explicitly investigated in Parts 2 and 3.
- The win prediction model is limited to early-game (15-minute) state and does not incorporate full-game dynamics.
- Regional team ratings reflect a modest number of teams per region (10 to 23), and regional comparisons carry meaningful uncertainty, as demonstrated by the overlapping bootstrap confidence intervals in Part 3.
- The patch-impact analysis examines two objectives identified from a specific prior anomaly and does not constitute an exhaustive scan across all measured in-game variables.

## References

- Elo, A. (1978). *The Rating of Chessplayers, Past and Present.* (Elo rating system methodology)
- Hyndman, R.J., & Athanasopoulos, G. *Forecasting: Principles and Practice.* (ARIMA and ETS methodology; `fable` and `feasts` packages)
- Oracle's Elixir (oracleselixir.com), data source. Game statistics are the property of Riot Games; use of this data is subject to Riot Games' terms and policies.
