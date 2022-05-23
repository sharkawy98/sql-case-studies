# 📊 Data Analysis Solutions

## 1. How many customers has Foodie-Fi ever had?
```sql
SELECT COUNT(DISTINCT customer_id) as total_customers
FROM   subscriptions;
```
![image](https://user-images.githubusercontent.com/36075516/169700957-e1fb2ae7-c276-4b9d-a92b-42f6d95ea1b7.png)


## 2. What is the monthly distribution of `trial` plan `start_date` values for our dataset - use the start of the month as the group by value
```sql 
SELECT 
  DATEPART(MONTH, start_date) as month,
  COUNT(customer_id) as trials_count
FROM subscriptions
WHERE plan_id = 0
GROUP BY DATEPART(MONTH, start_date)
ORDER BY 1;
```
![image](https://user-images.githubusercontent.com/36075516/169701366-4b48d980-77a7-466e-be35-20bac70f9f80.png)


## 3. What plan `start_date` values occur after the year 2020 for our dataset? Show the breakdown by count of events for each `plan_name`
```sql
SELECT 
  p.plan_name,
  COUNT(s.plan_id) as events
FROM 
  subscriptions s
  JOIN plans p
ON s.plan_id = p.plan_id
WHERE DATEPART(YEAR, start_date) > 2020 
GROUP BY p.plan_name;
```
![image](https://user-images.githubusercontent.com/36075516/169714430-71b9741a-1a32-45eb-9fc1-9cf6cfae455e.png)


## 4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place?
```sql
DECLARE @total_cust float = (SELECT COUNT(DISTINCT customer_id) FROM subscriptions);

SELECT
  COUNT(customer_id) as customers_churned,
  (COUNT(customer_id) / @total_cust) * 100 as churned_perct
FROM subscriptions
WHERE plan_id = 4;
```
![image](https://user-images.githubusercontent.com/36075516/169714897-0bffe73d-1f0f-46e8-a377-a79e29cb1ce3.png)


## 5. How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?
```sql
WITH churned_after_trial AS(
  SELECT
    customer_id,
    CASE 
      WHEN 
        plan_id = 4  --churn plan
        AND
        LAG(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) = 0 --trial plan
      THEN 1
      ELSE 0
    END as is_churned
  FROM subscriptions
)

SELECT 
  SUM(is_churned) as churned_customers,
  FLOOR(SUM(is_churned) / CAST(COUNT(DISTINCT customer_id) AS float) * 100) as churn_perct
FROM churned_after_trial;
```
![image](https://user-images.githubusercontent.com/36075516/169716203-0cf1d7bc-a027-4393-b1e6-ebab0099a23b.png)


## 6. What is the number and percentage of customer plans after their initial free trial?
```sql
DECLARE @total_cust float = (SELECT COUNT(DISTINCT customer_id) FROM subscriptions);

WITH plans_after_trial AS(
  SELECT
    plan_id,
    -- will be used to get the plan directly after the free trial
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date) as plan_order
  FROM subscriptions
  WHERE plan_id <> 0
)

SELECT
  p2.plan_name,
  COUNT(p1.plan_id) AS plans_after_trial,
  COUNT(p1.plan_id) / @total_cust * 100 AS percentage
FROM
  plans_after_trial p1
  JOIN plans p2
  ON p1.plan_id = p2.plan_id
WHERE p1.plan_order = 1
GROUP BY p2.plan_name;
```
![image](https://user-images.githubusercontent.com/36075516/169717954-b8ca70db-742b-46ea-8bbe-a82e6eff51ea.png)


## 7. What is the customer count and percentage breakdown of all 5 `plan_name` values at `2020-12-31`?
```sql
DECLARE @total_cust float = (
  SELECT COUNT(DISTINCT customer_id) 
  FROM subscriptions
  WHERE start_date <= '2020-12-31'
);

WITH customer_plans AS(
  SELECT
    customer_id,
    plan_id,
    -- order by descending start date to get the latest plan of each customer
    ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY start_date DESC) as plan_reverse_order
  FROM subscriptions
  WHERE start_date <= '2020-12-31'
)

SELECT
  p.plan_name,
  COUNT(cp.customer_id) AS total_customers,
  COUNT(cp.customer_id) / @total_cust * 100 AS customers_perct
FROM
  customer_plans cp
  JOIN plans p
  ON cp.plan_id = p.plan_id
WHERE plan_reverse_order = 1
GROUP BY p.plan_name;
```
![image](https://user-images.githubusercontent.com/36075516/169817870-e3203d83-4f30-41d9-9f9a-19a5bbc53ef1.png)


## 8. How many customers have upgraded to an annual plan in 2020?
```sql
SELECT 
  plan_id,
  COUNT(customer_id) as customers_count
FROM subscriptions
WHERE 
  start_date <= '2020-12-31'
  AND
  plan_id = 3
GROUP BY plan_id;
```
![image](https://user-images.githubusercontent.com/36075516/169819572-82c380ba-a3e2-4b4e-91e0-c439f31ed823.png)


## 9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi?
```sql
WITH annual_customers AS(
  SELECT 
    customer_id,
    plan_id,
    start_date AS annual_start_date
  FROM subscriptions
  WHERE plan_id = 3
)

SELECT AVG(DATEDIFF(DAY, s.start_date, c.annual_start_date)*1.0) AS average_days
FROM 
  subscriptions s
  JOIN annual_customers c
  ON s.customer_id = c.customer_id
-- filter with free trial plan to get customer's join date 
WHERE s.plan_id = 0;
```
### Part of the result of joining `subscriptions` table with `annual_customers` CTE
![image](https://user-images.githubusercontent.com/36075516/169837325-f18f1de5-a800-4bb0-a1f0-501df88d224a.png)
### Final result
![image](https://user-images.githubusercontent.com/36075516/169839757-aa707f9d-003f-49a4-ba89-cb5e80597897.png)


## 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)
```sql
WITH trial_plan AS(
  SELECT 
    customer_id,
    start_date AS join_date
  FROM subscriptions
  WHERE plan_id = 0
),
annual_plan AS(
  SELECT 
    customer_id,
    start_date AS annual_start_date
  FROM subscriptions
  WHERE plan_id = 3
),
buckets AS(
  SELECT 
    tp.customer_id,
    join_date,
    annual_start_date,
    -- create buckets of 30 days period from 1 to 12 (i.e monthly buckets)
    DATEDIFF(DAY, join_date, annual_start_date)/30 + 1 AS bucket
  FROM 
    trial_plan tp
    JOIN annual_plan ap
    ON tp.customer_id = ap.customer_id
)

SELECT 
  CASE 
    WHEN bucket = 1 THEN CONCAT(bucket-1, ' - ', bucket*30, ' days')
    ELSE CONCAT((bucket-1)*30 + 1, ' - ', bucket*30, ' days')
  END AS period,
  COUNT(customer_id) AS total_customers,
  CAST(AVG(DATEDIFF(DAY, join_date, annual_start_date)*1.0) AS decimal(5, 2)) AS average_days
FROM buckets
GROUP BY bucket;
```
![image](https://user-images.githubusercontent.com/36075516/169862597-940ce163-ea6a-409a-9af4-743f7413d485.png)


## 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?
```sql
WITH downgraded_customers AS(
  SELECT
    CASE 
      WHEN 
        plan_id = 2  --pro monthly plan 
        AND
        LEAD(plan_id) OVER (PARTITION BY customer_id ORDER BY start_date) = 1  --basic monthly plan
      THEN 1
      ELSE 0
    END as is_downgraded
  FROM subscriptions
)

SELECT SUM(is_downgraded) AS total_downgrads
FROM downgraded_customers;
```
![image](https://user-images.githubusercontent.com/36075516/169864480-e7214d65-56b8-41c3-a9ab-6acafe6e772d.png)
