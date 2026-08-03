# PostgreSQL Setup

## Overview

PostgreSQL acts as the source transactional database in this CDC pipeline.

All customer transactions, products, inventory, orders, payments, shipments and returns are stored inside PostgreSQL.

PeerDB continuously captures changes from PostgreSQL using Logical Replication (Write Ahead Log - WAL).

---

# Environment

Operating System

- Ubuntu 24.04 LTS

Database

- PostgreSQL 18

Database Name

```
cdc_demo
```

Application User

```
app_user
```

---

# PostgreSQL Installation

Update packages

```bash
sudo apt update
```

Install PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib
```

Verify installation

```bash
psql --version
```

Start service

```bash
sudo systemctl enable postgresql

sudo systemctl start postgresql
```

Verify status

```bash
sudo systemctl status postgresql
```

---

# Database Creation

Login

```bash
sudo -u postgres psql
```

Create database

```sql
CREATE DATABASE cdc_demo;
```

Create application user

```sql
CREATE USER app_user WITH PASSWORD '********';
```

Grant privileges

```sql
GRANT ALL PRIVILEGES ON DATABASE cdc_demo TO app_user;
```

Connect

```sql
\c cdc_demo
```

Grant schema permissions

```sql
GRANT ALL ON SCHEMA public TO app_user;
```

---

# Logical Replication

Logical Replication was enabled to allow PeerDB to read WAL records.

Updated configuration

```
wal_level = logical

max_replication_slots = 10

max_wal_senders = 10
```

Restart PostgreSQL

```bash
sudo systemctl restart postgresql
```

Verify configuration

```sql
SHOW wal_level;

SHOW max_replication_slots;

SHOW max_wal_senders;
```

Expected

```
logical

10

10
```

---

# Replication Role

The application user was granted replication permission.

```sql
ALTER ROLE app_user WITH REPLICATION;
```

Verify

```sql
SELECT rolreplication
FROM pg_roles
WHERE rolname='app_user';
```

Expected

```
t
```

---

# Publication

PeerDB requires a publication.

Created publication

```sql
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

Verify

```sql
SELECT *
FROM pg_publication;
```

---

# Schema

The following tables were created.

Categories

Suppliers

Customers

Products

Inventory

Orders

Order Items

Payments

Shipments

Returns

Each table contains

- Primary Key

- Foreign Keys

- Appropriate Data Types

- Business Relationships

---

# Data Generation

Historical data was generated using Python.

Generated records

| Table | Rows |
|---------|------:|
| Categories | 101 |
| Suppliers | 500 |
| Customers | 10000 |
| Products | 5000 |
| Inventory | 5000 |
| Orders | 100000 |
| Order Items | 300270 |
| Payments | 100000 |
| Shipments | 100000 |
| Returns | 10000 |

---

# Validation

Verify counts

```sql
SELECT COUNT(*) FROM categories;

SELECT COUNT(*) FROM suppliers;

SELECT COUNT(*) FROM customers;

SELECT COUNT(*) FROM products;

SELECT COUNT(*) FROM inventory;

SELECT COUNT(*) FROM orders;

SELECT COUNT(*) FROM order_items;

SELECT COUNT(*) FROM payments;

SELECT COUNT(*) FROM shipments;

SELECT COUNT(*) FROM returns;
```

Expected

```
Categories      101

Suppliers       500

Customers       10000

Products        5000

Inventory       5000

Orders          100000

Order Items     300270

Payments        100000

Shipments       100000

Returns         10000
```

---

# Replication Validation

Verify publication

```sql
SELECT *
FROM pg_publication;
```

Verify replication slot

```sql
SELECT *

FROM pg_replication_slots;
```

Verify replication status

```sql
SELECT

application_name,

state,

sync_state

FROM pg_stat_replication;
```

---

# Summary

PostgreSQL was configured as the source transactional database with Logical Replication enabled.

A dedicated replication user and publication were created for PeerDB.

The database was populated with realistic historical e-commerce data, providing a complete source dataset for initial snapshot and continuous Change Data Capture.
