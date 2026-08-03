# PostgreSQL to ClickHouse CDC using PeerDB on AWS

![Architecture](architecture/architecture.png)

## Project Overview

This project demonstrates a production-style Change Data Capture (CDC) pipeline that continuously replicates data from PostgreSQL to ClickHouse using PeerDB.

The pipeline performs:

- Initial Snapshot
- Continuous CDC
- Automatic Schema Mapping
- Low Latency Replication
- Historical Data Synchronization
- Live Inserts
- Live Updates
- Live Deletes

The complete infrastructure is deployed on AWS EC2 instances.

---

# Architecture

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
                 MinIO (Snapshot)
                     │
                     ▼
               ClickHouse
             Analytics Database
```

---

# Technologies Used

| Technology | Purpose |
|------------|----------|
| PostgreSQL 18 | Source Database |
| ClickHouse 26 | Analytical Database |
| PeerDB | CDC Engine |
| Docker | PeerDB Deployment |
| Python | Historical Data Generator |
| Faker | Synthetic Data |
| AWS EC2 | Infrastructure |
| MinIO | Snapshot Storage |

---

# AWS Infrastructure

Three EC2 instances were used.

## PostgreSQL Server

- Ubuntu 24.04
- PostgreSQL 18
- Logical Replication Enabled

---

## ClickHouse Server

- Ubuntu 24.04
- ClickHouse 26
- Native TCP Port 9000
- HTTP Port 8123

---

## PeerDB Server

- Ubuntu 24.04
- Docker
- Docker Compose
- PeerDB
- MinIO
- Temporal

---

# Dataset

The following e-commerce dataset was generated.

| Table | Records |
|---------|---------:|
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

# Features

- Initial Snapshot
- Continuous CDC
- Automatic Replication
- Low Latency
- Soft Deletes
- Historical Synchronization
- Automatic Table Creation
- Schema Detection
- Production Ready Architecture

---

# Project Workflow

```
Generate Historical Data
        │
        ▼
Insert into PostgreSQL
        │
        ▼
Logical Replication (WAL)
        │
        ▼
PeerDB
        │
        ▼
Snapshot
        │
        ▼
ClickHouse
        │
        ▼
Continuous CDC
```

---

# PostgreSQL Configuration

Logical Replication was enabled.

```
wal_level = logical

max_replication_slots = 10

max_wal_senders = 10
```

A publication was created for all tables.

```
CREATE PUBLICATION peerdb_publication
FOR TABLE
categories,
suppliers,
customers,
products,
inventory,
orders,
order_items,
payments,
shipments,
returns;
```

The application user was granted replication permission.

```
ALTER ROLE app_user WITH REPLICATION;
```

---

# Historical Data Generation

Python was used to generate realistic e-commerce data.

Generated Tables

- Customers
- Categories
- Suppliers
- Products
- Inventory
- Orders
- Order Items
- Payments
- Shipments
- Returns

The data generator uses

- Faker
- execute_values()
- Batch Inserts
- Randomized Business Logic

---

# PeerDB Configuration

Created PostgreSQL Peer

Created ClickHouse Peer

Created Mirror

Mirror Settings

- Initial Copy
- CDC Enabled
- Sync Interval 60 Seconds
- Parallelism 4

---

# Live CDC Demo

## Insert

Inserted into PostgreSQL

```sql
INSERT INTO customers
(
first_name,
last_name,
email,
phone,
city,
state,
country
)
VALUES
(
'Prithviraj',
'Powar',
'prithviraj.peerdb@test.com',
'9876543210',
'Kolhapur',
'Maharashtra',
'India'
);
```

Automatically replicated into ClickHouse.

Verified successfully.

---

# Final Architecture

```
                    PostgreSQL
                          │
                 WAL Logical Replication
                          │
                          ▼
                     PeerDB Mirror
                 Snapshot + Streaming
                          │
                          ▼
                         MinIO
                          │
                          ▼
                     ClickHouse
                  Analytics Database
```

---

# Troubleshooting

During implementation the following production issues were solved.

- ClickHouse listening only on localhost
- Docker Permission Denied
- Replication Role Permission
- Publication Missing
- Snapshot Failed
- MinIO Endpoint Issue
- host.docker.internal DNS Error
- Docker Networking
- AWS Security Groups
- Replication Slot Activation
- Peer Validation Errors

---

# Results

Successfully implemented

- PostgreSQL Logical Replication
- Historical Snapshot
- Continuous CDC
- Live Inserts
- Automatic Replication
- Production Deployment on AWS

---

# Future Work

- PostgreSQL → Debezium → Kafka → ClickHouse
- Kafka Streams
- Materialized Views
- ClickHouse Kafka Engine
- Real-Time Dashboard
- Grafana
- Apache Superset

---

# Author

Prithviraj Powar

Data Engineer
