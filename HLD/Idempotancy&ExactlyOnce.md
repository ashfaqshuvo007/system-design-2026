# Idempotency & exactly‑once semantics

---

Distributed systems are built on unreliable networks. If a client sends a request but never gets the response (network blip), it retries. Without protection, that means duplicate side effects – double charges, double order placements. Idempotency and exactly-once are our defense.

**Idempotency**

- an operation that produces the same result no matter how many times you execute it, as long as the input parameters are identical.
- Mathematically: `f(f(x))`
- In HTTP: `GET`, `PUT` (full replacement), `DELETE` are idempotent by spec. `POST` is not; two identical `POST` requests create two resources.

**Exactly-once semantics**

- the guarantee that a given message/request is processed exactly once, even if the sender retries due to timeouts or failures.
- It’s an end-to-end promise, not just an API property.
- In practice, this is very hard to achieve end‑to‑end; most systems settle for “at‑least‑once + idempotent consumers” or “at‑most‑once + human error‑tolerant flows”.

**At-least-once** delivery is the default in most messaging systems: they’ll keep redelivering until acknowledged. Without an idempotent receiver, at-least-once becomes at-least-twice.

## How To Make Something Idempotent

---

We as developers should be able to design idempotent APIs and processing logic. Typical patterns to achieve this:

**Idempotency keys (client‑side)**

- Client sends a `idempotency_key` (UUID) with each request.

- Server stores `idempotency_key → result` (e.g., in Redis or DB) for a TTL.

- On duplicate key, return the stored result instead of re‑executing.

- Payment systems, order creation, and webhook APIs often use this.

**Idempotent update (state-based)**

- Instead of `POST /inc_votes`, use `PUT /votes` with the full desired state. Or `PATCH /votes` with optimistic‑locking (e.g., `version` field).

- That way, if the same request is retried, the update is the same.

**Deduplication in consumers**

- Message‑processing loop:
  - Consumer reads a message with `message_id`.

  - Before processing, store `processed(message_id)` in a durable store (DB / Redis).

  - If already seen, skip; else, process and commit.

- This gives you “effectively once” processing, even if the message is delivered multiple times.

## Idempotency vs. Strong "exactly-once":

---

Trade-offs reasoning:

| Scenario                                 | Typical choice                                | Why                                                                            |
| ---------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------------ |
| Reading / queries (e.g., GET)            | Idempotent by design                          | No side effects; retries are harmless. dev                                     |
| Updates where state is known (e.g., PUT) | Idempotent API                                | Same body → same outcome; easy to retry. dev                                   |
| Order creation / payment initiation      | Idempotent API with idempotency key           | Clients can retry safely; you avoid double‑charging. youtube                   |
| Analytics / logging                      | At‑least‑once + tolerate some duplicates      | Cost of deduplication is higher than duplicate events. youtube                 |
| Real‑money payments / ledger updates     | Idempotent processing + strict reconciliation | You care about correctness, not throughput; you run reconciliations. wikipedia |

## Challenges in Implementation:

---

### Idempotency

- **Partial failures after business logic execution but before storing the result**: write the result and the key in the same transactional scope as the side effect. If the business logic is a database write, insert the idempotency record in the same DB transaction. For side effects outside the DB, use an outbox pattern.

- **Expiry vs. retry windows**: Keys must live longer than the maximum expected retry window. If a key expires early, a late retry creates a duplicate. This is a business decision.

- **Storage scaling**: Every API call requires a write and a read to the idempotency store. For high-throughput systems, this can become a bottleneck. You mitigate with sharding by key, caching with short TTL, and careful capacity planning.

- **Client guidance**: The client must generate a new key for every unique operation intent, not reuse keys across different operations. This is documentation and SDK responsibility.

### Exactly-once semantics

True “exactly-once delivery” is impossible in a distributed system (Two Generals’ Problem). What we build is effectively-once processing: the infrastructure ensures that duplicates are filtered out and state mutations happen as if the message was processed once.

**In messaging (Kafka)**:

- Idempotent producer: The broker deduplicates messages via a producer ID and sequence number. Prevents duplicate writes caused by producer retries.

- Transactional API: Writes to multiple partitions and consumer offset commits are wrapped in an atomic transaction. Together with a consumer that reads committed messages only, you get exactly-once stream processing.

**In a request-reply pattern (e.g., an RPC)**:

- Combine idempotent endpoints with deduplication at the receiver.
- That’s exactly-once – the client retries with the same idempotency key, the server ensures single execution.

## What a senior engineer expects in interviews

---

In a MAANG‑style system design interview, you’ll be tested on:

**Conceptual understanding**

- Can you clearly define idempotency vs. exactly‑once?

- Can you distinguish “at‑least‑once delivery” from “exactly‑once processing”?

**Design‑level questions they might ask**

- “How would you design a payment API that clients can safely retry?”

- “How would you make a message‑processing service idempotent?”

- “What happens if the network disconnects after the server executes the logic but before it returns the response?”

**Trade‑off reasoning**

- “Should every API be idempotent?” → No; only where repeats are common (async, retries).

- “Can you have true exactly‑once semantics across services?” → Practically, no; you approximate via idempotency + coordination.

**Operational awareness**

- How do you measure and detect duplicates?

- How do you run reconciliation (e.g., nightly diff between service A and payment service)?

- How do you keep idempotency‑key storage performant and not a single‑point‑of‑failure?

## Exercises

---

Concrete exercise you can do now (for you)

To internalize this:

- Design a /charge‑card endpoint
  - The frontend sends amount, card_id, idempotency_key.

  - You sketch how the backend stores idempotency keys, where you lock / don’t lock, and how you avoid double‑charging on retries.

- Design a Kafka‑style consumer
  - Consumer processes order_created events.

  - Show how you avoid duplicate order creation, e.g., using event_id + DB dedupe table.
