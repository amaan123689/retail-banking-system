# Retail Banking System – Transaction Handling Strategy

## 1. Core Principle

Balance mutation must remain strongly consistent.

All financial operations (deposit, withdraw, transfer) are executed
inside a single ACID database transaction within the Account Service.

No distributed two-phase commit is used.

---

# 2. Deposit Flow

Step 1: Client → API Gateway → Account Service

Step 2: Account Service begins database transaction

Inside the transaction:

- Validate account exists
- Validate account status = ACTIVE
- Validate idempotency key
- Add amount to balance
- Increment version field (optimistic locking)
- Persist idempotency record
- Commit transaction

Step 3: After successful commit:
- Publish TransactionCompletedEvent to Kafka

Step 4: Event consumers:
- Transaction Service stores record
- Audit Service logs action
- Notification Service sends alert
- Fraud Service evaluates activity

---

# 3. Withdrawal Flow

Step 1: Client → Gateway → Account Service

Step 2: Begin database transaction

Inside transaction:

- Validate account exists
- Validate status = ACTIVE
- Validate sufficient balance
- Validate minimum balance rule
- Validate daily transaction limit
- Validate idempotency key
- Deduct amount
- Update daily transferred amount
- Increment version field
- Persist idempotency record
- Commit

Step 3: Publish TransactionCompletedEvent

---

# 4. Transfer Flow

Transfer is implemented as atomic debit-credit within Account Service.

Step 1: Client → Gateway → Account Service

Step 2: Begin database transaction

Inside transaction:

- Validate source and target accounts exist
- Validate both accounts are ACTIVE
- Validate sufficient balance in source
- Validate minimum balance rule
- Validate daily limit
- Validate idempotency key
- Debit source account
- Credit target account
- Increment version fields
- Persist idempotency record
- Commit

Step 3: Publish TransactionCompletedEvent

Important:
If any validation fails → transaction is rolled back.

---

# 5. Why Balance Is Updated Before Event Publishing

Event publishing happens AFTER successful database commit.

This guarantees:

- No event exists without committed balance update.
- Consumers only react to confirmed transactions.
- Prevents inconsistent system state.

---

# 6. Optimistic Locking Strategy

Account entity contains:

- version field (BIGINT)

When updating:

- WHERE id = ? AND version = ?

If version mismatch:
- Update fails
- OptimisticLockingFailureException thrown
- Operation retried or returned as conflict (409)

This prevents concurrent double-spending.

---

# 7. Idempotency Strategy

Each financial request must include:

Header:
Idempotency-Key: unique-value

Account Service:

- Checks if key exists.
- If exists → return stored response.
- If not → process normally.
- Store response payload against key.

This prevents duplicate debit due to network retries.

---

# 8. Failure Handling

If DB transaction fails:
- No event published.
- Client receives error response.

If Kafka publish fails:
- Retry mechanism triggered.
- Dead Letter Topic used if retries exhausted.

If downstream service fails:
- Core transaction remains successful.
- Failure isolated from balance mutation.

---

# 9. Consistency Model

Balance updates → Strong consistency  
Transaction history → Eventual consistency  
Audit logs → Eventual consistency  
Notifications → Eventual consistency

This is intentional to protect financial correctness.

---

# 10. No Distributed Transaction Across Services

System intentionally avoids:

- Two-phase commit
- Global transaction managers

Instead:

- Balance authority centralized in Account Service
- Events propagate outcome

This simplifies reasoning and improves reliability.

---

# 11. Edge Case Handling

- Duplicate request → Idempotency prevents double debit.
- Concurrent withdrawals → Optimistic locking prevents negative balance.
- Service restart → Kafka ensures event redelivery.
- Notification failure → Does not rollback transaction.