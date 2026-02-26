


# 2️⃣ Medallion Architecture (Bronze → Silver → Gold)

## ✅ Concept

In real-world data engineering, we use layered architecture:

| Layer      | Purpose                        |
| ---------- | ------------------------------ |
| **Bronze** | Raw data (as received)         |
| **Silver** | Cleaned & transformed data     |
| **Gold**   | Business-ready aggregated data |

This helps maintain:

* Data quality
* Traceability
* Scalability
* Production safety

Example:

```
catalog_name.bronze.customers
catalog_name.silver.customers_enriched
catalog_name.gold.customer_summary
```

---

# 3️⃣ Reading Data in PySpark

## ✅ Concept: DataFrame Reader API

We use:

```python
spark.read
```

### Example: Reading CSV

```python
df = spark.read.format("csv") \
    .option("inferSchema", True) \
    .option("header", True) \
    .load("path/to/file")
```

### Important Options

| Option             | Meaning                                  |
| ------------------ | ---------------------------------------- |
| `inferSchema=True` | Spark automatically detects column types |
| `header=True`      | First row is column names                |

---

## ✅ Why inferSchema is Important?

Without it:

* All columns become **string**

With it:

* Spark automatically detects:

  * Integer
  * Date
  * String
  * Double

Check schema:

```python
df.printSchema()
```

---

# 4️⃣ Basic Transformations

Transformations are changes applied to DataFrames.

---

## 🔹 A. Updating or Creating Columns

### Function: `withColumn()`

```python
df = df.withColumn("column_name", transformation)
```

If column exists → it updates
If column doesn't exist → it creates

---

### Example 1: Convert Name to Uppercase

```python
from pyspark.sql.functions import *

df = df.withColumn("name", upper("name"))
```

---

## 🔹 B. Extract Domain from Email

### Concept: `split()` function

Splits a column based on delimiter.

Example:

```
john@gmail.com → ["john", "gmail.com"]
```

Code:

```python
df = df.withColumn(
    "domain",
    split("email", "@")[1]
)
```

---

# 5️⃣ Aggregations

## ✅ Concept: Group By

Similar to SQL:

```sql
SELECT domain, COUNT(customer_id)
FROM table
GROUP BY domain
```

### PySpark Version:

```python
df.groupBy("domain") \
  .agg(count("customer_id").alias("total_customers"))
```

---

## 🔹 Sorting Results

```python
from pyspark.sql.functions import col

df.groupBy("domain") \
  .agg(count("customer_id").alias("total_customers")) \
  .sort(col("total_customers").desc())
```

⚠ Important: Use `col()` before `.desc()`

---

# 6️⃣ Visualization in Databricks

## ✅ Concept

Databricks allows you to:

* Convert table results into charts
* Create bar charts, pie charts, line charts
* Add filters using natural language

Example:

* Pie chart of customers by email domain
* Bar chart of average product price by category

Benefits:

* Quick analysis
* No need to wait for dashboards
* Great for developers

---

# 7️⃣ Adding Processing Timestamp

## ✅ Concept

Every transformed dataset should have:

```
processed_date
```

Why?

* Helps track batch timing
* Supports incremental loads
* Helps debugging

### Example:

```python
df = df.withColumn("process_date", current_timestamp())
```

---

# 8️⃣ Writing Data – DataFrame Writer API

## ✅ Concept

To save data:

```python
df.write
```

---

## 🔹 Writing in Delta Format

```python
df.write.format("delta") \
    .mode("append") \
    .saveAsTable("catalog.silver.table_name")
```

---

# 9️⃣ Write Modes Explained

| Mode      | Behavior                     |
| --------- | ---------------------------- |
| append    | Adds new data                |
| overwrite | Deletes old data, writes new |
| error     | Throws error if table exists |
| ignore    | Does nothing if table exists |

---

# 🔟 Upsert (Merge) – Most Important Concept

## ✅ Concept

Upsert = Update + Insert

If record exists → Update
If record doesn’t exist → Insert

Prevents duplicates.

---

## Why Upsert is Important?

Without Upsert:

Run 1 → 200 rows
Run 2 → 400 rows ❌ (duplicates)

With Upsert:

Run 1 → 200 rows
Run 2 → 200 rows ✅

---

## 🔹 How to Implement Upsert in PySpark

### Step 1: Check if table exists

```python
if spark.catalog.tableExists("catalog.silver.table_name"):
```

---

### Step 2: Create Delta Table Object

```python
from delta.tables import DeltaTable

delta_obj = DeltaTable.forName(
    spark,
    "catalog.silver.table_name"
)
```

---

### Step 3: Apply Merge

```python
delta_obj.alias("target") \
    .merge(
        df.alias("source"),
        "target.id = source.id"
    ) \
    .whenMatchedUpdateAll() \
    .whenNotMatchedInsertAll() \
    .execute()
```

---

# 1️⃣1️⃣ Real-World Example Tables

### Customers (Dimension Table)

* Customer ID
* Name
* Email
* Domain extracted
* Process date

---

### Products (Dimension Table)

Example Aggregation:

```python
df.groupBy("category") \
  .agg(avg("price"))
```

Use case:

* Find average product price per category

---

### Stores (Dimension Table)

Example Cleaning:

Remove underscore:

```python
df = df.withColumn(
    "store_name",
    regexp_replace("store_name", "_", "")
)
```

---

### Sales (Fact Table)

Fact tables:

* Contain numeric measures
* Link to dimension IDs

Example: Price per Sale

```python
df = df.withColumn(
    "price_per_sale",
    round(col("total_amount") / col("quantity"), 2)
)
```

---

# 1️⃣2️⃣ Fact vs Dimension Table

| Dimension           | Fact                    |
| ------------------- | ----------------------- |
| Descriptive data    | Numeric measurable data |
| Customers, Products | Sales, Transactions     |
| Smaller size        | Large size              |

---

# 1️⃣3️⃣ Production Best Practices

✅ Always add process_date
✅ Use Delta format
✅ Use Upsert instead of append
✅ Separate Bronze, Silver, Gold
✅ Test merge twice to confirm no duplicates
✅ Use catalog.schema.table naming

---

# 1️⃣4️⃣ Environment Strategy

Real companies use:

| Environment | Purpose     |
| ----------- | ----------- |
| Dev         | Development |
| QA          | Testing     |
| Prod        | Production  |

Each environment has:

```
bronze
silver
gold
```

---

# 1️⃣5️⃣ Debugging in Databricks

Common mistakes:

* Missing bracket `)`
* Wrong alias
* Wrong DataFrame variable
* Case-sensitive column names

Use:

* Diagnose Error
* printSchema()
* display()

---



---

# 🏁 Final Understanding

Bronze = Raw
Silver = Clean + Enriched + Processed
Gold = Business Ready






