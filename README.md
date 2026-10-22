# SQL Data Warehouse Project
### End-to-End Data Engineering with Microsoft SQL Server

**Building a structured analytical data warehouse by integrating CRM and ERP data through SQL-based ingestion, transformation, data quality processing, and dimensional modeling.**

---

## Table of Contents

- [Project Overview](#project-overview)
- [Business Context](#business-context)
- [Project Objectives](#project-objectives)
- [Solution Architecture](#solution-architecture)
- [Data Sources](#data-sources)
- [Data Engineering Workflow](#data-engineering-workflow)
- [Data Warehouse Layers](#data-warehouse-layers)
- [Data Transformation and Integration](#data-transformation-and-integration)
- [Dimensional Data Modeling](#dimensional-data-modeling)
- [Technology Stack](#technology-stack)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Execution Workflow](#execution-workflow)
- [Data Quality and Validation](#data-quality-and-validation)
- [Documentation](#documentation)
- [Engineering Concepts Demonstrated](#engineering-concepts-demonstrated)

---

## Project Overview

This project focuses on designing and implementing a SQL Server data warehouse that consolidates data from two operational source systems: **Customer Relationship Management (CRM)** and **Enterprise Resource Planning (ERP)**.

The solution follows a layered data architecture that separates raw data ingestion, data cleansing and standardization, and analytical data modeling. SQL scripts are organized by processing layer to make the workflow easier to understand, maintain, and validate.

The resulting warehouse is designed to provide a consistent, integrated representation of customer, product, and sales data for analytical querying and reporting.

Rather than querying disconnected source files independently, the project establishes a structured process for bringing data together and preparing it for downstream analysis.

## Business Context

Operational systems are typically designed to support individual business processes. Customer information, product details, sales transactions, locations, and product classifications may therefore be maintained across different datasets and structures.

Analyzing these sources independently can make it difficult to establish consistent relationships, apply common data quality rules, and produce reliable cross-domain reporting.

A data warehouse addresses this challenge by integrating data into a common analytical environment.

This project demonstrates that process through the following workflow:

- Consolidating CRM and ERP source data.
- Preserving raw source records in a dedicated landing layer.
- Applying consistent cleansing and standardization rules.
- Integrating related business entities across source systems.
- Organizing the resulting data into an analytical model.

## Project Objectives

The primary objectives are to:

1. **Integrate multiple sources:** Consolidate six CSV datasets from CRM and ERP systems into one warehouse.
2. **Implement layered architecture:** Separate raw ingestion, transformation, and analytical presentation into Bronze, Silver, and Gold layers.
3. **Develop SQL-based ETL processing:** Use SQL Server scripts and stored procedures to load and transform data.
4. **Improve data quality:** Address inconsistencies, invalid values, formatting differences, and data standardization requirements.
5. **Build an analytical data model:** Organize customer, product, and sales information for efficient analytical queries.
6. **Establish documentation and validation:** Document the architecture, data flow, naming conventions, requirements, and data structures.

## Solution Architecture

The warehouse uses a three-layer architecture, with data progressing from source ingestion through transformation to business-oriented presentation.

![Data Warehouse Architecture](02_docs/data_architecture.png)

### End-to-End Data Flow

```text
CRM Source Files                 ERP Source Files
       |                                |
       +---------------+----------------+
                       |
                       v
              +----------------+
              |  Bronze Layer  |
              | Raw Data Load  |
              +----------------+
                       |
                       v
              +----------------+
              |  Silver Layer  |
              | Clean & Prepare|
              +----------------+
                       |
                       v
              +----------------+
              |   Gold Layer   |
              | Analytical Model|
              +----------------+
                       |
                       v
              Analytical Queries
              and Reporting
```

The Bronze and Silver layers use tables to store successive representations of the data. The Gold layer presents business-oriented analytical objects, using a dimensional model to support querying.

## Data Sources

The project uses six CSV files from two operational domains.

| Source System | File | Intended Data Domain |
|---|---|---|
| CRM | `cust_info.csv` | Customer information |
| CRM | `prd_info.csv` | Product information |
| CRM | `sales_details.csv` | Sales transaction details |
| ERP | `CUST_AZ12.csv` | Additional customer attributes |
| ERP | `LOC_A101.csv` | Customer location information |
| ERP | `PX_CAT_G1V2.csv` | Product category and classification information |

### CRM — Customer Relationship Management

The CRM datasets provide customer, product, and sales information. They form the primary source for the customer and sales entities used in the warehouse.

### ERP — Enterprise Resource Planning

The ERP datasets contribute additional customer attributes, location information, and product classifications. These attributes can be integrated with CRM data to provide a more complete analytical representation of the business.

The source files are loaded into SQL Server through the Bronze layer before subsequent transformations are applied.

## Data Engineering Workflow

The processing workflow is organized into distinct stages.

### 1. Source Analysis

Understand the structure and meaning of the source datasets before designing target tables and transformation rules.

This includes identifying source entities, examining columns and data types, understanding relationships, and identifying data quality issues that must be addressed.

### 2. Raw Data Ingestion

Load the source CSV files into Bronze tables using SQL Server loading operations.

The Bronze layer preserves the source-oriented representation of the data and provides a starting point for downstream processing.

### 3. Data Cleansing and Standardization

Transform raw records in the Silver layer to improve consistency and reliability.

Depending on the source entity, this includes handling invalid values, standardizing formats, normalizing attributes, and applying appropriate data type conversions.

### 4. Data Integration

Combine related information from CRM and ERP datasets using suitable business keys and relationship rules.

Integration allows customer and product attributes from separate operational sources to contribute to a unified analytical model.

### 5. Dimensional Modeling

Organize the curated data into a sales-oriented dimensional model containing customer and product dimensions and a sales fact table.

### 6. Validation and Documentation

Use SQL checks and project documentation to examine the resulting data structures, transformation rules, and relationships.

---

## Data Warehouse Layers

### Bronze Layer — Raw Data

**Purpose:** Preserve source data in its original, source-oriented representation.

The Bronze layer acts as the initial landing area for the CRM and ERP CSV files.

**Responsibilities:**

- Load source records into SQL Server tables.
- Maintain source-aligned table structures.
- Minimize transformations during initial ingestion.
- Support traceability and troubleshooting.
- Provide the input for Silver-layer processing.

**Loading approach:** Full loading using the project's configured SQL loading scripts.

### Silver Layer — Cleansed and Standardized Data

**Purpose:** Improve data consistency and prepare records for integration and analysis.

The Silver layer applies transformation rules to the raw data received from Bronze.

**Responsibilities:**

- Clean inconsistent or invalid values.
- Standardize formats and representations.
- Apply appropriate data type conversions.
- Normalize attributes where required.
- Enrich records using related source information.
- Prepare consistent datasets for downstream modeling.

This layer separates data quality processing from raw ingestion, making the transformation logic easier to understand and maintain.

### Gold Layer — Business-Ready Data

**Purpose:** Present integrated data in a structure designed for analytical queries.

The Gold layer organizes the transformed data into business-oriented objects using dimensional modeling concepts.

**Responsibilities:**

- Integrate relevant customer, product, and sales information.
- Apply business-oriented transformation rules.
- Establish fact and dimension relationships.
- Provide analytical structures for reporting and querying.
- Document the meaning of business-facing tables and columns.

The intended Gold model contains a sales fact table and customer and product dimensions.

## Data Transformation and Integration

The transformation workflow addresses the differences between operational source systems and the requirements of analytical data modeling.

### Data Cleansing

Data cleansing improves the reliability of source records before they are integrated. The relevant SQL logic addresses the data quality issues identified during source analysis.

### Data Standardization

Standardization establishes consistent representations for values such as dates, categorical attributes, identifiers, and other fields where source formats differ.

### Data Normalization

Normalization and related transformation rules help establish consistent attributes and reduce avoidable inconsistencies in the Silver layer.

### Data Integration

CRM and ERP datasets are combined according to their business meaning and available keys. For example, customer-related information from separate systems can be aligned to support a more complete customer dimension, while product attributes and classifications contribute to product analysis.

### Data Loading

The project uses SQL scripts and stored procedures to organize loading operations. Bronze loading handles source ingestion, while Silver loading incorporates transformation logic.

The project follows the layer sequence:

**Source files → Bronze ingestion → Silver transformation → Gold modeling**

## Dimensional Data Modeling

The Gold layer follows a dimensional modeling approach centered on sales analysis.

![Data Model](02_docs/data_model.png)

### Fact Table

**`gold.fact_sales`**

The sales fact table represents sales transactions and their associated measures. It connects to customer and product dimensions to support analytical queries.

Typical analytical measures represented in the model include sales amount, quantity, and price, alongside transaction dates and identifiers.

### Customer Dimension

**`gold.dim_customers`**

The customer dimension provides customer-level descriptive attributes, including customer identifiers, names, geographic information, and relevant demographic attributes available from the source data.

It enables sales analysis by customer and customer characteristics.

### Product Dimension

**`gold.dim_products`**

The product dimension provides product-level descriptive attributes, including product identifiers, names, categories, subcategories, and other available product properties.

It enables sales analysis across products and product classifications.

### Analytical Value

Together, the fact and dimension tables provide a structured foundation for questions such as:

- How do sales vary across products?
- How can sales transactions be analyzed by customer?
- How can product categories be compared?
- How can sales be examined across available transaction dates?

These are analytical use cases supported by the intended model; specific findings depend on the source data and executed queries.

## Technology Stack

| Technology | Role |
|---|---|
| Microsoft SQL Server | Relational database engine and warehouse platform |
| T-SQL | Table definitions, data loading, transformation logic, and analytical queries |
| SQL Server Management Studio (SSMS) | Database administration and SQL development environment |
| Git | Source control for project files and scripts |
| GitHub | Repository hosting and project documentation |
| diagrams.net / draw.io | Architecture and data model diagrams |

## Repository Structure

```text
sql-data-warehouse-project/
├── 00_setup/
│   └── Environment setup resources
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
│   ├── bronze/
│   │   ├── ddl_bronze.sql
│   │   └── proc_load_bronze.sql
│   ├── silver/
│   │   ├── ddl_silver.sql
│   │   └── proc_load_silver.sql
│   └── gold/
│       └── ddl_gold.sql
│
├── 04_tests/
├── .gitignore
└── README.md
```

The structure separates environment setup, source datasets, documentation, SQL implementation, and validation resources. Individual files should be present in the repository as they are added and committed.

## Getting Started

### Prerequisites

- Microsoft SQL Server installed and running.
- SQL Server Management Studio (SSMS).
- The six project source CSV files.
- Access to execute database initialization and data loading scripts.
- SQL Server read access to the configured source-file paths.

### Step 1 — Obtain the Repository

Clone the repository or download its files, preserving the directory structure.

### Step 2 — Configure the Source Files

Place the CRM and ERP CSV files in their corresponding folders under `01_datasets/`.

The Bronze loading script must point to the actual location of these files. Update its file paths if your local environment uses a different directory.

### Step 3 — Initialize the Database

Open and execute:

```text
03_scripts/init_database.sql
```

This script initializes the `DataWarehouse` database and the `bronze`, `silver`, and `gold` schemas according to its definitions.

**Important:** Review the initialization script before execution. Initialization scripts may drop and recreate database objects, so do not rerun it after building data or objects you need to preserve without first checking its behavior.

### Step 4 — Create the Bronze Tables

Open and execute:

```text
03_scripts/bronze/ddl_bronze.sql
```

This script defines the raw-layer tables.

### Step 5 — Load the Bronze Layer

Open the Bronze loading script and ensure the source paths are correct:

```text
03_scripts/bronze/proc_load_bronze.sql
```

Execute the script according to its procedure definition to load the source files.

### Step 6 — Create and Load the Silver Layer

Execute the Silver table definitions:

```text
03_scripts/silver/ddl_silver.sql
```

Then execute the Silver loading procedure definitions and transformation workflow:

```text
03_scripts/silver/proc_load_silver.sql
```

Check the script contents and procedure execution instructions before running them.

### Step 7 — Build the Gold Layer

Execute:

```text
03_scripts/gold/ddl_gold.sql
```

This script defines the Gold-layer objects according to the project's data model.

### Step 8 — Validate the Results

Run the applicable SQL checks in `04_tests/` and inspect the resulting tables, views, and relationships.

## Data Quality and Validation

Data quality is an important part of the warehouse workflow because inconsistent source records can affect analytical results.

The project separates data quality processing into the Silver layer and uses validation resources to examine the resulting data structures.

Relevant validation areas include:

- **Completeness:** Check for missing values in important attributes.
- **Consistency:** Verify that values follow the expected formats and representations.
- **Uniqueness:** Examine keys that are expected to identify individual records.
- **Referential integrity:** Check relationships between fact and dimension records.
- **Transformation correctness:** Confirm that cleansing and standardization rules produce the intended results.
- **Load validation:** Compare source and target records where appropriate.

Validation outcomes should be determined by executing the actual SQL checks against the loaded data.

## Documentation

The `02_docs/` directory provides supporting material for understanding the warehouse design and implementation.

| Document | Purpose |
|---|---|
| `Project_Requirements.pdf` | Defines the project scope and business requirements |
| `data_architecture.png` | Illustrates the overall warehouse architecture |
| `data_layers.pdf` | Explains the responsibilities of Bronze, Silver, and Gold |
| `data_flow.png` | Illustrates the flow of data through the warehouse |
| `data_integration.png` | Documents integration across source entities |
| `data_model.png` | Illustrates the analytical data model |
| `data_catalog.md` | Describes Gold-layer tables and their columns |
| `naming_conventions.md` | Defines naming conventions for database objects |
| `Project_Notes_Sketches.pdf` | Provides conceptual explanations and project learning material |
| `ETL.png` | Illustrates the ETL process |
| `requirements.md` | Provides the written project requirements |

## Engineering Concepts Demonstrated

- SQL Server database and schema initialization.
- Layered data warehouse architecture.
- ETL processing using SQL and stored procedures.
- CSV-based source ingestion.
- Data cleansing and standardization.
- Integration of CRM and ERP source data.
- Dimensional modeling with fact and dimension tables.
- Data quality validation.
- SQL code organization and version control.
- Technical documentation and data lineage.

---

**Project focus:** SQL Server · Data Engineering · ETL · Data Integration · Data Quality · Dimensional Modeling
