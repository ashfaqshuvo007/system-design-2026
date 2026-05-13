# Consensus Algorithms - A High Level Understanding

Consensus algorithms ensure `multiple servers` agree on a `shared state` despite failures, a core need for scalable distributed systems.

- Nodes in a cluster must agree on data or leader despite crashes or network delays—this is consensus, solving the FLP impossibility by assuming crashes but not perfect sync.
- Raft breaks it into leader election (random timeouts elect a leader), log replication (leader appends entries, followers replicate), and safety (commit only when majority acknowledges).
- Paxos uses proposers, acceptors, and learners for multi-decree agreement but is more complex.

## Types and When to Use

- **Raft**: Prefer for new designs—readable state machine, easier debugging (e.g., etcd, Consul use it). Use when leader-centric ops fit (write-heavy coordination).
  - **Strength**: Unambiguous, all state transitions are explicit. It splits the problem into leader election, log replication, and safety, making it testable and debuggable. Membership changes are simpler with joint consensus. It’s the default recommendation today for fault‑tolerant replicated state machines.
  - **Cost**: The `single‑leader bottleneck` – all writes flow through the leader, so throughput is capped by one node’s network bandwidth and I/O. Leader failover pauses the write path; while reads can be served from followers with linearizable guarantees using leases or read‑index, that adds latency/complexity.

- **Paxos**: Legacy in Google (Chubby), suited for symmetric roles or when minimizing leader overhead. Avoid unless locked into existing stacks.
  - **Strength**: Very well studied, can be optimised for throughput with batching/pipelining, and in theory can handle multiple concurrent leaders (though in practice most deployments use a single distinguished proposer).
  - **Cost**: Extremely hard to implement correctly. The original paper is opaque, edge cases (leader changes, reconfiguration) are infamous. I’ve seen teams lose months to subtle bugs. Also, without a single stable leader, performance becomes erratic due to continuous proposal conflicts.

- Consider consensus for strong consistency in replicated logs (not CRDTs/Eventual Consistency for availability-first).

## Real-World Implications

- Powers KV stores (etcd, ZooKeeper), databases (CockroachDB uses Raft), service discovery.
- In generic scalability fix, Raft enables leader-driven sharding/coordination without single points beyond quorums.

## Implementation Challenges

- **Network partitions**: Quorum stalls writes (_f_ - fault tolerance requires 2 _f_ +1 nodes).

- **Leader election storms**: Mitigate via jittered timeouts.

- **Log divergence**: Raft's log matching via index-term checks; costly rollbacks.

- Clock skew irrelevant (logical clocks), but GC pauses can drop leadership.

## Trade-offs

| Aspect        | Raft                                      | Paxos                           | Notes                        |
| ------------- | ----------------------------------------- | ------------------------------- | ---------------------------- |
| Latency       | Leader proxy adds 2 RPCs                  | Symmetric, potential pipelining | Raft simpler for 99th %ile   |
| Complexity    | Low (state machine)                       | High (phases per slot)          | Raft 10x easier to implement |
| Throughput    | Leader bottleneck                         | Balanced                        | Paxos for read-heavy         |
| Debuggability | Visual states (follower/candidate/leader) | Opaque ballots                  | Raft wins production         |

# Interview Probes

- _"Design a distributed counter"_: Probe CAP (CP via consensus), Raft vs. vector clocks.

- _"etcd vs ZooKeeper"_: Raft simplicity vs Paxos+ZAB stability.

- _"Scale to 1M ops/sec"_: Leader election overhead? Vertical scale leader, shard state.

- _"Handle Byzantine faults?"_: No—use PBFT (costly O(n^2)).

**Strong signals**: Quorum math, FLP mention, real uses (Kubernetes etcd).
**Weak**: Ignoring partitions or claiming "atomic broadcasts suffice."

- _“Design a highly consistent distributed key‑value store that tolerates up to f failures.”_
  - Look for: choosing 2f+1 nodes, picking Raft or Multi-Paxos, clearly separating the control plane (leader election) from the data plane (log replication), and handling reads (linearizable vs eventual). A sharp candidate will discuss the CAP trade‑off – this is a CP system during a partition and how they’d size the cluster.

- _“How would you increase write throughput if the leader becomes a bottleneck?”_
  - listening for: pipelining, batching, separating the leader’s network I/O from disk I/O, but ultimately sharding the data. If they propose multi‑leader consensus, they’d better explain conflicts and commit ordering. “Leaderless” designs are a red flag unless they can prove they understand the cost.

- _“Walk me through what happens when the leader crashes and a new one is elected.”_
  - The candidate should articulate that a new term starts, the candidate requests votes with the highest last‑log‑index/term, and upon winning, commits a no‑op entry to establish its committed log prefix. They must mention any uncommitted entries from the previous term being overwritten, and how safety is preserved.

- _“You need to change cluster membership from 3 to 5 nodes. How does Raft handle that safely?”_
  - Expectation: joint consensus (C_old,new), requiring majorities from both the old and new sets, until the new config is committed. No “switch all at once” answer – that splits the brain.

- _“Why would you ever choose Paxos over Raft in a greenfield project today?”_
  - Acceptable answers: your org already has hardened Paxos libraries (e.g., internal Google infrastructure), you need specific performance characteristics of a proven Multi-Paxos implementation, or you’re publishing a paper. Otherwise, Raft wins on engineering cost.
