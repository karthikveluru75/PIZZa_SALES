KRTHIK_PIZZA is a data-driven SQL project that explores pizza sales performance using structured queries across multiple tables. It leverages advanced SQL techniques to uncover insights like revenue trends, order patterns, and top-performing pizzas by category.

🎯 Vision & Mission
Vision: To empower data-driven decisions in the pizza industry through clear, actionable insights. Mission: To deliver precise, ranked sales intelligence by integrating pizza types, pricing, and order data into one powerful query.

📊 Data Sources
pizza_types.csv: Contains pizza names, categories, and ingredients.
pizzas.csv: Includes pizza sizes and prices.
order_details.csv: Tracks pizza orders and quantities.
orders.csv: (Blocked, but assumed to contain order timestamps and IDs)

🧠 Key Questions Answered
Your queries answer a rich set of business questions, including:

🔹 Basic Analysis

1.How many total orders were placed?
SELECT 
    COUNT(order_id) AS total_orders
FROM
    orders;

    
2.What is the total revenue generated from pizza sales?
SELECT 
    ROUND(SUM(order_details.quantity * pizzas.price),
            2) AS total_sales
FROM
    order_details
        JOIN
    pizzas ON pizzas.pizza_id = order_details.pizza_id;

    
3.Which pizza has the highest price?
SELECT 
    pizza_types.name, pizzas.price
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
ORDER BY pizzas.price DESC
LIMIT 1;


3.What is the most commonly ordered pizza size?
SELECT 
    pizzas.size,
    COUNT(order_details.order_details_id) AS order_count
FROM
    pizzas
        JOIN
    order_details ON pizzas.pizza_id = order_details.pizza_id
GROUP BY pizzas.size
ORDER BY order_count DESC;


4.What are the top 10 most ordered pizza types?
SELECT 
    pizza_types.name,
    SUM(order_details.quantity) AS quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.name
ORDER BY quantity DESC limit 10;


🔸 Intermediate Analysis

1.What is the total quantity ordered per pizza category?
SELECT 
    pizza_types.category,
    SUM(order_details.quantity) AS quantity
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY quantity DESC;


2.How are orders distributed by hour of the day?
SELECT 
    HOUR(order_time) AS hour, COUNT(order_id) AS order_count
FROM
    orders
GROUP BY HOUR(order_time);


3.How many pizza types exist in each category?
SELECT 
    category, COUNT(name)
FROM
    pizza_types
GROUP BY category;


4.What is the average number of pizzas ordered per day?
SELECT 
    ROUND(AVG(quantity), 0) avg_pizzas_ordered_per_day
FROM
    (SELECT 
        orders.order_date, SUM(order_details.quantity) AS quantity
    FROM
        orders
    JOIN order_details ON orders.order_id = order_details.order_id
    GROUP BY orders.order_date) AS order_quantity;

    

🔺 Advanced Analysis

1.What are the top 3 pizza types by revenue?
SELECT 
    pizza_types.category,
    SUM(order_details.quantity * pizzas.price) / (SELECT 
            ROUND(SUM(order_details.quantity * pizzas.price),
                        2) AS total_sales
        FROM
            order_details
                JOIN
            pizzas ON pizzas.pizza_id = order_details.pizza_id) * 100 AS revenue
FROM
    pizza_types
        JOIN
    pizzas ON pizza_types.pizza_type_id = pizzas.pizza_type_id
        JOIN
    order_details ON order_details.pizza_id = pizzas.pizza_id
GROUP BY pizza_types.category
ORDER BY revenue DESC;


2.What is each category’s percentage contribution to total revenue?
select order_date, sum(revenue) over (order by order_date) as cum_revenue 
from (select orders.order_date, sum(order_details.quantity*pizzas.price) as revenue 
from order_details join pizzas on order_details.pizza_id = pizzas.pizza_id 
join orders on orders.order_id = order_details.order_id group by orders.order_date) as sales;

3.How does cumulative revenue grow over time?
SELECT * 
FROM (
    SELECT pt.pizza_type_id, pt.name, pt.category,
           SUM(od.quantity * p.price) AS revenue,
           RANK() OVER (PARTITION BY pt.category ORDER BY SUM(od.quantity * p.price) DESC) AS ranking
    FROM order_details od
    JOIN pizzas p ON p.pizza_id = od.pizza_id
    JOIN pizza_types pt ON pt.pizza_type_id = p.pizza_type_id
    GROUP BY pt.pizza_type_id, pt.name, pt.category
) AS ranked_pizzas
WHERE ranking <= 5;
