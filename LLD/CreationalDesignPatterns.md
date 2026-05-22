# Creational Design Patterns: System Design Refresher

Creational design patterns solve how objects are created without hardcoding their exact classes. This decouples creation logic from usage, making code more flexible and testable.

| Pattern   | Core Idea                                                                                  | When You’d Use It (Simple Example)                                                                           |
| --------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------ |
| Factory   | Define an interface for creating objects; let subclasses decide which class to instantiate | You have multiple payment processors (Stripe, PayPal, Razorpay)—factory returns the right one based on input |
| Singleton | Ensure only one instance exists globally                                                   | Config manager, database connection pool, logging service                                                    |
| Builder   | Construct complex objects step-by-step                                                     | Building a User object with 10+ optional fields (name, email, preferences, metadata)                         |

## Factory Design Pattern

**Why it matters at scale**:

- Decouples component selection: In microservices, factories let you swap implementations (e.g., Redis vs. Memcached cache) without changing client code.

- Enables plugin architectures: AWS SDK uses factories to create different service clients (S3Client, DynamoDBClient) from a common interface.

**Implementation challenges**:

- Over-engineering: Don’t use Factory for simple cases (e.g., creating a Logger—just instantiate it).

- Testing complexity: Mocking factories requires extra setup in unit tests.

**When to use**:

- You have multiple interchangeable implementations of an interface.

- Creation logic is non-trivial (e.g., reading config, choosing based on environment).

**Trade-offs**

| Pros                                                                           | Cons                                                   |
| ------------------------------------------------------------------------------ | ------------------------------------------------------ |
| Open/closed principle: Add new implementations without modifying existing code | Adds indirection; harder to trace object creation flow |
| Centralized creation logic                                                     | Can become a “god factory” if overused                 |

---

## Singleton Pattern

**Why it matters at scale**:

- _Resource conservation_: Single database connection pool prevents exhausting DB connections under high load.

- _Shared state_: Cache managers (e.g., in-memory LRU cache) need one global instance to avoid stale data.

**Implementation challenges**:

- _Thread safety_: In concurrent environments (Java/Python), naive singletons break. Use double-checked locking or language-level solutions (e.g., Python’s module-level singleton, Java’s enum singleton).

- _Testing difficulty_: Global state makes tests brittle; you must reset state between tests.

- _Distributed systems_: Singleton doesn’t work across multiple machines. Use distributed singletons (e.g., Redis-based lock, leader election via etcd/Consul).

**When to use**:

- You must have exactly one instance (e.g., config, thread pool).

- Creation is expensive and reuse is critical.

**Trade-offs**

| Pros                                 | Cons                                                                    |
| ------------------------------------ | ----------------------------------------------------------------------- |
| Controlled access to unique instance | Global state = tight coupling, hard to test                             |
| Lazy initialization saves resources  | Not truly singleton in distributed systems without extra infrastructure |

---

## Builder Pattern

**Why it matters at scale**:

- _Readability_: Constructing complex objects with many parameters becomes self-documenting:

  ```java
  User user = new User.Builder()
  .name("Ashfaq")
  .email("ashfaq@example.com")
  .preferences(new Preferences())
  .build();
  ```

- _Immutable objects_: Builder enables creating immutable objects with many fields (critical for thread safety in concurrent systems).

**Implementation challenges**:

- _Boilerplate_: Builders add 2–3x more code (Builder class + constructor).

- _Validation_: You must validate at `build()` time; invalid intermediate states are possible.

**When to use**:

- Object has many optional parameters (5+).

- You need immutable objects with complex construction.

- Object construction involves validation or templating.

**Trade-offs**

| Pros                  | Cons                                               |
| --------------------- | -------------------------------------------------- |
| Clear, chainable API  | More boilerplate code                              |
| Supports immutability | Builder itself must be managed/lifetime controlled |

---

## Implementation Challenges

| Challenge             | Factory                                  | Singleton                                  | Builder                                      |
| --------------------- | ---------------------------------------- | ------------------------------------------ | -------------------------------------------- |
| Thread safety         | Low risk (creation is usually stateless) | **High risk**—requires synchronization     | Low risk (builder is typically thread-local) |
| Testing               | Moderate (mock factory)                  | **Hard** (global state)                    | Easy (inject builder)                        |
| Distributed systems   | Works fine                               | **Fails** without distributed coordination | Works fine                                   |
| Over-engineering risk | Medium                                   | High                                       | Low-Medium                                   |

---

## Why and When To Consider These Patterns

**Use creational patterns when**:

- Creation logic is _complex or environment-dependent_.

- You need _flexibility_ to swap implementations.

- You’re building _libraries/frameworks_ (not just app code).

- _Immutability_ and _thread_ safety are priorities.

**Avoid when**:

- Object creation is trivial (e.g., `new Logger()`).

- You’re prototype-building and moving fast.

- The pattern adds abstraction without clear benefit.

---

## Best Practices

- _Prefer dependency injection_ over Singleton’s `getInstance()`.

- _Document why_ you’re using a pattern (e.g., “Factory used to support multi-cloud provider abstraction”).

- _Test creational logic separately_ — factories/builders are logic-heavy.

- _Don’t cascade patterns_: Building a Singleton with a Factory inside a Builder is usually overkill.

- _In distributed systems_, assume Singleton doesn’t exist—use distributed locks or consensus algorithms instead.

---

## System Design Interview Questions

**Factory Pattern**:

- “Design a notification service that supports email, SMS, and push notifications. How would you structure object creation?”

  → Expect: Factory returning NotificationSender based on type; mention dependency injection for testability.

- “How would you add a new notification channel without modifying existing code?”

  → Expect: Open/closed principle; new SMSNotificationFactory subclass.

**Singleton Pattern**:

- “Design a rate limiter for an API gateway. How do you ensure only one rate limiter instance exists across 100 microservices?”

  → Expect: Singleton doesn’t work here; use distributed rate limiter (Redis + Lua script, or token bucket with centralized store).

- “How do you make a thread-safe Singleton in Java?”

  → Expect: Double-checked locking, enum singleton, or private static final instance with lazy initialization.

**Builder Pattern**:

- “Design a query builder for a search API with 20+ optional filters. How do you ensure type safety and readability?”

  → Expect: Builder with chainable methods; validate at build() time.

- “When would Builder be worse than a constructor with default parameters?”

  → Expect: Builder adds boilerplate; avoid for simple objects with 2–3 fields.

**Trade-off Questions:**

- “Why not use Singleton for everything that needs shared state?”

  → Expect: Global state = tight coupling, hard to test, doesn’t scale in distributed systems.

- “When does Factory become anti-pattern?”

  → Expect: When creation is trivial; adds unnecessary indirection and complexity.

---

## Key Takeaways

**You’re expected to**:

- _Know when NOT to use a pattern_ (often more important than knowing when to use it).

- Understand _distributed-system implications_ (Singleton breaks across machines).

- Articulate _trade-offs clearly_ in interviews: “I’d use Factory here because X, but the cost is Y.”

- These patterns are tools—not rules. In large-scale systems, the right choice depends on _scalability requirements_, _testability needs_, and _operational complexity_.
