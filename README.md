# plainerflow

A Python package for plain flow operations with SQLAlchemy integration.

## Installation

### From PyPI (recommended)

```bash
pip install plainerflow
```

### Manual Installation

If pip is not available in your environment, you can use the package by adding it to your Python path:

```python
import sys
sys.path.insert(0, "/path/to/the/right/plainerflow/subdirectory")
import plainerflow
```

## Using PlainerFlow

PlainerFlow provides a set of components designed to work together for data transformation pipelines. See the complete working example in [`pipeline_example.py`](pipeline_example.py) which demonstrates importing CSV data, transforming it using SQL, and validating the results.

### Complete Pipeline Example

The [`pipeline_example.py`](pipeline_example.py) script shows a full data transformation pipeline that:

1. **Connects to a database** using CredentialFinder (with PostgreSQL testcontainer fallback)
2. **Defines table references** using DBTable for customers, orders, and derived tables
3. **Loads CSV data** from the `readme_example_data/` directory
4. **Transforms data** using SQL queries organized in a FrostDict
5. **Validates results** using InLaw test classes
6. **Displays sample output** showing the transformation results

**To run the example:**

```bash
# Install dependencies
pip install plainerflow pandas great-expectations testcontainers

# Run the complete pipeline
python pipeline_example.py
```

**Expected output:**
```
=== PlainerFlow Pipeline Example Program ===
Step 1: Connecting to database...
✅ Connected to PostgreSQL test container

Step 2: Defining table references...
Will create tables: public.customers, public.orders, public.customer_summary

Step 3: Defining data loading SQL...

Step 4: Defining complete SQL pipeline...

Step 5: Executing complete SQL pipeline...
===== EXECUTING SQL LOOP =====
create_customers_DBTable: DROP TABLE IF EXISTS public.customers CASCADE;...
create_orders_DBTable: DROP TABLE IF EXISTS public.orders CASCADE;...
load_customers_data: INSERT INTO public.customers...
load_orders_data: INSERT INTO public.orders...
create_customer_summary: CREATE TABLE IF NOT EXISTS public.customer_summary AS...
create_order_metrics: CREATE TABLE IF NOT EXISTS order_metrics AS...
customer_summary_sample: SELECT * FROM public.customer_summary LIMIT 3
order_metrics_report: SELECT * FROM order_metrics ORDER BY order_count DESC
===== SQL LOOP COMPLETE =====

Step 6: Defining validation tests...

Step 7: Running data validation tests...
===== IN-LAW TESTS =====
Running: Customer summary should have same number of rows as customers
PASS
Running: Customer summary should have no null names  
PASS
Running: Active customers with orders should have positive total_spent
PASS
Summary: 3 passed

✅ Pipeline completed successfully!
   - Validation results: 3 passed, 0 failed
🎉 All validation tests passed!
```

### Key Components Explained

#### 1. CredentialFinder - Automatic Database Connection
```python
# Automatically detects your environment and provides a database connection
engine = CredentialFinder.detect_config(verbose=True)
# Supports: Spark/Databricks, Google Colab, .env files, SQLite fallback
```

#### 2. DBTable - Database Table References
```python
# Define table references before tables exist
customers_table = DBTable(database='analytics', table='customers')
orders_table = DBTable(database='analytics', table='orders')

# Create child tables with suffixes
backup_table = customers_table.make_child('backup')  # analytics.customers_backup

# Use in SQL queries via f-strings
sql = f"SELECT * FROM {customers_table} WHERE status = 'active'"
```

#### 3. FrostDict - Immutable SQL Configuration
```python
# Create frozen dictionary for SQL templates
sql_queries = FrostDict({
    'create_table': f"CREATE TABLE {table_name} AS SELECT ...",
    'update_data': f"UPDATE {table_name} SET ..."
})

# Keys cannot be reassigned once set
sql_queries['new_query'] = "SELECT 1"  # Works - new key
sql_queries['create_table'] = "SELECT 2"  # Raises FrozenKeyError
```

#### 4. SQLoopcicle - SQL Execution Loop
```python
# Execute SQL statements in order
SQLoopcicle.run_sql_loop(sql_queries, engine)

# Dry-run mode to preview without execution
SQLoopcicle.run_sql_loop(sql_queries, engine, is_just_print=True)
```

#### 5. InLaw - Data Validation Framework
```python
# Create validation test classes
class MyDataTest(InLaw):
    title = "Descriptive test name"
    
    @staticmethod
    def run(engine):
        sql = "SELECT COUNT(*) as row_count FROM my_table"
        gdf = InLaw.to_gx_dataframe(sql, engine)
        
        result = gdf.expect_column_values_to_be_between(
            column="row_count", min_value=1, max_value=1000
        )
        
        return True if result.success else f"Row count out of range: {gdf.iloc[0]['row_count']}"

# Run all validation tests
InLaw.run_all(engine)
```


## Basic Usage

For simpler use cases, you can use individual components:

```python
import plainerflow

# Just get a database connection
engine = plainerflow.CredentialFinder.detect_config()

# Define a table reference
my_table = plainerflow.DBTable(database='mydb', table='users')
print(f"Table reference: {my_table}")  # Output: mydb.users

# Create frozen configuration
config = plainerflow.FrostDict({'query': f'SELECT * FROM {my_table}'})

# Execute SQL
plainerflow.SQLoopcicle.run_sql_loop(config, engine)
```

## Dependencies

- SQLAlchemy >= 1.4.0

## Development

### Setting up development environment

1. Clone the repository:
```bash
git clone https://github.com/ftrotter/plainerflow.git
cd plainerflow
```

2. Create a virtual environment:
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install development dependencies:
```bash
pip install -e ".[dev]"
```

### Building the package

```bash
python -m build
```

### Uploading to PyPI

```bash
python -m twine upload dist/*
```

## License

This project is licensed under the CC0 1.0 Universal License - see the [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.
