# Retail Banking System – Concurrency Control Strategy

## 1. Problem Statement

In financial systems, multiple concurrent operations may attempt to modify the same account balance.

Example scenario:

Initial balance: 1000

Thread A withdraws 800  
Thread B withdraws 500

If both read balance before update:
- Both see 1000
- Both succeed
- Final balance becomes negative

This is unacceptable in a banking system.

---

## 2. Chosen Strategy: Optimistic Locking

The system uses optimistic locking via a version field in the Account entity.

Account table contains:

- version (BIGINT)

Each update operation:

UPDATE accounts
SET balance = ?, version = version + 1
WHERE id = ? AND version = ?

If version does not match:
- No rows updated
- OptimisticLockingFailureException thrown

---

## 3. Why Optimistic Locking?

Compared to pessimistic locking:

Pessimistic Locking:
- Locks row at DB level
- Reduces concurrency
- Can cause deadlocks
- Higher contention under load

Optimistic Locking:
- No row lock during read
- Higher throughput
- Detects conflict only at write time
- Suitable for systems with moderate contention

For retail banking workloads, optimistic locking provides better scalability.

---

## 4. Conflict Handling Strategy

If optimistic locking fails:

Option A:
- Return HTTP 409 Conflict

Option B:
- Retry operation (limited attempts)

System design:
- Retry up to configurable number of times.
- If still failing → return 409.

---

## 5. Double-Spend Prevention

Double-spending is prevented by:

1. Optimistic locking (version check)
2. Balance validation inside transaction
3. Idempotency key validation

All three combined ensure financial safety.

---

## 6. Isolation Level

Database isolation level:

- READ COMMITTED (default PostgreSQL level)

Reason:
- Prevents dirty reads
- Combined with optimistic locking ensures correctness
- Avoids performance cost of SERIALIZABLE isolation

---

## 7. Atomicity Guarantee

All balance modifications happen inside a single ACID transaction.

If any validation fails:
- Transaction rolled back
- No partial updates occur

---

## 8. Concurrency Testing Strategy (Future Implementation)

Planned testing approach:

- Simulate concurrent withdrawal using multithreaded test.
- Validate no negative balance occurs.
- Validate only one transaction succeeds when balance insufficient.

---

## 9. Summary

Concurrency safety is achieved using:

- ACID transaction boundary
- Optimistic locking via version field
- Idempotency enforcement
- Proper validation order

This ensures:

- No negative balances
- No race condition losses
- High throughput under concurrent load