# Case Study 5 - Data Mart🛒
Data Mart is an online supermarket that specialises in fresh produce. In June 2020 - large scale supply changes were made at Data Mart. All Data Mart products now use sustainable packaging methods in every single step from the farm all the way to the customer.

The key business question to be answered are the following:
-  What was the quantifiable impact of the changes introduced in June 2020?
-  Which platform, region, segment and customer types were the most impacted by this change?
-  What can we do about future introduction of similar sustainability updates to the business to minimise impact on sales?

<img src = "https://github.com/user-attachments/assets/9f4f3f81-9192-4a07-b8a4-bcd64ba6b4d8" alt = "Image" width = "500" height = "520">

# Task
For this task we will have to analyse the sales performance and also quantify the impact of sustainable packaging on the sales performance for Data Mart and it’s separate business areas.

# Table of content
The case study has been divided into the following parts:
-  [ERD Diagram](#erd-diagram)
-	[Case Study Questions with Solutions](#case-study-questions-with-solutions)
	-	[A. Data Cleansing Steps](#a-data-cleansing-steps)
   	- [B. Data Exploration](#b-data-exploration)
    -  [C. Before & After Analysis](#c-before-&-after-analysis)
    - [D. Bonus Question](#d-bonus-question)
-  [Learnings](#learnings)
***
# ERD Diagram
![image](https://github.com/user-attachments/assets/16f07e30-8a80-48c0-9209-1055776b6488)

For this case study there is only one table `data_mart.weekly_sales`. The columns in the table are defined as follows:
-  Data Mart has international operations using a multi-`region` strategy
-  Data Mart has both, a retail and online `platform` in the form of a Shopify store front to serve their customers
-  Customer `segment` and `customer_type` data relates to personal age and demographics information that is shared with Data Mart
-  `transactions` is the count of unique purchases made through Data Mart and `sales` is the actual dollar amount of purchases
Each record in the dataset is related to a specific aggregated slice of the underlying sales data rolled up into a `week_date` value which represents the start of the sales week.
### Example Table `data_mart.weekly_sales`
<img src = "https://github.com/user-attachments/assets/e4d3fec6-c6e2-4641-b4ad-cc63a5ff6f58" alt = "Image" width = "700" height = "450">

***

# Case Study Questions with Solutions

## A. Data Cleansing Steps
In a single query, perform the following operations and generate a new table in the `data_mart` schema named `clean_weekly_sales`:
-  Convert the `week_date` to a `DATE` format
-  Add a `week_numbe`r as the second column for each `week_date` value, for example any value from the 1st of January to 7th of January will be 1, 8th to 14th will be 2 etc
-  Add a `month_number` with the calendar month for each `week_date` value as the 3rd column
-  Add a `calendar_year` 
olumn as the 4th column containing either 2018, 2019 or 2020 values
-  Add a new column called `age_band` after the original `segment` column using the following mapping on the number inside the segment value
<img src = "https://github.com/user-attachments/assets/95aaef7f-d059-4739-a131-00284f03b30c" alt = "Image" width = "280" height = "220">

-  Add a new `demographic` column using the following mapping for the first letter in the `segment`` values:
<img src = "https://github.com/user-attachments/assets/38c7377d-d5f7-45d0-9e38-c9435dd1c6de" alt = "Image" width = "150" height = "150">

-  Ensure all `null` string values with an `unknown` string value in the original `segment` column as well as the new `age_band` and `demographic` columns
-  Generate a new `avg_transaction` column as the `sales` value divided by `transactions` rounded to 2 decimal places for each record

### New Table 'clean_weekly_sales`:
````sql
CREATE TABLE data_mart.clean_weekly_sales AS
	SELECT TO_DATE(week_date, 'DD/MM/YY') AS week_date,
			EXTRACT(WEEK FROM TO_DATE(week_date, 'DD/MM/YYYY')) AS week_number,
			EXTRACT(MONTH FROM TO_DATE(week_date, 'DD/MM/YYYY')) AS month_number,
			EXTRACT(YEAR FROM TO_DATE(week_date, 'DD/MM/YYYY')) AS calendar_year,
			region,
			platform,
			segment,
			CASE 
				WHEN RIGHT(segment,1) = '1' THEN 'Young Adults'
				WHEN RIGHT(segment,2) = '2' THEN 'Middle Aged'
				WHEN RIGHT(segment,1) IN ('3', '4') THEN 'Retirees'
				ELSE 'Unknown'
			END AS age_band,
			CASE 
				WHEN LEFT(segment,1) = 'C' THEN 'Couples'
				WHEN LEFT(segment,2) = 'F' THEN 'Families'
				ELSE 'Unknown'
			END AS demographic,
			transactions,
			sales,
			ROUND((sales::numeric/transactions),2) AS avg_transaction
FROM data_mart.weekly_sales
GROUP BY
    week_date,
    week_number,
    month_number,
    calendar_year,
    region,
    platform,
    segment,
    age_band,
    demographic,
    transactions,
    sales

````
## B. Data Exploration

### 1. What day of the week is used for each week_date value?
_Approach Taken_
-	Use `DISTINCT` and `TO_CHAR` functios to get the day of the week

````sql
SELECT DISTINCT(TO_CHAR(week_date, 'day')) AS week_day 
FROM clean_weekly_sales
````

### 2.What range of week numbers are missing from the dataset?
_Approach Taken_
-	CTE 'week_number_cte`, use `GENERATE_SERIES` function to generate `week_number` from 1 to 52
-	`LEFT JOIN` the CTE with `clean_weekly_sales` table to get the missing numbers

````sql
WITH week_number_cte AS (
		SELECT GENERATE_SERIES(1,52) AS week_number
		)
SELECT DISTINCT wn.week_number
FROM week_number_cte wn
LEFT JOIN clean_weekly_sales cws ON wn.week_number = cws.week_number
WHERE cws.week_number IS NULL
````
![image](https://github.com/user-attachments/assets/fda1e5b6-2c26-4756-86fc-0cfbdf8ca052)

### 3. How many total transactions were there for each year in the dataset?
_Approach Taken_
-	Use `SUM` to calculate the toal transactions and `GROUP BY` for grouping them with the years

````sql
SELECT calendar_year,
		SUM(transactions) AS total_transactions
FROM clean_weekly_sales
GROUP BY calendar_year
ORDER BY calendar_year
````
![image](https://github.com/user-attachments/assets/2f02bb78-5e7e-43d2-994c-cd731905349a)

### 4. What is the total sales for each region for each month?


***
# Learnings
-	`To_DATE` converts text onto a valid date object
-	'To_CHAR` formats the date for display purpose
	-	Example	`TO_CHAR(TO_DATE(week_date, 'DD/MM/YY'), 'DD/MM/YYYY') AS formatted_week_date`
