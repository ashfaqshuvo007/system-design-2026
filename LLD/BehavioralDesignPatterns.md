# Behavioral design patterns

Blueprint for how objects communicate and responsibilities are distributed. Unlike structural patterns (how things are composed) or creational patterns (how things are created), behavioral patterns focus on communication and control flow between objects. Think like this:

- Basic: _"How do I make two objects talk without tight coupling?"_
- Advanced: _"How do I design systems where behavior changes dynamically without reqriting the core logic?"_

---

## Observer Pattern

### **Core Idea**

An object maintains a list of dependants (observers) and notifies them automatically of state changes.

| Aspect             | Detail                                                                                |
| ------------------ | ------------------------------------------------------------------------------------- |
| **When to use**    | When one object's state change must update many others                                |
| **Real-world use** | Event-driven architectures: notification services, feed updates, real-time dashboards |
| **Example**        | User posts → feed service, notification service, analytics service all                |

### **Implementation challenges at scale**:

- _Observer explosion_: Too many subscribers → notification storms

- _Ordering guarantees_: Observers may receive updates in unpredictable order

- _Memory leaks_: Forgotten unsubscriptions in long-running services

### **When NOT to use**:

- When you need guaranteed delivery ordering

- When observers are few and relationship is static (simple function call is cleaner)

---

## Strategy Pattern

### Core Idea

Define a family of algorithms, encapsulate each one, and make them interchangeable. The client chooses which strategy to use at runtime.

| Aspect         | Detail                                                                                                      |
| -------------- | ----------------------------------------------------------------------------------------------------------- |
| When to use    | When you have multiple algorithms for the same task and need to switch between them                         |
| Real-world use | Payment processing (Stripe/PayPal/Bank), load balancing algorithms, recommendation engines                  |
| Example        | Shipping cost calculator: FedExStrategy, UPSStrategy, DHLStrategy — interchangeable based on user selection |

### Implementation challenges at scale

- **Strategy bloat**: Too many strategies → hard to maintain

- **Runtime selection overhead**: Need clean factory/provider pattern

- **Testing complexity**: Each strategy needs independent test coverage

### Trade-offs:

| Pros                                      | Cons                                         |
| ----------------------------------------- | -------------------------------------------- |
| Eliminates large conditional statements   | Increases number of classes                  |
| SRP: each strategy has one responsibility | Clients must understand strategy differences |
| Easy to add new algorithms                | Can over-engineer simple cases               |

### When NOT to use:

- Only 1–2 algorithms that rarely change

- When the selection logic is complex and opaque

---

## State Pattern

### Core idea:

Allow an object to change its behavior when its internal state changes. The object appears to change its class.

| Aspect               | Detail                                                                                               |
| -------------------- | ---------------------------------------------------------------------------------------------------- |
| When to use          | When an object has distinct states and transitions affect behavior youtube                           |
| Real-world MAANG use | Order lifecycle (pending→paid→shipped→delivered), workflow engines, payment state machines           |
| Example              | E-commerce order: `PendingState` allows payment, `ShippedState` allows tracking but not cancellation |

### Implementation challenges at scale

- **State explosion**: Too many states → combinatorial complexity

- **Transition validation**: Need to enforce valid state transitions

- **Persistence**: State must be serialized and restored correctly

### Trade-offs:

| Pros                                          | Cons                                       |
| --------------------------------------------- | ------------------------------------------ |
| Explicit state transitions                    | More classes than simple if-else           |
| Single Responsibility: each state is isolated | State machine can become hard to visualize |
| Prevents invalid state combinations           | Debugging requires tracking state history  |

### When NOT to use:

- Object has only 2–3 states (simple boolean flags suffice)

- State transitions are rarely used or unpredictable

---

## Why Behavioral Patterns Matter at Scale

### Event-driven systems (Observer):

At scale, you can't have synchronous RPC chains. Observer enables decoupled event processing via message queues (Kafka, SQS). But you must handle:

- Dead letter queues for failed notifications

- Idempotency (same event sent twice)

- Backpressure when consumers lag

### Algorithm flexibility (Strategy):

In microservices, you need to swap algorithms without redeployment:

- A/B testing different recommendation strategies

- Regional payment processors (different strategies per region)

- Dynamic load balancing (least-connections vs round-robin based on load)

### Workflow correctness (State):

Distributed systems need explicit state machines to prevent invalid transitions:

- Order fulfillment pipelines

- Payment authorization → capture → refund

- Database migration states

---

## Common Implementation Challenges:

