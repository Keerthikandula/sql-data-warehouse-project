# SQL Data Warehouse Project

**End-to-End Data Engineering Project | Microsoft SQL Server | ETL | Data Integration | Dimensional Modeling**

## 1. Project Overview

This project demonstrates the development of a SQL Server data warehouse that integrates customer, product, location, category, and sales data from two source systems: **Customer Relationship Management (CRM)** and **Enterprise Resource Planning (ERP)**.

The project follows a three-layer data warehouse architecture consisting of **Bronze, Silver, and Gold**. Data is first loaded from CSV files into the Bronze layer, cleaned and standardized in the Silver layer, and integrated into business-oriented analytical views in the Gold layer.

The final analytical model consists of customer and product dimensions and a sales fact, organized into a star schema to support customer, product, and sales analysis.

The implementation is organized into database initialization, table creation, data loading, transformation, analytical modeling, and documentation scripts.

## 2. Project Objectives

- Build a SQL Server data warehouse using data from CRM and ERP source systems.
- Organize data processing into Bronze, Silver, and Gold layers.
- Load raw source data from six CSV files using SQL Server bulk-loading functionality.
- Create reusable stored procedures to orchestrate Bronze and Silver data loads.
- Identify and handle data-quality issues before integrating the data.
- Standardize customer and product attributes across source systems.
- Integrate related CRM and ERP records into unified analytical entities.
- Create customer and product dimension views and a sales fact view.
- Organize the Gold layer into a star schema.
- Document the architecture, data flow, source-to-target integration, naming conventions, and Gold-layer data definitions.

## 3. Source Systems and Datasets

The warehouse uses six CSV files distributed across two source systems. The CRM files provide customer, product, and transaction information, while the ERP files contribute additional customer, location, and product-category information.

| Source system | CSV file | Data represented |
|---|---|---|
| CRM | `cust_info.csv` | Customer information and attributes |
| CRM | `prd_info.csv` | Product information and attributes |
| CRM | `sales_details.csv` | Sales transaction details |
| ERP | `CUST_AZ12.csv` | Additional customer attributes |
| ERP | `LOC_A101.csv` | Customer location information |
| ERP | `PX_CAT_G1V2.csv` | Product category information |

The files are stored in separate source directories to preserve their source-system organization.

```text
01_datasets/
├── source_crm/
│   ├── cust_info.csv
│   ├── prd_info.csv
│   └── sales_details.csv
└── source_erp/
    ├── CUST_AZ12.csv
    ├── LOC_A101.csv
    └── PX_CAT_G1V2.csv
```

The source files are the inputs to the warehouse pipeline. Their fields are initially retained in the Bronze layer, then selected, cleaned, converted, and integrated as the data moves through the subsequent layers.

## 4. Data Warehouse Architecture

The solution uses a three-layer architecture that separates source ingestion from data cleaning and analytical modeling.

![Data Warehouse Architecture](02_docs/data_architecture.png)

### Bronze Layer — Raw Data Ingestion

The Bronze layer stores source-oriented copies of the six CRM and ERP datasets.

Its main responsibilities are:

- Create tables corresponding to the source files.
- Load CSV data into SQL Server using `BULK INSERT`.
- Preserve source values before applying business transformations.
- Use a full-load approach that clears existing table data before reloading.
- Keep source-system prefixes in table names to make data lineage easier to follow.
- Add load metadata to support traceability in the downstream layers.

The Bronze layer is the starting point for data-quality investigation and subsequent transformations.

### Silver Layer — Data Cleaning and Standardization

The Silver layer contains cleaned and standardized versions of the source entities.

Its main responsibilities are:

- Investigate data-quality problems in the Bronze tables.
- Handle duplicate customer records and select the appropriate record where the source provides a relevant creation timestamp.
- Remove unwanted leading and trailing spaces from text values.
- Standardize abbreviations and inconsistent categorical values.
- Handle missing values according to the transformation rules.
- Convert values to appropriate data types.
- Clean and validate customer, product, and sales-related data.
- Integrate the cleaned records into a consistent structure for the Gold layer.
- Maintain load metadata using a data warehouse creation timestamp.

The Silver layer retains the separation between individual source entities while improving their consistency and reliability.

### Gold Layer — Business-Ready Analytical Views

The Gold layer presents integrated data through analytical views designed for reporting and analysis.

Its main responsibilities are:

