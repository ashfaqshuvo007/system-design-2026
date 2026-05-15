# Reliability Patterns - Circuit breaker, retry with backoff, and bulkhead

These three patterns are not “nice-to-have” extras; they are control mechanisms to stop one failure from becoming a system-wide outage.

At a high level, retry with backoff helps when a dependency is temporarily unhealthy, circuit breaker stops repeated calls to a failing dependency, and bulkhead isolates failures so one overloaded part does not sink everything.

## What are these:

Think of a distributed system like a building with many doors and hallways. When one door is stuck, you do not keep pushing it forever; when one hallway is crowded, you do not let the crowd block every other hallway.

- Retry with backoff: Try again after a short wait, and increase the wait each time. This is useful for transient issues like brief network hiccups or a service that is starting up.

- Circuit breaker: If a dependency keeps failing, stop calling it for a while and fail fast. That prevents wasting resources and reduces cascading failures.

- Bulkhead: Split resources into separate pools so one failing area does not consume all threads, connections, or memory.

**A simple example**: if a payment service calls a third-party fraud API and that API times out, retrying once or twice may help; if it keeps timing out, the circuit breaker should open; and if the fraud API uses a dedicated connection pool, that is bulkhead isolation.

## Where each fit in the failure chain

As a technically sound engineer, the key is not just knowing the definitions, but knowing where each pattern belongs in the failure chain. Retries are for recovery from transient faults, circuit breakers are for limiting damage from persistent faults, and bulkheads are for containing blast radius through resource isolation.

**Retry with backoff**

Use retries when the operation is safe to repeat, ideally when the request is idempotent or has an idempotency key. Without that, retries can create duplicate writes, duplicate charges, or inconsistent state. Add jitter to the backoff so many clients do not retry at the same time and amplify load spikes.

- **Main challenge**: retries can make a bad situation worse. Too many retries increase latency, consume thread pools, and can overload the downstream service even further.

**Circuit breaker**

Use a circuit breaker when repeated failures indicate a dependency is unhealthy and you want fast failure instead of hanging requests. A typical state machine is closed, open, and half-open, where half-open allows limited test traffic to see whether the dependency recovered.

- **Main challenge**: choosing thresholds. If it opens too early, you reject traffic unnecessarily; if it opens too late, you still burn resources and worsen the outage. It also needs good observability, because otherwise you only notice it through user complaints.

**Bulkhead**

Use bulkheads when different request classes or dependencies compete for the same shared resource pool. Common examples are separate thread pools, separate connection pools, separate queues, or service-level isolation between critical and noncritical work.

- **Main challenge**: partitioning resources well. Too much isolation wastes capacity; too little isolation means one hot path can starve everything else. Bulkheads are especially useful in multi-tenant systems or systems with mixed workloads.

## Trade-offs

These patterns solve different problems, and they also interact.

| Pattern            | Best for                      | Main risk                                            | When to avoid                                                                                        |
| ------------------ | ----------------------------- | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| Retry with backoff | Transient failures            | Load amplification, duplicate side effects           | Non-idempotent operations without protection system-design-learning-hub.vercel                       |
| Circuit breaker    | Persistent downstream failure | False opens, delayed recovery if thresholds are poor | Very low-latency paths where fail-fast is harmful without fallback system-design-learning-hub.vercel |
| Bulkhead           | Blast-radius reduction        | Underutilized capacity, more operational complexity  | Small systems where the overhead is not justified system-design-learning-hub.vercelyoutube           |

**A good rule**: do not use retries as your first line of defense against a sick dependency. Retries should be bounded, deliberate, and paired with timeouts, idempotency, and backoff. If the dependency is clearly unhealthy, circuit breaking is usually the safer move.

## Real-world implications

In production, these patterns affect customer experience directly.

**Retries** can hide brief blips and improve success rate, but they also increase tail latency if you retry too aggressively.

**Circuit breakers** protect the caller and help the system recover, but they can temporarily reduce availability for a dependency that may already be healing.

**Bulkheads** are often invisible when everything is healthy, which is exactly why they matter. When a noncritical batch job shares resources with user-facing traffic, bulkheads keep a background spike from causing an outage in the critical path. This is why strong teams design for failure isolation, not just failure recovery.

## What Interviewers Look For

In a system design interview, a strong candidate should be able to explain not just the pattern, but the operational reasoning behind it. Interviewers usually want to see whether you can connect failure mode, mitigation, and trade-off.

**Possible questions**:

- When would you retry, and when would you fail fast?

- How do you prevent retries from causing duplicate writes?

- What should trigger a circuit breaker to open?

- What does half-open mode do, and why is it needed?

- How would you bulkhead a system with both user traffic and batch processing?

- How do timeouts, retries, and circuit breakers work together?

- What metrics would you monitor to know these protections are helping?

A strong answer mentions idempotency, bounded retries, jitter, timeout selection, fallback behavior, and resource isolation.

**A mid-level engineer should be able to say**: use retry with backoff for transient, safe-to-repeat failures; use a circuit breaker for repeated downstream failure to avoid cascading damage; use bulkheads to isolate critical paths from noisy neighbors. You should also explain that these are not substitutes for good timeouts, monitoring, and graceful degradation.

**The shortest interview-ready version, memorize this line**: “Retry helps recovery, circuit breaker limits damage, bulkhead limits blast radius”.
