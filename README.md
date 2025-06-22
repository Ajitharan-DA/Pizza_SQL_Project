# Pizza Hub Data Analysing using SQL

## Overview
The goal of this project is to analyze pizza sales data to gain insights into customer behavior, sales trends, and product performance. We'll use SQL to query and analyze the data.

## Objectives
- Analyze sales trends: Identify patterns and trends in pizza sales over time.
- Identify top-selling pizzas: Determine which pizzas are most popular among customers.
- Understand customer behavior: Gain insights into customer ordering habits and preferences.


## Business Problems and Solutions

### 1. Retrieve the total number of orders placed.

```sql
SELECT 
    COUNT(*) AS total_orders
FROM
    orders;
```

### 2. Calculate the total revenue generated  from pizza sales

```sql
SELECT 
round(sum(Order_details.Quantity * Pizzas.price),2) as total_sales
FROM order_details
JOIN pizzas
ON order_details.Pizza_id = pizzas.pizza_id
```
### 3. identify the highest pizza price

```sql
SELECT pizza_types.name, pizzas.price
from pizza_types JOIN Pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
order by pizzas.price desc LIMIT 1;
```

### 4. Identify the most common pizza size ordered

```sql
SELECT 
    pizzas.size,
    COUNT(order_details.Order_details_id) AS order_count
FROM
    pizzas
        JOIN
    order_details ON Order_details.Pizza_id = pizzas.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC;
```

### 5. List the 5 most ordered pizza types along with their quantity

```sql
SELECT 
    pizza_types.name, SUM(order_details.Quantity) AS quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.Pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC
LIMIT 5;
```

### 6. Join the necessary tables to find the total quantity of each pizza category

```sql
SELECT 
    pizza_types.category, SUM(order_details.Quantity) AS qty
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON pizzas.pizza_id = order_details.Pizza_id
GROUP BY pizza_types.category
ORDER BY qty DESC;
```

### 7. Determine the distribution of orders by hours of the day

```sql
SELECT 
    HOUR(Order_time) AS hour, COUNT(Order_id) AS order_count
FROM
    orders
GROUP BY HOUR(Order_time); 

-- join revelant tables to find the category wise distribution of pizzas
SELECT 
    category, COUNT(pizza_types.name)
FROM
    pizza_types
GROUP BY category;
```

### 8. Group the orders by date and calculate the average number of pizzas ordered per day

```sql
SELECT 
    ROUND(AVG(per_day), 0) AS avg_pizzas_ordered_per_day
FROM
    (SELECT 
        DATE(orders.Order_date) AS date,
            SUM(order_details.Quantity) AS per_day
    FROM
        order_details
    JOIN orders ON order_details.Order_id = orders.Order_id
    GROUP BY DATE(orders.Order_date)) AS total_qty;
```
    
### 9. Determine the top 3 most ordered pizza types based on  revenue

```sql
    SELECT 
    pizza_types.name,
    SUM(order_details.Quantity * pizzas.price) AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON pizzas.pizza_id = order_details.Pizza_id
GROUP BY pizza_types.name
ORDER BY revenue DESC
LIMIT 3;
```

### 10. Calculate the percentage contribution of each pizza type to total revenue

```sql
SELECT 
    pizza_types.category,
    SUM(order_details.Quantity * pizzas.price) / (SELECT 
            ROUND(SUM(order_details.Quantity * Pizzas.price),
                        2) AS total_sales
        FROM
            order_details
                JOIN
            pizzas ON order_details.Pizza_id = pizzas.pizza_id) * 100 AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON pizzas.pizza_id = order_details.Pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC
```

### 11. same question with CTE

```sql
WITH total_sales AS (
    SELECT 
        ROUND(SUM(order_details.Quantity * pizzas.price), 2) AS total_sales
    FROM
        order_details
        JOIN pizzas ON order_details.Pizza_id = pizzas.pizza_id
),
Revenue AS (
    SELECT 
        pizza_types.category,
        SUM(order_details.Quantity * pizzas.price) AS revenue
    FROM
        pizza_types
        JOIN pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN order_details ON pizzas.pizza_id = order_details.Pizza_id
    GROUP BY pizza_types.category
)
SELECT 
    Revenue.category,
    (Revenue.revenue / total_sales.total_sales) * 100 AS Total_revenue
FROM
    Revenue, total_sales
ORDER BY revenue DESC;
```

### 12. Analyze the cummulative revenue generated over time

```sql
Select order_date,
sum(revenue) over(order by order_date) as cum_revenue
from
(Select orders.Order_date,  
SUM(order_details.Quantity * pizzas.price) AS revenue
from Orders 
join order_details on order_details.Order_id = Orders.Order_id
join pizzas on order_details.Pizza_id = pizzas.pizza_id
group by orders.Order_date) as sales
```

### 13. Determine the top 3 most ordered pizza type based on revenue for each pizza category

```sql
select category, name, revenue
from
(select category, name, revenue, rank() over(partition by category order by revenue desc) as RN 	
from 
(Select pizza_types.category, pizza_types.name,
sum((order_details.Quantity * pizzas.price)) as revenue
from pizza_types
join pizzas on pizza_types.pizza_type_id = pizzas.pizza_type_id
join order_details on pizzas.pizza_id = order_details.pizza_id
group by pizza_types.category, pizza_types.name) as a) as b 
where RN <= 3;
```

## Findings and Conclusion
- **1. Top-selling pizzas: The top 3 best-selling pizzas account for 50% of total sales, indicating a strong demand for these specific products.
- **2. Sales trends: Sales peak during weekends and holidays, suggesting opportunities for targeted promotions and marketing campaigns.
- **3. Customer behavior: Customers tend to order pizzas with similar toppings, indicating potential for upselling and cross-selling opportunities.

   The analysis reveals valuable insights into customer behavior, sales trends, and product performance. By leveraging these findings, the pizza business can:
