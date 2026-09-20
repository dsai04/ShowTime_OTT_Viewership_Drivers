# 🎥 ShowTime OTT: What Drives First-Day Viewership

An OLS regression model that explains first-day content views on ShowTime, an OTT streaming platform. It uses 1,000 titles and 7 predictors: weekly platform visitors, ad impressions, trailer views, release day, season, genre, and whether a major sports event fell on release day.

The dataset and business case come from the Great Learning Postgraduate Program in Data Science.

## Results at a glance

- **Model:** OLS regression on 1,000 OTT titles, evaluated on a held-out 30% test set.
- **Test performance:** R² = 76.6%, MAPE = 9.0%, MAE of about 41K views (0.041 million). Train R² is 79.2%.
- **Significant predictors:** weekly platform visitors, trailer views, major sports events, release day and season. Trailer views are the strongest.
- **Sports events:** a major sports event is associated with about 60K lower first-day views, controlling for the other factors.
- **Release day:** a Saturday release is associated with about 58K higher views than a Friday release, controlling for the other factors.
- **Assumptions:** checked with VIF, residual analysis, Shapiro-Wilk, Durbin-Watson and Goldfeld-Quandt tests. No significant multicollinearity or heteroscedasticity.
- **No effect found:** ad impressions and genre are not significant in this data.

## Business question

ShowTime wants to know which factors drive first-day viewership, so it can act on them. Possible causes named in the brief: fewer visitors, lower marketing spend, release-timing clashes, weekends and holidays.

## Data

1,000 rows, 8 columns. No missing values, no duplicate rows. All view and visitor counts are in millions.

| Variable | Type | Description |
|---|---|---|
| `visitors` | Numeric | Average visitors to the platform in the past week |
| `ad_impressions` | Numeric | Ad impressions across all campaigns for the content |
| `major_sports_event` | Binary | 1 if a major sports event fell on release day |
| `genre` | Categorical (8) | Genre of the content |
| `dayofweek` | Categorical (7) | Day of release |
| `season` | Categorical (4) | Season of release |
| `views_trailer` | Numeric | Views of the content trailer |
| `views_content` | Numeric | First-day views of the content (target) |

## Method

1. Checked shape, data types, missing values and duplicates.
2. Univariate and bivariate analysis of every variable.
3. Flagged outliers with boxplots. None were removed: the titles with very high trailer views (up to about 200 million) are also the titles with the highest first-day views, so removing them would remove real signal.
4. One-hot encoded genre, day of week and season, dropping the first category of each. The baselines are Action, Friday and Fall.
5. Split the data 70/30 (700 train, 300 test, `random_state=1`).
6. Fitted an OLS regression with statsmodels and compared train and test performance.
7. Checked the five regression assumptions.

## Findings

### Exploratory analysis

- First-day views are right-skewed. Most titles get 0.4 to 0.5 million views; the maximum is 0.89 million.
- Trailer views and first-day views have a correlation of 0.75, the strongest relationship in the data. For comparison: visitors 0.26, major sports event -0.24, ad impressions 0.05.
- Titles released on a major sports event day have lower first-day views.
- Friday has the lowest median views, and Wednesday and Saturday the highest. Fall is the weakest season.
- Median views look similar across genres.

### Model coefficients

Coefficients are in millions of views, so 0.0603 means 60,300 views. Day, season and genre effects are compared with the baselines (Friday, Fall, Action). Each effect holds the other predictors fixed.

| Predictor | Coefficient | Effect on first-day views | p-value | Significant at 5% |
|---|---|---|---|---|
| Trailer views | 0.00233 | +2,330 per extra 1 million trailer views | < 0.001 | Yes |
| Visitors | 0.1295 | +12,900 per extra 0.1 million weekly visitors | < 0.001 | Yes |
| Major sports event | -0.0603 | -60,300 | < 0.001 | Yes |
| Saturday | 0.0579 | +57,900 vs Friday | < 0.001 | Yes |
| Wednesday | 0.0474 | +47,400 vs Friday | < 0.001 | Yes |
| Sunday | 0.0363 | +36,300 vs Friday | < 0.001 | Yes |
| Monday | 0.0337 | +33,700 vs Friday | 0.005 | Yes |
| Thursday | 0.0173 | +17,300 vs Friday | 0.011 | Yes |
| Tuesday | 0.0228 | +22,800 vs Friday | 0.096 | No |
| Summer | 0.0442 | +44,200 vs Fall | < 0.001 | Yes |
| Winter | 0.0272 | +27,200 vs Fall | < 0.001 | Yes |
| Spring | 0.0226 | +22,600 vs Fall | < 0.001 | Yes |
| Ad impressions | 0.0000036 | No measurable effect | 0.582 | No |
| Genre (7 dummies vs Action) | 0.0006 to 0.0131 | No measurable effect | 0.11 to 0.95 | No |

In plain terms:

- **Trailer views matter most.** An increase of one standard deviation in trailer views (about 35 million) goes with about 81,500 more first-day views. The same step in weekly visitors (about 0.23 million) goes with about 30,000 more.
- **Sports events hurt.** A major sports event on release day is associated with about 60,300 fewer first-day views (~60K), roughly 13% of the average title (473,000), controlling for the other factors.
- **Friday is the weakest release day**, yet it is also the most common release day in the data. A Saturday release is associated with about 57,900 more views (~58K) than a Friday release, controlling for the other factors. Wednesday is next at about 47,400.
- **Fall is the weakest season.**
- **Ad impressions and genre show no detectable effect.** This does not prove they don't matter. It means this dataset does not show it.

### Model performance

| Metric | Train | Test |
|---|---|---|
| RMSE | 0.0485 | 0.0506 |
| MAE | 0.0382 | 0.0408 |
| R² | 0.7916 | 0.7664 |
| Adjusted R² | 0.7852 | 0.7488 |
| MAPE | 8.56% | 9.03% |

The gap between train and test is small, so there is no sign of overfitting. The overall F-test is significant (F = 129.0, p = 1.3e-215).

### Assumption checks

| Assumption | Check | Result |
|---|---|---|
| No multicollinearity | Variance Inflation Factor (VIF) | All VIFs below 3 (highest 2.57, `genre_Others`) |
| Linearity | Residuals vs fitted values | No clear pattern |
| Independent errors | Durbin-Watson | 2.00, no sign of autocorrelation |
| Normal errors | Histogram, Q-Q plot, Shapiro-Wilk | Close to normal, slight tail deviations; p = 0.28, no evidence against normality |
| Constant variance | Goldfeld-Quandt | p = 0.11, no evidence of unequal variance |

Conclusion: no significant multicollinearity or heteroscedasticity. The dataset has no date column, so Durbin-Watson is only a rough check on independence.

## Repository contents

| File | Description |
|---|---|
| `Showtime_ott.ipynb` | Full analysis: EDA, model, assumption checks, equation |
| `ShowTime_OTT.pdf` | Business report with charts and all test results |
| `README.md` | This file |

## How to run

The dataset (`ottdata.csv`) is not included in this repository. To rerun the notebook, place the file in the same folder as the notebook and load it with `pd.read_csv("ottdata.csv")`.

```
pip install pandas numpy scipy matplotlib seaborn scikit-learn statsmodels
jupyter notebook Showtime_ott.ipynb
```

## Tools

Python, pandas, NumPy, SciPy, statsmodels, scikit-learn, Matplotlib, Seaborn

## Author

Dhruv Sai Gedda | [LinkedIn](https://www.linkedin.com/in/dhruv-sai) | [Portfolio](https://dsai04.github.io/)
