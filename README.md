# Sales Data - Dashboard

### Dashboard Link : https://app.powerbi.com/groups/me/reports/3de72ac1-fbf2-4825-a335-e15f67f685fa/82a2685beb940e1db3b5?experience=power-bi

Sales Analysis Project

A complete end-to-end Data Analytics Project using SQL + Power BI. 

This project helped me to get more undertanding about the SQL queris and more hands on practice.

### Project Overview

The objective of this project is to demonstrate real-world data cleaning, transformation, and analysis skills using SQL and Power BI, targeting a mid-level Data Analyst role.

In this project we have cleaned the data in the way such it is get ready for the dashboard creation.

Somewhere we had NULL values, but we can't keeps them as it is so I have also creared the Default values which are best fit for them

We had mltiple errors in the data for example different kind of date values and charectors

* Negative values in the Quantity_raw column and Price_raw Column

* Upper Case and Lower Case issue in Status_raw and Payment_raw

* Blank places and missing values

* Text values in Numeric Column

* Mixed Data Type

We have used below SQL codes to clean those errors so that we can use this data to create the dashboard.


### Below are the exact SQL queries used to clean the dataset.

1) Deleting the duplicates

         with cte as (
         select ROW_NUMBER() over (partition by order_id,   
         customer_id, 
         product order by Order_date_raw) as rn
         from NEW_SALES) 

        DELETE FROM cte where rn>1;

2) Converting into date type

       UPDATE NEW_SALES
       SET ORDER_DATE_RAW =
       (CASE
       WHEN ISDATE(ORDER_DATE_RAW) = 1 THEN TRY_CONVERT(DATE,  
       ORDER_DATE_RAW)
       WHEN ORDER_DATE_RAW LIKE '%/%/%' THEN TRY_CONVERT(DATE,   
       ORDER_DATE_RAW, 103)
       WHEN ORDER_DATE_RAW LIKE '%-%-$' THEN TRY_CONVERT (DATE, 
       ORDER_DATE_RAW, 110)
       ELSE NULL
       END);

3) Updating the null values with previous date

       with ordered as (
       select *, LAG(order_date_raw) OVER (ORDER BY order_id)   
       AS prev_date FROM NEW_SALES)

       update ordered
       set order_date_raw = prev_date
       WHERE order_date_raw is null and prev_date is not null;

4)  Removing the errors from the QUANTITY_RAW

        UPDATE NEW_SALES
        SET QUANTITY_RAW =
        CASE
        WHEN TRY_CAST(QUANTITY_RAW AS INT) IS NOT NULL AND
	    TRY_CAST(QUANTITY_RAW AS int) BETWEEN 1 AND 50
	    THEN CAST(QUANTITY_RAW AS int)
	    WHEN QUANTITY_RAW LIKE 'five' then 5
	    ELSE null
        END;

        UPDATE NEW_SALES
        SET QUANTITY_RAW = 
        CASE WHEN QUANTITY_RAW LIKE '-%' THEN ABS(QUANTITY_RAW)
        ELSE NULL
        END;

5) UPDATING QUNTITY_RAW wherever is null

       UPDATE NEW_SALES
       SET QUANTITY_RAW = 1
       WHERE QUANTITY_RAW IS NULL;
 
6) updating the NEW_SALES into Float

       UPDATE NEW_SALES
       SET price_raw = 
       CASE
       WHEN TRY_CAST(price_raw AS float) IS NOT null THEN CAST    
       (price_raw AS FLOAT)
       WHEN price_raw LIKE '₹%'	THEN TRY_CAST(REPLACE  
       (price_raw,   '₹%','') AS float)
       WHEN price_raw LIKE 'USD%' THEN TRY_CAST(REPLACE
       (PRICE_RAW, 'USD ','') AS float) * 90.56
       END;


       UPDATE NEW_SALES
       SET price_raw = 
       CASE WHEN price_raw < 0 then try_cast(ABS(price_raw) AS    
       float) end;

7) Coneverting QUANTITY_RAW into Float value

       alter table new_sales
       alter column QUANTITY_RAW int;

