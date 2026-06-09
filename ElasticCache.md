# AWS ElastiCache POC Lab

## Objective

Build a caching layer for an application using AWS ElastiCache and demonstrate:

1. Reducing database load
2. Improving application response time
3. Using Redis as an in-memory cache
4. Understanding cache hit and cache miss scenarios
5. Scaling applications using distributed caching

---

# What is AWS ElastiCache?

Amazon ElastiCache is a fully managed in-memory caching service that supports:

* Redis
* Memcached

It stores frequently accessed data in memory rather than retrieving it from a database every time.

Because RAM is much faster than disk storage, applications respond significantly faster.

---

# Why Do We Need ElastiCache?

Without cache:

```text
User
 |
 v
Application
 |
 v
Database
```

Every request hits the database.

Problems:

* High latency
* Increased DB load
* Expensive database scaling

---

With cache:

```text
User
 |
 v
Application
 |
 v
ElastiCache
 |
 +---- Cache Hit --> Return Data
 |
 +---- Cache Miss --> Database
```

Most requests never reach the database.

---

# Real-World Example

Imagine your Network Nuts LMS.

Student opens:

```text
Course Details
```

10,000 students may request:

```text
RHCSA Course Information
```

Without cache:

```text
10,000 Database Queries
```

With cache:

```text
1 Database Query
9999 Cache Reads
```

---

# How ElastiCache Works

```text
Application
     |
     v
ElastiCache
     |
     +------ Found?
     |          |
     |         Yes
     |          |
     |          v
     |      Return Data
     |
     No
     |
     v
Database
     |
     v
Store in Cache
```

---

# Redis vs Memcached

| Feature         | Redis     | Memcached |
| --------------- | --------- | --------- |
| Data Structures | Yes       | No        |
| Persistence     | Yes       | No        |
| Replication     | Yes       | Limited   |
| Pub/Sub         | Yes       | No        |
| Popularity      | Very High | Moderate  |

Most modern applications use Redis.

---

# POC Architecture

```text
User
 |
 v
FastAPI Application
 |
 v
ElastiCache Redis
 |
 v
RDS PostgreSQL
```

Services Used:

* EC2
* ElastiCache Redis
* RDS PostgreSQL
* IAM
* CloudWatch

---

# POC Scenario

Create a student lookup API.

Request:

```text
GET /student/1
```

Application:

1. Check Redis
2. If found → return data
3. If not found → query PostgreSQL
4. Store result in Redis

---

# Step 1: Create RDS Database

Launch:

Amazon RDS

Example database:

```sql
CREATE TABLE students (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  course VARCHAR(100)
);
```

Insert sample data:

```sql
INSERT INTO students VALUES
(1,'Aryan','DevOps'),
(2,'Rahul','AWS'),
(3,'Neha','Kubernetes');
```

---

# Step 2: Create Redis Cluster

Navigate:

```text
ElastiCache
→ Redis OSS
→ Create Cluster
```

Configuration:

```text
Cluster Mode Disabled
Node Type: cache.t3.micro
```

Lab-friendly and inexpensive.

---

# Step 3: Launch FastAPI Server

Install:

```bash
pip install fastapi uvicorn redis psycopg2-binary
```

---

# Step 4: Connect to Redis

```python
import redis

redis_client = redis.Redis(
    host="redis-endpoint",
    port=6379,
    decode_responses=True
)
```

---

# Step 5: Connect to PostgreSQL

```python
import psycopg2

conn = psycopg2.connect(
    host="rds-endpoint",
    database="studentdb",
    user="postgres",
    password="password"
)
```

---

# Step 6: Create API

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/student/{student_id}")
def get_student(student_id: int):

    cache_key = f"student:{student_id}"

    cached = redis_client.get(cache_key)

    if cached:
        return {
            "source": "cache",
            "data": cached
        }

    cur = conn.cursor()

    cur.execute(
        "SELECT * FROM students WHERE id=%s",
        (student_id,)
    )

    result = cur.fetchone()

    redis_client.setex(
        cache_key,
        300,
        str(result)
    )

    return {
        "source": "database",
        "data": result
    }
```

---

# Step 7: Test Cache Miss

First request:

```bash
curl http://server/student/1
```

Response:

```json
{
  "source":"database",
  "data":[1,"Aryan","DevOps"]
}
```

Flow:

```text
Redis Miss
     |
     v