- Combine relevant customer attributes from CRM and ERP.
- Combine product attributes with the corresponding ERP product-category information.
- Connect sales transactions to the customer and product entities.
- Expose customer and product dimensions and a sales fact.
- Apply consistent analytical naming conventions.
- Provide a star-schema structure that simplifies downstream querying.

The Gold layer is organized around three principal analytical objects:

| Object | Type | Purpose |
|---|---|---|
| `gold.dim_customers` | View | Unified customer attributes |
| `gold.dim_products` | View | Product attributes and category information |
| `gold.fact_sales` | View | Sales transactions associated with customer and product keys |

## 5. Database and Schema Setup

The project initializes a SQL Server database named `DataWarehouse` and organizes its objects into three schemas.

```text
DataWarehouse
├── bronze
├── silver
└── gold
```

The initialization script is:

`03_scripts/init_database.sql`

It establishes the database and schemas used by the subsequent scripts.

The layer schemas provide a consistent location for each stage of the pipeline:

- `bronze` — source-oriented tables and raw data loading.
- `silver` — cleaned tables and transformation procedures.
- `gold` — analytical views for dimensions and facts.

Run the initialization script before executing the layer-specific scripts. Review any database creation or reset statements before running them against an existing database.

## 6. Bronze Layer Implementation

### 6.1 Bronze Table Creation

The script `03_scripts/bronze/ddl_bronze.sql` defines the tables used to store the six source datasets.

The source-system prefix distinguishes CRM tables from ERP tables.

| Source file | Bronze table |
|---|---|
| `cust_info.csv` | `bronze.crm_cust_info` |
| `prd_info.csv` | `bronze.crm_prd_info` |
| `sales_details.csv` | `bronze.crm_sales_details` |
| `CUST_AZ12.csv` | `bronze.erp_cust_az12` |
| `LOC_A101.csv` | `bronze.erp_loc_a101` |
| `PX_CAT_G1V2.csv` | `bronze.erp_px_cat_g1v2` |

The table definitions specify the columns and data types used to receive the source data. At this stage, the purpose is ingestion rather than business-level cleaning.

### 6.2 Loading CSV Files

The Bronze loading logic uses SQL Server's `BULK INSERT` statement to import data from the source CSV files.

A typical load follows this sequence:

1. Identify the source file and target Bronze table.
2. Clear existing rows from the target table.
3. Import the source data.
4. Repeat the operation for the remaining source tables.

The full-load approach is implemented using `TRUNCATE TABLE` followed by `BULK INSERT`. This makes the procedure suitable for rebuilding the raw layer from the supplied source files, provided the files are accessible and the SQL Server environment is configured correctly.

Example of the loading pattern:

```sql
TRUNCATE TABLE bronze.crm_cust_info;

BULK INSERT bronze.crm_cust_info
FROM 'path_to_source_file/cust_info.csv'
WITH (
    FIRSTROW = 2,
    FIELDTERMINATOR = ',',
    TABLOCK
);
```

This example illustrates the loading pattern. The file path, CSV format settings, and other options must match the actual source file and local SQL Server environment.

**Execution requirement:** the source paths used by `BULK INSERT` must be accessible to the SQL Server process. A path that works in the user's file explorer is not necessarily accessible to the database engine.

### 6.3 Bronze Load Stored Procedure

The script `03_scripts/bronze/proc_load_bronze.sql` consolidates the Bronze loading operations into a reusable stored procedure.

Instead of executing each table load manually every time, the procedure runs the configured sequence of truncation and source-file imports.

The procedure follows this general pattern:

```sql
TRUNCATE TABLE bronze.crm_cust_info;

-- Bulk-load the corresponding source CSV.

-- Repeat the configured load for the other source tables.
```

The actual procedure contains the individual table-loading statements. It should be executed after the Bronze tables have been created and the source paths have been configured.

This provides a single entry point for refreshing the raw layer.

## 7. Silver Layer Implementation

### 7.1 Silver Table Creation

The script `03_scripts/silver/ddl_silver.sql` creates the six Silver tables corresponding to the CRM and ERP source entities.

The tables retain the source-system organization while allowing the transformation logic to produce cleaned, standardized values.

The Silver table definitions also include a warehouse metadata column using the `dwh_` prefix and a default timestamp based on `GETDATE()`, as described in the project workflow.

This distinguishes source-provided fields from metadata generated during warehouse processing.

### 7.2 Data-Quality Investigation

Before transforming each Bronze entity, the project investigates the quality of its existing data.

The checks described in the workflow include:

