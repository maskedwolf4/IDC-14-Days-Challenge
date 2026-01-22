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
---

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

# Day 7
## Workflows & Job Orchestration

### Learn:

- Databricks Jobs vs notebooks
- Multi-task workflows
- Parameters & scheduling
- Error handling

### 🛠️ Tasks:

1. Add parameter widgets to notebooks
2. Create multi-task job (Bronze→Silver→Gold)
3. Set up dependencies
4. Schedule execution

![Notebook](assets/NBJob.png)
**NoteBook**

![Jobs](assets/Job.png)
**Job and Tasks**

![Job Running](assets/Jobruning.png)
**Job Running**

![Job Success](assets/JobSuccess.png)
**Job Success**

![Schedule](assets/Schedule.png)
**Scheduling Job**


# DAY 8
## Unity Catalog Governance**


### Learn:

- Catalog → Schema → Table hierarchy
- Access control (GRANT/REVOKE)
- Data lineage
- Managed vs external tables

### 🛠️ Tasks:

1. Create catalog & schemas
2. Register Delta tables
3. Set up permissions
4. Create views for controlled access

![UnityCatalog](assets/unitycatalog.png)

## End of Phase 3
---

## PHASE 3: ADVANCED ANALYTICS (Days 9-11)

# DAY 9 (17/01/26) 

## SQL Analytics & Dashboards

### Learn:

- SQL warehouses
- Complex analytical queries
- Dashboard creation
- Visualizations & filters

### 🛠️ Tasks:

1. Create SQL warehouse
2. Write analytical queries
3. Build dashboard: revenue trends, funnels, top products
4. Add filters & schedule refresh

![SQL](assets/SQL.png)


### DAY 10 (18/01/26) 

## Performance Optimization

### Learn:

- Query execution plans
- Partitioning strategies
- OPTIMIZE & ZORDER
- Caching techniques

### 🛠️ Tasks:

1. Analyze query plans
2. Partition large tables
3. Apply ZORDER
4. Benchmark improvements

![Optimize](assets/optimize.png)

# DAY 11 (19/01/26) 

## Statistical Analysis & ML Prep

### Learn:

- Descriptive statistics
- Hypothesis testing
- A/B test design
- Feature engineering

### 🛠️ Tasks:

1. Calculate statistical summaries
2. Test hypotheses (weekday vs weekend)
3. Identify correlations
4. Engineer features for ML

![Feature Engineering](assets/FeatureEng.png)
**Feature Engineering**


![Descriptive Stats](assets/DesStats.png)
**Descriptive Stats**

## PHASE 4: AI & ML (Days 12-14)

# DAY 12 (20/01/26) 
## MLflow Basics

### Learn:

- MLflow components (tracking, registry, models)
- Experiment tracking
- Model logging
- MLflow UI

### 🛠️ Tasks:

1. Train simple regression model
2. Log parameters, metrics, model
3. View in MLflow UI
4. Compare runs

![Mlflow](assets/mlflow.png)

# DAY 13 (21/01/26) 
## Model Comparison & Feature Engineering

### Learn:

- Training multiple models
- Hyperparameter tuning
- Feature importance
- Spark ML Pipelines

### 🛠️ Tasks:

1. Train 3 different models
2. Compare metrics in MLflow
3. Build Spark ML pipeline
4. Select best model

![MLFlow](assets/mlflow2.png)


# DAY 14 (22/01/26)
## AI-Powered Analytics: Genie & Mosaic AI

### Learn:

- Databricks Genie (natural language → SQL)
- Mosaic AI capabilities
- Generative AI integration
- AI-assisted analysis

### 🛠️ Tasks:

1. Use Genie to query data with natural language
2. Explore Mosaic AI features
3. Build simple NLP task
4. Create AI-powered insights

![GenieAI](assets/genieai.png)

![Trasnformer](assets/transformer.png)

# End of Learning Phase