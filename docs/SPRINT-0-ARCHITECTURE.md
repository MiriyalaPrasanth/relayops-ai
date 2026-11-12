# Sprint 0 Architecture Blueprint

## 1. Product purpose

RelayOps AI is a multi-tenant event-operations platform. It receives events from external systems, accepts valid events durably, processes them outside the inbound request path, and makes event status and recommended actions visible to an operations team.

It is a learning project, but every design decision should be credible for a production SaaS platform.

## 2. Primary users and use cases

### Platform integrator

Registers a source system and sends signed events to RelayOps.

### Operations engineer

Views event status, failures, retries, related incidents, and recommended next actions.

### Tenant administrator

Manages organization membership, roles, event sources, and notification destinations.

### Initial use cases

1. **Receive deployment failure:** a source posts a signed `deployment.failed` event.
2. **Prevent duplicate work:** the source retries the same event after a network timeout; RelayOps recognizes the idempotency key and does not notify twice.
3. **Absorb a burst:** an incident produces thousands of events per second; the API acknowledges accepted events quickly while background work accumulates safely.
4. **Recover from a downstream failure:** notification delivery fails; RelayOps retries with backoff and makes failed work available for investigation and replay.
5. **Assist incident response:** a later AI feature summarizes the event and retrieves only approved runbook material.

## 3. Scope boundaries

### In scope

- Public REST API versioning starting at `/api/v1`.
- Webhook-style event ingestion, request validation, authentication, and signature verification.
- Durable event records and idempotency.
- Asynchronous processing, retry, dead-letter, and replay patterns.
- Tenant-aware authorization, metrics, logs, traces, tests, containers, Kubernetes, CI/CD, and a controlled AI assistant.

### Explicitly out of scope for the first release

- Payment processing, inventory, or customer-facing e-commerce.
- Automatic production remediation by AI.
- Custom password storage or a custom identity provider.
- Multi-region active-active deployment.
- Microservices before an actual scaling or ownership reason exists.

## 4. Baseline architecture and planned evolution

```text
External provider
    |
    | signed HTTP event
    v
Ingress API (Spring Boot)
    |-- validate schema, tenant, signature, idempotency key
    |-- persist accepted event
    |-- return acknowledgement
    v
PostgreSQL event store
    |
    | later: outbox / message queue
    v
Processing worker
    |-- classify, retry, update status
    |-- notify destination
    v
Operations API and dashboard
```

Sprint 1 implements only the ingress API shape and health endpoint. Sprint 2 makes data durable. Sprint 4 adds a queue and independently scalable workers. Redis, Kubernetes, AI, and cloud services appear only in their scheduled sprints.

## 5. Target AWS deployment architecture

AWS deployment is part of the project, but it is not the first implementation step. The local implementation must keep clear boundaries so the same application can move to AWS without a redesign.

```text
Internet
   |
Route 53 + AWS WAF + Application Load Balancer
   |
Amazon EKS ingress
   |
RelayOps API pods <--> worker pods
   |                       |
Amazon RDS PostgreSQL   Amazon ElastiCache for Redis
   |                       |
Amazon S3              managed message broker or Kafka service
   |
CloudWatch + OpenTelemetry / Prometheus / Grafana
```

Planned AWS responsibilities:

- **EKS:** run API and worker containers with autoscaling and rolling deployments.
- **RDS PostgreSQL:** durable event data, backups, replication, and managed patching.
- **ElastiCache Redis:** shared caching, rate-limit counters, and short-lived coordination.
- **S3:** large event payloads, exports, model artefacts, and audit-ready archives.
- **ECR:** container-image registry for CI/CD deployment.
- **IAM and Secrets Manager:** least-privilege machine access and secrets storage.
- **Application Load Balancer and WAF:** TLS termination, routing, and edge protection.
- **CloudWatch:** AWS infrastructure logs, metrics, alarms, and cost visibility.

Terraform will define this infrastructure in Sprint 12. AWS cost, IAM boundaries, networking, backups, and disaster recovery will be treated as design requirements rather than afterthoughts.

## 6. Event lifecycle

```text
RECEIVED -> VALIDATED -> ACCEPTED -> QUEUED -> PROCESSING -> COMPLETED
                                      |                         |
                                      v                         v
                                   REJECTED                  RETRYING
                                                                  |
                                                                  v
                                                           DEAD_LETTERED
```

- `RECEIVED`: request reached the API; it is not yet trusted.
- `VALIDATED`: payload, schema, tenant, and signature are valid.
- `ACCEPTED`: the event is durably stored; only now may the API return a successful acknowledgement.
- `QUEUED` and later states are asynchronous processing concerns.
- `DEAD_LETTERED` means automatic attempts stopped; a human can inspect and replay the work.

## 7. Quality targets and measurable signals

These are learning-project targets, not production guarantees.

| Area | Initial target | Signal |
| --- | --- | --- |
| Ingestion volume | 1,000,000 events/hour average | accepted events/second |
| Burst handling | 2,000 events/second target | queue depth and rejection rate |
| Ingestion latency | p95 below 200 ms after durable acceptance | request-duration histogram |
| Durability | no silently lost acknowledged event | accepted-versus-completed reconciliation |
| Availability learning target | 99.9% monthly | successful acknowledgement ratio |
| Tenant isolation | zero cross-tenant reads/writes | authorization tests and audit logs |

## 8. Failure scenarios we will deliberately reproduce

| Failure | Why a basic application fails | Planned solution |
| --- | --- | --- |
| Provider retries a request | one event is processed twice | unique idempotency key and durable status |
| Notification endpoint times out | request thread blocks or event disappears | background worker, timeout, retry, dead-letter queue |
| Traffic spike | API threads and database connections saturate | short ingress path, queue, rate limit, autoscaling |
| Deploy has a bug | healthy traffic reaches an unhealthy release | health probes, CI/CD checks, rollback strategy |
| Cache is stale | user sees an outdated event state | explicit TTL and invalidation rules |
| AI answer is incorrect | operator could take unsafe action | source citations, evaluation, human approval, no direct remediation |

## 9. Security model

- Human users authenticate through an existing identity provider using OIDC/OAuth 2.0.
- APIs validate issuer, audience, expiry, scopes/roles, and tenant claims; decoding a JWT alone is insufficient.
- Machine-to-machine access uses restricted service credentials.
- External webhook sources prove origin through provider-specific signatures or shared secrets.
- Secrets never enter Git; local values use an ignored `.env` file and production values use a secret manager.
- Every database record is tenant scoped and every query must enforce that scope.

## 10. Initial source layout

```text
src/main/java/com/relayops/
  ingress/        HTTP controllers and request DTOs
  events/         event lifecycle and application services
  identity/       authentication, authorization, and tenant context
  processing/     worker and retry behavior
  notification/   notification ports and adapters
  shared/         errors, validation, configuration, observability
```

## 11. Sprint 0 completion criteria

- The product purpose, users, boundaries, event lifecycle, and production targets are understood.
- The reason for durable acceptance before acknowledgement is understood.
- The distinction between synchronous ingress and asynchronous processing is understood.
- The learner passes the Sprint 0 quiz with at least 9 correct answers out of 10.

