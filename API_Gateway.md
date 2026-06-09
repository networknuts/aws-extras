# AWS API Gateway POC Lab

## Objective

Build a serverless API using AWS API Gateway and demonstrate:

1. Creating REST APIs
2. Exposing backend services securely
3. Integrating with Lambda
4. Authentication and throttling
5. Building production-grade API architectures

---

# What is AWS API Gateway?

Amazon API Gateway is a fully managed service for creating, publishing, securing, monitoring, and managing APIs.

Think of it as the front door for your applications.

```text
Users
  |
  v
API Gateway
  |
  v
Backend Services
```

Instead of exposing applications directly, users communicate through API Gateway.

---

# Why Do We Need API Gateway?

Without API Gateway:

```text
Client
  |
  v
EC2 Application
```

Problems:

* No centralized security
* No rate limiting
* No API management
* Difficult monitoring

---

With API Gateway:

```text
Client
  |
  v
API Gateway
  |
  +---- Authentication
  +---- Authorization
  +---- Rate Limiting
  +---- Monitoring
  |
  v
Backend Services
```

---

# Real-World Example

Imagine Network Nuts has:

```text
Student Portal
Course APIs
Payment APIs
Certificate APIs
```

Without API Gateway:

```text
Users
 |
 +--- app1.company.com
 |
 +--- app2.company.com
 |
 +--- app3.company.com
```

Hard to manage.

---

With API Gateway:

```text
api.company.com
 |
 +--- /courses
 |
 +--- /payments
 |
 +--- /certificates
```

Single API entry point.

---

# How API Gateway Works

```text
User
 |
 v
API Gateway
 |
 +---- Validate Request
 |
 +---- Authenticate User
 |
 +---- Apply Throttling
 |
 +---- Log Request
 |
 v
Backend
```

---

# API Types

## REST API

Traditional API management.

Supports:

```text
Authentication
Rate Limiting
Caching
API Keys
```

Most commonly used in enterprises.

---

## HTTP API

Lightweight version.

Benefits:

```text
Lower Cost
Lower Latency
```

Best for modern microservices.

---

## WebSocket API

Used for:

```text
Real-Time Chat
Gaming
Notifications
```

---

# POC Architecture

```text
Client
 |
 v
API Gateway
 |
 v
AWS Lambda
 |
 v
JSON Response
```

Services Used:

* API Gateway
* Lambda
* IAM
* CloudWatch

---

# POC Scenario

Build:

```text
GET /student
```

API

Returns:

```json
{
  "name":"Aryan",
  "course":"DevOps"
}
```

---

# Step 1: Create Lambda Function

Navigate:

```text
Lambda
→ Create Function
```

Name:

```text
student-api
```

Runtime:

```text
Python 3.12
```

Code:

```python
import json

def lambda_handler(event, context):

    return {
        "statusCode": 200,
        "body": json.dumps({
            "name": "Aryan",
            "course": "DevOps"
        })
    }
```

Deploy.

---

# Step 2: Create API Gateway

Navigate:

```text
API Gateway
→ Create API
```

Choose:

```text
HTTP API
```

---

# Step 3: Configure Integration

Integration:

```text
Lambda Function
```

Select:

```text
student-api
```

---

# Step 4: Create Route

Route:

```text
GET /student
```

Attach:

```text
student-api Lambda
```

---

# Step 5: Deploy API

Deploy:

```text
Default Stage
```

Example URL:

```text
https://abc123.execute-api.us-east-1.amazonaws.com/student
```

---

# Step 6: Test API

Request:

```bash
curl https://api-url/student
```

Response:

```json
{
  "name":"Aryan",
  "course":"DevOps"
}
```

---

# Step 7: Add Query Parameters

Lambda:

```python
import json

def lambda_handler(event, context):

    student_id = event["queryStringParameters"]["id"]

    return {
        "statusCode":200,
        "body":json.dumps({
            "student_id":student_id
        })
    }
```

Request:

```bash
curl "https://api-url/student?id=101"
```

Response:

```json
{
  "student_id":"101"
}
```

---

# Step 8: Add POST Request

Create route:

```text
POST /student
```

Lambda:

```python
import json

def lambda_handler(event, context):

    body = json.loads(event["body"])

    return {
        "statusCode":200,
        "body":json.dumps({
            "message":"Student Created",
            "name":body["name"]
        })
    }
```

Request:

```bash
curl -X POST \
-H "Content-Type: application/json" \
-d '{"name":"Aryan"}' \
https://api-url/student
```

