# Retail Banking System – Architecture Overview

## 1. Architectural Style

The system follows a hybrid microservices architecture:

- Synchronous REST communication for core financial operations
- Asynchronous event-driven communication using Kafka for side effects

This approach ensures:

- Strong consistency for balance mutations
- Loose coupling between services
- Scalability and resilience
- Simplified transaction management

---

## 2. High-Level Architecture

Client
↓
API Gateway
↓
Core Domain Services
↓
Event Publishing (Kafka)
↓
Supporting Services (Consumers)

---

## 3. Microservices Overview

### 3.1 Infrastructure Services

- API Gateway (Spring Cloud Gateway)
- Service Registry (Eureka)
- Config Server
- Kafka Broker
- PostgreSQL (per service database)

---

### 3.2 Core Domain Services

Authentication Service
- Manages users, roles, tokens
- Issues JWT access & refresh tokens

Customer Service
- Manages customer profile data
- Handles KYC status reference

Account Service (Balance Authority)
- Owns account table and balance
- Performs deposit, withdrawal, transfer
- Implements optimistic locking
- Enforces daily limit and minimum balance
- Publishes domain events after commit

Transaction Service
- Stores transaction history
- Consumes transaction events

Policy Service
- Stores configurable rules
- Validates financial constraints before balance mutation

---

### 3.3 Supporting Services

Audit Service
- Consumes financial events
- Stores immutable audit records

Notification Service
- Sends alerts on transaction completion

Fraud Detection Service
- Monitors suspicious transaction patterns
- Generates fraud alerts

Reporting Service
- Aggregates transaction data for analytics

---

## 4. Communication Model

### 4.1 Synchronous Communication (REST)

Used for:
- Authentication
- Account operations
- Policy validation
- Customer operations

Critical operations must complete before response is returned.

---

### 4.2 Asynchronous Communication (Kafka)

Used for:
- Transaction recording
- Audit logging
- Notification triggering
- Fraud detection
- Reporting updates

Events are published only after successful database commit.

---

## 5. Data Ownership Principle

Each microservice owns its own database.

No service directly accesses another service's database.

All cross-service communication must happen via:

- REST APIs
- Kafka events

This ensures loose coupling and service autonomy.

---

## 6. Transaction Strategy

Balance mutation occurs only inside Account Service.

Within a single ACID transaction:
- Validation
- Balance update
- Version increment

After successful commit:
- Domain event is published

This avoids distributed two-phase commit.

---

## 7. Hybrid Architecture Rationale

A fully event-driven Saga-based approach was considered.

However, the hybrid approach was chosen because:

- Balance consistency must remain strongly consistent
- Simpler reasoning about financial safety
- Reduced operational complexity
- Easier debugging and maintainability
- More realistic for enterprise backend systems

---

## 8. Failure Isolation Strategy

- Failure of Notification Service does not affect Account Service.
- Failure of Fraud Service does not block transaction completion.
- Circuit breakers protect against cascading REST failures.
- Kafka ensures reliable event delivery.

---

## 9. Deployment Architecture (Local)

All services are containerized using Docker.

Docker Compose orchestrates:
- All microservices
- Kafka & Zookeeper
- PostgreSQL instances
- Service Registry
- Config Server
- API Gateway

Services communicate over internal Docker network.