# Disaster Recovery

Disaster recovery is the part of reliability engineering that answers one question: **what happens when a whole system, site, or region is lost**? For a junior engineer, think of it as “how do we get the service back after a major outage without losing too much data or time?” For a mid-level engineer, you also need to reason about recovery time, recovery point, failure domains, operational complexity, and cost trade-offs.

## Core idea

Disaster recovery usually means preparing for failures bigger than a single server or pod failure: a datacenter outage, cloud region loss, storage corruption, bad deploys, or a large operator mistake. The goal is to restore service within an acceptable **RTO** and data loss within an acceptable **RPO**. RTO is how long the service can be down; RPO is how much data you can afford to lose.

## Main recovery types

### 1. Backup-based recovery

This is the simplest approach: you keep copies of data and restore from them after a disaster. It is usually the cheapest option and works well when downtime is acceptable and data loss can be limited to the last backup window.

Use it when:

- The system can tolerate slower recovery.
- Data is important, but not every second of it must be preserved.
- You want protection against accidental deletion, corruption, or ransomware.

Trade-offs:

- Restore can be slow.
- You may lose data since the last backup.
- You must test restore procedures, not just the backups.

### 2. Replication-based recovery

Replication keeps data copied to another node, zone, or region in near real time. It improves availability and reduces recovery time because a secondary copy already exists and can take over faster.

Use it when:

- You need lower RTO than backup-only recovery.
- The service is user-facing and downtime is costly.
- You can afford the operational and storage overhead.

Trade-offs:

- More complexity than backups.
- Depending on sync vs async replication, you may trade latency for durability or risk data loss.
- Replication alone does not protect you from bad writes being replicated everywhere.

### 3. Multi-region failover

This is the highest-availability approach: the system can shift traffic from one region to another if an entire region fails. It is often paired with replication and traffic management.

Use it when:

- The product has strict availability needs.
- Regional outage would be a major business event.
- You need geographic resilience for compliance or user experience.

Trade-offs:

- Highest cost and complexity.
- Harder to keep data consistent across regions.
- Failover itself can be error-prone and must be rehearsed.

## Practical differences

| Approach              |           RTO |           RPO |   Cost | Complexity | Typical use                           |
| --------------------- | ------------: | ------------: | -----: | ---------: | ------------------------------------- |
| Backup                |          High |        Higher |    Low |        Low | Internal tools, non-critical data     |
| Replication           | Medium to low | Low to medium | Medium |     Medium | Core services needing faster recovery |
| Multi-region failover |           Low |           Low |   High |       High | Large-scale customer-facing systems   |

## Real-world implications

Disaster recovery is not just about technology. It affects business continuity, support load, customer trust, compliance, and revenue. A system that is “technically backed up” but takes 12 hours to restore may still be unacceptable for a checkout system, messaging platform, or authentication service.

It also changes how you design everything:

- Databases need replication and backup strategy.
- Deployments need rollback and data migration safety.
- DNS, load balancers, and traffic routing need failover logic.
- Runbooks and on-call response need to be clear.
- Recovery has to be practiced through game days or drills.

## Implementation challenges

The hard parts are usually not the happy path.

- **Consistency vs availability.** Strong consistency across regions is expensive and can increase latency.
- **Failover correctness.** Switching traffic is easy to plan and hard to do safely.
- **Split brain.** Two regions may both think they are primary if coordination fails.
- **Data corruption propagation.** Replication can copy bad data quickly.
- **Testing recovery.** Many teams discover problems only during real incidents.
- **Operational burden.** More replicas and regions mean more monitoring, more automation, and more failure modes.

## When to choose what

Choose **backup-first** when the system is not highly time-sensitive and you mainly need recoverability. Choose **replication** when downtime matters and you need a warm standby or faster recovery. Choose **multi-region failover** when the business impact of regional loss is high enough to justify the cost and complexity.

A useful rule: if losing hours of service is acceptable, backups may be enough; if minutes matter, replication starts making sense; if a regional outage is unacceptable, you need multi-region design.

## Interview signals

In a system design interview, strong candidates do not just say “use multi-region.” They explain the failure model and tie it to business requirements.

Questions you might be asked:

- What are your RTO and RPO targets?
- What failures are you designing for: node, zone, region, or operator error?
- Is replication synchronous or asynchronous, and why?
- How do you prevent split brain during failover?
- How do you test restore and failover regularly?
- What data can tolerate eventual consistency?
- How do backups protect you from logical corruption and bad deploys?
- What happens to user requests during failover?

## Junior vs mid-level thinking

As a junior engineer, focus on understanding that backups restore data, replication keeps a live copy, and multi-region failover survives broader outages. Also remember that all three are about matching protection level to business need, not picking the fanciest option.

As a mid-level engineer, you should discuss failure domains, RTO/RPO, consistency trade-offs, operational automation, and incident readiness. You should also be able to explain why a cheaper design may be better if the business can tolerate longer recovery, and why a more expensive design is justified for critical systems.

The key idea is simple: **reliability is not free, and disaster recovery is about buying the right amount of resilience for the actual risk**.
