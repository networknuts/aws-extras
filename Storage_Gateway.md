# AWS Storage Gateway POC Lab

## Objective

Build a hybrid-cloud storage solution using AWS Storage Gateway and demonstrate:

1. Connecting on-premises storage to AWS
2. Storing files locally while backing them up to AWS
3. Accessing cloud storage through NFS/SMB
4. Understanding hybrid cloud storage architectures
5. Using Storage Gateway as a bridge between datacenters and AWS

---

# What is AWS Storage Gateway?

AWS Storage Gateway is a hybrid cloud service that connects on-premises environments with AWS storage services.

It allows servers and applications running in a datacenter to use AWS storage without changing application code.

Think of it as:

```text
Your Datacenter
       |
       v
Storage Gateway
       |
       v
AWS Storage
```

Applications continue using:

* NFS
* SMB
* iSCSI

while data is stored or backed up in AWS.

---

# Why Use Storage Gateway?

Many companies have:

```text
Legacy Applications
VMware Infrastructure
NAS Devices
File Servers
Backup Servers
```

These systems often cannot directly use cloud storage.

Storage Gateway solves this problem.

---

# Types of Storage Gateway

## File Gateway

Provides:

```text
NFS
SMB
```

and stores files in:

```text
Amazon S3
```

---

## Volume Gateway

Provides:

```text
iSCSI Volumes
```

for servers.

Backed by:

```text
Amazon S3
```

with snapshots stored in:

```text
Amazon EBS Snapshots
```

---

## Tape Gateway

Used by backup software.

Appears as:

```text
Virtual Tape Library (VTL)
```

Backed by:

```text
Amazon S3
Amazon S3 Glacier
```

---

# How Storage Gateway Works

```text
Application Server
        |
        v
Storage Gateway
        |
        v
Local Cache
        |
        v
Amazon S3
```

Frequently accessed data remains local.

All data is securely stored in AWS.

---

# POC Architecture

```text
Linux Server
      |
      | NFS Mount
      |
      v
Storage Gateway VM
      |
      v
Amazon S3 Bucket
```

Services Used:

* EC2
* S3
* Storage Gateway
* IAM

---

# POC Scenario

Network Nuts has:

```text
Training Videos
PDF Notes
Assignments
Lab Files
```

stored on a local file server.

Goal:

```text
Keep local access
Automatically store files in AWS
```

without changing applications.

---

# Step 1: Create S3 Bucket

Create bucket:

```text
networknuts-storagegateway-demo
```

This becomes the cloud storage backend.

---

# Step 2: Deploy Storage Gateway

AWS provides:

```text
VMware
Hyper-V
KVM
EC2
```

deployment options.

For lab simplicity:

Deploy gateway as an EC2 instance.

Navigate:

```text
Storage Gateway
→ Create Gateway
```

Choose:

```text
File Gateway
```

---

# Step 3: Activate Gateway

AWS provides:

```text
Activation Key
```

Register gateway.

Allocate:

```text
150 GB Disk
```

Example:

```text
100 GB Cache
50 GB Upload Buffer
```

---

# Step 4: Configure File Share

Choose:

```text
NFS
```

or

```text
SMB
```

Backend:

```text
S3 Bucket
```

Select:

```text
networknuts-storagegateway-demo
```

---

# Step 5: Mount Share on Linux

Example:

```bash
sudo mkdir /gateway-share

sudo mount -t nfs \
<gateway-ip>:/share \
/gateway-share
```

Verify:

```bash
df -h
```

---

# Step 6: Upload Files

Create files:

```bash
echo "AWS Storage Gateway Demo" > demo.txt
```

Copy training videos:

```bash
cp training.mp4 /gateway-share/
```

Copy PDFs:

```bash
cp kubernetes.pdf /gateway-share/
```

---

# Step 7: Verify in S3

Open:

```text
S3 Bucket
```

You should see:

