## 1. Join All The Things, Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
```sql
WITH join_all_cte AS(
    SELECT
        s.customer_id,
        men.product_name,
        men.price,
        s.order_date,
        mem.join_date,
        CASE
            WHEN s.order_date >= mem.join_date THEN 'Y'
            ELSE 'N'
        END as "is_member"
    FROM
        menu men,
        sales s
        LEFT JOIN members mem
        ON s.customer_id = mem.customer_id
    WHERE men.product_id = s.product_id
)

SELECT * FROM join_all_cte;
```


## 2. Danny also requires further information about the ranking of customer products, but he purposely does not need the ranking for non-member purchases so he expects null ranking values for the records when customers are not yet part of the loyalty program.
```sql
SELECT
	*,
	CASE
		WHEN [is_member] = 'N' THEN null
		ELSE
			dense_rank() over (partition by customer_id, [is_member]
			order by order_date)
	END as rank
FROM join_all_cte;
```