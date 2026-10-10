# subscription_data_analysis_dashboard
#### End-to-end subscription analytics project covering data cleaning, SQL Server, Google BigQuery, data modeling, DAX, Power BI and Looker Studio.

---

## Table of content

- [SQL Query](#SQL-Query-for-final-result)
  
## Subscription Analytics & Business Intelligence Dashboard
I recently completed an end-to-end Subscription Analytics project for an online services business, transforming raw subscription data into interactive dashboards and actionable business insights.
What I worked on:
• Cleaned and prepared subscription data using Google BigQuery and SQL Server
• Connected cleaned datasets to Power BI and Google Looker Studio
• Created data models and relationships between multiple tables
• Developed required DAX measures and calculated fields
• Designed interactive dashboards with clear KPIs, charts, filters, and business-focused visualizations
• Created custom rounded-corner and gradient bar/column charts in Power BI
• Analyzed subscription performance, profitability, cancellations, pauses, and trends
• Investigated geographical differences in subscription performance
• Identified potential reasons behind lower subscription volumes in September and October
• Analyzed why the United Kingdom has stronger subscription performance compared with other markets
• Evaluated whether the current subscription trend is healthy and sustainable
• Identified areas where marketing strategies and customer acquisition initiatives could improve subscriber growth


## Business Questions!

1. Why is the subscription cancelled or paused?
2. Why are September and October subscriptions lower?
3. Why is the United Kingdom subscription higher, and why are the others not?
4. Are any marketing strategies needed to improve subscribers?


## SQL Query for final result

``` sql

-- Drop the target table if it already exists
IF OBJECT_ID('[subscriptions].[dbo].[subscriptions_cleaned_data]', 'U') IS NOT NULL
    DROP TABLE [subscriptions].[dbo].[subscriptions_cleaned_data];

WITH cleaned_new_table AS (
  SELECT
    subscription_id,
    customer_id,
    CASE 
      WHEN customer_email LIKE '%@%' THEN LOWER(TRIM(customer_email)) 
    END AS customer_email,
    
    CASE
      WHEN UPPER(TRIM(country)) IN ('UK', 'U.K.', 'UNITED KINGDOM') THEN 'United Kingdom'
      WHEN UPPER(TRIM(country)) IN ('DE', 'GERMANY') THEN 'Germany'
      WHEN UPPER(TRIM(country)) IN ('IE', 'IRELAND') THEN 'Ireland'
      WHEN UPPER(TRIM(country)) IN ('IT', 'ITALY') THEN 'Italy'
      ELSE UPPER(LEFT(TRIM(country), 1)) + LOWER(SUBSTRING(TRIM(country), 2, LEN(country)))
    END AS country,
    
    UPPER(LEFT(TRIM([plan]), 1)) + LOWER(SUBSTRING(TRIM([plan]), 2, LEN([plan]))) AS [plan],
    
    CASE
      WHEN monthly_price < 0 THEN ABS(monthly_price)
      WHEN monthly_price > 100 THEN monthly_price / 100
      ELSE monthly_price
    END AS monthly_price,
    
    REPLACE(LOWER(TRIM([status])), 'canceled', 'cancelled') AS [status],
    
    CASE 
      WHEN TRY_CAST(start_date AS DATE) <= CAST(GETDATE() AS DATE) THEN TRY_CAST(start_date AS DATE) 
    END AS start_date,
    
    TRY_CAST(end_date AS DATE) AS end_date,
    
    CASE
      WHEN payment_method IS NULL OR payment_method IN ('N/A', 'NULL') THEN 'unknown'
      ELSE payment_method
    END AS payment_method,
    
    updated_at,
    
    CASE
      WHEN TRY_CAST(end_date AS DATE) < (
        CASE WHEN TRY_CAST(start_date AS DATE) <= CAST(GETDATE() AS DATE) THEN TRY_CAST(start_date AS DATE) END
      ) THEN 'ended before it started'
      WHEN LOWER(REPLACE(LOWER(TRIM([status])), 'canceled', 'cancelled')) LIKE '%active%' AND TRY_CAST(end_date AS DATE) IS NOT NULL 
        THEN 'active with an end date'
      WHEN LOWER(REPLACE(LOWER(TRIM([status])), 'canceled', 'cancelled')) LIKE '%cancel%' AND TRY_CAST(end_date AS DATE) IS NULL 
        THEN 'cancelled without an end date'
      WHEN LOWER(TRIM([plan])) NOT LIKE '%free%' AND (
        CASE
          WHEN monthly_price < 0 THEN ABS(monthly_price)
          WHEN monthly_price > 100 THEN monthly_price / 100
          ELSE monthly_price
        END
      ) = 0 
        THEN 'paid plan with zero price'
      ELSE 'ok'
    END AS quality_flag

  FROM [subscriptions].[dbo].[subscriptions_raw]
)
-- Create new table in SQL Server using INTO
SELECT *
INTO [subscriptions].[dbo].[subscriptions_cleaned_data]
FROM cleaned_new_table;
```

Happy data analysis!
😄

