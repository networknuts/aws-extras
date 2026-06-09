# AWS FSx for Lustre POC Lab

## Objective

Build a high-performance shared file system using AWS FSx for Lustre and demonstrate:

1. High-speed parallel file access
2. Integration with Amazon S3
3. HPC (High Performance Computing) workloads
4. AI/ML training data storage
5. Analytics and simulation workloads

---

# What is AWS FSx for Lustre?

Amazon FSx for Lustre is a fully managed, high-performance file system designed for workloads that require extremely fast storage access.

"Lustre" stands for:

```text
Linux + Cluster
```

It is one of the most widely used file systems in:

* Supercomputers
* AI/ML Training
* GenAI Workloads
* Scientific Simulations
* Financial Modeling
* Media Rendering

---

# Why Not Use EBS or EFS?

| Feature         | EBS      | EFS      | FSx Lustre     |
| --------------- | -------- | -------- | -------------- |
| Shared Storage  | No       | Yes      | Yes            |
| Parallel Access | No       | Limited  | Yes            |
| HPC Workloads   | No       | Moderate | Excellent      |
| AI Training     | Moderate | Moderate | Excellent      |
| Throughput      | Medium   | High     | Extremely High |

---

# Real-World Problem

Imagine training a Large Language Model.

You have:

```text
50 TB Training Data
20 GPU Nodes
```

Every GPU server must read the same dataset.

Using local disks:

```text
GPU 1 -> Copy Dataset
GPU 2 -> Copy Dataset
GPU 3 -> Copy Dataset
```

Huge storage waste.

---

With FSx Lustre:

```text
GPU Nodes
     |
     v
FSx Lustre
     |
     v
Shared Dataset
```

All nodes access the same files simultaneously.

---

# How FSx for Lustre Works

```text
Compute Nodes
      |
      v
FSx Lustre
      |
      v
Amazon S3
```

Frequently used pattern:

```text
S3 = Data Lake

FSx Lustre = High-Speed Working Storage
```

---

# Common Use Cases

## AI / Machine Learning

Training:

```text
Images
Videos
Documents
Embeddings
```

Used with:

* Amazon SageMaker
* GPU Clusters
* Kubernetes AI Platforms

---

## Scientific Simulations

Examples:

```text
Weather Models
Fluid Dynamics
Physics Simulations
```

---

## Media Rendering

Rendering:

```text
Movies
Animations
Visual Effects
```

Thousands of rendering nodes read/write simultaneously.

---

## Financial Modeling

Examples:

```text
Risk Calculations
Trading Simulations
Monte Carlo Analysis
```

---

# POC Architecture

```text
S3 Dataset
     |
     v
FSx Lustre
     |
     v
EC2 Linux Nodes
```

Services Used:

* S3
* FSx Lustre
* EC2
* IAM

---

# POC Scenario

Network Nuts wants to demonstrate AI training.

Dataset:

```text
10,000 Images
```

stored in S3.

Training servers:

```text
EC2 Instances
```

need fast access.

---

# Step 1: Create S3 Bucket

Create:

```text
networknuts-lustre-demo
```

Upload:

```text
images/
documents/
videos/
```

Example:

```bash
aws s3 cp dataset/ s3://networknuts-lustre-demo/ --recursive
```

---

# Step 2: Create FSx Lustre

Navigate:

```text
FSx
→ Create File System
```

Choose:

```text
Lustre
```

Deployment Type:

```text
Scratch 2
```

for labs.

---

## Scratch vs Persistent

### Scratch

```text
Temporary
Cheaper
Faster
```

Best for:

```text
Training
Analytics
Testing
```

---

### Persistent

```text
Durable
Replicated
Production
```

Best for:

```text
Long-running workloads
```

---

# Step 3: Link S3 Bucket

Select:

```text
Import Data Repository
```

Choose:

```text
s3://networknuts-lustre-demo
```

AWS automatically imports metadata.

---

# Step 4: Launch Linux EC2

Launch:

```text
Amazon Linux 2023
```

or

```text
Ubuntu
```

Same VPC as FSx.

---

# Step 5: Install Lustre Client

Amazon Linux:

```bash
sudo dnf install -y lustre-client
```

Ubuntu:

```bash
sudo apt install -y lustre-client-modules
```

---

