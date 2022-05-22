# 🚶 Customer Journey Solutions

## Question
Based off the 8 sample customers provided in the sample from the subscriptions table (customer_id 1, 2, 11, 13, 15, 16, 18, 19), 
write a brief description about each customer’s onboarding journey. Try to keep it as short as possible - you may also want to run 
some sort of join to make your explanations a bit easier!


## My approach to solve this question
I joined the two tables `subscriptions`, `plans` and selected only the required sample. In addition, I added a derived column `days_difference` 
from the `start_date` column by getting the difference between start_date of each plan and the plan before it directly for each customer, 
by using the window function `LAG()`. This derived column will provide further insights.

### Solution
```sql
SELECT 
  s.customer_id,
  p.plan_name,
  s.start_date,
  DATEDIFF(day, 
    LAG(start_date) OVER (PARTITION BY customer_id ORDER BY start_date), 
    start_date
  ) as days_difference
FROM  
  subscriptions s
  JOIN   plans p
  ON s.plan_id = p.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19);
```
### Result
![image](https://user-images.githubusercontent.com/36075516/169678298-f250db1b-affa-4417-9f57-22922c64e6b1.png)

### Customer’s onboarding journey
- Customer 1 signed up for a free trial on 1/8/2020 and downgraded to the basic monthly plan after the 7 days of free trial immediately.
- Customer 2 signed up for a free trial on 20/9/2020 and upgraded to the pro annual plan after the 7 days of free trial immediately.
- Customer 11 signed up for a free trial on 19/11/2020 and canceled his subscription after the 7 days of free trial immediately.
- Customer 13 signed up for a free trial on 15/12/2020, downgraded to the basic monthly plan right after the 7 days of free trial 
and upgraded to the pro monthly plan after 97 days (about 3 months later).
- The journey for the rest of customers is totally illustrated at the above table [⬆️](#result)
