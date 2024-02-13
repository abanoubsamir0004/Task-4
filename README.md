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

## Improved Query
```
SELECT *
FROM SAMPLE_SUPERSTORE_PURCHASE 
WHERE Type LIKE '%Consumer%'
      AND (YEAR(PURCHASE_DATE_ID)=2022 AND DAY(Purchase_DATE_ID) = 01);
```

## Explanation 

- Improved Query Clarity: Enhances readability by utilizing a wildcard operator (%) in the LIKE clause to encompass all consumer-related transaction types, simplifying the query structure.

- Optimized Date Condition: Refines the condition for the purchase date using the YEAR() and DAY() functions, specifically targeting transactions occurring on the 1st day of any month in 2022, thereby streamlining query logic and reducing redundancy. 
