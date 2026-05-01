# Caching

- Keeping fast copies of data we frequently use.
- **Trade-off**: Sometimes may lead to slightly stale data but makes the app much faster.
- **Cache invalidation**: deciding when and how to update or delete the cached answer.

We need to think about three aspects:

1. **Where do we cache ?**
2. **How do we read/write data ?**
3. **How do we trade off consistency vs. latency vs. complexity ?**

---

![Cache Tier](../images/cacheTier.png)

## 1. Where do we cache (layers)

You can place caches at different layers:

- **Browser / CDN**
  - Static assets (CSS, JS, images, logos) via `Cache‑Control`, `Etag`, `CDN`.

  - Benefit: No request ever reaches our backend for static stuff.

- **Application layer cache**
  - In‑memory store (Redis / Memcached) before the database.

  - _Example_: User profile by `user_id`, product details by `product_id`, expensive aggregate like “top 10 trending items”.

- **Database‑level cache**
  - Query‑caching, indexes, or in‑memory DB (e.g., Redis‑based for reads).

- **Client‑side cache (web / mobile)**
  - Local storage, `localStorage`, `IndexedDB`, or in‑memory caches in the app.

- For **horizontal scaling**, we usually combine:
  - `CDN → Reverse proxy / API gateway → App → Redis cluster → DB.`
    Each layer filters out the “easy” requests so the DB only sees the hard ones.

## 2. Common Caching Strategies (How do we read/write data)

Different caching strategies are used depending on the data type, size and access patterns

### 2.1 Read-through caching (cache-aside)

- First, check if cache has the available response
- If not, queries the DB, stores response in Cache and sends it back to the client.

  **On write**:
  - Writes to DB, invalidate (or update) the cache key.

  **Usecase**:
  - When reads are frequent and write is not.

  **Trade-off**:
  - Some risk of stale data between write and invalidation.

### 2.2 Write‑through Caching

- On reads, Same as Read-through

  **On write**:
  - Writes to both _cache_ and _db_ (synchronously).

  **Pros**:
  - Data is always consistent.

  **Cons**:
  - Slower write operations as DB is the critical path.

### 2.3 Write‑Behind / Write‑Back

- Read operations, same as read-through.

  **On write**:
  - write to cache immediately, then queue the DB write (asynchronously).

  **Pros**:
  - very fast writes, good for write‑heavy systems.

  **Cons**:
  - risk of data loss if cache crashes before DB write.

### 2.4 Refresh‑Ahead / Pre‑warming

- Read operations, same as read-through.

- Proactively load data into cache during low‑traffic or startup (e.g., warm popular product pages before a sale).

- Helps avoid “cache‑stampede” on first‑time reads.

**Interview Tip**:

For interviews and design, you should be able to:

- Draw a simple diagram: `Client → CDN → API → Redis → DB`.

- Explain the `cache hit rate goal` (e.g., 80–95% for heavy‑read services).

- Talk about cache `invalidation strategies` (time‑based TTL, event‑based invalidation on writes, versioning keys).

## 3. Trade‑offs and operational concerns

While exploring Caching as an option, we also need to discuss:

- **Consistency vs. Latency**
  - _Strongly Consistent_: `Write-Through`, valid cache always matches DB.
  - _Eventually Consistent_: `Read-Through(cache-aside)` with TTL, faster but may show stale data briefly.

- **Cache Size adn Eviction**
  - _Can't cache_ everything. (OOM, cost)
  - _Eviction Policies_:
    - `LRU`: Least recently used
    - `LFU`: Least frequently used
    - `TTL`: Time-To-Live
- **Cache stampede / dog‑piling**
  - When a popular key expires, hundreds of requests hit the DB at once.
  - _Mitigations_:
    - Soft Expiry and background refresh
    - Sharding keys
    - Using a lock/ semaphore per key

- **Observability**:
  - Monitor
    - Chache hit rate
    - Memory usage
    - Latency of cache vs. DB calls
  - Alerts if hit rate suddenly drops (e.g. After deployment, invalidation got broken)

## 4. Product Perspective

Given a service is hitting scalibility issues, questions we should ask:

- `What are the heavy read paths?`
  - Which endopints are causing DB poressure ?
  - Can we attach a Redis Cache before them >

- `What is the Data consistency requirement?`
  - Is it okay if a user sees a 10‑second‑old profile?
  - That would push us toward Cache‑Aside with TTL.
  - If not, we might consider Write‑Through or final‑write‑wins with versioned keys.

- `What is the write‑to‑read ratio?`
  - Mostly reads (e.g., product catalog, feeds): cache‑aside + CDN.
  - Mostly writes (e.g., analytics ingest): maybe bypass cache or use write‑behind.

- `Where can we layer the cache?`
  - Static assets → CDN.
  - Frequently read user / product data → Redis cluster per service.
  - Some global shared data → distributed cache (e.g., Redis‑cluster).
