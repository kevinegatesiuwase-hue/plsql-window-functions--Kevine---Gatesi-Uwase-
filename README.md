# plsql-window-functions--Kevine---Gatesi-Uwase-
## step 4: Window Functions implementation
### ranking
SELECT
  r.region_name,
  TO_CHAR(t.sale_date, 'YYYY') AS sales_year,
  'Q' || TO_CHAR(t.sale_date, 'Q') AS sales_quarter,
  p.product_name,
  SUM(t.amount) AS total_revenue,
  RANK() OVER (
    PARTITION BY r.region_name, TO_CHAR(t.sale_date, 'YYYY'), TO_CHAR(t.sale_date, 'Q')
    ORDER BY SUM(t.amount) DESC
  ) AS revenue_rank
FROM transactions t
JOIN customers c ON t.customer_id = c.customer_id
JOIN regions r ON c.region_id = r.region_id
JOIN products p ON t.product_id = p.product_id
GROUP BY r.region_name, TO_CHAR(t.sale_date, 'YYYY'), TO_CHAR(t.sale_date, 'Q'), p.product_name
ORDER BY r.region_name, sales_year, sales_quarter, revenue_rank;
<img width="1920" height="1080" alt="Screenshot (39)" src="https://github.com/user-attachments/assets/ce6914f5-0506-4c3f-83f4-bfc6ee119cd7" />
### aggregate 

WITH monthly_sales AS (
  SELECT
    TRUNC(sale_date, 'MM') AS month_start,
    SUM(amount) AS monthly_revenue
  FROM transactions
  GROUP BY TRUNC(sale_date, 'MM')
)
SELECT
  month_start,
  monthly_revenue,
  SUM(monthly_revenue) OVER (
    ORDER BY month_start ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS running_total
FROM monthly_sales
ORDER BY month_start;

<img width="1920" height="1080" alt="Screenshot (40)" src="https://github.com/user-attachments/assets/44215f8e-8c50-4397-8986-14350cd55761" />
### navigation  

WITH monthly_sales AS (
  SELECT
    TRUNC(sale_date, 'MM') AS month_start,
    SUM(amount) AS monthly_revenue
  FROM transactions
  GROUP BY TRUNC(sale_date, 'MM')
)
SELECT
  month_start,
  monthly_revenue,
  LAG(monthly_revenue) OVER (ORDER BY month_start) AS prev_month,
  ROUND(
    (monthly_revenue - LAG(monthly_revenue) OVER (ORDER BY month_start)) 
    / NULLIF(LAG(monthly_revenue) OVER (ORDER BY month_start), 0) * 100, 
    2
  ) AS growth_percent
FROM monthly_sales
ORDER BY month_start;
<img width="1920" height="1080" alt="Screenshot (41)" src="https://github.com/user-attachments/assets/f843f75d-f905-44b1-9932-b11284545036" />
### Distribution
WITH customer_revenue AS (
  SELECT
    c.customer_id,
    c.cust_name,
    SUM(t.amount) AS total_revenue
  FROM customers c
  LEFT JOIN transactions t ON c.customer_id = t.customer_id
  GROUP BY c.customer_id, c.cust_name
)
SELECT
  customer_id,
  cust_name,
  total_revenue,
  NTILE(4) OVER (ORDER BY total_revenue DESC) AS revenue_quartile,
  CUME_DIST() OVER (ORDER BY total_revenue DESC) AS cumulative_distribution
FROM customer_revenue
ORDER BY total_revenue DESC;


<img width="1920" height="1080" alt="Screenshot (43)" src="https://github.com/user-attachments/assets/72a9288b-eaf5-485d-a1c3-7fdbe94386e4" />
### moving  average
WITH monthly_sales AS (
  SELECT
    TRUNC(sale_date, 'MM') AS month_start,
    SUM(amount) AS monthly_revenue
  FROM transactions
  GROUP BY TRUNC(sale_date, 'MM')
)
SELECT
  month_start,
  monthly_revenue,
  ROUND(
    AVG(monthly_revenue) OVER (
      ORDER BY month_start 
      ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ), 
    2
  ) AS moving_avg_3_months
FROM monthly_sales
ORDER BY month_start;
<img width="1920" height="1080" alt="Screenshot (45)" src="https://github.com/user-attachments/assets/870d0394-08c7-4a35-994c-2aed759eda5b" />
