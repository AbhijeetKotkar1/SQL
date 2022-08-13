**1. What is the total amount each customer spent at the restaurant?**

``` sql
SELECT 
  s.customer_id, 
  SUM(m.price) AS total_amount_spent 
FROM 
  sales AS s 
  INNER JOIN menu AS m ON s.product_id = m.product_id 
GROUP BY 
  s.customer_id
```

**2. How many days has each customer visited the restaurant?**
``` sql
select 
  customer_id, 
  count(distinct order_date) as times_visited 
from 
  test.dbo.sales 
group by 
  customer_id

```

**3. What was the first item from the menu purchased by each customer?**
``` sql
WITH ordered_sales_ranked as (
select customer_id, order_date, product_name,
DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY s.order_date) AS rank
from test.dbo.sales s inner join test.dbo.menu m on s.product_id = m.product_id
)

SELECT customer_id, product_name
FROM ordered_sales_ranked
WHERE rank = 1
GROUP BY customer_id, product_name;
```

**4. What is the most purchased item on the menu and how many times was it purchased by all customers?**
``` sql
select TOP 1  (COUNT(s.product_id)) as most_purchased, m.product_name 
from test.dbo.sales s
join test.dbo.menu m 
 ON s.product_id = m.product_id
group by s.product_id, m.product_name
ORDER BY most_purchased DESC;
```
