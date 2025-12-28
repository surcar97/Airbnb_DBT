# Airbnb dbt Project

A comprehensive dbt project for transforming Airbnb listing, host, and review data into a dimensional data model. This project demonstrates best practices for data transformation, testing, documentation, and multi-warehouse compatibility.

## 📋 Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Supported Warehouses](#supported-warehouses)
- [Documentation](#documentation)
- [Common Commands](#common-commands)
- [Testing](#testing)
- [Dependencies](#dependencies)
- [Resources](#resources)
- [Contributing](#contributing)

## 🎯 Overview

This dbt project transforms raw Airbnb data into a clean, tested, and documented dimensional model. It includes:

- **9 models** organized into source, dimension, fact, and mart layers
- **Comprehensive data tests** using dbt's built-in tests and dbt_expectations
- **Documentation** with descriptions, docs blocks, and exposures
- **Multi-warehouse support** for PostgreSQL, SQL Server, Snowflake, and Databricks
- **Snapshots** for tracking slowly changing dimensions
- **Seeds** for reference data (full moon dates)

### Data Model

```
sources (raw_hosts, raw_listings, raw_reviews)
  └─> src_* (ephemeral staging models)
       └─> dim_* (dimension tables/views)
            └─> dim_listings_w_hosts (denormalized dimension)
       └─> fct_reviews (incremental fact table)
            └─> mart_fullmoon_reviews (business mart)
```

## 📁 Project Structure

```
airbnb_dbt/
├── analyses/          # Ad-hoc analysis queries
├── assets/            # Images and documentation assets
├── docs/              # Project documentation
│   ├── PROJECT_INVENTORY.txt
│   └── INSTALLATION.md
├── macros/            # Reusable SQL macros
├── models/            # dbt models (SQL files + YAML configs)
│   ├── dim/          # Dimension models
│   ├── fct/          # Fact models
│   ├── mart/         # Business mart models
│   └── src/          # Source staging models
├── profiles_templates/ # Connection profile templates
├── seeds/            # CSV seed files
├── snapshots/        # Snapshot definitions
├── tests/            # Custom test definitions
├── dbt_project.yml   # Project configuration
├── packages.yml      # dbt package dependencies
└── README.md         # This file
```

## 🚀 Quick Start

### Prerequisites

- Python 3.8 or higher
- Access to a supported data warehouse (PostgreSQL, SQL Server, Snowflake, or Databricks)
- Warehouse credentials and connection details

### Installation

1. **Clone this repository:**
   ```bash
   git clone <your-repo-url>
   cd airbnb_dbt
   ```

2. **Create a Python virtual environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dbt Core and your adapter:**
   ```bash
   # For PostgreSQL
   pip install dbt-postgres

   # For SQL Server
   pip install dbt-sqlserver

   # For Snowflake
   pip install dbt-snowflake

   # For Databricks
   pip install dbt-databricks
   ```

4. **Install project dependencies:**
   ```bash
   dbt deps
   ```

5. **Configure your connection:**
   - Copy `.env.example` to `.env` and fill in your credentials
   - Copy `profiles_templates/profiles.yml` to `~/.dbt/profiles.yml` (or `%USERPROFILE%\.dbt\profiles.yml` on Windows)
   - Uncomment and configure the profile matching your warehouse
   - Update the `profile` field in `dbt_project.yml` if needed

6. **Test your connection:**
   ```bash
   dbt debug
   ```

7. **Run the project:**
   ```bash
   dbt seed    # Load seed data
   dbt run     # Build all models
   dbt test    # Run all tests
   ```

For detailed installation instructions, see [docs/INSTALLATION.md](docs/INSTALLATION.md).

## 🗄️ Supported Warehouses

This project supports four major data warehouses:

| Warehouse | Adapter Package | Documentation |
|-----------|----------------|---------------|
| **PostgreSQL** | `dbt-postgres` | [dbt Postgres Setup](https://docs.getdbt.com/docs/core/connect-data-platform/postgres-setup) |
| **SQL Server** | `dbt-sqlserver` | [dbt SQL Server Setup](https://docs.getdbt.com/docs/core/connect-data-platform/sqlserver-setup) |
| **Snowflake** | `dbt-snowflake` | [dbt Snowflake Setup](https://docs.getdbt.com/docs/core/connect-data-platform/snowflake-setup) |
| **Databricks** | `dbt-databricks` | [dbt Databricks Setup](https://docs.getdbt.com/docs/core/connect-data-platform/databricks-setup) |

Connection templates with detailed comments are available in `profiles_templates/profiles.yml`.

### Warehouse-Specific Notes

**Snowflake (Original):** This project was originally developed for Snowflake. Some SQL functions may need adaptation for other warehouses:
- `NVL()` → Use `COALESCE()` for PostgreSQL/Databricks
- `DATEADD()` → Use warehouse-specific date functions
- `TO_DATE()` → Use warehouse-specific casting

## 📚 Documentation

### Generate Documentation Site

```bash
dbt docs generate
dbt docs serve
```

This will:
- Generate a static documentation site
- Include model lineage graphs
- Display all tests, descriptions, and metadata
- Open in your browser at `http://localhost:8080`

### Project Inventory

A complete project inventory is available at [docs/PROJECT_INVENTORY.txt](docs/PROJECT_INVENTORY.txt), including:
- Model inventory with dependencies
- Source definitions
- Test coverage
- Macro documentation
- Connection mapping details

## 🔧 Common Commands

```bash
# Connection and setup
dbt debug                    # Test connection and configuration
dbt deps                     # Install dbt packages

# Data operations
dbt seed                     # Load seed files
dbt run                      # Build all models
dbt run --select dim_*       # Build specific models
dbt build                    # Run models and tests together

# Testing
dbt test                     # Run all tests
dbt test --select dim_listings_cleansed  # Test specific model

# Documentation
dbt docs generate            # Generate documentation artifacts
dbt docs serve               # Serve documentation site

# Snapshots
dbt snapshot                 # Run snapshots

# Compilation
dbt compile                  # Compile SQL without running
dbt show <model_name>        # Show compiled SQL for a model
```

## 🧪 Testing

This project includes comprehensive data quality tests:

- **Generic tests:** `unique`, `not_null`, `relationships`, `accepted_values`
- **Custom generic tests:** `positive_values`, `minimum_row_count`
- **Package tests:** `dbt_expectations` for advanced validations
- **Singular tests:** Custom SQL tests for business rules
- **Unit tests:** Model-level unit tests for `mart_fullmoon_reviews`

Test failures are stored in the `_test_failures` schema (configurable in `dbt_project.yml`).

## 📦 Dependencies

This project uses the following dbt packages (defined in `packages.yml`):

- **dbt-labs/dbt_utils** (v1.3.1): Utility macros for common operations
- **metaplane/dbt_expectations** (v0.10.9): Great Expectations-style data quality tests

Install with: `dbt deps`

## 🔗 Resources

### Official dbt Documentation

- [dbt Core Documentation](https://docs.getdbt.com/docs/introduction)
- [dbt Commands Reference](https://docs.getdbt.com/reference/dbt-commands)
- [Connection Profiles](https://docs.getdbt.com/docs/core/connect-data-platform/connection-profiles)
- [Model Configuration](https://docs.getdbt.com/docs/build/models)
- [Testing](https://docs.getdbt.com/docs/build/tests)
- [Documentation](https://docs.getdbt.com/docs/collaborate/documentation)
- [Snapshots](https://docs.getdbt.com/docs/build/snapshots)

### Adapter Documentation

- [PostgreSQL Adapter](https://docs.getdbt.com/docs/core/connect-data-platform/postgres-setup)
- [SQL Server Adapter](https://docs.getdbt.com/docs/core/connect-data-platform/sqlserver-setup)
- [Snowflake Adapter](https://docs.getdbt.com/docs/core/connect-data-platform/snowflake-setup)
- [Databricks Adapter](https://docs.getdbt.com/docs/core/connect-data-platform/databricks-setup)

### Package Documentation

- [dbt_utils](https://github.com/dbt-labs/dbt-utils)
- [dbt_expectations](https://github.com/calogica/dbt-expectations)

## 🤝 Contributing

Contributions are welcome! Please ensure:
- All tests pass (`dbt test`)
- Documentation is updated
- Code follows dbt best practices
- No secrets are committed

---

**Questions or Issues?** Check the [INSTALLATION.md](docs/INSTALLATION.md) for troubleshooting or refer to the official dbt documentation.

