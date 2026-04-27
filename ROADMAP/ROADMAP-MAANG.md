# 🗓 Refined Roadmap for Mid‑Size Companies

For mid‑size companies targeting intermediate‑to‑senior backend roles, the interview system‑design round typically asks you to design a realistic feature (e.g., “Design a URL shortener like Bitly”, “Design an API rate limiter”) and then talk about how you would scale it, deploy it, monitor it, and handle failures. Your experience with Docker, GitHub‑based CI/CD, and basic Kubernetes gives you a solid foundation to demonstrate that you can build and ship—not just whiteboard.

## 🗓 Refined 8‑Week Roadmap (2 h/day)

Each week focuses on one classic system‑design interview question. You’ll design it first, then code the core components, containerise them, and set up a CI/CD pipeline using GitHub Actions. As you progress, you’ll layer on scaling concepts (caching, sharding, queues, consistency) so that by week 8 you can confidently discuss how each piece changes under load.

---

### Week 1: URL Shortener – Scalability, Caching, and CI/CD Foundation

**Interview Question**: “Design a URL shortener like TinyURL.”

- **Days 1-2**: Design the system (REST API, DB schema, hash function). Read Xu’s Volume 1 chapter.

- **Days 3-4**: Implement the core API in Java/Python with PostgreSQL. Write Dockerfile and a docker-compose.yml (app + DB). Containerise everything.

- **Day 5**: Add Redis cache for read‑heavy short URLs.

- **Day 6**: Set up a GitHub Actions workflow that builds the Docker image on push, runs tests, and (simulate) pushes to a registry.

- **Day 7**: Write README: explain trade‑offs (e.g., why Base62 vs. hash collision, cache invalidation).

**Key CI/CD takeaway**: You’ll have a pipeline that builds and tests every commit—this is a strong talking point in mid‑size company interviews.

---

### Week 2: Distributed Rate Limiter – Algorithms, Sorted Sets, and Middleware

**Interview Question:** “Design a rate limiter for an API.”

- **Days 1-2**: Design (token bucket, sliding window log). Learn how Redis sorted sets can implement a sliding window.

- **Days 3-5**: Implement a rate‑limiting middleware that uses Redis (Python/Flask or Java/Spring). Containerise.

- **Day 6**: Add CI pipeline with integration tests (spin up Redis container in CI).

- **Day 7**: Write README explaining how you’d scale it (consistent hashing, Lua scripts, sticky sessions).

---

### Week 3: Real‑time Chat Service – WebSockets, Message Queues, and Event‑Driven

**Interview Question:** “Design a chat system (WhatsApp/Messenger).”

- **Days 1-2**: High‑level design (client‑server, WebSocket connection manager, message queue, DB). Read Xu’s chat chapter.

- **Days 3-5**: Implement a simple backend with a WebSocket server, RabbitMQ for async message delivery, and PostgreSQL for history. Containerise all services with Docker Compose.

- **Day 6**: Add CI pipeline; include a smoke test that simulates a client connection.

- **Day 7**: Add idempotency keys and exactly-once delivery semantics; document trade‑offs.

---

### Week 4: Video/Movie Streaming Platform (Simplified) – CDN, Chunked Storage, and Packaging

**Interview Question:** “Design a video streaming service like YouTube.”

- **Days 1-2**: Focus on storage, transcoding, and CDN. Understand chunked upload, adaptive bitrate streaming.

- **Days 3-5**: Build a minimal API to accept video files, store chunks in object storage (MinIO for local S3‑like), and return a manifest. Use FFmpeg in a sidecar container for a simulated transcode.

- **Day 6**: CI pipeline + documentation.

- **Day 7**: Discuss scaling: how you’d use AWS S3, CloudFront, and offload transcoding to a job queue (K8s job or AWS Batch).

---

### Week 5: E‑commerce Checkout – Microservices, Saga, and gRPC

**Interview Question:** “Design an order placement system for an e‑commerce site.”

- **Days 1-2**: Design microservices (catalog, cart, order, payment) and the Saga choreography pattern for distributed transactions.

- **Days 3-5**: Implement two services (order and payment) using gRPC; containerise; run them in Docker Compose. Simulate a happy‑path and a compensation flow.

- **Day 6**: CI pipeline; add GitHub Actions job to start the services, run integration tests, and tear down.

- **Day 7**: Write about idempotency, outbox pattern, and how you’d monitor sagas.

---

### Week 6: Search Engine (Simplified) – Inverted Index, Crawler, and Ranking

**Interview Question:** “Design a web crawler and search engine.”

- **Days 1-2**: Design the components: crawler, indexer, querier, ranking. Use Xu’s book for reference.

- **Days 3-5**: Build a single‑process search engine: a toy web crawler (Python), an inverted index builder, and a simple query API with TF‑IDF ranking. Containerise everything.

- **Day 6**: CI pipeline + add Prometheus metrics (e.g., crawl duration, index size) and a Grafana dashboard via Docker Compose.

- **Day 7**: Discuss sharding the index, distributed crawling, and eventual consistency.

---

### Week 7: Multi‑Factor Authentication (MFA) Service – Security, JWT, and Observability

**Interview Question:** “Design an API authentication service with OAuth2/JWT and MFA.”

- **Days 1-2**: Design the token generation, TOTP flow, and rate limiting.

- **Days 3-5**: Implement JWT issuance/validation, TOTP generation (libs), and a rate‑limiter (from week 2). Containerise; integrate with Prometheus/Grafana for token‑request metrics.

- **Day 6**: CI pipeline; security scanning (e.g., trivy for Docker images).

- **Day 7**: Write about security best practices, storing secrets, and horizontal scaling of auth services.

---

### Week 8: System‑Design Mock Interviews & Refinement

- **Days 1-4**: Every day, pick a classic problem (design Twitter, Uber, a file storage service, a notification system) and talk through it aloud for 35–40 minutes, draw diagrams on a whiteboard (or Excalidraw), then code a skeleton API. Record yourself and review.

- **Days 5-7**: Polish the GitHub repository with all seven projects. Write a SYSTEM_DESIGN.md summarising your approach for each and linking to the code. This becomes a portfolio you can share with recruiters.
