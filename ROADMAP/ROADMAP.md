# 🗓 8‑Week Practical Roadmap (2 Hours/Day)

Each week pairs theory with a hands‑on project. Build with Java/Python + AWS (or GCP) to leverage your existing skills.

## Progress Tracker

| Week | Focus Area                           | Hands-On Project                                                                                                                                                             | Status                                   |
| ---- | ------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| 1    | Scalability & Caching                | **Distributed URL Shortener** – implement a REST URL shortener with a cache (Redis), then scale it horizontally behind a load balancer.                                      | <font color="yellow">In Progress </font> |
| 2    | Databases & Sharding                 | **Multi‑tenant Blog Platform** – design a schema that shards posts by user‑ID; practice write‑through caching and read‑replicas.                                             | Pending                                  |
| 3    | APIs & Message Queues                | **Real‑time Chat Backend** – use WebSockets for real‑time messaging, RabbitMQ for async message delivery, and store chat history in PostgreSQL.                              | Pending                                  |
| 4    | Distributed Systems (CAP, Consensus) | **Key‑Value Store** – implement a simple eventually‑consistent key‑value store with hinted handoff and read‑repair; simulate network partitions and observe behavior.        | Pending                                  |
| 5    | Microservices & Patterns             | **E‑Commerce Order Service** – split into microservices (catalog, cart, order, payment) communicating via gRPC; implement the Saga pattern for order checkout.               | Pending                                  |
| 6    | Search & Data Pipelines              | **Custom Search Engine** – build a web crawler, inverted index, and a query parser; return ranked results. This touches both algorithms and system design.                   | Pending                                  |
| 7    | Security & Observability             | **Multi‑Factor Auth (MFA) Service** – implement TOTP generation, JWT‑based API authentication, rate limiting, and expose metrics (Prometheus + Grafana).                     | Pending                                  |
| 8    | Mock Interviews & Refinement         | **Mock HLD + LLD** – each day pick a classic problem (e.g., design Twitter, Uber, parking lot) and draw the architecture, then explain it aloud. Record yourself and review. | Pending                                  |

**Tip:** For each project, write a short README that explains the trade‑offs you made (e.g., why eventual consistency over strong consistency, why Kafka over RabbitMQ). This practice directly mirrors interview expectations.

## 🧠 How to Leverage Alex Xu’s Books

- Volume 1 provides the step‑by‑step framework for solving any system‑design question. As you complete each project, go back to the relevant chapter (e.g., after building the URL shortener, read Xu’s solution) to compare approaches.

- Volume 2 adds advanced patterns like geospatial indexing and real‑time collaboration. Use it for Weeks 6–8 when you tackle complex systems.

- Object‑Oriented Design Interview (Xu’s newest book) contains 11 real LLD problems with solutions. Pick one problem per week (e.g., Week 3: elevator system; Week 5: restaurant management) and implement it before reading the solution.

## 🚀 Additional Ways to Stand Out in the US Market

- **1. AI/ML Integration** – add a simple AI feature to one of your projects (e.g., use a pre‑trained model to classify chat messages or detect anomalies). This demonstrates AI‑first thinking, which companies value.

- **2. Cloud‑Native Depth** – since you already know AWS/GCP/Azure, pick one and earn an Associate‑level certification (e.g., AWS Solutions Architect). It’s a visible signal of cloud proficiency.

- **3. Behavioral Preparation** – US interviews place heavy weight on “STAR” stories. Prepare 5–7 stories that highlight impact, trade‑offs, and collaboration.

- **4. Network Strategically** – focus on mid‑size companies and startups that hire intermediate backend engineers; they often have less intense system‑design bars than FAANG but still value the skill.
