# Installation Guide

Complete step-by-step installation instructions for the Airbnb dbt project, based on official dbt documentation.

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Python Virtual Environment Setup](#python-virtual-environment-setup)
3. [Installing dbt Core and Adapters](#installing-dbt-core-and-adapters)
4. [Project Setup](#project-setup)
5. [Connection Configuration](#connection-configuration)
6. [Verifying Installation](#verifying-installation)
7. [Running the Project](#running-the-project)
8. [Troubleshooting](#troubleshooting)
9. [References](#references)

## Prerequisites

### Python Version

dbt Core requires **Python 3.8 or higher**. Check your Python version:

```bash
python --version
# or
python3 --version
```

If Python is not installed or the version is too old:
- **Windows:** Download from [python.org](https://www.python.org/downloads/)
- **macOS:** Use Homebrew: `brew install python3`
- **Linux:** Use your package manager: `sudo apt install python3` (Ubuntu/Debian)

### Data Warehouse Access

You need:
- Access to one of the supported warehouses (PostgreSQL, SQL Server, Snowflake, or Databricks)
- Valid credentials (username, password, connection details)
- Appropriate permissions to create schemas, tables, and views

### System Dependencies

**For SQL Server:**
- ODBC Driver for SQL Server must be installed
- Check installed drivers:
  - **Windows:** ODBC Data Sources (odbcad32.exe)
  - **Linux:** `odbcinst -q -d`
  - **macOS:** Check `/usr/local/etc/odbcinst.ini`

**For PostgreSQL:**
- No additional system dependencies required (uses psycopg2)

**For Snowflake:**
- No additional system dependencies required

**For Databricks:**
- No additional system dependencies required

## Python Virtual Environment Setup

Using a virtual environment is recommended to isolate project dependencies.

### Create Virtual Environment

```bash
# Navigate to project directory
cd airbnb_dbt

# Create virtual environment
python -m venv venv
# or
python3 -m venv venv
```

### Activate Virtual Environment

**Windows (PowerShell):**
```powershell
.\venv\Scripts\Activate.ps1
```

**Windows (Command Prompt):**
```cmd
venv\Scripts\activate.bat
```

**macOS/Linux:**
```bash
source venv/bin/activate
```

You should see `(venv)` in your terminal prompt when activated.

### Deactivate (when done)

```bash
deactivate
```

## Installing dbt Core and Adapters

### Install dbt Core

dbt Core is installed automatically when you install an adapter. However, you can install it separately:

```bash
pip install dbt-core
```

### Install Warehouse Adapter

Install **only the adapter** for your target warehouse:

**PostgreSQL:**
```bash
pip install dbt-postgres
```
Reference: [dbt Postgres Setup](https://docs.getdbt.com/docs/core/connect-data-platform/postgres-setup)

**SQL Server:**
```bash
pip install dbt-sqlserver
```
Reference: [dbt SQL Server Setup](https://docs.getdbt.com/docs/core/connect-data-platform/sqlserver-setup)

**Snowflake:**
```bash
pip install dbt-snowflake
```
Reference: [dbt Snowflake Setup](https://docs.getdbt.com/docs/core/connect-data-platform/snowflake-setup)

**Databricks:**
```bash
pip install dbt-databricks
```
Reference: [dbt Databricks Setup](https://docs.getdbt.com/docs/core/connect-data-platform/databricks-setup)

### Verify Installation

```bash
dbt --version
```

You should see output showing dbt version and installed adapters.

## Project Setup

### 1. Install Project Dependencies

This project uses dbt packages defined in `packages.yml`. Install them:

```bash
dbt deps
```

This will:
- Download `dbt-labs/dbt_utils` (v1.3.1)
- Download `metaplane/dbt_expectations` (v0.10.9)
- Install them to `dbt_packages/` directory

### 2. Set Up Environment Variables

**Create `.env` file:**

```bash
# Copy the example file
cp .env.example .env
```

**Edit `.env` and fill in values for your warehouse:**

Open `.env` in a text editor and uncomment/fill in the section for your warehouse. For example, for PostgreSQL:

```bash
DBT_POSTGRES_HOST=localhost
DBT_POSTGRES_PORT=5432
DBT_POSTGRES_USER=your_username
DBT_POSTGRES_PASSWORD=your_password
DBT_POSTGRES_DBNAME=airbnb
DBT_POSTGRES_SCHEMA=dev
DBT_POSTGRES_THREADS=4
```

**Load environment variables:**

**Linux/macOS:**
```bash
export $(cat .env | xargs)
# or use a tool like python-dotenv
```

**Windows (PowerShell):**
```powershell
Get-Content .env | ForEach-Object {
    $name, $value = $_.split('=')
    Set-Content env:\$name $value
}
```

**Windows (Command Prompt):**
```cmd
# Use setx or a tool like python-dotenv
```

**Alternative: Use python-dotenv**

```bash
pip install python-dotenv
```

Then create a script or use it in your shell configuration.

## Connection Configuration

### 1. Locate dbt Profiles Directory

dbt looks for `profiles.yml` in:
- **Linux/macOS:** `~/.dbt/profiles.yml`
- **Windows:** `%USERPROFILE%\.dbt\profiles.yml` or `C:\Users\<username>\.dbt\profiles.yml`

Create the directory if it doesn't exist:

```bash
# Linux/macOS
mkdir -p ~/.dbt

# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path $env:USERPROFILE\.dbt
```

### 2. Copy Profile Template

```bash
# Linux/macOS
cp profiles_templates/profiles.yml ~/.dbt/profiles.yml

# Windows (PowerShell)
Copy-Item profiles_templates/profiles.yml $env:USERPROFILE\.dbt\profiles.yml
```

### 3. Configure Profile

Open `~/.dbt/profiles.yml` (or `%USERPROFILE%\.dbt\profiles.yml` on Windows) in a text editor.

**Uncomment and configure ONE profile** matching your warehouse. For example, for PostgreSQL:

```yaml
airbnb:
  target: dev
  outputs:
    dev:
      type: postgres
      host: "{{ env_var('DBT_POSTGRES_HOST') }}"
      port: "{{ env_var('DBT_POSTGRES_PORT') | int }}"
      user: "{{ env_var('DBT_POSTGRES_USER') }}"
      password: "{{ env_var('DBT_POSTGRES_PASSWORD') }}"
      dbname: "{{ env_var('DBT_POSTGRES_DBNAME') }}"
      schema: "{{ env_var('DBT_POSTGRES_SCHEMA', 'dev') }}"
      threads: "{{ env_var('DBT_POSTGRES_THREADS', '4') | int }}"
```

**Important:**
- The profile name (`airbnb`) must match the `profile` field in `dbt_project.yml`
- Comment out or remove other warehouse profiles
- Ensure all environment variables are set

### 4. Verify Profile Name in Project

Check `dbt_project.yml` to ensure the profile name matches:

```yaml
name: 'airbnb'
profile: 'airbnb'  # Must match profile name in profiles.yml
```

## Verifying Installation

### Test Connection

```bash
dbt debug
```

This command checks:
- ✅ dbt installation and version
- ✅ Profile configuration
- ✅ Connection to warehouse
- ✅ Required permissions

**Expected output:**
```
Running with dbt=1.x.x
...
Connection test: [OK connection ok]
```

If you see errors, see the [Troubleshooting](#troubleshooting) section.

### List Models

```bash
dbt list
```

This shows all models, tests, seeds, and snapshots in the project.

## Running the Project

### 1. Load Seed Data

```bash
dbt seed
```

This loads `seeds/seed_full_moon_dates.csv` into your warehouse.

### 2. Run Models

**Run all models:**
```bash
dbt run
```

**Run specific models:**
```bash
dbt run --select dim_*
dbt run --select fct_reviews
```

**Run models and tests together:**
```bash
dbt build
```

### 3. Run Tests

**Run all tests:**
```bash
dbt test
```

**Run tests for specific model:**
```bash
dbt test --select dim_listings_cleansed
```

### 4. Generate Documentation

```bash
# Generate documentation artifacts
dbt docs generate

# Serve documentation site (opens in browser)
dbt docs serve
```

The documentation site will open at `http://localhost:8080` and includes:
- Model lineage graphs
- Test results
- Column descriptions
- Source definitions

### 5. Run Snapshots

```bash
dbt snapshot
```

This creates initial snapshots of raw source tables.

## Troubleshooting

### Profile Not Found

**Error:** `Profile "airbnb" not found in profiles.yml`

**Solutions:**
1. Verify `profiles.yml` exists in `~/.dbt/` (or `%USERPROFILE%\.dbt\` on Windows)
2. Check profile name matches `profile` field in `dbt_project.yml`
3. Verify YAML syntax is correct (use a YAML validator)
4. Check file permissions

### Adapter Not Installed

**Error:** `No adapter named 'postgres' found`

**Solutions:**
1. Install the correct adapter: `pip install dbt-postgres` (or your adapter)
2. Verify installation: `dbt --version` should show the adapter
3. Ensure virtual environment is activated
4. Reinstall if needed: `pip install --upgrade dbt-postgres`

### Connection/Authentication Errors

**Error:** `Connection test failed`

**Solutions:**
1. Verify environment variables are set: `echo $DBT_POSTGRES_HOST` (Linux/macOS)
2. Check credentials are correct
3. Verify network connectivity to warehouse
4. Check firewall rules
5. For SQL Server: Verify ODBC driver is installed
6. For Snowflake: Check account identifier format
7. For Databricks: Verify HTTP path and token are correct

### Permission Errors

**Error:** `Insufficient privileges`

**Solutions:**
1. Ensure user has `CREATE SCHEMA` permission
2. Ensure user has `CREATE TABLE` and `CREATE VIEW` permissions
3. For Snowflake: Verify role has access to database and warehouse
4. For Databricks: Check cluster/warehouse permissions

### Environment Variables Not Loading

**Solutions:**
1. Verify `.env` file exists and has correct format (no spaces around `=`)
2. Load variables manually: `export DBT_POSTGRES_HOST=localhost`
3. Use `python-dotenv` for automatic loading
4. Check shell configuration (`.bashrc`, `.zshrc`, etc.)

### SQL Syntax Errors (Warehouse Compatibility)

**Error:** `Syntax error: unexpected token`

**Solutions:**
1. Some SQL functions are warehouse-specific:
   - `NVL()` (Snowflake) → Use `COALESCE()` for PostgreSQL/Databricks
   - `DATEADD()` → Use warehouse-specific date functions
   - `TO_DATE()` → Use warehouse-specific casting
2. Check [PROJECT_INVENTORY.txt](PROJECT_INVENTORY.txt) for warehouse-specific notes
3. Adapt SQL in models if needed for your warehouse

### Package Installation Issues

**Error:** `Could not find package`

**Solutions:**
1. Verify internet connection
2. Check `packages.yml` syntax
3. Run `dbt deps --verbose` for detailed error messages
4. Try clearing cache: `rm -rf dbt_packages/` then `dbt deps`

### Python Version Issues

**Error:** `Python version X.Y.Z is not supported`

**Solutions:**
1. Upgrade Python to 3.8 or higher
2. Use `pyenv` or similar tool to manage Python versions
3. Verify virtual environment uses correct Python version

## References

### Official dbt Documentation

- **dbt Core:** [docs.getdbt.com](https://docs.getdbt.com/docs/introduction)
- **Installation:** [Install dbt Core](https://docs.getdbt.com/docs/get-started/installation)
- **Connection Profiles:** [Connection Profiles](https://docs.getdbt.com/docs/core/connect-data-platform/connection-profiles)
- **Commands Reference:** [dbt Commands](https://docs.getdbt.com/reference/dbt-commands)

### Adapter-Specific Documentation

- **PostgreSQL:** [Postgres Setup](https://docs.getdbt.com/docs/core/connect-data-platform/postgres-setup)
- **SQL Server:** [SQL Server Setup](https://docs.getdbt.com/docs/core/connect-data-platform/sqlserver-setup)
- **Snowflake:** [Snowflake Setup](https://docs.getdbt.com/docs/core/connect-data-platform/snowflake-setup)
- **Databricks:** [Databricks Setup](https://docs.getdbt.com/docs/core/connect-data-platform/databricks-setup)

### Additional Resources

- **dbt Community:** [dbt Discourse](https://discourse.getdbt.com/)
- **dbt Slack:** [getdbt.slack.com](https://www.getdbt.com/community/join-the-community)
- **Project Inventory:** [PROJECT_INVENTORY.txt](PROJECT_INVENTORY.txt)

---

**Still having issues?** Check the official dbt documentation or the dbt community forums for help.