```text
demo.txt
training.mp4
kubernetes.pdf
```

uploaded automatically.

---

# Demonstrate Local Cache

Read file repeatedly:

```bash
cat demo.txt
```

First access:

```text
May retrieve from AWS
```

Subsequent access:

```text
Served from cache
```

Benefits:

* Faster access
* Reduced latency
* Reduced AWS requests

---

# Demonstrate Cloud Backup

Delete local cache disk (simulation).

Files remain available because:

```text
Master Copy
Stored in Amazon S3
```

This demonstrates durability.

---

# Real-World Architecture

```text
Corporate Datacenter
        |
        +----------------+
        |                |
        v                v
Windows Servers    Linux Servers
        |
        v
Storage Gateway
        |
        v
Amazon S3
        |
        v
Glacier Lifecycle
```

---

# Use Case 1: File Server Extension

Instead of purchasing larger NAS devices:

```text
On-Prem NAS
      |
      v
Storage Gateway
      |
      v
Amazon S3
```

Users continue using NFS/SMB.

Storage becomes virtually unlimited.

---

# Use Case 2: Backup Solution

Many backup products support:

```text
Virtual Tape Libraries
```

Storage Gateway can emulate tape drives.

```text
Backup Software
        |
        v
Tape Gateway
        |
        v
S3 / Glacier
```

No physical tapes required.

---

# Use Case 3: Disaster Recovery

Store:

```text
VM Backups
Databases
Application Data
```

in AWS.

If datacenter fails:

```text
Restore from AWS
```

---

# Use Case 4: Media Files

Video companies store:

```text
Videos
Images
Archives
```

locally while automatically syncing to AWS.

---

# Comparison with Direct S3 Access

| Feature        | Direct S3 | Storage Gateway |
| -------------- | --------- | --------------- |
| NFS            | No        | Yes             |
| SMB            | No        | Yes             |
| Local Cache    | No        | Yes             |
| Legacy Apps    | Difficult | Easy            |
| Hybrid Cloud   | Limited   | Excellent       |
| Tape Emulation | No        | Yes             |

---

# Advanced POC

## Lifecycle Policies

Automatically move data:

```text
S3
 |
 v
S3 Glacier Instant Retrieval
 |
 v
S3 Glacier Flexible Retrieval
```

for cost savings.

---

## Cross Region DR

Replicate bucket to another region.

```text
Storage Gateway
       |
       v
S3 Mumbai
       |
       v
S3 Singapore
```

Provides disaster recovery.

---

# Interview Questions

### What problem does Storage Gateway solve?

**Answer:**
It allows on-premises applications to use AWS storage services through familiar protocols like NFS, SMB, and iSCSI without modifying applications.

---

### What are the three types of Storage Gateway?

**Answer:**

1. File Gateway
2. Volume Gateway
3. Tape Gateway

---

### What is File Gateway?

**Answer:**
File Gateway presents NFS or SMB shares and stores files as objects in Amazon S3.

---

### What is Volume Gateway?

**Answer:**
Volume Gateway provides iSCSI block storage backed by AWS and supports EBS snapshots.

---

### What is Tape Gateway?

**Answer:**
Tape Gateway presents virtual tape libraries for backup applications and stores tapes in S3 and Glacier.

---

### Where is data cached?

**Answer:**
Storage Gateway maintains a local cache on the gateway appliance to provide low-latency access to frequently used data.

---

# Learning Outcomes

After this lab, students should understand:

1. What hybrid cloud storage is.
2. Why AWS Storage Gateway exists.
3. File Gateway, Volume Gateway, and Tape Gateway use cases.
4. NFS, SMB, and iSCSI integrations.
5. Local caching concepts.
6. S3 integration.
7. Backup and disaster recovery architectures.
8. Hybrid cloud migration strategies.
9. Storage Gateway interview concepts.
10. Real-world enterprise storage architectures using AWS.