- **Primary-key completeness:** identify missing identifiers.
- **Duplicate detection:** group records by their expected key and identify repeated values.
- **Text consistency:** detect unwanted leading and trailing spaces.
- **Categorical consistency:** inspect distinct values and identify inconsistent abbreviations.
- **Data-type suitability:** determine whether source values can be represented using the intended target data types.
- **Relationship consistency:** examine the identifiers needed to integrate entities in the Gold layer.

For example, duplicate identifiers can be investigated using `GROUP BY` and `HAVING`, while text-space problems can be identified by comparing a string with its trimmed value.

Example duplicate-detection query:

```sql
SELECT
    cst_id,
    COUNT(*) AS record_count
FROM bronze.crm_cust_info
GROUP BY cst_id
HAVING COUNT(*) > 1;
```

The column name in this example is illustrative; use the actual identifier defined in the Bronze table.

The objective is to identify the source problem before choosing the corresponding transformation rule.

### 7.3 Customer Data Cleaning

The customer transformation workflow addresses several source-data issues.

**Duplicate customer records**

Where the source contains repeated customer identifiers, the workflow uses `ROW_NUMBER()` with a partition by customer identifier and an ordering based on the creation timestamp. The most recent record is selected where that timestamp is appropriate for resolving the duplicate.

**Whitespace removal**

The `TRIM()` function is used to remove unwanted leading and trailing spaces from text fields, including customer names.

**Categorical standardization**

Abbreviated values are mapped to consistent descriptive values. The workflow includes standardization of gender and marital-status attributes, with missing values handled according to the defined mapping rules.

**Case normalization**

The `UPPER()` function is used where necessary to make mappings more robust to differences in letter casing.

These transformations make customer attributes more consistent before CRM and ERP records are integrated.

### 7.4 Product Data Cleaning

The product transformation workflow prepares the CRM product data for analytical use.

The processing includes investigating product identifiers, duplicate records, text values, product attributes, and the source fields needed for integration with ERP category information.

Product identifiers and source-system fields are retained according to the table definitions and naming conventions. Product records are then prepared for the unified Gold product dimension.

### 7.5 Sales Data Cleaning

The sales transformation workflow prepares the transaction data for the Gold fact view.

The workflow investigates the sales-related identifiers, dates, quantities, and monetary values before mapping the source fields into the Silver structure.

The transformation logic addresses the consistency of the values used in downstream calculations and joins. Date and numeric conversions must be aligned with the source formats and the corresponding SQL data types.

### 7.6 ERP Data Cleaning

The three ERP entities provide complementary information used to enrich CRM customer and product records.

- `bronze.erp_cust_az12` provides additional customer attributes.
- `bronze.erp_loc_a101` provides customer location information.
- `bronze.erp_px_cat_g1v2` provides product category information.

The Silver transformations prepare these entities for joining with the corresponding CRM records. Identifiers, categorical fields, missing values, and text consistency are handled according to the source structure and the transformation rules.

### 7.7 Silver Load Stored Procedure

The script `03_scripts/silver/proc_load_silver.sql` consolidates the Silver transformation and loading operations into a reusable stored procedure.

Its overall process is:

1. Read source-oriented records from the Bronze tables.
2. Apply the defined cleaning and standardization logic.
3. Insert the transformed results into the corresponding Silver tables.
4. Refresh the Silver tables using the configured full-load approach.

The workflow uses truncation before reloading to avoid accumulating duplicate rows across repeated full-load executions.

The procedure depends on the Bronze layer being populated and the Silver table definitions being available.

## 8. CRM and ERP Data Integration

The warehouse brings together related information from two systems rather than treating each source file as an independent analytical dataset.

The CRM system provides the principal customer, product, and sales entities. The ERP system contributes customer attributes, location information, and product categories.

### Customer Integration

The customer dimension combines customer information from CRM with relevant attributes from the ERP customer and location entities.

The integration uses the relevant customer identifiers and joins the cleaned Silver tables to produce a unified customer record.

The resulting dimension provides a single analytical entity for customer-level reporting.

### Product Integration

The product dimension combines CRM product information with ERP product-category data.

The relevant product-category relationship is used to enrich the product record with category information, allowing product analysis to use a consistent structure.

### Sales Integration

The sales fact view uses the cleaned transaction records and relates them to the customer and product dimensions through the corresponding business identifiers and generated dimension keys.

The relationships between the fact and dimensions are checked to identify missing or inconsistent matches.

![Data Integration](02_docs/data_integration.png)

## 9. Gold Layer and Dimensional Model

