# 🙂 Runner and Customer Experience Solutions

### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT 
	DATEPART(WEEK, registration_date) AS [week],
	COUNT(runner_id) AS runners_signed_up
FROM   runners
GROUP  BY DATEPART(WEEK, registration_date);
```
![image](https://user-images.githubusercontent.com/36075516/162617145-8eb9219c-74ce-4875-84e8-ac22ae64aa8d.png)

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
SELECT 
	r.runner_id,
	AVG(DATEDIFF(MINUTE, c.order_time, r.pickup_time)) AS avg_time_to_hq
FROM   
	#cleaned_runner_orders r,
	#cleaned_customer_orders c
WHERE c.order_id = r.order_id
GROUP  BY r.runner_id;
```
![image](https://user-images.githubusercontent.com/36075516/162617256-80eb919e-4092-415b-9cb2-a4d047d34693.png)

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
WITH prep_time_cte AS
(
	SELECT 
		c.order_id,
		COUNT(c.pizza_id) AS num_of_pizzas,
		DATEDIFF(MINUTE, c.order_time, r.pickup_time) AS time_taken_per_order,
		DATEDIFF(MINUTE, c.order_time, r.pickup_time) / COUNT(c.pizza_id) AS time_taken_per_pizza
	FROM   
		#cleaned_runner_orders r,
		#cleaned_customer_orders c
	WHERE c.order_id = r.order_id
	GROUP BY 
		c.order_id,
		c.order_time,
		r.pickup_time
)
SELECT 
	num_of_pizzas,
	AVG(time_taken_per_order) AS avg_total_time_taken,
	AVG(time_taken_per_pizza) AS avg_time_taken_per_pizza
FROM prep_time_cte
GROUP BY num_of_pizzas;
```
![image](https://user-images.githubusercontent.com/36075516/162618848-9e94c83d-29f0-4b1e-bbb4-717b4aee0c5e.png)
> We can deduce that the order preparation time increases if the number of pizzas ordered increases

### 4. What was the average distance travelled for each customer?
```sql
SELECT 
	c.customer_id,
	ROUND(AVG(r.distance), 2) AS avg_distance
FROM   
	#cleaned_runner_orders r,
	#cleaned_customer_orders c
WHERE c.order_id = r.order_id
GROUP BY c.customer_id;
```
![image](https://user-images.githubusercontent.com/36075516/162619267-e774b0fe-bae3-4511-9603-3d70e0f5932b.png)

### 5. What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT 
	MAX(duration) AS max_delivery_time,
	MIN(duration) AS min_delivery_time,
	MAX(duration) - MIN(duration) AS time_difference
FROM #cleaned_runner_orders;
```
![image](https://user-images.githubusercontent.com/36075516/162619453-74d0f208-2515-42b4-ae1d-8196eac82e4f.png)

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
```sql
SELECT 
	runner_id,
	distance,
	duration,
	ROUND(distance/ duration * 60, 2) AS speed_km_hr
FROM #cleaned_runner_orders
WHERE cancellation is null
ORDER BY 
	runner_id,
	speed_km_hr
;
```
![image](https://user-images.githubusercontent.com/36075516/162626242-80a6fc26-0b06-4d4c-8d3a-b44db91c8851.png)
> I noticed that as runner becomes older, he goes faster

### 7. What is the successful delivery percentage for each runner?
```sql
SELECT 
	runner_id,	
	count(order_id) as total_orders,
	count(pickup_time) as total_orders_delivered,
	cast(count(pickup_time) as float) / cast(count(order_id) as float) * 100 
		as successful_delivery_percent
FROM #cleaned_runner_orders
GROUP BY runner_id;
```
![image](https://user-images.githubusercontent.com/36075516/162653779-a967bb29-d707-437c-99e3-440c04c2a610.png)
