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
## Explanation:
- Handling Tie Scenarios: DENSE_RANK() ensures equitable ranking for products with identical sales quantities, preventing the omission of equally performing items.
- Top N Products Retrieval: The solution employs a WHERE clause, to filter and retrieve the top N products based on their sales quantities within the specified period.

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
## Explanation 1:

- Cumulative Sales Calculation: The solution utilizes the SUM() window function with PARTITION BY to compute cumulative sales for each product category, facilitating the understanding of revenue trends over time.
- Window Function Implementation: By specifying ROWS UNBOUNDED PRECEDING, it ensures accurate aggregation of sales amounts, providing a running total for each product category based on the chronological order of sales dates.

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
## Explanation 2:
- Cumulative Sales Calculation: This solution calculates cumulative sales for each product category by employing the SUM() window function within the PARTITION BY clause, ensuring accurate aggregation of sales amounts over time.

- Window Function Implementation: By utilizing SUM(total_sales) OVER (PARTITION BY product_category ORDER BY sale_date), it generates a running total of sales amounts within each product category, aiding in the analysis of revenue trends over successive sale dates.

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
## Explanation:
- High-AVG Customer Identification: The solution calculates average order values per customer within regions, assigning ranks using DENSE_RANK() to pinpoint customers with the highest averages within each region.

- Window Function Application: Employing a CTE, it partitions data by region, allowing accurate ranking of customers by their average order values, facilitating targeted analysis of high-spending customers within specific geographical areas.

# 4. inding products with the biggest year-over-year sales growth:
## Original Query
```
SELECT product_id,amount, sale_date
FROM sales;
```

## Solution 1: I calculated all products year-over-year growth in sales for each year
```
WITH Yearl_Sales AS (

    SELECT 
    
        product_id,
        EXTRACT(YEAR FROM sale_date) AS sale_year,
        SUM(amount) AS total_sales,
        LAG(SUM(amount)) OVER (PARTITION BY product_id ORDER BY sale_year) AS prev_year_sales
        
    FROM sales2
    GROUP BY 
        product_id, sale_year
)

SELECT 
    product_id,
    sale_year,
    total_sales,
    prev_year_sales,
    ROUND(((total_sales - prev_year_sales) / prev_year_sales) * 100, 2) as yoy_growth

FROM Yearl_Sales
WHERE prev_year_sales IS NOT NULL
ORDER BY 
    product_id, sale_year;
```
## Explanation 1:

- Sales Growth Analysis: The solution calculates year-over-year sales growth for each product by comparing total sales amounts between consecutive years, facilitating the identification of products with significant growth trends.

- Window Function Utilization: Employing LAG() within a CTE, it captures previous year sales data, enabling precise calculation of year-over-year growth rates and pinpointing products experiencing notable increases in sales.

## Solution 2: I calculated the products with the biggest year-over-year growth in sales for each year
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
## Explanation 2:

- Top Yearly Growth Identification: This solution determines products with the most significant year-over-year sales growth for each year by ranking the calculated growth rates within each year's data. It enables the precise identification of products experiencing the highest growth trends annually.

- Ranking and Selection: Utilizing RANK() within a CTE, it ranks products based on their year-over-year growth rates, facilitating the extraction of products with the highest growth rates by selecting those with a rank of 1 for each respective year.

## Solution 3: I calculated the product with the biggest year-over-year growth sales.
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
## Explanation 3:

- Max Year-over-Year Growth Product Identification: This solution identifies the product with the largest year-over-year sales growth by ranking all products based on their calculated growth rates. It offers a comprehensive approach to pinpointing the single product experiencing the most substantial growth in sales across all years analyzed.

- Overall Growth Ranking: By employing RANK() without partitioning, the solution ranks products globally based on their year-over-year growth rates, ensuring that the product with the highest growth is selected regardless of the specific year.

# Part 2 :
- Those queries have errors. Please identify the errors and write the correct queries.
- Here are some examples of queries with errors related to window functions:

## 


