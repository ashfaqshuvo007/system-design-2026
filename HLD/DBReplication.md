# DB Replication

Db replication can be used in many Db management systems, usually with master/slave relationship between the original(master) and the copies (slave).

- Master DB only supports `WRITE` operation.
- A slave DB gets copies of the data from the master and only supports `READ` operations.
- Data-modifying commands like `insert`, `update` or `delete` dealt with master DB.

## Replication Advantages:

- **Better Performance**
  - The `master-slave` model improves performance.
  - More queries to be handled in parallel.

- **Reliability**
  - If one DB server is lost.
  - As multiple replications are present, no data loss.

- **High Availability**
  - DB replication across different geo-locations.
  - In case of one DB is offline, data can be accessed from another server.

## Main Types by architecture

- **Single Leader (Master-Slave)**
  - Writes only to master.
  - Slaves are replicated for reads.

- **Multi-Leader (Master-Master)**
  - Multiple masters accepts writes.
  - Sync changes bidirectionally.

- **Leaderless (e.g. Dynamo-style)**
  - All nodes handle reads/writes.
  - Achieves eventual consistency via read repair.

## By sync method (crosses architectures):

- **Synchronous**
  - Master waits for replica `ACK` before commit (ZERO data loss).

- **Asynchronous**
  - Master commits immidiately, replicats later (FASTER but lag risk).

| Type          | Sync Mode Fit  | Use Case                        |
| ------------- | -------------- | ------------------------------- |
| Single-Leader | Async common   | Read scalingtryexponent         |
| Multi-Leader  | Often async    | Write distributiongeeksforgeeks |
| Leaderless    | Async/eventual | High availabilitytryexponent    |

## Challenges

- **Replication Lag**
  - Async slaves serve stale data.
  - Worsens with network issues or high load.
  - Mitigate with sticky sessions, read‑after‑write validation tokens, or reading from leader for critical operations.

- **Consistency**
  - Sync blocks writes(latency).
  - Async risks loss on failure.
  - Multi-leader needs conflict resolution (e.g., last-write-wins).

- **Failover**
  - Master failure stops writes in single-leader until promotion.

- **Complexity/Cost**:
  - More replicas mean monitoring, bandwidth overhead, ops burden.

## Trade-offs

| Aspect            | Single-Leader Async     | Multi-Leader Async   | Sync (Any)                            |
| ----------------- | ----------------------- | -------------------- | ------------------------------------- |
| Write Throughput  | Low (master bottleneck) | High                 | Lower (waits)aerospike                |
| Read Scaling      | Excellent               | Good                 | Good, but slowertryexponent           |
| Consistency       | Eventual (stale reads)  | Eventual + conflicts | Strong (zero loss)purestorage         |
| Failover Downtime | Seconds-minutes         | Low                  | Low, but latency hithokstadconsulting |
| Complexity        | Low                     | High                 | Mediumslideshare                      |

## Making the Decision

| Need                                               | Likely choice                                                                 |
| -------------------------------------------------- | ----------------------------------------------------------------------------- |
| Read‑heavy web app, okay with seconds of staleness | Single‑leader, async read replicas                                            |
| Need zero data loss on failover                    | Semi‑sync or sync replication, plus consensus‑driven failover                 |
| Multi‑region active‑active with low‑latency writes | Multi‑leader (e.g., MySQL Group Replication, CockroachDB) with conflict rules |
| Massive write volume, relaxed consistency          | Leaderless, tunable quorum (Cassandra,DynamoDB)                               |
| Strong consistency across regions at scale         | Spanner, CockroachDB (trades latency for external consistency)                |
