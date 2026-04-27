# System Design Interview 2026

Collection of topics, codebases, system designs and roadmap for upcoming System Design interviews

---

## 📑 Table of Contents

- [📋 Quick Start](#quick-start)
- [🗓️ Roadmap](#roadmap)
- [📚 Documentation](#documentation)
- [🎯 Career Preparation](#career-preparation)
- [🔧 Technical Stack](#technical-stack)
- [📋 Resources](#resources)

---

## Quick Start

Start with the **[8-Week Practical Roadmap](ROADMAP.md)** for your learning path. Track your progress using the provided table. For detailed implementation guidelines, see the [Checklist](CHECKLIST.md).

---

## Roadmap

### 📅 Learning Path

| Resource | Purpose |
|----------|---------|
| [Main Roadmap](ROADMAP.md) | 8-week curriculum with hands-on projects and progress tracker |
| [MAANG Roadmap](ROADMAP/ROADMAP-MAANG.md) | Advanced track for FAANG-level system design challenges |

---

## Documentation

### 🏗️ System Design References

| Document | Content |
|----------|---------|
| [High-Level Design (HLD)](HLD/HLD.md) | Architectural patterns, system components, and design principles |
| [Low-Level Design (LLD)](LLD/LLD.md) | Detailed implementation patterns and object-oriented design |
| [Checklist](CHECKLIST.md) | Repository structure best practices and interview-ready templates |

---

## Career Preparation

### 🎯 Target Market Context

#### Navigating the 2025–2026 US Market

The US tech job market has become a tale of two narratives. While AI-driven layoffs are real — 55% of hiring managers expect further cuts in 2026 — total software-engineering job openings have actually rebounded sharply, reaching their highest level in over three years (67,000+ openings).

**For a backend engineer with your profile** (2-3 years of experience, full-stack & cloud exposure, Linux comfort), the key to standing out is to show that you can design systems end-to-end, not just code them.

System-design interviews are where many candidates stumble, but they are also where you can differentiate yourself. This framework is grounded in Alex Xu's books and verified community roadmaps, built for your 6-8 week, 2-hour-per-day schedule and your preference for practical implementation.

#### 💼 Mid-Size Company Interview Preparation

Mid-size companies (100-2000 employees) often ask take-home system designs or live whiteboarding focused on pragmatic trade-offs.

---

## Technical Stack

### 🔧 Integrating Your Existing Skills

**Containerization & Orchestration:**
- Docker & Docker Compose will be your default for every project. This signals that you think in terms of reproducible environments.
- By Weeks 5–6, experiment with Kubernetes (using `kind` locally). For mid-size companies, solid Docker Compose + CI is often sufficient; K8s is a bonus.

**CI/CD Pipeline:**
- GitHub Actions for every project. This demonstrates your ability to automate testing and deployment.

**Cloud Operations (AWS/GCP/Azure):**
- In each project README, explicitly map local services to managed equivalents:
  - Docker Redis → AWS ElastiCache
  - PostgreSQL → RDS
  - RabbitMQ → Amazon MQ / SQS
- This demonstrates cloud-agnostic thinking and readiness for production deployment.

### 🚀 Capabilities You'll Develop

By the end of this roadmap, you'll be able to:

1. **Design systems on the fly** using the Xu framework (requirements → capacity estimation → API → data model → HLD → deep dives)
2. **Point to working code** with CI/CD pipelines, proving you can deliver
3. **Discuss scaling bottlenecks** (caching, sharding, async processing) with concrete examples
4. **Answer follow-up questions confidently:**
   - "How would you deploy this?"
   - "How do you handle an outage?"
   - "How would you monitor this?"
   - Because you've already containerized and instrumented everything

---

## Resources

### 📖 Alex Xu's Framework

- **Volume 1** provides the step-by-step framework for solving any system-design question. As you complete each project, go back to the relevant chapter (e.g., after building the URL shortener, read Xu's solution) to compare approaches.

- **Volume 2** adds advanced patterns like geospatial indexing and real-time collaboration. Use it for Weeks 6–8 when you tackle complex systems.

- **Object-Oriented Design Interview** contains 11 real LLD problems with solutions. Pick one problem per week and implement it before reading the solution.

### 🌟 Stand Out in the US Market

1. **AI/ML Integration** – Add a simple AI feature to one of your projects. This demonstrates AI-first thinking, which companies value.

2. **Cloud-Native Depth** – Earn an Associate-level certification (AWS Solutions Architect, GCP Professional, etc.). It's a visible signal of cloud proficiency.

3. **Behavioral Preparation** – Prepare 5–7 STAR stories that highlight impact, trade-offs, and collaboration.

4. **Network Strategically** – Focus on mid-size companies and startups that hire intermediate backend engineers.

---

**For best practices on repository structure and interview-ready documentation**, see [Checklist.md](CHECKLIST.md).
