# Highly Available Three-Tier VPC Architecture

## Project Overview

This project demonstrates the design of a **highly available, secure, and scalable three-tier architecture on AWS** for an e-commerce platform (GlobalBook Inc.). The architecture follows AWS best practices and is suitable for production workloads.

The system is logically divided into:

* **Presentation Tier (Web)**
* **Application Tier**
* **Data Tier**

Each tier is isolated using dedicated subnets, route tables, and security groups, and deployed across multiple Availability Zones (AZs).

<img src="Three-Tier arch.drawio.png" width="1000" />

---

## Architecture Goals

* **High Availability:** Survive infrastructure and AZ failures
* **Scalability:** Scale each tier independently based on load
* **Security:** Enforce network isolation and least-privilege access
* **Operational Simplicity:** Managed access, monitoring, and automation

---

## AWS Services Used

* Amazon VPC
* Public & Private Subnets
* Internet Gateway (IGW)
* NAT Gateway
* Application Load Balancer (ALB)
* Auto Scaling Groups (ASG)
* Amazon EC2
* Amazon RDS (Multi-AZ)
* AWS Systems Manager (SSM)
* VPC Endpoints (Interface & Gateway)
* Amazon Route 53
* Amazon CloudWatch

---

## Network Architecture

### VPC Configuration

* **CIDR Block:** `10.0.0.0/16`
* **Availability Zones:** 2 AZs for fault tolerance

### Subnet Design

| Tier | Subnet Type | AZ   | CIDR        |
| ---- | ----------- | ---- | ----------- |
| Web  | Public      | AZ-1 | 10.0.0.0/24 |
| Web  | Public      | AZ-2 | 10.0.2.0/24 |
| App  | Private     | AZ-1 | 10.0.3.0/24 |
| App  | Private     | AZ-2 | 10.0.4.0/24 |
| Data | Private     | AZ-1 | 10.0.5.0/24 |
| Data | Private     | AZ-2 | 10.0.6.0/24 |

---

## Routing & Internet Access

### Public Route Table

* `10.0.0.0/16 → local`
* `0.0.0.0/0 → Internet Gateway`

Used by:

* Application Load Balancer
* Web tier EC2 instances

### Private Route Table

* `10.0.0.0/16 → local`
* `0.0.0.0/0 → NAT Gateway`

Used by:

* Application tier
* Data tier (for updates only)

---

## Presentation Tier (Web Tier)

**Components:**

* Application Load Balancer (ALB)
* Auto Scaling Group (EC2)
* Public subnets in multiple AZs

**Responsibilities:**

* Handle incoming HTTP/HTTPS traffic
* Serve frontend content
* Forward requests to Application Tier

**High Availability:**

* Multi-AZ ALB
* Auto Scaling replaces unhealthy instances

---

## Application Tier

**Components:**

* Auto Scaling Group (EC2)
* Private subnets

**Responsibilities:**

* Business logic processing
* Communication with database

**Security:**

* No public IPs
* Accepts traffic only from Web Tier

---

## Data Tier

**Components:**

* Amazon RDS (Multi-AZ)
* Private subnets

**Responsibilities:**

* Persistent data storage
* Transactions, users, products

**High Availability:**

* Synchronous Multi-AZ replication
* Automatic failover

---

## Security Design

### Security Groups

* **ALB:** Allows HTTP/HTTPS from internet
* **Web Tier:** Allows traffic from ALB only
* **App Tier:** Allows traffic from Web Tier only
* **Data Tier:** Allows traffic from App Tier only

### Network Isolation

* Only ALB is internet-facing
* App and Data tiers are private
* No SSH access from the internet

---

## Management & Access

### AWS Systems Manager (SSM)

* Secure access to EC2 instances
* No SSH keys required

### VPC Endpoints

* **Interface Endpoint:** SSM
* **Gateway Endpoint:** S3

Benefits:

* Reduced NAT usage
* Improved security

---

## Monitoring & Logging

### Amazon CloudWatch

* EC2 metrics
* Load balancer health checks
* Auto Scaling alarms
* Application logs

---

## Scalability Strategy

| Tier     | Scaling Method                   |
| -------- | -------------------------------- |
| Web      | Auto Scaling Group               |
| App      | Auto Scaling Group               |
| Database | Vertical scaling / Read Replicas |

Each tier scales independently.

---

## High Availability Summary

* Multi-AZ deployment
* Load balancing
* Stateless EC2 instances
* Auto Scaling
* Multi-AZ RDS

Failure in one AZ does not impact service availability.

---

## Cost Optimization

* Auto Scaling minimizes idle capacity
* NAT Gateway shared per AZ
* VPC Endpoints reduce data transfer costs
* Right-sized EC2 and RDS instances

---

## Conclusion

This architecture meets all requirements for a production-grade e-commerce platform by providing:

* High availability
* Strong security isolation
* Elastic scalability
* Operational efficiency

It can be extended with services such as AWS WAF, CloudFront, and database read replicas.
