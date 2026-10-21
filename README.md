# sql-data-warehouse-project
Building a modern data warehouse with SQL Server, including ETL processes, data modeling and analytics 

# SQL Data Warehouse Project

## Project Overview

This project implements a SQL Server data warehouse that integrates sales-related data from CRM and ERP source systems into a structured, analytics-ready environment.

The warehouse follows a layered architecture — **Bronze, Silver, and Gold** — to organize raw data, apply data cleansing and transformation, and prepare business-ready data for analytical queries.

## Architecture

- **Bronze Layer:** Loads raw CRM and ERP source data into SQL Server tables with minimal transformation.
- **Silver Layer:** Cleans, standardizes, normalizes, and enriches data to improve consistency and quality.
- **Gold Layer:** Organizes transformed data into business-oriented tables and views for reporting and analysis.

## Data Sources

The project uses six CSV source files from two systems:

- **CRM:** Customer information, product information, and sales details.
- **ERP:** Customer details, customer locations, and product category information.

## Technology Stack

- Microsoft SQL Server
- T-SQL
- SQL Server Management Studio (SSMS)
- Git and GitHub

## ETL Workflow

1. **Extract:** Read the CRM and ERP CSV files.
2. **Load:** Store the source data in the Bronze layer.
3. **Transform:** Clean and standardize data in the Silver layer.
4. **Model:** Build business-ready tables and views in the Gold layer.
5. **Analyze:** Query the curated data for sales, customer, and product insights.

## Data Model

The Gold layer uses a sales-oriented dimensional model consisting of:

- **Fact table:** `fact_sales`
- **Dimension tables:** `dim_customers` and `dim_products`

This structure supports analytical queries across sales transactions, customers, and products.

## Repository Structure

```text
sql-data-warehouse-project/
├── 00_setup/
├── 01_datasets/
├── 02_docs/
├── 03_scripts/
│   ├── init_database.sql
│   ├── bronze/
│   ├── silver/
│   └── gold/
├── 04_tests/
├── .gitignore
└── README.md
```

## Project Objectives

- Integrate CRM and ERP data into a centralized SQL Server data warehouse.
- Implement a structured Bronze, Silver, and Gold architecture.
- Apply SQL-based data cleansing and transformation.
- Develop a dimensional data model for analytical queries.
- Document the architecture, data flow, and data processing workflow.
