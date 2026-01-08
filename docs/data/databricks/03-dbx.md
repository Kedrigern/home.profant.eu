---
title: Databricks
icon: simple/databricks
---

Databricks is a commercial solution called a lakehouse (combination of data lake and data warehouse). Mostly known for:

- separating storage and compute
- storage:
	- data lake:
		- can easily store raw data (any kind of file)
		- uses cloud object/blob storage, so it is cheap even for big data
	- data warehouse:
		- holds structured data (primarily in delta tables)
- compute:
	- queries are possible in SQL and Python (lazy DataFrame)
	- Databricks prepares clusters for you
	- there is an option for serverless compute (can be tricky to manage costs)

**Control plane** (orchestration) and **data plane** (your data) are separated. So your data are stored in your cloud and Databricks only manages metadata and compute, but cluster itself runs in your cloud account too.

Jobs (workflows): automate data pipelines.
Lakeflow connect: integrates with various data sources (files, cloud storage, Kafka, etc.).
Supports batch, incremental, and streaming modes.

Your data are organized in **workspace**. Main entities in workspace are:
- **notebooks**: interactive code cells, syntax: Python, SQL, Markdown
- **jobs**: workflows, there are divided into **tasks**.
- **tables**: structured data

## Unity catalog and privileges

Centralized data governance and access control.

DBX Roles:

- account admin: manage metastore, workspaces and users
- metastore admin: create catalogs, manage data object
- workspace admin
- owner: table, schema, function

Control plane: store metadata, provided by DBX
Data plane: store data, managed by user (cloud storage like S3, ADLS, GCS)

3 namespace: `<catalog>.<schema>.<table>`

To access a specific table, the user must be granted `SELECT` on the table itself, `USE SCHEMA` on the containing schema, and `USE CATALOG` on the parent catalog. This provides just enough access for read operations without overprovisioning.

**Catalog Explorer**: web UI for browsing catalogs, schemas, tables, and views.

### Delta sharing

A Delta Sharing identifier is a unique string used in Databricks-to-Databricks sharing to identify a recipient's Unity Catalog metastore. This identifier allows the data provider to grant access to shared data.

The format of the sharing identifier is: `<cloud>:<region>:<uuid>`

Example: `aws:us-west-2:19a84bee-54bc-43a2-87de-023d0ec16016`

In this example:

- `aws`: represents the cloud provider (Amazon Web Services).
- `us-west-2`: represents the specific AWS region.
- `19a84bee-54bc-43a2-87de-023d0ec16016`: is the Universally Unique Identifier (UUID) of the recipient's Unity Catalog metastore.

Recipients can obtain their sharing identifier from their Databricks workspace using Catalog Explorer or by running a SQL query like `SELECT CURRENT_METASTORE();`. This identifier is then provided to the data provider, who uses it to create a recipient and grant access to shares.

## Notebooks

DBX contains interactive notebooks:

- Markdown, SQL, python cells, start your cell with `%md`, `%sql`, `%python` to change it
- Also you can use `%run /path/to/other/notebook` to include another notebook
- Or `%fs ls '/dir'`
- predefined variables:
  - `dbutils`: utilities for file system, secrets, widgets, etc., for example `dbutils.fs.ls('/mnt/data')`. `dbutils.help()` for more info.
  - `spark`: SparkSession
  - `sqlContext`: SQL context
- predefined function:
  - `display()`: display DataFrame or visualization 
  - `displayHTML()`: render HTML content
  - `spark.read` and `spark.write`: read and write data
- contains interative debugger
- variable explorer
- version history
- limit for output is 30 MB

**Databricks Connect** is a client library for the Databricks Runtime that allows you to connect popular IDEs.

## DevOps

### Assets bundle (DAB)

Databricks Assets Bundles (DABs):

- code in yaml
- exact definition of Databricks resources 

```yaml
# databricks.yml

bundle:
  name: <name>

include:
  - resources/*.yml

targets:
  dev:
    mode: development
    default: true
    workspace:
      host: {{workspace_host}}
  prd:
    mode: production
    workspace:
      host: {{workspace_host}}
```

```yaml
resources:
    jobs:
```

### Git

Version control for notebooks and code. Cannot delete branch.

## Data streaming

Continously ingests new data as it arrives. Suitable for real-time data processing. Default interval half second (0.5s).

- `spark.readStream` (autoloader with continous trigger)
- Declarative pipeline trigger mode continous

Use cases: sensor data, logs, real-time analytics, sources: Kafka, event hubs, etc.

Streaming table: The easiest way is to keep adding new files to the storage. This table periodically updates itself automatically with new data. This table is useful for ever-growing data, like data from sensors. It is efficient because it just appends new parquet files containing new data.

```SQL
CREATE OR REFRESH STREAMING TABLE  my_table_name
SCHEDULE EVERY 1 WEEK             -- refresh/update period
AS
SELECT *
FROM STREAM read_files(
  '/volumes/project/my_streaming' -- path to your files
  format => 'CSV',
  sep => '|',
  header => true
);
```

