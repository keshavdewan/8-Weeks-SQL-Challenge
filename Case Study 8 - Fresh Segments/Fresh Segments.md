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
    - 
    <img src = "https://github.com/user-attachments/assets/08b46fbc-0aad-4dc9-b6fc-35c8d6a66f10" alt = "Image" width = "550" height = "500">
    
-  `interest_map`
    -  This mapping table links the `interest_id` with their relevant interest information. We will need to join this table onto the previous `interest_details` table to obtain the `interest_name` as well as any details about the summary information
      <img src = "https://github.com/user-attachments/assets/aa80920e-e1d3-46c0-9db9-8ca081486bbe" alt = "Image" width = "550" height = "400">

***

## Case Study Questions with Solutions

### A. Data Exploration and Cleansing

#### 1. Update the `fresh_segments.interest_metrics` table by modifying the `month_year` column to be a date data type with the start of the month

