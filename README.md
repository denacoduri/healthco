<img width="567" alt="Screen Shot 2021-10-05 at 11 50 22 AM" src="https://user-images.githubusercontent.com/84096042/136061033-a4724a54-2622-4682-91c3-5518614754c4.png">

# Health Analytics Mini Case Study
## A SQL debugging exercise to answer business questions about Health Co users

This case study is from Danny Ma's [Serious SQL course](https://www.datawithdanny.com/courses/serious-sql). We’ve just received a request from the General Manager of Analytics at Health Co requesting assistance with their analysis of the health.user_logs dataset. We’ve been asked to debug their SQL script and use the resulting query outputs to quickly answer a few questions that the GM has requested for a board meeting about their active users.

The data we will be using is from the following `health.user_logs` table (first 10 rows displayed):

| id                                       | log_date                 | measure        | measure_value | systolic | diastolic |
|------------------------------------------|--------------------------|----------------|---------------|----------|-----------|
| fa28f948a740320ad56b81a24744c8b81df119fa | 2020-11-15T00:00:00.000Z | weight         | 46.03959      |          |           |
| 1a7366eef15512d8f38133e7ce9778bce5b4a21e | 2020-10-10T00:00:00.000Z | blood_glucose  | 97            | 0        | 0         |
| bd7eece38fb4ec71b3282d60080d296c4cf6ad5e | 2020-10-18T00:00:00.000Z | blood_glucose  | 120           | 0        | 0         |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-10-17T00:00:00.000Z | blood_glucose  | 232           | 0        | 0         |
| d14df0c8c1a5f172476b2a1b1f53cf23c6992027 | 2020-10-15T00:00:00.000Z | blood_pressure | 140           | 140      | 113       |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-10-21T00:00:00.000Z | blood_glucose  | 166           | 0        | 0         |
| 0f7b13f3f0512e6546b8d2c0d56e564a2408536a | 2020-10-22T00:00:00.000Z | blood_glucose  | 142           | 0        | 0         |
| 87be2f14a5550389cb2cba03b3329c54c993f7d2 | 2020-10-12T00:00:00.000Z | weight         | 129.060012817 | 0        | 0         |
| 0efe1f378aec122877e5f24f204ea70709b1f5f8 | 2020-10-07T00:00:00.000Z | blood_glucose  | 138           | 0        | 0         |
| 054250c692e07a9fa9e62e345231df4b54ff435d | 2020-10-04T00:00:00.000Z | blood_glucose  | 210           |          |           |


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

### For questions 2-8 we created a temporary table

- Buggy Code:
```sql
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_cout
SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) as unique_measures
  FROM health.user_logs
  GROUP BY 1; 
  ```
  <img width="332" alt="Screen Shot 2021-10-05 at 1 31 38 PM" src="https://user-images.githubusercontent.com/84096042/136073601-e54f61af-89cd-4156-8fda-b80ef4c9aff8.png">

  There are 2 errors here. There is a spelling mistake in the 2nd line. ```user_measure_cout``` should be ```user_measure_count```. Also, we need to use ```AS``` before ```SELECT``` when creating a temporary table.
  
  - Debugged Code:
```sql
DROP TABLE IF EXISTS user_measure_count;
CREATE TEMP TABLE user_measure_count
AS 
SELECT
    id,
    COUNT(*) AS measure_count,
    COUNT(DISTINCT measure) as unique_measures
  FROM health.user_logs
  GROUP BY 1; 
  
  --check the output of the temp table:
  SELECT * FROM user_measure_count
  LIMIT 20;
  ```
  <img width="782" alt="Debug Q2-8" src="https://user-images.githubusercontent.com/84096042/136072376-3406d34a-171f-4f06-ab90-9eb826d6f577.png">

### 2. How many total measurements do we have per user on average?

- Buggy Code:
```sql
SELECT
  ROUND(MEAN(measure_count))
FROM user_measure_count;
```
<img width="338" alt="Screen Shot 2021-10-05 at 1 36 51 PM" src="https://user-images.githubusercontent.com/84096042/136074387-2645b80b-1c33-40ad-b5fe-1f31f0735365.png">

The mistake here is that ```MEAN``` does not exist as a function in PostgreSQL. We need to use the ```AVG``` function to find the mean here. It is not necessary, but I also used ```avg_measure_count``` as an alias. Without an alias, the column heading for the average will default to "round," and that can be confusing to anyone interpreting the output.

- Debugged Code:
```sql
SELECT
  ROUND(AVG(measure_count)) AS avg_measure_count
FROM user_measure_count;
```
<img width="202" alt="Debug Q2" src="https://user-images.githubusercontent.com/84096042/136072335-2e018540-4b85-4362-8009-bb8d99fe0530.png">

### 3. What about the median number of measurements per user?

- Buggy Code
```sql
SELECT
  PERCENTILE_CONTINUOUS(0.5) WITHIN GROUP (ORDER BY id) AS median_value
FROM user_measure_count;
```
<img width="658" alt="Screen Shot 2021-10-05 at 12 23 07 PM" src="https://user-images.githubusercontent.com/84096042/136079056-596ff71f-dc16-417b-b7fe-8463d682a17e.png">

Here, ```PERCENTILE_CONTINUOUS``` does not exist as a function in SQL. We need to use `PERCENTILE_CONT` to find the 50th percentile AKA the median. Also, `measure_count` should come after `ORDER BY`, since we are finding the median of the measure counts.

