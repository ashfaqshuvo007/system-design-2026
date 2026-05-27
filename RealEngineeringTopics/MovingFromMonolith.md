# Moving From Monolith

Monoliths become critical bottlenecks at scale:

- One bug = entire system down

- Deployment = deploy entire app (even for 1-line change)

- Scaling = scale everything (wasteful)

- Tech debt accumulates faster than you can fix it

Moving from a monolithic architecture—where all functionality is bundled together—into a microservices architecture isn’t a flip of a switch. It’s a complex journey that, if done recklessly, can cause more harm than good.

That’s why smart teams follow specific transition patterns. These patterns help reduce risk, maintain stability, and allow the system to evolve gradually rather than completely rebuild.

---

## 1. Strangler Fig Pattern

### How it Works

```text

                    ┌─────────────────────┐
                    │   API Gateway/Proxy │
                    │   (Routes requests) │
                    └──────────┬──────────┘
                               │
         ┌─────────────────────┼─────────────────────┐
         │                     │                     │
         ▼                     ▼                     ▼
 ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
 │  New Service 1│   │  New Service 2│   │  New Service 3│
 │  (User Auth)  │   │  (Payment)    │   │ (Inventory)   │
 └──────┬────────┘   └──────┬────────┘   └──────┬────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                    ┌───────▼───────┐
                    │   MONOLITH    │
                    │  (Legacy)     │
                    └───────────────┘

```

- Place an `API Gateway` in front of the Monolith.

- Route new features to new microservices.

- Route old features still to the monolith.

- Gradually migrate more features to services.

- When all functinalities are moved to microservices, decomission the monolith.

### Why it's Critical

- **Zero Downtime**: Users never see an outage

- **Incremental Risk**: If one service break, the rest works.

- **Team Autonomy**: Different teams can own different services.

- **Rollback easy**: Just change gateway routing.

### Real-Life Example: Netflix

Netflix migrated from a monolith to microservices over 7 years using Strangler Fig. They:

- Started by migrating non-critical features (recommendations)

- Gradually moved core features (streaming, user profiles)

- Used API Gateway to route traffic

- Achieved zero downtime during the entire 7-year migration

---

## 2. Parallel Run Pattern

**How it works**

```text

                    ┌─────────────┐
                    │   Client    │
                    └──────┬──────┘
                           │
           ┌───────────────┴───────────────┐
           │                               │
           ▼                               ▼
 ┌─────────────────┐           ┌─────────────────┐
 │   Monolith      │           │   New Service   │
 │(Source of Truth)│           │  (Async Check)  │
 └────────┬────────┘           └────────┬────────┘
          │                             │
          └───────────────┬─────────────┘
                          │
                  ┌───────▼────────┐
                  │ Comparison     │
                  │ Handler        │
                  │ (Detects       │
                  │  mismatches)   │
                  └────────────────┘


```

- Client request goes to monolith (fast path, no delay)

- Monolith asynchronously sends request to new service

- Comparison handler compares outputs from both systems

- If outputs match → safe to proceed

- If outputs differ → Slack alert, investigate

### Why It's Critical

- **Live production data testing**: No fake test data—real customer traffic

- **Data validation**: Catches edge cases you'd never write tests for

- **Easy rollback**: Feature flag lets you switch back instantly

- **Confidence metric**: You can measure "99.5% consistency" before full switch

### Limitations

- **Bespoke solution**: Building comparison logic takes time

- **Code mess**: Temporary feature flags and dual code paths

- **Double load**: System handles ~2x requests during parallel run

### Real-Life Example: Zalando Returns Migration

Zalando extracted returns logic from monolith to microservice:

- **Problem**: Old code had no tests, unclear logic, critical business function

- **Approach**: Monolith handled request → async call to new service → compare responses

- **Monitoring**: Prometheus + Grafana tracked consistency per endpoint

- **Result**: Each endpoint needed 95%+ consistency before switch. ~700 lines of cleanup code post-migration

---

## 3. Collaborator Pattern

**How it works**

