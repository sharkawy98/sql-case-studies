# 🥘 Ingredient Optimisation Solutions

## Data Cleaning for this part of the case study:
### 1. The ingredients of each pizza a stored at `pizza_recipes` table as a comma separated string including ids of all its toppings
* SQL Transformations: using `string_split()` sql function to change comma separated ids into multiple rows, then join data with `pizza_toppings` table to get topping name
	
	```sql
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
	Befor Transformation | After Transformation
	:--:|:--:
	![image](https://user-images.githubusercontent.com/36075516/163698435-9c1322d7-81b4-4c05-aab2-6c69841d0225.png)  |  ![image](https://user-images.githubusercontent.com/36075516/163698478-81f7a565-bb75-4b9e-8f0c-957a0a19d07f.png)
    
### 2. Each order contain 1 or many pizzas so it is difficult to select each one seperately for further analysis
* SQL Transformations: alter `customer_orders_table` and add a new column `record_id` to be able to select each pizza ordered

    ```sql
	ALTER TABLE #cleaned_customer_orders
	ADD record_id INT IDENTITY(1,1);
    ```
	| After Transformation |
	:--:
	|![image](https://user-images.githubusercontent.com/36075516/163920912-256e1f3e-c660-4f48-ab0c-2718472ae054.png)|

### 3. Extras and exclusions of each pizza ordered are stored as comma separated string
* SQL Transformations: using `string_split()` sql function to change comma separated ids into multiple rows

    ```sql
	-- to generate extra table
	SELECT		
		c.record_id,
		TRIM(e.value) AS topping_id
	INTO #extras
	FROM 
		#cleaned_customer_orders as c
		CROSS APPLY string_split(c.extras, ',') as e
	;
	
	-- to generate exclusions table
	SELECT		
		c.record_id,
		TRIM(e.value) AS topping_id
	INTO #exclusions
	FROM 
		#cleaned_customer_orders as c
		CROSS APPLY string_split(c.exclusions, ',') as e
	;
    ```
	Before Transformation| Extras after Transformation | Exclusions after Transformation
	:--:|:--:|:--:
	![image](https://user-images.githubusercontent.com/36075516/163920550-36eb1334-9f9a-401b-8c9a-48fed5c370c4.png) | ![image](https://user-images.githubusercontent.com/36075516/163920694-1b4160d1-4176-4e7e-9f16-5cb2237c6cc8.png) | ![image](https://user-images.githubusercontent.com/36075516/163920746-432c7253-bfdf-4eba-a0ec-a0290c810b70.png)



## Case study questions:
### 1. What are the standard ingredients for each pizza?
```sql
SELECT 
	p.pizza_name,
	STRING_AGG(t.topping_name, ', ') as ingredients
FROM 
	pizza_names p
	JOIN #cleaned_toppings t
	ON p.pizza_id = t.pizza_id
GROUP BY p.pizza_name;
```
![image](https://user-images.githubusercontent.com/36075516/163697973-de25a357-7662-40b3-8c33-d6cccbed0832.png)

### 2. What was the most commonly added extra?
```sql
SELECT 
	topping_name,
	count(transaction_id) as num_of_addition
FROM 
	#extras e,
	#cleaned_toppings t
WHERE e.topping_id = t.topping_id
GROUP BY topping_name
ORDER BY 2 DESC;
```
![image](https://user-images.githubusercontent.com/36075516/163752964-8d1eae17-0132-4119-972f-33a2e80738d8.png)

### 3. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

#### My approach to solve this interesting question:
I created 3 CTEs `extras_cte`, `exclusions_cte` and `union_cte` the later was created; to combine record's extras and exclusions if exist as a queryable result to make `LEFT JOIN` with `customer_orders` table to get final result
```sql
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
`extras_cte`

![image](https://user-images.githubusercontent.com/36075516/163838926-c88bfc46-847e-48db-87ad-f2dc2fddca35.png)

`exclusions_cte`

![image](https://user-images.githubusercontent.com/36075516/163838984-e49242cd-1bde-4cfb-affd-7889076442dd.png)

`union_cte`

![image](https://user-images.githubusercontent.com/36075516/163840580-2193ba94-35e5-4248-bc5e-b809c56cbe6d.png)

#### The desired final result 🎉

![image](https://user-images.githubusercontent.com/36075516/163838831-32ac201b-01e2-4dd5-a7f2-5be16de3714a.png)


### 4. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
#### My explanation & approach:
- This question wants to add 2x symbol before each topping added as an extra ingredient and remove exclusions by the way
- I created a CTE to combine all the toppings (added & excluded) with relevant data to get the desired result

```sql
WITH ingredients_cte AS
(
	SELECT
	c.record_id, 
	p.pizza_name,
	CASE
		WHEN t.topping_id 
		IN (select topping_id from #extras e where c.record_id = e.record_id)
		THEN '2x' + t.topping_name
		ELSE t.topping_name
	END as topping
	FROM 
		#cleaned_customer_orders c
		JOIN pizza_names p
			ON c.pizza_id = p.pizza_id
		JOIN #cleaned_toppings t 
			ON c.pizza_id = t.pizza_id
	WHERE t.topping_id NOT IN (select topping_id from #exclusions e where c.record_id = e.record_id)
)

SELECT 
	record_id,
	CONCAT(pizza_name+':',STRING_AGG(topping, ', ')) as ingredients_list
FROM ingredients_cte
GROUP BY 
	record_id,
	pizza_name
ORDER BY 1;
```
Part from `ingredients_cte` result

![image](https://user-images.githubusercontent.com/36075516/163913008-4d41e650-f8e1-4907-9a74-6fce4b153a8f.png)

__Final result__

![image](https://user-images.githubusercontent.com/36075516/163912894-adf9759e-e607-430e-99da-dafc79d2b476.png)



### 5. What is the total quantity of each ingredient used in all ordered pizzas sorted by most frequent first?
#### My explanation & approach:
- This question wants to show each topping and the number of times it is used in ordered pizzas including extras & exclusion
- I created a CTE to automate the corresponding addition for each ordered pizza topping 
```sql
WITH ingredients_cte AS
(
SELECT 
	c.record_id,
	t.topping_name,
	CASE
		-- if extra ingredient add 2
		WHEN t.topping_id 
		IN (select topping_id from #extras e where e.record_id = c.record_id) 
		THEN 2
		-- if excluded ingredient add 0
		WHEN t.topping_id 
		IN (select topping_id from #exclusions e where e.record_id = c.record_id) 
		THEN 0
		-- normal ingredient add 1
		ELSE 1 
	END as times_used
	FROM   
		#cleaned_customer_orders AS c
		JOIN #cleaned_toppings AS t
		ON c.pizza_id = t.pizza_id
) 

SELECT 
    topping_name,
    SUM(times_used) AS times_used 
FROM ingredients_cte
GROUP BY topping_name
ORDER BY 2 DESC;
```
Part of `ingredients_cte` result

![image](https://user-images.githubusercontent.com/36075516/163917632-121697b9-3e22-4665-9828-da2fc5ff9b1c.png)

__Final result__

![image](https://user-images.githubusercontent.com/36075516/163917489-b9a38c8f-6358-4b0f-86de-388a2bdd0a53.png)
