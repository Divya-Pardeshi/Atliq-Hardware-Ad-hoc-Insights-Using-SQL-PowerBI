# Consumer-Goods-Ad-hoc-Insights-Using-SQL-PowerBI
# SQL Challenge: Atliq Hardwares Ad hoc Insights Consumer Goods Domain

## Project Overview
As part of a challenge for Atliq Hardwares, a leading computer hardware producer, I performed a comprehensive SQL analysis to provide actionable insights from their data. My task involved crafting SQL queries to answer specific business questions.

## Problem Statement
Atliq Hardwares, a prominent computer hardware producer with an extensive presence both in India and internationally, sought to enhance its data-driven decision-making capabilities. The management recognized the need for deeper insights into their data to make informed decisions. This SQL challenge was designed to address this need by analyzing various aspects of their sales and product data.

## Tasks
- Analyze and provide insights based on specific business questions related to sales, products, and customer data.
- Demonstrate proficiency in SQL and data analysis.
- Present findings in a clear and actionable manner for top-level management.

## Dataset Overview

The analysis was conducted using data from the 'gdb023' (atliq_hardware_db) database, which includes six main tables. 

- dim_customer: contains customer-related data
- dim_product: contains product-related data
- fact_gross_price: contains gross price information for each product
- fact_manufacturing_cost: contains the cost incurred in the production of each product
- fact_pre_invoice_deductions: contains pre-invoice deductions information for each product
- fact_sales_monthly: contains monthly sales data for each product.

## Tools Used

- **SQL**: For querying and analyzing the data.
- **Visualization Tool**: Power Bi for data visualisation.
- **Canva**: For Presentation
  
## Ad-hoc Requests & SQL Queries

### 1. Provide the list of markets in which customer "Atliq Exclusive" operates its business in the APAC region.

```SELECT DISTINCT market FROM dim_customer
WHERE customer = 'Atliq Exclusive'
AND region = 'APAC';

### 2.What is the percentage of unique product increase in 2021 vs. 2020? The final output contains these fields: unique_products_2020, unique_products_2021 and percentage_chg
```WITH CTE AS
(SELECT 
    COUNT(DISTINCT product_code) AS unique_products_2020
FROM
    fact_sales_monthly
WHERE
    fiscal_year = 2020),
CTE2 AS
(SELECT 
    COUNT(DISTINCT product_code) AS unique_products_2021
FROM
    fact_sales_monthly
WHERE
    fiscal_year = 2021)
SELECT unique_products_2020, unique_products_2021,
ROUND(((unique_products_2021-unique_products_2020)*100.0/unique_products_2020),2) AS percentage_chg
FROM 
cte,cte2;```

### 3.Provide a report with all the unique product counts for each segment and sort them in descending order of product counts. The final output contains 2 fields: segment and product_count
```SELECT segment, COUNT(DISTINCT product_code) AS product_count FROM 
dim_product
GROUP BY segment
ORDER BY product_count DESC;```

### 4. Follow-up: Which segment had the most increase in unique products in 2021 vs 2020? The final output contains these fields: segment, product_count_2020, product_count_2021 and difference
```WITH cte1 AS
(SELECT dp.segment,
    COUNT(DISTINCT dp.product_code) AS product_count_2020
FROM dim_product dp
JOIN fact_sales_monthly fsm 
ON dp.product_code = fsm.product_code
WHERE fsm.fiscal_year = 2020
GROUP BY dp.segment),
cte2 AS
(SELECT dp.segment,
    COUNT(DISTINCT dp.product_code) AS product_count_2021
FROM dim_product dp
JOIN fact_sales_monthly fsm 
ON dp.product_code = fsm.product_code
WHERE fsm.fiscal_year = 2021
GROUP BY dp.segment)
SELECT cte1.segment,product_count_2020,product_count_2021, (product_count_2021-product_count_2020) AS difference
FROM cte1 JOIN cte2
ON cte1.segment = cte2.segment;```

### 5. Get the products that have the highest and lowest manufacturing costs.The final output should contain these fields: product_code, product and manufacturing_cost
```(SELECT 
    dp.product_code, dp.product, manufacturing_cost
FROM
    dim_product dp
         JOIN
    fact_manufacturing_cost fmc ON dp.product_code = fmc.product_code
ORDER BY manufacturing_cost DESC
LIMIT 1)
UNION
(SELECT 
    dp.product_code, dp.product, manufacturing_cost
FROM
    dim_product dp
        JOIN
    fact_manufacturing_cost fmc ON dp.product_code = fmc.product_code
ORDER BY manufacturing_cost ASC
LIMIT 1);```

### 6. Generate a report which contains the top 5 customers who received an average high pre_invoice_discount_pct for the fiscal year 2021 and in the Indian market. The final output contains these fields: customer_code, customer and average_discount_percentage
```SELECT 
    dc.customer_code,
    dc.customer,
    ROUND(AVG(pre_invoice_discount_pct) * 100, 2) AS average_discount_percentage
