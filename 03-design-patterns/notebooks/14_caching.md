# Design Patterns 14: Caching Strategies

Caching reduces latency and database load by storing frequently accessed data in faster storage (RAM).

## 1. Cache-Aside (Lazy Loading)
The Application talks to both Cache and DB.
1.  App: "Cache, do you have user 1?"
2.  Cache: "No (Miss)."
3.  App: "DB, give me user 1."
4.  DB: returns User 1.
5.  App: "Cache, store User 1."
6.  App: Returns data to client.

## 2. Write-Through
The Application treats Cache as the main store.
1.  App writes to Cache.
2.  Cache synchronously writes to DB.
*   **Pros**: Consistency. Cache always matches DB.
*   **Cons**: Higher write latency.
