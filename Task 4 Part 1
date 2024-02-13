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

# 4. Finding products with the biggest year-over-year sales growth:
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
## Explanation 2:

- Max Year-over-Year Growth Product Identification: This solution identifies the product with the largest year-over-year sales growth by ranking all products based on their calculated growth rates. It offers a comprehensive approach to pinpointing the single product experiencing the most substantial growth in sales across all years analyzed.

- Overall Growth Ranking: By employing RANK() without partitioning, the solution ranks products globally based on their year-over-year growth rates, ensuring that the product with the highest growth is selected regardless of the specific year.


## Solution 3: I calculated the products with the biggest year-over-year growth in sales for each year
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
## Explanation 3:

- Top Yearly Growth Identification: This solution determines products with the most significant year-over-year sales growth for each year by ranking the calculated growth rates within each year's data. It enables the precise identification of products experiencing the highest growth trends annually.

- Ranking and Selection: Utilizing RANK() within a CTE, it ranks products based on their year-over-year growth rates, facilitating the extraction of products with the highest growth rates by selecting those with a rank of 1 for each respective year.


# Part 2 :
- Those queries have errors. Please identify the errors and write the correct queries.
- Here are some examples of queries with errors related to window functions:

# Query 1

## Orignal query

```
SELECT *, RANK() OVER (PARTITION BY customer_id) AS rank
FROM sales
ORDER BY rank DESC;
```

## Corrected query

```
SELECT

    *,
    SUM(amount) as total_amount,
    DENSE_RANK() OVER (PARTITION BY customer_id order by total_amount DESC) AS rank

FROM sales
GROUP BY CUSTOMER_ID
ORDER BY
    customer_id, rank DESC;
```

## Explanation: 

- The original query lacked an "ORDER BY" clause to properly operate the RANK() window function, potentially resulting in misleading rankings.
  
- Additionally, it used the RANK() function without considering potential ties in the ranking, which could lead to unexpected results.
  
- The corrected query addresses these issues by:
    - Adding an "ORDER BY" clause to ensure proper operation of the ranking window function.
    - Replacing RANK() with DENSE_RANK() to handle potential ties and ensure a consistent ranking.
    - Grouping the data by customer_id to calculate the total amount for each customer.

# Query 2

## Orignal query

```
SELECT *, SUM(amount) OVER (ORDER BY sale_date) AS running_total
FROM sales;
```

## Corrected query

```
SELECT

    customer_id,
    sale_date, 
    sum(amount) as day_sales,
    SUM(day_sales) OVER (PARTITION BY customer_id ORDER BY sale_date) AS running_total

FROM sales
GROUP BY
    customer_id, sale_date
ORDER BY
    customer_id, sale_date
```

## Explanation: 

- The original query attempted to calculate a running total of the "amount" column ordered by "sale_date" without properly partitioning by customer. This resulted in incorrect running totals across different customers.
  
- The corrected query first calculates the daily sales amount for each customer. Then, it computes the running total of sales for each customer by properly partitioning the data by customer_id and ordering by sale_date.

- Finally, it groups the results by customer_id and sale_date, ensuring accurate computation of running totals for each customer over time.
  
# Query 3

## Orignal query

```
SELECT *,AVG(amount) OVER (PARTITION BY customer_id) AS avg_amount,
       COUNT(*) 
FROM sales;
```

## Corrected query

```
SELECT DISTINCT

    customer_id,
    AVG(amount) OVER (PARTITION BY customer_id) AS avg_amount,
    COUNT(*) OVER (PARTITION BY customer_id) AS customer_count
    
FROM sales;
```

## Explanation: 

- The original query calculated the count the total number of rows but didn't partition by customer_id, resulting in a count across all customers.

- The corrected query ensures partitioning the data by customer_id for both average amount calculation and customer count, providing insights into each customer's average amount and the number of transactions they made.

# Query 4

## Orignal query

```
SELECT 
    *,
    ROW_NUMBER() OVER (sale_date ORDER BY sale_date) AS row_num,
    ROW_NUMBER() OVER (amount ORDER BY amount) AS another_row_num
FROM sales;
```

## Corrected query: 

```
SELECT 
    *,
    DENSE_RANK() OVER (ORDER BY sale_date) AS row_num,
    DENSE_RANK() OVER (ORDER BY amount) AS another_row_num
FROM sales
ORDER BY
    row_num, another_row_num
```

## Explanation: 

- The original query attempted to assign row numbers based on the values of "sale_date" and "amount" independently using the ROW_NUMBER() function.
  
- However, ROW_NUMBER() does not allow specifying the partitioning column explicitly, leading to incorrect results where row numbers restart for each distinct value of sale_date or amount.
  
- The corrected query addresses this issue by using the DENSE_RANK() function instead of ROW_NUMBER().
  
- By ordering the rows by sale_date and amount respectively, DENSE_RANK() assigns unique ranks to each row within each partition based on these criteria.
  
- Finally, the query orders the results by the assigned row numbers for clarity and consistency.
