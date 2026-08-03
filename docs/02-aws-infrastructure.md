# AWS Infrastructure

## Overview

The entire CDC pipeline was deployed on Amazon Web Services (AWS) using three dedicated EC2 instances.

Each instance was assigned a specific responsibility to simulate a production deployment where services are isolated for better scalability, security and maintainability.

---

# Infrastructure Diagram

```
                    AWS Cloud
┌─────────────────────────────────────────────────────┐

      EC2 Instance 1
   PostgreSQL Database Server
        Ubuntu 24.04
        PostgreSQL 18

              │
              │ Logical Replication (WAL)
              ▼

      EC2 Instance 2
         PeerDB Server
        Ubuntu 24.04
        Docker
        PeerDB
        MinIO
        Temporal

              │
              │ Snapshot + CDC
              ▼

      EC2 Instance 3
      ClickHouse Server
        Ubuntu 24.04
        ClickHouse 26

└─────────────────────────────────────────────────────┘
```

---

# EC2 Instance Details

## PostgreSQL Server

Purpose

Source transactional database.

Responsibilities

- Store application data
- Generate WAL records
- Publish logical replication changes
- Maintain OLTP workload

Software

- Ubuntu 24.04
- PostgreSQL 18

---

## PeerDB Server

Purpose

Acts as the Change Data Capture engine.

Responsibilities

- Read PostgreSQL WAL
- Perform Initial Snapshot
- Detect Schema
- Replicate Changes
- Manage Replication Slot
- Manage Publication
- Stream data into ClickHouse

Software

- Ubuntu 24.04
- Docker
- Docker Compose
- PeerDB
- MinIO
- Temporal

---

## ClickHouse Server

Purpose

Analytical Database.

Responsibilities

- Store replicated data
- Execute OLAP queries
- High-speed aggregation
- Business Intelligence workloads

Software

- Ubuntu 24.04
- ClickHouse 26

---

# Networking

Private communication was established between all EC2 instances using AWS Private IPv4 addresses.

No data replication traffic used the public internet.

Communication Flow

```
PostgreSQL
172.x.x.x

↓

PeerDB
172.x.x.x

↓

ClickHouse
172.x.x.x
```

---

# Security Groups

The following ports were configured.

## PostgreSQL

| Port | Purpose |
|------|----------|
| 5432 | PostgreSQL |

---

## ClickHouse

| Port | Purpose |
|------|----------|
| 8123 | HTTP Interface |
| 9000 | Native Client |

---

## PeerDB

| Port | Purpose |
|------|----------|
| 3000 | PeerDB UI |
| 9001 | MinIO |
| 7233 | Temporal |

---

# Connectivity Validation

The following connectivity tests were executed.

PostgreSQL

```
nc -zv <postgres-private-ip> 5432
```

ClickHouse HTTP

```
nc -zv <clickhouse-private-ip> 8123
```

ClickHouse Native

```
nc -zv <clickhouse-private-ip> 9000
```

MinIO

```
curl http://<peerdb-private-ip>:9001/minio/health/live
```

All connectivity tests completed successfully.

---

# Docker Deployment

PeerDB was deployed using Docker Compose.

Containers

- PeerDB UI
- PeerDB Server
- Flow Worker
- Flow Snapshot Worker
- Temporal
- MinIO
- PostgreSQL Catalog

---

# Deployment Sequence

Step 1

Create PostgreSQL EC2.

↓

Step 2

Install PostgreSQL.

↓

Step 3

Generate Historical Data.

↓

Step 4

Create ClickHouse EC2.

↓

Step 5

Install ClickHouse.

↓

Step 6

Create PeerDB EC2.

↓

Step 7

Install Docker.

↓

Step 8

Deploy PeerDB.

↓

Step 9

Create Peers.

↓

Step 10

Create Mirror.

↓

Step 11

Initial Snapshot.

↓

Step 12

Continuous CDC.

---

# Advantages

Separating services across multiple EC2 instances provides

- Better Security
- Better Isolation
- Independent Scaling
- Easier Maintenance
- Production Style Architecture

---

# Final Infrastructure

The final deployment successfully replicated data from PostgreSQL into ClickHouse through PeerDB using logical replication and continuous CDC.

The entire pipeline operated over AWS private networking with isolated compute instances.
