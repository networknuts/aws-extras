# AWS Elastic Network Interface (ENI) POC Lab

## Objective

Build a lab demonstrating AWS Elastic Network Interfaces (ENIs) and learn:

1. What an ENI is
2. How multiple network interfaces work on an EC2 instance
3. Moving IP addresses between instances
4. High availability and failover scenarios
5. Real-world networking use cases

---

# What is an Elastic Network Interface (ENI)?

An ENI is a virtual network card that can be attached to an EC2 instance.

Think of it as:

```text
Physical Server
     |
     +---- Network Card 1
     +---- Network Card 2
```

In AWS:

```text
EC2 Instance
     |
     +---- ENI-1
     +---- ENI-2
```

Each ENI can have:

* Private IP addresses
* Public IP addresses
* Security Groups
* MAC Address
* Elastic IPs

---

# Why Do We Need ENIs?

Normally:

```text
EC2 Instance
     |
     v
Private IP
```

If the server fails:

```text
Private IP Lost
```

Applications may become unreachable.

---

Using ENI:

```text
ENI
 |
 +---- Private IP
 +---- Security Groups
```

Move ENI to another server:

```text
EC2-A (Failed)
      |
      X

Detach ENI

Attach ENI

EC2-B
```

Application becomes available again.

---

# How ENI Works

```text
Application
      |
      v
Private IP
      |
      v
ENI
      |
      v
EC2 Instance
```

The network identity belongs to the ENI, not the server.

---

# Real-World Use Cases

## High Availability

```text
Primary Server
      |
      v
ENI
```

Failover:

```text
Primary Down
      |
      v
Move ENI
      |
      v
Standby Server
```

---

## Firewall Appliances

Examples:

* Fortinet
* Palo Alto
* Check Point

Often use:

```text
ENI 1 → Public Network

ENI 2 → Internal Network
```

---

## Multi-Homed Servers

```text
Web Traffic
     |
     v
ENI-1

Management Traffic
     |
     v
ENI-2
```

---

# POC Architecture

```text
          VPC
           |
    -----------------
    |               |
    v               v

EC2-A          EC2-B
   |
   v
 Secondary ENI
```

Services Used:

* VPC
* EC2
* ENI
* Security Groups

---

# POC Scenario

Create:

```text
Primary Server
Standby Server
```

Move an ENI between them.

Demonstrate:

```text
Application IP remains same
```

even though server changes.

---

# Step 1: Launch EC2 Instances

Launch:

```text
EC2-A
EC2-B
```

Example:

```text
Amazon Linux 2023
t2.micro
```

Same subnet.

---

# Step 2: Create Secondary ENI

Navigate:

```text
EC2
→ Network Interfaces
→ Create Network Interface
```

Configuration:

```text
Subnet:
Same subnet as EC2

Private IP:
10.0.1.100
```

Security Group:

```text
Allow SSH
Allow HTTP
```

---

# Step 3: Attach ENI

Attach to:

```text
EC2-A
```

Verify:

```bash
ip addr
```

Output:

```text
eth0
10.0.1.25

eth1
10.0.1.100
```

---

# Step 4: Create Web Server

Install nginx:

