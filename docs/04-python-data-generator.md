# Python Data Generator

## Overview

To demonstrate a realistic Change Data Capture (CDC) pipeline, a synthetic e-commerce dataset was generated using Python.

Instead of using random SQL scripts, a configurable Python application was developed to generate realistic business data and populate PostgreSQL.

The generator creates historical data before enabling CDC replication.

---

# Objective

The purpose of the data generator is to

- Generate realistic business data
- Maintain foreign key relationships
- Simulate an e-commerce platform
- Create sufficient historical data for CDC testing
- Support repeatable testing

---

# Technologies Used

| Technology | Purpose |
|------------|----------|
| Python 3 | Application Development |
| Faker | Generate Realistic Data |
| Psycopg2 | PostgreSQL Connectivity |
| execute_values() | Batch Inserts |
| PostgreSQL | Data Storage |

---

# Project Structure

```
cdc-data-generator/

├── generators/
│   ├── categories.py
│   ├── suppliers.py
│   ├── customers.py
│   ├── products.py
│   ├── inventory.py
│   ├── orders.py
│   ├── order_items.py
│   ├── payments.py
│   ├── shipments.py
│   └── returns.py
│
├── utils/
│   ├── constants.py
│   └── helpers.py
│
├── db.py
├── main.py
├── requirements.txt
└── reset.sql
```

---

# Generated Dataset

The application generated the following historical dataset.

| Table | Records |
|--------|---------:|
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

# Data Relationships

The generator maintains relational integrity.

```
Categories
      │
      ▼
Products
      │
      ▼
Inventory

Customers
      │
      ▼
Orders
      │
      ▼
Order Items
      │
      ▼
Products

Orders
 │
 ├── Payments
 │
 ├── Shipments
 │
 └── Returns
```

---

# Data Generation Flow

```
Categories

↓

Suppliers

↓

Customers

↓

Products

↓

Inventory

↓

Orders

↓

Order Items

↓

Payments

↓

Shipments

↓

Returns
```

The generation order ensures that all foreign key references already exist before dependent tables are populated.

---

# Batch Inserts

Instead of inserting one row at a time, the application uses PostgreSQL batch inserts.

```
execute_values()
```

Advantages

- Faster inserts
- Reduced network overhead
- Lower transaction cost
- Better scalability

---

# Faker Library

The Faker library was used to generate realistic values including

- Customer Names
- Emails
- Cities
- States
- Phone Numbers
- Product Names
- Supplier Names

This makes the dataset suitable for demonstrations and analytics.

---

# Business Logic

Randomized business logic was implemented for

- Product Prices
- Inventory Quantity
- Order Status
- Payment Method
- Shipment Status
- Return Reason
- Order Dates

The generated data resembles a real e-commerce workload.

---

# Configuration

The generator uses a centralized configuration file.

Example parameters include

- Customer Count
- Supplier Count
- Product Count
- Order Count
- Payment Count
- Shipment Count
- Return Count
- Batch Size

This allows datasets of different sizes to be generated without changing application logic.

---

# Reset Script

A reset script truncates all tables before generating a new dataset.

Benefits

- Clean testing environment
- Repeatable execution
- Sequence reset
- Foreign key consistency

---

# Execution

Run the generator

```bash
python main.py
```

The application automatically executes every generator module in sequence.

---

# Validation

After generation, row counts are verified for every table.

Example

```sql
SELECT COUNT(*) FROM customers;

SELECT COUNT(*) FROM orders;

SELECT COUNT(*) FROM order_items;
```

The generated data was successfully inserted into PostgreSQL and later replicated into ClickHouse using PeerDB.

---

# Summary

The Python data generator creates a realistic e-commerce dataset with relational integrity and configurable scale.

Using batch inserts and Faker, it efficiently generates historical data that serves as the foundation for snapshot loading and continuous CDC testing.
