# AWS SQS POC Lab

## Objective

Build a decoupled application using AWS SQS and demonstrate:

1. Sending messages to a queue
2. Processing messages asynchronously
3. Understanding producer-consumer architecture
4. Handling spikes in workload
5. Demonstrating fault tolerance and scalability

---

# What is AWS SQS?

Amazon Simple Queue Service (SQS) is a fully managed message queue service that enables applications to communicate asynchronously.

Instead of applications talking directly to each other:

```text
App A ---> App B
```

they communicate through a queue:

```text
App A ---> SQS ---> App B
```

This decouples applications and improves scalability and reliability.

---

# Why Do We Need SQS?

Without SQS:

```text
Web Application
      |
      v
Image Processing Service
```

If image processing becomes slow or unavailable:

```text
User Requests Fail
```

With SQS:

```text
Web Application
      |
      v
SQS Queue
      |
      v
Image Processing Service
```

The web application remains responsive even if processing is delayed.

---

# Real World Example

Imagine Network Nuts launches a free AI interview evaluation platform.

Thousands of students submit resumes simultaneously.

Without SQS:

```text
Upload Resume
      |
      v
AI Evaluation
      |
      v
Return Result
```

User waits for processing.

---

With SQS:

```text
Upload Resume
      |
      v
SQS Queue
      |
      v
Worker Processes Later
```

User receives:

```text
Request Accepted
```

and processing happens asynchronously.

---

# How SQS Works

```text
Producer
    |
    v
+------------+
|    SQS     |
+------------+
    |
    v
Consumer
```

Producer:

* Sends messages

Consumer:

* Receives messages
* Processes messages
* Deletes messages

---

# Queue Types

## Standard Queue

Characteristics:

```text
Unlimited Throughput
At-Least-Once Delivery
Best-Effort Ordering
```

Use Cases:

* Logs
* Orders
* Notifications

---

## FIFO Queue

Characteristics:

```text
Exactly Once Processing
Strict Ordering
```

Use Cases:

* Banking
* Financial Transactions
* Payment Systems

---

# POC Architecture

```text
User
 |
 v
FastAPI Application
 |
 v
Amazon SQS
 |
 v
Python Worker
 |
 v
Result Database
```

Services Used:

* EC2
* SQS
* IAM
* Python
* CloudWatch

---

# POC Scenario

Student uploads a PDF.

Instead of immediately processing:

```text
PDF Upload
      |
      v
SQS Message
      |
      v
Worker
      |
      v
Process PDF
```

---

# Step 1: Create Queue

Navigate:

```text
SQS
→ Create Queue
```

Name:

```text
student-processing-queue
```

Type:

```text
Standard
```

---

# Step 2: Create IAM Role

Permissions:

```json
{
  "Effect": "Allow",
  "Action": [
    "sqs:SendMessage",
    "sqs:ReceiveMessage",
    "sqs:DeleteMessage"
  ],
  "Resource": "*"
}
```

Attach role to EC2.

---

# Step 3: Producer Application

Install boto3:

```bash
pip install boto3
```

Producer code:

```python
import boto3
import json

sqs = boto3.client("sqs")

QUEUE_URL = "YOUR_QUEUE_URL"

response = sqs.send_message(
    QueueUrl=QUEUE_URL,
    MessageBody=json.dumps({
        "student":"Aryan",
        "course":"DevOps"
    })
)

print(response["MessageId"])
```

Run:

```bash
python producer.py
```

Message enters queue.

---

# Step 4: Verify Queue

Open:

```text
SQS
→ Queue
→ Monitoring
```

Observe:

```text
Messages Available = 1
```

---

# Step 5: Consumer Application

consumer.py

```python
import boto3

sqs = boto3.client("sqs")

QUEUE_URL = "YOUR_QUEUE_URL"

while True:

    response = sqs.receive_message(
        QueueUrl=QUEUE_URL,
        MaxNumberOfMessages=1,
        WaitTimeSeconds=20
    )

    messages = response.get("Messages", [])

    for message in messages:

        print("Processing:", message["Body"])

        sqs.delete_message(
            QueueUrl=QUEUE_URL,
            ReceiptHandle=message["ReceiptHandle"]
        )
```

Run:

