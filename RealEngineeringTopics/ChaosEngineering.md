# Chaos Engineering: System Design Refresher

## What Is Chaos Engineering?

**Chaos engineering** is the practice of intentionally injecting faults into a system to test its resilience and identify weaknesses before they cause real outages.

- **For a junior engineer**: Think of it as "deliberately breaking things on purpose" in a controlled way to learn what happens when your system fails.
- **For you (mid-level)**: It's a proactive SRE discipline that experiments on distributed systems to build confidence they can withstand turbulent production conditions.

---

## How Chaos Engineering Is Handled

Chaos engineering follows **five core principles**:

| Principle                            | What It Means                                                  |
| ------------------------------------ | -------------------------------------------------------------- |
| Build hypothesis around steady state | Define "normal" metrics (latency, throughput) before testing   |
| Replicate real-world conditions      | Simulate realistic failures (network outages, server crashes)  |
| Run experiments in production        | Only production with real traffic reveals true resiliency      |
| Automate experiments                 | Make it continuous (CI/CD integration), not one-off            |
| Determine blast radius               | Minimize customer impact through tiered service categorization |

---

## Steps to Execute a Chaos Experiment

Follow this structured workflow:

1. **Define steady state** — Identify baseline metrics (e.g., request latency < 200ms, 99.9% success rate)
2. **Formulate hypothesis** — Single testable statement: _"If we kill this pod, user login won't be affected"_
3. **Start in controlled environment** — Begin in DEV, graduate to UAT, then PROD.
4. **Inject failures** — Delete VMs, stop databases, add firewall rules, simulate network latency
5. **Automate execution** — Use tools integrated into CI/CD with automated rollback
6. **Derive actionable insights** — Analyze results, fix vulnerabilities, document findings

---

## How This Helps in System Design

| Benefit                         | Impact on Design                                                           |
| ------------------------------- | -------------------------------------------------------------------------- |
| **Identifies weak spots**       | Proactively fixes vulnerabilities before real outages                      |
| **Validates fault tolerance**   | Confirms system recovers quickly without service disruption                |
| **Reduces MTTR**                | Teams with chaos engineering reduce mean time to resolution by 23% to <1hr |
| **Improves availability**       | Frequent practitioners achieve >99.9% availability                         |
| **Prepares incident response**  | Refines detection, diagnosis, and remediation processes                    |
| **Reveals system interactions** | Exposes how components interact under stress in complex architectures      |

---

## Real-Life Examples

### Netflix (Chaos Monkey)

- **What**: Randomly kills production servers during business hours
- **Why**: Ensures high availability in streaming services by forcing automatic failover
- **Outcome**: Engineers must design for failure from day one

### Upwork

- **What**:
  - RDS failover simulation using AWS tools
  - Container shutdowns with automatic traffic routing
  - Controlled traffic spikes for surge testing
- **Why**: Supports globally distributed infrastructure connecting freelancers and businesses

### Common Failure Scenarios to Test

- Server crashes
- Network outages
- Database failures
- Dependency timeouts
- CPU/memory exhaustion

---

## Tradeoffs During System Design

When designing for chaos, consider these tradeoffs:

| Tradeoff                            | Consideration                                                                                  |
| ----------------------------------- | ---------------------------------------------------------------------------------------------- |
| **Complexity vs. Resilience**       | Adding retry logic, circuit breakers, and redundancies increases code complexity               |
| **Cost vs. Availability**           | Running redundant instances across zones increases cloud costs but improves uptime             |
| **Performance vs. Fault Tolerance** | Health checks and failover mechanisms add latency overhead                                     |
| **Speed vs. Safety**                | Running experiments in production risks customer impact; non-prod misses real-world conditions |
| **Automation vs. Control**          | Fully automated chaos may cause unexpected blast radius; manual gates slow iteration           |

---

## Key Takeaway for Your Role

As a mid-level backend engineer working with API gateways, Kubernetes, and cloud-native systems [user_background], chaos engineering should be **baked into your architecture decisions**, not added later. Design for failure assumption: every dependency can fail, every network can partition, every instance can die. Your system must recover gracefully without user impact.

Start small: run a chaos experiment on your staging Kubernetes cluster by killing random pods and observing if your nginx load balancer reroutes traffic correctly.
