# Case Study 6 - Clique Bait🎣
Clique Bait is an online seafood store, founded by Danny who also happens to be its CEO.

# Task
In this case study - we will be required to support Danny’s vision and analyse his dataset and come up with creative solutions to calculate funnel fallout rates for the Clique Bait online store.

<img src = "https://github.com/user-attachments/assets/33df4d09-be8b-48f2-9e46-8bdbbbd38625" alt = "Image" width = "500" height = "520">

# Table of Contents 
The case study has been divided into the following parts:
-  [Entity Relation Diagram](#entity-relation-diagram)
-  [Case Study Questions with Solutions](#case-study-questions-with-solutions)
    -   [A. Digital Analysis](#a-digital-analysis)
    -   [B. Product Funnel Analysis](#b-product-funnel-analysis)
    -   [C. Campaign Analysis](#c-campaign-analysis)
-   [Learnings](#learnings)

***

# Entity Relation Diagram
Data can be taken from [dbfiddle](https://www.db-fiddle.com/f/jmnwogTsUE8hGqkZv9H7E8/17)

![image](https://github.com/user-attachments/assets/0f52fdd6-9389-4878-8342-ddd8057b2cf7)

The database consists of 5 Tables:
1. `users` - Customers who visit the Clique Bait website are tagged via their `cookie_id`
<img width="366" alt="134623074-7c51d63a-c0a4-41e0-a6fc-257e4ca3997d" src="https://github.com/user-attachments/assets/2e654e7a-d962-48f9-aaab-a50322de28df" />

2. `events` - Customer visits are logged in this events table at a `cookie_id` level and the `event_type` and `page_id` values can be used to join onto relevant satellite tables to obtain further information about each event. The `sequence_number` is used to order the events within each visit.
<img width="849" alt="134623132-dfa2acd3-60c9-4305-9bea-6b39a9403c14" src="https://github.com/user-attachments/assets/eb4ef4c8-f553-4da0-a8c2-ef6df66f9dc2" />

3. `event identifier` - The `event_identifier` table shows the types of events which are captured by Clique Bait’s digital data systems.
<img width="273" alt="134623311-1ad16fe7-36e3-45b6-9dc6-8114333cf473" src="https://github.com/user-attachments/assets/6aa77aae-3da2-48eb-87d7-eb003b1c7e4f" />

4. `campaign identifier` - This table shows information for the 3 campaigns that Clique Bait has ran on their website so far in 2020.
<img width="792" alt="134623354-0977d67c-fc61-4e61-90ee-f24a29682a9b" src="https://github.com/user-attachments/assets/202c1f21-80c3-466a-b5fb-8ee8cd25663b" />

5. `page hierarchy` - This table lists all of the pages on the Clique Bait website which are tagged and have data passing through from user interaction events.
<img width="576" alt="134623202-3158ca06-6f04-4b67-91f1-e184761e885c" src="https://github.com/user-attachments/assets/bec578da-e854-4d74-877f-8f1402b15675" />

***
# Case Study Questions with Solutions
## A. Digital Analysis

### 1. How many users are there?
_Approach Taken_
-    used `COUNT(DISTINCT)` function for calculating the numbers from `user_id`

````sql
SELECT COUNT(DISTINCT user_id)
FROM clique_bait.users
````
![image](https://github.com/user-attachments/assets/442829ec-aaa4-40e1-8584-d0b37893008b)

### 2. How many cookies does each user have on average?
_Approach Taken_
-  CTE `cookie_count` counts the number of cookies for each user
-  Final `SELECT` statement calculates the average of this count

````sql
WITH cookie_count AS(
		SELECT user_id,
				COUNT(cookie_id) AS cookie_id_count
		FROM clique_bait.users
		GROUP BY user_id
		)
SELECT ROUND(AVG(cookie_id_count),0) AS avg_user_cookies
FROM cookie_count
````
![image](https://github.com/user-attachments/assets/4b47ee0e-afc9-4331-946f-247b7e1172d1)

### 3. What is the unique number of visits by all users per month?
_Approach Taken_
-    Used `EXTRACT` to get the months and `DISTINCT` to count the visits

````sql
SELECT 	EXTRACT(MONTH FROM event_time) AS month,
		COUNT(DISTINCT visit_id) AS unique_visits 
FROM clique_bait.events
GROUP BY month
````

![image](https://github.com/user-attachments/assets/e80d727f-71ed-4132-ba46-6d26c0da9402)

### 4. What is the number of events for each event type?


