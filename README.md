## 📊 Finding Top Selling Products per Category (BigQuery SQL)

### Business Problem
The marketing team wants to launch a "Best Sellers of 2025" campaign. They need to identify which specific products generated the most revenue within *every distinct category*, filtering out any cancelled or returned orders.

### Technical Deep Dive
* **CTEs (Common Table Expressions):** Used to modularize the aggregation logic (`sale_product_2025`) from the ranking logic (`best_selling_products`), making the code clean and maintainable.
* **Window Functions (`DENSE_RANK`):** Leveraged `DENSE_RANK() OVER (PARTITION BY ... ORDER BY ...)` to dynamically rank products within their respective categories. If two products tied for revenue, they received the same rank without skipping subsequent numbers.
* **BigQuery Optimizations:** Utilized native string formatting `FORMAT_TIMESTAMP` to handle temporal partitions efficiently.

### Key Skills Demonstrated
* **Data Warehouse:** Google BigQuery
* **Data Transformation:** Multi-level CTE nesting, Advanced Analytical Window Functions, Data manipulation and aggregation, Data type filtering and formatting
* **BI & Visualization:** Power BI Desktop

## 💻 How to Run the SQL Script
The optimized database query code used to drive this entire report can be found below:
```sql
WITH sale_product_2025 AS (
    SELECT 
        p.category AS product_category,
        p.name AS product_name,
        p.brand AS brand,
        SUM(oi.sale_price) AS total_revenue
    FROM `warehouse2016.products` AS p 
    INNER JOIN `warehouse2016.order_items` AS oi 
        ON oi.product_id = p.id  
    WHERE FORMAT_TIMESTAMP('%Y', oi.created_at) = '2025'
      AND oi.status = 'Shipped'
    GROUP BY 
        p.category,
        p.name,
        p.brand 
),

best_selling_products AS (
    SELECT 
        sp25.product_category,
        sp25.product_name,
        sp25.brand,
        ROUND(sp25.total_revenue, 2) AS revenue,
        DENSE_RANK() OVER (
            PARTITION BY sp25.product_category 
            ORDER BY sp25.total_revenue DESC
        ) AS rank_in_category
    FROM sale_product_2025 AS sp25
)

SELECT 
    product_category,
    rank_in_category,
    product_name,
    brand,
    revenue
FROM best_selling_products
ORDER BY 
    product_category ASC, 
    rank_in_category ASC;
