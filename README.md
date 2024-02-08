# Part1

## Dataset:

- Table: sales
- Columns: product_id, customer_id,,product_category, sale_date, amount, quantity


# 1.  Identify the most popular products in terms of sales quantity during a specific timeframe & Finding top N products sold within a specific dates:

## Original Query
```
SELECT *
FROM sales
WHERE sale_date BETWEEN '2023-01-01' AND '2023-02-07'
ORDER BY quantity DESC
LIMIT 10;
```
## Solution
```
WITH ranked_products AS (

    SELECT 
        product_id,
       SUM(quantity) AS total_quantity,
       DENSE_RANK() OVER (ORDER BY SUM(quantity) DESC) AS Product_Sold_Rank
       
    FROM sales
    WHERE sale_date BETWEEN '2023-01-01' AND '2023-02-07'
    GROUP BY product_id
)

SELECT 

    product_id,
    total_quantity,
    Product_Sold_Rank

FROM ranked_products
WHERE Product_Sold_Rank <= 10;
```

# 2.  Calculating cumulative sales by product category.

## Original Query
```
SELECT product_category, total_sales
FROM sales
ORDER BY product_category, sale_date
```
## Solution 1
```
SELECT 

    product_category,
    sale_date,
    amount AS total_sales,
    SUM(amount) OVER (PARTITION BY product_category ORDER BY sale_date ROWS UNBOUNDED PRECEDING) AS cumulative_sales
    
FROM sales
ORDER BY
  product_category, sale_date;
```

## Solution 2
```
SELECT

   product_category,
   sale_date,
   SUM(amount) AS total_sales,
   SUM(total_sales) OVER (PARTITION BY product_category ORDER BY sale_date) AS cumulative_sales
   
FROM sales
GROUP BY
  product_category, sale_date
ORDER BY
  product_category, sale_date;
```



