---
title: Databricks
icon: simple/databricks
full_width: true
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


Here is the translation of the technical summary into English, including short code snippets to illustrate the differences.


## 1. Work Distribution (Orchestration)

**Job (Workflow):** The unit of orchestration. It is a scheduled workload that defines **WHO** (cluster), **WHEN** (schedule/trigger), and **WHAT** (list of tasks).

* **Usage:** typically used to run production ETL pipelines (e.g., nightly batch).

**Task:** A specific step within a Job.

* Tasks form a **DAG** (Directed Acyclic Graph) to define dependencies (e.g., Task B runs only after Task A succeeds).
* **Task Types:**
* **Notebook Task:** Executes a standard notebook (Imperative).
* **DLT Pipeline Task:** Triggers a DLT pipeline update (Declarative).

**liquid clustering**: Predictive optimization that automatically optimizes data layout based on query patterns. So your data is organized in a way that speeds up your queries.


## 2. Development Methodology (Notebook vs. DLT)

### Notebook (Imperative)

You must define the **process**. You are responsible for managing state, checkpoints, and schema enforcement manually.

```python
# Notebook: You manage the "How"
# 1. Define Checkpoint (Critical manual step)
checkpoint_path = "/mnt/checkpoints/users_bronze"

# 2. Read Stream
df = spark.readStream.load("/mnt/source/users")

# 3. Write Stream (Explicit write command)
(df.writeStream
    .format("delta")
    .option("checkpointLocation", checkpoint_path) # Manual state management
    .toTable("bronze_users")
)
```

### Delta Live Tables - DLT (Declarative)

You define the **target state**. The framework automatically handles infrastructure, checkpoints, lineage, and data quality.

```python
# DLT: You define the "What"
import dlt

# 1. Define the target table
@dlt.table(
    name="bronze_users",
    comment="Raw user data"
)
def ingest_users():
    # 2. Return the transformation (No write command, no manual checkpoint)
    return spark.readStream.load("/mnt/source/users")

```

Delta Live Tables (DLT) has been recently renammed to Lakeflow Declarative Pipeline, 

### noteebook vs Delta Live Table

DLT are declarative and mask complexity about autoloader.

<div class="grid" markdown>

```python title="notebook"
source_path = "/Volumes/my_catalog/my_schema/inbox/users.csv"

df = spark.read.format("csv") \
    .option("header", "true") \
    .load(source_path)

df.write.format("delta") \
    .mode("append") \
    .saveAsTable("my_catalog.my_schema.bronze_users")
```
 
```python title="DLT"
import dlt

@dlt.table(
     name="bronze_users",
     comment="Raw data from CSV loaded incrementally"
)
def ingest_users():
    """
    Read all what will be in the folder
    Use Auto Loader
    """
    return (
        spark.readStream.format("cloudFiles")
        .option("cloudFiles.format", "csv")
        .load("/Volumes/my_catalog/my_schema/inbox/")
    )
```

</div>

## 3. Structured Streaming (Processing Engine)

The core engine for processing data. It processes data in **Micro-batches**. It is used inside both Notebooks and DLT.

### Trigger Modes (Execution Frequency)

**A) Triggered (Batch Mode)**

* **Config:** `.trigger(availableNow=True)`
* **Behavior:** Processes all available data since the last run, then **stops** the cluster.
* **Use Case:** Cost-effective periodic jobs (e.g., Daily ETL).

```python
# Process everything available, then shut down
df.writeStream \
  .trigger(availableNow=True) \
  .toTable("target_table")

```

**B) Continuous (Micro-batch Loop)**

* **Config:** `.trigger(processingTime="10 seconds")`
* **Behavior:** Runs in a loop. Checks for data every 10 seconds. If the previous batch finished early, it waits. If it took longer, it starts immediately. The cluster runs **24/7**.
* **Use Case:** Low-latency requirements (near real-time).

```python
# Check for data every 10 seconds indefinitely
df.writeStream \
  .trigger(processingTime="10 seconds") \
  .toTable("target_table")

```


## 4. Auto Loader (Ingestion Tool)

A specific **source** for Structured Streaming optimized for file ingestion from cloud storage (S3/ADLS).

* **Identifier:** `format("cloudFiles")`
* **Function:** Efficiently detects new files without listing the entire directory.

```python
# Usage within a Stream (works in both Notebooks and DLT)
df = (spark.readStream
      .format("cloudFiles")           # Activates Auto Loader
      .option("cloudFiles.format", "csv")
      .load("/mnt/source/incoming_files/")
)
```

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




## Metadata and schema mismatches

During ingestion you can apply various metadata:
`_metadata.file_modification_time`
`_metadata.file_name`

And handle schema mismatches:
`_rescued_data`


---

Ingestion into existing table: `MERGE INTO` (upsert), various strategy. God for slowly chaging dimensions (SCDs), incremental loads, and complex change data capture (CDC).


## Comparasion

![tables](./img/tables.png)

Managed tables: Databricks-managed storage.
External tables: data stored outside Databricks (e.g., cloud storage).


## Datatypes

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

## CONSTRAINTS

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

```
streaming table -> joined streaming table <- static table
streaming table -> materialized view <- streaming table
```

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

## 4. Sources

https://airflow.apache.org/
https://www.dask.org/