```bash
sudo dnf install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

Create page:

```bash
echo "Primary Server" | sudo tee /usr/share/nginx/html/index.html
```

---

# Step 5: Access Using ENI IP

Open:

```text
http://10.0.1.100
```

Response:

```text
Primary Server
```

Notice:

```text
Traffic uses ENI IP
```

---

# Step 6: Simulate Failure

Stop nginx:

```bash
sudo systemctl stop nginx
```

or stop instance.

---

# Step 7: Detach ENI

Navigate:

```text
EC2
→ Network Interfaces
```

Detach ENI from:

```text
EC2-A
```

---

# Step 8: Attach ENI to EC2-B

Attach same ENI:

```text
10.0.1.100
```

to:

```text
EC2-B
```

---

# Step 9: Start Web Server on EC2-B

Install nginx:

```bash
echo "Standby Server" | sudo tee /usr/share/nginx/html/index.html
```

Start:

```bash
sudo systemctl start nginx
```

---

# Step 10: Verify Failover

Open:

```text
http://10.0.1.100
```

Response:

```text
Standby Server
```

Important:

```text
IP Address Never Changed
```

Only server changed.

---

# Understanding ENI Components

Each ENI contains:

```text
Private IP
Security Groups
MAC Address
Elastic IP Mapping
```

These move together.

---

# Primary vs Secondary ENI

| Feature           | Primary ENI        | Secondary ENI            |
| ----------------- | ------------------ | ------------------------ |
| Can Detach        | No                 | Yes                      |
| Created at Launch | Yes                | No                       |
| Move Between EC2  | No                 | Yes                      |
| Common Usage      | Default Networking | HA & Advanced Networking |

---

# Security Groups per ENI

Example:

```text
ENI-1
   |
   +--- SSH Only

ENI-2
   |
   +--- HTTP Only
```

Different interfaces can have different security policies.

---

# Multiple ENI Example

```text
EC2 Instance
      |
      +---- ENI-1
      |       |
      |       +--- Public Traffic
      |
      +---- ENI-2
              |
              +--- Database Traffic
```

Common in enterprise environments.

---

# Firewall Appliance Example

```text
Internet
    |
    v
ENI-1
    |
Firewall
    |
ENI-2
    |
Internal Network
```

Used by:

* Fortinet
* Palo Alto
* Check Point

virtual appliances on AWS.

---

# Kubernetes Example

AWS CNI uses ENIs extensively.

```text
Worker Node
      |
      +---- ENI
      |
      +---- Pod IPs
```

Each ENI provides IP addresses for pods.

This is why Kubernetes on AWS can assign VPC IPs directly to pods.

---

# Advanced POC 1: Elastic IP Failover

Attach:

```text
Elastic IP
     |
     v
ENI
```

Move ENI:

```text
EC2-A → EC2-B
```

Public IP moves automatically.

---

# Advanced POC 2: Multi-Network Server

```text
Application
      |
      +---- ENI-1 (Users)

      +---- ENI-2 (Management)

      +---- ENI-3 (Database)
```

Traffic separation.

---

# Advanced POC 3: Database Failover

```text
Primary DB
      |
      v
ENI
```

Move ENI:

```text
Primary
   |
 Fail
   |
 Standby
```

Application continues using same IP.

---

# Interview Questions

### What is an ENI?

**Answer:**
An Elastic Network Interface is a virtual network card that can be attached to an EC2 instance and contains IP addresses, security groups, and networking configuration.

---

### Can an ENI be moved between instances?

**Answer:**
Yes. Secondary ENIs can be detached from one instance and attached to another within the same Availability Zone.

---

### What is the benefit of ENIs?

**Answer:**
They allow network identity (IP address and security settings) to move independently of compute resources.

---

### What is the difference between Primary and Secondary ENI?

**Answer:**

Primary ENI:

* Created automatically
* Cannot be detached

Secondary ENI:

* Created manually
* Can be detached and moved

---

### How does AWS Kubernetes use ENIs?

**Answer:**
The AWS VPC CNI plugin allocates pod IP addresses from ENIs attached to worker nodes.

---

### Can Security Groups be attached to ENIs?

**Answer:**
Yes. Security groups are associated directly with ENIs and move with the interface.

---

# Learning Outcomes

After completing this lab, students should understand:

1. What an Elastic Network Interface is.
2. How AWS networking is abstracted through ENIs.
3. Primary vs Secondary ENIs.
4. Failover using ENIs.
5. Security groups on ENIs.
6. Multi-homed server architectures.
7. Firewall appliance networking.
8. Elastic IP association with ENIs.
9. Kubernetes AWS CNI networking concepts.
10. Real-world AWS networking and high-availability use cases.
