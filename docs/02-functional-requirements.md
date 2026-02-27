# Retail Banking System – Functional Requirements

## 1. Authentication & Authorization

### 1.1 User Registration
- The system shall allow customer self-registration.
- The system shall allow employee registration (admin-controlled).
- Passwords must be stored using BCrypt hashing.
- Email must be unique per user.

### 1.2 Login
- The system shall authenticate users using email and password.
- Upon successful authentication, the system shall issue:
    - Access Token (short-lived JWT)
    - Refresh Token (long-lived)

### 1.3 Role-Based Access Control
- The system shall support the following roles:
    - CUSTOMER
    - EMPLOYEE
    - ADMIN
- Access to APIs shall be restricted based on role.
- Account operations must validate ownership before execution.

---

## 2. Customer Management

### 2.1 Create Customer
- Employees shall be able to create customer profiles.
- Customer must contain:
    - Full name
    - Email
    - Phone number
    - Address
    - KYC status

### 2.2 Update Customer
- Customers may update limited profile fields.
- Employees may update all profile fields.

### 2.3 Delete Customer
- Only ADMIN may delete a customer.
- Customer deletion must fail if active accounts exist.

---

## 3. Account Management

### 3.1 Create Account
- Employees shall create accounts for customers.
- Each account must include:
    - Account number (unique)
    - Account type (SAVINGS / CURRENT)
    - Balance
    - Account status (ACTIVE / FROZEN / CLOSED)
    - Daily transaction limit
    - Minimum balance

### 3.2 View Account
- Customers shall view only their own accounts.
- Employees may view any customer account.

### 3.3 Freeze Account
- Employees may freeze accounts.
- Frozen accounts cannot perform debit operations.

---

## 4. Deposit Operation

- The system shall allow deposits into ACTIVE accounts.
- Deposits shall:
    - Increase balance
    - Be stored as a transaction record
    - Trigger audit and notification events
- Deposit must be idempotent using Idempotency-Key.

---

## 5. Withdrawal Operation

- The system shall allow withdrawals from ACTIVE accounts.
- Withdrawal shall:
    - Validate sufficient balance
    - Validate minimum balance rule
    - Validate daily transaction limit
    - Use optimistic locking for concurrency safety
    - Be idempotent
- On success:
    - Balance is updated
    - Transaction event is published

---

## 6. Transfer Operation

- The system shall allow transfers between accounts.
- Transfer shall:
    - Validate source and target account existence
    - Validate sufficient balance
    - Validate minimum balance
    - Validate daily transaction limit
    - Perform atomic debit-credit within Account Service
    - Be idempotent
    - Use optimistic locking
- On success:
    - Publish TransactionCompletedEvent

---

## 7. Transaction History

- The system shall store transaction history in Transaction Service.
- Transaction record shall contain:
    - Transaction ID
    - Type (DEPOSIT / WITHDRAW / TRANSFER)
    - Amount
    - Source Account
    - Target Account (if applicable)
    - Timestamp
    - Status (SUCCESS / FAILED)
- Customers may view their own transaction history.
- Employees may view any transaction history.

---

## 8. Policy Enforcement

- The system shall enforce:
    - Minimum account balance
    - Daily transaction limit
    - Service charge rules
- Policy thresholds must be configurable.
- Policy validation must occur before balance mutation.

---

## 9. Audit Logging

- Every financial operation must generate an audit event.
- Audit log must include:
    - User ID
    - Operation type
    - Timestamp
    - Result (SUCCESS / FAILURE)
    - IP address (if available)

---

## 10. Fraud Detection (Basic)

- The system shall flag suspicious activity such as:
    - Multiple large transfers within short duration
    - Transactions exceeding configured threshold
- Fraud events must be stored separately.
- Fraud alerts must not block transaction completion in initial version.

---

## 11. Event Publishing

- Account Service shall publish events after successful commit.
- Events must be consumed by:
    - Transaction Service
    - Audit Service
    - Notification Service
    - Fraud Detection Service
- Kafka shall guarantee at-least-once delivery.

---

## 12. Idempotency

- All financial operations must require an Idempotency-Key header.
- Duplicate requests with same key must return original response.
- Idempotency data must be stored and validated.

---

## 13. Concurrency Control

- Account entity must use optimistic locking via version field.
- Concurrent modification must result in retry or failure.
- System must prevent negative balance under concurrent load.