FROM
    dim_customer dc
        LEFT JOIN
    fact_pre_invoice_deductions fpre ON dc.customer_code = fpre.customer_code
WHERE
    fiscal_year = 2021 AND market = 'India'
GROUP BY dc.customer_code , dc.customer
ORDER BY average_discount_percentage DESC
LIMIT 5;```

### 7. Get the complete report of the Gross sales amount for the customer “Atliq Exclusive” for each month. This analysis helps to get an idea of low and high-performing months and take strategic decisions. The final report contains these columns: Month, Year and Gross sales Amount
```SELECT MONTHNAME(fsm.date) AS MONTH, 
fsm.fiscal_year as YEAR,
CONCAT(FORMAT(SUM(fsm.sold_quantity*fgp.gross_price)/1000000,2),'M')
AS Gross_Sales_Amount
FROM fact_sales_monthly fsm
JOIN dim_customer c 
ON fsm.customer_code = c.customer_code
JOIN fact_gross_price fgp
ON fsm.product_code = fgp.product_code
WHERE c.customer = 'AtliQ Exclusive'
GROUP BY MONTH, YEAR
ORDER BY fsm.fiscal_year;```

### 8. In which quarter of 2020, got the maximum total_sold_quantity? The final output contains these fields sorted by the total_sold_quantity, Quarter and total_sold_quantity
```SELECT 
    CASE
        WHEN date BETWEEN '2019-09-01' AND '2019-11-01' THEN 'Q1'
        WHEN date BETWEEN '2019-12-01' AND '2020-02-01' THEN 'Q2'
        WHEN date BETWEEN '2020-03-01' AND '2020-05-01' THEN 'Q3'
        WHEN date BETWEEN '2020-06-01' AND '2020-08-01' THEN 'Q4'
    END AS Quarters,
    ROUND(SUM(sold_quantity) / 1000000, 2) AS total_sold_quantity_in_mln
FROM
    fact_sales_monthly
WHERE
    fiscal_year = 2020
GROUP BY quarters
ORDER BY total_sold_quantity_in_mln DESC;```

### 9. Which channel helped to bring more gross sales in the fiscal year 2021 and the percentage of contribution? The final output contains these fields: channel, gross_sales_mln and percentage
```WITH cte AS (
    SELECT 
        channel, 
        ROUND(SUM(gross_price * sold_quantity) / 1000000, 2) AS gross_sales_mln
    FROM 
        fact_sales_monthly s
    JOIN 
        fact_gross_price g USING (product_code, fiscal_year)
    JOIN 
        dim_customer c USING (customer_code)
    WHERE 
        fiscal_year = 2021
    GROUP BY 
        channel
)
SELECT 
    channel,
    gross_sales_mln,
    ROUND((gross_sales_mln * 100) / SUM(gross_sales_mln) OVER (), 2) AS pct
FROM 
    cte
ORDER BY 
    pct DESC;```

### 10. Get the Top 3 products in each division that have a high total_sold_quantity in the fiscal_year 2021? The final output contains these fields: division, product_code, product and total_sold_quantity 
```WITH cte1 AS (
    SELECT p.division,
           s.product_code,
           CONCAT(p.product, " (", p.variant, ")") AS product,
           SUM(s.sold_quantity) AS total_sold_qty,
           ROW_NUMBER() OVER (PARTITION BY p.division ORDER BY SUM(s.sold_quantity) DESC) AS rank_order
    FROM dim_product p
    JOIN fact_sales_monthly s 
        ON p.product_code = s.product_code
    WHERE s.fiscal_year = 2021
    GROUP BY p.division, s.product_code, p.product, p.variant
)
SELECT division, product_code, product, total_sold_qty, rank_order
FROM cte1
WHERE rank_order IN (1, 2, 3)
ORDER BY division, rank_order ASC;```


## Power BI Report
Click here to explore the report: https://app.powerbi.com/view?r=eyJrIjoiNDNkZDczNDMtZTIwNC00MGFjLTg5YWEtZDdhNzM1YTRmMjE1IiwidCI6ImRkY2EyNGM4LTJkNDUtNGQwNS1hZWY1LWMyYTZhMTFhNzdmYSJ9&pageName=6e276cc4ba3db0cece31
