# API Design Best Practices

MAANG‑style best practices for API design (patterns, security, scalability, OWASP relevance), for engineers at any level, practical guidance that can be applied immediately. Each recommendation below is short, actionable, and backed by industry guidance.

## Learnable Items:

- **API style and patterns**:
  Prefer resource-based REST for CRUD-style services.
  Use gRPC for internal high-throughput RPCs.
  GraphQL when clients need flexible queries.

- **Naming and verbs**:
  Use nouns for resources (e.g., /orders), HTTP methods for actions (`GET`, `POST`, `PUT/PATCH`, `DELETE`).
  Keep URLs hierarchical and predictable.

- **Versioning**:
  Version early and explicitly (e.g., /v1/)
  Deprecate with headers or docs rather than breaking clients.

- **Idempotency**:
  Make safe retry possible — `GET/PUT/DELETE` idempotent.
  For `POST` operations that must be retried (e.g., payments), require an idempotency key.

- **Errors & contracts**:
  Return structured errors (code, message, details), consistent HTTP status codes , document them in an OpenAPI spec.
  Treat API contracts as first-class artifacts.

- **Authentication basics**:
  Use TLS everywhere and token-based auth (OAuth2 / Bearer tokens).
  Store secrets securely and rotate credentials.

---

## Applied Patterns, Trade-offs, and Checklist

### Design patterns to use

- **API Gateway + Microservices**:
  Use an API Gateway for routing, rate limiting, auth offload, and aggregation
  keep microservices focused by domain.

- **Backend-for-Frontend (BFF)**:
  Create BFFs per client-type (web, mobile) to tailor payloads and reduce chattiness.

- **CQRS + Event Sourcing (when needed)**:
  Separate read/write models for high-scale workloads; use event streams for eventual consistency and replays.
  Use only when complexity is justified.

- **Throttling & Circuit Breaker**:
  Apply client and route rate limits at the gateway.
  Use circuit breakers in service clients to avoid cascading failures.

### API surface design

- Model around use cases (not DB tables). Design endpoints for real workflows and keep them coarse enough to avoid chatty clients.

- **Pagination & filtering**: Prefer cursor pagination for large result sets; expose filter/sort via query params.

- **Pagination example**: /users?limit=50&cursor=abc123 — avoids offset skew under heavy writes.

### Observability and ops

- Logs, traces, metrics by endpoint and by user/client; correlate trace IDs across the call path (gateway → services → DB).

- Instrument latency, error rate, and saturation metrics.

- Health checks, canary deploys, and slow-roll releases for new endpoints.

### Contract-first workflow

Use OpenAPI/Swagger as source of truth: generate server stubs, client SDKs, and interactive docs; include schema validation in CI.

API governance: linting rules, common middleware, and PR checklists to enforce consistent naming, headers, auth, and pagination.

---

## Security Best Practices

Use OWASP API guidance: treat OWASP API Top 10 as mandatory checklist (BOLA, broken auth, SSRF, misconfig, excessive exposure). OWASP focuses on real API-specific risks and should guide design & reviews.

### Authentication & authorization

- Centralize auth: offload to a trusted authorization server or gateway (OAuth2 / OAuth 2.1 with PKCE for public clients). Validate tokens at gateway and re-check scopes/roles at service boundaries for sensitive actions.

- Prefer short-lived access tokens and use refresh tokens where appropriate; use Proof Key for Code Exchange (PKCE) for mobile/SPA.

### Least privilege & fine-grained checks

Implement object-level and property-level authorization checks server-side (avoid trusting client-supplied IDs or hidden fields). Enforce function-level roles for admin operations.

### Input validation & output encoding

Validate types, lengths, and allowed values at the gateway or service boundary; never return more properties than necessary (avoid excessive data exposure).

### Rate limiting, quotas, and abuse protection

Enforce per-client and per-route limits, apply progressive backoff and blocking for abusive clients; track usage for billing/alerts.

### Secrets, keys, and TLS

Enforce TLS, HSTS, and secure cookie attributes; use a secrets manager for keys and rotate them automatically. Never log secrets.

### Third-party/SSRF protections

Validate outbound URLs, restrict network egress with allow‑lists, and sanitize inputs used in request URIs to avoid SSRF.

### Inventory and configuration management

Maintain an API inventory (endpoints, versions, owners) to avoid exposed deprecated endpoints and misconfigurations. Automate scanning and configuration drift checks.

---

## Scalability and structuring an API system

### Layered architecture

- **Edge**: CDN + WAF for static/resource caching and basic DDoS/WAF protections.

- **Gateway**: Auth, rate-limit, routing, and protocol translations (REST ↔ gRPC).

- **Services**: Single responsibility, horizontally scalable, with autoscaling and resource limits.

### Data partitioning & caching

- Cache safe `GET` responses at CDN or edge and use cache keys including auth scope where needed; apply write-through or invalidation strategies for freshness.

- Shard or partition data by tenant/user for large datasets; choose replication vs. eventual consistency per data access patterns.

### Asynchronous workflows

Offload long-running work to background jobs/event buses; return 202 + location header for async processing. Use retries with idempotency.

### Performance tuning

Use gRPC for low-latency internal service-to-service calls; reduce payloads (compression, selective fields), and batch requests where it reduces round trips.

### How important OWASP standards are

OWASP API Top 10 is essential — treat it like a required checklist for any public or internal API because it documents the most common, high-impact risks and practical mitigations. Integrate those checks into design reviews, CI security gates, and penetration tests.

---

## Quick practical checklist

- OpenAPI spec updated and validated.

- Auth handled at gateway; token validated and scopes checked.

- Object-level and property-level authorization tests present.

- Idempotency for side-effectful endpoints (payments, orders).

- Rate limits configured and circuit-breaker patterns for downstream calls.

- Structured error codes and sample responses documented.

- Logging/tracing added and no secrets in logs.

- OWASP Top 10 checklist passed for the endpoint.

---

## Example (short)

Endpoint: POST /v1/orders

- Gateway validates TLS & bearer token, checks rate limit, and forwards.

- Service requires idempotency-key header; service validates payload schema from OpenAPI; performs object-level auth to ensure user owns the payment method; enqueues order processing (returns 202 + job id).

- Background worker picks job, calls payment gateway (with egress allow-list & retry policy), updates status, emits events for downstream services.
