# TASK 4 - Part 2

# Query 1 

## Original Query
```
SELECT 
Purchase_Transaction_ID, 
Purchase_DATE_ID,
Ship_Mode, 
Region,
Category,
Sub_Category,
Type,
Sales, 
Quantity,
Discount, 
Profit
FROM SAMPLE_SUPERSTORE_PURCHASE
WHERE 
Type IN ('Second Class - Consumer', 'Standard Class - Consumer','First Class - Consumer','Same Day - Consumer')
and Purchase_DATE_ID IN 
('2022-12-01',
'2022-11-01',
'2022-01-01' ,
'2022-10-01' ,
'2022-06-01' ,
'2022-04-01' ,
'2022-09-01' ,
'2022-02-01' ,
'2022-07-01' ,
'2022-05-01' ,
'2022-08-01' ,
'2022-03-01')
```

## Improved Query 1 
```
ALTER TABLE SAMPLE_SUPERSTORE_PURCHASE
ADD  shipment_type varchar(50),
     customer_type varchar(50),
     PURCHASE_DAY INT,
     PURCHASE_MONTH INT,
     PURCHASE_YEAR INT;

UPDATE SAMPLE_SUPERSTORE_PURCHASE
SET 
    shipment_type = sr.shipment_type,
    customer_type = sr.customer_type,
    PURCHASE_DAY = sr.PURCHASE_DAY,
    PURCHASE_MONTH = sr.PURCHASE_MONTH,
    PURCHASE_YEAR = sr.PURCHASE_YEAR
FROM (
    SELECT
        Purchase_Transaction_ID,
        SUBSTRING(Type, 1, POSITION('-' IN Type) - 1) AS shipment_type,
        SUBSTRING(Type, POSITION('-' IN Type) + 2) AS customer_type,
        DAY(PURCHASE_DATE_ID) as PURCHASE_DAY,
        MONTH(PURCHASE_DATE_ID) as PURCHASE_MONTH,
        YEAR(PURCHASE_DATE_ID) as PURCHASE_YEAR
    FROM
        SAMPLE_SUPERSTORE_PURCHASE
) AS sr
WHERE
    SAMPLE_SUPERSTORE_PURCHASE.Purchase_Transaction_ID = sr.Purchase_Transaction_ID;

SELECT *
FROM SAMPLE_SUPERSTORE_PURCHASE 
WHERE customer_type = 'Consumer' 
      AND PURCHASE_DAY = 01
      AND PURCHASE_YEAR = 2022;
```

## Explanation 1
 
- Updated Table Structure (Snowflake): When utilizing Snowflake directly, an ALTER TABLE statement is necessary to add columns for shipment_type, customer_type, PURCHASE_DAY, PURCHASE_MONTH, and PURCHASE_YEAR. These columns are then populated through a subsequent UPDATE statement, splitting relevant information from the 'Type' column and extracting day, month, and year components from the 'PURCHASE_DATE_ID' column. This update enhances the table's structure to accommodate more detailed analysis and reporting.
  
## Improved Query 2 
```
WITH SAMPLE_SUPERSTORE_PURCHASE_TRANSFORMED  AS (
    
    SELECT
        *,
        SUBSTRING(Type, 1, POSITION('-' IN Type) - 1) AS shipment_type,
        SUBSTRING(Type, POSITION('-' IN Type) + 2) AS customer_type,
        
        DAY(PURCHASE_DATE_ID) as PURCHASE_DAY,
        MONTH(PURCHASE_DATE_ID) as PURCHASE_MONTH,
        YEAR(PURCHASE_DATE_ID) as PURCHASE_YEAR
    FROM
        SAMPLE_SUPERSTORE_PURCHASE
) 

SELECT 
    *

FROM SAMPLE_SUPERSTORE_PURCHASE_TRANSFORMED
WHERE customer_type = 'Consumer' 
      AND PURCHASE_DAY = 01
      AND PURCHASE_YEAR = 2022;
```

## Explanation 2
 
- Alternative Approach with dbt (Snowflake): Alternatively, when leveraging dbt with Snowflake, a common table expression (CTE) named SAMPLE_SUPERSTORE_PURCHASE_TRANSFORMED is utilized to perform the column splitting and extraction operations. This CTE avoids the need for altering the table structure directly and allows for data transformation within the query itself. The subsequent SELECT statement then applies the desired filters to retrieve relevant data.
