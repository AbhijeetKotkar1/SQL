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

**5. Which item was the most popular for each customer?**
``` sql
WITH rank_cte_group AS
(
 SELECT s.customer_id, m.product_name, 
  COUNT(m.product_id) AS order_count,
  DENSE_RANK() OVER(PARTITION BY s.customer_id
  ORDER BY COUNT(s.customer_id) DESC) AS rank
FROM test.dbo.menu AS m
JOIN test.dbo.sales AS s
 ON m.product_id = s.product_id
GROUP BY s.customer_id, m.product_name
)

SELECT customer_id, product_name, order_count
FROM rank_cte_group 
WHERE rank = 1;

```

**6. Which item was purchased first by the customer after they became a member?**
``` sql
WITH cte as (
select  s.customer_id, me.product_name, DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date) as rank
from test.dbo.sales s 
inner join test.dbo.members m 
on s.customer_id = m.customer_id
inner join test.dbo.menu me
on s.product_id = me.product_id
where order_date >= join_date
)

select customer_id, product_name from cte where rank = 1
```

**7. Which item was purchased just before the customer became a member?**
``` sql
WITH cte as (
select  s.customer_id, me.product_name, DENSE_RANK() OVER (PARTITION BY s.customer_id ORDER BY order_date desc) as rank
from test.dbo.sales s 
inner join test.dbo.members m 
on s.customer_id = m.customer_id
inner join test.dbo.menu me
on s.product_id = me.product_id
where order_date < join_date
)

select customer_id, product_name from cte where rank = 1
```
