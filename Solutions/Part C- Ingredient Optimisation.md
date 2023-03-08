# Questions

1. What are the standard ingredients for each pizza?
2. What was the most commonly added extra?
3. What was the most common exclusion?
4. Generate an order item for each record in the customers_orders table in the format of one of the following:
  - Meat Lovers
  - Meat Lovers - Exclude Beef
  - Meat Lovers - Extra Bacon
  - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers
5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
  - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"
6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?


# Creating Temp Tables

--Create a new temporary table extrasBreak to separate extras into multiple rows
~~~sql
DROP TABLE IF EXISTS extrasBreak;

CREATE TEMPORARY TABLE extrasBreak AS
SELECT 
  c.record_id,
  CAST(TRIM(e.extras) AS INTEGER) AS extra_id
FROM pizza_runner.customer_orders c
LEFT JOIN LATERAL unnest(string_to_array(c.extras, ',')) AS e(extras) ON true;


SELECT *
FROM extrasBreak
~~~
![](https://github.com/imanjokko/PizzaRunner/blob/main/images/extrasBreak.png)

--Create a new temporary table exclusionsBreak to separate exclusions into multiple rows
~~~sql
DROP TABLE IF EXISTS exclusionsBreak;

CREATE TEMPORARY TABLE exclusionsBreak AS
SELECT 
  c.record_id,
  CAST(TRIM(e.exclusions) AS INTEGER) AS exclusions_id
FROM pizza_runner.customer_orders c
LEFT JOIN LATERAL unnest(string_to_array(c.exclusions, ',')) AS e(exclusions) ON true;


SELECT *
FROM exclusionsBreak
~~~
![](https://github.com/imanjokko/PizzaRunner/blob/main/images/exclusionsBreak.png)

----Create a new temporary table toppingsBreak to separate toppings into multiple rows
~~~sql
DROP TABLE IF EXISTS toppingsBreak

CREATE TEMP TABLE toppingsBreak AS
SELECT
  pr.pizza_id,
  CAST(TRIM(t.value) AS INTEGER) AS topping_id,
  pt.topping_name
FROM
  pizza_runner.pizza_recipes pr
CROSS JOIN LATERAL
  unnest(string_to_array(pr.toppings, ',')) t(value)
JOIN
  (SELECT DISTINCT topping_id, topping_name FROM pizza_runner.pizza_toppings) pt
ON CAST(TRIM(t.value) AS INTEGER) = pt.topping_id
GROUP BY pr.pizza_id, t.value, pt.topping_name
ORDER BY pr.pizza_id;


SELECT *
FROM toppingsBreak;
~~~
![](https://github.com/imanjokko/PizzaRunner/blob/main/images/toppingsBreak.png)

# Solutions

1. What are the standard ingredients for each pizza?
~~~sql
WITH CTE AS (
    SELECT pn.pizza_name, n.pizza_id, pt.topping_name
    FROM pizza_runner.new_recipes AS n
    INNER JOIN pizza_runner.pizza_toppings AS pt
        ON n.toppings = pt.topping_id
    INNER JOIN pizza_runner.pizza_names AS pn
        ON pn.pizza_id = n.pizza_id
    ORDER BY pn.pizza_name, n.pizza_id
)
SELECT pizza_name, string_agg(topping_name, ', ') as std_toppings
FROM CTE
GROUP BY pizza_name;
~~~
 pizza_name | std_toppings
:----------:|:-------------:
 Meatlovers |BBQ Sauce, Pepperoni, Cheese, Salami, Chicken, Bacon, Mushrooms, Beef
 Vegetarian |Tomato Sauce, Cheese, Mushrooms, Onions, Peppers, Tomatoes
 
 ---
2. What was the most commonly added extra?
~~~sql
SELECT COUNT (coc.extras),
		pt.topping_name
FROM pizza_runner.customerorders_cleaned AS coc
INNER JOIN pizza_runner.pizza_toppings AS pt
ON coc.extras = pt.topping_id
GROUP BY extras, topping_name
ORDER BY count DESC
LIMIT 1;
~~~
  count | topping_name
:------:|:-------------:
  4     | Bacon
  
---
3. What was the most common exclusion?
~~~sql
SELECT COUNT (coc.exclusions),
		pt.topping_name
FROM pizza_runner.customerorders_cleaned AS coc
INNER JOIN pizza_runner.pizza_toppings AS pt
ON coc.exclusions = pt.topping_id
GROUP BY exclusions, topping_name
ORDER BY count DESC
LIMIT 1;
~~~
  count | topping_name
:------:|:--------------:
 4 	| Cheese
 
---
4. Generate an order item for each record in the customers_orders table in the format of one of the following:
   - Meat Lovers
   - Meat Lovers - Exclude Beef
   - Meat Lovers - Extra Bacon
   - Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

To solve this question: 
- Add a record_id to the customer_orders table to select each ordered pizza more easily
- I then created temp tables, see the *Creating Temp Tables* section above for the queries used
- Create 3 CTEs: extras_cte, exclusions_cte, and union_cte combining two tables
- Use the union_cte to LEFT JOIN with the customer_orders table and JOIN with the pizza_name table
- Use the CONCAT_WS with STRING_AGG to get the result

--adding record id column
~~~sql
ALTER TABLE pizza_runner.customer_orders
ADD COLUMN record_id SERIAL

SELECT *
FROM pizza_runner.customer_orders;
~~~
 Old Table                                    | Altered Table
:--------------------------------------------:|:--------------:
  ![](https://github.com/imanjokko/PizzaRunner/blob/main/images/customerordersaltered.png)|![](https://github.com/imanjokko/PizzaRunner/blob/main/images/customerordersrecordid.png)


~~~SQL
WITH cteExtras AS 
(SELECT
e.record_id,
'Extra ' || STRING_AGG(t.topping_name, ', ') AS record_options
FROM extrasBreak AS e
JOIN pizza_runner.pizza_toppings AS t
ON e.extra_id = t.topping_id
GROUP BY e.record_id),
cteExclusions AS 
(SELECT
e.record_id,
'Exclusion ' || STRING_AGG(t.topping_name, ', ') AS record_options
FROM exclusionsBreak AS e
JOIN pizza_runner.pizza_toppings AS t
ON e.exclusions_id = t.topping_id
GROUP BY e.record_id),
cteUnion AS 
(SELECT * FROM cteExtras
UNION
SELECT * FROM cteExclusions)

SELECT
c.record_id,
c.order_id,
c.customer_id,
c.pizza_id,
c.order_time,
CONCAT_WS(' - ', p.pizza_name, STRING_AGG(u.record_options, ' - ')) AS pizza_info
FROM pizza_runner.customer_orders c
LEFT JOIN cteUnion u
ON c.record_id = u.record_id
JOIN pizza_runner.pizza_names p
ON c.pizza_id = p.pizza_id
GROUP BY
c.record_id,
c.order_id,
c.customer_id,
c.pizza_id,
c.order_time,
p.pizza_name
ORDER BY record_id;
~~~
![](https://github.com/imanjokko/PizzaRunner/blob/main/images/SectionCno5.png)

5. Generate an alphabetically ordered comma separated ingredient list for each pizza order from the customer_orders table and add a 2x in front of any relevant ingredients
  - For example: "Meat Lovers: 2xBacon, Beef, ... , Salami"

To solve this question:
- Create a CTE in which each line displays an ingredient for an ordered pizza (add '2x' for extras and remove exclusions as well)
  - Use CONCAT and STRING_AGG to get the result

~~~sql
WITH ingredients AS (
  SELECT 
    co.record_id,
    co.order_id,
    co.customer_id,
    co.pizza_id,
    co.order_time,
    pn.pizza_name,
    CASE 
      WHEN t.topping_id::int IN 
	( SELECT eb.extra_id 
        FROM extrasBreak eb 
        WHERE eb.record_id = co.record_id) THEN '2x ' || pt.topping_name
      ELSE pt.topping_name
    END AS topping
  FROM 
    pizza_runner.customer_orders co
    LEFT JOIN pizza_runner.pizza_names pn ON co.pizza_id = pn.pizza_id
    LEFT JOIN pizza_runner.pizza_recipes pr ON co.pizza_id = pr.pizza_id
    LEFT JOIN unnest(string_to_array(pr.toppings, ',')) AS t(topping_id) ON TRUE
    LEFT JOIN pizza_runner.pizza_toppings pt ON t.topping_id::int = pt.topping_id
  WHERE 
    NOT EXISTS 
	(SELECT 1 
      FROM exclusionsBreak eb 
      WHERE co.record_id = eb.record_id 
      AND pt.topping_id = eb.exclusions_id))
SELECT 
  record_id,
  order_id,
  customer_id,
  pizza_id,
  order_time,
  CONCAT(pizza_name, ': ', STRING_AGG(topping, ', ')) AS ingredients_list
FROM ingredients
GROUP BY 
  record_id, 
  order_id,
  customer_id,
  pizza_id,
  order_time,
  pizza_name
ORDER BY 
  record_id;
~~~
![](https://github.com/imanjokko/PizzaRunner/blob/main/images/sectioncno6.png)

6. What is the total quantity of each ingredient used in all delivered pizzas sorted by most frequent first?
To solve this question:
- Create a CTE to record the number of times each ingredient was used
  - if extra ingredient, add 2
  - if excluded ingredient, add 0
  - no extras or no exclusions, add 1
~~~sql
WITH frequentIngredients AS (
  SELECT 
    c.record_id,
    t.topping_name,
    CASE
      -- if extra ingredient, add 2
      WHEN t.topping_id IN (
          SELECT extra_id 
          FROM extrasBreak e
          WHERE e.record_id = c.record_id) 
      THEN 2
      -- if excluded ingredient, add 0
      WHEN t.topping_id IN (
          SELECT exclusions_id 
          FROM exclusionsBreak e 
          WHERE c.record_id = e.record_id)
      THEN 0
      -- no extras, no exclusions, add 1
      ELSE 1
    END AS times_used
  FROM pizza_runner.customer_orders c
  JOIN toppingsBreak t
    ON t.pizza_id = c.pizza_id
  JOIN pizza_runner.pizza_names p
    ON p.pizza_id = c.pizza_id
)

SELECT 
  topping_name,
  SUM(times_used) AS times_used 
FROM frequentIngredients
GROUP BY topping_name
ORDER BY times_used DESC;
~~~
  topping_name  | times_used
:--------------:|:--------------:
  Mushrooms     | 13
  Bacon         | 13
  Chicken       | 11
  Cheese        | 11
  Pepperoni     | 10
  Salami        | 10
  Beef          | 10
  BBQ Sauce     | 9 
  Tomatoes      | 4
  Onions        | 4
  Peppers       | 4
  Tomato Sauce  | 4
  
  # Insights
- The most used ingredients are Mushrooms and Bacon
- The most common extra is Bacon
- The most common exclusion is Cheese
