# 8 Weeks SQL Challenge
This repository contains solutions for #8WeekSQLChallenge, they are interesting real-world case studies that will allow you to apply and enhance your SQL skills in many use cases.</br>
I used __Microsoft SQL Server__ in writing SQL queries to solve these case studies.


## Table of Contents
- [SQL skills gained](#sql-skills-gained)
- [Case study 1](#case-study-1--dannys-diner)
- [Case study 2](#case-study-2--pizza-runner)
- [Case study 3](#case-study-3--foodie-fi)
- [Some interesting queries from my solutions](#some-interesting-queries)


## SQL skills gained
- Data cleaning & transformation
- Aggregations
- Joins
- CTEs 
- Variables
- Window functions 
    - Ranking (ROW_NUMBER, DENSE_RANK)
    - Analytics (LEAD, LAG)
- CASE WHEN statements
- Subqueries 
- UNION & INTERSECT
- DATETIME functions
- Data type conversion
- TEXT functions, text and string manipulation


## Case Study [#1](https://8weeksqlchallenge.com/case-study-1/) : Danny's Diner
> [My solutions](https://github.com/sharkawy98/sql-case-studies/tree/main/case_study_1)
### ERD 
![image](https://user-images.githubusercontent.com/36075516/162094702-20266a74-afb8-483b-ba5a-37a676df2948.png)

## Case Study [#2](https://8weeksqlchallenge.com/case-study-2/) : Pizza Runner
> [My solutions](https://github.com/sharkawy98/sql-case-studies/tree/main/case_study_2)
### ERD 
![image](https://user-images.githubusercontent.com/36075516/162094574-8d642898-dd21-4231-9e1f-e37dfa69a293.png)

## Case Study [#3](https://8weeksqlchallenge.com/case-study-3/) : Foodie-Fi
> [My solutions](https://github.com/sharkawy98/sql-case-studies/tree/main/case_study_3)
### ERD 
![image](https://user-images.githubusercontent.com/36075516/163922765-6055bd1c-9ba7-4e5c-b6c0-4201295ba3d7.png)

## Case Study [#4](https://8weeksqlchallenge.com/case-study-4/) : Data Bank
> [My solutions](https://github.com/sharkawy98/sql-case-studies/tree/main/case_study_4)
### ERD 
![image](https://user-images.githubusercontent.com/36075516/171291126-bcc6d12e-e2a5-48a4-a2b5-b4840ca2b809.png)


## Some interesting queries
```sql
-- Case study 1, question 10
-- Question: In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?

WITH dates_cte AS(
	SELECT *, 
		DATEADD(DAY, 6, join_date) AS valid_date, 
		EOMONTH('2021-01-1') AS last_date
	FROM members
)

SELECT
	s.customer_id,
	sum(CASE
		WHEN s.product_id = 1 THEN price*20
		WHEN s.order_date between d.join_date and d.valid_date THEN price*20
		ELSE price*10 
	END) as total_points
FROM
	dates_cte d,
	sales s,
	menu m
WHERE
	d.customer_id = s.customer_id
	AND
	m.product_id = s.product_id
	AND
	s.order_date <= d.last_date
GROUP BY s.customer_id;
```
---
```sql
-- Case study 2, data cleaning
SELECT 
    order_id,
    runner_id,
    cast(CASE 
        WHEN pickup_time = 'null' THEN null
        ELSE pickup_time
    END as datetime) as pickup_time,
    cast(CASE 
        WHEN distance = 'null' THEN null
        ELSE TRIM('km' from distance)
    END as float) as distance,
    cast(CASE
        WHEN duration = 'null' THEN null
        ELSE SUBSTRING(duration, 1, 2)
    END as int)as duration,
    CASE
        WHEN cancellation in ('null', '') THEN null
        ELSE cancellation
    END as cancellation
INTO #cleaned_runner_orders
FROM runner_orders;
```
---
```sql
-- Case study 2, c) ingredient optimisation data cleaning
-- Question: The ingredients of each pizza a stored at pizza_recipes table as a comma separated string including ids of all its toppings, change this string to multiple rows

 SELECT		
    p.pizza_id,
    TRIM(t.value) AS topping_id,
    pt.topping_name
 INTO #cleaned_toppings
 FROM 
    pizza_recipes as p
    CROSS APPLY string_split(p.toppings, ',') as t
    JOIN pizza_toppings as pt
    ON TRIM(t.value) = pt.topping_id 
 ;
```
---
```sql
-- Case study 2, c) ingredient optimisation question 3
-- Question: Generate an order item for each record in the customers_orders table in the format of one of the following:
-- Meat Lovers
-- Meat Lovers - Exclude Beef
-- Meat Lovers - Extra Bacon
-- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

WITH extras_cte AS
(
	SELECT 
		record_id,
		'Extra ' + STRING_AGG(t.topping_name, ', ') as record_options
	FROM
		#extras e,
		pizza_toppings t
	WHERE e.topping_id = t.topping_id
	GROUP BY record_id
),
exclusions_cte AS
(
	SELECT 
		record_id,
		'Exclude ' + STRING_AGG(t.topping_name, ', ') as record_options
	FROM
		#exclusions e,
		pizza_toppings t
	WHERE e.topping_id = t.topping_id
	GROUP BY record_id
),
union_cte AS
(
	SELECT * FROM extras_cte
	UNION
	SELECT * FROM exclusions_cte
)

SELECT 
	c.record_id,
	CONCAT_WS(' - ', p.pizza_name, STRING_AGG(cte.record_options, ' - '))
FROM 
	#cleaned_customer_orders c
	JOIN pizza_names p
	ON c.pizza_id = p.pizza_id
	LEFT JOIN union_cte cte
	ON c.record_id = cte.record_id
GROUP BY
	c.record_id,
	p.pizza_name
ORDER BY 1;
```
---
```sql
-- Case study 3, b) data analysis question 5
-- Question: How many customers have churned straight after their initial free trial - what percentage is this rounded to the nearest whole number?

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

```sql
-- Case study 3, b) data analysis question 10
-- Question: Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60 days etc)

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
