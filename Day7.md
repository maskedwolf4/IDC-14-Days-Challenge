# Workflows & Job Orchestration in Databricks

---

## 1. Databricks Jobs vs. Notebooks

It is crucial to distinguish between the **development** environment (Notebooks) and the **execution** framework (Jobs).

### The Notebook (Interactive Development)

* **Purpose:** Exploratory Data Analysis (EDA), testing code, and visualization.
* **State:** Stateful. Variables persist in memory until the cluster is detached.
* **Cluster:** Runs on an **All-Purpose (Interactive) Cluster**. These clusters are expensive because they are designed to stay "warm" and responsive for human interaction.

### The Job (Production Execution)

* **Purpose:** Automated, scheduled, and reliable execution of code.
* **State:** Stateless (mostly). Each run starts fresh.
* **Cluster:** Runs on a **Job Cluster**.
* **Cost Efficiency:** Job clusters are significantly cheaper (often ~50% less) than interactive clusters because they spin up for the specific job and terminate immediately after completion.
* **Isolation:** Each job gets its own isolated resources, preventing "noisy neighbor" issues where one user's heavy query slows down your production pipeline.



> **Anti-Pattern Warning:** Do not use `%run ./other_notebook` inside a notebook to orchestrate complex pipelines. This creates a "monolithic" failure point. If the 5th step fails, you have to re-run the whole chain. Use **Workflows** instead.

---

## 2. Multi-Task Workflows

Modern orchestration relies on **DAGs (Directed Acyclic Graphs)**. A Databricks Workflow allows you to chain multiple tasks together with dependencies.

### Why use Multi-Task Workflows?

1. **Modularity:** Instead of one giant notebook with 5000 lines of code, break it into 3 tasks: `Ingest`, `Clean`, `Aggregated`.
2. **Polyglot:** Task A can be a **Python** notebook, Task B can be a **SQL** query, and Task C can be a **dbt** command. They all run in the same workflow.
3. **Parallelism:** If Task B and Task C both depend on Task A, but not on each other, Databricks will run B and C **simultaneously**, reducing total runtime.

### Common Task Types

* **Notebook:** Run a specific notebook.
* **Python Script:** Run a `.py` file (better for software engineering standards).
* **JAR:** Run a compiled Java/Scala application.
* **Delta Live Tables (DLT):** Trigger a declarative pipeline.
* **SQL:** Execute a query or alert.

---

## 3. Parameters & Scheduling

Hard-coding values (like dates or file paths) is a bad practice. Workflows allow you to pass dynamic variables.

### Parameters

Parameters can be defined at the **Job Level** (global) or **Task Level** (local).

**1. Setting Parameters (In the Job UI):**
You define keys and values, such as `environment` = `prod` or `process_date` = `2023-10-25`.

**2. Retrieving Parameters (In the Code):**
Inside a Notebook, you use `dbutils.widgets` to grab these values.

```python
# 1. Initialize the widget (good for testing locally)
dbutils.widgets.text("process_date", "2023-01-01")
dbutils.widgets.dropdown("env", "dev", ["dev", "prod"])

# 2. Get the value passed by the Job
execution_date = dbutils.widgets.get("process_date")
current_env = dbutils.widgets.get("env")

print(f"Processing data for {execution_date} in {current_env}")

```

### Scheduling

* **Cron Schedules:** Standard Unix cron syntax (e.g., `0 0 8 * * ?` for "Every day at 8 AM").
* **Continuous:** The job restarts immediately after it finishes (useful for near-real-time micro-batches).
* **File Arrival Triggers:** The job starts automatically when a file lands in a specific S3/ADLS location.

---

## 4. Error Handling & Reliability

Production jobs *will* fail. The goal is to handle failures gracefully without waking up the on-call engineer at 3 AM.

### A. Retries

Transient errors happen (e.g., a momentary network blip with S3).

* **Configuration:** You can configure a task to "Retry 3 times" with an "Interval of 5 minutes".
* **Result:** The system creates a buffer. If the network blip resolves in 2 minutes, the job succeeds on the second try without marking the workflow as "Failed".

### B. Timeouts

Sometimes a job doesn't fail, it "zombies" (hangs indefinitely).

* **Configuration:** Set a **Timeout** (e.g., 2 hours).
* **Result:** If the job runs longer than 2 hours, Databricks forcibly kills it and marks it as failed. This prevents a "zombie" cluster from racking up costs for 48 hours over the weekend.

### C. Repair and Rerun

If a workflow has 10 tasks and Task #9 fails:

* **Old Way:** Restart the whole job from Task #1. (Wastes time and money).
* **Workflows Way:** Click **"Repair and Rerun"**. The system keeps the successful state of Tasks 1-8 and only re-runs Task #9 and its dependents.

### D. Notifications

You can configure alerts for specific job events:

* **On Start:** (Rarely used, too noisy).
* **On Success:** Good for downstream dependencies.
* **On Failure:** Critical. Send an email or a Slack notification (via Webhook) to the engineering team.

### Code-Level Error Handling (Try/Except)

While Job settings handle *infrastructure* errors, your code should handle *logic* errors.

```python
try:
    df = spark.read.load(path)
    # ... logic ...
except Exception as e:
    # Log the specific error
    print(f"CRITICAL ERROR: Data missing at {path}. Exception: {e}")
    # Explicitly fail the notebook so the Job knows it failed
    dbutils.notebook.exit("FAILURE") 

```
