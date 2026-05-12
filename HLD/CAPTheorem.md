# 🔥 **CAP Theorem**

CAP stands for **Consistency**, **Availability**, **Partition tolerance**. It’s a proven theorem (Gilbert & Lynch, 2022) about distributed systems: you can only guarantee two of those three properties during a network partition.

- Consistency **(C)**: Every read sees the latest write, or an error. Think linearizability.

- Availability **(A)**: Every non-failing node responds to requests (no errors), although the response may not be the latest data.

- Partition tolerance **(P)**: The system continues to function despite arbitrary message loss between nodes.

A partition is not optional in real networks – it’s a given. _So the real choice is_: when a partition happens, do you sacrifice Availability (refuse requests to keep data consistent) or Consistency (serve potentially stale/conflicting data to stay available)? That gives you CP or AP systems.

## 🚀 Real-world Implications

CAP defines the system’s behaviour only during a network partition. It does not say you drop one property permanently. Our job is to decide, for a given business operation, which side of the trade-off is acceptable when links fail.

- **If we choose CP**: As soon as a minority node can’t reach the leader/quorum, that node refuses reads/writes (returns errors like `Unavailable`). This is what we do for things like payment captures or inventory decrements – we’d rather fail than double-deduct.

- **If we choose AP**: Every replica continues to serve reads/writes. Later, we resolve conflicts – last-write-wins, CRDTs, custom merging. This is how DynamoDB and Cassandra can accept writes during regional outages.

- **What Makes It Mendatory**
  - Any shared state across nodes is subject to partitions. Not planning for it means you’re implicitly signing up for unpredictable data corruption in production (split-brain, lost writes, inconsistent reads).
  - We must explicitly encode whether the system yields availability or consistency for each critical path. For scaling problem, skipping this leads to data anomalies we’ll only notice after a noisy neighbour event or rack failure.

## ⚖️ Trade-offs

When scoping a new service or endpoint, ask:

- **Is stale data acceptable?** If yes, lean AP with an eventual consistency model, and define the staleness bound (e.g., Amazon’s DynamoDB “strongly consistent reads” are CP, eventual are AP).

- **Does a human’s offline action risk irrecoverable loss?** (e.g., financial double‑spend). Then CP is required for the critical path – but maybe non‑critical read paths can be AP.

- **What’s the blast radius of unavailability?** For a retail checkout, refusing writes costs money. You might choose AP with a compensating reconciliation (credit card auth vs. capture).

- **Geographic distribution**: Multi-region deployments force AP for writes if you want low latency, because synchronous cross-DC consensus is too slow. You can, however, use CP for strongly consistent read-only keyspaces (e.g., user passwords).

| Choice | Wins                       | Losses                 | Use Case             |
| ------ | -------------------------- | ---------------------- | -------------------- |
| CP     | No stale reads             | Downtime on partitions | Inventory, finance   |
| AP     | High uptime, scales writes | Eventual consistency   | Caching, logs, feeds |

## ⚠️ Real‑world implementation challenges

- **Partial partition handling**: Clients often see different partial connectivity. Your system must handle asymmetric views without violating the chosen guarantee.

- **Tuning quorums**: In practice, many systems (DynamoDB, Cassandra) let you set `R + W > N` for strong consistency on a per-request basis, becoming CP for that operation. The challenge is correctly picking those values and understanding the latency/durability impact.

- **Latency vs. consensus**: CP systems rely on consensus (Paxos/Raft), which adds tail latency during reconfiguration. We need to budget for that in our `Service Level Objectives (SLOs)`.

- **Conflict resolution on AP side**: Once we go AP, we need a strategy for divergent writes. CRDTs work for some data types; otherwise we’re left with application-level merges or LWW which loses data. That choice must be deliberate, documented, and tested.

- **Testing partitions**: We must inject true network partitions (Chaos Engineering) not just node kills. Silent packet drops and asymmetric partitions expose bugs in fence tokens, read-repair, and anti-entropy.

## Interview Questions

- "Design URL shortener at 1B/day":
  - Pick AP (Cassandra), justify tunable consistency for writes.
  - Follow-up: Handle hot partitions? (Client sharding + consistent hashing).

- **"Banking transfer"**:
  - Force CP, explain quorum failure modes.

- **"Trade-off Twitter timeline"**:
  - AP with fan-out writes, read-repair at query-time.

- **"Design a banking ledger vs. a social media feed.”**
  - Ledger: CP, synchronous replication, idempotency keys, idempotent receiver.
  - Feed: AP, eventual consistency, timeline merging by timestamp (or logical clock), conflict resolution via LWW or application logic. Show you understand the business impact of trade-offs, not just the acronym.

- **“Design a distributed key‑value store like DynamoDB.”**
  - Talk about consistent hashing, replication factor N, R/W quorums.
  - Explicitly call out that the system defaults to AP but supports tunable consistency – R+W > N makes it a CP system for that operation.
  - Discuss hinted handoff, read repair, anti-entropy.
  - Show you know that the ring membership gossip protocol itself is partition-tolerant.

- **“When does CAP mislead you?”**
  - Point out that it assumes full partition (perfectly dropping all messages) and 100% availability/consistency requirements.
  - In practice, you can have partial availability (e.g., serve reads but not writes), and you can gracefully degrade.
  - Also, latency and partitions are related (the PACELC theorem extends CAP for normal operation).
  - A perfect answer highlights that high latency is often treated as a partition, so CP systems start timing out early and hurting availability even without total failure.
