# Retail Banking System – API Contracts

## 1. API Design Principles

- All APIs are versioned using URI versioning: /api/v1/
- All requests and responses use JSON format.
- All secured endpoints require Authorization: Bearer <access_token>
- Financial operations require Idempotency-Key header.
- All responses include:
    - requestId (correlation ID)
    - timestamp
    - status
    - data or error

---

# 2. Authentication Service APIs

## 2.1 Register User

POST /api/v1/auth/register

Request:
{
"email": "user@example.com",
"password": "password123",
"role": "CUSTOMER"
}

Response:
{
"requestId": "uuid",
"status": "SUCCESS",
"data": {
"userId": "uuid"
}
}

---

## 2.2 Login

POST /api/v1/auth/login

Request:
{
"email": "user@example.com",
"password": "password123"
}

Response:
{
"requestId": "uuid",
"status": "SUCCESS",
"data": {
"accessToken": "jwt-token",
"refreshToken": "refresh-token"
}
}

---

# 3. Customer Service APIs

## 3.1 Create Customer (EMPLOYEE only)

POST /api/v1/customers

Request:
{
"fullName": "John Doe",
"phone": "9999999999",
"address": "Mumbai"
}

Response:
{
"status": "SUCCESS",
"data": {
"customerId": "uuid"
}
}

---

## 3.2 Get Customer Details

GET /api/v1/customers/{customerId}

---

# 4. Account Service APIs

## 4.1 Create Account (EMPLOYEE only)

POST /api/v1/accounts

Request:
{
"customerId": "uuid",
"accountType": "SAVINGS",
"minimumBalance": 10000,
"dailyLimit": 500000
}

Response:
{
"status": "SUCCESS",
"data": {
"accountNumber": "ACC123456"
}
}

---

## 4.2 Get Account Details

GET /api/v1/accounts/{accountNumber}

---

## 4.3 Deposit

POST /api/v1/accounts/{accountNumber}/deposit

Headers:
Idempotency-Key: unique-key

Request:
{
"amount": 10000
}

---

## 4.4 Withdraw

POST /api/v1/accounts/{accountNumber}/withdraw

Headers:
Idempotency-Key: unique-key

Request:
{
"amount": 5000
}

---

## 4.5 Transfer

POST /api/v1/accounts/transfer

Headers:
Idempotency-Key: unique-key

Request:
{
"sourceAccount": "ACC123",
"targetAccount": "ACC456",
"amount": 10000
}

---

# 5. Transaction Service APIs

## 5.1 Get Transaction History

GET /api/v1/transactions?accountNumber=ACC123&page=0&size=10

Response:
{
"status": "SUCCESS",
"data": [
{
"transactionId": "uuid",
"type": "TRANSFER",
"amount": 10000,
"timestamp": "2026-01-01T10:00:00"
}
]
}

---

# 6. Policy Service APIs

## 6.1 Get Active Rules

GET /api/v1/policies

---

## 6.2 Update Rule (ADMIN only)

PUT /api/v1/policies/{ruleId}

Request:
{
"ruleValue": 20000,
"isActive": true
}

---

# 7. Standard Error Response Format

{
"requestId": "uuid",
"status": "ERROR",
"error": {
"code": "INSUFFICIENT_BALANCE",
"message": "Account does not have sufficient balance"
}
}

---

# 8. HTTP Status Codes

- 200 OK – Successful operation
- 201 Created – Resource created
- 400 Bad Request – Validation failure
- 401 Unauthorized – Authentication required
- 403 Forbidden – Access denied
- 404 Not Found – Resource not found
- 409 Conflict – Optimistic locking failure
- 500 Internal Server Error – Unexpected failure