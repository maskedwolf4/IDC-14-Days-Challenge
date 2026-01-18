# Spark & Databricks Performance Optimization Guide

---

## 1. Query Execution Plans

Before optimizing, you must diagnose. The **Execution Plan** shows you exactly how Spark intends to run your query.

### The Catalyst Optimizer

Spark uses the Catalyst Optimizer to turn your code into an efficient physical plan.

1. **Unresolved Logical Plan:** Checks if column names exist.
2. **Logical Plan:** Optimizes logical steps (e.g., "Filter before Join").
3. **Physical Plan:** Determines how to actually execute it on the cluster (e.g., "Use HashAggregate", "Use SortMergeJoin").

### How to Read a Plan

You can view the plan using `df.explain()` or the **Spark UI (SQL Tab)**.

**Key Red Flags to Look For:**

* **Full Table Scans:** Are you reading 1TB of data to find 5 rows? (Solution: Partitioning/ZORDER).
* **Exchange (Shuffle):** Data moving between nodes. This is the most expensive operation in Spark.
* **Spill to Disk:** If a single task runs out of RAM, it writes temporary data to the hard drive, slowing down processing by 100x.
* **Skew:** One task takes 1 hour while 99 others take 1 minute.

```python
# Extended true gives you Parsed, Analyzed, Optimized, and Physical plans
df.explain(extended=True)

```

---

## 2. Partitioning Strategies

Partitioning controls how data is split across the cluster. There are two types: **Storage Partitioning** (Folder structure) and **Memory Partitioning** (Spark Tasks).

### A. Storage Partitioning (Partitioning by Column)

This splits data into directories based on a column value (e.g., `year=2024/month=01`).

* **Benefit (Partition Pruning):** If you filter `WHERE year = 2024`, Spark completely ignores all other folders.
* **The Trap (Over-partitioning):** Do **not** partition by high-cardinality columns (like `user_id`).
* *Bad:* Creating 1 million tiny directories. This causes file listing slowness (S3/ADLS latency).
* *Rule of Thumb:* Partitions should contain at least 1GB of data. If you have less than 1TB of data, you probably don't need folder partitions; use ZORDER instead.



### B. Memory Partitioning (`repartition` vs `coalesce`)

This controls the parallelism of your job.

* **`repartition(n)`:** Performs a **Full Shuffle**. It distributes data evenly across `n` partitions. Use this to fix Data Skew or increase parallelism.
* **`coalesce(n)`:** Decreases the number of partitions **without a full shuffle**. It merges local partitions. Use this before writing to disk to avoid creating tiny files.

```python
# Use repartition to increase parallelism (incurs shuffle)
df = df.repartition(100) 

# Use coalesce to reduce file count (efficient, no full shuffle)
df.write.parquet("...") # writes 100 files
df.coalesce(1).write.parquet("...") # writes 1 file

```

---

## 3. OPTIMIZE & ZORDER (Data Skipping)

While partitioning relies on directory structures, **Z-Ordering** relies on file statistics (Min/Max values) to skip data.

### How it works

Standard Parquet/Delta files store min/max stats for columns. If you sort your data by `id` before writing, the files look like this:

* File A: IDs 1-100
* File B: IDs 101-200

If you query `WHERE id = 50`, Spark reads File A and **skips** File B.

**Z-Ordering** extends this concept to multiple columns. It uses a space-filling curve algorithm to physically co-locate related data points.

### When to use

* **High Cardinality:** Use on columns like `timestamp`, `product_id`, or `user_id` where folder partitioning is too granular.
* **Frequent Filters:** Use on columns most commonly found in your `WHERE` clauses.

```sql
-- Compacts small files AND co-locates data by region and date
OPTIMIZE sales_data 
ZORDER BY (region_id, transaction_date);

```

---

## 4. Caching Techniques

Caching saves intermediate results so Spark doesn't have to re-compute them.

### A. Spark Cache (`.cache()` / `.persist()`)

Stores data in the **Executor's RAM** (JVM Heap).

* **Lazy:** The cache is not built until the first Action (e.g., `count()`) is called.
* **Use Case:** When you reuse the same DataFrame multiple times in a script (e.g., training an ML model iteratively).
* **Risk:** If the dataset is too big for RAM, it may spill to disk or cause OOM (Out Of Memory) errors, actually slowing you down.

```python
# Standard Cache (Memory and Disk)
df.cache()
df.count() # Triggers the cache build

# Custom Persistence (e.g., Memory Only)
from pyspark import StorageLevel
df.persist(StorageLevel.MEMORY_ONLY)

# Always unpersist when done to free up RAM!
df.unpersist()

```

### B. Delta Cache (Disk Cache)

*Note: Often confused with Spark Cache, but distinct.*
This stores copies of remote files (S3/ADLS) on the **local SSDs** of the worker nodes.

* **Mechanism:** When you read a file from S3, Databricks automatically keeps a copy on the worker's SSD. The second time you read it, it's instant.
* **Benefit:** Accelerated read speeds without using up your RAM (JVM Heap).
* **Configuration:** Enabled by default on most Databricks instance types (search for "Fleet" or "Cache Accelerated" nodes).

### Summary: Which Caching to use?

| Scenario | Strategy |
| --- | --- |
| **Iterative ML Algorithms** | Use `df.cache()` (RAM is fastest). |
| **ETL with multiple branches** | Use `df.cache()` on the common upstream DataFrame. |
| **Simple one-pass ETL** | **Do NOT cache.** It adds overhead to serialize/deserialize data. |
| **Interactive Dashboarding** | Rely on **Delta Cache** (Disk Cache) automatically. |