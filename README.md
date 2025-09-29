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
