# Retail Banking System – Database Design

## 1. Database Strategy

- Each microservice owns its own PostgreSQL database.
- No cross-service joins are allowed.
- Each service manages its own schema migrations.
- Monetary values are stored in smallest currency unit (e.g., cents) using BIGINT.
- All critical tables must include:
    - created_at
    - updated_at

---

# 2. Authentication Service Database

## Table: users

| Column Name      | Type        | Constraints                    |
|------------------|------------|--------------------------------|
| id               | UUID       | Primary Key                    |
| email            | VARCHAR    | Unique, Not Null               |
| password_hash    | VARCHAR    | Not Null                       |
| role             | VARCHAR    | Not Null                       |
| is_active        | BOOLEAN    | Default TRUE                   |
| created_at       | TIMESTAMP  | Not Null                       |
| updated_at       | TIMESTAMP  | Not Null                       |

Indexes:
- Unique index on email

---

# 3. Customer Service Database

## Table: customers

| Column Name     | Type       | Constraints         |
|-----------------|-----------|---------------------|
| id              | UUID      | Primary Key         |
| user_id         | UUID      | Not Null            |
| full_name       | VARCHAR   | Not Null            |
| phone           | VARCHAR   |                     |
| address         | VARCHAR   |                     |
| kyc_status      | VARCHAR   | Default PENDING     |
| created_at      | TIMESTAMP | Not Null            |
| updated_at      | TIMESTAMP | Not Null            |

Note:
- user_id references Authentication Service logically (not via foreign key)

---

# 4. Account Service Database

## Table: accounts

| Column Name            | Type       | Constraints                    |
|------------------------|-----------|--------------------------------|
| id                     | UUID      | Primary Key                    |
| account_number         | VARCHAR   | Unique, Not Null               |
| customer_id            | UUID      | Not Null                       |
| account_type           | VARCHAR   | Not Null                       |
| balance                | BIGINT    | Not Null                       |
| minimum_balance        | BIGINT    | Not Null                       |
| daily_limit            | BIGINT    | Not Null                       |
| daily_transferred_amt  | BIGINT    | Default 0                      |
| status                 | VARCHAR   | Default ACTIVE                 |
| version                | BIGINT    | Not Null (Optimistic Locking)  |
| created_at             | TIMESTAMP | Not Null                       |
| updated_at             | TIMESTAMP | Not Null                       |

Indexes:
- Unique index on account_number
- Index on customer_id

Important:
- version field is used for optimistic locking.
- balance must never go negative.
- daily_transferred_amt resets daily (handled via scheduled job).

---

## Table: idempotency_keys

| Column Name      | Type       | Constraints        |
|------------------|-----------|--------------------|
| id               | UUID      | Primary Key        |
| idempotency_key  | VARCHAR   | Unique, Not Null   |
| account_id       | UUID      | Not Null           |
| response_payload | TEXT      | Not Null           |
| created_at       | TIMESTAMP | Not Null           |

Purpose:
Prevents duplicate financial operations.

---

# 5. Transaction Service Database

## Table: transactions

| Column Name      | Type       | Constraints                  |
|------------------|-----------|------------------------------|
| id               | UUID      | Primary Key                  |
| transaction_type | VARCHAR   | Not Null                     |
| amount           | BIGINT    | Not Null                     |
| source_account   | VARCHAR   |                              |
| target_account   | VARCHAR   |                              |
| status           | VARCHAR   | Not Null                     |
| initiated_by     | UUID      | Not Null                     |
| created_at       | TIMESTAMP | Not Null                     |

Indexes:
- Index on source_account
- Index on target_account
- Index on created_at

---

# 6. Policy Service Database

## Table: policy_rules

| Column Name    | Type       | Constraints      |
|----------------|-----------|------------------|
| id             | UUID      | Primary Key      |
| rule_name      | VARCHAR   | Unique           |
| rule_value     | BIGINT    | Not Null         |
| is_active      | BOOLEAN   | Default TRUE     |
| created_at     | TIMESTAMP | Not Null         |
| updated_at     | TIMESTAMP | Not Null         |

Purpose:
Allows configurable rule thresholds.

---

# 7. Audit Service Database

## Table: audit_logs

| Column Name   | Type       | Constraints      |
|---------------|-----------|------------------|
| id            | UUID      | Primary Key      |
| user_id       | UUID      |                  |
| action_type   | VARCHAR   | Not Null         |
| entity_id     | VARCHAR   |                  |
| result        | VARCHAR   | Not Null         |
| metadata      | TEXT      |                  |
| created_at    | TIMESTAMP | Not Null         |

Important:
- Audit logs are immutable.
- No updates allowed after insert.

---

# 8. Fraud Service Database

## Table: fraud_alerts

| Column Name   | Type       | Constraints      |
|---------------|-----------|------------------|
| id            | UUID      | Primary Key      |
| account_id    | UUID      | Not Null         |
| transaction_id| UUID      | Not Null         |
| risk_score    | INTEGER   | Not Null         |
| status        | VARCHAR   | Default OPEN     |
| created_at    | TIMESTAMP | Not Null         |

---

# 9. Data Integrity Rules

1. Balance must never be negative.
2. Monetary values must be positive.
3. Version field must increment on each update.
4. Transaction record must not exist without corresponding event.
5. Idempotency keys must expire after configurable duration.

---

# 10. Why Balance Is Stored Directly

Balance is stored in the Account table instead of being derived from transaction history because:

- Real-time aggregation is expensive.
- Concurrency complexity increases.
- Strong consistency is easier to maintain with single authoritative balance field.