```bash
python consumer.py
```

Output:

```text
Processing:
{
 "student":"Aryan",
 "course":"DevOps"
}
```

---

# Step 6: Demonstrate Asynchronous Processing

Send:

```text
100 Messages
```

using loop:

```python
for i in range(100):
    sqs.send_message(...)
```

Observe:

```text
Queue Depth = 100
```

Workers process gradually.

---

# Step 7: Scale Consumers

Start:

```text
1 Worker
```

Queue drains slowly.

Start:

```text
5 Workers
```

Queue drains much faster.

Demonstrates horizontal scaling.

---

# Step 8: Demonstrate Fault Tolerance

Stop consumer.

Send:

```text
50 Messages
```

Messages remain safely stored.

Restart consumer.

Processing resumes automatically.

No data loss.

---

# Step 9: Visibility Timeout Demo

Consumer receives message.

Before deleting it:

```text
Worker Crashes
```

Message becomes invisible temporarily.

After timeout:

```text
Message Reappears
```

Another worker processes it.

This prevents message loss.

---

# Step 10: Dead Letter Queue (DLQ)

Create:

```text
student-processing-dlq
```

Configure:

```text
Max Receives = 3
```

If message fails repeatedly:

```text
Main Queue
      |
      v
Dead Letter Queue
```

Useful for troubleshooting bad messages.

---

# CloudWatch Monitoring

Open:

```text
CloudWatch
```

Monitor:

* Messages Sent
* Messages Received
* Queue Depth
* Oldest Message Age

Example:

```text
Messages Sent: 5000

Messages Processed: 4900

Messages Failed: 100
```

---

# Real World Architectures

## E-Commerce Orders

```text
Website
   |
   v
SQS
   |
   v
Order Processor
```

Example:

Amazon

uses queue-based architectures extensively.

---

## Video Processing

```text
Video Upload
      |
      v
SQS
      |
      v
Transcoding Workers
```

---

## Email Processing

```text
Application
      |
      v
SQS
      |
      v
Email Service
```

Users do not wait for email delivery.

---

## AI Workloads

Very relevant to your AI interview evaluator or image generation projects.

```text
Frontend
     |
     v
SQS
     |
     v
AI Workers
     |
     v
OpenAI API
```

Benefits:

* Rate limiting protection
* Burst handling
* Retry mechanisms

---

# Standard vs FIFO

| Feature    | Standard    | FIFO              |
| ---------- | ----------- | ----------------- |
| Throughput | Very High   | Lower             |
| Ordering   | Best Effort | Guaranteed        |
| Duplicates | Possible    | Prevented         |
| Use Case   | Most Apps   | Financial Systems |

---

# Advanced POC (AWS Native)

```text
User Upload
      |
      v
S3 Bucket
      |
      v
Lambda Trigger
      |
      v
SQS
      |
      v
ECS Workers
      |
      v
RDS Database
```

This architecture is very common in production.

---

# Interview Questions

### What problem does SQS solve?

**Answer:**
SQS decouples applications and enables asynchronous communication, improving scalability, fault tolerance, and reliability.

---

### What is Visibility Timeout?

**Answer:**
It temporarily hides a message after a consumer receives it, preventing multiple consumers from processing the same message simultaneously.

---

### What is a Dead Letter Queue?

**Answer:**
A DLQ stores messages that repeatedly fail processing, enabling troubleshooting and preventing endless retries.

---

### Difference between Standard and FIFO Queue?

**Answer:**

**Standard Queue**

* Higher throughput
* Possible duplicate messages
* Best-effort ordering

**FIFO Queue**

* Exactly-once processing
* Strict ordering
* Lower throughput

---

### Why use SQS with ECS or Lambda?

**Answer:**
SQS acts as a buffer between request generation and processing, allowing ECS tasks or Lambda functions to scale independently.

---

# Learning Outcomes

After completing this lab, students should understand:

1. What message queues are.
2. Why asynchronous processing is important.
3. Producer-consumer architecture.
4. Standard vs FIFO queues.
5. Visibility timeout.
6. Dead Letter Queues.
7. Scaling consumers.
8. Fault tolerance and retries.
9. CloudWatch monitoring for queues.
10. Real-world cloud-native architectures using SQS.
