# AWS Simple Workflow Service (SWF) POC Lab

## Objective

Build a workflow orchestration system using AWS SWF and demonstrate:

1. Coordinating long-running business processes
2. Managing workflow state
3. Human and automated task execution
4. Workflow retries and fault tolerance
5. Understanding orchestration vs messaging

---

# What is AWS Simple Workflow Service?

Amazon Simple Workflow Service (SWF) is a workflow orchestration service that helps coordinate tasks across distributed applications and human interactions.

Think of SWF as:

```text
Workflow Manager
```

that tracks:

* What task should run next
* Which tasks completed
* Which tasks failed
* Which tasks need retrying
* Workflow progress

---

# Why Do We Need SWF?

Imagine a student purchases a course.

The process involves:

```text
Payment
   |
Create User
   |
Send Welcome Email
   |
Generate LMS Access
   |
Issue Invoice
```

Without orchestration:

```text
Application Code
    |
    +-- Many if/else conditions
    +-- Retry logic
    +-- State management
```

Complex and difficult to maintain.

---

With SWF:

```text
Workflow
     |
     +-- Payment
     |
     +-- Create User
     |
     +-- Send Email
     |
     +-- LMS Access
```

SWF tracks the workflow automatically.

---

# Real-World Example

Network Nuts enrollment process:

```text
Student Registers
        |
        v
Payment Success
        |
        v
Create LMS Account
        |
        v
Assign Batch
        |
        v
Send Zoom Link
        |
        v
Send Invoice
```

Each step can be managed as part of a workflow.

---

# How SWF Works

Core components:

## Workflow

Entire business process.

Example:

```text
Course Enrollment Workflow
```

---

## Activity

Individual task.

Examples:

```text
Create User
Send Email
Generate Invoice
```

---

## Worker

Executes activities.

Example:

```text
Python Worker
```

running on EC2.

---

## Decider

Determines what should happen next.

Example:

```text
If Payment Success
      |
      v
Create User
```

---

# SWF Architecture

```text
Client
  |
  v
Workflow Execution
  |
  v
SWF
  |
  +---- Decider
  |
  +---- Worker
```

---

# SWF vs SQS

Many students confuse these.

| Feature        | SQS       | SWF       |
| -------------- | --------- | --------- |
| Messaging      | Yes       | No        |
| Workflow State | No        | Yes       |
| Task Ordering  | Basic     | Advanced  |
| Long Processes | Difficult | Excellent |
| Human Tasks    | No        | Yes       |

---

# POC Architecture

```text
Student Registration
        |
        v
SWF Workflow
        |
        +---- Create User
        |
        +---- Send Email
        |
        +---- Generate Invoice
```

Services Used:

* SWF
* EC2
* IAM
* Python

---

# POC Scenario

Simulate:

```text
Course Enrollment Workflow
```

Steps:

1. Student pays
2. User account created
3. Welcome email sent
4. Course assigned

---

# Step 1: Create Domain

Navigate:

```text
SWF
→ Domains
→ Create Domain
```

Name:

```text
networknuts-training
```

Retention:

```text
7 Days
```

---

# Step 2: Define Workflow

Workflow:

```text
CourseEnrollmentWorkflow
```

Version:

```text
1.0
```

Timeout:

```text
3600 seconds
```

---

# Step 3: Define Activities

Activities:

```text
CreateUser
SendWelcomeEmail
AssignCourse
```

---

# Step 4: Create Worker

Install:

```bash
pip install boto3
```

Worker:

```python
import boto3

swf = boto3.client("swf")

while True:

    task = swf.poll_for_activity_task(
        domain="networknuts-training",
        taskList={"name":"training-workers"}
    )

    if "taskToken" in task:

        print("Processing Activity")

        swf.respond_activity_task_completed(
            taskToken=task["taskToken"],
            result="success"
        )
```

---

# Step 5: Create Decider

Decider determines workflow progression.

Example:

```python
if activity_completed:
    schedule_next_activity()
```

Logic:

```text
Create User
     |
     v
Send Email
     |
     v
Assign Course
```

---

# Step 6: Start Workflow Execution

Start:

