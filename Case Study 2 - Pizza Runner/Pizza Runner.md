# Pizza Runner
Over 115 million kilograms of pizza are consumed daily worldwide, inspiring Danny to launch “Pizza Runner,” a retro-style pizza delivery business with an Uber-like model. 
To bring this vision to life, he recruited “runners” to deliver pizzas from his home, which he dubbed Pizza Runner Headquarters, and hired freelance developers to create a mobile app for customer orders.

<img src="https://user-images.githubusercontent.com/81607668/127271856-3c0d5b4a-baab-472c-9e24-3c1e3c3359b2.png" alt="Image" width="500" height="520">

# Case Study Questions with Solutions
The case study has been divided into different question categories that includes the following:
  - [Data Cleaning and Transformation](#-data-cleaning--transformation)
  - [A. Pizza Metrics](#a-pizza-metrics)
  - [B. Runner and Customer Experience](#b-runner-and-customer-experience)
  - [C. Ingredient Optimisation](#c-ingredient-optimisation)
  - [D. Pricing and Ratings](#d-pricing-and-ratings)
  - [E. Bonus Data Manipulation Challenge] (#e-bonus-data-manipulation-challenge)
  - [Learnings](Learnings)

# Task
Our task will be to clean this data and apply some basic calculations so Dany can better direct his runners and optimise Pizza Runner’s operations.

# ERD Diagram
![Pizza Runner](https://github.com/katiehuangx/8-Week-SQL-Challenge/assets/81607668/78099a4e-4d0e-421f-a560-b72e4321f530)

## ERD Diagram directly in Postgres
![Pizza Runner ERD](https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/1a547c32e85ca4044aefe22e3f7afde0c5d9f605/ref_images/Pizza_Runner_ERD.pgerd.png)

## Data Cleaning and Transformation
### 1. Table: customer_orders
<img width="1063" alt="image" src="https://user-images.githubusercontent.com/81607668/129472388-86e60221-7107-4751-983f-4ab9d9ce75f0.png">

In the table `customer_orders`, we  have `null` or `missing` values in `exclusions` and `extras` column. So we need to  remove these null values and replace them
with `blank` space. This will be done by:
- Creating a temporary table - `customer_orders_temp`
- Removing null values and replace it with blank space ''

````sql
  CREATE TEMP TABLE customer_orders_temp AS
  SELECT 
    order_id, 
    customer_id, 
    pizza_id, 
    CASE
  	  WHEN exclusions IS null OR exclusions LIKE 'null' THEN ' '
  	  ELSE exclusions
  	  END AS exclusions,
    CASE
  	  WHEN extras IS NULL or extras LIKE 'null' THEN ' '
  	  ELSE extras
  	  END AS extras,
  	order_date
  FROM pizza_runner.customer_orders
````



***

### 2. Table: runner_orders
This table requires more cleaning work then the previous table:
- create a temporary table - `runner_orders_temp`
- remove `null`/blank spaces from different columns
- from column `distance` remove ' km' or 'km'
- from column `duration` remove 'mins' or 'minutes'

<img width="1037" alt="image" src="https://user-images.githubusercontent.com/81607668/129472585-badae450-52d2-442e-9d50-e4d0d8fce83a.png">

````sql
CREATE TEMP TABLE runner_orders_temp AS 
SELECT order_id,
		runner_id,
		CASE WHEN pickup_time IS NULL OR pickup_time LIKE 'null' THEN ' '
			 ELSE pickup_time
			 END AS pickup_time,
		CASE WHEN distance IS NULL OR distance LIKE 'null' THEN ' '
			 WHEN distance LIKE '%km' THEN TRIM('km' FROM distance)
			 ELSE distance
			 END AS distance,
		CASE WHEN duration IS NULL OR duration LIKE 'null' THEN ' '
			 WHEN duration LIKE '%mins' THEN TRIM('mins' FROM duration)
			 WHEN duration LIKE '%minute' THEN TRIM('minute' FROM duration)
			 WHEN duration LIKE '%minutes' THEN TRIM('minutes' FROM duration)
			 ELSE duration
			 END AS duration,
		CASE WHEN cancellation IS NULL OR cancellation = '' OR cancellation LIKE 'null' THEN ' '
			 ELSE cancellation
			 END AS cancellation
FROM pizza_runner.runner_orders
````
Correct the data type in running_orders_temp table - unable to get the correct output so I updated the datatypes from within the PGAdmin* iteself to:
````sql
ALTER TABLE runner_orders_temp
    ALTER COLUMN pickup_time TYPE TIMESTAMP USING 
        CASE WHEN pickup_time = ' ' THEN NULL 
        ELSE pickup_time::timestamp END,
    ALTER COLUMN distance TYPE FLOAT USING 
        CASE WHEN distance = ' ' THEN NULL 
        ELSE distance::float END,
    ALTER COLUMN duration TYPE INTEGER USING 
        CASE WHEN duration = ' ' THEN NULL 
        ELSE duration::integer END;
````

*took Claud's help for this

***

## Questions with Solutions
## A. Pizza Metrics

### 1. How many pizzas were ordered?
SELECT COUNT(*) AS total_pizzas
FROM customer_orders_temp

| pizzas |
 -----------  |
| 14        | 

### 2. How many unique customer orders were made?
SELECT COUNT(DISTINCT(order_id))
FROM customer_orders_temp

#### Solution:
| successful_orders |
 -----------  |
| 10         | 


### 3. How many successful orders were delivered by each runner?
SELECT runner_id,
		COUNT(*) AS successful_orders
FROM runner_orders_temp 
WHERE cancellation = ' '
GROUP BY runner_orders_temp.runner_id

#### Solution:
| runner_id | successful_orders |
| ----------- | -----------  |
| 1          | 4           |
| 2          | 3          |
| 3          | 1           |

### 4. How many of each type of pizza was delivered?

````sql
SELECT 	pizza_names.pizza_id,
		COUNT(customer_orders_temp.order_id) AS total_pizzas
FROM pizza_runner.pizza_names
JOIN customer_orders_temp ON pizza_names.pizza_id = customer_orders_temp.pizza_id
JOIN runner_orders_temp ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE cancellation = ' '
GROUP BY pizza_names.pizza_id
````

#### Solution:
| pizza_id | total_pizzas |
| ----------- | -----------  |
| 1          | 9          |
| 2          | 3          |

### 5. How many Vegetarian and Meatlovers were ordered by each customer?**

````sql
SELECT 	pizza_names.pizza_name,
		customer_orders_temp.customer_id AS customers,
		COUNT(customer_orders_temp.order_id) AS total_pizzas
FROM pizza_runner.pizza_names
JOIN customer_orders_temp ON pizza_names.pizza_id = customer_orders_temp.pizza_id
JOIN runner_orders_temp ON customer_orders_temp.order_id = runner_orders_temp.order_id
WHERE cancellation = ' '
GROUP BY pizza_names.pizza_name,customers
ORDER  BY total_pizzas DESC
````
#### Solution:
| pizza_name | customers | total_pizzas |
| ----------- | -----------  |----  |
| Meatlovers  | 104         |  3   |
| Meatlovers  | 101         |   2  |
| Meatlovers  | 103         |   2  |
| Meatlovers  | 102         |   2  |
| Vegetarian  | 103         |   1  |
| Vegetarian  | 102         |    1 |
| Vegetarian  | 105         |    1 |

### 6. What was the maximum number of pizzas delivered in a single order?

````sql
SELECT 	customer_orders_temp.order_id AS order_id,
		COUNT(pizza_names.pizza_id) AS pizza_delivered
FROM customer_orders_temp
JOIN pizza_runner.pizza_names ON customer_orders_temp.pizza_id = pizza_names.pizza_id 
GROUP BY orders
ORDER  BY pizza_delivered DESC
````
#### Solution:
| orders | pizza_delivered | 
| ----------- | -----------  |
| 4         |  3   |
| 10         |   2  |
|  3         |   2  |
|  2         |   1  |
|  7         |   1  |
| 1         |    1 |
|  9         |    1 |
|  8         |    1 |
|  5       |    1 |
|  6        |    1 |

### 7. For each customer, how many delivered pizzas had at least 1 change and how many had no changes?

````sql
SELECT	c.customer_id AS customer_id,
		SUM(CASE WHEN c.exclusions <> ' ' OR c.extras <> ' ' THEN 1
		ELSE 0
		END) AS change,
		SUM(CASE WHEN c.exclusions = ' ' AND c.extras = ' ' THEN 1
		ELSE 0
		END) AS no_change
FROM customer_orders_temp c
JOIN runner_orders_temp r ON c.order_id = r.order_id
WHERE r.cancellation = ' '
GROUP BY customer_id
ORDER  BY customer_id 
		
````
#### Solution:
| change | customer_id | no change |
| ----------- | -----------  |----  |
| 2  | 101         |  0   |
| 2  | 102         |   1  |
| 2  | 103         |   0  |
| 3  | 104         |   0  |
| 2  | 105         |   1  |

### 8.How many pizzas were delivered that had both exclusions and extras?

````sql
SELECT  
  COUNT(*) AS pizza_count
FROM customer_orders_temp AS c
JOIN runner_orders_temp AS r
  ON c.order_id = r.order_id
WHERE r.distance >= 1 
  AND exclusions <> ' ' 
  AND extras <> ' ';
````
| pizza_count|
| ----------- |
| 7  |

This is an incorrect answer!!

### 9. What was the total volume of pizzas ordered for each hour of the day?

````sql
SELECT EXTRACT(HOUR FROM c.order_date) AS hour,
		COUNT(pizza_id) AS ordered_pizzas
FROM customer_orders_temp c
GROUP BY hour
ORDER BY hour
````
#### Solution:
| hour | ordered_pizzas| 
| ----------- | -----------  |
| 11        |  1  |
| 13         |   3  |
|  18         |   3  |
|  19         |   1  |
|  21         |  3  |
| 23         |    3 |

### 10. What was the volume of orders for each day of the week?

````sql
SELECT 
    CASE EXTRACT(DOW FROM c.order_date)
        WHEN 0 THEN 'Sunday'
        WHEN 1 THEN 'Monday'
        WHEN 2 THEN 'Tuesday'
        WHEN 3 THEN 'Wednesday'
        WHEN 4 THEN 'Thursday'
        WHEN 5 THEN 'Friday'
        WHEN 6 THEN 'Saturday'
    END AS day_of_week,
    COUNT(c.pizza_id) AS ordered_pizzas
FROM 
    customer_orders_temp c
GROUP BY 
    day_of_week, EXTRACT(DOW FROM c.order_date)
ORDER BY 
    EXTRACT(DOW FROM c.order_date)
````
#### Solution:
| day_of_week | ordered_pizzas| 
| ----------- | -----------  |
| Wednesday        |  5  |
| Thursday        |   3  |
| Friday         |   1  |
|  Saturday      |   5  |


***
### Learnings
##### 1. Creating ERD Diagram in Postgre SQL itself
##### 2. Creating temporary tables
##### 3. Use of TRIM in CASE Statements
