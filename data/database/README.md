# Olist Database

This folder contains the SQLite database created from the cleaned Olist e-commerce datasets.

The database provides a structured, relational version of the cleaned data that can be queried using SQL and used for further analysis.

## Purpose

The database organizes the cleaned Olist datasets into related tables, making the data easier to:

- Query using SQL
- Explore and validate
- Analyze across related entities
- Use from Python or database tools

The database is created from the cleaned and validated data produced during the data preparation stage.

## Contents

This folder contains:

- SQLite database containing the cleaned Olist tables
- Related tables representing the original Olist data model
- Primary and foreign key relationships between related entities

## Database Structure

The database preserves the relational structure of the Olist dataset.

Key entities include:

- Customers
- Orders
- Order Items
- Products
- Sellers
- Payments
- Reviews

The tables are connected through the appropriate identifiers, allowing related information to be queried across the dataset.

For example:

```text
Customers
    ↓
Orders
    ↓
Order Items
    ↓
Products / Sellers
```

Additional tables such as payments and reviews are related to orders through the corresponding order identifiers.

## Role in the Project

The database represents the structured storage stage of the data pipeline:

```text
Raw Olist Data
      ↓
Data Inspection
      ↓
Data Cleaning & Validation
      ↓
Cleaned CSVs
      ↓
SQLite Database
      ↓
SQL Analysis / Downstream Analytics
```

The database is primarily used as a structured SQL representation of the cleaned Olist data.

The customer purchase analytics workflow uses the cleaned CSV datasets directly for its customer-level feature engineering and modeling process.

## Usage

The database can be opened and queried using:

- SQLite
- Python
- VS Code database extensions
- DBeaver
- Other SQLite-compatible database tools

Example query:

```sql
SELECT *
FROM orders
LIMIT 10;
```

## Related Project

The database was created as part of the:

[Olist Data Quality & ETL Pipeline](https://github.com/ReinSoup/OLIST-Data-Quality-ETL)

The resulting structured data provides a validated SQL representation of the Olist dataset that can be used for downstream analysis.
