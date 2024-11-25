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
