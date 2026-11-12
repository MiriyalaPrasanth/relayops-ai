# RelayOps AI

RelayOps AI is a production-oriented event and webhook processing platform. It accepts signed events from external providers, records them durably, processes them asynchronously, and will later enrich and route them with AI.

## Project status

**Sprint 0 — foundation and architecture in progress**. The application has not been implemented yet, so it is not deployable or runnable at this stage. This README will be updated in every sprint with exact local-run, test, container, and deployment commands.

## Why this project exists

The project is a hands-on path through backend architecture, distributed systems, cloud deployment, DevOps, security, observability, and AI/ML production concerns. We introduce infrastructure only after recreating the production problem that requires it.

## Technology direction

- Java and Spring Boot for the core application
- Maven for builds and dependency management
- PostgreSQL for durable application data
- Redis for caching and short-lived coordination, introduced when justified
- RabbitMQ for background event processing, then Kafka only if the later requirements justify it
- Docker Compose for local dependencies
- Kubernetes and Terraform for cloud deployment
- AWS as the target cloud: EKS, RDS for PostgreSQL, ElastiCache for Redis, S3, ECR, IAM, CloudWatch, and an Application Load Balancer
- A Python AI/ML service introduced in a later sprint where the ML ecosystem provides value

## Target production flow

```text
Webhook provider -> load balancer / gateway -> RelayOps API -> durable event store
                                                       -> queue -> worker -> notifications / AI
```

## Prerequisites

| Tool | Target version | Status on this machine |
| --- | --- | --- |
| Java JDK | 21 LTS | Java 17 is currently installed |
| Maven | 3.9+ | Installed (3.9.9) |
| Docker Desktop | Current, WSL 2 backend | Not currently available on PATH |
| Python | 3.11+ | Installed (3.14.0) |
| Git | Current | To be verified before the first commit |

Java 17 can run many Spring Boot applications, but the project target will be Java 21 LTS. Docker Desktop is required before we add PostgreSQL, Redis, RabbitMQ, or container-based local deployment.

## Documentation

- [Sprint 0 architecture blueprint](docs/SPRINT-0-ARCHITECTURE.md)
- [Sprint 0 learning log](docs/learning-log/SPRINT-0.md)
- [Architecture decision records](docs/adr/)

## Delivery standard

Every completed component must include:

1. A documented production problem and the design decision that addresses it.
2. Local setup and verification commands.
3. Tests and expected results.
4. Container and deployment instructions when applicable.
5. Known limitations, operational metrics, and rollback notes.

