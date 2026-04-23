# System Design 01: Distributed Systems Fundamentals

## 1. CAP Theorem

In a distributed system, you can guarantee at most 2 of 3:
*   **Consistency**: All nodes see the same data at the same time.
*   **Availability**: Every request gets a response (not guaranteed to be the latest).
*   **Partition Tolerance**: System works despite network partitions.

Since network partitions are unavoidable in practice, the real choice is **CP vs AP**:
*   **CP systems**: PostgreSQL, HBase, Zookeeper — sacrifice availability during partitions.
*   **AP systems**: Cassandra, CouchDB, DynamoDB — sacrifice consistency during partitions.

---

## 2. Consistency Models

| Model | Guarantee | Example |
| :--- | :--- | :--- |
| **Strong / Linearizable** | Read always returns latest write | Single-node DB, RDBMS |
| **Sequential** | All nodes see operations in same order | Cassandra quorum reads |
| **Causal** | Causally related ops seen in order | Some distributed KV stores |
| **Eventual** | All replicas converge eventually | DynamoDB, Cassandra default |
| **Read-your-writes** | Writer always sees their own writes | Session-level guarantee |

---

## 3. Replication Strategies

**Single-leader (Master-replica)**:
*   All writes to leader, reads from any replica.
*   Simple; used by PostgreSQL, MySQL.
*   Leader is a bottleneck and single point of failure.

**Multi-leader**:
*   Multiple nodes accept writes — useful for multi-datacenter.
*   Conflict resolution is complex (Last-Write-Wins, CRDTs).

**Leaderless (Dynamo-style)**:
*   Any node accepts reads and writes.
*   Quorum: W + R > N guarantees consistency.
*   Used by Cassandra, DynamoDB.

---

## 4. Distributed Transactions

**Two-Phase Commit (2PC)**:
1.  Coordinator asks all participants to PREPARE.
2.  If all PREPARED → COMMIT; otherwise ABORT.
*   Problem: Coordinator failure leaves participants blocked forever.

**Saga Pattern**:
*   Break transaction into local transactions, each with a compensating action.
*   On failure: execute compensating transactions in reverse order.
*   No distributed locks needed; used in microservices.

---

## 5. Consensus: Raft Intuition

Raft guarantees that exactly one leader exists per term, preventing split-brain.

1.  **Leader election**: Candidate requests votes; majority wins.
2.  **Log replication**: Leader appends entries and sends to followers.
3.  **Commit**: Entry committed when acknowledged by majority.

Used in: etcd (Kubernetes backbone), CockroachDB, Consul.

---

## 6. Key Architectural Decisions

*   **Idempotency**: Design operations so retries are safe. Essential for distributed systems.
*   **Timeouts everywhere**: Never wait forever for a remote call.
*   **Bulkhead pattern**: Isolate failures so one subsystem doesn't bring down others.
*   **Backpressure**: Signal upstream to slow down when you're overwhelmed.
