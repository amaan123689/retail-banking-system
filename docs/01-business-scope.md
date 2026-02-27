# Retail Banking System – Business Scope

## 1. Purpose of the System

The Retail Banking System is a distributed microservices-based backend platform designed to support core retail banking operations such as account management, money transfers, transaction tracking, policy enforcement, and fraud detection.

The system aims to simulate an enterprise-grade banking backend with proper transactional safety, role-based access control, and event-driven extensibility.

---

## 2. Objectives

- Provide secure banking operations for customers and bank employees.
- Ensure transactional consistency for balance updates.
- Prevent double-spending and concurrency issues.
- Support scalable and decoupled microservice architecture.
- Enable auditability and observability of financial operations.
- Demonstrate production-grade backend engineering practices.

---

## 3. Actors

### 3.1 Customer
- Register and authenticate
- View profile information
- View accounts
- Deposit funds
- Withdraw funds
- Transfer money to another account
- View transaction history

### 3.2 Bank Employee
- Create and manage customer profiles
- Open and manage customer accounts
- Freeze or reactivate accounts
- View customer transaction summaries
- Enforce operational policies

### 3.3 Admin
- Configure policy rules (minimum balance, transaction limits)
- Manage employee roles
- View system-wide audit logs
- Monitor fraud alerts

### 3.4 System
- Publish domain events
- Enforce transaction safety
- Maintain audit trails
- Trigger fraud detection logic

---

## 4. In-Scope Features

- Role-based authentication and authorization (JWT-based)
- Customer management
- Account creation and lifecycle management
- Balance storage and updates
- Deposit, withdrawal, and transfer operations
- Optimistic locking for concurrency control
- Idempotent transaction handling
- Daily transaction limit enforcement
- Minimum balance enforcement
- Transaction history storage
- Audit logging via event-driven architecture
- Notification triggers (e.g., transaction alerts)
- Fraud detection (basic pattern-based rules)
- Dockerized deployment using Docker Compose

---

## 5. Out-of-Scope Features

- Real-world payment gateway integrations
- External banking network integration (e.g., SWIFT, UPI)
- Real KYC document verification APIs
- Multi-currency support
- High-frequency trading capabilities
- Advanced ML-based fraud detection
- Production cloud hardening and DevOps pipelines

---

## 6. Assumptions

- Each microservice owns its own database.
- All monetary values are stored in smallest currency unit (e.g., cents).
- Account Service is the single authority for balance mutation.
- Events are published only after successful database commit.
- Kafka guarantees at-least-once delivery.
- Retry mechanisms are implemented for transient failures.

---

## 7. Constraints

- System follows hybrid architecture (REST + Kafka).
- No distributed two-phase commit across services.
- Horizontal scalability is supported via containerization.
- All sensitive operations require authenticated access.
- Database consistency is prioritized over eventual consistency for balance updates.

---

## 8. Design Philosophy

This system intentionally evolves from a demo-level microservices banking application into an enterprise-oriented architecture.

The focus is on:

- Clear service boundaries
- Transactional integrity
- Concurrency safety
- Idempotency
- Observability
- Resilience

The architecture prioritizes correctness and clarity over premature optimization.