| Challenge                         | Why It Happens                                   | Mitigation                                          |
| --------------------------------- | ------------------------------------------------ | --------------------------------------------------- |
| Tight coupling hidden as patterns | Observers directly reference subject internals   | Use interfaces, dependency injection                |
| Over-engineering                  | Applying pattern where simple code works         | Ask: "Will this change in 6 months?"                |
| Testing complexity                | Multiple strategies/states increase paths        | Contract tests per strategy, state transition tests |
| Distributed state                 | State pattern in microservices needs persistence | Use event sourcing, state databases                 |
| Memory leaks                      | Observers not unregistered                       | Weak references, explicit lifecycle management      |

---

## When to Consider These Patterns (Decision Framework)

| Question                                                    | Observer | Strategy | State  |
| ----------------------------------------------------------- | -------- | -------- | ------ |
| Do I have multiple recipients for one event?                | ✅ Yes   | ❌       | ❌     |
| Do I need to swap algorithms at runtime?                    | ❌       | ✅ Yes   | ❌     |
| Does my object have distinct modes with different behavior? | ❌       | ❌       | ✅ Yes |
| Is this changing frequently (monthly/quarterly)?            | ✅       | ✅       | ✅     |
| Is this a one-time/static relationship?                     | ❌       | ❌       | ❌     |

**Rule of thumb**: Use patterns when you anticipate change, not just for current requirements.

---

## Best Practices

- Start simple, refactor to pattern when needed
  Don't preemptively apply patterns. Extract when you see code smells (large if-else, duplicated logic).

- Document the contract
  For Observer: what events are published? For Strategy: what's the input/output interface? For State: what transitions are valid?

- Test boundaries, not just happy paths

  **Observer**: what if one observer fails?

  **Strategy**: what if no strategy matches?

  **State**: what if invalid transition requested?

- Use existing frameworks when possible

  **Observer**: Kafka, Redis Pub/Sub, RxJS

  **Strategy**: Strategy pattern in design, but leverage existing libraries

  **State**: State machine libraries (AWS Step Functions, XState)

- Monitor and observability

  **Observer**: track notification latency, failure rates

  **Strategy**: a/b test strategy performance

  **State**: log state transitions for debugging

---

## System Design Interview Questions (What Interviewers Look For)

### **Observer Pattern Questions**

- "Design a notification system for a social media platform."
  What they're testing: Do you recognize the observer pattern? Can you handle scale (millions of followers)? Do you consider eventual consistency?

- "How would you handle a follower list that grows from 1K to 1M users?"
  What they're testing: Observer explosion problem, fan-out strategies (pull vs push), caching, batching.

- "What if one observer is slow and blocks others?"
  What they're testing: Async processing, queue-based decoupling, timeout handling.

### Strategy Pattern Questions

- "Design a payment processing system supporting multiple providers."
  What they're testing: Strategy pattern recognition, interface design, fallback strategies, idempotency.

- "How do you add a new payment provider without downtime?"
  What they're testing: Plugin architecture, feature flags, gradual rollout.

- "How would you A/B test two recommendation algorithms?"
  What they're testing: Strategy selection logic, user segmentation, metrics tracking.

### State Pattern Questions

-"Design an order fulfillment system for an e-commerce platform."
What they're testing: State machine design, valid transitions, persistence of state.

-"What happens if the service crashes during a state transition?"
What they're testing: Distributed transactions, idempotency, recovery mechanisms.

-"How do you prevent a user from cancelling a shipped order?"
What they're testing: State validation, business rules enforcement.

---

## High-Level Understanding Signals

| Signal               | Junior/Mid Response               | Senior Response                                                                                 |
| -------------------- | --------------------------------- | ----------------------------------------------------------------------------------------------- |
| Trade-off awareness  | "Observer is good for decoupling" | "Observer is good but watch for notification storms at scale; consider fan-out via SQS instead" |
| Failure modes        | Doesn't mention failure           | Discusses dead letter queues, retry strategies, idempotency                                     |
| Scale considerations | "It works for 100 users"          | "At 1M users, push-based observer fails; need pull-based or hybrid"                             |
| Pattern misuse       | Applies pattern everywhere        | Knows when NOT to use pattern (over-engineering)                                                |
| Distributed systems  | Assumes single process            | Considers network failures, eventual consistency, persistence                                   |

---

## Key Takeaways

- Behavioral patterns solve communication and control flow problems, not just code organization.

- At MAANG scale, these patterns extend beyond classes to distributed systems:

  **Observer** → Event buses (Kafka, SNS)

  **Strategy** → Configurable microservices

  **State** → Distributed state machines

- The interview differentiator is trade-off awareness: Knowing when to use a pattern, when to avoid it, and how it fails at scale.

- Best practice: Start with simple code, extract to pattern when you see repeated change patterns, not preemptively.
