# Project Overview

## Introduction

Modern organizations generate millions of transactions every day. Analytical workloads should never run directly on transactional databases because they affect application performance and increase response time.

To solve this problem, organizations replicate operational data into analytical databases where reporting, dashboards, machine learning and business intelligence workloads can be executed independently.

This project demonstrates a complete production-style Change Data Capture (CDC) pipeline that continuously replicates data from PostgreSQL into ClickHouse using PeerDB.

The entire infrastructure was deployed on AWS EC2 instances and validated using a realistic e-commerce dataset.

---

# Business Problem

Suppose an e-commerce company stores all customer transactions inside PostgreSQL.

Typical business questions include:

- How many orders were placed today?
- Which products generate the highest revenue?
- Which city has the maximum customers?
- Which supplier has the most sales?
- Which payment method is most popular?
- What is today's return percentage?

Running analytical queries directly on PostgreSQL can significantly affect application performance.

To avoid this, analytical databases such as ClickHouse are used.

The challenge is continuously keeping ClickHouse synchronized with PostgreSQL.

This project solves that problem using PeerDB.

---

# Solution

The architecture continuously captures changes from PostgreSQL using Logical Replication.

PeerDB performs two major tasks.

1. Initial Snapshot

All existing historical data is copied from PostgreSQL into ClickHouse.

2. Continuous CDC

Every INSERT, UPDATE and DELETE operation is automatically replicated into ClickHouse.

No manual intervention is required.

---

# Project Architecture

```
                PostgreSQL
                     │
       Logical Replication (WAL)
                     │
                     ▼
                 PeerDB
          Snapshot + CDC Engine
                     │
                     ▼
                 MinIO Storage
                     │
                     ▼
               ClickHouse
```

---

# Components

## PostgreSQL

Acts as the source transactional database.

Stores

- Customers
- Orders
- Products
- Payments
- Inventory
- Shipments
- Returns

---

## PeerDB

PeerDB performs

- Initial Snapshot
- WAL Reading
- Schema Detection
- Automatic Table Creation
- CDC Replication

---

## MinIO

PeerDB temporarily stores snapshot files inside MinIO before ClickHouse imports them.

---

## ClickHouse

Acts as the analytical database.

Supports

- Real-time Analytics
- Fast Aggregations
- Large Scale Reporting
- OLAP Queries

---

# Dataset

The following e-commerce dataset was generated using Python.

| Table | Rows |
|--------|------:|
| Categories | 101 |
| Suppliers | 500 |
| Customers | 10,000 |
| Products | 5,000 |
| Inventory | 5,000 |
| Orders | 100,000 |
| Order Items | 300,270 |
| Payments | 100,000 |
| Shipments | 100,000 |
| Returns | 10,000 |

---

# Workflow

Step 1

Generate historical data.

↓

Step 2

Insert historical data into PostgreSQL.

↓

Step 3

Enable Logical Replication.

↓

Step 4

Create PeerDB Mirror.

↓

Step 5

PeerDB performs Initial Snapshot.

↓

Step 6

Historical data is copied into ClickHouse.

↓

Step 7

PeerDB continuously streams WAL changes.

↓

Step 8

ClickHouse always remains synchronized.

---

# Features

- Production Style Architecture
- Initial Snapshot
- Continuous CDC
- Automatic Replication
- Historical Synchronization
- Low Latency
- Schema Detection
- Automatic Table Creation
- Soft Delete Support
- AWS Deployment

---

# Validation

The following scenarios were tested successfully.

✅ Historical Snapshot

✅ Live INSERT

✅ Live UPDATE

✅ Live DELETE

✅ Continuous Replication

---

# Outcome

At the end of the implementation, PostgreSQL and ClickHouse remained synchronized through PeerDB.

Every INSERT, UPDATE and DELETE operation performed on PostgreSQL was automatically replicated into ClickHouse without manual intervention.

This demonstrates a production-ready real-time data replication pipeline suitable for analytical workloads.
