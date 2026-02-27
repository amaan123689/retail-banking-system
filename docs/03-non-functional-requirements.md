# Retail Banking System – Non-Functional Requirements

## 1. Availability

- The system shall be designed for high availability.
- Failure of non-critical services (e.g., Notification Service) must not block financial transactions.
- Core financial operations (Account Service) must prioritize consistency over availability.

---

## 2. Scalability

- Each microservice shall be horizontally scalable.
- Services must be stateless (except database layer).
- Load balancing shall be supported via API Gateway.
- Kafka consumers must support scaling through partitioning.

---

## 3. Performance

- Average API response time for synchronous operations shall be < 300ms under normal load.
- Transaction commit time shall remain under acceptable latency for financial systems.
- Event publishing must not significantly delay response to client.

---

## 4. Consistency Model

- Account balance updates must follow strong consistency.
- Cross-service communication (e.g., audit logging) may follow eventual consistency.
- No distributed two-phase commit shall be implemented.
- Balance mutation shall occur within a single ACID database transaction.

---

## 5. Concurrency Safety

- Account Service must implement optimistic locking using versioning.
- Concurrent balance updates must not result in negative balances.
- System must handle retry logic for optimistic locking failures.

---

## 6. Security

- All APIs must require authentication except registration.
- JWT-based authentication shall be implemented.
- Access control must be role-based.
- Passwords must be securely hashed using BCrypt.
- Sensitive data must not be logged.
- HTTPS must be enforced in production deployment.

---

## 7. Idempotency

- Financial operations must support idempotent execution.
- Duplicate requests must not cause duplicate debits or credits.
- Idempotency keys must be stored with expiration policy.

---

## 8. Observability

- Structured logging must be implemented.
- Logs must include correlation IDs for tracing.
- Each service must expose health endpoints.
- Metrics collection must be supported (Micrometer-compatible).
- Distributed tracing support shall be considered for future extension.

---

## 9. Reliability & Resilience

- Retry mechanism must be implemented for transient failures.
- Kafka consumers must support Dead Letter Topics (DLT).
- Circuit breaker must be implemented for inter-service REST calls.
- System must degrade gracefully when non-critical services fail.

---

## 10. Data Integrity

- All monetary values must be stored in smallest currency unit (e.g., cents).
- Database constraints must enforce:
    - Non-null critical fields
    - Unique account numbers
    - Positive monetary values
- Balance must never be derived from transaction sum in runtime.

---

## 11. Auditability

- Every financial operation must produce an audit trail.
- Audit records must be immutable.
- Audit logs must be tamper-resistant.

---

## 12. Deployment

- All services must be containerized using Docker.
- System must be deployable via Docker Compose for local development.
- Configuration must be externalized via Config Server.

---

## 13. Maintainability

- Code must follow clean architecture principles.
- Each microservice must have:
    - Clear controller layer
    - Service layer
    - Repository layer
    - Exception handling strategy
- API contracts must be documented using OpenAPI.

---

## 14. Extensibility

- Event-driven architecture must allow adding new consumers without modifying Account Service.
- Policy rules must be configurable without code changes where possible.
- Future microservices must integrate through events rather than tight coupling.