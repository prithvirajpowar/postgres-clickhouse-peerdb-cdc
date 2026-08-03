# ClickHouse Setup

## Overview

ClickHouse is used as the destination analytical database in this project.

PeerDB continuously replicates data from PostgreSQL into ClickHouse using Change Data Capture (CDC), making ClickHouse available for high-performance analytical queries.

---

# Environment

Operating System

- Ubuntu 24.04 LTS

Database

- ClickHouse 26.x (Latest Stable)

Deployment

- Dedicated AWS EC2 Instance

---

# Installation

Update packages

```bash
sudo apt update
```

Install ClickHouse using the official installation guide.

Verify installation

```bash
clickhouse-client --version
```

Start the service

```bash
sudo systemctl enable clickhouse-server

sudo systemctl start clickhouse-server
```

Verify service status

```bash
sudo systemctl status clickhouse-server
```

---

# Network Configuration

By default, ClickHouse listens only on localhost.

To allow PeerDB running on another EC2 instance to connect, the listen host was updated.

Configuration file

```
/etc/clickhouse-server/config.xml
```

Updated configuration

```xml
<listen_host>0.0.0.0</listen_host>
```

Restart ClickHouse

```bash
sudo systemctl restart clickhouse-server
```

Verify listening ports

```bash
sudo ss -ltn | grep -E "8123|9000"
```

Expected

```
0.0.0.0:8123

0.0.0.0:9000
```

---

# Connectivity Test

From the PeerDB EC2 instance

HTTP Port

```bash
nc -zv <clickhouse-private-ip> 8123
```

Native Port

```bash
nc -zv <clickhouse-private-ip> 9000
```

Both ports should be reachable.

---

# Database Creation

Login

```bash
clickhouse-client
```

Create database

```sql
CREATE DATABASE cdc_demo;
```

Verify

```sql
SHOW DATABASES;
```

---

# Destination Tables

PeerDB automatically creates destination tables during the initial snapshot.

Replicated Tables

- public_categories
- public_suppliers
- public_customers
- public_products
- public_inventory
- public_orders
- public_order_items
- public_payments
- public_shipments
- public_returns

No manual table creation is required.

---

# Table Engine

PeerDB creates ClickHouse tables using engines optimized for CDC workloads.

These tables support

- Versioning
- Soft Deletes
- Efficient Merges
- Fast Analytical Queries

Additional metadata columns are created automatically.

Examples

- _peerdb_synced_at
- _peerdb_version
- _peerdb_is_deleted

These columns help track replication state and support update/delete handling.

---

# Validation

Verify tables

```sql
SHOW TABLES;
```

Verify row counts

```sql
SELECT count() FROM public_customers;

SELECT count() FROM public_orders;

SELECT count() FROM public_order_items;
```

The counts should match the PostgreSQL source after the initial snapshot completes.

---

# Live Verification

Insert a record into PostgreSQL.

Wait for the configured sync interval.

Query ClickHouse

```sql
SELECT *
FROM public_customers
WHERE email='prithviraj.peerdb@test.com';
```

The record should appear automatically without manual intervention.

---

# Performance

ClickHouse provides

- High-speed aggregations
- Columnar storage
- Compression
- Parallel query execution
- Excellent analytical performance

This makes it well suited for reporting and business intelligence workloads while PostgreSQL continues handling transactional operations.

---

# Summary

ClickHouse serves as the analytical destination in the CDC pipeline.

After enabling network access and creating the destination database, PeerDB automatically created tables, performed the initial snapshot, and continuously synchronized changes from PostgreSQL.

The result is a real-time analytical database that stays synchronized with the operational PostgreSQL database.
