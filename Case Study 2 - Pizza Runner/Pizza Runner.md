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
  	order_time
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
        ELSE duration::integer END
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
    CASE EXTRACT(DOW FROM c.order_time)
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
    day_of_week, EXTRACT(DOW FROM c.order_time)
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
## B. Runner and Customer Experience
### 1. How many runners signed up for each 1 week period? (i.e. week starts 2021-01-01)

````sql
SELECT COUNT(runner_id) AS runner_signup,
       (DATE_PART('day', AGE(registration_date, DATE '2021-01-01')) / 7)::int AS signup_week
FROM pizza_runner.runners
GROUP BY signup_week
ORDER BY signup_week
````
#### Solution:
| runner_signup | signup_week| 
| ----------- | -----------  |
| 2        |  1  |
| 1        |   2  |
| 1         |   3  |

### 2. What was the average time in minutes it took for each runner to arrive at the Pizza Runner HQ to pickup the order?

````sql
WITH avg_pickup_time AS (
  SELECT 
    c.order_id,
    r.runner_id,
    EXTRACT(EPOCH FROM (r.pickup_time - c.order_time)) / 60 AS pickup_minutes
  FROM customer_orders_temp c
  JOIN runner_orders_temp r 
    ON c.order_id = r.order_id
  WHERE r.cancellation = ' '
)
SELECT runner_id,
  		AVG(pickup_minutes) AS avg_pickup_mins
FROM avg_pickup_time
WHERE pickup_minutes > 1
GROUP BY runner_id
````
#### Solution:
| runner_id | avg_pickup_mins| 
| ----------- | -----------  |
| 1        |  15.6777777777777778  |
| 2        |   23.7200000000000000  |
| 3         |   10.4666666666666667  |

### 3. Is there any relationship between the number of pizzas and how long the order takes to prepare?

````sql
WITH prep_time AS(
	SELECT 	c.order_id,
			COUNT(c.pizza_id) AS pizza_numbers,
			c.order_time,
			r.pickup_time,
			EXTRACT(EPOCH FROM (r.pickup_time - c.order_time))/60 AS prep_time_mins
	FROM customer_orders_temp c
	JOIN runner_orders_temp r ON c.order_id = r.order_id
	WHERE r.cancellation = ' '
	GROUP BY c.order_id,c.order_time,r.pickup_time
)
SELECT pizza_numbers,
		AVG(prep_time_mins) AS avg_prep_time
FROM prep_time
GROUP BY pizza_numbers
````
#### Solution:
| pizza_numbers | avg_prep_time| 
| ----------- | -----------  |
| 1        |  12.3566666666666667 |
| 2        |  18.3750000000000000 |
| 3         |   29.2833333333333333  |

### 4. What was the average distance travelled for each customer?

````sql
SELECT  c.customer_id,
		AVG(r.distance) AS avg_distance
FROM customer_orders_temp c
JOIN runner_orders_temp r ON c.order_id = r.order_id
WHERE r.cancellation = ' '
GROUP BY c.customer_id
````
#### Solution:
| customer_id | avg_distance| 
| ----------- | -----------  |
| 101        |  20  |
| 102        |   16.7333  |
| 103         |   23.39  |
|  104      |   10  |
|  105      |   25  |

### 5. What was the difference between the longest and shortest delivery times for all orders?

````sql
SELECT MAX(r.duration) - MIN(r.duration) AS difference
FROM runner_orders_temp r
WHERE duration IS NOT NULL
````
#### Solution:
| difference | 
| ----------- | 
| 30        | 

### 6. What was the average speed for each runner for each delivery and do you notice any trend for these values?

````sql
SELECT 	r.order_id,
	r.runner_id,
	AVG(r.distance/r.duration * 60) AS avg_speed
FROM runner_orders_temp r
WHERE r.cancellation = ' '
GROUP BY r.runner_id, r.order_id
````
#### Solution:
| order_id | runner_id | avg_speed |
| ----------- | -----------  |----  |
|1|	1|	37.5|
|2|	1|	44.44444444444444|
|3|	1|	40.2|
|10|	1|	60|
|4|	2|	35.099999999999994|
|7|	2|	60|
|8|	2|	93.6|
|5 |	3|	40|

### 7. What is the successful delivery percentage for each runner?

````sql
SELECT 
  runner_id, 
  ROUND(100 * SUM(
    CASE WHEN distance = 0 THEN 0
    ELSE 1 END) / COUNT(*), 0) AS success_perc
