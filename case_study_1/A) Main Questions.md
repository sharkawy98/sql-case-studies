## 1. What is the total amount each customer spent at the restaurant?
```sql
SELECT
	customer_id,
	sum(price) as spent
FROM
	sales s,
	menu m
WHERE m.product_id = s.product_id
GROUP BY customer_id;
```
| customer_id | spent |
|-------------|-------|
| A           | 76    |
| B           | 74    |
| C           | 36    |


## 2. How many days has each customer visited the restaurant?
```sql
SELECT
	customer_id,
	count(DISTINCT order_date) as visits
FROM sales 
GROUP BY customer_id;
```
| customer_id | visits |
|-------------|--------|
| A           | 4      |
| B           | 6      |
| C           | 2      |


## 3. What was the first item from the menu purchased by each customer?
```sql
WITH cust_orders_cte AS(
	SELECT
		customer_id,
		order_date,
		product_name,
		row_number() over (partition by customer_id 
		order by order_date) as rank
	FROM
		sales s,
		menu m
	WHERE m.product_id = s.product_id
)

SELECT 
	customer_id,
	product_name
FROM cust_orders_cte
WHERE rank = 1;
```
| customer_id | product_name |
|-------------|--------------|
| A           | sushi        |
| B           | curry        |
| C           | ramen        |


## 4. What is the most purchased item on the menu and how many times was it purchased by all customers?
```sql
SELECT 
	top(1)
	product_name,
	count(s.product_id) as purchased
FROM
	sales s,
	menu m
WHERE m.product_id = s.product_id
GROUP BY product_name
ORDER BY purchased DESC;
```
| product_name | purchased |
|--------------|-----------|
| ramen        | 8         |


## 5. Which item was the most popular for each customer?
```sql
WITH fav_item_cte AS(
	SELECT
		customer_id,
		product_name,
		count(s.product_id) as ordered,
		dense_rank() over (partition by customer_id 
		order by count(s.product_id) desc) as rank
	FROM
		sales s,
		menu m
	WHERE m.product_id = s.product_id
	GROUP BY 
		customer_id,
		product_name
)

SELECT 
	customer_id,
	product_name,
	ordered
FROM fav_item_cte
WHERE rank = 1;
```
| customer_id | product_name | ordered |
|-------------|--------------|---------|
| A           | ramen        | 3       |
| B           | sushi        | 2       |
| B           | curry        | 2       |
| B           | ramen        | 2       |
| C           | ramen        | 3       |


## 6. Which item was purchased first by the customer after they became a member?
```sql
WITH memb_orders_cte AS(
	SELECT
		s.customer_id,
		order_date,
		join_date,
		product_id,
		row_number() over (partition by s.customer_id
		order by order_date) as rank
	FROM
		sales s,
		members m
	WHERE
		m.customer_id = s.customer_id
		AND
		order_date >= join_date
)

SELECT 
	customer_id,
	product_name,
	order_date, 
	join_date
FROM 
	memb_orders_cte mo,
	menu m
WHERE 
	m.product_id = mo.product_id
	AND
	rank = 1
;
```
| customer_id | product_name | order_date | join_date  |
|-------------|--------------|------------|------------|
| A           | curry        | 2021-01-07 | 2021-01-07 |
| B           | sushi        | 2021-01-11 | 2021-01-09 |


## 7. Which item was purchased just before the customer became a member?
```sql
WITH before_memb_orders_cte AS(
	SELECT
		s.customer_id,
		order_date,
		join_date,
		product_id,
		row_number() over (partition by s.customer_id
		order by order_date desc) as rank
	FROM
		sales s,
		members m
	WHERE
		m.customer_id = s.customer_id
		AND
		order_date < join_date
)

SELECT 
	customer_id,
	product_name,
	order_date, 
	join_date
FROM 
	before_memb_orders_cte mo,
	menu m
WHERE 
	m.product_id = mo.product_id
	AND
	rank = 1
;
```
| customer_id | product_name | order_date | join_date  |
|-------------|--------------|------------|------------|
| A           | sushi        | 2021-01-01 | 2021-01-07 |
| B           | sushi        | 2021-01-04 | 2021-01-09 |


## 8. What is the total items and amount spent for each member before they became a member?
```sql
SELECT
	s.customer_id,
	count(s.product_id) as items,
	sum(men.price) as spent
FROM
	sales s,
	members mem,
	menu men
WHERE
	mem.customer_id = s.customer_id
	AND
	men.product_id = s.product_id
	AND
	order_date < join_date
GROUP BY s.customer_id;
```
| customer_id | items | spent |
|-------------|-------|-------|
| A           | 2     | 25    |
| B           | 3     | 40    |


## 9. If each $1 spent equates to 10 points and sushi has a 2x points multiplier - how many points would each customer have?
```sql
SELECT
	customer_id,
	sum(CASE
		WHEN s.product_id = 1 THEN price*20
		ELSE price*10 
	END) as total_points
FROM
	sales s,
	menu m
WHERE m.product_id = s.product_id
GROUP BY customer_id;
```
| customer_id | total_points |
|-------------|--------------|
| A           | 860          |
| B           | 940          |
| C           | 360          |


## 10. In the first week after a customer joins the program (including their join date) they earn 2x points on all items, not just sushi - how many points do customer A and B have at the end of January?
```sql
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
| customer_id | total_points |
|-------------|--------------|
| A           | 1370         |
| B           | 820          |