# Step 6: Mount File System

Create mount point:

```bash
sudo mkdir /fsx
```

Mount:

```bash
sudo mount -t lustre \
fs-012345678.fsx.us-east-1.amazonaws.com@tcp:/fsx \
/fsx
```

Verify:

```bash
df -h
```

---

# Step 7: Verify Imported Data

Check files:

```bash
ls -lah /fsx
```

Output:

```text
images
videos
documents
```

Files from S3 are visible immediately.

---

# Step 8: Demonstrate High-Speed Access

Copy files:

```bash
time cp -r /fsx/images /tmp/
```

Observe:

```text
High Throughput
Low Latency
```

---

# Step 9: Write New Files

Create file:

```bash
echo "AI Training Data" > /fsx/demo.txt
```

Export back to S3:

```bash
aws fsx create-data-repository-task
```

or configure automatic export.

---

# Step 10: Multiple Clients Demo

Launch:

```text
EC2 Node 1
EC2 Node 2
EC2 Node 3
```

Mount:

```text
/fsx
```

on all nodes.

Create file:

```bash
Node 1:
touch /fsx/test.txt
```

Immediately visible on:

```text
Node 2
Node 3
```

Demonstrates shared storage.

---

# AI/ML Architecture

A common architecture:

```text
Training Data
      |
      v
Amazon S3
      |
      v
FSx Lustre
      |
      v
GPU Cluster
      |
      v
Model Output
```

Examples:

* Computer Vision
* LLM Fine-Tuning
* Deep Learning

---

# Kubernetes AI Example

Relevant to your Kubeflow/KServe training.

```text
S3
 |
 v
FSx Lustre
 |
 v
Kubernetes
 |
 +-- Training Pods
 +-- Katib Experiments
 +-- Kubeflow Pipelines
```

Training jobs share the same dataset.

---

# HPC Architecture

```text
Compute Cluster
      |
      v
FSx Lustre
      |
      v
Petabytes of Data
```

Used in:

* Research Labs
* Universities
* Weather Forecasting

---

# S3 Integration

One major advantage:

```text
S3
 |
 v
FSx Lustre
```

Data appears as:

```text
Regular Files
```

Applications don't need S3 APIs.

---

# Advanced POC 1: AI Image Training

Dataset:

```text
10,000 Images
```

stored in S3.

Training script:

```python
for image in os.listdir("/fsx/images"):
    train(image)
```

No need to download files manually.

---

# Advanced POC 2: Shared Data Science Platform

```text
Data Scientists
      |
      v
FSx Lustre
```

Everyone accesses:

```text
Datasets
Models
Results
```

centrally.

---

# Advanced POC 3: Kubernetes Training Jobs

Mount FSx into pods.

```text
Kubeflow Training Job
      |
      v
FSx Lustre
```

All pods share:

```text
Training Data
Checkpoints
Logs
```

---

# Interview Questions

### What is FSx for Lustre?

**Answer:**
A fully managed high-performance file system optimized for HPC, analytics, and machine learning workloads.

---

### Why is it popular for AI?

**Answer:**
It provides extremely high throughput and parallel file access, allowing many GPU nodes to read training datasets simultaneously.

---

### What is the relationship between S3 and FSx Lustre?

**Answer:**
FSx Lustre can import data from S3 and export results back to S3 while presenting files through a POSIX-compatible file system.

---

### What is Scratch Deployment?

**Answer:**
A temporary, lower-cost deployment suitable for short-lived workloads such as training jobs and analytics.

---

### What is Persistent Deployment?

**Answer:**
A durable deployment designed for long-running production workloads requiring higher availability.

---

### When would you choose FSx Lustre over EFS?

**Answer:**
When workloads require very high throughput, parallel access, AI training, HPC simulations, or large-scale analytics.

---

# Learning Outcomes

After completing this lab, students should understand:

1. What FSx for Lustre is.
2. Why HPC workloads require specialized storage.
3. Integration between FSx Lustre and S3.
4. Scratch vs Persistent deployment models.
5. Shared storage for AI/ML training.
6. High-performance file system concepts.
7. Multi-node parallel file access.
8. Kubernetes and Kubeflow integration possibilities.
9. Real-world HPC architectures on AWS.
10. Common FSx Lustre interview questions and enterprise use cases.
