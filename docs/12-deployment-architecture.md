# Retail Banking System – Deployment Architecture

## 1. Deployment Philosophy

The system is containerized using Docker.

All services are orchestrated locally using Docker Compose.

Objectives:

- Reproducible environment
- Isolated service deployment
- Simplified local setup
- Environment parity between development and staging

---

## 2. Containerized Services

The following services are containerized:

Infrastructure:
- PostgreSQL (separate DB per service)
- Kafka
- Zookeeper
- Service Registry (Eureka)
- Config Server
- API Gateway

Core Services:
- Authentication Service
- Customer Service
- Account Service
- Transaction Service
- Policy Service

Supporting Services:
- Audit Service
- Notification Service
- Fraud Detection Service
- Reporting Service

---

## 3. Docker Networking

All containers run inside a shared Docker network.

Characteristics:

- Services communicate using service names (e.g., account-service:8080)
- No external DB exposure required for internal communication
- Only API Gateway exposed to host machine

---

## 4. Port Exposure Strategy

Only the following ports are exposed externally:

- API Gateway (e.g., 8080)
- Kafka (if required for local debugging)
- Eureka Dashboard (optional)
- Config Server (optional)

Databases are not exposed publicly unless needed for debugging.

---

## 5. Configuration Management

Configuration is externalized via Config Server.

Each service reads configuration from:

- application.yml (default)
- profile-specific configuration
- environment variables

Sensitive data (e.g., DB passwords) must be injected via environment variables.

---

## 6. Service Startup Order

Startup dependency order:

1. Zookeeper
2. Kafka
3. PostgreSQL instances
4. Service Registry
5. Config Server
6. Core Microservices
7. Supporting Microservices
8. API Gateway

Docker Compose `depends_on` will manage basic ordering.

---

## 7. Environment Variables

Each service requires:

- DATABASE_URL
- DATABASE_USERNAME
- DATABASE_PASSWORD
- KAFKA_BOOTSTRAP_SERVERS
- EUREKA_SERVER_URL
- JWT_SECRET

No secrets hardcoded in source code.

---

## 8. Horizontal Scaling

Docker Compose allows:

- Scaling services using `--scale` flag.
- Kafka partitions allow consumer parallelism.
- API Gateway supports routing to multiple instances.

Future extension:
Migration to Kubernetes for production-grade scaling.

---

## 9. Health Monitoring

Each service must expose:

- /actuator/health
- /actuator/info

Docker health checks can be configured to:

- Restart unhealthy containers
- Prevent dependent services from starting prematurely

---

## 10. Logging Strategy

Each container logs to stdout.

Docker aggregates logs.

Future extension:
- Centralized logging (ELK stack or OpenSearch)

---

## 11. Local Development Flow

Developer workflow:

1. Clone repository.
2. Run `docker-compose up --build`.
3. Wait for all services to register.
4. Access system via API Gateway.
5. Swagger UI available per service.

This ensures one-command system startup.

---

## 12. Deployment Scope

This project simulates enterprise deployment patterns.

However, it does not include:

- CI/CD pipeline automation
- Production Kubernetes cluster setup
- Cloud load balancer configuration

These are future enhancements.