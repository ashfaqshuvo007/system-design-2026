# Database Indexing

When scaling a product – DB layer is usually the first **bottleneck**. Indexing isn’t “add an index on every column”. It’s about `access pattern → index type` Indexing is key to scaling database performance by `speeding up reads without full table scans`, but it introduces write overhead and storage costs.

## Index Types

| Type          | Description                                                            | Use Case                                              |
| ------------- | ---------------------------------------------------------------------- | ----------------------------------------------------- |
| Clustered     | Physically sorts table rows; one per table (often PK).                 | Range scans, sorts (e.g., dates, IDs). acceldata+1    |
| Non-clustered | Separate B-tree with column values + row pointers; multiple OK.        | Equality searches on non-PK (e.g., email). crownrms+1 |
| Composite     | Multi-column (left-prefix order matters: (A,B) helps A or A+B, not B). | Frequent multi-filter queries. linkedin+1             |
| Hash          | Hash table for exact matches; no ranges.                               | = lookups (e.g., keys in memory DBs). red-gate        |
| Bitmap        | Bit vectors for low-cardinality (e.g., gender); good for AND/OR.       | Analytics, few distinct values. red-gate              |

## Challenges

- **Write amplification**:
  - Each `INSERT/UPDATE/DELETE` updates all indexes (e.g., 6 indexes = 7x writes).

- **Storage**:
  - Indexes `~same size as table`; over-indexing balloons costs.

- **Optimizer issues**:
  - Too many indexes confuse query planner; low-selectivity skipped.

- **Maintenance**:
  - Fragmentation needs REBUILD; large tables need partitioning.

## Trade-offs

| Aspect      | Pro                                   | Con                                                                            |
| ----------- | ------------------------------------- | ------------------------------------------------------------------------------ |
| Read Speed  | O(log N) vs O(N) scan. geeksforgeeks  | Covering indexes avoid table touch but larger.                                 |
| Write Speed | -                                     | Multiplies cost per index. Read-heavy: OK; write-heavy: avoid excess. crownrms |
| Storage     | Selective indexes efficient.          | Bitmap for low-card OK; avoid large-text. geeksforgeeks                        |
| Scale       | Partial for ranges; composite prefix. | Hash no ranges; clustered leaf-level only. gyandesk                            |

## Scaling Recommendations

- **Startups (low volume)**:
  - Single clustered on _PK + 2-3 non-clustered_ on hot filters.
  - Monitor slow queries, add surgically.

- **Growth**:
  - Covering composites for top queries; bitmap for categoricals.
  - Partition large tables.

- **Large-scale** (e.g., MAANG):
  - Read replicas for queries; sharding; external search (ES) for full-text.
  - Drop unused indexes quarterly; use index hints sparingly.
  - Analyze EXPLAIN plans.
