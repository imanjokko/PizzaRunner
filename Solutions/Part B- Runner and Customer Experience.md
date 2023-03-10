# Questions
1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
3. Is there any relationship between the number of pizzas and how long the order takes to prepare?
4. What was the average distance travelled for each customer?
5. What was the difference between the longest and shortest delivery times for all orders?
6. What was the average speed for each runner for each delivery and do you notice any trend for these values?
7. What is the successful delivery percentage for each runner?

# Solutions
1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
~~~sql
SELECT date_part('week', registration_date +INTERVAL '3 days') AS week_start, 
	COUNT(*) AS num_signups
FROM pizza_runner.runners
GROUP BY week_start
ORDER BY week_start;
--I added the "INTERVAL '3 days'" to ensure that signups within a 3-day period are included in the same week
--This is because postgres's week starts on Mondays, which is the 4th and would've excluded sign ups from earlier
~~~
 week_start   | num_signups
:------------:|:--------------:
 1            | 2
 2            | 1
 3            | 1
 ---
2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
~~~sql
SELECT runner_id, avg(time_to_arrival) as avg_arrival_time
FROM 
	(SELECT r.runner_id, (r.pickup_time - c.order_time) AS time_to_arrival
FROM pizza_runner.runner_orders r
INNER JOIN pizza_runner.customer_orders c
ON r.order_id = c.order_id) as arrival_time
GROUP BY runner_id
~~~
 runner_id    | avg_arrival_time
:------------:|:-------------------:
 3            | 00:10:28
 2            | 00:23:43
 1            | 00:15:41
 ---
 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

   --viewing prep time per order
 ~~~sql
SELECT c.order_id, count (*) as num_pizzas, 
	(r.pickup_time - c.order_time) AS time_to_prepare
FROM pizza_runner.runner_orders r
INNER JOIN pizza_runner.customer_orders c
	ON r.order_id = c.order_id
WHERE r.pickup_time IS NOT NULL
GROUP BY c.order_id, r.pickup_time, c.order_time
~~~
 order_id  | num_pizzas  | time_to_prepare
:---------:|:-----------:|:-----------------:
 1         | 1           | 00:10:32
 2         | 1           | 00:10:02
 7         | 1           | 00:10:16
 4         | 3           | 00:29:17
 3         | 2           | 00:21:14
 10        | 2           | 00:15:31
 5         | 1           | 00:10:28
 8         | 1           | 00:20:29
 
 --looking at the average prep time by amount of pizzas in the order
~~~sql
SELECT num_pizzas, AVG(time_to_prepare) AS avg_prep_time
FROM 
	(SELECT c.order_id, count (*) AS num_pizzas, 
	(r.pickup_time - c.order_time) AS time_to_prepare
FROM pizza_runner.runner_orders r
INNER JOIN pizza_runner.customer_orders c
	ON r.order_id = c.order_id
WHERE r.pickup_time IS NOT NULL
GROUP BY c.order_id, r.pickup_time, c.order_time) AS prep_time
GROUP BY num_pizzas
~~~
 num_pizzas | avg_prep_time
:----------:|:--------------:
  3         | 00:29:17
  2         | 00:18:22
  1         | 00:12:21
 
- The answer is yes; ON AVERAGE, there appears to be a positive relationship (correlation) between the pizza numbers and delivery time.
  -  **However, this is not to be mistaken for causation**
---
4. What was the average distance travelled for each customer?
~~~sql
SELECT c.customer_id, CEIL(AVG(r.distance_km)) as avg_distance_km
FROM pizza_runner.customer_orders c
INNER JOIN pizza_runner.runner_orders r
	ON c.order_id = r.order_id
WHERE r.cancellation IS NULL
GROUP BY c.customer_id
~~~
 customer_id  | avg_distance_km
:------------:|:----------------:
  101         | 20
  102         | 17
  103         | 24
  104         | 10
  105         | 25
---
5. What was the difference between the longest and shortest delivery times for all orders?
~~~sql
SELECT (max(duration_mins)) - (min(duration_mins)) as duration_diff
FROM pizza_runner.runner_orders
~~~
 duration_diff
:--------------:
  30
---
6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

--converting distance and duration to m/s for speed calculation
~~~sql
ALTER TABLE pizza_runner.runner_orders
ADD COLUMN duration_secs INT

UPDATE pizza_runner.runner_orders
SET duration_secs = (duration_mins * 60)

ALTER TABLE pizza_runner.runner_orders
ADD COLUMN distance_m FLOAT

UPDATE pizza_runner.runner_orders
SET distance_m = (distance_km * 1000)
~~~
  Old table                        | Updated table
 :--------------------------------:|:-------------------------------------:
  ![](https://github.com/imanjokko/PizzaRunner/blob/main/images/runnerordersaltered.png) | ![](https://github.com/imanjokko/PizzaRunner/blob/main/images/runnerordersupdated.png)

--speed for each delivery
~~~sql
SELECT runner_id, order_id, CEIL((distance_m/duration_secs)) AS delivery_speed
FROM pizza_runner.runner_orders
WHERE cancellation IS NULL
~~~
 runner_id | order_id | delivery_speed
:---------:|:--------:|:---------------:
  1        | 1        | 11
  1        | 2        | 13
  1        | 3        | 12
  2        | 4        | 10
  3        | 5        | 12
  2        | 7        | 17
  2        | 8        | 26
  1        | 10       | 17
  
--calculating avg speed per runner
~~~sql
SELECT runner_id, CEIL(AVG(distance_m/duration_secs)) AS avg_runner_speed
FROM pizza_runner.runner_orders
GROUP BY runner_id
~~~

 runner_id | avg_runner_speed
:---------:|:-----------------:
  3        | 12
  2        | 18
  1        | 13
  
 - I did not notice any trends here

---
7. What is the successful delivery percentage for each runner?
~~~sql
SELECT runner_id, 
  ROUND(100 * SUM(CASE WHEN cancellation IS NULL THEN 1 ELSE 0 END) 
		/ COUNT(*)) AS success_perc
FROM pizza_runner.runner_orders
GROUP BY runner_id;
~~~
 runner_id  | success_perc
:----------:|:--------------:
  3         | 50
  2         | 75
  1         | 100
  
---
  # Insights
  
  - Runner 1 has the highest percentage of succesful deliveries
  - Runner 2 is the fastest rider
  - Customer 105 is the one riders have travelled the most distance for
  - Week 1 had the most runner signups
  - The last rider to sign up; runner 4, has not made any deliveries yet
