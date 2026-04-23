# System Design 06: Scalability Patterns

## 1. Horizontal vs. Vertical Scaling

| | Vertical (Scale Up) | Horizontal (Scale Out) |
| :--- | :--- | :--- |
| **How** | Bigger machine | More machines |
| **Limit** | Hardware ceiling | Theoretically unlimited |
| **Cost** | Exponential at high end | Linear |
| **Complexity** | Simple | Requires distribution logic |
| **SPOF** | Yes | No (with load balancer) |
| **Best for** | Databases (start here) | Stateless services |

**Strategy**: Vertical first (easier), horizontal when you hit limits.

---

## 2. Load Balancing Algorithms

| Algorithm | Description | Best For |
| :--- | :--- | :--- |
| **Round-Robin** | Rotate sequentially | Uniform request cost |
| **Least Connections** | Route to server with fewest active conns | Variable request cost |
| **Weighted Round-Robin** | Traffic proportional to server capacity | Heterogeneous hardware |
| **IP Hash** | Same client → same server | Session affinity |
| **Consistent Hash** | Key → server, minimal remapping | Caches, sharded stores |

---

## 3. Consistent Hashing

Problem with `hash(key) % N`: adding one server remaps ≈ 50% of all keys.

Consistent hashing places servers on a ring. Adding/removing a server only remaps ~1/N keys.

```
Ring:  0 ────── server-A ────── server-B ────── server-C ─── 2³²
Key "user:123" → hashes to position → walks clockwise → hits server-B
```

Virtual nodes (vnodes): Each physical server gets 100–150 virtual positions on the ring, improving balance.

Used by: Cassandra, DynamoDB, Memcached, many CDNs.

---

## 4. Caching Hierarchy

```
L1: In-process cache (Python dict) — μs, per-instance, lost on restart
L2: Shared distributed cache (Redis) — ms, all instances share
L3: CDN edge cache (CloudFront, Fastly) — ms, geographically distributed
L4: Database — 10–100ms
L5: Object storage (S3) — 100ms–1s
```

Cache eviction policies:
*   **LRU** (Least Recently Used): Good general default.
*   **LFU** (Least Frequently Used): Good for stable popular items.
*   **TTL**: Time-based expiry for time-sensitive data.

---

## 5. AI-Specific Caching Strategies

| Pattern | What to Cache | TTL | Savings |
| :--- | :--- | :--- | :--- |
| **Semantic cache** | LLM responses for similar queries | 1 hr | High |
| **Embedding cache** | Vectors for seen documents | Indefinite | High |
| **Prompt prefix cache** | Anthropic/OpenAI shared prefix | Session | 50–90% |
| **Feature cache** | Computed ML features | 5 min | Medium |
| **API response cache** | Identical API responses | 1 min | Medium |

---

## 6. Read/Write Separation

Route heavy read traffic to read replicas, writes to primary:

```
Writes → Primary DB
Reads  → Read Replica 1
         Read Replica 2
         Read Replica 3
```

For ML systems: write predictions to primary, read feature lookups from replicas.
