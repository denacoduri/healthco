<img width="567" alt="Screen Shot 2021-10-05 at 11 50 22 AM" src="https://user-images.githubusercontent.com/84096042/136061033-a4724a54-2622-4682-91c3-5518614754c4.png">

# Health Analytics Mini Case Study
## A SQL debugging exercise to answer business questions about Health Co users

This case study is from Danny Ma's [Serious SQL course](https://www.datawithdanny.com/courses/serious-sql). We’ve just received a request from the General Manager of Analytics at Health Co requesting assistance with their analysis of the health.user_logs dataset. We’ve been asked to debug their SQL script and use the resulting query outputs to quickly answer a few questions that the GM has requested for a board meeting about their active users.


## Business Questions

### 1. How many unique users exist in the logs dataset?

- Buggy Code:

```sql
SELECT
  COUNT DISTINCT user_id
FROM health.user_logs;
```
<img width="346" alt="Screen Shot 2021-10-05 at 12 28 55 PM" src="https://user-images.githubusercontent.com/84096042/136064275-93d4fb21-6608-45c5-a347-cd2c804bd4b1.png">


This code isn't working because parentheses are missing after ```COUNT```, and the column ```user_id``` does not exist. The correct column is ```id```.

- Debugged Code:
```sql
SELECT
COUNT(DISTINCT id) AS row_count
FROM health.user_logs;
```
<img width="200" alt="Debug Q1" src="https://user-images.githubusercontent.com/84096042/136063283-6f1b5aad-233f-4977-9216-5a77ed03bd69.png">

### 2. How many total measurements do we have per user on average?





4. What about the median number of measurements per user?
5. How many users have 3 or more measurements?
6. How many users have 1,000 or more measurements?

Looking at the logs data - what is the number and percentage of the active user base who:

6. Have logged blood glucose measurements?
7. Have at least 2 types of measurements?
8. Have all 3 measures - blood glucose, weight and blood pressure?

For users that have blood pressure measurements:

9. What is the median systolic/diastolic blood pressure values?

Buggy Code:
```sql
-- 1. How many unique users exist in the logs dataset?
SELECT
  COUNT DISTINCT user_id
FROM health.user_logs;

-- for questions 2-8 we created a temporary table
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_cout
SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) as unique_measures
  FROM health.user_logs
  GROUP BY 1; 

-- 2. How many total measurements do we have per user on average?
SELECT
  ROUND(MEAN(measure_count))
FROM user_measure_count;

-- 3. What about the median number of measurements per user?
SELECT
  PERCENTILE_CONTINUOUS(0.5) WITHIN GROUP (ORDER BY id) AS median_value
FROM user_measure_count;

-- 4. How many users have 3 or more measurements?
SELECT
  COUNT(*)
FROM user_measure_count
HAVING measure >= 3;

-- 5. How many users have 1,000 or more measurements?
SELECT
  SUM(id)
FROM user_measure_count
WHERE measure_count >= 1000;

-- 6. Have logged blood glucose measurements?
SELECT
  COUNT DISTINCT id
FROM health.user_logs
WHERE measure is 'blood_sugar';

-- 7. Have at least 2 types of measurements?
SELECT
  COUNT(*)
FROM user_measure_count
WHERE COUNT(DISTINCT measures) >= 2;

-- 8. Have all 3 measures - blood glucose, weight and blood pressure?
SELECT
  COUNT(*)
FROM usr_measure_count
WHERE unique_measures = 3;

-- 9.  What is the median systolic/diastolic blood pressure values?
SELECT
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY systolic) AS median_systolic
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure is blood_pressure;
