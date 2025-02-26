### Retail Data Analysis

### Overview
This project analyzes retail order data obtained from Kaggle. The dataset is processed using Python and SQL to extract key insights, such as top-selling products, revenue growth trends, and category-wise sales performance. The results provide valuable business intelligence to enhance decision-making.

### Data Processing
The dataset is downloaded from Kaggle, preprocessed in Python using pandas, and then loaded into a SQL Server database for analysis. The following steps are performed:

1. **Data Extraction**: The dataset is extracted from a compressed file.
2. **Data Cleaning**:
   - Null values are handled by replacing `'Not Available'` and `'unknown'` with `NaN`.
   - Column names are standardized (converted to lowercase and spaces replaced with underscores).
3. **Feature Engineering**:
   - New columns are derived, including `profit`, calculated as `sale_price - cost_price`.
   - `order_date` is converted to a datetime format.
4. **Data Storage**:
   - Unnecessary columns (`list_price`, `cost_price`, and `discount_percent`) are dropped.
   - The cleaned data is loaded into a SQL Server database (`df_orders` table) using SQLAlchemy.

### SQL Queries
The following SQL queries are used to extract key insights from the dataset:

##### Top 10 Highest Revenue-Generating Products
```sql
SELECT TOP 10 product_id, SUM(sale_price) AS sales
FROM df_orders
GROUP BY product_id
ORDER BY sales DESC;
````

##### Top 5 Highest Selling Products in Each Region
```sql
WITH cte AS (
    SELECT region, product_id, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY region, product_id
)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER(PARTITION BY region ORDER BY sales DESC) AS rn
    FROM cte
) A
WHERE rn <= 5;
````

##### Month-over-Month Sales Growth for 2022 and 2023
```sql
WITH cte AS (
    SELECT YEAR(order_date) AS order_year, MONTH(order_date) AS order_month, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY YEAR(order_date), MONTH(order_date)
)
SELECT order_month,
    SUM(CASE WHEN order_year = 2022 THEN sales ELSE 0 END) AS sales_2022,
    SUM(CASE WHEN order_year = 2023 THEN sales ELSE 0 END) AS sales_2023
FROM cte
GROUP BY order_month
ORDER BY order_month;
````

##### Month with the Highest Sales for Each Category
```sql
WITH cte AS (
    SELECT category, FORMAT(order_date, 'yyyyMM') AS order_year_month, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY category, FORMAT(order_date, 'yyyyMM')
)
SELECT * FROM (
    SELECT *, ROW_NUMBER() OVER(PARTITION BY category ORDER BY sales DESC) AS rn
    FROM cte
) A
WHERE rn = 1;
````

##### Subcategory with the Highest Growth in Profit 
(2023 vs 2022)
```sql
WITH cte AS (
    SELECT sub_category, YEAR(order_date) AS order_year, SUM(sale_price) AS sales
    FROM df_orders
    GROUP BY sub_category, YEAR(order_date)
), cte2 AS (
    SELECT sub_category,
        SUM(CASE WHEN order_year = 2022 THEN sales ELSE 0 END) AS sales_2022,
        SUM(CASE WHEN order_year = 2023 THEN sales ELSE 0 END) AS sales_2023
    FROM cte
    GROUP BY sub_category
)
SELECT TOP 1 *, (sales_2023 - sales_2022) AS growth
FROM cte2
ORDER BY growth DESC;
````

## Technologies Used
- Python for data preprocessing
- Pandas for data manipulation
- SQL Server for storing and querying data
- SQLAlchemy for database connection
- Kaggle API for downloading datasets

## How to Run the Project
### Install the required dependencies:

pip install pandas sqlalchemy kaggle


### Steps:
1. Download the dataset using Kaggle API.
   - To configure the Kaggle API, create a `kaggle.json` file with your credentials.
   - The `kaggle.json` file should contain your Kaggle username and API key in the following format:
   {
     "username": "your_kaggle_username",
     "key": "your_kaggle_api_key"
   }
2. Extract and clean the data using the provided Python script.
3. Load the cleaned data into SQL Server.
4. Run the provided SQL queries to generate insights.

Conclusion
This project provides a structured approach to analyzing retail sales data, identifying trends, and supporting data-driven business decisions. The combination of Python and SQL enables efficient data processing and insightful analytics.
