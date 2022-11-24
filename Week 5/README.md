# Return an aggregated result with a machine-learning (ML) prediction using Google BigQuery ML and serve out results using Google App Engine.

Command Used: \
/* select initial features and label to feed into your model* / \

CREATE OR REPLACE TABLE week5.propensity_data AS \
  SELECT 
    fullVisitorId,
    bounces,
    time_on_site,
    will_buy_on_return_visit \
  FROM (\
        /* select features */ \
        SELECT
          fullVisitorId,
          IFNULL(totals.bounces, 0) AS bounces,
          IFNULL(totals.timeOnSite, 0) AS time_on_site \
        FROM
          `data-to-insights.ecommerce.web_analytics` \
        WHERE
          totals.newVisits = 1
        AND date BETWEEN '20160801' # train on first 9 months of data
        AND '20170430'
       )\
  JOIN ( \
        SELECT
          fullvisitorid, \
          IF (
              COUNTIF (
                       totals.transactions > 0
                       AND totals.newVisits IS NULL
                      ) > 0,
              1,
              0
             ) AS will_buy_on_return_visit \
        FROM
          `bigquery-public-data.google_analytics_sample.*` \
        GROUP BY
          fullvisitorid
       ) \
  USING (fullVisitorId)
  ORDER BY time_on_site DESC;

CREATE OR REPLACE MODEL `week5.rpm_bqml_model` \
OPTIONS(MODEL_TYPE = 'logistic_reg',
        labels = [ 'will_buy_on_return_visit' ],
        auto_class_weights=TRUE
        ) \
AS
SELECT * EXCEPT (fullVisitorId) \
FROM `week5.propensity_data`;

gcloud config set project msds434-365612

gsutil mb 'gs://msds434-365612-bucket'

bq extract -m week5.rpm_bqml_model gs://msds434-365612-bucket/bq_model

/* Deploy Model*/

gcloud ai-platform versions create --model=rpm_bqml_model V_1 --framework=tensorflow --python-version=3.7 --runtime-version=1.15 --origin=gs://msds434-365612-bucket/bq_model/ --staging-bucket=gs://msds434-365612-bucket

/* Test Model */

{
   "instances":[{
                "bounces": 0, 
                "time_on_site": 7363
               }]
}

https://us-central1-ml.googleapis.com/v1/projects/msds434-365612/models/rpm_bqml_model/versions/V_1:predict?access_token=<<your_access_token>>