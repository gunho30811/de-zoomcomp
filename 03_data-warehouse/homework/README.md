# Module 3 Homework — Data Warehousing & BigQuery (Zoomcamp 2026)

## Dataset

* **Yellow Taxi Trip Records**
* **Period:** January 2024 – June 2024
* **Format:** Parquet
* **Source:** NYC TLC Trip Record Data
* **Pipeline Flow:**
  **Public NYC source → Local download → Google Cloud Storage → BigQuery External Table → BigQuery Materialized Table**

---

# Loading Data to Google Cloud Storage

The Parquet files for **January–June 2024** were downloaded locally and uploaded to a **GCS bucket** under:

```
gs://kestra-zoomcamp-will-demo-485315/yellow/2024/
```

Files used:
- yellow_tripdata_2024-01.parquet
- yellow_tripdata_2024-02.parquet
- yellow_tripdata_2024-03.parquet
- yellow_tripdata_2024-04.parquet
- yellow_tripdata_2024-05.parquet
- yellow_tripdata_2024-06.parquet


Only the first **six monthly files** were used to match the homework requirements.

---

# BigQuery Setup

## Create External Table

```sql
CREATE OR REPLACE EXTERNAL TABLE zoomcamp.yellow_2024_ext
OPTIONS (
  format = 'PARQUET',
  uris = [
    'gs://kestra-zoomcamp-will-demo-485315/yellow/2024/yellow_tripdata_2024-01.parquet',
    'gs://kestra-zoomcamp-will-demo-485315/yellow/2024/yellow_tripdata_2024-02.parquet',
    'gs://kestra-zoomcamp-will-demo-485315/yellow/2024/yellow_tripdata_2024-03.parquet',
    'gs://kestra-zoomcamp-will-demo-485315/yellow/2024/yellow_tripdata_2024-04.parquet',
    'gs://kestra-zoomcamp-will-demo-485315/yellow/2024/yellow_tripdata_2024-05.parquet',
    'gs://kestra-zoomcamp-will-demo-485315/yellow/2024/yellow_tripdata_2024-06.parquet'
  ]
);
```

## Create Materialized Table

```sql
CREATE OR REPLACE TABLE zoomcamp.yellow_2024 AS
SELECT * FROM zoomcamp.yellow_2024_ext;
```

---

# Question 1. Counting Records

```sql
SELECT COUNT(*) FROM zoomcamp.yellow_2024;
```

**Answer:**
**20,332,093**

---

# Question 2. Data Read Estimation

```sql
-- External table
SELECT DISTINCT PULocationID FROM zoomcamp.yellow_2024_ext;

-- Materialized table
SELECT DISTINCT PULocationID FROM zoomcamp.yellow_2024;
```

**Answer:**
**0 MB for the External Table and 155.12 MB for the Materialized Table**

---

# Question 3. Understanding Columnar Storage

```sql
SELECT PULocationID FROM zoomcamp.yellow_2024;
SELECT PULocationID, DOLocationID FROM zoomcamp.yellow_2024;
```

**Answer:**
BigQuery uses **columnar storage**, so it scans only the requested columns.
Querying two columns requires scanning more data than querying one column, increasing the estimated bytes processed.

---

# Question 4. Counting Zero Fare Trips

```sql
SELECT COUNT(*)
FROM zoomcamp.yellow_2024
WHERE fare_amount = 0;
```

**Answer:**
**8,333**

---

# Question 5. Partitioning and Clustering Strategy

```sql
CREATE OR REPLACE TABLE zoomcamp.yellow_2024_part
PARTITION BY DATE(tpep_dropoff_datetime)
CLUSTER BY VendorID AS
SELECT * FROM zoomcamp.yellow_2024;
```

**Answer:**
**Partition by tpep_dropoff_datetime and Cluster on VendorID**

---

# Question 6. Partition Benefits

```sql
SELECT DISTINCT VendorID
FROM zoomcamp.yellow_2024
WHERE DATE(tpep_dropoff_datetime)
BETWEEN '2024-03-01' AND '2024-03-15';

SELECT DISTINCT VendorID
FROM zoomcamp.yellow_2024_part
WHERE DATE(tpep_dropoff_datetime)
BETWEEN '2024-03-01' AND '2024-03-15';
```

**Answer:**
**310.24 MB for the non-partitioned table and 26.84 MB for the partitioned table**

---

# Question 7. External Table Storage Location

**Answer:**
**GCP Bucket**

---

# Question 8. Clustering Best Practice

**Answer:**
**False**

---

# Question 9. Table Scan Explanation (Not Graded)

```sql
SELECT COUNT(*) FROM zoomcamp.yellow_2024;
```

**Explanation:**
`COUNT(*)` scans the entire table.
Since partition pruning cannot be applied, more bytes must be read.