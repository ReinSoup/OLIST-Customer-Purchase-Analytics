# Cleaned Data

This folder contains the cleaned Olist datasets used as the input for the customer purchase analytics workflow.

The files were produced during the data cleaning stage of the upstream Olist Data Quality & ETL project and are used here as the validated source data for customer-level analysis.

## Purpose

The cleaned datasets provide a consistent starting point for:

- Customer identity analysis
- Order validation
- Purchase history construction
- Order-level monetary calculations
- Customer-level feature engineering
- Repeat-purchase modeling

The cleaning process addresses data quality issues identified during the inspection stage while preserving the structure and relationships of the original Olist dataset.

## Contents

The folder contains cleaned CSV versions of the Olist datasets used by the project.

The primary datasets used in this project are:

| Dataset | Role in this project |
|---|---|
| `olist_customers_dataset.csv` | Customer identity and location information |
| `olist_orders_dataset.csv` | Order status and purchase timeline |
| `olist_order_items_dataset.csv` | Item-level order values used to calculate order revenue |

## Data Grain

The datasets have different levels of granularity:

```text
Customers
1 row = customer record

Orders
1 row = order

Order Items
1 row = item within an order