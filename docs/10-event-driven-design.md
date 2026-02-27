# Retail Banking System – Event-Driven Design

## 1. Event-Driven Philosophy

The system follows a hybrid model:

- Core financial mutation → Synchronous (REST + ACID)
- Side effects → Asynchronous (Kafka events)

Events are published only after successful database commit.

This guarantees:
- No phantom events
- No financial inconsistency
- Loose coupling between services

---

# 2. Kafka Topics

## 2.1 transaction.completed

Published by:
- Account Service

Consumed by:
- Transaction Service
- Audit Service
- Notification Service
- Fraud Detection Service
- Reporting Service

---

## 2.2 fraud.alert (future extension)

Published by:
- Fraud Detection Service

Consumed by:
- Notification Service
- Audit Service

---

# 3. Event Schema – TransactionCompletedEvent

Example structure:

{
"eventId": "uuid",
"eventType": "TRANSACTION_COMPLETED",
"transactionId": "uuid",
"transactionType": "DEPOSIT | WITHDRAW | TRANSFER",
"sourceAccount": "ACC123",
"targetAccount": "ACC456",
"amount": 10000,
"initiatedBy": "userId",
"timestamp": "2026-01-01T10:00:00Z"
}

Event fields must be immutable.

---

# 4. Event Publishing Strategy

Publishing occurs:

1. Inside Account Service
2. After successful DB commit
3. Within same request lifecycle

If event publishing fails:
- Retry mechanism triggered
- If still fails → move to retry topic

---

# 5. Delivery Guarantee

Kafka provides:

- At-least-once delivery

This means:
- Consumers must be idempotent.
- Duplicate event processing must not cause data corruption.

---

# 6. Consumer Responsibilities

Each consuming service must:

- Validate event schema
- Handle duplicates safely
- Log processing status
- Handle failure gracefully

Consumers must not:

- Block other consumers
- Throw unhandled exceptions
- Cause cascading failures

---

# 7. Idempotent Consumer Design

Example:

Transaction Service:

- Before inserting transaction record:
    - Check if transactionId already exists.
- If exists:
    - Ignore duplicate event.
- If not:
    - Insert new record.

This prevents duplication due to event redelivery.

---

# 8. Retry Strategy

Consumers must implement:

- Configurable retry attempts (e.g., 3 retries)
- Exponential backoff

If retries exhausted:
- Publish event to Dead Letter Topic (DLT)

---

# 9. Dead Letter Topic (DLT)

Purpose:

- Capture failed event processing
- Prevent infinite retry loops
- Allow manual inspection

Example:

transaction.completed.dlt

DLT records must include:
- Original event
- Failure reason
- Retry count

---

# 10. Event Ordering

Partitioning strategy:

- Events may be partitioned by accountNumber
- This ensures ordering for same account

Guarantee:
Events for same account are processed in order.

---

# 11. Event Versioning

Each event must include:

- eventType
- version field (future extension)

This allows backward compatibility
when schema evolves.

---

# 12. Why Hybrid Instead of Full Saga

Full Saga approach was evaluated.

However:

- Balance mutation must remain strongly consistent.
- Simpler architecture preferred for financial safety.
- Hybrid provides best trade-off between reliability and complexity.

---

# 13. Failure Isolation

Failure of one consumer:

- Does not affect other consumers.
- Does not rollback financial transaction.
- Does not block core Account Service.

This ensures resilience and loose coupling.