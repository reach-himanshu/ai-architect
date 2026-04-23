# System Design 03: Message Queues

## 1. When to Use Message Queues

Use a queue when:
*   Processing is too slow for synchronous response (ML inference, video encoding).
*   You need to decouple producers from consumers.
*   Traffic is bursty — queues absorb spikes.
*   Multiple services need the same event (fan-out).
*   You need guaranteed delivery even if the consumer is temporarily down.

---

## 2. Kafka vs RabbitMQ vs SQS

| Feature | Kafka | RabbitMQ | AWS SQS |
| :--- | :--- | :--- | :--- |
| **Retention** | Days/weeks (replay) | Until consumed | 4–14 days |
| **Ordering** | Per partition | Per queue | FIFO optional |
| **Throughput** | Millions/sec | ~50K/sec | ~3K–30K/sec |
| **Replay** | Yes (offsets) | No | No |
| **Ops burden** | High | Medium | Managed |
| **Best for** | Event streaming, ML pipelines | Task queues | AWS-native, serverless |

---

## 3. Kafka Concepts

*   **Topic**: Named stream of records — like a table that grows forever.
*   **Partition**: A topic is split into ordered, immutable partitions. More partitions = more parallelism.
*   **Offset**: Consumer's position within a partition — consumer controls when to commit.
*   **Consumer Group**: Multiple consumers share partition assignment — scale out processing.
*   **Replication Factor**: How many brokers hold copies of each partition.

**Key property**: Multiple consumer groups can read the same topic independently at their own pace.

---

## 4. Delivery Guarantees

| Guarantee | Description | Trade-off |
| :--- | :--- | :--- |
| **At-most-once** | May lose messages, never duplicates | Fastest, lowest durability |
| **At-least-once** | No loss, may duplicate | Good default with idempotent consumers |
| **Exactly-once** | No loss, no duplicates | Expensive; needs transactional producers |

**For AI pipelines**: At-least-once + idempotent consumers is the practical choice.

---

## 5. Dead Letter Queues (DLQ)

Messages that fail after N retries go to the DLQ:

```
Producer → Main Queue → Consumer (fails 3×) → DLQ → Alert + Manual Review
```

Always configure a DLQ. Without it, poison messages block the queue forever or are silently lost.

---

## 6. Backpressure

When consumers are slower than producers, the queue grows unboundedly.

Solutions:
*   **Scale out consumers** — add more consumer instances.
*   **Rate-limit producers** — slow the source.
*   **Shed load** — drop low-priority messages.
*   **Circuit breaker** — stop accepting new work until queue drains.
