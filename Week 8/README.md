# Use the Google Cloud Platform Billing API and create a cost forecast using BigQuery ML.

/***Create Arima Model using BigQuery***/ \
CREATE OR REPLACE MODEL billing_sample.arima_model \
OPTIONS \
  (model_type = 'ARIMA_PLUS',
   time_series_timestamp_col = 'date_col',
   time_series_data_col = 'cost',
   auto_arima = TRUE,
   data_frequency = 'AUTO_FREQUENCY',
   decompose_time_series = TRUE
  ) AS \
SELECT
  date(start_time) as date_col,
  round(sum(cost),2) as cost \
FROM
  `billing_sample.sample_data` \
  group by 1



/***Create timeseries with forecast values***/ \
SELECT
 history_timestamp,
 history_value,
 NULL AS forecast_value,
 NULL AS prediction_interval_lower_bound,
 NULL AS prediction_interval_upper_bound \
FROM \
 (
   SELECT
     date(start_time) AS history_timestamp,
     round(sum(cost),2) AS history_value
   FROM
     `billing_sample.sample_data`
   GROUP BY 1
   ORDER BY 1 ASC
 ) \
UNION ALL \
SELECT
 date(forecast_timestamp) AS timestamp ,
 NULL AS history_value,
 forecast_value,
 prediction_interval_lower_bound,
 prediction_interval_upper_bound \
FROM \
 ML.FORECAST(MODEL `billing_sample.arima_model`,
             STRUCT(30 AS horizon, 0.8 AS confidence_level)) \
order by 1