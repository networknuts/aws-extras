# AWS FSx for Windows File Server POC Lab

## Objective

Build a shared Windows file server using AWS FSx for Windows and demonstrate:

1. Creating a managed Windows file share
2. Accessing files through SMB
3. Integrating with Active Directory
4. Setting NTFS permissions
5. Understanding enterprise file-sharing architectures

---

# What is AWS FSx for Windows File Server?

Amazon FSx for Windows File Server is a fully managed Windows-native file storage service that provides shared file storage using the SMB protocol.

It behaves like a traditional Windows File Server but AWS manages:

* Hardware
* Storage
* Patching
* Backups
* High Availability

---

# Why Use FSx for Windows?

Many enterprise applications require:

```text
SMB File Shares
Windows ACLs
Active Directory Authentication
Shared Network Drives
```

Examples:

* User Home Directories
* Department Shared Drives
* Application File Repositories
* Microsoft Workloads
* Profile Storage

---

# Traditional File Server Problem

Without FSx:

```text
Windows Server
      |
      +-- Storage
      +-- Patching
      +-- Backup
      +-- Monitoring
      +-- HA Setup
```

Administrators manage everything.

---

With FSx:

```text
Applications
      |
      v
Amazon FSx
```

AWS manages infrastructure.

---

# How FSx for Windows Works

```text
Windows Users
       |
       v
Active Directory
       |
       v
Amazon FSx
       |
       v
Shared File Storage
```

Users authenticate through Active Directory and access SMB shares.

---

# Where is FSx for Windows Used?

## Corporate Shared Drives

```text
HR Drive
Finance Drive
Engineering Drive
Legal Drive
```

---

## Application Storage

Applications store:

```text
Documents
Images
Reports
Exports
Logs
```

---

## User Home Directories

```text
\\fileserver\users
```

Each employee receives a personal folder.

---

# POC Architecture

```text
Windows Client
       |
       v
Active Directory
       |
       v
Amazon FSx
```

Services Used:

* VPC
* AWS Managed Microsoft AD
* FSx for Windows
* EC2 Windows Server

---

# POC Scenario

Network Nuts has instructors working remotely.

Requirements:

```text
Shared Course Content
Shared Labs
Shared Student Data
Central Authentication
```

Solution:

```text
FSx + Active Directory
```

---

# Step 1: Create AWS Managed Microsoft AD

Navigate:

```text
Directory Service
→ Create Directory
```

Choose:

```text
AWS Managed Microsoft AD
```

Example:

```text
Domain:
networknuts.local
```

Size:

```text
Standard Edition
```

Deploy in two subnets.

---

# Step 2: Create FSx File System

Navigate:

```text
FSx
→ Create File System
```

Choose:

```text
Windows File Server
```

Configuration:

```text
Storage Capacity:
32 GB

Deployment Type:
Single-AZ
```

For production:

```text
Multi-AZ
```

is preferred.

---

# Step 3: Join Active Directory

Select:

```text
networknuts.local
```

during FSx creation.

AWS automatically joins the file system to the domain.

---

# Step 4: Create Windows EC2

Launch:

```text
Windows Server 2022
```

Join it to:

```text
networknuts.local
```

domain.

---

# Step 5: Connect to File Share

Example FSx DNS:

```text
\\fs-123456.networknuts.local\share
```

Map network drive:

```powershell
net use Z: \\fs-123456.networknuts.local\share
```

or through Windows Explorer.

---

# Step 6: Create Shared Folders

Example folders:

```text
Courses
Assignments
Videos
Students
```

Create:

```powershell
mkdir Z:\Courses
mkdir Z:\Assignments
```

---

# Step 7: Upload Files

Copy:

```text
RHCSA Notes.pdf
AWS Slides.pptx
DevOps Videos.mp4
```

Verify all domain users can access them.

---

# Step 8: Configure NTFS Permissions

Example:

```text
HR Group
     |
     +-- Read/Write HR Folder

Students Group
     |
     +-- Read Only Course Folder

Instructors Group
     |
     +-- Full Control
```

Configure via:

```text
Folder Properties
→ Security
```

---

# Step 9: Demonstrate Shared Access

User A uploads:

```text
kubernetes.pdf
```

User B immediately sees:

```text
kubernetes.pdf
```

This demonstrates centralized storage.

---

# Step 10: Backup and Restore

FSx automatically creates backups.

Open:

```text
FSx
→ Backups
```

Restore file system from backup.

Demonstrate recovery.

---

# High Availability Architecture

Production deployment:

```text
Users
   |
   v
FSx Multi-AZ
   |
   +-------- AZ-1
   |
   +-------- AZ-2
```

If one AZ fails:

```text
Automatic Failover
```

Minimal disruption.

---

# Real-World Architecture

```text
Corporate Users
       |
       v
AWS Managed AD
       |
       v
Amazon FSx
       |
       v
Shared Storage
```

Used by:

* Enterprises
* Banks
* Insurance Companies
* Government Organizations

---

# Comparison: EBS vs EFS vs FSx Windows

| Feature          | EBS | EFS     | FSx Windows |
| ---------------- | --- | ------- | ----------- |
| Shared Storage   | No  | Yes     | Yes         |
| SMB Support      | No  | No      | Yes         |
| Linux Native     | Yes | Yes     | Limited     |
| Windows Native   | No  | Limited | Yes         |
| Active Directory | No  | Limited | Yes         |
| NTFS Permissions | No  | No      | Yes         |

---

# Advanced POC 1: Home Directories

Create:

```text
\\fsx\users\john
\\fsx\users\alice
\\fsx\users\bob
```

Users only access their folders.

Common enterprise setup.

---

# Advanced POC 2: Application Storage

Deploy application:

```text
EC2 Web Server
      |
      v
FSx Share
```

Store:

```text
Uploaded Documents
Reports
Generated Files
```

on FSx.

---

# Advanced POC 3: DFS Namespace

Integrate with:

Microsoft Distributed File System

Example:

```text
\\networknuts.local\courses
```

instead of direct FSx hostname.

Useful for large enterprises.

---

# Common Enterprise Use Cases

## HR Department

```text
Employee Records
Policies
Payroll Documents
```

---

## Training Institute

```text
Course Videos
Assignments
Lab Manuals
Student Records
```

---

## Application File Storage

```text
Shared Uploads
Reports
Exports
```

---

# Interview Questions

### What is FSx for Windows?

**Answer:**
A fully managed Windows-native file storage service that supports SMB, Active Directory integration, and NTFS permissions.

---

### Which protocol does FSx for Windows use?

**Answer:**

```text
SMB (Server Message Block)
```

---

### Can FSx integrate with Active Directory?

**Answer:**
Yes. It supports both:

* AWS Managed Microsoft AD
* Self-managed Active Directory

---

### What are the benefits of Multi-AZ deployment?

**Answer:**

* High Availability
* Automatic Failover
* Increased Resilience

---

### Why choose FSx instead of EFS?

**Answer:**
FSx for Windows provides:

* SMB support
* Active Directory integration
* Windows ACLs
* NTFS permissions

which EFS does not provide natively.

---

# Learning Outcomes

After completing this lab, students should understand:

1. What Amazon FSx for Windows File Server is.
2. SMB-based file sharing concepts.
3. Active Directory integration.
4. NTFS permissions and access control.
5. Shared enterprise storage architectures.
6. High availability with Multi-AZ deployments.
7. Managed backups and recovery.
8. Home directory implementations.
9. Application file-sharing use cases.
10. Real-world enterprise Windows storage architectures on AWS.