```text
                    ┌─────────────────────┐
                    │    Client Request   │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │      MONOLITH       │
                    │   (Core Logic)      │
                    └──────────┬──────────┘
                               │
          ┌────────────────────┼────────────────────┐
          │                    │                    │
          ▼                    ▼                    ▼
 ┌───────────────┐   ┌───────────────┐   ┌───────────────┐
 │ Collaborator  │   │ Collaborator  │   │ Collaborator  │
 │  Service A    │   │  Service B    │   │  Service C    │
 │ (Email Notify)│   │ (Analytics)   │   │ (Res. Enrich) │
 └───────────────┘   └───────────────┘   └───────────────┘

```

- Monolith handles **core business logic** unchanged

- Collaborator services **wrap** the monolith

- Services **add new features** without touching monolith code

- Monolith triggers events → services listen and respond

### Why It's Critical

- **No core changes**: Don't touch risky legacy code

- **Extend functionality**: Add features without refactoring

- **Gradual decoupling**: Services can eventually take over core logic

- **Low risk**: Monolith stays as fallback

### Real-Life Example: E-commerce Order Processing

A company had a monolithic order system:

**Problem**: Needed email notifications + analytics tracking, but monolith was too risky to modify

**Solution**:

- Collaborator Service A: Listened to order events → sent emails

- Collaborator Service B: Tracked analytics in data warehouse

- Collaborator Service C: Enriched API responses with recommendation data

- **Result**: New features shipped in weeks, monolith untouched, zero risk to core

---

## 4. Change Data Capture (CDC)

**How it works**

```text
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  Source Database│────►│   CDC Log       │────►│  Message Queue  │
│  (MySQL/PG)     │     │  Reader         │     │  (Kafka)        │
└─────────────────┘     └─────────────────┘     └────────┬────────┘
           │                                             │
           │  INSERT/UPDATE/DELETE                       │
           │  (Captured from WAL/binlog)                 │
           └─────────────────────────────────────────────┘
                                                       │
                    ┌──────────────────┬───────────────┼───────────────┐
                    │                  │               │               │
                    ▼                  ▼               ▼               ▼
         ┌────────────────┐  ┌───────────────┐  ┌──────────┐  ┌──────────┐
         │ Downstream     │  │ Downstream    │  │ Cache    │  │ Analytics│
         │ Service 1      │  │ Service 2     │  │ Update   │  │ Pipeline │
         │ (Search Index) │  │(Notifications)│  │          │  │          │
         └────────────────┘  └───────────────┘  └──────────┘  └──────────┘
```

- CDC tool (Debezium/Canal) `reads database transaction` log (WAL/binlog)

- Captures `INSERT/UPDATE/DELETE` in real-time

- Publishes `changes to message queue` (Kafka)

- Downstream services `consume events and update` their data stores

### Why It's Critical

- **Real-time sync**: No batch delays, data stays consistent

- **Zero impact on source**: Reading from log, not querying tables

- **Event sourcing**: Services get immutable change history

- **Decouples data ownership**: Each service owns its data, no shared DB.

### Real-Life Example: E-commerce Search Index Migration

A company moved from monolithic DB to microservices:

**Problem**: Search index out of sync, slow queries affecting checkout

**CDC solution**:

- Debezium captured product changes from MySQL binlog

- Kafka streamed events to Search Service

- Search Service rebuilt Elasticsearch index in real-time

- **Result**: Search always in sync, 60% faster checkout (no DB joins), zero downtime migration

---

## Pattern Selection Cheat Sheet

| Scenario                                             | Best Pattern  | Why                                       |
| ---------------------------------------------------- | ------------- | ----------------------------------------- |
| Migrating large monolith with no downtime allowed    | Strangler Fig | Gradual replacement, API Gateway routing  |
| High-risk migration, unclear legacy behavior         | Parallel Run  | Live data validation, confidence metrics  |
| Need new features but can't touch legacy code        | Collaborator  | Augment without modifying core            |
| Multiple services need same data, avoiding shared DB | CDC           | Real-time sync, event-driven architecture |

## Key Takeaways

As an engineer leading migrations:

- Start with Strangler Fig for most migrations—it's the safest default

- Use Parallel Run when the business risk is high (e.g., payment processing)

- Choose Collaborator when you need features fast but can't refactor

- Apply CDC when you're breaking data ownership boundaries

Critical success factors:

- Feature flags for instant rollback

- Comprehensive monitoring at every step

- Per-endpoint consistency thresholds (not one-size-fits-all)

- Plan cleanup: temporary code (comparison handlers, feature flags) must be removed
