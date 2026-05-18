# Microservices Patterns

Microservices - also known as the microservice architecture - is an architectural style that structures an application as a collection of two or more services that are:

- Independently deployable
- Loosely coupled

Services are typically organized around business capabilities. Each service is often owned by a single, small team.

---

## Quick reference table (patterns & when‑to‑use)

| Pattern           | When to consider it                                                         | Main trade‑off                                              |
| ----------------- | --------------------------------------------------------------------------- | ----------------------------------------------------------- |
| Service discovery | Large dynamic microservices fleet (cloud, autoscaling, many replicas).      | Extra hop, more complexity, but highly decoupled.           |
| API gateway       | Many clients, many services, need auth/rate‑limiting/central routing.       | Latency, potential SPOF if not well‑designed.               |
| Saga pattern      | Business workflows that span multiple services, can’t share DBs.            | Complexity around compensations, idempotency, and ordering. |
| Sidecar pattern   | Need shared cross‑cutting concerns (logging, TLS, retries) across services. | More resource usage and debugging surface.                  |
| Strangler Fig     | Incremental migration from monolith to microservices.                       | Routing complexity and dual‑path testing.                   |

---

## 1. Service discovery & API gateway

### **Service discovery** answers: “Where is Service‑X running right now?”

- Each service registers itself (host, port, health) with a registry (e.g., Eureka, Consul, Kubernetes’ `Services`).
- Clients call the registry first, then talk to the correct instance.

### **API gateway** is a **single entrypoint** for clients.

- It routes `/users` → UserService, `/orders` → OrderService, etc.
- It can also do auth, rate limiting, logging, and TLS termination.

### **Why you care**:

- In a big product, you don’t want clients knowing IPs/ports of 50+ services; that couples them tightly.
- Dynamic scaling (Kubernetes, autoscaling) means IPs change all the time; service discovery handles that.

### **When to use**:

- **Service discovery** almost always with microservices in the cloud (Kubernetes, AWS, etc.).
- **API gateway** when you have multiple clients (mobile, web, third‑party) and want to centralize auth/routing/logging.

### **Trade‑offs**:

- Extra hop via gateway → small latency, but you gain control over routing, observability, and versioning.
- Gateway becomes a **single point of failure**; you must make it resilient (load‑balanced, HA, automated failover).

### **Interview questions you might see**:

- “How do services find each other in a microservice architecture?”
- “How would you route a `/profile` endpoint to the correct service?”
- “What happens if your API gateway goes down?”

---

## 2. Saga pattern for distributed transactions

- In a **monolith**, you can wrap everything in one DB transaction: update order → update inventory → commit all or rollback.
- In **microservices**, each service has its own DB, so you cannot have a single ACID transaction across services.

### **Saga pattern**

- replaces one big transaction with a **sequence of local transactions**, each with a **compensating action** if something fails.

Example (order placement):

1. OrderService creates `order_status=PENDING`.
2. If OrderService successfully creates the order, it sends an async event / command to InventoryService to reserve items.
3. If InventoryService fails, it sends a message back to OrderService, which **cancels** the order (compensating action).

You either:

- **Choreography**: Services talk directly via events (Event‑Driven architectures).
- **Orchestration**: A central Saga orchestrator invokes each service step‑by‑step and calls compensations.

### **Why Saga**:

- Required whenever you have **business logic spanning multiple services** (e.g., order + payment + inventory).
- You sacrifice atomicity but keep each service independent and loosely coupled.

### **Challenges**:

- Handling **partial failures** cleanly: what if the compensation itself fails?
- Making sure compensations are **idempotent** and **re‑entrant** (because retries and retries + retries).
- **Event ordering** and **duplication** in event‑driven sagas; you need message‑queue guarantees (at‑least‑once, dedup IDs).

### **When to choose**:

- Need strong consistency across services but cannot share DBs → Saga.
- If eventual consistency is OK, you may prefer **event‑driven architectures + async processing** instead of strict Saga flows.

### **Interview questions you might see**:

- “How would you design order placement across Order, Payment, and Inventory services?”
- “What happens if Inventory fails after the order is created?”
- “Compare Saga vs 2PC vs transactions in a monolith.”

---

## 3. Sidecar pattern

### **Sidecar** = a small helper container/process that runs **alongside** your main service, sharing its lifecycle.

- It’s like a **utility belt** that handles cross‑cutting concerns so the main service stays focused on business logic.

Examples:

- Logging sidecar: collects logs from your app container and ships them to a log backend.
- Monitoring sidecar: exposes metrics endpoints, pushes stats to Prometheus, etc.
- Sidecar proxy (e.g., Istio‑envoy): handles TLS, retries, circuit‑breaking, and observability.

### **Why you care**:

- In Kubernetes, you can add a sidecar to every pod without changing the main app’s code.
- Separation of concerns: the app doesn’t have to embed retries, TLS, etc.; the sidecar does.

### **Trade‑offs**:

- More processes = more memory, more inter‑process communication, more debugging surface.
- Risk of “over‑sidecaring”: too many sidecars per pod can make the deployment complex and hard to reason about.

### **When to use**:

- You want to standardize observability, security, or networking logic across many services (e.g., service mesh via Istio/Envoy sidecars).
- When you cannot or don’t want to bake these features into each language‑specific service.

### **Interview questions you might see**:

- “How would you add logging / TLS / retries for all services without changing each service?”
- “What is a sidecar in Kubernetes, and how does it relate to service mesh?”

---

## 4. Strangler Fig pattern (migration)

- You have a **large, old monolith** that you want to replace with microservices, but you cannot rewrite it all at once.

### **Strangler Fig** is an incremental refactoring:

- You **wrap** the monolith behind a gateway.
- You slowly **replace small pieces** of functionality with new microservices.
- Over time, the new services “strangle” the old monolith until it’s gone.

**Analogy**: like a fig tree that slowly grows around a host tree and takes over.

### **How it works in practice**:

- Gateway routes certain paths to the new microservice (e.g., `/v2/users/` → UserService), others to the old monolith.
- You gradually shift traffic, validate correctness and performance, then decommission parts of the monolith.

### **Why you care**:

- Lets you modernize without a **big‑bang rewrite** (which is risky and hard to get approval for).
- Business can keep shipping features while the underlying architecture upgrades.

### **Challenges**:

- **Routing complexity**: gateway must know which paths are new vs legacy.
- **Data consistency**: sometimes the new service must talk to the old DB or there’s a shared schema during migration.
- **Dual‑path testing**: you may have to test both old and new flows for the same user journeys.

### **When to use**:

- You’re migrating from a monolith to microservices and want **low‑risk, incremental** change.
- The team or org is not comfortable with a full rewrite.

### **Interview questions you might see**:

- “How would you migrate a legacy e‑commerce monolith to microservices?”
- “How do you avoid cutting over all at once?”
- “What role does the API gateway play during migration?”
