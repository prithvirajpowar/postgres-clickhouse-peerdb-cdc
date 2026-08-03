# PeerDB Setup

# Overview

PeerDB is the Change Data Capture (CDC) engine used in this project.

It continuously replicates data from PostgreSQL into ClickHouse with minimal latency by reading PostgreSQL Write Ahead Log (WAL).

PeerDB performs two major operations.

- Initial Snapshot
- Continuous Change Data Capture

This allows ClickHouse to remain synchronized with PostgreSQL without affecting transactional workloads.

---

# Environment

Operating System

- Ubuntu 24.04

Deployment

- Docker Compose

PeerDB Version

- Stable v0.37.0

---

# Components

The following services were deployed.

| Component | Purpose |
|------------|----------|
| PeerDB UI | Web Interface |
| PeerDB Server | Control Plane |
| Flow Worker | CDC Processing |
| Snapshot Worker | Initial Snapshot |
| Temporal | Workflow Engine |
| MinIO | Snapshot Storage |
| Catalog PostgreSQL | Internal Metadata |

---

# Docker Installation

Update packages

```bash
sudo apt update
```

Install Docker

```bash
sudo apt install docker.io docker-compose-v2
```

Enable Docker

```bash
sudo systemctl enable docker

sudo systemctl start docker
```

Verify

```bash
docker ps
```

---

# Clone PeerDB

```bash
git clone https://github.com/PeerDB-io/peerdb.git

cd peerdb
```

---

# Deploy PeerDB

Start all services

```bash
docker compose up -d
```

Verify

```bash
docker ps
```

Expected running containers

- peerdb-ui
- peerdb-server
- flow-worker
- flow-snapshot-worker
- temporal
- temporal-ui
- temporal-admin-tools
- catalog
- minio

---

# PeerDB UI

Open

```
http://<peerdb-public-ip>:3000
```

The UI allows

- Create Peers
- Create Mirrors
- Monitor Snapshot
- Monitor CDC
- View Logs

---

# PostgreSQL Peer

Configured

- Host
- Port 5432
- Database
- Username
- Password

Validation completed successfully.

---

# ClickHouse Peer

Configured

- Host
- Port 9000
- Database
- Username
- Password

Validation completed successfully.

---

# PostgreSQL Preparation

Logical Replication

```
wal_level = logical
```

Replication Role

```sql
ALTER ROLE app_user WITH REPLICATION;
```

Publication

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

---

# Mirror Configuration

Mirror Name

```
postgres_to_clickhouse
```

Mode

- CDC

Initial Copy

- Enabled

Sync Interval

- 60 Seconds

Parallelism

- 4

Publication

```
peerdb_publication
```

Tables Selected

- categories
- suppliers
- customers
- products
- inventory
- orders
- order_items
- payments
- shipments
- returns

---

# Snapshot

PeerDB automatically performed

- Schema Detection
- Table Creation
- Historical Data Copy

The following tables were created automatically.

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

---

# Continuous CDC

After snapshot completion PeerDB automatically switched to continuous streaming.

Every

- INSERT
- UPDATE
- DELETE

performed on PostgreSQL was replicated into ClickHouse.

---

# Validation

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

Within the configured synchronization interval the record appeared inside ClickHouse.

This confirmed successful end-to-end replication.

---

# Challenges Encountered

Several production issues were encountered during implementation.

## PostgreSQL Replication Permission

Problem

```
permission denied to start WAL sender
```

Solution

```sql
ALTER ROLE app_user WITH REPLICATION;
```

---

## Missing Publication

Problem

PeerDB validation failed because no publication existed.

Solution

```sql
CREATE PUBLICATION peerdb_publication
...
```

---

## ClickHouse Listening Only On Localhost

Problem

ClickHouse accepted connections only from localhost.

Solution

Updated

```
listen_host
```

to

```
0.0.0.0
```

Restarted ClickHouse.

---

## Docker Permission

Problem

```
permission denied while trying to connect to Docker daemon
```

Solution

```bash
sudo usermod -aG docker $USER

newgrp docker
```

---

## MinIO Endpoint

Problem

Initial snapshot failed.

Mirror logs reported

```
host.docker.internal
DNS_ERROR
```

Root Cause

PeerDB Docker Compose used

```
host.docker.internal
```

which is intended for Docker Desktop environments.

ClickHouse was deployed on a separate EC2 instance and could not resolve that hostname.

Solution

Updated

```yaml
PEERDB_CLICKHOUSE_AWS_CREDENTIALS_AWS_ENDPOINT_URL_S3
```

from

```
http://host.docker.internal:9001
```

to

```
http://<peerdb-private-ip>:9001
```

Restarted PeerDB containers.

Deleted the failed mirror.

Created a new mirror.

Snapshot completed successfully.

---

# Final Result

Mirror Status

```
Running
```

Historical Snapshot

Completed Successfully.

Continuous CDC

Running Successfully.

Data Validation

Successful.

---

# Summary

PeerDB successfully replicated the complete e-commerce dataset from PostgreSQL to ClickHouse.

The implementation included snapshot loading, continuous CDC, automatic schema creation, and live synchronization while resolving real-world deployment issues encountered in a multi-EC2 AWS environment.