```python
swf.start_workflow_execution(
    domain="networknuts-training",
    workflowId="student-1001",
    workflowType={
        "name":"CourseEnrollmentWorkflow",
        "version":"1.0"
    },
    taskList={"name":"decider"}
)
```

---

# Step 7: Observe Workflow Execution

Execution:

```text
student-1001

Step 1:
Create User

Step 2:
Send Email

Step 3:
Assign Course
```

Each step tracked independently.

---

# Step 8: Simulate Failure

Example:

```text
Send Email Failed
```

SWF records:

```text
Activity Failed
```

Workflow remains active.

---

# Step 9: Retry Activity

SWF supports retries.

Example:

```text
Attempt 1 → Fail

Attempt 2 → Success
```

Workflow continues.

---

# Step 10: Complete Workflow

Final state:

```text
COMPLETED
```

Student successfully enrolled.

---

# Human Workflow Example

One unique SWF feature:

```text
Human Approval
```

Example:

```text
Course Refund Request
        |
        v
Manager Approval
        |
        v
Refund Process
```

Workflow waits until approval occurs.

---

# Real-World Architecture

```text
Application
      |
      v
SWF
      |
      +---- Worker 1
      |
      +---- Worker 2
      |
      +---- Human Approval
      |
      +---- Final Processing
```

---

# Banking Example

Loan approval:

```text
Application Submitted
        |
        v
Credit Check
        |
        v
Risk Analysis
        |
        v
Manager Approval
        |
        v
Loan Issued
```

Perfect workflow use case.

---

# Insurance Example

```text
Claim Submitted
      |
      v
Document Validation
      |
      v
Fraud Check
      |
      v
Human Review
      |
      v
Approval
```

---

# Training Institute Example

```text
Student Registration
       |
       v
Payment Verification
       |
       v
Create LMS User
       |
       v
Assign Batch
       |
       v
Send Zoom Link
```

---

# SWF vs Step Functions

This is a common interview topic.

| Feature            | SWF     | Step Functions |
| ------------------ | ------- | -------------- |
| Visual Workflow    | No      | Yes            |
| Serverless         | Partial | Yes            |
| Modern Service     | Older   | Newer          |
| Coding Required    | More    | Less           |
| AWS Recommendation | Legacy  | Preferred      |

---

# Modern Equivalent

Today AWS recommends:

AWS Step Functions

for most workflow orchestration use cases.

SWF still exists but is less commonly chosen for new projects.

---

# Advanced POC 1: AI Resume Evaluation

Relevant to your lead-generation platform.

```text
Resume Uploaded
      |
      v
SWF Workflow
      |
      +---- Extract Text
      |
      +---- Generate Questions
      |
      +---- Evaluate Answers
      |
      +---- Generate Report
```

---

# Advanced POC 2: Course Purchase Workflow

```text
Payment
   |
   v
SWF
   |
   +---- Create User
   +---- Generate Invoice
   +---- Send Email
```

---

# Advanced POC 3: Image Processing

```text
Image Upload
      |
      v
SWF
      |
      +---- Resize
      |
      +---- Watermark
      |
      +---- Store
```

---

# Interview Questions

### What is SWF?

**Answer:**
Amazon SWF is a workflow orchestration service that coordinates distributed tasks and manages workflow state.

---

### What are the main SWF components?

**Answer:**

1. Workflow
2. Activity
3. Worker
4. Decider

---

### What is a Decider?

**Answer:**
The decider determines the next step in a workflow based on the workflow history and activity outcomes.

---

### What is an Activity?

**Answer:**
An activity is a unit of work performed by a worker, such as sending an email or creating a user account.

---

### Can SWF support human approvals?

**Answer:**
Yes. Workflows can pause and wait for human actions before continuing.

---

### What service is generally preferred over SWF today?

**Answer:**
AWS Step Functions is the modern workflow orchestration service recommended for most new applications.

---

# Learning Outcomes

After completing this lab, students should understand:

1. What workflow orchestration is.
2. How SWF manages workflow state.
3. Activities, workers, and deciders.
4. Retry and fault-tolerance concepts.
5. Long-running workflow management.
6. Human approval workflows.
7. Business process automation.
8. SWF vs SQS differences.
9. SWF vs Step Functions comparison.
10. Real-world enterprise workflow orchestration patterns.
