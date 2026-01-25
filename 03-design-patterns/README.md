# Software Design & Architecture Patterns

This module covers essential design patterns ranging from low-level Object-Oriented patterns to high-level System Architecture and Cloud resilience patterns.

## 🚀 Roadmap & Sequence

### 1. Fundamental Principles (The "Why")
*   **01. SOLID Principles:** The 5 commandments of clean OOD.
*   **02. IoC & Dependency Injection:** Inversion of Control using construction injection (Decoupling logic from dependencies).

### 2. Classic GoF Patterns (The "How" of Objects)
*   **03. Creational:** Singleton, Factory Method, Builder.
*   **04. Structural:** Adapter, Decorator, Facade, Proxy.
*   **05. Behavioral:** Observer, Strategy, Command, Iterator.

### 3. Enterprise Application Patterns (Ordering Code)
*   **06. MVC & MVT:** Model-View-Controller (and Template). Separating concerns.
*   **07. Repository & Registry:** 
    *   *Repository*: Abstractions for data access (e.g., swapping SQL for NoSQL).
    *   *Registry*: Global access point for services/configurations.

### 4. Distributed & Cloud Patterns (Resiliency & Scale)
*   **08. Circuit Breaker:** Preventing cascading failures in microservices.
*   **09. Retry (Repeater) & Backoff:** Handling transient failures gracefully.
*   **10. Master-Slave (Leader-Follower):** Database replication and read/write splitting strategies.
*   **11. Rate Limiting (Function Clipping):** Throttling requests to protect resources (Token Bucket, Leaky Bucket).

### 5. Data & Integration Patterns
*   **12. ETL & Pipelines:** Extract-Transform-Load fundamentals for data engineering.
*   **13. SOA & Microservices:** Service discovery, API Gateway, and Event-Driven concepts.

### 6. System Design & Optimization Patterns (New)
*   **14. Caching Strategies:** Cache-Aside, Write-Through, and Write-Back policies.
*   **15. Data Fetching:** Batching requests and Pagination strategies (Offset vs. Cursor).
*   **16. Data Processing:** Streaming (Generators/Buffers) vs. Batch Processing.
*   **17. Data Optimization:** Encoding (JSON vs Protobuf), Compression, and Compaction techniques.

---

## 📁 Format
Each topic will include:
1.  **Real-World Scenario:** "Why do we need this?" (e.g., "Netflix needs to survive a database failure").
2.  **Implementation:** Python code showing the pattern in action.
3.  **Diagram/Flow:** Visual explanation (via text/mermaid).
