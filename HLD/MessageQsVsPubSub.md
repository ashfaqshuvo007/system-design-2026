# Message queues (Kafka, RabbitMQ) vs. pub/sub

**A message queue** is usually used for task processing. A producer puts a job into a queue, and one consumer takes it and handles it; if you use multiple consumers, they compete for messages so each message is processed by only one consumer.

**Pub/sub** is used for event distribution. A producer publishes an event to a topic, and all subscribers to that topic can receive it independently. Kafka often looks like pub/sub from the outside, but its partitions and consumer groups make it strong for high-throughput streaming and load-balanced consumption.

## Question: Work Distribution or Event Fan-out?

- **Use a queue when**:
  - A task should be processed by only one worker.

  - You need retry, backpressure, or a clear work backlog.

  - The workload is operational, like image resizing, email sending, or payment follow-up.

- **Use pub/sub when**:
  - The same event must go to multiple services.
  - You want loose coupling between producers and consumers.
  - You are building event-driven flows like notifications, analytics, cache invalidation, or audit pipelines.

## Trade-offs

| Area             | Message queue                                          | Pub/sub                                                  |
| ---------------- | ------------------------------------------------------ | -------------------------------------------------------- |
| Delivery model   | One message to one consumer                            | One message to many subscribers automq+1                 |
| Main strength    | Reliable work distribution                             | Broad event distribution confluent+1                     |
| Scaling pattern  | Competing consumers                                    | Many independent subscribers confluent+1                 |
| Failure handling | Usually easier to reason about per task                | Harder when many downstream systems fail differently     |
| Replay           | Depends on product; classic queues are often transient | Kafka-style systems support replay well confluentyoutube |

## Challenges

- **Queues**:
  - Hot partitions or one slow worker can create backlog.
  - You need idempotency because retries can cause duplicate processing.
  - Ordering is tricky once you scale horizontally.

- **Pub/Sub**:
  - One slow subscriber can become a bottleneck.
  - Schema changes can break many consumers at once.
  - Fan-out can amplify load and cost when one event triggers many downstream actions.

## Kafka vs RabbitMQ

- **RabbitMQ** is usually the better fit when you want classic queue semantics, flexible routing via exchanges, acknowledgments, and task-oriented message handling.

- **Kafka** is better when you want high-throughput event streaming, retention, replay, and multiple consumer groups reading the same event stream.

Kafka is not “just pub/sub”; it is a distributed log with pub/sub-style consumption layered on top, which is why it scales well for streaming use cases.

## Simple Rule for Choice

- If the question is “_Who should do this work?_”, use a message queue.
- If the question is “_Who should know this happened?_”, use pub/sub.
