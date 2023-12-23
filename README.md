# My Analysis for Women Empowerment in Côte d'Ivoire: Weaving the Threads of Women's Progress ( Full Project)

**Name**: Moaz Wael Hanafy <br />
**Email**: moazwael1997@gmail.com <br />
**LinkedIn**: [View Profile](https://www.linkedin.com/in/moaz-wael-14212323a) 


### project overview
This project involves a comprehensive analysis of business data, utilizing a blend of SQL for robust data querying, Power BI for advanced data visualization and dashboard creation, and Excel for data manipulation and additional analysis. The primary objective is to uncover actionable insights into customer behaviors, sales patterns, and operational efficiency. By integrating these powerful tools, the project aims to provide a holistic view of the business's performance, enabling data-driven decision-making. Key deliverables include an interactive dashboard for easy data exploration and detailed reports for strategic planning. Targeted towards business stakeholders and decision-makers, this project is set to drive informed strategies and optimize business outcomes.

### Tools
- Excel for data cleaning.
- Mysql for analysis.
- PoweBi for creating a report.

### Explore data analysis

#### 1- Calculate monthly profits for 2021 and 2022 and the percentage change:

```SQL
-- Calculate monthly profits for 2021 and 2022 and the percentage change
WITH Profit AS (  
    SELECT 
        EXTRACT(MONTH FROM T.date) AS Number,
        MONTHNAME(T.date) AS Month,
        SUM(CASE WHEN YEAR(T.date) = 2021 THEN T.quantity * P.profit ELSE 0 END) AS Total_Profit_2021,
        SUM(CASE WHEN YEAR(T.date) = 2022 THEN T.quantity * P.profit ELSE 0 END) AS Total_Profit_2022
    FROM 
        transactions AS T
    INNER JOIN 
        product_data AS P ON P.product_id = T.product_id
    GROUP BY 
        Number,
        Month
),
TotalProfit2021 AS (
    SELECT 
        SUM(Total_Profit_2021) AS Total_Annual_Profit_2021
    FROM 
        Profit
)
SELECT 
    P.Number,
    P.Month,
    ROUND(P.Total_Profit_2021, 2) AS Total_Profit_2021,
    ROUND(P.Total_Profit_2022, 2) AS Total_Profit_2022,
    ROUND(((P.Total_Profit_2022 - P.Total_Profit_2021) / NULLIF(TP.Total_Annual_Profit_2021, 0)) * 100, 2) AS Percentage_Difference
FROM 
    Profit P,
    TotalProfit2021 TP
ORDER BY 
    P.Number;
```
###### Findings
![Percentage_Difference](https://github.com/MoazWael2/ecommerce-food-app-analysis/assets/137816418/e3b309ba-a895-4932-b7f1-647bf3513af3)

#### 2- How does customer spend vary by demographic segments?
###### SQL Query
```SQL
 -- How does customer spend vary by demographic segments?
 SELECT 
      C.customer_country as country,
      ROUND((Count(T.quantity) / Count(distinct T.customer_id) ),2) AS AVG_Purchase_Frequancy,
      Round((SUM(P.profit) / Count(distinct T.customer_id))*100,2) AS AVG_Profit_Per_Customer,
      Round((SUM(T.quantity * product_retail_price) / Count(T.quantity))*100,2) AS AVG_Order_Value
 FROM 
    transactions AS T
INNER JOIN 
    product_data AS P ON P.product_id = T.product_id
INNER JOIN
   customer_data AS C ON T.customer_id = C.customer_id
GROUP BY 
        C.customer_country;
``
###### Findings
![spending](https://github.com/MoazWael2/ecommerce-food-app-analysis/assets/137816418/123d5bb8-53aa-419a-8a15-b1d98fb993f6)

#### 3- What are the top-five performing products each quarter?
###### SQL Query
``` SQL
-- SQL Query to Find Top 5 Products by Total Orders in Each Quarter for 2022

-- Calculate total orders for each product in each quarter of 2022
WITH Product_Order AS (
    SELECT 
        CASE 
            WHEN date BETWEEN '2022-01-01' AND '2022-03-31' THEN 'Q1'
            WHEN date BETWEEN '2022-04-01' AND '2022-06-30' THEN 'Q2'
            WHEN date BETWEEN '2022-07-01' AND '2022-09-30' THEN 'Q3'
            WHEN date BETWEEN '2022-10-01' AND '2022-12-31' THEN 'Q4' 
        END AS Quarter,
        P.product_name,
        COUNT(T.quantity) AS Total_Order
    FROM 
        transactions AS T
    INNER JOIN 
        product_data AS P ON P.product_id = T.product_id
    WHERE 
        YEAR(T.date) = 2022
    GROUP BY 
        Quarter,
        P.product_name
),

-- Assign a row number to each product within its quarter
Roww AS (
    SELECT 
        ROW_NUMBER() OVER (PARTITION BY Quarter ORDER BY Total_Order DESC) AS Row_,
        Quarter,
        product_name,
        Total_Order
    FROM  
        Product_Order
)

-- Select the top 5 products in each quarter
SELECT
    Quarter,
    product_name,
    Total_Order
FROM 
    Roww
WHERE 
    Row_ <= 5
ORDER BY 
    Quarter, Total_Order DESC;
```
###### Findings
![top product](https://github.com/MoazWael2/ecommerce-food-app-analysis/assets/137816418/23103728-6fd9-4b6f-b6f0-2f8c94be101d)

#### 4- Which customers have increased their purchase frequency the most in the past year?
###### SQL Query
```SQL
-- SQL Query to Identify Customers with Increased Purchase Frequency from 2021 to 2022

-- Calculate purchase frequency for each customer in 2021
WITH APF2021 AS (
    SELECT 
        CONCAT(C.first_name, ' ', C.last_name) AS Full_Name,
        C.customer_acct_num,
        COUNT(T.transaction_id) AS purchase_frequency_2021  -- Assuming transaction_id is a unique identifier for each transaction
    FROM 
        customer_data AS C
    INNER JOIN 
        transactions AS T ON T.customer_id = C.customer_id 
    WHERE 
        YEAR(T.date) = 2021
    GROUP BY 
        Full_Name, C.customer_acct_num
),

-- Calculate purchase frequency for each customer in 2022
APF2022 AS (
    SELECT 
        CONCAT(C.first_name, ' ', C.last_name) AS Full_Name,
        C.customer_acct_num,
        COUNT(T.transaction_id) AS purchase_frequency_2022  -- Assuming transaction_id is a unique identifier for each transaction
    FROM 
        customer_data AS C
    INNER JOIN 
        transactions AS T ON T.customer_id = C.customer_id 
    WHERE 
        YEAR(T.date) = 2022
    GROUP BY 
        Full_Name, C.customer_acct_num
)

-- Compare purchase frequencies between 2021 and 2022 to find customers with increased frequency
SELECT 
    A2.Full_Name,
    A2.customer_acct_num,
    A1.purchase_frequency_2021,
    A2.purchase_frequency_2022
FROM 
    APF2022 AS A2
INNER JOIN 
    APF2021 AS A1 ON A2.customer_acct_num = A1.customer_acct_num
WHERE 
    A2.purchase_frequency_2022 > A1.purchase_frequency_2021
ORDER BY 
    (A2.purchase_frequency_2022 - A1.purchase_frequency_2021) DESC;
```
###### Findings
![22 12 2023_15 56 04_REC](https://github.com/MoazWael2/ecommerce-food-app-analysis/assets/137816418/e4054433-bb8c-4299-9582-e84fc0503e5f)


#### 4- Is there an improvement in the return rates of the top products with the highest returns from 2021 to 2022?
###### SQL Query
```SQL
--  Is there an improvement in the return rates of the top products with the highest returns from 2021 to 2022?
WITH return2021 AS (
    SELECT 
        P.product_id,
        P.product_name,
        SUM(R.quantity) AS Total_items_return2021
    FROM 
        returns AS R
    INNER JOIN 
        product_data AS P ON P.product_id = R.product_id
    WHERE
        YEAR(R.Returnn_date) = 2021
    GROUP BY 
        P.product_id, P.product_name
), 
return2022 AS (
    SELECT 
        P.product_id,
        P.product_name,
        SUM(R.quantity) AS Total_items_return2022
    FROM 
        returns AS R
    INNER JOIN 
        product_data AS P ON P.product_id = R.product_id
    WHERE
        YEAR(R.Returnn_date) = 2022
    GROUP BY 
        P.product_id, P.product_name
)
SELECT 
    R21.product_id,
    R21.product_name,
    R21.Total_items_return2021,
    COALESCE(R22.Total_items_return2022, 0) AS Total_items_return2022,
    R21.Total_items_return2021 - COALESCE(R22.Total_items_return2022, 0) AS difference_in_returns
FROM 
    return2021 AS R21
LEFT JOIN 
    return2022 AS R22 ON R21.product_id = R22.product_id
ORDER BY 
    R21.Total_items_return2021 DESC
LIMIT 10;
```
###### Findings
![Return](https://github.com/MoazWael2/ecommerce-food-app-analysis/assets/137816418/1520883c-8b8c-40c0-8578-adbdc676d3cf)


#### 5- What are theTop 10 Products by Net Profit and Profit Margin?
###### SQL Query
```SQL
-- SQL Query to Identify Top 10 Products by Net Profit and Profit Margin

-- Calculate Net Profit and Revenue for each product
WITH Net_Profit AS ( 
    SELECT 
        P.product_name,
        -- Calculate Net Profit for each product
        SUM((T.quantity - R.quantity) * (P.product_retail_price - P.product_cost)) AS NET_PROFIT,
        -- Calculate Revenue for each product
        SUM((T.quantity - R.quantity) * P.product_retail_price) AS Revenue
    FROM
        product_data AS P
    LEFT JOIN
        -- Aggregate transactions by product
        (SELECT product_id, SUM(quantity) AS quantity FROM transactions GROUP BY product_id) AS T
        ON T.product_id = P.product_id
    LEFT JOIN 
        -- Aggregate returns by product
        (SELECT product_id, SUM(quantity) AS quantity FROM returns GROUP BY product_id) AS R
        ON R.product_id = P.product_id
    GROUP BY
        P.product_name
)
-- Select Top 10 products by Net Profit and Profit Margin
SELECT 
    product_name,
    -- Round Net Profit to 2 decimal places
    ROUND(SUM(NET_PROFIT), 2) AS Net_Profit,
    -- Calculate and round Profit Margin to 2 decimal places
    ROUND(SUM(NET_PROFIT / Revenue) * 100, 2) AS Profit_Margin
FROM 
    Net_Profit
GROUP BY 
    product_name
ORDER BY
    -- Order by Net Profit and Profit Margin in descending order
    Net_Profit DESC, Profit_Margin DESC
LIMIT 10;
```
###### Findings
![10 Product has highest profit margin and net profit](https://github.com/MoazWael2/ecommerce-food-app-analysis/assets/137816418/68ff5d1c-ff3c-4269-acd0-f560c4d6194a)

#### 6- Which products have the highest return rate?
###### SQL Query
###### Findings


