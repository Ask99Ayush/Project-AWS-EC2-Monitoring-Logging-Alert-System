# AWS EC2 Monitoring, Logging & Alert System

## 🏗 Production-Grade Architecture Diagram

```markdown
![Production Architecture](assets/architecture.png)
```

---

## 📘 Project Introduction

Modern production systems cannot rely on manual checking of servers.
Infrastructure must be monitored continuously, alerts must be automated, and logs must be archived securely.

This project demonstrates how to build a **baseline production monitoring and logging system** for an Amazon EC2 instance using fully managed AWS services.

It shows how real-world DevOps teams design:

* Infrastructure Monitoring
* Automated Alerting
* Centralized Logging
* Long-Term Log Archival

All without installing agents or using hardcoded credentials.

---

# 📑 Table of Contents

1. [Project Overview](#project-overview)
2. [Problem Statement](#problem-statement)
3. [Monitoring vs Logging vs Alerting](#monitoring-vs-logging-vs-alerting)
4. [High-Level Architecture](#high-level-architecture)
5. [Detailed Architecture Explanation](#detailed-architecture-explanation)
6. [AWS Services Used](#aws-services-used)
7. [System Data Flow (Step-by-Step)](#system-data-flow-step-by-step)
8. [EC2 Configuration Details](#ec2-configuration-details)
9. [CloudWatch Alarm Configuration](#cloudwatch-alarm-configuration)
10. [SNS Notification System](#sns-notification-system)
11. [Logging & Archival Design](#logging--archival-design)
12. [Security Best Practices Applied](#security-best-practices-applied)
13. [Testing & Validation Process](#testing--validation-process)
14. [Production Relevance](#production-relevance)
15. [Interview Explanation (2-Minute Version)](#interview-explanation-2-minute-version)
16. [Resume-Ready Description](#resume-ready-description)

---

# Project Overview

This project builds a **production-ready monitoring, alerting, and logging pipeline** for an Amazon EC2 instance.

The system continuously:

* Monitors CPU utilization
* Detects sustained abnormal usage
* Sends automated email alerts
* Archives logs securely in Amazon S3

It uses only AWS-managed services and follows cloud best practices.

---

# Problem Statement

In real production systems:

* Servers rarely fail instantly
* Performance degrades gradually
* High CPU usage may continue unnoticed
* Small issues escalate into outages

Without monitoring:

* Teams discover issues after users complain
* Downtime increases
* Debugging becomes difficult

This project answers:

> How do we detect infrastructure problems before users are impacted?

---

# Monitoring vs Logging vs Alerting

Understanding the difference is critical.

## Monitoring

Monitoring tracks numeric health indicators of infrastructure.

Example:

CPUUtilization = 82%

Monitoring answers:

* Is the system healthy right now?

---

## Logging

Logging stores detailed event records.

Examples:

* Application errors
* System events
* Security activity

Logging answers:

* What exactly happened?
* Why did it happen?

---

## Alerting

Alerting sends notifications when something abnormal happens.

Example:

* Send email when CPU > 70% for 10 minutes

Alerting answers:

* Who needs to act immediately?

---

## How They Work Together

Monitoring detects abnormal behavior.
Alerting triggers response.
Logging supports root cause analysis.

This project starts with monitoring and alerting first, then extends to logging.

---

# High-Level Architecture

## Monitoring & Alerting Flow

Amazon EC2
→ CloudWatch Metrics
→ CloudWatch Alarm
→ SNS Topic
→ Email Notification

---

## Logging & Archival Flow

CloudWatch Logs
→ Subscription Filter
→ Kinesis Data Firehose
→ Amazon S3

---

# Detailed Architecture Explanation

### 1️⃣ EC2 Instance

* Instance Type: t2.micro 
* Represents a production server
* Generates default infrastructure metrics

No agents installed.

---

### 2️⃣ Amazon CloudWatch (Metrics)

CloudWatch automatically collects:

* CPUUtilization
* NetworkIn
* NetworkOut
* DiskReadOps
* DiskWriteOps

No configuration required for default metrics.

---

### 3️⃣ CloudWatch Alarm

Alarm is configured with:

* Threshold: CPU > 70%
* Period: 5 minutes
* Evaluation Periods: 2

Meaning:

CPU must remain above 70% for 10 continuous minutes before triggering.

This avoids false positives from short spikes.

Alarm states:

* OK
* ALARM
* INSUFFICIENT_DATA

---

### 4️⃣ Amazon SNS

When alarm enters ALARM state:

* CloudWatch publishes event to SNS
* SNS sends notification to subscribers

In this project:

* Protocol: Email
* Subscription must be confirmed

SNS decouples monitoring from notification delivery.

---

### 5️⃣ CloudWatch Logs

Used to centralize logs from AWS-managed services.

EC2 does NOT send OS logs by default (no agent used).

---

### 6️⃣ Kinesis Data Firehose

Firehose acts as a managed log delivery pipeline.

It:

* Buffers log data
* Compresses logs
* Retries failed deliveries
* Streams to S3

Fully managed and auto-scaling.

---

### 7️⃣ Amazon S3 (Log Archive)

Used for:

* Long-term storage
* Compliance
* Auditing

Best practices applied:

* Private bucket
* Encryption enabled
* Lifecycle policies configured

---

# System Data Flow (Step-by-Step)

1. EC2 runs workload.
2. AWS hypervisor collects metrics.
3. CloudWatch stores metrics.
4. Alarm evaluates CPU usage.
5. CPU exceeds threshold for 10 minutes.
6. Alarm changes to ALARM state.
7. SNS sends email notification.
8. Logs are captured in CloudWatch Logs.
9. Subscription filter streams logs to Firehose.
10. Firehose delivers compressed logs to S3.

Entire workflow is automated.

---

# EC2 Configuration Details

## Security Group

Inbound:

* SSH (Port 22) restricted to one IP

Outbound:

* Allow all (needed for AWS APIs)

---

## IAM Role

Attached IAM Role:

* No access keys stored
* Least privilege permissions
* Secure service-to-service communication

---

## Region Consistency

All services are deployed in the same AWS region to ensure compatibility.

---

# CloudWatch Alarm Configuration

| Parameter          | Value          |
| ------------------ | -------------- |
| Metric             | CPUUtilization |
| Threshold          | > 70%          |
| Period             | 5 Minutes      |
| Evaluation Periods | 2              |
| Total Duration     | 10 Minutes     |

Why 70%?

* Below 70% is usually safe for burstable instances
* Sustained 70%+ indicates potential overload

---

# SNS Notification System

Flow:

CloudWatch Alarm → SNS Topic → Email

Important:

Email subscription must be confirmed before receiving alerts.

---

# Logging & Archival Design

Why not direct CloudWatch → S3?

Because CloudWatch Logs cannot directly stream to S3.

Firehose is required as a managed delivery pipeline.

Benefits:

* Near real-time streaming
* Built-in compression
* Automatic scaling
* No infrastructure management

---

# Security Best Practices Applied

* IAM roles instead of access keys
* Least privilege policies
* S3 encryption enabled
* Private bucket access
* Log lifecycle rules
* No hardcoded credentials
* Single-region consistency

---

# Testing & Validation Process

## CPU Alarm Testing

* Generate artificial CPU load
* Verify metric increase
* Confirm alarm transitions

## Email Notification Testing

* Confirm subscription
* Trigger alarm
* Verify email receipt

## Log Delivery Testing

* Check S3 bucket
* Validate compressed files
* Verify timestamps

---

# Production Relevance

This architecture represents:

* Baseline observability in real systems
* Agentless monitoring
* Scalable managed services
* Cost-efficient log storage

It can be extended to:

* Multi-AZ deployments
* Auto Scaling groups
* Application-level logging
* Multi-account monitoring

---

# Interview Explanation (2-Minute Version)

This project implements a production-grade EC2 monitoring and logging system using AWS-managed services. CloudWatch automatically collects EC2 metrics and evaluates CPU thresholds using alarms. When sustained abnormal usage is detected, SNS sends automated email notifications. Logs from AWS-managed services are centralized in CloudWatch Logs and streamed to S3 via Kinesis Data Firehose for secure long-term archival. The system avoids agents, uses IAM role-based access, and follows least-privilege security practices.

---

# Resume-Ready Description

Designed and implemented a production-oriented AWS EC2 monitoring, alerting, and logging architecture using CloudWatch, SNS, S3, and Kinesis Data Firehose. Built an automated observability pipeline with CPU-based alarms, real-time email alerts, centralized logging, and secure long-term archival while applying IAM least-privilege principles and cloud security best practices.

---

