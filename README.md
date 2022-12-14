# Case Study #2 Pizza Runner

<img src='https://img.shields.io/badge/Microsoft%20SQL%20Server-CC2927?style=for-the-badge&logo=microsoft%20sql%20server&logoColor=white)'/>

![image](https://github.com/Shailesh-python/Case-Study-2-Pizza-Runner/blob/main/Pizza%20Runner%20Challenge.png)

The following are my solutions to the Case Study 2 Pizza Runner questions in 
[Danny Ma's Serious SQL course](https://www.datawithdanny.com/ "Data With Danny")
<br/>
<br/>
Danny has shared with you 6 key datasets for this case study :
[Data Set](https://github.com/Shailesh-python/Case_Study_2_Pizza_Runner/blob/main/Datasets%20and%20Tables)
<br/>
- `runners`
- `customer_orders`
- `runner_orders`
- `pizza_names`
- `pizza_recipes`
- `pizza_toppings`
<br/>

## A. Pizza Metrics

## [Question #1](#case-study-questions)
> How many pizzas were ordered?
```sql
SELECT 
	COUNT(order_id) AS total_orders
FROM pizza_runner.customer_orders;
```
| total_orders |
|--------------|
|     14       |

## [Question #2](#case-study-questions)
> How many unique customer orders were made?
```sql
SELECT 
  COUNT(DISTINCT order_id) AS total_orders
FROM pizza_runner.customer_orders;
```
| total_orders |
|--------------|
|     10       |

## [Question #3](#case-study-questions)
> How many successful orders were delivered by each runner??
```sql
SELECT 
	o.runner_id,
	SUM(CASE 
		WHEN O.cancellation IS NULL THEN 1
		WHEN O.cancellation = 'null' THEN 1 
		WHEN O.cancellation = '' THEN 1
		ELSE 0
	     END
	    ) AS Delivered 
FROM pizza_runner.runner_orders o
GROUP BY o.runner_id;
```
| runner_id | successful_orders |
|-----------|-------------------|
|    1	    |       4           |
|    2	    |       3           |
|    3	    |       1           |

## [Question #4](#case-study-questions)
> How many of each type of pizza was delivered?
```sql
WITH CTE AS
(
SELECT
	CO.pizza_id,
	SUM(CASE 
		WHEN RO.cancellation IS NULL THEN 1
		WHEN RO.cancellation = 'null' THEN 1 
		WHEN RO.cancellation = '' THEN 1
		ELSE 0
	    END
	       ) AS delivered_pizza_count 
FROM pizza_runner.runner_orders RO
LEFT JOIN pizza_runner.customer_orders CO
	ON RO.order_id = CO.order_id
GROUP BY CO.pizza_id
)	
	SELECT 
		pizza_name,
		delivered_pizza_count
	FROM CTE INNER JOIN pizza_runner.pizza_names PN 
		ON CTE.pizza_id = PN.pizza_id;
```
| pizza_name  | delivered_pizza_count |
|-----------  |-----------------------|
| Meatlovers  |       9               |
| Vegetarians |       3               |

## [Question #5](#case-study-questions)
> How many Vegetarian and Meatlovers were ordered by each customer?
```sql
SELECT 
	CO.customer_id,
	SUM(CASE WHEN PN.pizza_name LIKE 'Meatlovers' THEN 1 ELSE 0 END) Meatlovers,
	SUM(CASE WHEN PN.pizza_name LIKE 'Meatlovers' THEN 0 ELSE 1 END) Vegetarian
FROM pizza_runner.customer_orders CO
INNER JOIN pizza_runner.pizza_names PN
	ON CO.pizza_id=PN.pizza_id
GROUP BY CO.customer_id;
```
| customer_id  | Meatlovers | Vegetarians |
|--------------|------------|-------------|
|     101      |     2      |      1      |
|     102      |     2      |      1      |
|     103      |     3      |      1      |
|     104      |     3      |      0      |
|     105      |     0      |      1      |

## [Question #6](#case-study-questions)
> What was the maximum number of pizzas delivered in a single order??
```sql
SELECT 
	TOP 1
	co.order_id,
	COUNT(1) AS pizza_delivered
FROM pizza_runner.customer_orders co
INNER JOIN pizza_runner.runner_orders ro
	ON co.order_id = ro.order_id
WHERE ro.cancellation NOT IN ('Restaurant Cancellation','Customer Cancellation')
	OR ro.cancellation IS NULL
	OR ro.cancellation ='null'
GROUP BY co.order_id
ORDER BY pizza_delivered DESC;
```
| order_id  | pizza_delivered |
|-----------|-----------------|
| 4         |       3         |

## [Question #7](#case-study-questions)
> For each customer, how many delivered pizzas had at least 1 change and how many had no changes?
```sql
SELECT 
	CO.customer_id,
	SUM(CASE WHEN co.exclusions NOT IN ('null','') THEN 1 ELSE 0 END) AS atleast_one_change,
	SUM(CASE WHEN co.exclusions IN ('null','') THEN 1 ELSE 0 END) AS no_change
	FROM pizza_runner.customer_orders co
INNER JOIN pizza_runner.runner_orders ro
	ON co.order_id = ro.order_id
WHERE ro.cancellation NOT IN ('Restaurant Cancellation','Customer Cancellation')
	OR ro.cancellation IS NULL
	OR ro.cancellation ='null'
GROUP BY CO.customer_id;
```
| customer_id  | atleast_one_change | no_change   |
|--------------|--------------------|-------------|
|     101      |     0              |      2      |
|     102      |     0              |      3      |
|     103      |     3              |      0      |
|     104      |     1              |      2      |
|     105      |     0              |      1      |

## [Question #8](#case-study-questions)
> How many pizzas were delivered that had both exclusions and extras?
```sql
WITH CTE AS
(
SELECT 
	CO.order_id,
	(CASE WHEN CO.exclusions IS NULL OR CO.exclusions ='' OR CO.exclusions='NULL' THEN 0 ELSE 1 END) AS Has_exclusion,
	(CASE WHEN CO.extras IS NULL OR CO.extras ='' OR CO.extras='NULL' THEN 0 ELSE 1 END) AS Has_extras
FROM pizza_runner.customer_orders co
INNER JOIN pizza_runner.runner_orders ro
	ON co.order_id = ro.order_id
WHERE (ro.cancellation NOT IN ('Restaurant Cancellation','Customer Cancellation')
	OR ro.cancellation IS NULL
	OR ro.cancellation ='null')
)
	SELECT COUNT(DISTINCT CTE.order_id) AS both_extras_exclusion
	FROM CTE
	WHERE CTE.Has_exclusion=1 AND CTE.Has_extras = 1;
```
| both_extra_exclusion  | 
|-----------------------|
|     1                 |

## [Question #9](#case-study-questions)
> What was the total volume of pizzas ordered for each hour of the day?
```sql
SELECT
	DATEPART(HOUR,CO.order_time) as day_hour,
	COUNT(CO.order_id) AS orders_per_hour
FROM pizza_runner.customer_orders CO
GROUP BY DATEPART(HOUR,CO.order_time);
```
| day_hour    | orders_per_hour|
|-------------|----------------|
| 11          |       1        |
| 13          |       3        |
| 18          |       3        |
| 19          |       1        |
| 21          |       3        |
| 23          |       3        |

## [Question #10](#case-study-questions)
> What was the volume of orders for each day of the week?
```sql
-- add 2 to adjust 1st day of the week as Monday
SELECT 
	FORMAT(DATEADD(DAY, 2, co.order_time),'dddd') AS day_of_week,
	COUNT(co.order_id) AS total_pizzas_ordered
FROM pizza_runner.customer_orders co
GROUP BY FORMAT(DATEADD(DAY, 2, co.order_time),'dddd');
```
| week_day    | orders_per_day |
|-------------|----------------|
| Friday      |       5        |
| Monday      |       5        |
| Saturday    |       3        |
| Sunday      |       1        |

## Part B. Runner and Customer Experience

## [Question #1](#case-study-questions)
> How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)
```sql
SELECT 
	DATEPART(WEEK, DATEDIFF(DD,-4,registration_date))-1 AS registration_week,
	COUNT(runner_id) AS runner_signup
FROM pizza_runner.runners
GROUP BY DATEPART(WEEK, DATEDIFF(DD,-4,registration_date))-1;
```
| registration_week | runner_signup |
|-------------------|---------------|
| 1                 |       2       |
| 2                 |       1       |
| 3                 |       1       |

## [Question #2](#case-study-questions)
> What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?
```sql
WITH CTE AS
(
SELECT 
	DISTINCT
	CO.order_id,
	CO.order_time,
	RO.pickup_time,
	DATEDIFF(MINUTE, CO.order_time, RO.pickup_time) AS average_minutes
FROM pizza_runner.customer_orders CO
INNER JOIN pizza_runner.runner_orders RO
ON CO.order_id = RO.order_id
WHERE RO.pickup_time <> 'null'
)	
	SELECT ROUND(AVG(CTE.AVERAGE_MINUTES) * 1.0, 3) AS avg_pickup_minutes
	FROM CTE 
	WHERE CTE.AVERAGE_MINUTES > 0;
```
| avg_pickup_minutes |
|--------------------|
|     16.0           |

## [Question #3](#case-study-questions)
> Is there any relationship between the number of pizzas and how long the order takes to prepare?
```sql
SELECT 
	CO.order_id,
	DATEDIFF(MINUTE, CO.order_time, RO.pickup_time) AS diff_minutes,
	COUNT(CO.order_id) as pizza_count
FROM pizza_runner.customer_orders CO
INNER JOIN pizza_runner.runner_orders RO
ON CO.order_id = RO.order_id
WHERE RO.pickup_time <> 'null'
GROUP BY CO.order_id,DATEDIFF(MINUTE, CO.order_time, RO.pickup_time)
ORDER BY CO.order_id ASC;
```
| order_id | diff_minutes | pizza_count |
|----------|--------------|-------------|
|     1    |     10       |      1      |
|     2    |     10       |      1      |
|     3    |     21       |      2      |
|     4    |     30       |      3      |
|     5    |     10       |      1      |
|     7    |     10       |      1      |
|     8    |     21       |      1      |
|     10   |     16       |      2      |

## [Question #4](#case-study-questions)
> What was the average distance travelled for each customer?
```sql
SELECT 
	co.customer_id,
	AVG(
	    CONVERT(DECIMAL(5,3),TRIM(REPLACE(ro.distance,'km','')))
	   ) AS avg_distance
FROM pizza_runner.customer_orders CO
INNER JOIN pizza_runner.runner_orders RO
	ON CO.order_id = RO.order_id
WHERE ro.distance <> 'null'
GROUP BY CO.customer_id;
```
| customer_id | avg_distance |
|-------------|--------------|
| 101         |       20     |
| 102         |       16.73  |
| 103         |       23.4   |
| 104         |       10     |
| 105         |       25     |

## [Question #5](#case-study-questions)
> What was the difference between the longest and shortest delivery times for all orders?
```sql
----- I will create a custom function to extract numeric part-------
CREATE FUNCTION udf_GetNumeric(@input VARCHAR(256))
RETURNS VARCHAR(256)
AS
BEGIN
  DECLARE @intAlpha INT
  SET @intAlpha = PATINDEX('%[^0-9]%', @input)
  BEGIN
    WHILE @intAlpha > 0
    BEGIN
      SET @input = STUFF(@input, @intAlpha, 1, '' )
      SET @intAlpha = PATINDEX('%[^0-9]%', @input )
    END
  END
  RETURN ISNULL(@input,0)
END
---------------------------------------
## [Question #5](#case-study-questions)
> What was the difference between the longest and shortest delivery times for all orders?
```sql
SELECT
	CAST(MAX(dbo.udf_GetNumeric(ro.duration)) AS INT) 
	-  CAST(MIN(dbo.udf_GetNumeric(ro.duration)) AS INT) AS max_difference
FROM pizza_runner.customer_orders CO
INNER JOIN pizza_runner.runner_orders RO
	ON CO.order_id = RO.order_id
WHERE ro.distance <> 'null';
```
| max_difference |
|----------------|
| 30             |

## [Question #6](#case-study-questions)
> What was the difference between the longest and shortest delivery times for all orders?
```sql
WITH CTE AS
(
	SELECT 
		co.customer_id,
		AVG(CONVERT(DECIMAL(5,3),TRIM(REPLACE(ro.distance,'km','')))) AS total_distance,
		AVG(CAST(dbo.udf_GetNumeric(ro.duration) AS INT)/60.00) AS total_time
	FROM pizza_runner.customer_orders CO
	INNER JOIN pizza_runner.runner_orders RO
		ON CO.order_id = RO.order_id
	WHERE ro.distance <> 'null'
	GROUP BY CO.customer_id
)	
	SELECT 
		CTE.customer_id,
		CTE.total_distance,
		CTE.total_time,
		CTE.total_distance/CTE.total_time AS [avg_speed (kmh)]
	FROM CTE
```
![image](https://github.com/Shailesh-python/Case_Study_2_Pizza_Runner/blob/main/pizza_runner%20B6.jpg)

## [Question #7](#case-study-questions)
> What is the successful delivery percentage for each runner?
```sql
SELECT 
	t.runner_id,
	(100 * COUNT(T.is_distance))/COUNT(*) as delievery_percentage
FROM
(
SELECT 
	ro.runner_id,
	ro.distance,
	CASE WHEN ro.distance = 'null' THEN NULL ELSE 0 END AS is_distance
FROM pizza_runner.runner_orders ro
) T
group by t.runner_id
```
| runner_id | delievery_percentage|
|-----------|---------------------|
| 1         |       100           |
| 2         |       75            |
| 3         |       50            |

## C. Ingredient Optimisation

## [Question #1](#case-study-questions)
> What are the standard ingredients for each pizza?
```sql

```

## [Question #2](#case-study-questions)
> What was the most commonly added extra?
```sql
```
