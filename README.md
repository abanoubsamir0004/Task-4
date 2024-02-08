# Part 1

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


# 3. Identifying customers with the highest average order value within a region:

## Original Query
```
SELECT customer_id, region , amount
FROM sales
```

## Solution 
```
WITH Avg_Order_Values AS (
    SELECT 
        
        customer_id,
        region,
        ROUND(AVG(amount), 2) AS customer_avg_per_region,
        DENSE_RANK() OVER (PARTITION BY region ORDER BY customer_avg_per_region DESC) AS rank_within_region
    FROM sales
    GROUP BY 
        customer_id, region
)
SELECT 
    customer_id,
    region,
    customer_avg_per_region
    
FROM Avg_Order_Values
WHERE rank_within_region = 1
ORDER BY customer_avg_per_region DESC;
```

# 4. inding products with the biggest year-over-year sales growth:
## Original Query
```
SELECT product_id,amount, sale_date
FROM sales;
```
## Solution 1: I calculated the products with the biggest year-over-year growth in sales for each year
```
WITH Yearly_Sales AS (
    SELECT 
        product_id,
        EXTRACT(YEAR FROM sale_date) AS sale_year,
        SUM(amount) AS total_sales,
        LAG(SUM(amount)) OVER (PARTITION BY product_id ORDER BY sale_year) AS prev_year_sales
        
    FROM 
        sales2
    GROUP BY 
        product_id, sale_year
), 
Yearly_Growth_with_ranking AS (
    SELECT 
        *,
        ROUND(((total_sales - prev_year_sales) / prev_year_sales) * 100, 2) AS yoy_growth,
        RANK() OVER (PARTITION BY sale_year ORDER BY yoy_growth DESC) AS year_rank

    FROM Yearly_Sales
    WHERE prev_year_sales IS NOT NULL
    ORDER BY sale_year
)
SELECT   
    product_id,
    sale_year,
    total_sales,
    prev_year_sales,
    yoy_growth
FROM Yearly_Growth_with_ranking
WHERE Year_rank=1
```
## Solution 2: I calculated the product with the biggest year-over-year growth sales.
```
WITH Yearly_Sales AS (
    SELECT 
        product_id,
        EXTRACT(YEAR FROM sale_date) AS sale_year,
        SUM(amount) AS total_sales,
        LAG(SUM(amount)) OVER (PARTITION BY product_id ORDER BY sale_year) AS prev_year_sales
        
    FROM 
        sales2
    GROUP BY 
        product_id, sale_year
), 
Yearly_Growth_with_ranking AS (
    SELECT 
        *,
        ROUND(((total_sales - prev_year_sales) / prev_year_sales) * 100, 2) AS yoy_growth,
        RANK() OVER ( ORDER BY yoy_growth DESC) AS growth_rank 

    FROM Yearly_Sales
    WHERE prev_year_sales IS NOT NULL
    ORDER BY sale_year
)
SELECT   
    product_id,
    sale_year,
    total_sales,
    prev_year_sales,
    yoy_growth
FROM Yearly_Growth_with_ranking
WHERE growth_rank=1
```

# Part 2 :
- Those queries have errors. Please identify the errors and write the correct queries.
- Here are some examples of queries with errors related to window functions:

## 