8) Schema of the Table

       SELECT * FROM INFORMATION_SCHEMA.COLUMNS
       WHERE TABLE_NAME LIKE 'NEW_SALES';
    
9) Updating the Price Raw wherever is null

       WITH CTE AS (
       SELECT SUM(PRICE_RAW) * 1.0 / SUM(QUANTITY_RAW) AS  
       AVG_PRICE
       FROM NEW_SALES
       WHERE PRICE_RAW IS NOT NULL)

       UPDATE n
       SET PRICE_RAW = QUANTITY_RAW * C.AVG_PRICE
       FROM NEW_SALES n
       CROSS JOIN CTE C
       WHERE n.PRICE_RAW IS NULL;

10) Rouding of the Null into 2 Decimal value

        UPDATE new_sales
        SET price_raw = ROUND(price_raw, 2);

11) Updating the Status_raw

        UPDATE NEW_SALES
        set status_raw = 
        CASE
        WHEN LOWER(status_raw) LIKE '%completed%' Then 'Completed'
        WHEN LOWER(status_raw) LIKE '%pending%' Then 'Pending'
        WHEN LOWER(status_raw) LIKE '%cancel%' then 'Cancelled'
        WHEN LOWER(status_raw) LIKE '%return%' then 'Returned'
        Else 'Unknown'
        End;

12) Updating the Payment_raw

        UPDATE NEW_SALES
        SET payment_raw =
        CASE 
        WHEN LOWER(payment_raw) = 'upi' THEN 'UPI'
        WHEN LOWER(payment_raw) = 'cash' THEN 'Cash'
        WHEN LOWER(payment_raw) = 'card' THEN 'Card'
        WHEN LOWER(payment_raw) = 'not paid' THEN 'Unknown'
        ELSE payment_raw
        END;

13) Storing all Non-Null values into new tabel "Sales_cleaned"

        SELECT 
        order_id,
        customer_id,
        order_date_raw AS order_date,
        product,
        quantity_raw AS quantity,
        price_raw AS price,
        status_raw AS status,
        payment_raw AS payment,
        (quantity_raw * price_raw) AS total_sales
        INTO sales_cleaned
        FROM new_sales
        WHERE quantity_raw IS NOT NULL
        AND price_raw IS NOT NULL
        AND order_date_raw IS NOT NULL;

### Columns in Dataset

order_id = Unique order identifier (contains duplicates)

customer_id	= Customer identifier

order_date_raw = Order date in multiple formats & invalid values

product = Product name

quantity_raw = Quantity with text, nulls, negatives & outliers

price_raw = Price with currency symbols, text & invalid values

status_raw = Order status with case issues & unknown values

payment_raw	= Payment mode with inconsistent formatting


### SQL Cleaning & Transformation Approach

All data cleaning was done only using SQL (no Power BI data cleaning).

#### SQL Concepts Covered

TRY_CAST, TRY_CONVERT

ISDATE()

CASE WHEN

CTE (WITH)

ROW_NUMBER() for de-duplication

UPDATE statements

String cleaning (REPLACE, LOWER, UPPER, TRIM)

Conditional logic

Outlier handling


## Business KPIs Created

The following KPIs were calculated after data cleaning:

Total Revenue

Total Orders

Average Order Value (AOV)

Total Customers

Revenue by Product

Orders by Payment Method

Order Status Distribution

Customer Segmentation (High / Medium / Low Value)

## Power BI Dashboard (3 Pages)

![Sales Dashboard 1](https://raw.githubusercontent.com/rawdyt/Sales-Analysis/main/Dashboard%20Page%201.png)

![Sales Dashboard 2](https://raw.githubusercontent.com/rawdyt/Sales-Analysis/main/Dashboard%20Page%202.png)

![Sales Dashboard 3](https://raw.githubusercontent.com/rawdyt/Sales-Analysis/main/Dashboard%20Page%203.png)


### Key Learnings

Handling messy real-world data using SQL

Preserving data integrity during cleaning

Translating business problems into KPIs

Building story-driven Power BI dashboards

Explaining data decisions clearly for stakeholders

