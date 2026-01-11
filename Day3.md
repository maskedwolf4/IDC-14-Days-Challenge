# PySpark and Data Engineering Concepts Guide

## 1. PySpark vs. Pandas: A Detailed Comparison

While both libraries are essential tools for data manipulation in Python, they serve different purposes based on the scale of data and the architectural requirements.

### Architecture & Execution Model

* **Pandas (Single Node / In-Memory):**
* **Architecture:** Pandas runs on a single machine. It loads the entire dataset into the RAM (Random Access Memory) of that one computer.
* **Execution:** It uses **Eager Execution**. When you type a command (e.g., `df.head()`), it runs immediately.
* **Limitation:** If your dataset is larger than your machine's RAM, Pandas will fail (MemoryError).


* **PySpark (Distributed / Cluster):**
* **Architecture:** PySpark runs on a cluster of computers (nodes). It uses a Driver (master) to coordinate tasks across Executors (workers).
* **Execution:** It uses **Lazy Execution**. Transformations (like filtering or mapping) are not executed immediately. Instead, Spark builds a DAG (Directed Acyclic Graph) of instructions. Processing only happens when an *Action* (like `show()`, `count()`, or `write`) is called.
* **Advantage:** It can process terabytes or petabytes of data by splitting files into partitions across many machines.



### Comparison Table

| Feature | Pandas | PySpark |
| --- | --- | --- |
| **Data Size** | Small to Medium (< 10GB usually). | Massive (Terabytes/Petabytes). |
| **Processing** | Single-threaded (mostly). | Distributed Multi-threaded. |
| **Speed** | Faster for small data (less overhead). | Faster for large data (parallelism). |
| **Syntax** | Index-based; very flexible API. | SQL-like; functional programming style. |
| **Missing Values** | Handles `NaN` (Not a Number). | Handles `null` (distinct from NaN). |

### When to use which?

> **Rule of Thumb:** If your data fits easily into your laptop's RAM, stick with **Pandas** for simplicity and speed. If you are dealing with Big Data or streaming data that exceeds single-machine capacity, use **PySpark**.

---

## 2. Joins (Inner, Left, Right, Outer)

Joins are used to combine rows from two or more dataframes based on a related column between them.

### The Setup

Imagine two DataFrames:

* **Employees:** `id`, `name`, `dept_id`
* **Departments:** `dept_id`, `dept_name`

### A. Inner Join

Returns only the rows where there is a match in **BOTH** DataFrames.

* *Result:* If an employee has a `dept_id` that doesn't exist in the Departments table, they are excluded.

```python
# PySpark
df_inner = employees.join(departments, on="dept_id", how="inner")

# Pandas
df_inner = pd.merge(employees, departments, on="dept_id", how="inner")

```

### B. Left Join (Left Outer)

Returns **ALL** rows from the Left DataFrame and the matched rows from the Right DataFrame.

* *Result:* You get all employees. If an employee has no matching department, the department columns will be `null` (or `NaN`).

```python
# PySpark
df_left = employees.join(departments, on="dept_id", how="left")

# Pandas
df_left = pd.merge(employees, departments, on="dept_id", how="left")

```

### C. Right Join (Right Outer)

Returns **ALL** rows from the Right DataFrame and the matched rows from the Left DataFrame.

* *Result:* You get all departments. If a department has no employees, the employee columns will be `null`.

```python
# PySpark
df_right = employees.join(departments, on="dept_id", how="right")

# Pandas
df_right = pd.merge(employees, departments, on="dept_id", how="right")

```

### D. Full Outer Join

Returns **ALL** rows when there is a match in **EITHER** the Left or Right DataFrame.

* *Result:* You get all employees and all departments. Unmatched sides will be filled with `null`.

```python
# PySpark
df_outer = employees.join(departments, on="dept_id", how="outer")

# Pandas
df_outer = pd.merge(employees, departments, on="dept_id", how="outer")

```

---

## 3. Window Functions

Window functions perform calculations across a set of table rows that are somehow related to the current row. Unlike `GroupBy`, which collapses rows, Window functions keep the original rows and add a result column.

**Key Components:**

1. **Partitioning:** Dividing data into groups (like `GroupBy`).
2. **Ordering:** Sorting data within those groups.
3. **Frame:** (Optional) Defining exactly which rows to look at (e.g., "3 rows prior to current").

### A. Running Totals (Cumulative Sum)

Calculates the sum of a column row-by-row within a partition.

*Scenario:* Calculate the cumulative salary spend for each department.

```python
from pyspark.sql.window import Window
from pyspark.sql.functions import sum, col

# Define Window: Partition by Dept, Order by Salary (or time)
windowSpec = Window.partitionBy("dept_id").orderBy("salary")

# Apply Window Function
df.withColumn("running_total", sum(col("salary")).over(windowSpec)).show()

```

### B. Rankings

There are three main ways to rank data. Assume we are ranking employees by salary.

| Salary | Row Number | Rank | Dense Rank |
| --- | --- | --- | --- |
| 1000 | 1 | 1 | 1 |
| 2000 | 2 | 2 | 2 |
| 2000 | 3 | 2 | **2** |
| 3000 | 4 | **4** | **3** |

1. **`row_number()`:** Assigns a unique sequential number to each row (1, 2, 3, 4). Ties are broken arbitrarily.
2. **`rank()`:** Assigns the same rank for ties, but **skips** the next number (1, 2, 2, 4).
3. **`dense_rank()`:** Assigns the same rank for ties, but does **NOT skip** the next number (1, 2, 2, 3).

```python
from pyspark.sql.functions import row_number, rank, dense_rank

windowSpec = Window.partitionBy("dept_id").orderBy(col("salary").desc())

df.withColumn("rank", rank().over(windowSpec)) \
  .withColumn("dense_rank", dense_rank().over(windowSpec)) \
  .withColumn("row_num", row_number().over(windowSpec)) \
  .show()

```

---

## 4. User-Defined Functions (UDFs)

UDFs allow you to write custom Python functions and apply them to PySpark DataFrames. However, they come with performance costs.

### Standard Python UDFs

These are the most basic UDFs.

* **How it works:** PySpark serializes the data row-by-row, sends it to a Python process, runs the function, and sends the result back to the JVM.
* **Performance:** **Slow**. It breaks the catalyst optimizer and incurs high serialization overhead.

```python
from pyspark.sql.functions import udf
from pyspark.sql.types import StringType

# 1. Define Python function
def categorize_salary(salary):
    if salary > 5000: return "High"
    return "Low"

# 2. Register UDF
salary_udf = udf(categorize_salary, StringType())

# 3. Use in DataFrame
df.withColumn("category", salary_udf(col("salary")))

```

### Pandas UDFs (Vectorized UDFs)

Introduced in Spark 2.3, built on top of Apache Arrow.

* **How it works:** Instead of row-by-row, it sends data in **batches** (vectors) to Pandas.
* **Performance:** Much faster than standard UDFs (often 10x-100x speedup) because it minimizes serialization overhead.

```python
from pyspark.sql.functions import pandas_udf
import pandas as pd

# 1. Define Pandas UDF
@pandas_udf("string")
def categorize_salary_vectorized(salary_series: pd.Series) -> pd.Series:
    return salary_series.apply(lambda x: "High" if x > 5000 else "Low")

# 2. Use in DataFrame
df.withColumn("category", categorize_salary_vectorized(col("salary")))

```

> **Best Practice:** Always try to use native PySpark functions (`pyspark.sql.functions`) first. If you must write custom logic, use **Pandas UDFs**. Use standard Python UDFs only as a last resort.

---
