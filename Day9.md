# Databricks SQL: Analytics & Dashboards

---

## 1. SQL Warehouses

A **SQL Warehouse** is the compute engine that executes your SQL queries. Unlike general-purpose clusters (used for Python/Scala engineering), SQL Warehouses are optimized specifically for SQL workloads and BI tools (Tableau, PowerBI).

### Types of SQL Warehouses

1. **Serverless:**
* **Best For:** Ad-hoc analysis, dashboards, and high concurrency.
* **Features:** Instant startup (seconds), automatic patching, and managed storage. The compute infrastructure is managed entirely by Databricks.

2. **Pro:**
* **Best For:** Teams that need specific networking customizations or advanced integration features but want performance comparable to Serverless.

3. **Classic:**
* **Best For:** Legacy workloads. Slower startup times (minutes) because it provisions VMs in your cloud account.


### Sizing and Scaling

Warehouses use "T-Shirt Sizing" (2X-Small to 4X-Large) to determine power.

* **Vertical Scaling (Size):** A larger size (e.g., Large vs. Small) processes a *single* complex query faster.
* **Horizontal Scaling (Clusters):** If 100 users query the dashboard at once, the warehouse automatically spins up additional clusters (Min/Max clusters) to handle the concurrency (Load Balancing).

---

## 2. Complex Analytical Queries

Databricks SQL uses **ANSI SQL** compatible syntax (Delta SQL). It includes powerful extensions for modern data analysis.

### Common Table Expressions (CTEs)

CTEs improve readability by breaking complex logic into modular steps.

```sql
WITH regional_sales AS (
    SELECT region, SUM(amount) as total_sales
    FROM sales
    WHERE order_date > '2023-01-01'
    GROUP BY region
),
top_regions AS (
    SELECT region
    FROM regional_sales
    ORDER BY total_sales DESC
    LIMIT 3
)
SELECT * FROM sales 
WHERE region IN (SELECT region FROM top_regions);

```

### The `QUALIFY` Clause

A powerful Spark SQL feature that filters window functions *without* needing a subquery.

* *Scenario:* Get the latest status update for every order.

```sql
SELECT 
    order_id, 
    status, 
    updated_at
FROM order_history
-- Partition by Order, Order by Time Descending, keep Rank 1
QUALIFY ROW_NUMBER() OVER (PARTITION BY order_id ORDER BY updated_at DESC) = 1;

```

### JSON & Higher-Order Functions

You can query semi-structured data without flattening it.

```sql
-- Extracting data from a JSON string column
SELECT 
    id, 
    details:address.city as city, -- Colon syntax for JSON navigation
    details:tags[0] as primary_tag
FROM raw_events;

```

---

## 3. Visualizations & Filters

Once a query runs, you can immediately visualize the results in the SQL Editor.

### Visualization Types

* **Charts:** Bar, Line, Area, Pie, Scatter, Bubble.
* **Data:** Pivot Tables, Counter (Big Number), Funnel.
* **Geospatial:** Choropleth maps (requires ISO country codes or state names) and Marker maps (requires Lat/Long).

### Query Parameters (Filters)

You can make queries dynamic using **Parameters**. These become filter widgets in the UI.

* **Syntax:** Double curly braces `{{ parameter_name }}`.

```sql
SELECT * FROM sales 
WHERE country = '{{ Country_Name }}' 
AND order_date >= '{{ Start_Date }}';

```

When you run this, Databricks creates a text box or dropdown menu for "Country_Name" and a date picker for "Start_Date".

---

## 4. Dashboard Creation

Dashboards aggregate multiple visualizations into a single presentation layer.

### Lakeview (AI/BI Dashboards)

The modern dashboarding experience in Databricks.

* **Canvas-Based:** Drag and drop visualizations onto a grid.
* **Dataset Centric:** You define a dataset *once*, and multiple charts can use it without writing new SQL for every single chart.
* **Cross-Filtering:** Clicking a bar in "Chart A" automatically filters "Chart B" to show data for that category.

### Features

1. **Draft vs. Published:** You can edit a dashboard in "Draft" mode without affecting the viewers. Once ready, you "Publish" the version.
2. **Scheduling:** You can schedule a dashboard to refresh every morning and email a PDF snapshot to stakeholders.
3. **Sharing:**
* **Permissions:** "Can View" vs "Can Edit".
* **Embedding:** Dashboards can be embedded into internal web portals (iframe).


### Best Practices

* **Cache:** Dashboards use the SQL Warehouse cache. If User A loads the dashboard, the results are cached. If User B loads it 1 minute later, it loads instantly without re-computing (saving money).
* **Limit Rows:** For visual performance, aggregate data in SQL (e.g., `GROUP BY`) rather than sending 1 million raw rows to the browser to render.