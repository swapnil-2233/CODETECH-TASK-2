# CODTECH-TASK-2
NAME-SWAPNIL BALAJI YEREWAD
TASK NAME-USE WINDOW FUNCTIONS, SUBQUERIES, AND CTES (COMMON TABLE EXPRESSIONS) FOR ADVANCED DATA ANALYSIS.DELIVERABLE: A REPORT GENERATED USING SQL QUERIES SHOWCASING TRENDS OR PATTERNS.
FEATURES-To complete the task of generating a report showcasing trends or patterns using advanced SQL queries with **window functions, subqueries, and Common Table Expressions (CTEs)**, you'll need to follow a structured approach. Below is an overview of how you can approach the analysis and create the SQL queries.

### 1. **Define the Problem and Data**
You should start by understanding the dataset you're working with and identifying the trends or patterns you want to showcase. For example:
- Sales trends over time
- Customer behavior patterns
- Product performance across different regions
- Employee performance or usage trends in a system

Assume that you are working with a **sales database** containing tables like:
- `sales (sale_id, product_id, customer_id, sale_date, quantity_sold, sale_amount)`
- `customers (customer_id, name, region)`
- `products (product_id, product_name, category)`

### 2. **Using Window Functions**

Window functions are great for calculating running totals, moving averages, ranking, and analyzing data within specific partitions (like regions or time periods).

**Example: Calculate the running total of sales for each product over time**

```sql
SELECT 
    product_id,
    sale_date,
    sale_amount,
    SUM(sale_amount) OVER (PARTITION BY product_id ORDER BY sale_date) AS running_total
FROM 
    sales
ORDER BY 
    product_id, sale_date;
```

In this query:
- `SUM(sale_amount) OVER (PARTITION BY product_id ORDER BY sale_date)` computes the running total of sales for each product.
- `PARTITION BY product_id` ensures that the running total is calculated separately for each product.
- `ORDER BY sale_date` orders the sales data by the sale date to calculate the running total correctly.

### 3. **Using Subqueries**

Subqueries can be used to retrieve intermediate results, which can be further analyzed or joined with other tables.

**Example: Get the total sales amount for each product along with the average sales amount per product**

```sql
SELECT 
    p.product_name,
    s.total_sales,
    (SELECT AVG(sale_amount) 
     FROM sales 
     WHERE product_id = p.product_id) AS average_sales_per_product
FROM 
    products p
JOIN 
    (SELECT product_id, SUM(sale_amount) AS total_sales 
     FROM sales 
     GROUP BY product_id) s
ON p.product_id = s.product_id;
```

In this query:
- The subquery `(SELECT product_id, SUM(sale_amount) AS total_sales FROM sales GROUP BY product_id)` calculates the total sales per product.
- The outer query uses this result and calculates the average sales per product using another subquery.

### 4. **Using Common Table Expressions (CTEs)**

CTEs are useful for breaking down complex queries into manageable parts and can also help improve readability.

**Example: Identify the top 5 products by total sales for each region using a CTE**

```sql
WITH ProductSales AS (
    SELECT 
        s.product_id,
        p.product_name,
        c.region,
        SUM(s.sale_amount) AS total_sales
    FROM 
        sales s
    JOIN 
        products p ON s.product_id = p.product_id
    JOIN 
        customers c ON s.customer_id = c.customer_id
    GROUP BY 
        s.product_id, p.product_name, c.region
)
SELECT 
    region,
    product_name,
    total_sales
FROM 
    ProductSales
WHERE 
    (region, total_sales) IN (
        SELECT 
            region, MAX(total_sales)
        FROM 
            ProductSales
        GROUP BY 
            region
    )
ORDER BY 
    region, total_sales DESC
LIMIT 5;
```

In this query:
- The CTE `ProductSales` calculates the total sales for each product in each region.
- The outer query identifies the top 5 products by total sales within each region.

### 5. **Putting It All Together: Trends and Patterns Report**

Here is an example of how you might summarize sales trends and patterns across different time periods and regions.

**Example: Quarterly sales trends with region breakdown**

```sql
WITH SalesByQuarter AS (
    SELECT
        EXTRACT(YEAR FROM sale_date) AS year,
        EXTRACT(QUARTER FROM sale_date) AS quarter,
        c.region,
        SUM(s.sale_amount) AS total_sales
    FROM 
        sales s
    JOIN 
        customers c ON s.customer_id = c.customer_id
    GROUP BY
        year, quarter, c.region
)
SELECT
    year,
    quarter,
    region,
    total_sales,
    LAG(total_sales) OVER (PARTITION BY region ORDER BY year, quarter) AS previous_quarter_sales,
    (total_sales - LAG(total_sales) OVER (PARTITION BY region ORDER BY year, quarter)) AS sales_growth
FROM
    SalesByQuarter
ORDER BY
    region, year, quarter;
```

In this query:
- The CTE `SalesByQuarter` calculates the total sales for each region by quarter.
- The window function `LAG(total_sales)` is used to compare current quarter sales to previous quarter sales.
- This allows you to track sales growth or decline across quarters.


