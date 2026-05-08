# REST, gRPC, GraphQL – when and why

**REST**: Resource‑oriented HTTP APIs with well‑understood semantics, strong tooling, and straightforward caching.

**GraphQL**: Query‑oriented facade that decouples client data needs from backend schema and endpoint granularity.

**gRPC**: Contract‑first, high‑performance RPC, especially suited for internal microservices and streaming scenarios.

We have three tools. Each solves a different scalibility trade-off.

## REST (Representational State Transfer)

- **How**: HTTP verbs (`GET`, `POST`, etc.) on resource URLs. Usually `JSON`.

- **When to use**:
  - Simple CRUD, public APIs, web caching (CDN friendly).
  - Early‑stage startup – low cognitive load, every language speaks `HTTP/JSON`.

- **Challenges**:
  - Over‑fetching / under‑fetching – client gets whole resource or needs multiple round trips (e.g. `/users/1`, then `/users/1/posts`).
  - No real‑time streaming (without WebSocket hacks).
  - Versioning – usually /v1/, /v2/ leads to bloat.

- **Scaling**:
  - Stateless → easy horizontal scaling.
  - Cache headers (`Cache-Control`, `ETag`) reduce load.
  - But chatty clients increase latency and server cost.

## gRPC (Google RPC)

- **How**:
  - HTTP/2 + Protocol Buffers (binary). Strongly typed .proto contracts.
  - Supports unary, server/client/bidirectional streaming.

- **When to use**:
  - Internal microservices (high throughput, low latency).
  - Polyglot environments – protobuf generates clients for 10+ languages.
  - Real‑time streaming (e.g. chat, live dashboards).

- **Challenges**:
  - Browser support – requires gRPC‑web (adds proxy overhead).
  - Binary payloads not human‑readable – harder to debug without tools (e.g. grpcurl).
  - Load balancing – HTTP/2 long‑lived connections; L4 balancers can break. Need client‑side awareness or service mesh (e.g. Envoy, linkerd).
  - Reflection is optional – poor discoverability compared to OpenAPI.

- **Scaling**:
  - Very efficient – protobuf serialization is faster & smaller than JSON.
  - Multiplexed streams over one TCP connection → saves handshakes.
  - Ideal for high‑RPS internal services.

## GraphQL

- **How**: Single endpoint (`/graphql`). Client sends a query specifying exact fields. Server resolves via resolvers (often to underlying `REST/gRPC/DB`).

- **When to use**:
  - Complex data graphs with many client types (mobile vs web vs IoT) – each asks only what it needs.
  - Rapid frontend iteration without backend changes.
  - Aggregating multiple backends (BFF pattern).

- **Challenges**:
  - `N+1 problem` – naive resolvers cause a query waterfall (e.g. fetch 100 users, then 100 DB calls for their posts). Solved with dataloader + batching.

  - Costly queries – malicious or dumb clients can request massive nested data. Need complexity analysis, depth limiting, persistent queries.

  - Caching is hard – most GraphQL implementations use `POST`, bypassing `HTTP cache`. Solutions: Apollo cache, CDN at query level (persisted queries), or `GET` for idempotent queries.
  - Schema complexity – stitching or federation adds ops overhead.

- **Scaling**:
  - Gateway becomes a bottleneck – must be horizontally scaled and efficient.

  - Resolvers should be fast – push heavy work to downstream services.

  - Works from startup (simple schema) to large scale (federation, e.g. Netflix, GitHub).

## Trade-off Summary:

| Aspect      |         REST         |                   gRPC                    |           GraphQL           |
| :---------- | :------------------: | :---------------------------------------: | :-------------------------: |
| Payload     |     JSON (text)      |             Protobuf (binary)             |         JSON (text)         |
| Transport   |    HTTP/1.1 or /2    |                  HTTP/2                   |       HTTP/1.1 or /2        |
| Browser     |        Native        |          Needs proxy (gRPC‑web)           |      Native (POST/GET)      |
| Streaming   |    No (SSE hack)     |               Bidirectional               |  No (subscriptions via WS)  |
| Cache       |     HTTP native      |               Not designed                |    Hard (needs gateway)     |
| DevEx       | Simple, tools common |          Codegen, strict typing           | Introspection, typed schema |
| Over‑fetch  |         Yes          | No (method = return exactly what you ask) |     No (fields select)      |
| Round trips |         Many         |                One (unary)                |             One             |

## Putting it together: startup → large scale

Here’s a compact guideline (focus: your system-design decisions):

| Phase / Concern                          | REST                                                                                  | GraphQL                                                                                                            | gRPC                                                                                         |
| ---------------------------------------- | ------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| Greenfield startup MVP                   | Default choice: quickest to build, easiest to hire for, great tooling and docs. tyk+2 | Usually overkill initially; adds infra and schema complexity. designgurus+1                                        | Overkill for external APIs, more ops work and tooling complexity. dev+2                      |
| Growing product with complex UI          | Still fine; may lead to many endpoints and client orchestration. systemdesignschool+1 | Strong candidate for client‑facing API to reduce round trips and accelerate front‑end iteration. designgurus+2     | Rare for clients; you’d still expose REST/GraphQL externally. devgenius+2                    |
| Microservices / internal traffic scaling | Works but less efficient; JSON and multiple calls add overhead at high scale. dev+1   | Often used as a thin aggregation layer; not ideal for deep internal service‑to‑service calls. designgurus+1        | Strong choice for service‑to‑service communication: performance, schema, streaming. dev+3    |
| Caching & CDNs                           | Very strong via HTTP semantics and path‑based keys. systemdesignschool+1              | Tricky; often done at app/gateway level, not edge CDNs, unless using GET + persisted queries. systemdesignschool+1 | Weak at HTTP level; cache in app or gateway. systemdesignschool+1                            |
| Evolving schema / versions               | Versioned endpoints, sometimes painful to clean up. systemdesignschool+1              | Excellent via field‑level deprecation and a single evolving schema. designgurus+1                                  | Proto evolution works well but still tied to RPC methods; versioning is more explicit. dev+1 |