```python
spark
.readStream.format("cloudFiles")
  .option("cloudFiles.format", "json")
  .option("cloudFiles.schemaLocation", "<checkpoint_path>")
  .load("/volumes/project/my_streaming")
.writeStream
  .option("checkpointLocation", "<checkpoint_path>")
  .trigger(processingTime="5 seconds")
  .toTable("my_table_name"
```

Force refresh: `REFRESH STREAMING TABLE my_table_name`

By default, if you don’t provide any trigger interval, the data will be processed every half second. This is equivalent to `trigger(processingTime=”500ms")`

### Metadata and schema mismatches

During ingestion you can apply various metadata:
`_metadata.file_modification_time`
`_metadata.file_name`

And handle schema mismatches:
`_rescued_data`


---

Ingestion into existing table: `MERGE INTO` (upsert), various strategy. God for slowly chaging dimensions (SCDs), incremental loads, and complex change data capture (CDC).


### Comparasion

![tables](./img/tables.png)

Managed tables: Databricks-managed storage.
External tables: data stored outside Databricks (e.g., cloud storage).



### Datatypes

**STREAMING TABLE:**

- support fot batch or streaming for incremental data processing.
- each time an streaming table is refreshed, it processes only new data since the last refresh.
- create SQL: `CREATE OR REFRESH STREAMING TABLE`
- In SQL you must to use `FROM STREAM read_files()` to invoke Auto Loader functionality for incremental streaming reads and checkpoints

**MATERIALIZED VIEW (MV):**

- Each time a materialized view is updated, query results are recalculated to reflect changes in upstream datasets
- Created and kept up-to-date by pipeline
- Use the `CREATE OR REFRESH MATERIALIZED VIEW` syntax
- Can be used anywhere in your pipeline, not just in the gold layer
- Where applicable, results are incrementally refreshed for materialized views, avoiding the need to completely rebuild the materialized view when new data arrives (Serverkess Only)
- Incremental refresh for materialized views is a cost-based optimizer, to power fast and efficient transformations for materialized views on Serveless compute

**VIEWS:**

TEMPORARY VIEW:

- temporary views are only persisted acriss the lifetime of the pipeline and private to the defining pipeline
- they are not registered as an object to Unity Catalog
- Use the `CREATE TEMPORARY VIEW` statement
- TM are useful as intermediate queries that are not exposed to end users

VIEW:

- no physial data
- they are registered as an object to Unity Catalog
- Use the `CREATE VIEW` statement

Limitations of views:

- the pipeline must be a Unity Catalog pipeline
- views cannot have streaming queries, or be used as a streaming source

CONSTRAINTS:

Levels:
WARN  default, row is written but a warning is logged
DROP  row is dropped
FAIL  entire transaction fails

```SQL
CREATE OR REFRESH STREAMING TABLE silver_db.orders_silver
(
CONSTRAINT valid_notify EXPECT (notifications IN ('Y', 'N')),
CONSTRAINT valid_id EXPECT (order_id IS NOT NULL) ON VIOLATION DROP ROW,
CONSTRAINT valid_date EXPECT (order_timestamp > "2021-01-01" ON VIOLATION FAIL UPDATE
)
AS SELECT ...
```

**JOINS**

streaming table -> joined streaming table <- static table
streaming table -> materialized view <- streaming table

**CDC**

Change data capture (CDC) is supported natively in Databricks Delta Lake. There are

TYPE 1: just overwrite old data with new data (no history)
TYPE 2: keep history by adding new records with timerange when they are valid

`APPLY CHANGES INTO`

```SQL
CREATE OR REFRESH STREAMING TABLE customers;

CREATE FLOW scd_type_1_flow AS
AUTO CDC INTO customers
    FROM STREAM updates
    KEYS (CustomerID)
    APPLY AS DELETE WHEN operation = "delete"
    SEQUENCE BY ProcessDate
    COLUMNS * EXCEPT (operation)
    STORED AS SCD TYPE 1;
```

There is also Change Data Feed (CDF) for tracking changes over time (notifiy).


### Miscellaneous

Delta Live Tables (DLT) has been recently renammed to Lakeflow Declarative Pipeline, 

## 4. Sources

### Jobs

Jobs are workflows, there are divided into **tasks**. Tasks are independent units of work that can be executed in parallel. Order is defined by Directed Acyclic Graph (DAG).

Jobs and tasks has input parameters, output parameters, and dependencies.

Jobs can be triggered manually or automatically based on schedules, or even triggered by events. For example triggered by update some tables. There are even condition triggers.


### Misc

Databricks Connect:" is a client library for the Databricks Runtime that allows you to connect popular IDEs (like PyCharm, VS Code, Jupyter Notebooks) to Databricks clusters. This enables you to write and test code locally while leveraging the power of Databricks for execution.
## 5. Sources

TODO:
  liquid clustering

https://airflow.apache.org/
https://www.dask.org/