- Debugged Code:

```sql
  SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY measure_count) AS median_measure_count
FROM user_measure_count;
  ```

<img width="206" alt="Debug Q3" src="https://user-images.githubusercontent.com/84096042/136072387-5e724988-e558-422b-a7dc-673b5f96ec72.png">


### 4. How many users have 3 or more measurements?

- Buggy Code:
```sql
SELECT
  COUNT(*)
FROM user_measure_count
HAVING measure >= 3;
```
<img width="333" alt="Screen Shot 2021-10-05 at 2 11 05 PM" src="https://user-images.githubusercontent.com/84096042/136079171-82faa897-49cd-4c08-8903-38b38950721e.png">

The column `measure` does not exist. We need to use `measure_count`. Also, the `HAVING` clause needs to be replaced with `WHERE` since we are filtering records from a table and not from a group.


- Debugged Code:

```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 3;
```

<img width="152" alt="Debug Q4" src="https://user-images.githubusercontent.com/84096042/136072402-7e18c839-2c76-4b94-a44c-bca286ffd99f.png">


### 5. How many users have 1,000 or more measurements?

- Buggy Code:
```sql
SELECT
  SUM(id)
FROM user_measure_count
WHERE measure_count >= 1000;
```
<img width="453" alt="Screen Shot 2021-10-05 at 2 11 44 PM" src="https://user-images.githubusercontent.com/84096042/136079249-7828f36e-60d1-4b27-b366-d028f23f7ce2.png">

Here, we need to use `COUNT` instead of `SUM` to count the rows. It is ok here to use `COUNT(id)` or `COUNT(*)`.

- Debugged Code:

```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE measure_count >= 1000;
```

<img width="151" alt="Debug Q5" src="https://user-images.githubusercontent.com/84096042/136072438-056da0b5-65c5-4941-b696-0c67645a1a9e.png">


### Looking at the logs data - what is the number and percentage of the active user base who:

### 6. Have logged blood glucose measurements?

- Buggy Code:
```sql
SELECT
  COUNT DISTINCT id
FROM health.user_logs
WHERE measure is 'blood_sugar';
```
<img width="340" alt="Screen Shot 2021-10-05 at 2 12 14 PM" src="https://user-images.githubusercontent.com/84096042/136079329-9448ca2a-799e-401d-88be-338d6cea4dc8.png">

```blood_sugar``` does not exist as one of the measures in our data. The correct measure is ```blood_glucose```. Parentheses are required after ```COUNT```.

- Debugged Code:

```sql
SELECT
  COUNT(DISTINCT id)
FROM health.user_logs
WHERE measure = 'blood_glucose';
```
<img width="155" alt="Debug Q6" src="https://user-images.githubusercontent.com/84096042/136072454-03957475-0187-4a50-9cb5-3c7d90a3c4fe.png">


### 7. Have at least 2 types of measurements?

- Buggy Code:
```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE COUNT(DISTINCT measures) >= 2;
```
<img width="341" alt="Screen Shot 2021-10-05 at 2 12 51 PM" src="https://user-images.githubusercontent.com/84096042/136079415-0c774533-3740-43f2-8181-84faef0bbec9.png">

```measures``` does not exist as a column heading. There is already a column for `unique_measures` in our temporary table that we can use.

- Debugged Code:

```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE unique_measures >= 2;
```

<img width="176" alt="Debug Q7" src="https://user-images.githubusercontent.com/84096042/136072463-5a7610d1-ae36-418e-b8d9-6458fc7c1dc7.png">


### 8. Have all 3 measures - blood glucose, weight and blood pressure?
- Buggy Code:
```sql
SELECT
  COUNT(*)
FROM usr_measure_count
WHERE unique_measures = 3;
```
<img width="438" alt="Screen Shot 2021-10-05 at 2 13 29 PM" src="https://user-images.githubusercontent.com/84096042/136079474-fb5c5593-1ba5-4ddd-bd90-ccc9a9dbfc2e.png">

This simply has a spelling error. ```usr_measure_count``` should be `user_measure_count`.

- Debugged Code:

```sql
SELECT
  COUNT(*)
FROM user_measure_count
WHERE unique_measures = 3;
```

<img width="146" alt="Debug Q8" src="https://user-images.githubusercontent.com/84096042/136072474-6cbccc4e-488b-43b3-860a-23d2754da387.png">


### For users that have blood pressure measurements:

### 9. What is the median systolic/diastolic blood pressure values?
- Buggy Code:
```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY systolic) AS median_systolic
  PERCENTILE_CONT(0.5) WITHIN (ORDER BY diastolic) AS median_diastolic
FROM health.user_logs
WHERE measure is blood_pressure;
```

<img width="270" alt="Screen Shot 2021-10-05 at 2 14 17 PM" src="https://user-images.githubusercontent.com/84096042/136079591-d8865d53-a4f4-4e7d-8458-cc1090e1f9e0.png">

`GROUP` needs to be used after `WITHIN` here, and `blood_pressure` should be in parentheses. The word "is" should be replaced with `=`.

- Debugged Code:

```sql
SELECT
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY diastolic) AS median_diastolic,
  PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY systolic) AS median_systolic
FROM health.user_logs
WHERE measure = 'blood_pressure';
```
<img width="453" alt="Debug Q9" src="https://user-images.githubusercontent.com/84096042/136072495-5ee9800f-f423-4993-ba54-e4527fc63310.png">
