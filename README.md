# AWS 3-Tier Multi-Region Disaster Recovery (Terraform)

This project demonstrates a highly available, resilient 3-tier architecture across multiple AWS regions. It is designed to withstand a total regional outage (such as those caused by geopolitical events or natural disasters) by utilizing Infrastructure as Code (IaC) for rapid, automated failover.

## 🌍 Architecture Overview

![Multi-Region DR Architecture](https://github.com/Umaramakrishna/Disaster_Recovery_3tier_Terraform-/raw/main/architecture_diagram.png)

The infrastructure consists of a **Primary Region (us-east-1)** and a **Disaster Recovery (DR) Region (us-west-2)**.

* **Web Tier:** Public-facing ALBs and Auto Scaling Groups.
* **App Tier:** Internal ALBs and private EC2 instances.
* **Database Tier:** Amazon RDS with Cross-Region Read Replicas.

---

## 🔄 Workflow Overview: Automated Disaster Recovery

This project implements a **Warm Standby** DR strategy. The workflow is divided into three critical phases:

### 1. The Baseline (Steady State)
* **Infrastructure Sync:** Identical VPCs, Subnets, and Security Groups are maintained in both regions via Terraform modules.
* **Data Replication:** RDS Cross-Region Read Replicas and S3 Cross-Region Replication (CRR) ensure data is always present in the DR site.
* **Monitoring:** Route53 health checks constantly monitor the health of the Primary ALB.

### 2. Detection (The Disaster Event)
* **Trigger:** If the Primary Region becomes unreachable due to a disaster, the Route53 health check fails.
* **Threshold:** After a defined timeout (e.g., 30 seconds), the system identifies the region as "Down."

### 3. Recovery (Failover)
* **DNS Switch:** Route53 automatically updates DNS records to point to the DR Region's Load Balancer.
* **Database Promotion:** The RDS Read Replica in the DR region is promoted to a standalone Master to allow application writes.
* **Scaling:** The Auto Scaling Group in the DR region scales up to handle the redirected traffic, achieving an **RTO (Recovery Time Objective)** of just a few minutes.

---

## 📂 Project Structure

```text
├── modules/                # Reusable IaC components
│   ├── vpc/                # Network configuration
│   ├── ec2/                # Compute & Scaling
│   └── rds/                # Database & Replication
├── us-east-1/              # Primary Region configuration
├── us-west-2/              # DR Region configuration
├── main.tf                 # Global orchestrator
└── terraform.tfstate       # State management

