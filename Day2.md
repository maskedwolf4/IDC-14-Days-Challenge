# Spark Concepts Reference

Apache Spark is a distributed data processing engine designed for large-scale data analytics. This document covers core architecture components, DataFrames vs RDDs comparison, lazy evaluation mechanism, and Databricks notebook magic commands.

## Spark Architecture

Spark follows a master-worker architecture with a single driver coordinating multiple executors across a cluster.

- **Driver**: The driver runs the main() function of a Spark application and creates the SparkContext (or SparkSession in newer versions). It converts user code into a logical and physical execution plan, schedules tasks, and monitors job progress. The driver communicates with the cluster manager to request resources and manages the Directed Acyclic Graph (DAG) of operations.
- **Executors**: Worker processes that run tasks assigned by the driver. Each executor handles one or more tasks concurrently, stores data in memory or disk, and reports status back to the driver. Executors are dynamically allocated and can cache intermediate results for iterative computations.
- **DAG (Directed Acyclic Graph)**: A dataflow graph representing the sequence of transformations on RDDs or DataFrames. Transformations build the DAG lazily, and actions trigger its execution by the DAG Scheduler (in the driver), which breaks it into stages and tasks for distribution to executors.

The cluster manager (YARN, Kubernetes, or standalone) allocates resources, while the driver oversees fault tolerance through lineage tracking in the DAG.

## DataFrames vs RDDs

DataFrames provide structured data handling with optimizations, while RDDs offer low-level, flexible APIs.

| Aspect              | RDDs (Resilient Distributed Datasets) | DataFrames                          |
|---------------------|---------------------------------------|-------------------------------------|
| **Structure**      | Unstructured collection of objects; no schema enforcement . | Structured with named columns and schema; like relational tables. |
| **Optimization**   | No automatic optimization; executes as written. | Catalyst optimizer rewrites queries for efficiency; Tungsten engine for memory management. |
| **Performance**    | Good for unstructured data or custom logic. | Faster for SQL queries and structured data due to optimizations (up to 10x in some cases). |
| **API Level**      | Low-level; functional transformations (map, filter). | High-level; SQL-like operations (select, groupBy). |
| **Type Safety**    | None (Scala/Java); dynamic in Python. | None in Python/Scala DataFrames; Datasets add type safety in Scala/Java. |
| **Use Case**       | Graph processing, MLlib custom algos. | ETL, analytics, Spark SQL . |

Both support lazy evaluation, but DataFrames benefit from whole-stage codegen and predicate pushdown.

## Lazy Evaluation

Spark employs lazy evaluation to optimize execution by delaying computation until necessary.

Transformations (map, filter, join) build a DAG but do not execute immediately—they record operations in the lineage. Actions (collect, count, save) trigger computation, allowing Spark to optimize the entire pipeline: reorder operations, eliminate redundancies, and pipeline stages.

Benefits include:
- **Optimization**: Catalyst optimizer analyzes the DAG for efficient plans.
- **Fault Tolerance**: Lineage allows recomputation of lost partitions without full restarts.
- **Efficiency**: Avoids unnecessary intermediate results.

Example: `df.filter(...).groupBy(...).count()` builds DAG on filter+groupBy; `count()` executes optimized plan.

## Notebook Magic Commands

Databricks notebooks support magic commands for seamless language switching and file operations (%fs is Databricks-specific).

- **`%sql`**: Runs SQL queries on Spark data. Example: `%sql SELECT * FROM table LIMIT 10`. Integrates with Delta tables and visualizations.
- **`%python`**: Executes Python code in a cell (default). Useful for PySpark DataFrames/RDDs.
- **`%fs`**: File system commands for DBFS/ADLS/S3. Examples:
  - `%fs ls dbfs:/mnt/data` lists files.
  - `%fs head dbfs:/path/file.csv` previews content.
  - `%fs mkdirs dbfs:/output` creates directories.

Other magics: `%md` for Markdown, `%sh` for shell, `%fs put`/`get` for uploads/downloads. Use `%%` for multi-line blocks.

