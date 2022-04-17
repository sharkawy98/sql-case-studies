# 🥘 Ingredient Optimisation Solutions

## Data Cleaning for this part of the case study:
### 1. The ingredients of each pizza a stored at `pizza_recipes` table as a comma separated string including ids of all its toppings
* Data before transformations
    
    ![image](https://user-images.githubusercontent.com/36075516/163698435-9c1322d7-81b4-4c05-aab2-6c69841d0225.png)
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
* Data after transformations
    
    ![image](https://user-images.githubusercontent.com/36075516/163698478-81f7a565-bb75-4b9e-8f0c-957a0a19d07f.png)


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
```

### 3. What was the most common exclusion?
```sql
```

### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
```sql
```

### 5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
- For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
```sql
```

### 6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
```sql
```
