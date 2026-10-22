# dbt BigQuery Data Transformation Project

## Project Overview

This project demonstrates a SQL-based data transformation workflow using **dbt (Data Build Tool) and Google BigQuery**. It organizes analytical data models, defines data quality tests, and manages seed data and project configurations within a structured dbt project.

The project uses a modular approach to SQL development, with model files organized into directories and configuration and testing logic maintained separately. It also includes dbt-generated documentation to provide a navigable view of the project's models and data resources.

## Project Objectives

- Configure a dbt project to work with Google BigQuery.
- Define SQL models for data transformation and analytical querying.
- Organize model definitions, seed data, tests, macros, analyses, and snapshots using dbt's project structure.
- Configure model-level and project-level settings through YAML files.
- Define data quality checks for model data.
- Generate project documentation to make model definitions and relationships easier to explore.

## Technologies Used

| Technology | Purpose |
|---|---|
| **dbt Core** | Project configuration, SQL model management, testing, and documentation generation |
| **Google BigQuery** | Cloud data warehouse targeted by the dbt project |
| **SQL** | Defining data models and transformation logic |
| **YAML** | Managing dbt project settings, model configurations, and schema definitions |
| **CSV** | Providing seed data for ingestion into the warehouse |
| **Git and GitHub** | Version control and project publication |

## Project Structure

```text
dbt-bigquery-course/
├── analyses/
├── macros/
├── models/
│   ├── example/
│   │   ├── customers.sql
│   │   ├── my_first_dbt_model.sql
│   │   ├── my_second_dbt_model.sql
│   │   └── schema.yml
│   └── schema.yml
├── seeds/
│   └── stadiums.csv
├── snapshots/
├── tests/
├── .gitignore
├── README.md
└── dbt_project.yml
```

*The structure above represents the project's principal directories and files. Empty directories may be represented by `.gitkeep` files in the repository.*

## Implementation Details

### 1. dbt Project Configuration

The `dbt_project.yml` file defines the project's name, version, profile, and resource paths. It configures where dbt locates models, analyses, tests, seeds, macros, and snapshots.

This configuration provides a consistent structure for managing the project's transformation resources.

### 2. SQL Data Models

The `models/` directory contains the SQL models used to define data transformations.

- **`customers.sql`** — Defines a customer-related data model.
- **`my_first_dbt_model.sql`** — Contains the first example SQL model.
- **`my_second_dbt_model.sql`** — Contains a second SQL model, demonstrating how multiple models can be maintained within one dbt project.
- **`schema.yml`** — Stores schema-level metadata and configured tests for the associated models.

The models are maintained as SQL files, allowing transformation logic to be organized, reviewed, and version-controlled independently.

### 3. Seed Data

The `seeds/stadiums.csv` file provides structured, file-based data that can be loaded into the warehouse using dbt's seed functionality.

Seed files are useful for reference datasets and other small, version-controlled data sources used by analytical models.

### 4. Schema Definitions and Data Quality

The schema YAML files provide a location for model documentation, column metadata, and dbt data tests.

These definitions can be used to validate configured data quality conditions, depending on the tests specified in the YAML files.

### 5. Resource Organization

The project includes separate directories for additional dbt resources:

- **`analyses/`** — Holds analytical SQL queries that are not configured as standard dbt models.
- **`macros/`** — Provides a location for reusable SQL-generation logic.
- **`snapshots/`** — Provides a location for tracking changes in source data over time.
- **`tests/`** — Holds standalone SQL tests for validating data conditions.

These directories separate different project responsibilities and support a maintainable project layout.

### 6. Documentation Generation

The project includes dbt-generated documentation that can be built from the project's model definitions and metadata.

The documentation interface provides a project navigation view, model details, and database exploration features. Where available, dbt's lineage graph can help users inspect dependencies between models.

## Project Workflow

The intended workflow for this project is:

1. Configure the dbt project and its BigQuery connection.
2. Define SQL models and organize them within the `models/` directory.
3. Configure schema metadata and applicable data tests.
4. Load CSV seed data into BigQuery when required.
5. Execute the configured dbt models and tests.
6. Generate and review the project documentation.
7. Maintain the project files using Git and publish the repository on GitHub.

## Project Outputs

The project is designed to produce the following outputs when the corresponding dbt commands execute successfully:

- **Warehouse models:** Tables or views created in BigQuery according to the configured model materializations.
- **Seed relations:** BigQuery relations created from the configured CSV seed files.
- **Data quality results:** Test results indicating whether the configured data conditions pass or fail.
- **Project documentation:** A browsable documentation site containing model metadata and, where supported by the project definitions, lineage information.

The exact relations created, their schemas, and the test results depend on the model SQL, project configuration, BigQuery connection, and commands executed.

## Running the Project

### Prerequisites

- Python and a compatible dbt Core installation.
- The appropriate dbt BigQuery adapter.
- Access to a Google Cloud project with the required BigQuery permissions.
- A configured dbt profile for BigQuery.

### Common Commands

Run these commands from the project directory after configuring the BigQuery connection.

```bash
# Install dependencies declared by the project, if applicable
dbt deps

# Check the project configuration and connection
dbt debug

# Execute the dbt models
dbt run

# Load the CSV seed data
dbt seed

# Execute configured data tests
dbt test

# Generate the documentation artifacts
dbt docs generate

# Serve the generated documentation locally
dbt docs serve
```

Run `dbt seed` before models that depend on seed relations. Use the commands appropriate to the dependencies and configuration of your project.

