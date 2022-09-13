# Case Study 2 Pizza Runner

![image](https://github.com/Shailesh-python/Case_Study_2_Pizza_Runner/blob/main/Pizza%20Runner%20Challenge.png)

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
FROM pizza_runner.customer_orders
```
| total_orders |
|--------------|
|     14       |

## [Question #2](#case-study-questions)
> How many unique customer orders were made?
```sql
SELECT 
  COUNT(DISTINCT order_id) AS total_orders
FROM pizza_runner.customer_orders
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
GROUP BY o.runner_id
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
		ON CTE.pizza_id = PN.pizza_id
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
GROUP BY CO.customer_id
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
ORDER BY pizza_delivered DESC
```
| order_id  | pizza_delivered |
|-----------|-----------------|
| 4         |       3         |

## [Question #6](#case-study-questions)
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
GROUP BY CO.customer_id
```
| customer_id  | atleast_one_change | no_change   |
|--------------|--------------------|-------------|
|     101      |     0              |      2      |
|     102      |     0              |      3      |
|     103      |     3              |      0      |
|     104      |     1              |      2      |
|     105      |     0              |      1      |

## [Question #7](#case-study-questions)
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