The Gold layer presents the integrated data in a dimensional model suitable for analytical queries.

![Gold Data Model](02_docs/data_model.png)

### 9.1 Customer Dimension — `gold.dim_customers`

The customer dimension combines the customer attributes selected from the cleaned CRM and ERP data.

It provides a unified customer identifier and the descriptive attributes defined by the Gold view.

The view follows the project's dimensional naming convention, including the use of a customer surrogate key ending in `_key`, where defined by the model.

This structure allows customer attributes to be queried from a single analytical object rather than repeatedly joining the underlying source entities.

### 9.2 Product Dimension — `gold.dim_products`

The product dimension combines product attributes from CRM with the corresponding ERP category information.

The view provides a consistent product-oriented structure for analytical queries. Its fields and surrogate-key naming follow the Gold data model and the project's naming conventions.

Product categories can be used alongside product attributes in downstream analysis.

### 9.3 Sales Fact — `gold.fact_sales`

The sales fact view represents the transaction records used for sales analysis.

It connects the sales data to the customer and product dimensions through their corresponding keys and exposes the transaction attributes defined in the Gold model.

This makes the fact view the central object for queries that analyze sales by customer or product.

### 9.4 Star Schema Relationships

The intended analytical structure places the sales fact at the center, with customer and product dimensions providing descriptive context.

```text
                 gold.dim_customers
                         |
                         |
                         v
                   gold.fact_sales
                         ^
                         |
                         |
                  gold.dim_products
```

The fact-to-dimension relationships allow sales records to be analyzed using customer and product attributes.

The model is designed around the latest supplied dataset; historical versioning such as Slowly Changing Dimension Type 2 is not part of the stated project scope.

### 9.5 Gold View Creation

The script `03_scripts/gold/ddl_gold.sql` contains the SQL definitions for the Gold analytical views.

The views are created after the Silver data is prepared. They apply the required source integration, column selection, aliases, and dimensional relationships.

The script establishes the analytical objects:

- `gold.dim_customers`
- `gold.dim_products`
- `gold.fact_sales`

Because these objects are views, their results are derived from the underlying Silver tables when queried.

## 10. Data Quality and Model Validation

Validation is performed at the stages where data quality and relationships affect the correctness of the warehouse output.

### Bronze Validation

- Confirm that each source file loads into its intended table.
- Check row counts after loading.
- Inspect the imported values and column mapping.
- Investigate load failures or malformed source records.

### Silver Validation

- Recheck key completeness and duplicate identifiers.
- Confirm that trimming and categorical mappings behave as intended.
- Review conversions and handling of missing or invalid values.
- Confirm that the Silver tables are populated after the load procedure runs.

### Gold Validation

- Confirm that the customer and product dimension views return the expected structure.
- Check the relationships between sales records and their customer and product keys.
- Investigate unmatched records.
- Review the consistency of dimension identifiers and fact records.
- Confirm that the integrated attributes originate from the intended CRM and ERP entities.

A successful SQL execution alone does not prove data quality. Validation results should be based on the actual output of the relevant queries.

## 11. End-to-End Execution Sequence

Run the project scripts in dependency order. Before loading data, confirm that SQL Server can access the CSV paths configured in the Bronze load procedure.

| Step | Script | Purpose |
|---|---|---|
| 1 | `03_scripts/init_database.sql` | Initialize the database and layer schemas |
| 2 | `03_scripts/bronze/ddl_bronze.sql` | Create Bronze source tables |
| 3 | `03_scripts/bronze/proc_load_bronze.sql` | Define the Bronze loading procedure |
| 4 | Execute the Bronze load procedure | Load the six CSV datasets |
| 5 | `03_scripts/silver/ddl_silver.sql` | Create Silver tables and metadata columns |
| 6 | `03_scripts/silver/proc_load_silver.sql` | Define the Silver transformation and loading procedure |
| 7 | Execute the Silver load procedure | Clean and load the Silver tables |
| 8 | `03_scripts/gold/ddl_gold.sql` | Create the Gold dimension and fact views |
| 9 | Run validation queries | Check data quality and Gold relationships |

The Gold layer depends on the Silver layer, and the Silver layer depends on the Bronze layer. Each prerequisite should be completed before moving to the next stage.

### Environment Configuration

The project uses Microsoft SQL Server and SQL Server Management Studio (SSMS).

Before execution:

