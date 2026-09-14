# Fuelling the Forecast: A Time Series Analysis of US Gasoline Supply (1991-2017)

End-to-end time series forecasting analysis of US weekly gasoline supply, aggregated to a 312-month series (Feb 1991-Jan 2017), covering exploratory diagnostics, model benchmarking, forecast evaluation on a held-out test set, and residual diagnostics for the final model.

## Overview

The dataset is `us_gasoline`: 1,355 weekly observations of US gasoline supply in millions of barrels per day, aggregated to 312 monthly observations. The goal is to understand and forecast American gasoline consumption patterns across 26 years of economic history, including the post-9/11 period, the 2008 financial crisis, and the shale revolution.

Five models were trained on data through December 2014 and evaluated on a 25-month held-out test set (Jan 2015-Jan 2017):

| Model | Test RMSE | Test MAE | Test MAPE |
|---|---|---|---|
| Mean benchmark | 0.790 | - | - |
| Seasonal naive (SNAIVE) | 0.479 | 0.403 | 4.33 |
| Auto ARIMA (0,1,1)(2,1,0)[12] | 0.285 | 0.227 | 2.47 |
| ETS(A,Ad,A) | 0.272 | - | - |
| **Manual SARIMA(1,1,1)(0,1,1)[12]** | **0.246** | **0.199** | **2.16** |

The manual SARIMA(1,1,1)(0,1,1)[12] was selected as the final model, outperforming the seasonal naive benchmark by 49% and the auto-selected ARIMA by 14%.

## Workflow

**Framing & Data**
Defines the forecasting question, confirms the series' frequency and history, and aggregates weekly data to monthly to support annual seasonal modelling.

**Exploratory Time-Series Insights**
Examines trend, seasonality, and variance via time plots, STL decomposition, and ACF/PACF. A simulated white-noise benchmark (Ljung-Box test) is used as a reference point against which the gasoline series (Ljung-Box statistic = 4472, p = 0) is shown to be strongly structured, not random. ADF and KPSS tests confirm d = 1 and D = 1 are required.

**Modelling Choices & Justification**
Five models are fitted: a mean benchmark, a seasonal naive benchmark, three ETS specifications (best: ETS(A,Ad,A)), an auto-selected ARIMA, and three manually specified SARIMA variants. The manual order, i.e., SARIMA(1,1,1)(0,1,1)[12], is guided directly by the ACF spike at lag 1 and lag 12 and the absence of a clean PACF cutoff.

**Forecast Evaluation**
All models are evaluated out-of-sample on the 25-month test window. The manual SARIMA achieves the lowest RMSE and MAE of any model tested, including the auto-ARIMA and all ETS variants.

**Diagnostics & Conclusions**
Residual diagnostics for the winning SARIMA confirm no remaining structure: Ljung-Box p = 0.276 (lag 24, dof 3), all ACF spikes within confidence bands, and an approximately normal residual distribution (Shapiro-Wilk p = 0.463). Two robustness checks, which are, a log-transformed refit (RMSE = 0.245) and a post-crisis-only training window (RMSE = 0.255), confirm the model is stable to both transformation and the 2008 structural break.

## Final Forecast

The winning model is refitted on the full series and used to forecast 24 months ahead (Feb 2017-Jan 2019):

- Summer peaks: ~9.5-9.7 million barrels/day (Jun-Aug)
- Winter troughs: ~8.6-8.8 million barrels/day (Dec-Feb)
- No significant trend shift forecast within the 24-month horizon; 95% prediction interval widens to roughly ±0.5 barrels/day by month 24

## Limitations

- Structural breaks (e.g. the 2008 crisis) are absorbed via differencing, not modelled explicitly, a comparable future shock would not be anticipated
- Stationarity of the differenced series is assumed to persist; long-term demand shifts (e.g. EV adoption) would violate this
- No exogenous variables (oil prices, refinery capacity, regulation) are included, and a SARIMAX or VAR approach could improve accuracy
- Forecast uncertainty grows with horizon; the 24-month forecast is a central scenario, not a precise prediction
- Aggregating weekly data to monthly smooths within-month variation

## Project Structure

```
data/                     # us_gasoline series (via fpp3 / tsibbledata)
fuelling-the-forecast.Rmd # full analysis, knits to HTML
figures/                  # exported plots (EDA, forecasts, diagnostics)
README.md
```

## Requirements

R with the `fpp3` ecosystem:
```r
install.packages("fpp3")   # includes tsibble, feasts, fable, ggplot2
```

## Reproducing the Analysis

```bash
git clone [repo-url]
cd [repo-name]
```
Open `fuelling-the-forecast.Rmd` in RStudio and knit to HTML.

## Author

Aditi Sharma
