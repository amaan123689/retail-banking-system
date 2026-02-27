# Retail Banking System – Resilience Strategy

## 1. Resilience Philosophy

The system must remain operational even when:

- Supporting services fail
- Kafka consumers crash
- Network interruptions occur
- Downstream services are temporarily unavailable

Core financial correctness must never be compromised.

---

# 2. Failure Classification

Failures are categorized as:

1. Critical Failures
    - Account Service database failure
    - Balance mutation failure

2. Non-Critical Failures
    - Notification failure
    - Audit logging failure
    - Fraud detection failure
    - Reporting failure

Critical failures must block the transaction.

Non-critical failures must not affect financial operations.

---

# 3. Circuit Breaker Strategy

REST calls between services use Circuit Breaker (Resilience4j).

Behavior:

- If repeated failures occur:
    - Circuit opens
    - Requests fail fast
- After cooldown:
    - Half-open state allows test requests

This prevents cascading failure across services.

---

# 4. Retry Strategy

### 4.1 REST Retry

Transient failures:
- Retry limited number of times
- Exponential backoff

Must NOT retry:
- Validation failures
- Business logic errors

---

### 4.2 Kafka Consumer Retry

Consumers implement:

- Retry attempts (configurable)
- Backoff strategy

If retries exhausted:
- Event published to Dead Letter Topic (DLT)

---

# 5. Dead Letter Topic (DLT)

Purpose:

- Capture permanently failed event processing
- Prevent infinite retry loops
- Enable manual inspection and recovery

DLT records must include:
- Original event payload
- Error message
- Retry count

---

# 6. Graceful Degradation

If Notification Service is down:

- Transaction still succeeds
- Notification logged for retry

If Fraud Service is down:

- Transaction still succeeds
- Fraud evaluation deferred

If Reporting Service is down:

- Transaction unaffected

Only Account Service failure blocks transaction.

---

# 7. Timeout Configuration

All REST calls must define timeouts:

- Connection timeout
- Read timeout

Prevents indefinite blocking threads.

---

# 8. Health Checks

Each service exposes:

- /actuator/health

Health indicators include:
- Database connectivity
- Kafka connectivity

Unhealthy services can be restarted automatically.

---

# 9. Bulkheading (Future Enhancement)

Future extension:

- Isolate thread pools per external dependency
- Prevent thread starvation

---

# 10. Idempotent Design as Resilience

Idempotency helps recover from:

- Network retries
- Duplicate message delivery
- Client re-submission

It ensures safe reprocessing.

---

# 11. Logging & Observability

On failure:

- Structured logs must include correlation ID
- Stack traces logged internally
- No sensitive data exposed

Future enhancement:
- Centralized log aggregation
- Alerting on failure thresholds

---

# 12. System Recovery Model

If a service crashes:

- Docker restarts container
- Kafka re-delivers unacknowledged messages
- Idempotent consumers prevent duplication
- System resumes without data corruption