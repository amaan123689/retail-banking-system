# Retail Banking System – Service Boundaries

## 1. Design Principle

Each microservice must have:

- Clear ownership of its data
- Clear ownership of its business logic
- No direct database sharing
- Minimal knowledge of other services

Service boundaries are defined based on domain ownership and data responsibility.

---

## 2. Authentication Service

### Owns:
- User entity
- Roles and permissions
- Password hashing
- JWT generation and validation
- Refresh token lifecycle

### Does NOT:
- Know about accounts
- Store financial data
- Validate transaction rules

Rationale:
Authentication must remain isolated from financial domain logic for security and separation of concerns.

---

## 3. Customer Service

### Owns:
- Customer profile data
- Personal information
- KYC status reference

### Does NOT:
- Store account balance
- Handle transactions
- Validate financial policies

Rationale:
Customer identity management is separate from financial operations.

---

## 4. Account Service (Financial Authority)

### Owns:
- Account entity
- Account status (ACTIVE / FROZEN / CLOSED)
- Balance field
- Daily transaction limit
- Minimum balance
- Optimistic locking version field
- Idempotency records

### Responsible For:
- Deposit
- Withdrawal
- Transfer
- Balance mutation
- Financial validation before mutation

### Does NOT:
- Store transaction history permanently
- Perform fraud analytics
- Handle notifications

Rationale:
Balance ownership must be centralized in one service to maintain strong consistency.

---

## 5. Transaction Service

### Owns:
- Transaction history records
- Transaction status
- Transaction metadata

### Consumes:
- TransactionCompletedEvent

### Does NOT:
- Modify account balance
- Validate financial policies

Rationale:
Transaction history is event-driven and separate from balance mutation to reduce coupling.

---

## 6. Policy Service

### Owns:
- Minimum balance rules
- Daily transaction limit rules
- Service charge rules
- Threshold configurations

### Provides:
- Validation API used by Account Service

### Does NOT:
- Modify balance
- Store transaction history

Rationale:
Business rules may evolve independently of account logic.

---

## 7. Audit Service

### Owns:
- Immutable audit logs
- Operation metadata
- User action records

### Consumes:
- Financial operation events
- Authentication events

### Does NOT:
- Affect financial transactions

Rationale:
Audit trail must be independent and append-only.

---

## 8. Notification Service

### Owns:
- Notification templates
- Notification delivery logs

### Consumes:
- TransactionCompletedEvent
- FraudAlertEvent

Rationale:
Notification must not block financial operations.

---

## 9. Fraud Detection Service

### Owns:
- Fraud detection rules
- Fraud alert records

### Consumes:
- TransactionCompletedEvent

### Does NOT:
- Block transaction execution in initial version

Rationale:
Fraud detection is reactive in this version to maintain system simplicity.

---

## 10. Reporting Service

### Owns:
- Aggregated financial reports
- Analytical projections

### Consumes:
- Transaction events

Rationale:
Reporting is read-heavy and must not interfere with transactional workloads.

---

## 11. API Gateway

### Responsible For:
- Routing
- Authentication validation
- Request filtering
- Rate limiting (future)
- Correlation ID injection

### Does NOT:
- Contain business logic

Rationale:
Gateway acts as entry point only.

---

## 12. Boundary Rules Summary

1. Only Account Service can mutate balance.
2. No service can query another service’s database directly.
3. Cross-service communication must use REST or Kafka.
4. Supporting services must never block core financial flow.
5. Policy validation must occur before balance mutation.

These boundary rules ensure maintainability, scalability, and correctness.