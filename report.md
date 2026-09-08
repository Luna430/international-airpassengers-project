# Passenger Volume Forecasting Report

Passenger numbers show a clear upward trend from 1949 to 1960. The series also exhibits a recurring annual seasonal pattern, with the magnitude of the fluctuations increasing as passenger volumes rise. This means the series is not simply moving randomly from month to month: it contains both long-term growth and a repeated yearly structure that a forecasting model needs to recognize.

The exploratory analysis confirmed these patterns quantitatively. Trend strength was approximately 1.00 and seasonal strength approximately 0.99, indicating that both components are extremely strong in this series. The autocorrelation function also remained high over multiple lags, with clear dependence around lags 12 and 24. This is consistent with the visual pattern of annual repetition and shows why a model that ignores time dependence would be inappropriate.

The forecasting process therefore began with a benchmark rather than immediately choosing a more complex model. Seasonal Naive was used as the main benchmark floor because it directly reflects the strong annual seasonal structure by using the value from the same month of the previous year. A simple Naive forecast was also fitted as an additional benchmark. All later models were judged against the Seasonal Naive floor rather than considered successful simply because they produced reasonable-looking forecasts.

## Recommendation

I recommend using AutoARIMA to forecast this series. Across eight rolling-origin cross-validation windows, AutoARIMA achieved a MASE of 0.647, compared with 1.313 for the Seasonal Naive benchmark, providing substantially better point-forecast accuracy across repeated forecast periods rather than on a single split.

The initial AutoGluon leaderboard provided useful model screening, but it was based on only one holdout window and therefore could not be treated as sufficient evidence for a final decision. On that holdout period, AutoARIMA was the strongest individual model according to `score_test`. However, the project evaluation rules require a stronger test because a model can perform well on one period and poorly on another.

For that reason, AutoARIMA was evaluated again using the same rolling-origin cross-validation framework as the Seasonal Naive benchmark. The evaluation used eight forecast origins, a 12-month forecast horizon, and a 12-month step between origins. This provides a much stronger basis for comparison because the model is tested repeatedly at different points in the historical series.

The cross-validation results confirmed the original shortlist. AutoARIMA achieved a MASE of 0.647 compared with 1.313 for Seasonal Naive. Its RMSSE was also lower, at approximately 0.683 compared with 1.246 for the benchmark. These results show that AutoARIMA reduced both typical forecast error and the effect of larger forecast misses.

The advantage was not limited to point forecasts. AutoARIMA also produced a better probabilistic forecast according to scaled CRPS, which evaluates the forecast distribution rather than only the central forecast. Its scaled CRPS was 0.0339, compared with 0.0616 for the Seasonal Naive model. Since lower values are preferred, AutoARIMA performed better on this measure as well.

Overall, the eight-window evaluation supports AutoARIMA as the strongest model tested in this project. It beats the benchmark on point accuracy and distributional quality. However, the prediction intervals and residual diagnostics show that the model is still not perfect, so the recommendation should be understood as the best model among those evaluated rather than as a complete solution to every feature of the series.

## Evaluation Summary

| Metric | Seasonal Naive | AutoARIMA | Interpretation |
|---|---:|---:|---|
| MASE | 1.313 | 0.647 | AutoARIMA has substantially lower point-forecast error |
| RMSSE | 1.246 | 0.683 | AutoARIMA also reduces larger forecast errors |
| Scaled CRPS | 0.0616 | 0.0339 | AutoARIMA produces a better forecast distribution |
| 80% coverage | 51.0% | 69.8% | AutoARIMA is closer to the nominal 80%, but still undercovers |
| Mean 80% interval width | 77.76 | 44.08 | AutoARIMA produces narrower uncertainty bands |

The table shows that AutoARIMA does not win on only one metric. It improves the point forecast, the scaled probabilistic score, and interval coverage relative to the benchmark. At the same time, the remaining gap between actual and nominal coverage is important and should not be ignored.

## Prediction Intervals

Forecast accuracy alone is not sufficient for decision-making because a forecast should also communicate uncertainty. For this reason, the 80% prediction intervals were evaluated using both their average width and their actual empirical coverage.

The average width of the AutoARIMA 80% prediction interval was approximately 44.08, compared with 77.76 for Seasonal Naive. AutoARIMA therefore produced considerably narrower uncertainty bands. Narrow intervals are useful only when they remain well calibrated, so width must be interpreted together with coverage.

AutoARIMA achieved approximately 69.8% actual coverage for its nominal 80% interval. Seasonal Naive achieved only about 51.0% coverage. AutoARIMA is therefore clearly closer to the intended 80% level and provides a more useful representation of forecast uncertainty than the benchmark.

However, the AutoARIMA result still represents undercoverage. An interval labelled as 80% would ideally contain the observed outcome approximately 80% of the time across repeated forecasts. A coverage rate of 69.8% means that the model's uncertainty bands are still somewhat too optimistic and narrower than the observed forecast errors justify.