Database Query
     |
     v
Store in Redis
```

---

# Step 8: Test Cache Hit

Second request:

```bash
curl http://server/student/1
```

Response:

```json
{
  "source":"cache",
  "data":"[1,'Aryan','DevOps']"
}
```

Database not accessed.

---

# Demonstrate Performance Improvement

Add artificial DB delay:

```python
import time

time.sleep(2)
```

Results:

| Request    | Response Time |
| ---------- | ------------- |
| Cache Miss | ~2 sec        |
| Cache Hit  | ~5 ms         |

Easy demonstration for students.

---

# Step 9: View Redis Keys

Connect:

```bash
redis-cli
```

Commands:

```bash
KEYS *
```

Output:

```text
student:1
student:2
student:3
```

Check value:

```bash
GET student:1
```

---

# Step 10: Demonstrate TTL

Check remaining TTL:

```bash
TTL student:1
```

Example:

```text
250
```

After expiration:

```text
Key Removed Automatically
```

Next request hits database again.

---

# CloudWatch Monitoring

Monitor:

```text
CPU Usage
Memory Usage
Cache Hits
Cache Misses
Network Throughput
Evictions
```

Useful metrics:

```text
CacheHitRate
CurrConnections
BytesUsedForCache
```

---

# Real-World Architectures

## E-Commerce

```text
Website
   |
   v
Redis Cache
   |
   v
Product Database
```

Product information is heavily cached.

---

## LMS Platform

```text
Student Portal
      |
      v
Redis
      |
      v
Course Database
```

Popular course pages are cached.

---

## Session Storage

Instead of storing sessions locally:

```text
Application
      |
      v
Redis
```

All servers share the same session store.

---

## AI Applications

Very relevant to your RAG applications.

```text
User Query
      |
      v
Redis Cache
      |
      v
OpenAI API
```

Repeated questions:

```text
"What is Kubernetes?"
```

can be served from cache.

This reduces:

* OpenAI cost
* Response time

---

# Advanced POC 1: Distributed Cache

```text
App Server 1
      |
App Server 2
      |
App Server 3
      |
      v
Redis Cluster
```

All application instances share the same cache.

---

# Advanced POC 2: Rate Limiting

Store request counters:

```text
user:123:requests
```

Redis tracks API usage.

Used extensively in:

* APIs
* SaaS products
* Authentication systems

---

# Advanced POC 3: Leaderboard

Redis Sorted Sets:

```python
ZADD leaderboard 100 Aryan
ZADD leaderboard 80 Rahul
```

Query:

```python
ZREVRANGE leaderboard 0 9
```

Useful for:

* Gaming
* Training Platforms
* Rankings

---

# Common Use Cases

### Database Query Caching

```text
Redis
 |
 v
Reduce DB Load
```

---

### Session Storage

```text
Login Sessions
JWT Metadata
User Context
```

---

### API Response Cache

```text
Weather APIs
Stock APIs
AI Responses
```

---

### Queue Systems

Redis can also be used for:

```text
Background Jobs
Task Queues
```

though for AWS-native queues, SQS is usually preferred.

---

# Interview Questions

### What is ElastiCache?

**Answer:**
A fully managed in-memory caching service that supports Redis and Memcached.

---

### Why is Redis faster than a database?

**Answer:**
Redis stores data in RAM rather than reading from disk-based database storage.

---

### What is a Cache Hit?

**Answer:**
Requested data is found in cache and returned without querying the database.

---

### What is a Cache Miss?

**Answer:**
Data is not present in cache, so the application retrieves it from the database and stores it in cache.

---

### What is TTL?

**Answer:**
Time To Live. It determines how long a cache entry remains before automatic expiration.

---

### Why use Redis with RDS?

**Answer:**
Redis reduces database load, lowers latency, and improves application scalability.

---

# Learning Outcomes

After completing this lab, students should understand:

1. What caching is and why it is important.
2. Redis fundamentals.
3. Cache hit vs cache miss.
4. TTL and cache expiration.
5. Database query caching.
6. Session storage architectures.
7. Distributed caching concepts.
8. CloudWatch monitoring for Redis.
9. Cost and performance benefits of caching.
10. Real-world cloud-native architectures using ElastiCache.
