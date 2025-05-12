# Predict Visitor Purchases with BigQuery ML

### 📌 Overview

BigQuery ML enables data analysts and SQL users to build and deploy machine learning models directly within BigQuery using standard SQL queries. This project predicts whether a visitor to the Google Merchandise Store will make a purchase using a **logistic regression model** trained on public Google Analytics data.

---

### 🌟 Objective

To predict user transactions on an eCommerce platform using historical session-level features, leveraging the scalability of BigQuery and the accessibility of SQL.

---

### 📚 What You'll Learn

* Creating datasets and views in BigQuery
* Training ML models using SQL (`CREATE MODEL`)
* Evaluating model performance using `ML.EVALUATE`
* Making predictions with `ML.PREDICT` per country and per user

---

### ⚙️ Requirements

* Google Cloud account (via Qwiklabs or standard GCP)
* Basic understanding of SQL
* Web browser (Chrome/Firefox)

---

### 🛠️ Project Workflow

#### ✅ Step 1: Create Dataset

Create a dataset named `bqml_lab` in your GCP project to store models and views.

#### ✅ Step 2: Explore & Prepare Data

```sql
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20160801' AND '20170631'
LIMIT 10000;
```

Save this as a view: `bqml_lab.training_data`

---

#### ✅ Step 3: Create ML Model

```sql
CREATE OR REPLACE MODEL `bqml_lab.sample_model`
OPTIONS(model_type='logistic_reg') AS
SELECT * FROM `bqml_lab.training_data`;
```

---

#### ✅ Step 4: Evaluate the Model

```sql
SELECT * FROM ML.EVALUATE(MODEL `bqml_lab.sample_model`);
```

🗸 *Add evaluation result image here*

---

#### ✅ Step 5: Prepare Test Data

```sql
SELECT
  IF(totals.transactions IS NULL, 0, 1) AS label,
  IFNULL(device.operatingSystem, "") AS os,
  device.isMobile AS is_mobile,
  IFNULL(geoNetwork.country, "") AS country,
  IFNULL(totals.pageviews, 0) AS pageviews,
  fullVisitorId
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
WHERE
  _TABLE_SUFFIX BETWEEN '20170701' AND '20170801';
```

Save this as view: `bqml_lab.july_data`

---

#### ✅ Step 6A: Predict Purchases per Country

```sql
SELECT
  country,
  SUM(predicted_label) AS total_predicted_purchases
FROM
  ML.PREDICT(MODEL `bqml_lab.sample_model`, (
    SELECT * FROM `bqml_lab.july_data`)
  )
GROUP BY country
ORDER BY total_predicted_purchases DESC
LIMIT 10;
```

🗸 *Add top 10 countries result image here*

---

#### ✅ Step 6B: Predict Purchases per User

```sql
SELECT
  fullVisitorId,
  SUM(predicted_label) AS total_predicted_purchases
FROM
  ML.PREDICT(MODEL `bqml_lab.sample_model`, (
    SELECT * FROM `bqml_lab.july_data`)
  )
GROUP BY fullVisitorId
ORDER BY total_predicted_purchases DESC
LIMIT 10;
```

🗸 *Add top 10 visitors result image here*

