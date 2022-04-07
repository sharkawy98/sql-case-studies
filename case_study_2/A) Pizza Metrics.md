# 🍕 Pizza Metrics Solutions

### 1. How many pizzas were ordered?
```sql
SELECT count(pizza_id) as orders
FROM #cleaned_customer_orders;
```
![image](https://user-images.githubusercontent.com/36075516/162095396-87a66c16-de28-43b0-83d5-5f1de5f6d80b.png)

### 2. How many unique customer orders were made?
```sql
SELECT count(distinct order_id) as unique_orders
FROM #cleaned_customer_orders;
```  
![image](https://user-images.githubusercontent.com/36075516/162095552-4eebb08c-1468-4256-968f-346ec2331af0.png)

### 3. How many successful orders were delivered by each runner?
```sql
SELECT 
    runner_id,
    count(order_id) as successful_orders
FROM #cleaned_runner_orders
WHERE cancellation is null
GROUP BY runner_id;
```
![image](https://user-images.githubusercontent.com/36075516/162095619-0163cadb-12a3-471c-88af-c1509c6332ce.png)

### 4. How many of each type of pizza was delivered?
```sql
SELECT
	p.pizza_name,
	COUNT(c.order_id) as delivered
FROM 
	#cleaned_customer_orders c,
	#cleaned_runner_orders r,
	pizza_names p
WHERE
	c.order_id = r.order_id
	AND
	c.pizza_id = p.pizza_id
	AND
	r.cancellation is null
GROUP BY p.pizza_name;
```
![image](https://user-images.githubusercontent.com/36075516/162097778-c6aab07e-6806-41da-8ba7-5e5eab42e9d6.png)


### 5. How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT
	customer_id,
	pizza_name,
	COUNT(c.order_id) as ordered
FROM 
	#cleaned_customer_orders c,
	pizza_names p
WHERE c.pizza_id = p.pizza_id
GROUP BY 
	customer_id,
	pizza_name
ORDER BY customer_id;
```
![image](https://user-images.githubusercontent.com/36075516/162098207-dade679b-bc12-4caa-bb22-47e534247e6f.png)

### 6. What was the maximum number of pizzas delivered in a single order?
```sql
SELECT TOP(1)
	c.order_id,
	count(pizza_id) as max_pizzas_delivered
FROM 
	#cleaned_customer_orders c,
	#cleaned_runner_orders r
WHERE
	c.order_id = r.order_id
	AND
	r.cancellation is null
GROUP BY c.order_id
ORDER BY 2 DESC;
```
![image](https://user-images.githubusercontent.com/36075516/162098790-8b7ceb88-1bf0-4f8d-b958-3ae02273a534.png)

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT 
	c.customer_id,
	sum(CASE
		WHEN c.exclusions is not null 
			OR c.extras is not null
			THEN 1
		ELSE 0
	END) AS at_least_1_change,
	sum(CASE
		WHEN c.exclusions is null 
			AND c.extras is null
			THEN 1
		ELSE 0
	END) AS no_changes
FROM 
	#cleaned_customer_orders c,
	#cleaned_runner_orders r
WHERE
	c.order_id = r.order_id
	AND
	r.cancellation is null
GROUP BY c.customer_id
```
![image](https://user-images.githubusercontent.com/36075516/162103247-2129c284-fe7e-4dcf-81cb-3fe437657abe.png)

### 8. How many pizzas were delivered that had both exclusions and extras?
```sql
SELECT
	count(pizza_id) as orders_fully_changed
FROM 
	#cleaned_customer_orders c,
	#cleaned_runner_orders r
WHERE
	c.order_id = r.order_id
	AND
	r.cancellation is null
	AND
	c.exclusions is not null
	AND
	c.extras is not null
;
```
![image](https://user-images.githubusercontent.com/36075516/162103847-e54a0718-6c99-4bf3-9dc5-4cd5baae3a19.png)

### 9. What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT 
	DATEPART(HOUR, order_time) as hour_of_day,
	COUNT(pizza_id) as total_orders
FROM #cleaned_customer_orders c
GROUP BY DATEPART(HOUR, order_time);
```
![image](https://user-images.githubusercontent.com/36075516/162107108-da4f3009-d117-4143-96d4-ce30f609df60.png)

### 10. What was the volume of orders for each day of the week?
```sql
SELECT 
	FORMAT(order_time, 'dddd') as day_of_week,
	COUNT(pizza_id) as total_orders
FROM #cleaned_customer_orders c
GROUP BY FORMAT(order_time, 'dddd');
```
![image](https://user-images.githubusercontent.com/36075516/162107205-ca098688-af94-4d44-9d77-4511b6e52718.png)
