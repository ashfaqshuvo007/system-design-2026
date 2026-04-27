# Checklist to stand-out:

## Repository Structure for Public Impact

A monorepo layout that tells a story:

```text

system-design-portfolio/
├── README.md # Overall portfolio summary (see template next section)
├── url-shortener/
│ ├── README.md # Project-specific design doc
│ ├── docker-compose.yml
│ ├── src/ # App code
│ └── .github/workflows/ # Or centralised workflows referencing this path
├── rate-limiter/
│ ├── ...
├── chat-service/
├── video-streaming/
├── ecommerce-checkout/
├── search-engine/
└── mfa-service/
```

## The “Interview‑Ready” README Template

For every project, use this structure:

````markdown
# System Design: [Name]

**Live Demo**: [link if deployed, e.g., via Render or fly.io]  
**CI Status**: [![Build & Test](https://github.com/.../actions/workflows/build-and-test.yml/badge.svg)](...)

## 1. Requirements & Scope

- Functional: ...
- Non-functional: (scale, latency, availability)

## 2. Capacity Estimation (Back‑of‑the‑Envelope)

- Reads/writes per second, storage, bandwidth.

## 3. High‑Level Design

![Architecture Diagram](architecture.png)

## 4. API Design

```http
POST /shorten ...
```
````

## Data Model

- Tables, indexes, partitioning strategy.

## Key Design Decisions & Trade‑offs

- Why Redis over in‑memory cache? (When scaling horizontally)

- Why eventual consistency for chat history? (Performance vs. strong consistency)

- Why gRPC for payment service? (Performance, strong typing)

## How to run locally:

For example,

```bash
docker compose up -d
```

## Scaling & Production Considerations

- How I’d replace Docker Compose with Kubernetes (example manifests in /k8s folder)

- CI/CD pipeline (link to workflow)

- Monitoring stack (Prometheus + Grafana screenshots)

## What I Learned/ Challenges

````text

**Pro tip**: Add an **architecture diagram** using Mermaid (in‑markdown) or an exported Excalidraw image. Mermaid is better because it stays versioned.

Example Mermaid diagram for URL shortener:
```mermaid
graph TD
    Client -->|REST| LB(Load Balancer)
    LB --> App1(App Instance 1)
    LB --> App2(App Instance 2)
    App1 --> Redis[(Redis Cache)]
    App2 --> Redis
    App1 --> DB[(PostgreSQL)]
    App2 --> DB
````

This immediately shows you think in diagrams – exactly what interviewers want.
