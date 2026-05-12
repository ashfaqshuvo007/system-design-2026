# Consistency models 🔥

Consistency models control what guarantees the system offers about how and when different clients see the same data. In distributed systems (MAANG‑style scale), you never get “perfect” guarantees everywhere; you pick models that match your user experience and availability needs, then live with the trade‑offs.

## The Types - (strong, eventual, causal)

Formally, consistency models describe how reads and writes behave across multiple replicas (databases, caches, services)

### Strong consistency (linearizability / strict serializability)

- After a successful write, _every_ subsequent read sees that value (or something newer).

- Often implemented via _single‑writer_ semantics, Paxos/Raft consensus, or distributed locks with high coordination overhead.

- **Example**:
  - Bank balance updates.
  - Critical configuration flags.

- **When to use**:
  - Reads must always reflect the latest committed write.
  - You can tolerate latency and lower availability during failures.

### Eventual consistency

- No immediate guarantee `that reads see the latest write`.

- If no new writes happen, replicas will eventually converge to the same value.

- Often seen in `peer‑to‑peer` replication, highly available key‑value stores (e.g., Dynamo‑style systems).

- **When to use**:
  - High availability and low latency matter more than immediate correctness (e.g., “likes” count, non‑critical user profiles).
  - You can tolerate temporary stale reads.

### Causal consistency

- If operation `B logically depends on operation A` (i.e., causally related), then every node that sees B must also see A.

- Unrelated operations can still be reordered; stronger than eventual, weaker than strict serializability.

- Typically implemented using `vector clocks` or `version vectors` to track causal dependencies.

- **When to use**:
  - Chat/messaging (if you reply to a message, you must see that message).
  - Collaborative apps where causality is user‑visible.

## Real‑world implications

1. **User experience**:
   - **Strong consistency** avoids “I updated it but I don’t see it” confusion but can increase latency.

   - **Eventual consistency** improves availability and latency but can show stale data, which is fine for “nice‑to‑have” features.

   - **Causal consistency** gives a “natural” feel for workflows (e.g., comment chains) without full serializability cost.

2. **Operational overhead**:
   - **Strong consistency** usually needs fencing, consensus (Raft/Paxos), distributed locks, or coordination quorums, which become bottlenecks at MAANG scale.

   - **Eventual consistency** lets replicas move independently, easing operations and cross‑region replication.

   - **Causal consistency** adds metadata (clocks/vectors) and more complex read/write logic.

3. **Global scale**:

- Multiregion deployments often layer models:
  - Strong consistency for core shard of a user’s data.

  - Eventual consistency for replicas in other regions.

  - Causal consistency for per‑user‑timeline operations (e.g., posts/replies).

## Trade-offs

| Model    | Guarantees you get                       | Typical cost / downside                         | Typical use case             |
| -------- | ---------------------------------------- | ----------------------------------------------- | ---------------------------- |
| Strong   | Reads always see latest committed write. | Higher latency, lower availability on failures. | Bank balances, config flags. |
| Eventual | Data converges if no new writes.         | Stale reads, surprising UX sometimes.           | Likes, secondary indexes.    |
| Causal   | Causal dependencies are preserved.       | Extra metadata, more complex logic.             | Messaging, comment threads.  |

## Why and when to pick each model

- **Always consider strong consistency** when:
  - Money, idempotency, or authorization are involved.

  - Debugging “which version is real?” becomes too expensive.

- **Always consider eventual consistency** when:
  - High availability and low latency are top priorities.

  - You can design UX to tolerate staleness (e.g., “updating…”, background sync).

- **Always consider causal consistency** when:
  - User workflows have explicit dependencies (post → comment → reply).

  - You want to avoid “messages out of order” but don’t want full serializability.

- **In practice**, MAANG‑style systems often mix models:
  - Strong consistency for canonical writes on a primary shard.

  - Eventual or causal for replicas, caches, and non‑critical counters.

## Implementation challenges

1. **Latency vs correctness**:
   - Strong consistency requires waiting for consensus/acks from multiple nodes, which increases tail latency.

   - Eventual consistency can return “fast‑but‑possibly‑stale” reads, so you must decide how lax you are (e.g., max staleness, read‑your‑own‑writes).

2. **Network partitions and failures**:
   - During partitions, strict consistency usually forces a “blocking” mode (fewer nodes online, higher latency).

   - Eventual consistency lets more nodes keep serving, but inconsistency can explode.

3. **Testing and debugging**:
   - Strong consistency is easier to reason about but harder to scale.

   - Causal and eventual consistency require sophisticated logging, tracing, and replay tooling to debug “why that value?” questions.

4. **Read‑your‑own‑writes (RYW)**:
   - Often layered on top of weaker models: route your own reads to the node that accepted your write, or use session tokens.

## Mental Model for Interviews

- For every new feature or API:
  1. Ask: “What is the worst that can happen if the user reads a stale value?”

  2. If the answer is “bad UX” only, drift towards eventual or causal.

  3. If the answer is “we lose money / break security,” default to strong and then only relax it if you can prove latencies and availability are unacceptable.

Consistency models are not academic trivia; they are knobs you turn while balancing _correctness_, _latency_, _availability_, and _complexity_. At your level, interviewers will expect you to justify why you chose one model over another, not just recall definitions.

## Interview‑style questions you should be ready to answer

These are typical in `MAANG‑style system‑design` rounds:

1. “Design a distributed banking ledger. How do you ensure users always see the latest balance?”
   - What they want: recognition that balances need strong consistency; discussion of locking, consensus, or single‑shard‑per‑account with follower replicas.

2. “How would you design a ‘likes’ feature that scales globally?”
   - What they want: willingness to accept eventual consistency, with techniques like local‑counter increment and async merge, and maybe RYW.

3. “How would you design a chat system where replies must appear after the message they reply to?”
   - What they want: mention of causal consistency (vector clocks, version vectors), or at least “sequence numbering per conversation” and careful ordering in the service layer.

4. “If our system supports both strong and eventual consistency, how do you decide which model to use for a given feature?”
   - What they want: back‑to‑back reasoning: data sensitivity, UX tolerance for staleness, and operational impact.

5. “Explain CAP theorem in the context of consistency choices.”
   - What they want: clear statement that you can’t have full consistency + availability + partition tolerance in one dimension; during a partition you must choose between strong consistency and availability (often implemented via eventual consistency).
