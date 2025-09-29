## Project Overview
![Apple_Store](https://github.com/SnehalAnalytics/SQL_Project-Analyzing-1M-Apple-Products-Sales-Records/blob/main/Apple_Store.jpeg)
This project involved designing and analyzing a relational database (apple_db) of over 1 million rows to manage Apple’s global sales, products, stores, categories, and warranty claims. A normalized schema was implemented with foreign key constraints, and the Sales table was modeled as the fact table for efficient reporting and analysis. To optimize performance, indexes and query tuning techniques were applied, reducing query execution times by over 90%. Comprehensive SQL queries were developed to solve 20+ business problems, including sales trends, warranty analytics, product performance, and store-level growth. This project demonstrates strong skills in SQL, data modeling, query optimization, and business-focused analytics to deliver actionable insights.

## Dataset

- **Size**: 1 million+ rows of sales data.
- **Period Covered**: The data spans from year 2019 to 2024.
- **Geographical Coverage**: Sales data from Apple stores across various countries.

## Database Schema
This project is built around five tables that capture store, product, category, sales, and warranty data for Apple retail operations::

1. **stores**: Contains information about Apple retail stores.
   - `store_id`: Unique identifier for each store.
   - `store_name`: Name of the store.
   - `city`: City where the store is located.
   - `country`: Country of the store.

2. **category**: Holds product category information.
   - `category_id`: Unique identifier for each product category.
   - `category_name`: Name of the category.

3. **products**: Details about Apple products.
   - `product_id`: Unique identifier for each product.
   - `product_name`: Name of the product.
   - `category_id`: References the category table.
   - `launch_date`: Date when the product was launched.
   - `price`: Price of the product.

4. **sales**: Stores sales transactions.
   - `sale_id`: Unique identifier for each sale.
   - `sale_date`: Date of the sale.
   - `store_id`: References the store table.
   - `product_id`: References the product table.
   - `quantity`: Number of units sold.

5. **warranty**: Contains information about warranty claims.
   - `claim_id`: Unique identifier for each warranty claim.
   - `claim_date`: Date the claim was made.
   - `sale_id`: References the sales table.
   - `repair_status`: Status of the warranty claim (e.g., Paid Repaired, Warranty Void).

## Entity Relationship Diagram (ERD)


## Project Focus

This project primarily focuses on developing and showcasing the following SQL skills:

-- **Database Design & Data Modeling**: Created normalized schema with primary/foreign keys (Stores, Products, Category, Sales, Warranty).
- **Complex Joins and Aggregations**: Demonstrating the ability to perform complex SQL joins and aggregate data meaningfully.
- **Window Functions**: Using advanced window functions for running totals, growth analysis, and time-based queries.
- **Data Segmentation**: Analyzing data across different time frames to gain insights into product performance.
- **Correlation Analysis**: Applying SQL functions to determine relationships between variables, such as product price and warranty claims.
- **Date & Time Functions**: Applied date extraction, interval-based segmentation, and monthly/annual grouping for time-series analysis.
- **Business Problem Solving**: Delivered insights on store performance, product sales trends, warranty claims, and customer behavior.

## Objectives

To design and implement data-driven analyses that uncover sales performance, product trends, and warranty claim patterns across stores and countries. The goal is to leverage SQL queries and analytical techniques to generate actionable insights on store performance, customer behavior, product lifecycle, and warranty risks—enabling better decision-making, performance tracking, and strategic planning.
