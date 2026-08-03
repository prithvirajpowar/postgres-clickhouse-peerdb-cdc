# CDC Testing & Validation

# Overview

After configuring PostgreSQL, ClickHouse and PeerDB, the complete Change Data Capture (CDC) pipeline was validated.

Testing was performed in two phases.

1. Initial Snapshot Validation
2. Continuous Change Data Capture Validation

The objective was to ensure that historical data and all future database changes were successfully replicated from PostgreSQL into ClickHouse.

---

# Initial Snapshot Validation

PeerDB automatically copied all historical records from PostgreSQL into ClickHouse.

The snapshot completed successfully and the mirror status changed to

```
Running
```

---

# Historical Data Validation

The following row counts were verified.

| Table | PostgreSQL | ClickHouse |
|---------|-----------:|-----------:|
| Categories | 101 | 101 |
| Suppliers | 500 | 500 |
| Customers | 10,000 | 10,000 |
| Products | 5,000 | 5,000 |
| Inventory | 5,000 | 5,000 |
| Orders | 100,000 | 100,000 |
| Order Items | 300,270 | 300,270 |
| Payments | 100,000 | 100,000 |
| Shipments | 100,000 | 100,000 |
| Returns | 10,000 | 10,000 |

The row counts matched successfully.

---

# INSERT Validation

A new customer was inserted into PostgreSQL.

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

---

# Verify in ClickHouse

```sql
SELECT *
FROM public_customers
WHERE email='prithviraj.peerdb@test.com';
```

Result

The record appeared successfully inside ClickHouse.

This confirmed that PeerDB continuously replicated INSERT operations.

---

# UPDATE Validation

The customer record was updated in PostgreSQL.

```sql
UPDATE customers

SET city='Pune'

WHERE email='prithviraj.peerdb@test.com';
```

Verification

```sql
SELECT *

FROM public_customers FINAL

WHERE email='prithviraj.peerdb@test.com';
```

Expected

```
City = Pune
```

This confirms UPDATE replication.

---

# DELETE Validation

The customer record was deleted.

```sql
DELETE

FROM customers

WHERE email='prithviraj.peerdb@test.com';
```

Verification

```sql
SELECT *

FROM public_customers FINAL

WHERE email='prithviraj.peerdb@test.com';
```

PeerDB marks deleted records using metadata columns depending on the configured delete strategy.

---

# Metadata Columns

PeerDB automatically adds metadata columns.

| Column | Purpose |
|----------|----------|
| _peerdb_synced_at | Synchronization Timestamp |
| _peerdb_version | CDC Version |
| _peerdb_is_deleted | Soft Delete Indicator |

These columns help maintain consistency and correctly apply updates and deletes.

---

# Replication Flow

```
Application

↓

PostgreSQL

↓

WAL

↓

PeerDB

↓

ClickHouse
```

---

# Validation Queries

Verify customers

```sql
SELECT count()

FROM public_customers;
```

Verify orders

```sql
SELECT count()

FROM public_orders;
```

Verify order items

```sql
SELECT count()

FROM public_order_items;
```

---

# Replication Status

Mirror Status

```
Running
```

Snapshot

```
Completed
```

Continuous CDC

```
Active
```

---

# Expected Behavior

| Operation | PostgreSQL | ClickHouse |
|------------|------------|------------|
| INSERT | Yes | Replicated |
| UPDATE | Yes | Replicated |
| DELETE | Yes | Replicated |
| INSERT into ClickHouse | No | PostgreSQL Unchanged |
| UPDATE in ClickHouse | No | PostgreSQL Unchanged |
| DELETE in ClickHouse | No | PostgreSQL Unchanged |

The replication pipeline is unidirectional.

PostgreSQL acts as the source of truth while ClickHouse acts as the analytical destination.

---

# Outcome

The CDC pipeline successfully replicated

- Historical Snapshot
- Live Inserts
- Live Updates
- Live Deletes

The implementation demonstrates a production-ready, low-latency Change Data Capture architecture suitable for analytical workloads.
