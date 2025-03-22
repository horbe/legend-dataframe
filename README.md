# Legend DataFrames

A DSL in Pure language for modeling SELECT statements that work with Snowflake, DuckDB, BigQuery, Databricks, Redshift, and Postgres.

## Overview

This project implements a Domain Specific Language (DSL) in Pure language for modeling SQL SELECT statements that can be executed against Snowflake, DuckDB, BigQuery, Databricks, Redshift, and Postgres databases. The DSL provides a unified interface for writing queries that can be translated to the appropriate SQL syntax for each database system.

## Features

- Model SELECT statements in Pure language
- Support for common SQL operations (filtering, grouping, aggregation, etc.)
- Type-safe operations with lambda expressions
- Inline TDS definition for quick data creation
- Window functions with frame support
- Compatible with Snowflake, DuckDB, BigQuery, Databricks, and Redshift
- Automatic SQL generation for the target database
- Support for database-specific features (e.g., Databricks QUALIFY and CLUSTER BY clauses)

## Project Structure

```
legend-dataframe/
├── src/
│   ├── main/
│   │   └── resources/
│   │       └── pure/
│   │           └── dsl/
│   │               ├── dataframe/           # Core DSL definitions
│   │               │   ├── dataframe.pure   # Main DataFrame operations
│   │               │   ├── frameUtils.pure  # Window frame utilities
│   │               │   └── metamodel/       # DataFrame metamodel classes
│   │               ├── snowflake/           # Snowflake-specific implementations
│   │               ├── duckdb/              # DuckDB-specific implementations
│   │               ├── bigquery/            # BigQuery-specific implementations
│   │               ├── databricks/          # Databricks-specific implementations
│   │               └── redshift/            # Redshift-specific implementations
│   └── test/
│       └── resources/
│           └── pure/
│               └── dsl/
│                   └── tests/               # Test cases
└── README.md
```

## Usage

### Basic DataFrame Operations

```pure
// Create a DataFrame from a table
let df = table('employees');

// Filter rows using type-safe filter function
let filtered = $df->filter({x | $x.salary > 50000});

// Select specific columns
let selected = $df->select({x | [~id, ~name, ~salary]});

// Group by and aggregate
let summary = $df->groupBy([~department])
                ->extend([
                   avg(~salary)->as('avg_salary'),
                   count()->as('employee_count')
                ]);
```

### Inline TDS Definition

```pure
// Define a DataFrame inline with TDS syntax
let df = #TDS
         id, name, salary, department
         1, 'John', 50000, 'Engineering'
         2, 'Jane', 60000, 'Sales'
         3, 'Bob', 55000, 'Engineering'
        #;

// Chain operations on the inline TDS
let highPaid = $df->filter({x | $x.salary > 55000})
                  ->select({x | [~name, ~department]});
```

### Window Functions

DataFrame supports window functions using the `over` function:

```pure
// Basic window function with partition by
let avgSalaryByDept = $df
   ->extend([
      avg(col('salary'))->over(~department)->as('avg_salary')
   ]);

// Window function with ordering
let salaryRank = $df
   ->extend([
      rank()->over([~department], [asc(~salary)])->as('salary_rank')
   ]);

// Window function with frame
let runningTotal = $df
   ->extend([
      sum(col('amount'))->over([~department], [asc(~date)], rows(unbounded(), 0))->as('running_total')
   ]);
```

### SQL Generation

```pure
// Generate SQL for different database systems
let duckSQL = $df->toDuckDBSQL();
let snowflakeSQL = $df->toSnowflakeSQL();
let bigQuerySQL = $df->toBigQuerySQL();
let databricksSQL = $df->toDatabricksSQL();
let redshiftSQL = $df->toRedshiftSQL();
let postgresSQL = $df->toPostgresSQL();

// Generic SQL generation with database parameter
let sql = $df->toSQL(DatabaseType.SNOWFLAKE);
```

## License

Apache License 2.0
