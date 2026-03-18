Superstore Sales Analysis — SQL Project

Tools: SQL Server (T-SQL)
Dataset: Superstore Time Series (Kaggle)
Author: Tracy Omogbai

Business Problem

Retail decisions are often driven by short-term spikes rather than real trends. This leads to poor inventory planning, ineffective discounting, and missed revenue opportunities.

Objective

Analyze 4 years of transactional data (2014–2017) to:

Validate data quality

Compare day-to-day performance

Identify top/bottom products

Track monthly sales trends

Smooth volatility using moving averages

Data Preparation

Validated dataset before analysis:

No null values

No true duplicates (multi-item orders confirmed)

No whitespace inconsistencies

Correct data types across all columns

Key note:
Discount is a rate, already applied to Sales:
Sales = Price × (1 - Discount)

Analysis & SQL
1. Day-to-Day Comparison (LEAD / LAG)
WITH daily_sales AS
(
    SELECT
        Order_Date,
        SUM(Quantity) AS TotalQty,
        ROUND(SUM(Discount), 2) AS TotalDiscount,
        ROUND(SUM(Sales), 2) AS TotalSales
    FROM Superstore_Staging
    GROUP BY Order_Date
)
SELECT
    Order_Date,
    TotalQty,
    LEAD(TotalQty) OVER (ORDER BY Order_Date) AS Qty_Next,
    LAG(TotalQty) OVER (ORDER BY Order_Date) AS Qty_Prev,
    TotalDiscount,
    LEAD(TotalDiscount) OVER (ORDER BY Order_Date) AS Discount_Next,
    LAG(TotalDiscount) OVER (ORDER BY Order_Date) AS Discount_Prev,
    TotalSales,
    LEAD(TotalSales) OVER (ORDER BY Order_Date) AS Sales_Next,
    LAG(TotalSales) OVER (ORDER BY Order_Date) AS Sales_Prev
FROM daily_sales
ORDER BY Order_Date
2. Sales Ranking
WITH daily_rank AS
(
    SELECT
        Order_Date,
        ROUND(SUM(Sales), 2) AS TotalSales
    FROM Superstore_Staging
    GROUP BY Order_Date
)
SELECT
    Order_Date,
    TotalSales,
    RANK() OVER (ORDER BY TotalSales DESC) AS Sales_Rank
FROM daily_rank
3. Monthly Sales Trends
SELECT
    YEAR(Order_Date) AS Year,
    DATENAME(MONTH, Order_Date) AS Month_Name,
    AVG(Sales) AS Avg_Sales
FROM Superstore_Staging
GROUP BY YEAR(Order_Date), MONTH(Order_Date), DATENAME(MONTH, Order_Date)
ORDER BY YEAR(Order_Date), MONTH(Order_Date)
4. Top & Bottom Products Per Month
WITH monthly_products AS
(
    SELECT
        YEAR(Order_Date) AS Year,
        MONTH(Order_Date) AS Month_Num,
        DATENAME(MONTH, Order_Date) AS Month,
        Category,
        Sub_Category,
        Product_Name,
        SUM(Quantity) AS Total_QTY,
        ROUND(SUM(Sales), 2) AS Total_Sales
    FROM Superstore_Staging
    GROUP BY YEAR(Order_Date), MONTH(Order_Date), DATENAME(MONTH, Order_Date),
             Category, Sub_Category, Product_Name
),
ranked AS
(
    SELECT *,
        ROW_NUMBER() OVER (PARTITION BY Year, Month_Num ORDER BY Total_Sales DESC) AS RN_Top,
        ROW_NUMBER() OVER (PARTITION BY Year, Month_Num ORDER BY Total_Sales ASC) AS RN_Bottom
    FROM monthly_products
)
SELECT *
FROM ranked
WHERE RN_Top = 1 OR RN_Bottom = 1
ORDER BY Year, Month_Num
5. 7-Day Moving Average (Trend Smoothing)
WITH daily_sales AS
(
    SELECT
        Order_Date,
        ROUND(SUM(Sales), 2) AS DailySales
    FROM Superstore_Staging
    GROUP BY Order_Date
)
SELECT
    Order_Date,
    DailySales,
    CASE
        WHEN ROW_NUMBER() OVER (ORDER BY Order_Date) >= 7
        THEN ROUND(AVG(DailySales) OVER (
             ORDER BY Order_Date
             ROWS BETWEEN 6 PRECEDING AND CURRENT ROW), 2)
        ELSE NULL
    END AS Moving_Avg_7Day
FROM daily_sales
ORDER BY Order_Date
Key Insights

Sales increased from 2014 → 2016, then declined in 2017

Revenue is highly volatile day-to-day

Moving average reveals spike-driven demand, not steady growth

Technology (Copiers, Phones) drives top revenue (high price, not volume)

Monthly performance varies significantly → seasonality + large orders

Recommendation

Do not rely on daily sales spikes for decision-making.
Focus on:

Trend-based forecasting (moving averages)

Monitoring high-value product categories

Investigating 2017 decline (pricing, discounts, demand shifts)

What This Project Demonstrates

Strong SQL fundamentals (CTEs, window functions, aggregation)

Data validation and cleaning

Analytical thinking → turning raw data into business insight

Ability to communicate findings clearly and concisely
