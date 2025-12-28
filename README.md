# Airbnb dbt Project

A comprehensive dbt project for transforming Airbnb listing, host, and review data into a dimensional data model. This project demonstrates best practices for data transformation, testing, documentation, and multi-warehouse compatibility.

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Quick Start](#quick-start)
- [Supported Warehouses](#supported-warehouses)
- [Documentation](#documentation)
- [Sources and Freshness](#sources-and-freshness)
- [Common Commands](#common-commands)
- [Testing](#testing)
- [Dependencies](#dependencies)
- [Resources](#resources)
- [Contributing](#contributing)

## Overview

This dbt project transforms raw Airbnb data into a clean, tested, and documented dimensional model. The project includes 9 models organized into source, dimension, fact, and mart layers, with comprehensive data tests using dbt's built-in tests and dbt_expectations.

**Key Features:**
- 9 models organized into source, dimension, fact, and mart layers
- Comprehensive data tests using dbt's built-in tests and dbt_expectations
- Documentation with descriptions, docs blocks, and exposures
- Multi-warehouse support for PostgreSQL, SQL Server, Snowflake, and Databricks
- Snapshots for tracking slowly changing dimensions
- Seeds for reference data (full moon dates)

### Data Model

The project follows a dimensional modeling approach with the following structure:

```
sources (raw_hosts, raw_listings, raw_reviews)
  └─> src_* (ephemeral staging models)
       └─> dim_* (dimension tables/views)
            └─> dim_listings_w_hosts (denormalized dimension)
       └─> fct_reviews (incremental fact table)
            └─> mart_fullmoon_reviews (business mart)
```

## Project Structure

The project is organized into standard dbt directories:

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

## Quick Start

### Prerequisites

Before getting started, make sure you have:
- Python 3.8 or higher installed
- Access to a supported data warehouse (PostgreSQL, SQL Server, Snowflake, or Databricks)
- Warehouse credentials and connection details

### Installation

Follow these steps to set up the project:

1. **Clone this repository:**
   ```bash
   git clone https://github.com/surcar97/Airbnb_DBT.git
   cd airbnb_dbt
   ```

2. **Create a Python virtual environment:**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dbt Core and your adapter:**
   
   Choose the adapter that matches your warehouse:
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

## Supported Warehouses

This project supports four major data warehouses. Connection templates with detailed comments are available in `profiles_templates/profiles.yml`.

| Warehouse | Adapter Package | Documentation |
|-----------|----------------|---------------|
| PostgreSQL | `dbt-postgres` | [dbt Postgres Setup](https://docs.getdbt.com/docs/core/connect-data-platform/postgres-setup) |
| SQL Server | `dbt-sqlserver` | [dbt SQL Server Setup](https://docs.getdbt.com/docs/core/connect-data-platform/sqlserver-setup) |
| Snowflake | `dbt-snowflake` | [dbt Snowflake Setup](https://docs.getdbt.com/docs/core/connect-data-platform/snowflake-setup) |
| Databricks | `dbt-databricks` | [dbt Databricks Setup](https://docs.getdbt.com/docs/core/connect-data-platform/databricks-setup) |

### Warehouse-Specific Notes

This project was originally developed for Snowflake. Some SQL functions may need adaptation for other warehouses:

- `NVL()` → Use `COALESCE()` for PostgreSQL/Databricks
- `DATEADD()` → Use warehouse-specific date functions
- `TO_DATE()` → Use warehouse-specific casting

## Documentation

### Generate Documentation Site

Generate and view the project documentation:

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

## Sources and Freshness

### What are Sources?

Sources in dbt represent external data tables that your dbt project depends on but doesn't create or manage. They're defined in YAML files and allow you to:
- Document external data dependencies
- Test source data quality
- Monitor data freshness
- Build lineage graphs that show where your data comes from

### Sources in This Project

This project defines sources in `models/sources.yml`:

**Source Name:** `airbnb`  
**Source Schema:** `raw` (configured in your warehouse)

**Source Tables:**
1. **listings** (identifier: `raw_listings`)
   - Column-level tests on `room_type` and `price`
   
2. **hosts** (identifier: `raw_hosts`)
   - No freshness rules defined
   
3. **reviews** (identifier: `raw_reviews`)
   - Freshness monitoring enabled (see below)

### How Freshness Works

Freshness monitoring in dbt checks when source data was last updated to ensure your pipelines are working with current data. It helps you:
- Detect stale data pipelines
- Identify upstream data loading issues
- Set up alerts for data freshness violations

**How it works:**
1. dbt queries the `loaded_at_field` (timestamp column) in your source table
2. Compares the most recent timestamp to the current time
3. Warns or errors if data is older than specified thresholds

### Freshness Implementation in This Project

The `reviews` source table has freshness monitoring configured in `models/sources.yml`:

```yaml
- name: reviews
  identifier: raw_reviews
  config:
    loaded_at_field: date          # Column that tracks when data was loaded
    freshness:
      warn_after: {count: 1, period: hour}    # Warn if data is > 1 hour old
      # error_after: {count: 24, period: hour}  # Error if > 24 hours (commented out)
```

**Configuration Details:**
- **`loaded_at_field: date`** - The column in `raw_reviews` that contains the timestamp of when each record was loaded
- **`warn_after`** - Issues a warning if the most recent data is older than 1 hour
- **`error_after`** - Would error if data is older than 24 hours (currently commented out)

**Available Periods:**
- `minute`, `hour`, `day`, `week`, `month`

### Checking Freshness

Run freshness checks with:

```bash
# Check freshness for all sources
dbt source freshness

# Check freshness for a specific source
dbt source freshness --select source:airbnb

# Check freshness for a specific table
dbt source freshness --select source:airbnb.reviews
```

**Output Example:**
```
Found 1 source, 3 tables, 1 freshness check
Pass: reviews [warn after 1 hour] (0 hours old)
```

### Source Files Location

- **Source definitions:** `models/sources.yml`
- **Source documentation:** See `docs/PROJECT_INVENTORY.txt` for complete source inventory

### Adding Freshness to Other Sources

To add freshness monitoring to other sources, update `models/sources.yml`:

```yaml
- name: hosts
  identifier: raw_hosts
  config:
    loaded_at_field: updated_at    # Use the timestamp column in your table
    freshness:
      warn_after: {count: 6, period: hour}
      error_after: {count: 24, period: hour}
```

**Best Practices:**
- Use a column that reliably tracks when data was last updated
- Set `warn_after` to catch issues early
- Set `error_after` to fail builds if data is too stale
- Document freshness expectations in your source YAML

For more information, see the [dbt Sources documentation](https://docs.getdbt.com/docs/build/sources).

## Common Commands

Here are the most commonly used dbt commands for this project:

**Connection and setup:**
```bash
dbt debug                    # Test connection and configuration
dbt deps                     # Install dbt packages
```

**Data operations:**
```bash
dbt seed                     # Load seed files
dbt run                      # Build all models
dbt run --select dim_*       # Build specific models
dbt build                    # Run models and tests together
```

**Source freshness:**
```bash
dbt source freshness         # Check freshness for all sources
dbt source freshness --select source:airbnb  # Check specific source
dbt source freshness --select source:airbnb.reviews  # Check specific table
```

**Testing:**
```bash
dbt test                     # Run all tests
dbt test --select dim_listings_cleansed  # Test specific model
```

**Documentation:**
```bash
dbt docs generate            # Generate documentation artifacts
dbt docs serve               # Serve documentation site
```

**Snapshots:**
```bash
dbt snapshot                 # Run snapshots
```

**Compilation:**
```bash
dbt compile                  # Compile SQL without running
dbt show <model_name>        # Show compiled SQL for a model
```

## Testing

This project includes comprehensive data quality tests:

- **Generic tests:** `unique`, `not_null`, `relationships`, `accepted_values`
- **Custom generic tests:** `positive_values`, `minimum_row_count`
- **Package tests:** `dbt_expectations` for advanced validations
- **Singular tests:** Custom SQL tests for business rules
- **Unit tests:** Model-level unit tests for `mart_fullmoon_reviews`

Test failures are stored in the `_test_failures` schema (configurable in `dbt_project.yml`).

## Dependencies

This project uses the following dbt packages (defined in `packages.yml`):

- **dbt-labs/dbt_utils** (v1.3.1): Utility macros for common operations
- **metaplane/dbt_expectations** (v0.10.9): Great Expectations-style data quality tests

Install with: `dbt deps`

## Resources

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

## Contributing

Contributions are welcome! Please ensure:
- All tests pass (`dbt test`)
- Documentation is updated
- Code follows dbt best practices
- No secrets are committed

---

**Questions or Issues?** Check the [INSTALLATION.md](docs/INSTALLATION.md) for troubleshooting or refer to the official dbt documentation.
