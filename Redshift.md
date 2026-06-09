# AWS Redshift POC Lab

## Objective

Build a small data warehouse using AWS Redshift and demonstrate:

1. Loading data into Redshift
2. Running analytical queries
3. Comparing OLTP vs OLAP workloads
4. Understanding columnar storage
5. Generating business reports from large datasets

---

# What is AWS Redshift?

Amazon Redshift is AWS's fully managed cloud data warehouse service.

It is designed for:

* Analytics
* Business Intelligence (BI)
* Reporting
* Dashboards
* Large-scale aggregations

Unlike a traditional database, Redshift is optimized for reading and analyzing massive amounts of data.

---

# Why Not Use PostgreSQL?

PostgreSQL is excellent for:

```text
Order Processing
Bank Transactions
User Login Systems
Inventory Updates
```

Redshift is excellent for:

```text
Sales Reports
Revenue Analysis
Trend Analysis
Customer Analytics
Executive Dashboards
```

---

# OLTP vs OLAP

| Feature | PostgreSQL/MySQL | Redshift     |
| ------- | ---------------- | ------------ |
| Purpose | Transactions     | Analytics    |
| Reads   | Small            | Massive      |
| Writes  | Frequent         | Batch        |
| Rows    | Millions         | Billions     |
| Queries | Simple           | Complex      |
| Storage | Row Based        | Column Based |

---

# How Redshift Works

```text
Applications
      |
      v
Data Sources
(CSV, RDS, S3, APIs)
      |
      v
Amazon S3
      |
      v
Amazon Redshift
      |
      v
BI Tools
(QuickSight, Tableau, Power BI)
```

---

# POC Architecture

```text
Sales CSV Files
        |
        v
     S3 Bucket
        |
        v
Amazon Redshift
        |
        v
SQL Analytics
        |
        v
QuickSight Dashboard
```

Services Used:

* S3
* Redshift
* IAM
* QuickSight

---

# Use Case

Imagine Network Nuts sells courses.

Each purchase contains:

```text
Order ID
Course Name
Price
Student City
Purchase Date
```

Over time:

```text
10 Million Orders
```

Need answers like:

* Total Revenue
* Monthly Revenue
* Top Courses
* Top Cities
* Revenue Trends

Redshift is built for this.

---

# Step 1: Create Sample Dataset

sales.csv

```csv
order_id,course_name,city,price,purchase_date
1,RHCSA,18000,Delhi,2026-01-01
2,AWS,15000,Mumbai,2026-01-02
3,DevOps,25000,Pune,2026-01-03
4,RHCSA,18000,Delhi,2026-01-05
5,Kubernetes,12500,Bangalore,2026-01-07
```

Generate:

```text
100,000 rows
```

for better analytics.

---

# Step 2: Upload to S3

Create bucket:

```text
networknuts-redshift-demo
```

Upload:

```text
sales.csv
```

---

# Step 3: Create IAM Role

Permissions:

```json
{
  "Effect": "Allow",
  "Action": [
    "s3:GetObject",
    "s3:ListBucket"
  ],
  "Resource": "*"
}
```

Attach role to Redshift.

---

# Step 4: Create Redshift Cluster

Navigate:

```text
Amazon Redshift
→ Create Cluster
```

Configuration:

```text
Node Type:
dc2.large

Nodes:
1

Database:
salesdb

Username:
admin
```

For a lab, a single-node cluster is sufficient.

---

# Step 5: Connect Using Query Editor V2

Open:

```text
Redshift
→ Query Editor V2
```

Connect to:

```text
salesdb
```

---

# Step 6: Create Table

```sql
CREATE TABLE sales (
    order_id INT,
    course_name VARCHAR(100),
    city VARCHAR(100),
    price INT,
    purchase_date DATE
);
```

---

# Step 7: Load Data from S3

```sql
COPY sales
FROM 's3://networknuts-redshift-demo/sales.csv'
IAM_ROLE 'arn:aws:iam::<account>:role/redshift-role'
CSV
IGNOREHEADER 1;
```

Redshift reads data directly from S3.

---

# Step 8: Run Analytics Queries

## Total Revenue

```sql
SELECT SUM(price)
FROM sales;
```

---

## Revenue By Course

```sql
SELECT
course_name,
SUM(price) revenue
FROM sales
GROUP BY course_name
ORDER BY revenue DESC;
```

---

## Revenue By City

```sql
SELECT
city,
SUM(price) revenue
FROM sales
GROUP BY city
ORDER BY revenue DESC;
```

---

## Monthly Revenue

```sql
SELECT
DATE_TRUNC('month', purchase_date),
SUM(price)
FROM sales
GROUP BY 1
ORDER BY 1;
```

---

## Top Selling Course

```sql
SELECT
course_name,
COUNT(*)
FROM sales
GROUP BY course_name
ORDER BY 2 DESC;
```

---

# Step 9: Demonstrate Columnar Storage

Traditional DB:

```text
Row 1
Row 2
Row 3
Row 4
```

Redshift:

```text
Order IDs
Prices
Cities
Dates
```

stored separately.

If query needs:

```sql
SELECT SUM(price)
```

Only the Price column is scanned.

Benefits:

* Faster Queries
* Less Disk I/O
* Better Compression

---

# Step 10: Connect QuickSight

Create dataset:

```text
Amazon Redshift
→ sales table
```

Create visuals:

* Revenue Trend
* Revenue by Course
* Revenue by City
* Orders per Month

---

# Real-World Architecture

```text
Applications
      |
      v
RDS/PostgreSQL
      |
      v
ETL Pipeline
(AWS Glue)
      |
      v
S3 Data Lake
      |
      v
Amazon Redshift
      |
      v
QuickSight
```

---

# Advanced POC (Optional)

Add:

### ETL

AWS Glue

Move data:

```text
CSV
→ Clean
→ Transform
→ Redshift
```

---

### Streaming Analytics

Amazon Kinesis

Flow:

```text
Website Events
        |
        v
Kinesis
        |
        v
Redshift
```

Real-time dashboards become possible.

---

# Common Companies Using Data Warehouses

Examples include:

* Netflix
* Airbnb
* Uber
* Amazon

Typical analytics:

```text
Revenue Trends
Customer Behavior
Product Popularity
Marketing Performance
Forecasting
```

---

# Interview Questions

### Why use Redshift instead of PostgreSQL?

**Answer:**
Redshift is optimized for analytical workloads using columnar storage, compression, parallel query execution, and distributed processing. PostgreSQL is optimized for transactional workloads.

---

### What is Columnar Storage?

**Answer:**
Instead of storing complete rows together, Redshift stores values of the same column together. This dramatically speeds up aggregation queries.

---

### What is the COPY command?

**Answer:**
`COPY` is the preferred method of loading large datasets from S3 into Redshift.

---

### What is Redshift Spectrum?

**Answer:**
Amazon Redshift Spectrum allows Redshift to query data directly in S3 without loading it into Redshift tables.

---

# Learning Outcomes

By completing this lab, students will understand:

1. What a data warehouse is.
2. Difference between OLTP and OLAP.
3. Why Redshift exists.
4. How to load data from S3.
5. How to run analytical SQL queries.
6. Columnar storage concepts.
7. Integration with QuickSight.
8. Real-world analytics architectures using AWS services.
9. ETL and data lake concepts.
10. Common Redshift interview questions and enterprise use cases.
