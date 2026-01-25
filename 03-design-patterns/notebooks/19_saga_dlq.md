# Design Patterns: Saga Pattern & Dead Letter Queues

Patterns for handling distributed transactions and failure recovery in microservices.

## 1. The Problem
In a monolith, a single database transaction ensures atomicity. In microservices, each service has its own database. How do we ensure a multi-step process either fully completes or fully rolls back?

## 2. Saga Pattern
A Saga is a sequence of local transactions, where each step has a **compensating action** to undo it if a later step fails.

### Choreography vs Orchestration
- **Choreography**: Each service listens for events and triggers the next step. Decentralized.
- **Orchestration**: A central Saga Coordinator tells each service what to do. Centralized.

## 3. Dead Letter Queue (DLQ)
A DLQ is a holding place for messages that could not be processed successfully after multiple retries.

### Use Cases
- Malformed messages.
- Bugs in consumer logic.
- Downstream service permanently unavailable.

### What to do with DLQ messages?
- Analyze and fix the bug.
- Manually reprocess after fixing.
- Alert operations team.
