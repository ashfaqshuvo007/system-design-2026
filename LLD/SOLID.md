# Solid Principles

SOLID is a mnemonic for 5 object-oriented design principles that make code understandable, flexible, and maintainable:

| Principle                                 | Core Idea                                             | Simple Example                                                        |
| ----------------------------------------- | ----------------------------------------------------- | --------------------------------------------------------------------- |
| S – Single Responsibility Principle (SRP) | A class has one reason to change (one job)            | User class handles user data; UserRepository handles DB access        |
| O – Open/Closed Principle (OCP)           | Open for extension, closed for modification           | Add new payment method via new class, not by editing PaymentProcessor |
| L – Liskov Substitution Principle (LSP)   | Subtypes must be substitutable for base types         | Dog extending Animal shouldn't break code expecting Animal            |
| I – Interface Segregation Principle (ISP) | Clients shouldn't depend on interfaces they don't use | Split Worker into Workable, Restable instead of one fat interface     |
| D – Dependency Inversion Principle (DIP)  | Depend on abstractions, not concrete implementations  | Service depends on Repository interface, not MySQLRepository          |

## Real-world implications in scalable systems

SOLID isn't just "good practice"—it's a scalability and maintainability lever in large codebases.

1. SRP in microservices & large modules

- A class doing both business logic and logging will become a bottleneck when logging strategy changes (e.g., switching to async logging).

- In MAANG-scale code, SRP reduces change impact analysis time: you know exactly what to touch.

2. OCP for feature expansion without regression

- Adding new features (e.g., new notification channels: SMS, Push, Email) via new classes instead of modifying existing NotificationCenter avoids breaking existing tests and deployments.

- This is critical in continuous deployment environments where you ship daily.

3. LSP for safe polymorphism in distributed systems

- If CachedCache extends Cache but violates caching semantics (e.g., returns stale data where fresh is guaranteed), higher-level logic breaks silently.

- In distributed caches or queue abstractions, LSP violations cause subtle, hard-to-debug consistency issues.

4. ISP for lean APIs in large teams

- A bulky interface like UserService with 20 methods forces implementing classes to provide stubs for unused methods → more code, more bugs.

- In large teams, ISP reduces unnecessary coupling between services.

5. DIP for testability and replaceability

- Depending on abstractions lets you swap implementations (e.g., real DB vs. in-memory mock) without changing business logic.

- This is essential for unit testing, CI pipelines, and A/B testing different implementations.

## Challenges in Implementation

| Challenge               | Why it happens                                   | Practical impact                                                          |
| ----------------------- | ------------------------------------------------ | ------------------------------------------------------------------------- |
| Over-engineering        | Applying SOLID everywhere, even in small modules | More classes, more indirection, harder to onboard new engineers           |
| Performance overhead    | Extra layers of abstraction, indirection         | Slight latency increase; sometimes unacceptable in latency-critical paths |
| Team maturity mismatch  | Juniors struggle with abstraction boundaries     | More code reviews, more bugs if misapplied                                |
| Legacy code constraints | Existing tightly-coupled code                    | Refactoring risk vs. benefit trade-off                                    |
| False sense of safety   | "SOLID-compliant" but still poorly designed      | Still hard to maintain if domain model is wrong                           |

**Key insight**: SOLID is a tool, not a dogma. At scale, you apply it where it buys you maintainability and flexibility, not everywhere by default.

---

## When to Use SOLID (and When Not To)

**Use SOLID when:**

- The module is core: logic likely to evolve (e.g., payment, authentication).

- Multiple teams will touch the code.

- You need high testability (unit tests, mocks).

- The system must support frequent feature additions without regression.

**Be cautious or relax SOLID when**:

- Writing performance-critical, stable code (e.g., tight loops, hot paths).

- Prototyping or MVP where speed > long-term maintainability.

- The module is small and unlikely to change.

- The team is early-stage and SOLID adds unnecessary cognitive load.

---

## Trade-offs

| Benefit                   | Cost                                                 |
| ------------------------- | ---------------------------------------------------- |
| Easier to extend & modify | More classes, more files                             |
| Better testability        | More indirection, slightly harder to trace execution |
| Reduced coupling          | More abstractions to understand                      |
| Clearer responsibilities  | Initial design takes more time                       |

In large-scale companies, the trade-off usually favors SOLID in core services, but not in every utility or script.

---

## Best Practices

**Start with requirements, not principles.**

- Identify what's likely to change, then apply SOLID where it matters.

**Refactor gradually.**

- Don't rewrite everything; apply SOLID when you touch a module (the "boy scout rule": leave it cleaner than you found it).

**Use design patterns alongside SOLID.**

- Strategy, Factory, Observer, Decorator naturally enforce SOLID.

**Enforce via code reviews.**

- Check: "Does this class have one reason to change?" "Can I add a feature without modifying this class?"

**Measure impact.**

- Track metrics like:
  - Number of files changed per feature

  - Regression bug rate

  - Onboarding time for new engineers

---

## Interview Questions for High-level Understanding

These are the kinds of questions asked in a system design / LLD interview to gauge understanding:

### **Conceptual questions**

- Explain SRP with a real example from your work. What happened when it was violated?
  → Tests practical experience, not just memorization.

- How does OCP help in a system that frequently adds new features?
  → Tests ability to connect principle to scalability.

- Give an example where violating LSP caused a bug. How did you fix it?
  → Tests understanding of polymorphism safety.

- When would you intentionally break a SOLID principle? Why?
  → Tests judgment and trade-off awareness (senior-level thinking).

### Design questions (LLD-style)

- Design a notification system that supports Email, SMS, Push, and future channels.
  - How do you apply OCP and SRP?

  - What interfaces/classes do you create?

- Design a payment processor supporting multiple payment methods.
  - How do you ensure adding a new method doesn't modify existing code?

  - Show how DIP helps in testing.

- Design a caching layer with multiple strategies (LRU, LFU, TTL).
  - How do you use OCP and Strategy pattern?

  - Where would LSP matter?

You have a large OrderService class doing validation, persistence, and notification.

- How do you refactor it using SOLID?
  - Tests ability to identify responsibilities and split them.

### Trade-off & judgment questions

In a latency-critical service, when might you relax SOLID?

- → Tests practical realism vs. theoretical purity.

How do you decide how much abstraction is "enough" in a new service?

- → Tests ability to balance maintainability vs. complexity.

## How This Applies to a Mid-Level Engineer

At this level, expect you to:

- Recognize when a module is getting too coupled or hard to extend.

- Propose SOLID-based refactors with clear justification (not just "because SOLID").

- Design classes and interfaces that are easy to test and extend.

- Communicate trade-offs clearly in design reviews.

- Guide juniors on when and how to apply SOLID without over-engineering.

You don't need to memorize every pattern, but you should be able to reason about design quality and justify your choices in interviews and real work.
