# Case Study 8 - Fresh Segments 🍊 
Danny created Fresh Segments, a digital marketing agency that helps other businesses analyse trends in online ad click behaviour for their unique customer base.

Clients share their customer lists with the Fresh Segments team who then aggregate interest metrics and generate a single dataset worth of metrics for further analysis.

In particular - the composition and rankings for different interests are provided for each client showing the proportion of their customer list who interacted with online assets related to each interest for each month.


<img src = "https://github.com/user-attachments/assets/62a44738-b601-4107-a53b-8c5478d425f3" alt = "Image" width = "500" height = "520">

# Task
Analyse aggregated metrics for an example client and provide some high level insights about the customer list and their interests.

# Table of Content
-  [Entity Relationship Diagram (ERD)](#entity-relationship-diagram)
-  [Case Study Questions with Solutions](#case-study-questions-with-solutions)
    -  [A. Data Exploration and Cleansing](#a-data-exploration-and-cleansing)
    -  [B. Interest Analysis](#b-interest-analysis)
    -  [C. Segment Analysis](#c-segment-analysis)
    -  [D. Index Analysis](#d-index-analysis)
  -  [Learnings](#learnings)

***

## Entity Relationship Diagram
Data can be extracted from [DB Fiddle](https://www.db-fiddle.com/f/iRdsT76vaus813crPP8Ma4/10)

-  `interest_metrics`
    -  Each record in this table represents the performance of a specific `interest_id` based on the client’s customer base interest measured through clicks and interactions with specific targeted advertising content.
    -  the `composition` metric means that "x%" of the client’s customer list interacted with the interest interest_id "y" - we can link  `interest_id` to a separate mapping table to find the segment name called “Vacation Rental Accommodation Researchers”
    -  The `index_value` is "z", means that the composition value is "Xtimes" the average composition value for all Fresh Segments clients’ customer for this particular interest in that particular month
    -  The `ranking` and `percentage_ranking` relates to the order of `index_value` records in each month year.
      <img src = "https://github.com/user-attachments/assets/aa80920e-e1d3-46c0-9db9-8ca081486bbe" alt = "Image" width = "550" height = "400">      
    
    
-  `interest_map`
    -  This mapping table links the `interest_id` with their relevant interest information. We will need to join this table onto the previous `interest_details` table to obtain the `interest_name` as well as any details about the summary information
      <img src = "https://github.com/user-attachments/assets/08b46fbc-0aad-4dc9-b6fc-35c8d6a66f10" alt = "Image" width = "550" height = "400">
***

## Case Study Questions with Solutions

### A. Data Exploration and Cleansing

#### 1. Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a date data type with the start of the month
_Approach taken_
-    Use of `ALTER` syntax to make changes to the column and table
-    Use `to_date` function to convert to DATE function and use 'MM-YYYY' format

````sql
ALTER TABLE fresh_segments.interest_metrics
ALTER COLUMN month_year TYPE DATE USING to_date(month_year, 'MM-YYYY')
````

#### 2. What is count of records in the `fresh_segments.interest_metrics` for each `month_year` value sorted in chronological order (earliest to latest) with the null values appearing first?
_Approach Taken_
-    Use of `NULLS FIRST` to add the null values to appear first and `ORDER BY` to order rest of the `month_year` in chronological order

````sql
SELECT 	month_year,
		COUNT(*) AS count
FROM fresh_segments.interest_metrics
GROUP BY month_year
ORDER BY month_year NULLS FIRST
````

![image](https://github.com/user-attachments/assets/3d02891d-3f5e-4ded-a668-eb480e2dcd24)

#### 3. What do you think we should do with these null values in the `fresh_segments.interest_metrics`
_Approach taken_
-   Ideally we should first check the amount or rather percentage of `NULL` values in `interest_id`, this will help us decide whether we should keep them or remove them from our data
-    Checking `NULL` value percentage in the table
````sql
SELECT 	
		ROUND(100 * (SUM(
						CASE WHEN interest_id IS NULL THEN 1 END) * 1.0 /
							COUNT(*)),2) AS null_percent
FROM fresh_segments.interest_metrics
````
![image](https://github.com/user-attachments/assets/830a9ec6-2995-4d92-91c6-41c80cba3432)

-    The `NULL` values are less than 10% (i.e. 8.36%) so we can remove the from the table by using `DELETE` command

````sql
DELETE FROM fresh_segments.interest_metrics
WHERE interest_id IS NULL
````

#### 4. How many `interest_id` values exist in the `fresh_segments.interest_metrics` table but not in the `fresh_segments.interest_map` table? What about the other way around?
_Approach taken_
-    Used `COUNT(DISTINCT) to count the `interest_id` and `id` in their respective tables
-    Used `CASE` statement to check for null values in appropriate columns
-    Used `FULL OUTER JOIN` to combine rows of both `interest_metrics` and `interest_map` table
-    converted `interest_id` column to Integer by using `::integer`

````sql
SELECT
    COUNT(DISTINCT interest_metrics.interest_id::integer) AS metrics_id,
    COUNT(DISTINCT interest_map.id) AS map_id,
    SUM(CASE WHEN interest_metrics.interest_id IS NULL THEN 1 END) AS not_in_metric,
    SUM(CASE WHEN interest_map.id IS NULL THEN 1 END) AS not_in_map
FROM fresh_segments.interest_metrics
FULL OUTER JOIN fresh_segments.interest_map ON
    interest_metrics.interest_id::integer = interest_map.id
````

![image](https://github.com/user-attachments/assets/bf61ce4f-8065-4d51-865d-70b0dac16d2d)
