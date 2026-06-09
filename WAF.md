# AWS WAF POC Lab

## Objective

Deploy a web application behind AWS WAF and demonstrate:

1. Allowing normal traffic
2. Blocking malicious IP addresses
3. Blocking SQL Injection attacks
4. Blocking XSS attacks
5. Viewing WAF logs and metrics

---

# What is AWS WAF?

AWS WAF (Web Application Firewall) protects web applications from common web exploits and malicious traffic.

Instead of protecting servers directly like a network firewall, WAF protects HTTP/HTTPS requests reaching applications.

Common threats:

* SQL Injection (SQLi)
* Cross Site Scripting (XSS)
* Bot Traffic
* Vulnerability Scanners
* DDoS Layer 7 Attacks
* Bad IP Addresses
* Geographic Restrictions

AWS WAF sits in front of:

* Application Load Balancer (ALB)
* CloudFront
* API Gateway
* App Runner
* Cognito

---

# How AWS WAF Works

```text
User
 |
 v
AWS WAF
 |
 |---- Check Rules
 |
 |---- Allow
 |---- Block
 |---- Count
 |
 v
Application Load Balancer
 |
 v
EC2 Application
```

Every HTTP request is inspected before reaching the application.

---

# POC Architecture

```text
Internet
    |
    v
+-------------------+
|      AWS WAF      |
+-------------------+
          |
          v
+-------------------+
|       ALB         |
+-------------------+
          |
          v
+-------------------+
| EC2 Web Server    |
| NGINX/Apache      |
+-------------------+
```

Services Used:

* VPC
* EC2
* Security Group
* Application Load Balancer
* AWS WAF
* CloudWatch

---

# Step 1: Create Web Server

Launch:

### EC2

* Amazon Linux 2023
* t2.micro
* Allow HTTP (80)

Install nginx:

```bash
sudo dnf install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

Create custom page:

```bash
echo "AWS WAF Demo" | sudo tee /usr/share/nginx/html/index.html
```

---

# Step 2: Create Application Load Balancer

Create:

### ALB

Listener:

```text
HTTP : 80
```

Target Group:

```text
EC2 Instance
```

Verify:

```text
http://ALB-DNS-NAME
```

returns:

```text
AWS WAF Demo
```

---

# Step 3: Create AWS WAF

Navigate:

```text
AWS WAF
→ Web ACL
→ Create Web ACL
```

Resource:

```text
Application Load Balancer
```

Associate:

```text
Your ALB
```

---

# Step 4: Add Managed Rules

AWS provides ready-made rules.

Add:

## Core Rule Set

Protects against:

* SQL Injection
* XSS
* Command Injection
* Path Traversal

Rule Group:

```text
AWS Managed Rules
→ Core Rule Set
```

---

# Step 5: Test SQL Injection

Normal request:

```text
http://alb-dns/
```

Works.

Now try:

```text
http://alb-dns/?id=' OR 1=1 --
```

WAF should block.

Response:

```text
403 Forbidden
```

---

# Step 6: Test XSS

Try:

```text
http://alb-dns/?search=<script>alert(1)</script>
```

Expected:

```text
403 Forbidden
```

---

# Step 7: Create IP Block Rule

Create custom rule.

Block:

```text
203.0.113.10
```

Example rule:

```text
IF Source IP = 203.0.113.10
THEN Block
```

Any request from that IP receives:

```text
403 Forbidden
```

---

# Step 8: Geo Blocking

Example:

```text
Block Country = China
```

Traffic from China:

```text
Blocked
```

Traffic from India:

```text
Allowed
```

Useful for:

* Region-restricted websites
* Compliance requirements

---

# Step 9: Rate Limiting

Protect against bots.

Rule:

```text
More than 100 requests
per 5 minutes
from same IP
```

Action:

```text
Block
```

Example:

```text
ab -n 500 -c 50 http://alb-dns/
```

After threshold:

```text
403 Forbidden
```

---

# Step 10: CloudWatch Monitoring

Open:

```text
CloudWatch
```

Metrics:

* Allowed Requests
* Blocked Requests
* Counted Requests

Example dashboard:

```text
Allowed Requests : 5000

Blocked Requests : 250

SQLi Attempts : 50

XSS Attempts : 20
```

---

# Real World Use Cases

## Banking

Protect:

* Net Banking
* Credit Card Portals
* Payment Gateways

Example:

HDFC Bank,
ICICI Bank

use WAF in front of internet-facing applications.

---

## E-Commerce

Protect:

* Login Pages
* Checkout APIs
* Payment Pages

Example:

Amazon,
Flipkart

---

## APIs

Protect:

* REST APIs
* Mobile Backend APIs

AWS WAF can be attached directly to:

Amazon API Gateway

---

## SaaS Applications

Protect:

* Customer portals
* Admin dashboards
* Authentication services

---

# Expected Learning Outcomes

After this lab, students should understand:

1. What a Web Application Firewall is.
2. Difference between Security Groups and WAF.
3. How AWS WAF evaluates requests.
4. Managed vs Custom Rules.
5. SQL Injection protection.
6. XSS protection.
7. Rate Limiting.
8. IP and Geo Blocking.
9. CloudWatch monitoring integration.
10. Real-world AWS security architecture.

# Interview Question

**Why use AWS WAF when Security Groups already exist?**

**Answer:**

Security Groups operate at Layer 3/4 (IP, Port, Protocol).

AWS WAF operates at Layer 7 (HTTP/HTTPS) and can inspect URLs, headers, query strings, cookies, and request payloads to block attacks like SQL Injection and XSS that Security Groups cannot detect.
