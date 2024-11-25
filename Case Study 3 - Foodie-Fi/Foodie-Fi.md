# Case Study 3 - Foodie Fi
Subscription based businesses are super popular and Danny realised that there was a large gap in the market - he wanted to create a new streaming service that only had food related content.
Danny launches his new startup Foodie-Fi in 2020 and started selling monthly and annual subscriptions, giving their customers unlimited on-demand access to exclusive food videos from around the world!

Danny created Foodie-Fi with a data driven mindset and wanted to ensure all future investment decisions and new features were decided using data. This case study focuses on using subscription style digital data to answer important business questions.

<img src="https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/main/Case%20Study%203%20-%20Foodie-Fi/ref_images/3.%20Foodie-fi.png" alt="Image" width ="500" height="520">

# ERD Diagram
<img src = "https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/main/Case%20Study%203%20-%20Foodie-Fi/ref_images/case-study-3-erd.png">

There are two tables in the database - 
1. `plans` - There are 5 customer plans.
    - Trial — Customer sign up to an initial 7 day free trial and will automatically continue with the pro monthly subscription plan unless they cancel, downgrade to basic or upgrade to an annual pro plan at any point during the trial.
    - Basic Monthly — Customers have limited access and can only stream their videos and is only available monthly at $9.90.
    - Pro Monthly — Customers have no watch time limits and are able to download videos for offline viewing. This plan starts at $19.90 a month
    - Pro Annualy - Same benefits of a Pro Plan, with an annual subscription of $199
    - Churn - When customers cancel their Foodie-Fi service — they will have a Churn plan record with a null price, but their plan will continue until the end of the billing period.
<img src = "https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/main/Case%20Study%203%20-%20Foodie-Fi/ref_images/plans.webp" alt="Image" width ="300" height="320">

2. `subscriptions` -
      - Customer subscriptions show the exact date where their specific `plan_id` starts.
      - If customers downgrade from a pro plan or cancel their subscription - the higher plan will remain in place until the period is over - the `start_date` in the subscriptions table will reflect the date that the actual plan changes.
      - When customers upgrade their account from a basic plan to a pro monthly or annual pro plan - the higher plan will take effect straightaway.
      - When customers churn - they will keep their access until the end of their current billing period but the `start_date` will be technically the day they decided to cancel their service.
<img src = "https://github.com/keshavdewan/8-Weeks-SQL-Challenge/blob/main/Case%20Study%203%20-%20Foodie-Fi/ref_images/subscriptions.webp" alt="Image" width ="300" height="320">

## Questions & Solutions

### A. Customer Journey
Based off the 8 sample customers provided in the sample from the subscriptions table, write a brief description about each customer’s onboarding journey.

_Approach taken_ - Randomly selected 8 customers from the subscription table, they could be seen in the table image itself
````sql
SELECT 	s.customer_id,
		s.plan_id,
		p.plan_name,
		p.price,
		s.start_date
FROM subscriptions s
JOIN plans p ON p.plan_id = s.plan_id
WHERE s.customer_id IN (1,2,11,13,15,16,18,19)
````
#### Solution:
| customer_id | plan_id | plan_name     | start_date  |
|-------------|---------|---------------|-------------|
| 1           | 0       | trial         | 2020-08-01  |
| 1           | 1       | basic monthly | 2020-08-08  |
| 2           | 0       | trial         | 2020-09-20  |
| 2           | 3       | pro annual    | 2020-09-27  |
| 11          | 0       | trial         | 2020-11-19  |
| 11          | 4       | churn         | 2020-11-26  |
| 13          | 0       | trial         | 2020-12-15  |
| 13          | 1       | basic monthly | 2020-12-22  |
| 13          | 2       | pro monthly   | 2021-03-29  |
| 15          | 0       | trial         | 2020-03-17  |
| 15          | 2       | pro monthly   | 2020-03-24  |
| 15          | 4       | churn         | 2020-04-29  |
| 16          | 0       | trial         | 2020-05-31  |
| 16          | 1       | basic monthly | 2020-06-07  |
| 16          | 3       | pro annual    | 2020-10-21  |
| 18          | 0       | trial         | 2020-07-06  |
| 18          | 2       | pro monthly   | 2020-07-13  |
| 19          | 0       | trial         | 2020-06-22  |
| 19          | 2       | pro monthly   | 2020-06-29  |
| 19          | 3       | pro annual    | 2020-08-29  |

- Customer behaviour observations:
    - Customer_id: 1 - started with a trial and converted the trial to basic monthly subscription
    - Customer_id: 2 - started with a trial and converted it to pro annual, customer 2 definitely liked the content of the app
    - Customer_id: 11 - didn't go ahead after the trail phase
    - Customer_id: 13 - choose basic monthly plan after the end of their trial and later after 3 months moved to a pro monthly plan
    - Customer_id: 15 - started with a trial, moved to pro monthly after the trial ended (which might have been done automatically) and later churned out after a month
    - Customer_id: 16 - moved to basic monthly after the trial and later moved to pro annual after 4 months of end of basic monthly 
    - Customer_id: 18 - started with trial and converted it to pro monthly
    - Customer_id: 19 - started with trial and converted it to pro monthly and later changed to pro annual after two monhts 

***
### B. Data Analysis Questions
## 1. How many customers has Foodie-Fi ever had?
_Approach taken_ - Use Count & Distinct functions for calculating the number of customers
````sql
SELECT 	COUNT(DISTINCT(customer_id)) AS customer_count
FROM subscriptions
````

#### Solution:
| customer_count | 
|-------------|
|1000           |
