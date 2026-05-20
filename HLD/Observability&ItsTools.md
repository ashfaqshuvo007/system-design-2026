# Observability & The Real-world Tools

Think of observability as three different questions:

- **Logging:** “What happened?”
- **Metrics:** “How often is it happening, and how bad is it?”
- **Tracing:** “Where did the request spend time?”

Logs are detailed event records, metrics are numeric time-series measurements, and traces show the path of one request across services.
Prometheus is a time-series monitoring and alerting toolkit, Jaeger is a distributed tracing system, and Elastic Stack is commonly used to store/search logs and related operational data.

---

## Logging

Logs are best for debugging specific failures and understanding the exact sequence of events. In Elastic Stack, logs can cover application activity, deprecation warnings, audit events, slow queries, and other operational details.

Use logging when you need context, such as:

- A request failed for one user.
- A service returned an unexpected error.
- You need to inspect the exact input or error path.

The main downside is volume: logs grow fast, cost money to store, and become noisy if they are not structured well.

---

## Metrics

Metrics are lightweight numbers recorded over time. Prometheus stores them as time series data with timestamps and labels, and its core metric types are counter, gauge, histogram, and summary.

Use metrics when you need:

- System health and trends.
- Alerting on failures or latency.
- Dashboards for traffic, errors, and saturation.

A common example is HTTP request latency, which Prometheus recommends modeling with histograms when you want aggregation across instances.

---

## Tracing

Tracing follows a single request across multiple services. Jaeger is built for distributed tracing in microservices systems and helps troubleshoot where latency or errors happen in a request path.

Use tracing when:

- A request crosses several services.
- You want to find the slow hop.
- You need a causal path, not just separate logs.

The **trade-off is overhead**: tracing requires instrumentation, propagation of trace context, sampling decisions, and storage for spans.

---

## When to use what

As developers, the key is not just knowing the tools, but knowing **why** each exists and what problem it solves in production.

A simple rule:

- **Logs** for deep debugging and audits.
- **Metrics** for alerting and service health.
- **Tracing** for request-path analysis in distributed systems.

In a monolith, metrics and logs may be enough. In microservices, tracing becomes much more valuable because one user request may touch many services, queues, and databases.

---

## Real-world implications

In production:

- **Observability** affects incident response, SLO tracking, and capacity planning.
- **Prometheus-style** metrics are great for dashboards and alerts because they are cheap to aggregate and query over time.
- **Logs** are best when you need the exact failure context, but they are expensive at scale and can become a privacy risk if you log sensitive data.
- **Tracing** helps identify latency bottlenecks across service boundaries, but sampling and context propagation need to be designed carefully.

Elastic Stack is often used for centralized log search and operational visibility, while Prometheus is commonly used for metrics scraping and alerting, and Jaeger for tracing.

---

## Common Trade-offs

| Area    | Strength                          | Cost / Risk                                         |
| ------- | --------------------------------- | --------------------------------------------------- |
| Logs    | Rich context for debugging        | High volume, expensive storage, sensitive data risk |
| Metrics | Cheap to aggregate and alert on   | Limited context, can hide root cause                |
| Traces  | Best for distributed request flow | Instrumentation overhead, sampling complexity       |

Prometheus histograms are useful for latency because they can be aggregated across multiple instances, while summaries are less suitable for cross-instance aggregation.

---

## Best practices

- Use **structured logs** with consistent fields like `request_id`, `user_id`, `service`, `latency`, and `error_code`.
- Keep logs actionable; avoid dumping entire objects or secrets.
- Prefer **counters** for totals, **gauges** for current values, and **histograms** for latency distributions.
- Propagate a correlation ID or trace ID through every service hop.
- Sample traces intelligently; full tracing for every request is often too expensive.
- Set alerts on symptoms, not on every raw error log.

---

## Interview angle

In a system design interview, a candidate who understands observability should be able to answer questions like:

- What signals would you add for a new microservice?
- How would you debug a latency spike across five services?
- What would you log, and what would you never log?
- Why use a histogram instead of a summary for request latency in Prometheus?
- How would you reduce log volume without losing debuggability?
- When would tracing be unnecessary?
- How would you correlate logs, metrics, and traces for the same request?

A strong answer usually shows this structure:

1. Define the problem.
2. Pick the right signal.
3. Explain storage and overhead.
4. Mention operational trade-offs.
5. Describe how it helps during incidents.

---

## Practical mental model

If you remember only one thing, remember this:

- **Metrics tell you something is wrong.**
- **Logs tell you what happened.**
- **Tracing tells you where it happened.**

That is the simplest production-ready way to think about observability in a MAANG-style environment.