---

# Step 9: Enable Authentication

Options:

* IAM
* JWT
* Lambda Authorizer
* Cognito

Example:

```text
JWT Authentication
```

Users must provide:

```text
Bearer Token
```

before accessing APIs.

---

# Step 10: Enable Throttling

Example:

```text
100 Requests / Second
```

If exceeded:

```text
429 Too Many Requests
```

Protects backend systems.

---

# CloudWatch Monitoring

API Gateway automatically publishes:

* Request Count
* Error Count
* Latency
* Integration Latency
* 4XX Errors
* 5XX Errors

Example:

```text
Requests: 10,000

4XX Errors: 100

5XX Errors: 10

Average Latency: 50ms
```

---

# Real-World Architectures

## Serverless API

```text
Users
  |
  v
API Gateway
  |
  v
Lambda
```

Very common startup architecture.

---

## Microservices

```text
Users
  |
  v
API Gateway
  |
  +---- User Service
  |
  +---- Course Service
  |
  +---- Payment Service
```

Single API endpoint.

---

## Kubernetes

Relevant to your training environment.

```text
Users
  |
  v
API Gateway
  |
  v
Application Load Balancer
  |
  v
Kubernetes Services
```

API Gateway handles security and throttling.

---

## AI Applications

Very relevant to your AI Interview Evaluator.

```text
Frontend
    |
    v
API Gateway
    |
    v
Lambda
    |
    v
OpenAI API
```

Benefits:

* Authentication
* Rate Limiting
* Usage Tracking

---

# Advanced POC 1: API Gateway + SQS

```text
User
 |
 v
API Gateway
 |
 v
SQS
 |
 v
Worker
```

User receives immediate response.

Background processing continues.

Perfect for:

* Resume Evaluators
* AI Image Generation
* Video Processing

---

# Advanced POC 2: API Gateway + ECS

```text
Users
  |
  v
API Gateway
  |
  v
ECS Service
```

Microservice architecture.

---

# Advanced POC 3: API Gateway + Lambda + DynamoDB

```text
User
 |
 v
API Gateway
 |
 v
Lambda
 |
 v
DynamoDB
```

Classic serverless architecture.

---

# API Gateway vs Load Balancer

| Feature                | API Gateway     | ALB                 |
| ---------------------- | --------------- | ------------------- |
| API Management         | Yes             | No                  |
| Authentication         | Yes             | Limited             |
| Rate Limiting          | Yes             | No                  |
| API Keys               | Yes             | No                  |
| Request Transformation | Yes             | No                  |
| Cost                   | Higher          | Lower               |
| Layer                  | Application/API | HTTP Load Balancing |

---

# Common Use Cases

### Mobile Backends

```text
Mobile App
     |
     v
API Gateway
```

---

### SaaS Platforms

```text
Customers
      |
      v
API Gateway
```

---

### AI Platforms

```text
Frontend
      |
      v
API Gateway
      |
      v
OpenAI / LLM Backend
```

---

### Internal APIs

```text
Developers
      |
      v
API Gateway
```

---

# Interview Questions

### What is API Gateway?

**Answer:**
A managed AWS service that provides API creation, security, monitoring, throttling, authentication, and routing capabilities.

---

### What are the API Gateway API types?

**Answer:**

1. REST API
2. HTTP API
3. WebSocket API

---

### What is the difference between REST API and HTTP API?

| Feature  | REST API | HTTP API    |
| -------- | -------- | ----------- |
| Features | Rich     | Lightweight |
| Cost     | Higher   | Lower       |
| Latency  | Higher   | Lower       |

---

### Can API Gateway invoke Lambda?

**Answer:**
Yes. Lambda integration is one of the most common API Gateway use cases.

---

### What is throttling?

**Answer:**
Limiting the number of requests that a client can send in a given time period to protect backend systems.

---

### Why use API Gateway instead of exposing Lambda directly?

**Answer:**
API Gateway provides routing, authentication, authorization, throttling, monitoring, API versioning, and usage plans.

---

# Learning Outcomes

After completing this lab, students should understand:

1. What API Gateway is.
2. REST vs HTTP vs WebSocket APIs.
3. API Gateway and Lambda integration.
4. Request routing concepts.
5. Authentication and authorization.
6. Throttling and rate limiting.
7. CloudWatch monitoring.
8. Microservices API patterns.
9. Serverless architectures.
10. Real-world enterprise API management using AWS API Gateway.
