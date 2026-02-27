# Retail Banking System – Security Architecture

## 1. Security Principles

- Zero trust between services.
- All external requests must pass through API Gateway.
- All APIs (except registration & login) require authentication.
- Financial operations require strict authorization checks.
- Sensitive data must never be logged.

---

# 2. Authentication Flow

## 2.1 Login Flow

1. Client sends email and password to Authentication Service.
2. Authentication Service:
    - Validates credentials.
    - Verifies password using BCrypt.
    - Generates:
        - Access Token (short-lived, e.g., 15 minutes)
        - Refresh Token (long-lived, e.g., 7 days)
3. Tokens are returned to client.

---

## 2.2 Token Structure

JWT Access Token contains:

- userId
- role
- issuedAt
- expiration
- tokenId

Token is signed using a secure secret key.

---

## 2.3 Token Validation

For each request:

1. Client sends:
   Authorization: Bearer <access_token>
2. API Gateway validates:
    - Signature
    - Expiry
3. Gateway forwards validated request to downstream services.

---

# 3. Role-Based Access Control (RBAC)

Roles:

- CUSTOMER
- EMPLOYEE
- ADMIN

Authorization rules:

CUSTOMER:
- Can access only own accounts.
- Can perform deposit, withdraw, transfer on own accounts.

EMPLOYEE:
- Can create customers.
- Can open accounts.
- Can freeze accounts.
- Can view customer transaction data.

ADMIN:
- Can modify policy rules.
- Can manage employee roles.
- Can view system-wide audit logs.

---

# 4. Ownership Validation

Even if role is CUSTOMER:

- Account Service must verify that account.customerId matches authenticated user.
- Direct account access without ownership validation is forbidden.

This prevents horizontal privilege escalation.

---

# 5. Refresh Token Flow

1. Client sends refresh token.
2. Authentication Service:
    - Validates token.
    - Generates new access token.
3. Old refresh tokens may be invalidated on logout.

---

# 6. Password Security

- Passwords stored using BCrypt hashing.
- No plain-text passwords stored.
- Password complexity rules enforced.

---

# 7. Secure Financial Operations

Financial operations require:

- Valid JWT token
- Valid role
- Ownership validation
- Idempotency-Key header

If any check fails → operation rejected.

---

# 8. API Gateway Responsibilities

- JWT validation
- Correlation ID injection
- Rate limiting (future)
- Logging request metadata
- Blocking unauthorized requests

Gateway does NOT contain business logic.

---

# 9. Service-to-Service Security

Internal service communication:

- Must include internal authentication mechanism (e.g., service token or mutual trust network).
- No open endpoints inside Docker network.
- Sensitive endpoints protected even internally.

---

# 10. Security Threat Mitigation

Mitigated threats:

- Brute force login (rate limiting future enhancement)
- Replay attack (idempotency key)
- Double-spend attack (optimistic locking)
- Horizontal privilege escalation (ownership validation)
- Token tampering (JWT signature verification)

---

# 11. Audit Logging for Security Events

Security events to be logged:

- Failed login attempts
- Unauthorized access attempts
- Role changes
- Account freeze operations