### 1. Top 10 Highest Revenue Generating Products
```sql
SELECT product_id, SUM(sale_price) AS sales
FROM df_orders
GROUP BY product_id
ORDER BY sales DESC
LIMIT 10;


### 2. Top 5 Highest Selling Products in Each Region
WITH ranked_products AS (
  SELECT region, product_id, SUM(sale_price) AS total_sales,
         RANK() OVER (PARTITION BY region ORDER BY SUM(sale_price) DESC) AS rank
  FROM df_orders
  GROUP BY region, product_id
)
SELECT * FROM ranked_products WHERE rank <= 5;

### 3. Month-over-Month Growth Comparison (2022 vs 2023)
WITH cte AS (
  SELECT YEAR(order_date) AS order_year, MONTH(order_date) AS order_month, 
         SUM(sale_price) AS sales
  FROM df_orders
  WHERE YEAR(order_date) IN (2022, 2023)
  GROUP BY YEAR(order_date), MONTH(order_date)
)
SELECT order_month,
       SUM(CASE WHEN order_year = 2022 THEN sales ELSE 0 END) AS sales_2022,
       SUM(CASE WHEN order_year = 2023 THEN sales ELSE 0 END) AS sales_2023
FROM cte
GROUP BY order_month
ORDER BY order_month;

### 4. Highest Profit Growth by Sub-Category in 2023 vs 2022
WITH cte AS (
  SELECT sub_category, YEAR(order_date) AS order_year, 
         SUM(profit) AS profit
  FROM df_orders
  WHERE YEAR(order_date) IN (2022, 2023)
  GROUP BY sub_category, YEAR(order_date)
),
growth_calc AS (
  SELECT sub_category,
         SUM(CASE WHEN order_year = 2022 THEN profit ELSE 0 END) AS profit_2022,
         SUM(CASE WHEN order_year = 2023 THEN profit ELSE 0 END) AS profit_2023
  FROM cte
  GROUP BY sub_category
)
SELECT *, (profit_2023 - profit_2022) AS profit_growth
FROM growth_calc
ORDER BY profit_growth DESC
LIMIT 1;