The Seasonal Naive benchmark performs even worse in this respect. Its mean interval is wider than AutoARIMA's, yet it covers only about half of the actual observations. This shows why interval width alone cannot determine whether an uncertainty band is honest. A wider interval is not automatically better if its empirical coverage still falls well below its nominal level.

AutoARIMA also achieved the better scaled CRPS value, 0.0339 compared with 0.0616 for Seasonal Naive. This result is important because CRPS evaluates the quality of the forecast distribution across multiple quantiles rather than checking only whether observations fall inside one particular interval. Therefore, the combination of lower CRPS and improved coverage supports AutoARIMA as the better probabilistic model, although interval calibration remains an area for improvement.

For a manager, the practical conclusion is that the AutoARIMA point forecasts are the stronger choice, but the 80% intervals should not be treated as perfectly calibrated uncertainty statements. Decisions that are highly sensitive to forecast uncertainty should account for the remaining undercoverage.

## Residuals

Residual analysis was used to determine whether the forecasting models left systematic time structure unexplained. Ideally, after a model captures the available structure in a series, its forecast errors should behave as randomly as possible rather than showing strong temporal dependence.

For the Seasonal Naive benchmark, the Ljung–Box test strongly rejected the idea that the residuals were white noise. The p-values were approximately 2.32 × 10^-43 at lag 12 and 1.71 × 10^-44 at lag 24. These values are extremely small and provide strong evidence that the benchmark leaves temporal dependence in its residuals.

This result is consistent with the benchmark's role. Seasonal Naive successfully uses the repeated yearly pattern, but it does not fully account for the other evolving structure in the series. The residuals therefore still contain information that could potentially be exploited by a more flexible forecasting model.

AutoARIMA reduced forecasting error substantially, but the residual diagnostics show that it also failed to remove all temporal dependence. When the Ljung–Box test was applied to AutoARIMA errors obtained from rolling-origin cross-validation, the p-values remained extremely small: approximately 5.71 × 10^-26 at lag 12 and 1.34 × 10^-37 at lag 24.

This means that AutoARIMA performs much better as a forecaster while still leaving systematic structure in its errors. In other words, better predictive performance does not imply that the model has explained every feature of the time series.

The residual diagnostics are important because they identify a limitation that is not visible from MASE alone. AutoARIMA has a much lower MASE and scaled CRPS than Seasonal Naive, but its residuals show that there is still predictive information that the model does not fully use.

The Ljung–Box test does not identify the exact source of that remaining dependence. Therefore, it would be incorrect to conclude from the test alone that the remaining structure is specifically caused by trend, seasonality, or another particular component. Instead, the test provides evidence that additional temporal structure remains and may be worth investigating.

This is why AutoARIMA should be considered the strongest model evaluated in this project, but not a complete description of the data-generating process. The model improves forecast accuracy substantially, yet both interval calibration and residual dependence leave clear opportunities for further improvement.

## One Change

The next change I would test is the addition of an external driver through dynamic regression. Specifically, I would add monthly mean temperature from a selected reference region as an exogenous variable and evaluate whether it helps explain some of the temporal dependence that remains in the AutoARIMA residuals.

The objective would not be to assume that temperature necessarily causes international passenger demand. Instead, the variable would be treated as a modelling hypothesis that must prove its value through out-of-sample evaluation. The driver should only be retained if it consistently improves forecasting performance across the same evaluation framework used for AutoARIMA and Seasonal Naive.

The dynamic regression model should therefore be tested using the same eight rolling-origin windows, 12-month forecast horizon, and 12-month step. Using the same folds is essential because changing the evaluation periods would make the comparison unfair.

The external variable must also obey the no-leakage rule. For every fold, model fitting and any information used to construct the predictor must be restricted to data that would have been available at that point in time. Future observations must not influence model estimation, feature preparation, or metric denominators.

I would judge the change using the same evidence used in this report. The new model should reduce MASE if it improves point accuracy, reduce scaled CRPS if it improves the forecast distribution, maintain or improve 80% interval coverage, and ideally leave weaker residual dependence.

If the additional driver improves only one metric while damaging another, both results should be reported rather than describing the model as an unconditional improvement. For example, lower MASE combined with worse coverage would represent a genuine trade-off rather than a complete win.

If the dynamic regression model produces consistent gains across the eight rolling-origin windows and reduces the remaining temporal dependence, it would provide evidence that useful predictive information exists outside the passenger series itself. If it does not, AutoARIMA should remain the preferred model because its current advantage over Seasonal Naive has already been demonstrated under repeated cross-validation.

## Final Decision

The evidence supports AutoARIMA as the model I would currently deploy from the models evaluated in this project. It achieved substantially lower MASE and scaled CRPS than the Seasonal Naive benchmark across eight rolling-origin windows, while also producing coverage closer to the intended 80% level.

The main limitation is uncertainty calibration: AutoARIMA's 80% interval covered approximately 69.8% of observations, so its uncertainty bands remain somewhat too narrow. Its Ljung–Box results also show that residual temporal dependence remains.

For these reasons, AutoARIMA is the strongest current forecasting choice, but future work should focus on explaining the remaining residual structure and improving uncertainty calibration rather than assuming that the current model has fully captured the series.
