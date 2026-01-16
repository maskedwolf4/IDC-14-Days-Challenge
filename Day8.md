# Unity Catalog: Governance and Structure

---

## 1. The Three-Level Namespace (Hierarchy)

In the legacy Hive metastore, data was often organized simply as `database.table`. Unity Catalog introduces a **Three-Level Namespace** to better organize data across teams, environments, and business units.

The hierarchy is: **`catalog`.`schema`.`table**`

### A. Metastore (The Container)

The top-level container. Typically, you have **one Metastore per Region** (e.g., `us-east-1`). It maps to a storage account (like an S3 bucket or ADLS container) where the actual data for managed tables lives.

### B. Catalog (Level 1)

The first level of logical isolation. Catalogs are often used to separate:

* **Environments:** `prod_catalog`, `dev_catalog`, `staging_catalog`.
* **Business Units:** `finance_catalog`, `marketing_catalog`.
* **Governance Zones:** `bronze_catalog`, `silver_catalog`, `gold_catalog`.

### C. Schema (Level 2)

Formerly known as a "Database." Schemas organize tables and views into logical groups within a catalog.

* *Example:* Inside `finance_catalog`, you might have schemas for `payroll`, `revenue`, and `forecasting`.

### D. Table / View / Volume (Level 3)

The actual assets.

* **Tables:** Structured data.
* **Views:** Virtual tables (saved queries).
* **Volumes:** Unstructured data (PDFs, images, CSVs) governed by Unity Catalog.

**SQL Access Example:**

```sql
SELECT * FROM prod_catalog.finance_schema.revenue_report;

```

---

## 2. Access Control (GRANT / REVOKE)

Unity Catalog moves access control away from filesystem ACLs (access control lists) and into standard SQL syntax. This means permissions are applied to the logical objects (Tables/Views), not the physical files (Parquet/JSON).

### The Inheritance Model

Permissions are inherited.

* If you `GRANT SELECT` on a **Catalog**, the user automatically gets `SELECT` on **ALL** Schemas and Tables inside that Catalog.
* This simplifies governance: you don't need to grant access to 500 individual tables.

### Common Privileges

* **`USE CATALOG`**: Allows the user to traverse the catalog.
* **`USE SCHEMA`**: Allows the user to traverse the schema.
* **`SELECT`**: Allows reading data.
* **`MODIFY`**: Allows `INSERT`, `UPDATE`, `DELETE`.
* **`CREATE TABLE`**: Allows creating new assets.

### Syntax Examples

**Granting Access:**

```sql
-- 1. Allow user to enter the catalog
GRANT USE CATALOG ON CATALOG prod_catalog TO `data_science_team`;

-- 2. Allow user to enter the schema
GRANT USE SCHEMA ON SCHEMA prod_catalog.finance TO `data_science_team`;

-- 3. Allow user to read a specific table
GRANT SELECT ON TABLE prod_catalog.finance.revenue TO `data_science_team`;

```

**Revoking Access:**

```sql
-- Revoke write access but keep read access
REVOKE MODIFY ON TABLE prod_catalog.finance.revenue FROM `junior_analyst`;

```

---

## 3. Data Lineage

Data Lineage answers the question: *"Where did this data come from, and who is using it?"*

### Automated Runtime Lineage

Unity Catalog automatically captures lineage in real-time as queries run. You do not need to manually document sources.

* **Table-Level Lineage:** Shows that Table A was created by joining Table B and Table C.
* **Column-Level Lineage:** Shows that the column `total_revenue` in the Gold table was calculated using `unit_price` from Silver and `quantity` from Bronze.

### Benefits

1. **Impact Analysis:** Before you change a column name in a Bronze table, you can see every downstream Dashboard and Gold table that will break.
2. **Debugging:** If a report shows wrong numbers, you can trace the data upstream to find exactly which ingestion job introduced the error.
3. **Governance:** You can prove to auditors that sensitive PII (Personally Identifiable Information) in the source did *not* leak into the public reporting table.

---

## 4. Managed vs. External Tables

When you create a table in Unity Catalog, you must choose how the physical data files are managed.

### Managed Tables (The Default)

* **What is it?** Unity Catalog manages both the **Metadata** and the **Physical Data**.
* **Location:** Data is stored in the "root storage" of the Metastore (or a specific Managed Location defined at the Catalog/Schema level).
* **Behavior:** If you `DROP TABLE`, Unity Catalog deletes the metadata **AND** physically deletes the files from the cloud storage.
* **Best For:** Most standard tables where you want Databricks to handle storage lifecycle and optimization.

```sql
-- No location specified = Managed Table
CREATE TABLE prod_catalog.finance.employees (
  id INT,
  name STRING
);

```

### External Tables

* **What is it?** Unity Catalog manages the **Metadata**, but YOU manage the **Physical Data**.
* **Location:** You explicitly point the table to an S3 bucket or ADLS container that you control.
* **Behavior:** If you `DROP TABLE`, Unity Catalog removes the table definition from the list, but **the actual files remain on S3**.
* **Best For:** Data that is shared with other tools (outside Databricks) or data that must persist even if the Databricks workspace is deleted.

```sql
-- Location specified = External Table
CREATE TABLE prod_catalog.finance.legacy_data (
  id INT,
  data STRING
)
LOCATION 's3://my-company-bucket/finance/legacy_data';

```

### Comparison Summary

| Feature | Managed Table | External Table |
| --- | --- | --- |
| **Storage Location** | Auto-managed (Root or Schema default). | User-defined path (S3/ADLS/GCS). |
| **Performance** | Higher (Access to some proprietary optimizations). | Standard. |
| **DROP TABLE** | Deletes Metadata + **Deletes Files**. | Deletes Metadata + **Keeps Files**. |
| **Use Case** | General analytics, temporary data, sandbox. | Bronze ingestion (raw files), shared data. |