FROM #runner_orders
GROUP BY runner_id
````
copy-pasted!

## C. Ingredient Optimisation

### 1. What are the standard ingredients for each pizza?

````sql
SELECT c.pizza_id,
		pizza_runner.pizza_names.pizza_name,
		pizza_runner.pizza_recipes.toppings
FROM customer_orders_temp c
JOIN pizza_runner.pizza_names ON c.pizza_id = pizza_names.pizza_id
JOIN pizza_runner.pizza_recipes ON c.pizza_id = pizza_recipes.pizza_id
GROUP BY c.pizza_id,
		pizza_runner.pizza_names.pizza_name,
		pizza_runner.pizza_recipes.toppings
````
#### Solution:
| pizza_id | pizza_name | toppings |
| ----------- | -----------  |----  |
|2|	Vegetarian|	4, 6, 7, 9, 11, 12|
|1|	Meatlovers|	1, 2, 3, 4, 5, 6, 8, 10|

### 2. What was the most commonly added extra?

```sql
WITH extras_cte AS(
	SELECT  pizza_id,
			NULLIF(TRIM(REGEXP_SPLIT_TO_TABLE(extras, ',')), '') AS extras_id
	FROM customer_orders_temp 
)
SELECT  ex.extras_id:: INTEGER,
		pizza_runner.pizza_toppings.topping_name AS extra_topping,
		COUNT(ex.extras_id::INTEGER) AS occurence_count
FROM extras_cte ex
JOIN pizza_runner.pizza_toppings ON ex.extras_id:: INTEGER = pizza_toppings.topping_id
GROUP BY ex.extras_id:: INTEGER, extra_topping
ORDER BY occurence_count DESC
````
#### Solution:
| extras_id | extra_topping| occurence_count |
| ----------- | -----------  |----  |
|1|	Bacon|		4|
|4|	Cheese|		1|
|5|	Chicken|	1|

### 3. What was the most common exclusion?

````sql
WITH exclusions_cte AS(
	SELECT  pizza_id,
			NULLIF(TRIM(REGEXP_SPLIT_TO_TABLE(exclusions, ',')), '') AS exclusions_id
	FROM customer_orders_temp 
)
SELECT  exc.exclusions_id:: INTEGER,
		pizza_runner.pizza_toppings.topping_name AS excluded_topping,
		COUNT(exc.exclusions_id::INTEGER) AS occurence_count
FROM exclusions_cte exc
JOIN pizza_runner.pizza_toppings ON exc.exclusions_id::INTEGER = pizza_toppings.topping_id
GROUP BY exc.exclusions_id::INTEGER, excluded_topping
ORDER BY occurence_count DESC
````
#### Solution:
| exclusions_id | excluded_topping| occurence_count |
| ----------- | -----------  |----  |
|4|	Cheese|			4|
|6|	Mushrooms|		1|
|2|	BBQ Sauce|		1|

### 4. Generate an order item for each record in the customers_orders table in the format of one of the following:
- Meat Lovers
- Meat Lovers - Exclude Beef
- Meat Lovers - Extra Bacon
- Meat Lovers - Exclude Cheese, Bacon - Extra Mushroom, Peppers

## D. Pricing and Ratings

### 1. If a Meat Lovers pizza costs $12 and Vegetarian costs $10 and there were no charges for changes — how much money has Pizza Runner made so far if there are no delivery fees?

````sql
SELECT SUM(
	CASE
	WhEN c.pizza_id = 1 THEN 12
	ELSE 10
	END) AS total_cost
FROM customer_orders_temp c
JOIN pizza_runner.runner_orders r ON c.order_id = r.order_id
WHERE r.cancellation = ''
````

*somehow solution changes everytime I log-in to my session...quite annoying!!!

### 2. What if there was an additional $1 charge for any pizza extras? 

````sql
SELECT 
    SUM(
        CASE 
            WHEN c.pizza_id = 1 THEN 12
            ELSE 10 
        END +
        COALESCE(array_length(string_to_array(NULLIF(c.extras, ''), ','), 1), 0)  
    ) AS total_amount
FROM customer_orders_temp c
JOIN runner_orders_temp r ON c.order_id = r.order_id
WHERE r.cancellation = ' ';
````
### 3. The Pizza Runner team now wants to add an additional ratings system that allows customers to rate their runner, how would you design an additional table for this new dataset — generate a schema for this new table and insert your own data for ratings for each successful customer order between 1 to 5.

````sql
DROP TABLE IF EXISTS pizza_runner.ratings;

CREATE TABLE pizza_runner.ratings (
    order_id INTEGER,
    ratings INTEGER
);

INSERT INTO pizza_runner.ratings (order_id, ratings)
VALUES
    (1, 3),
    (2, 5),
    (3, 3),
    (4, 1),
    (5, 5),
    (7, 3),
    (8, 4),
    (10, 3);
````

### 4. Using your newly generated table — can you join all of the information together to form a table which has the following information for successful deliveries? - customer_id, order_id,runner_id,rating, order_time, pickup_time, Time between order and pickup,vDelivery duration, Average speed, Total number of pizzas
````sql
/* Using your newly generated table — 
can you join all of the information together to form a table which 
has the following information for successful deliveries? 
- customer_id, order_id,runner_id,rating, order_time, 
pickup_time, Time between order and pickup,
Delivery duration, Average speed, Total number of pizzas */

SELECT  c.customer_id,
		c.order_id, 
		r.runner_id,
		ra.ratings,
		c.order_time,
		r.pickup_time,
		EXTRACT(EPOCH FROM (r.pickup_time - c.order_time)) / 60 AS time_between_order_and_pickup,
		r.duration,
		AVG(r.distance/r.duration * 60) AS avg_speed,
		COUNT(c.pizza_id) AS pizza_count
FROM customer_orders_temp  c
JOIN runner_orders_temp r ON c.order_id = r.order_id
JOIN pizza_runner.ratings ra ON c.order_id = ra.order_id
GROUP BY  c.customer_id, c.order_id, r.runner_id, ra.ratings, c.order_time, r.pickup_time, r.duration				
````
### 5. If a Meat Lovers pizza was $12 and Vegetarian $10 fixed prices with no cost for extras and each runner is paid $0.30 per kilometre traveled — how much money does Pizza Runner have left over after these deliveries?

````sql
SELECT 
    SUM(
        CASE 
            WHEN c.pizza_id = 1 THEN 12
            ELSE 10 
        END) - SUM(r.distance)*0.30 AS total_amount
FROM customer_orders_temp c
JOIN runner_orders_temp r ON c.order_id = r.order_id
WHERE r.cancellation = ' '
````

***
### Learnings
##### 1. ERD Diagram within Postgre SQL itself
##### 2. Temporary tables
	CREATE TEMP TABLE AS
##### 3. TRIM in CASE Statements
##### 4. EPOCH to calculate time difference
	*EXTRACT(EPOCH FROM timestamp): This function in PostgreSQL calculates the number of seconds that have elapsed since the epoch for the given timestamp
	For example:
	- EXTRACT(EPOCH FROM TIMESTAMP '2023-01-01 00:00:00') returns 1672531200, the number of seconds from 1970-01-01 to 2023-01-01.
	- Later we use  "/ 60" which converts the seconds into minutes to make it more human-readable and relevant to the analysis.*

##### 5. REGEXP_SPLIT_TO_TABLE
	REGEXP_SPLIT_TO_TABLE(exclusions, ',')
 	It splits the exclusions string into multiple rows using a specified delimiter (in this case, a comma ,).
	Each part becomes a separate row.
  	Example: For exclusions = '1,2,3', it generates
	|order_id    |exclusion|
	|1           |1|
	|1           |2|
	|1           |3|

##### 6. COALESCE
````sql
COALESCE(
    CASE 
        WHEN c.extras IS NOT NULL AND c.extras != '' THEN 
            array_length(string_to_array(c.extras, ','), 1)
        ELSE 0 
    END, 0)
````
	- `string_to_array(c.extras, ',')` - This function splits the string in c.extras by commas and returns an array.
		Example:
		If c.extras = '1,4,6', then string_to_array('1,4,6', ',') will return the array ['1', '4', '6']
  	- `array_length(string_to_array(c.extras, ','), 1)` - This function counts the number of elements (items) in the array produced by string_to_array.
The second parameter 1 refers to the dimension of the array we are interested in (since string_to_array produces a 1D array, we use 1 to count the length of that array).
		Example:
		If c.extras = '1,4,6', then array_length(['1', '4', '6'], 1) will return 3, since there are three items in the array.
  	- `COALESCE(..., 0)` - The COALESCE function returns the first non-NULL value in the list of arguments.
   		Example:
		If the CASE statement results in a number (e.g., 3 or 0), COALESCE just returns that number.
 		
