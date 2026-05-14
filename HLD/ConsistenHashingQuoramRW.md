# Consistent Hashing And Quoram RW

## Consistent Hashing

Map both nodes (servers) and keys onto a large circular space (hash ring), e.g., via `MD5/SHA-1`. A key is assigned to the first node encountered moving clockwise. When a node joins or leaves, only the keys in its immediate neighborhood are remapped – typically `K/N` keys move, not the entire dataset. To avoid hotspots and uneven load, we assign each physical node multiple virtual nodes (vnodes) scattered around the ring.

- Real-world: Dynamo, Cassandra, and Redis Cluster use variants of this.

- When to consider: You’re partitioning state across a dynamic set of nodes (distributed caches, KV stores, CDN edge selection) and you want minimal data movement on cluster changes.

## Consistent Hashing Implementation:

Ring space `2^160 (MD5)`. Node failure: successor absorbs arc (`O(log N)` lookup via finger tables). Hotspots mitigated by rendezvous hashing or multi-hash (`K=3 `probes). Real-world: Akka Cluster sharding, but challenges include skew (fix: bounded loads), network partitions _(CAP-P violates AP)_.

## Quorum Reads/Writes

In a leaderless, replicated system, each piece of data has N replicas. A write succeeds if it’s acknowledged by at least W replicas, a read succeeds if at least R replicas respond. If we configure `R + W > N`, any read quorum will intersect with the most recent write quorum, so a reader is guaranteed to see the latest acknowledged write (assuming no concurrent writes). This is the quorum overlap property.

- Example: `N=3, W=2, R=2` gives intersection. A write must persist to two replicas; a read fetches from two and picks the newest based on version information (vector clocks / timestamps).

- Real systems add sloppy quorum, hinted handoff, and read‑repair to handle temporary failures gracefully.

- Trade-offs: Higher `R/W` increase consistency but add latency (wait for slowest replica). Lower values improve availability and latency but may serve stale data.

## Quoram Variants

| Type     | W,R,N   | Consistency      | Latency | Use Case              |
| -------- | ------- | ---------------- | ------- | --------------------- |
| Strong   | W+R>N   | Linearizable     | High    | Banking               |
| Causal   | W+R>N+1 | Read-your-writes | Medium  | Social feeds          |
| Eventual | W+R≤N   | Best-effort      | Low     | Analytics designgurus |

## When to choose this architecture

Choose leaderless consistent hashing + tunable quorum when:

- You need high write availability and partition tolerance (AP of CAP).

- Data can be modelled as key‑value with eventual consistency; conflicts are resolvable (e.g., shopping cart merge, user session).

- You can tune consistency per operation (e.g., `QUORUM` for writes of financial transactions, `ONE` for less critical reads).
  Do not pick this when you need strict serializability across multiple keys – then you need a consensus‑based system (Raft/Paxos) or a strong leader‑based design.

## Interview expectations

A candidate should show they can reason about these trade‑offs under constraints. Typical questions:

1. _“Design a distributed URL shortener that handles 1M writes/sec.”_
   - Good answer: partition by hash of short key (consistent hashing), replicate with N=3, write with W=2, read with R=2, discuss how to handle hot keys with caching, and how to detect collisions.

2. _“What happens if you set W=1, R=N in Dynamo? What guarantee do you get?”_
   - That’s fast writes, but reads always see the latest written value (since any write goes to at least one node that is in the full read set) – you get read‑your‑writes consistency if the writer also reads, but no durability under failures (if the single written node dies, data is lost until repair). The candidate must mention durability loss.

3. _“Explain how you’d add a node to a cluster without downtime and with minimal impact.”_
   - They should talk about gradual data streaming, using hints while the node bootstraps, not changing token assignments abruptly, and monitoring load.

4. _“Why not just use modulo hashing?”_
   - Answer: remapping almost all keys on node addition/removal, causing massive cache misses/data movement. Consistent hashing solves this.

5. _“How do you handle read‑repair in a quorum read?”_
   - After a read quorum, the coordinator compares versions and pushes the latest to stale replicas asynchronously. This reduces the probability of reading stale data later, but it’s not a substitute for full anti‑entropy.

### Wrap-up

So consistent hashing gives us elastic partitioning, quorums give us tunable consistency vs. latency in a leaderless design. The real world is full of gotchas: vector clock explosion under high concurrency, gossip overhead at scale, rebalancing throttle tuning, and the constant temptation to set `R+W <= N` for performance that breaks the safety net. As a mid‑level engineer you’re expected to pick the right knobs for the specific workload, foresee the failure modes, and articulate the trade‑offs clearly in design reviews – and that’s exactly what interviewers probe for.