1. Connect to the intended SQL Server instance in SSMS.
2. Review and run the database initialization script.
3. Confirm that the source CSV files are present in the dataset directories.
4. Update any machine-specific file paths in the loading procedure.
5. Ensure the SQL Server service account can access the source files.
6. Execute the scripts in the sequence above.
7. Review the output and run the relevant validation queries.

The exact server configuration and file paths depend on the local environment.

## 12. Repository Structure

```text
SQL-Data-Warehouse-Project/
│
├── 00_setup/
│   └── SQL Server environment setup resources
│
├── 01_datasets/
│   ├── source_crm/
│   │   ├── cust_info.csv
│   │   ├── prd_info.csv
│   │   └── sales_details.csv
│   │
│   └── source_erp/
│       ├── CUST_AZ12.csv
│       ├── LOC_A101.csv
│       └── PX_CAT_G1V2.csv
│
├── 02_docs/
│   ├── ETL.png
│   ├── Project_Notes_Sketches.pdf
│   ├── Project_Requirements.pdf
│   ├── data_architecture.png
│   ├── data_catalog.md
│   ├── data_flow.png
│   ├── data_integration.png
│   ├── data_layers.pdf
│   ├── data_model.png
│   ├── naming_conventions.md
│   └── requirements.md
│
├── 03_scripts/
│   ├── init_database.sql
│   │
│   ├── bronze/
│   │   ├── ddl_bronze.sql
│   │   └── proc_load_bronze.sql
│   │
│   ├── silver/
│   │   ├── ddl_silver.sql
│   │   └── proc_load_silver.sql
│   │
│   └── gold/
│       └── ddl_gold.sql
│
├── 04_tests/
│   └── Data quality and model validation queries
│
├── .gitignore
└── README.md
```

## 13. SQL Scripts Reference

| Script | Responsibility |
|---|---|
| `init_database.sql` | Creates the database and layer schemas |
| `ddl_bronze.sql` | Defines the six Bronze tables |
| `proc_load_bronze.sql` | Defines the full-load procedure for importing source CSV data |
| `ddl_silver.sql` | Defines the six Silver tables and warehouse metadata columns |
| `proc_load_silver.sql` | Defines the procedure for cleaning and loading Silver data |
| `ddl_gold.sql` | Defines the customer and product dimension views and sales fact view |

The DDL scripts define the database objects. The stored procedure scripts consolidate repeatable data-loading operations, while the Gold script exposes the integrated analytical model.

## 14. Project Documentation

The `02_docs/` directory contains the diagrams, requirements, and data definitions associated with the warehouse.

| File | Purpose |
|---|---|
| `ETL.png` | Visual reference for the ETL workflow |
| `Project_Notes_Sketches.pdf` | Project notes and design sketches |
| `Project_Requirements.pdf` | Project requirements and scope |
| `data_architecture.png` | High-level warehouse architecture |
| `data_catalog.md` | Definitions of Gold-layer tables and columns |
| `data_flow.png` | Movement of data from source systems through the layers |
| `data_integration.png` | Relationships used to integrate CRM and ERP entities |
| `data_layers.pdf` | Responsibilities of the Bronze, Silver, and Gold layers |
| `data_model.png` | Dimensional model and star-schema relationships |
| `naming_conventions.md` | Naming rules for database objects and columns |
| `requirements.md` | Written requirements for the warehouse |

Together, these resources explain the intended design and how the source data maps into the final analytical model.

## 15. Technology Stack

- **Microsoft SQL Server** — relational database engine and warehouse storage.
- **SQL Server Management Studio (SSMS)** — SQL development, script execution, and data inspection.
- **T-SQL** — table definitions, stored procedures, data loading, cleaning, transformations, joins, and analytical view creation.
- **CSV** — source data format for the CRM and ERP datasets.
- **Git and GitHub** — source control and project repository management.
- **diagrams.net / draw.io** — architecture, data flow, integration, and dimensional-model diagrams.

## 16. Expected Project Output

After successful execution of the complete pipeline, the warehouse is organized into three layers:

- **Bronze:** six source-oriented tables populated from the CRM and ERP CSV files.
- **Silver:** six cleaned and standardized tables with warehouse load metadata.
- **Gold:** three analytical views representing customer, product, and sales data.

The Gold model provides a unified structure for querying sales transactions with customer and product context, while the layered design keeps raw ingestion, data cleaning, and analytical modeling separate.

The final row counts, detailed column definitions, and validation outcomes should be confirmed from the executed SQL scripts and query results.

adapted as a portfolio project with organized SQL scripts and supporting documentation.

