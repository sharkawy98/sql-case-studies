# 🔄 Data Cleaning & Transformations
I will generate new cleaned tables to be used for further analysis answering case study questions

## customer_orders 
* Table before cleaning:

![image](https://user-images.githubusercontent.com/36075516/162091921-a7232bd2-aebc-4a43-8c2e-e5f64e0d53bd.png)

* SQL Transformations:
```sql
SELECT 
    order_id,
    customer_id,
    pizza_id,
    CASE
        WHEN exclusions = 'null' THEN null
        ELSE exclusions
    END as exclusions,
    CASE
        WHEN extras = 'null' THEN null
        ELSE extras
    END as extras,
    order_time
INTO #cleaned_customer_orders
FROM customer_orders;
```

* Table after cleaning:

![image](https://user-images.githubusercontent.com/36075516/162092976-7eb8e3d2-0330-456d-80ee-06fcbeefcf3c.png)



## runner_orders 
* Table before cleaning:

![image](https://user-images.githubusercontent.com/36075516/162093228-f3301873-9201-46ec-a6ed-236cc48465eb.png)

* SQL Transformations:
```sql
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

* Table after cleaning:

![image](https://user-images.githubusercontent.com/36075516/162093341-c8f238f5-297c-4417-92ef-f38c92adf229.png)
