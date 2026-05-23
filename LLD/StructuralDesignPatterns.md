# Structural Design Patters

**Structural patterns** — Adapter, Decorator, and Facade — help you organize and evolve large systems by controlling interfaces, extending behavior, and simplifying interactions; use them deliberately to reduce coupling, improve testability, and hide complexity, while balancing performance, clarity, and operational costs.

## What are they:

- **Adapter** changes an interface so two pieces work together.

- **Decorator** adds responsibilities to objects at runtime without changing their class.

- **Facade** provides a simple unified interface over a complex subsystem.

## Used Boardly In

- **Adapters** when integrating third-party services or migrating APIs.

- **Decorators** for adding features like logging, auth, or metrics to requests.

- **Facades** when you need an easy entry-point for clients (SDKs, service gateways) that hides many moving parts.

---

## Adapter Pattern

- **Purpose**: convert one interface to another so components interoperate without changing their internals.

- **Common real-world uses**: API gateway adapters that normalize upstream/downstream contract differences; database driver adapters to swap DB vendors; compatibility layers during rolling upgrades or service splits.

- **Scalable-system considerations**: add statelessness to adapters where possible; place adapters at the edge (API layer or sidecar) to limit blast radius and scale independently. Added latency per adapter hop is a measurable cost—benchmark and avoid deep adapter chains.

- **Challenges & trade-offs**: performance overhead, complexity when adapters accumulate, and duplicated transformation logic. Use when you need decoupling and gradual migration; avoid if you can change both sides at once. Best practice: keep adapters thin, idempotent, and observable (metrics/tracing).

---

## Decorator Pattern

- **Purpose**: attach extra behavior to objects/requests without subclassing, usually composed at runtime.

- **Common real-world uses**: middleware stacks (auth, rate limit, instrumentation), request/response enrichers, protocol features added per-tenant. In distributed systems, decorators often appear as middleware, filters, or sidecars.

- **Scalable-system considerations**: implement decorators as lightweight, composable stateless functions so they can be horizontally scaled and placed in the request path or in sidecars. Watch ordering: decorator order often changes semantics (auth before business logic, metrics around everything).

- **Challenges & trade-offs**: increased request path latency and harder debugging when many decorators wrap one another; testing combinations explosion. Use when cross-cutting concerns must be orthogonal to business code. Best practice: keep decorators small, well-documented, and provide a way to enable/disable via config and feature flags. Add traces and structured logs per decorator.

---

## Facade Pattern

- **Purpose**: provide a simplified interface to a complex subsystem so clients have a single easy-to-use entrypoint.

- **Common real-world uses**: backend-for-frontend (BFF) providing tailored APIs per client; SDKs exposing a small API surface; orchestration services that hide multiple downstream calls behind one API.

- **Scalable-system considerations**: the facade can become a scaling and availability bottleneck—design it to be stateless and horizontally scalable, or decompose facades per client (BFF) to avoid one-size-fits-all overload. Cache within the facade for heavy-read operations but be mindful of cache invalidation.

- **Challenges & trade-offs**: facades can hide too much and cause leaky abstractions (clients unaware of costs or semantics), evolve into god-services, or centralize logic that should live near data. Use when you need client simplicity and to centralize orchestration. Best practice: keep facade responsibilities limited, document SLAs, and push complexity back to downstream teams when appropriate.

---

## Operational implications

- **Where to place them**: edge (API gateway, BFF), sidecar (service mesh), or library level (in-process middleware) depending on visibility and latency needs. Choose sidecars/adapters for language-agnostic cross-cutting concerns; in-process decorators for minimal hop overhead.

- **Observability and testing**: always add metrics, traces, and structured logs around adapter/decorator/facade boundaries; provide unit tests for small pieces and integration tests for composition. Chaos-test heavy facades and adapters that coordinate multiple services.

- **Performance**: every added layer risks latency; measure P50/P95/P99 and optimise hot paths. Prefer composition over deep inheritance; use caching, bulk/async calls, and circuit breakers where facades coordinate remote calls.

- **Evolution & maintainability**: use adapters for gradual migration, decorators for orthogonal concerns, and facades to provide stable client contracts. Keep patterns explicit in architecture docs so teams share the same mental model.

---

## Interview-focused questions (what an interviewer might ask)

**High-level understanding questions**:

- “Explain Adapter vs Facade vs Decorator and give a real example for each.”

- “When would you place an adapter as a sidecar versus in-process?”

**System/scale questions**:

- “Design a BFF for mobile and web that reduces N+1 calls—how would you use a Facade pattern and what trade-offs exist?”

- “You’re integrating a legacy payments provider with a new billing service—how do adapters help with a zero-downtime migration?”

**Operational and safety questions**:

- “How do you instrument and fail-safe multiple decorators in the request path?”

- “How would you prevent the facade from becoming a single point of failure?”

**Depth / follow-ups an interviewer might use**:

- “Show how you’d test composed decorators and detect configuration-induced behavior changes.”

- “How would you evolve a facade when downstream services change contracts?”

---

## Quick rubric to answer interview prompts (short checklist)

- **Purpose**: state the intent (convert interface, add behavior, simplify subsystem).

- **Placement**: in-process vs sidecar vs gateway (trade latency vs language-agnosticism).

- **Scaling**: statelessness, horizontal scaling, caching, circuit breakers.

- **Observability & testing**: metrics/traces, unit + integration + chaos tests.

- **Trade-offs**: extra latency, operational complexity, potential god-service, migration cost.

---

## Example mini-case (illustration)

**Problem**: Mobile app needs aggregated product info from Pricing, Inventory, and Reviews with low latency.

**Solution sketch**: build a Facade (BFF) that orchestrates parallel calls and caches results; use Decorators for auth, rate-limit, and tracing; use Adapters to normalize data from legacy Inventory and a new Inventory microservice during migration. Make the facade stateless, instrument P95 latency, and add circuit breakers around slow downstreams.
