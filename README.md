
![image](https://github.com/MominaKhurram/SQL/assets/145676374/5c137aeb-0c9b-48db-9211-79d14b77e87d)


In this case study, two tables were created with the names of plans and subscriptions and then values were entered with insert function.
# plans table
![image](https://github.com/MominaKhurram/SQL/assets/145676374/c7625a12-2ea9-4c12-bb7a-cc3f7b1400a4)

# subscritions table

![image](https://github.com/MominaKhurram/SQL/assets/145676374/0167c43f-09e4-4fd2-996f-7f50e806c712)

Different 
#Customer Journey
select s.customer_id, p.plan_name, s.start_date
from subscriptions s
JOIN plans p ON s.plan_id = p.plan_id;
  
  ![image](https://github.com/MominaKhurram/SQL/assets/145676374/0e021e10-6468-4754-86d3-73dea005393e)

  /*1. How many customers has Foodie-Fi ever had? */
  SELECT COUNT(distinct customer_id) AS total_customers
  FROM subscriptions;

  
 ![image](https://github.com/MominaKhurram/SQL/assets/145676374/99a552c4-d0b2-4382-a3a0-7f24328112d3)

  
   /* 2. What is the monthly distribution of trial plan start_date values for our dataset - use  the start of the month as the group by value */
  
SELECT month(start_date) AS months,
     	COUNT(customer_id) AS trial_plan_count	-- counts the no of rows with each group formed by 'GROUP BY'clause
FROM subscriptions
GROUP BY months
ORDER BY months ;

![image](https://github.com/MominaKhurram/SQL/assets/145676374/17f70dd3-8bdd-4ec9-b3fd-42ffe2237c92)

  /*3. What plan start_date values occur after the year 2020 for our dataset? Show the breakdown by count of events for each plan_name */
  
   SELECT p.plan_name,p.plan_id,count(*) AS count_event
  FROM subscriptions s
  JOIN plans p
	ON p.plan_id = s.plan_id
  WHERE YEAR(start_date) > 2020
  GROUP BY plan_name, p.plan_id
  ORDER BY p.plan_id;
  
  ![image](https://github.com/MominaKhurram/SQL/assets/145676374/0b9ab664-8a94-4413-8ccc-24ae015792a1)

  
/*4. What is the customer count and percentage of customers who have churned rounded to 1 decimal place? */  
  
 SELECT COUNT(*)  AS total_churned_customers,
ROUND(count(*) / (SELECT COUNT(DISTINCT customer_id) FROM subscriptions)
  * 100, 1) AS churned_percentage
  FROM subscriptions
  WHERE plan_id = 4;

![image](https://github.com/MominaKhurram/SQL/assets/145676374/b5e3f13c-0859-402a-b3f4-ac5b4161eefd)

  
/* 5. How many customers have churned straight after their initial free trial - what percentage is
 this rounded to the nearest whole number?*/

  
 WITH CTE_churn AS (
	select *,
    LAG(plan_id,1) OVER(partition by customer_id order by plan_id) as previous_plan
    FROM subscriptions
    )
    select count(previous_plan) as count_churn,
		round(count(*) / (select count(distinct customer_id) from subscriptions)*100,0) as percentage_churn
        from cte_churn
        where plan_id = 4 and previous_plan = 0;
 

 ![image](https://github.com/MominaKhurram/SQL/assets/145676374/57dd5b11-8893-4c80-990e-3a6c90c9f1e5)

 
 
/* 6. What is the number and percentage of customer plans after their initial free trial?*/

WITH CTE_next_plan AS (
	select *,
    LEAD(plan_id,1) OVER(partition by customer_id order by plan_id) as next_plan
    FROM subscriptions
    )
    select next_plan, count(*) as no_of_customer,
		round(count(*) / (select count(distinct customer_id) from subscriptions)*100,1) as percentage_next_plan
        from cte_next_plan
        where next_plan is not null and plan_id = 0
        group by next_plan
        order by next_plan;

 ![image](https://github.com/MominaKhurram/SQL/assets/145676374/d3226eb2-57c7-4de8-98c1-1210eab05c96)


/*7. What is the customer count and percentage breakdown of all 5 plan_name values at 2020-12-31?*/
WITH CTE_next_date AS (
	select *,
    LEAD(start_date,1) OVER(partition by customer_id order by start_date) as next_date
    FROM subscriptions
    where start_date<= '2020-12-31'),
    plans_breakdown as(
    select 
    plan_id,
		count(distinct customer_id) as number_of_customer
from CTE_next_date
	where (next_date is not null and (start_date< '2020-12-31' and next_date > '2020-12-31'))
		OR(next_date is null and start_date < '2020-12-31')
        group by plan_id)  
    select plan_id,
		number_of_customer,
		round( number_of_customer/ (select count(distinct customer_id) from subscriptions)*100,1) as percentage_of_customer
        from plans_breakdown
        group by plan_id, number_of_customer
        order by plan_id;

 ![image](https://github.com/MominaKhurram/SQL/assets/145676374/c9ad02ac-2019-463f-ad49-2e1d95823f88)

    
/*8. How many customers have upgraded to an annual plan in 2020? */
WITH cte_upgraded_annual_plan AS (
select distinct(customer_id)
FROM subscriptions
where plan_id = 3
    AND year(start_date) = 2020
)
select count(* ) from cte_upgraded_annual_plan;

![image](https://github.com/MominaKhurram/SQL/assets/145676374/d39c8d94-36e5-4f15-b2c0-a4b33e0b57d1)


/*9. How many days on average does it take for a customer to an annual plan from the day they join Foodie-Fi? */

WITH AnnualPlanCustomers AS (
    SELECT 
        customer_id,
        MIN(start_date) AS join_date,
        MIN(CASE WHEN plan_id = 3 THEN start_date END) AS annual_plan_start_date
    FROM 
        subscriptions
    GROUP BY 
        customer_id
    HAVING 
        annual_plan_start_date IS NOT NULL
)
SELECT 
    ROUND(AVG(DATEDIFF(annual_plan_start_date, join_date)),0) AS average_days_to_annual_plan
FROM 
    AnnualPlanCustomers;

![image](https://github.com/MominaKhurram/SQL/assets/145676374/adeb9675-d7e2-4904-91c3-e90aea14032f)


/* 10. Can you further breakdown this average value into 30 day periods (i.e. 0-30 days, 31-60  days etc) */

WITH AnnualPlanUpgrades AS (
    SELECT 
        customer_id,
        MIN(start_date) AS join_date,
        MIN(CASE WHEN plan_id = 3 THEN start_date END) AS upgrade_date,
        DATEDIFF(MIN(CASE WHEN plan_id = 3 THEN start_date END), MIN(start_date)) AS difference_in_days
    FROM 
        subscriptions
    GROUP BY 
        customer_id
    HAVING 
        upgrade_date IS NOT NULL
),
AverageDays AS (
    SELECT 
        AVG(difference_in_days) AS avg_days_to_annual_plan
    FROM 
        AnnualPlanUpgrades
)

SELECT 
    CASE 
        WHEN difference_in_days BETWEEN 0 AND 30 THEN '0-30 days'
        WHEN difference_in_days BETWEEN 31 AND 60 THEN '31-60 days'
        ELSE '> 60 days'
    END AS period,
    COUNT(*) AS num_customers,
    AVG(difference_in_days) AS average_days_to_annual_plan
FROM 
    AnnualPlanUpgrades
GROUP BY 
    period;
    
![image](https://github.com/MominaKhurram/SQL/assets/145676374/88fe3590-73a0-4ec3-9499-acbe2d54483b)


/* 11. How many customers downgraded from a pro monthly to a basic monthly plan in 2020?*/
WITH ProMonthlyCustomers AS (
    SELECT DISTINCT customer_id
    FROM subscriptions
    WHERE plan_id = 2  -- Pro monthly plan
        AND start_date >= '2020-01-01'
        AND start_date < '2021-01-01'
),
BasicMonthlyCustomers AS (
    SELECT DISTINCT customer_id
    FROM subscriptions
    WHERE plan_id = 1  -- Basic monthly plan
        AND start_date >= '2020-01-01'
        AND start_date < '2021-01-01'
),
DowngradedPlan AS (
    SELECT DISTINCT p.customer_id
    FROM ProMonthlyCustomers p
    JOIN BasicMonthlyCustomers b ON p.customer_id = b.customer_id
)
SELECT COUNT(*) AS num_downgrades
FROM DowngradedPlan;

![image](https://github.com/MominaKhurram/SQL/assets/145676374/c3a93ab0-1a6a-4a6b-923c-105caaf785fc)
