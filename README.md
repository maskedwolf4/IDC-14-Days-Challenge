# IDC-14-Days-Challenge
This Repo is for submission for IDC 21 Days Challenge Submission 

The challenge is organised by [DataBricks](https://docs.databricks.com/aws/en/introduction/) [CodeBasics](https://codebasics.io/) [IndianDataClub](https://www.indiandataclub.com/)

# Phase 1 - FOUNDATION

## Day 1 
### Learn:

- Why Databricks vs Pandas/Hadoop?
- Lakehouse architecture basics
- Databricks workspace structure
- Industry use cases (Netflix, Shell, Comcast)

### Tasks:

1. Create Databricks Community Edition account
2. Navigate Workspace, Compute, Data Explorer
3. Create first notebook
4. Run basic PySpark commands

![Home Page](assets/home.png)
**HomePage**

![WorkSpace](assets/workspace.png)
**WorkSpace**

![Compute](assets/compute.png)
**Compute**

![Notebook](assets/notebook.png)
**NoteBook**

## Day 2
### Explored about Apache Spark Fundamentals

### Learn:

- Spark architecture (driver, executors, DAG)
- DataFrames vs RDDs
- Lazy evaluation
- Notebook magic commands (`%sql`, `%python`, `%fs`)

### 🛠️ Tasks:

1. Upload sample e-commerce CSV
2. Read data into DataFrame
3. Perform basic operations: select, filter, groupBy, orderBy
4. Export results

![Loading Data](assets/loading.png)
**Loading Data**

![Operations](assets/operations.png)
**Basic Operations**

![Apache Spark Docs](assets/pysparkdocs.png)
**Apache Spark Docs**

# Day 3
### PySpark Transformations Deep Dive**

### Learn:

- PySpark vs Pandas comparison
- Joins (inner, left, right, outer)
- Window functions (running totals, rankings)
- User-Defined Functions (UDFs)

# Day 4
### Delta Lake Introduction
### 🛠️ Tasks:

1. Load full e-commerce dataset
2. Perform complex joins
3. Calculate running totals with window functions
4. Create derived features

![Revenue](assets/revenue.png)

![WindowFunction](assets/udf.png)

![UDF](assets/udf.png)

### Learn:

- What is Delta Lake?
- ACID transactions
- Schema enforcement
- Delta vs Parquet

### 🛠️ Tasks:

1. Convert CSV to Delta format
2. Create Delta tables (SQL and PySpark)
3. Test schema enforcement
4. Handle duplicate inserts


![Delta](assets/delta.png)

![Schema Enforcement](assets/schemaenforcement.png)

## End of Phase 1

# Phase 2 -  DATA ENGINEERING

# Day 5

## Learn:

- Time travel (version history)
- MERGE operations (upserts)
- OPTIMIZE & ZORDER
- VACUUM for cleanup

### 🛠️ Tasks:

1. Implement incremental MERGE
2. Query historical versions
3. Optimize tables
4. Clean old files

![Time Travel and Optimization](assets/TT&Opt.png)
**Time Travel and Optimization**

![Merge](assets/Merge.png)
**Merge**


# Day 6
## Medallion Architecture
### Learn:

- Bronze (raw) → Silver (cleaned) → Gold (aggregated)
- Best practices for each layer
- Incremental processing patterns

### 🛠️ Tasks:

1. Design 3-layer architecture
2. Build Bronze: raw ingestion
3. Build Silver: cleaning & validation
4. Build Gold: business aggregates


![Bronze and Silver](assets/B&S.png)
**Bronze and Silver**

![Gold](assets/Gold.png)
**Gold**