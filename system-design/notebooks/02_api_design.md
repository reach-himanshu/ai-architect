# System Design 02: API Design

## 1. REST Principles

REST (Representational State Transfer) constraints:
*   **Stateless**: Each request contains all necessary information — no server-side session.
*   **Uniform Interface**: Standard HTTP verbs identify operations, not URLs.
*   **Resource-based**: URLs name resources (`/users/123`), not actions (`/getUser`).
*   **Cacheable**: Responses declare cacheability with HTTP headers.

| Method | Purpose | Idempotent | Safe |
| :--- | :--- | :--- | :--- |
| GET | Read | Yes | Yes |
| POST | Create | No | No |
| PUT | Replace | Yes | No |
| PATCH | Partial update | No | No |
| DELETE | Remove | Yes | No |

---

## 2. API Versioning Strategies

| Strategy | Example | Recommendation |
| :--- | :--- | :--- |
| URL path | `/api/v1/users` | Best for public APIs — explicit and cacheable |
| Request header | `X-API-Version: 2` | Clean URLs, less visible |
| Query param | `/users?version=2` | Easy to test, easy to forget |
| Accept header | `Accept: application/vnd.api+json;v=2` | Purist REST, complex |

**Versioning rule**: Never break a published contract. Additive changes (new fields) are safe; removing fields is a breaking change.

---

## 3. Pagination Strategies

**Offset pagination** — simple but problematic at scale:
```
GET /posts?offset=100&limit=20
Problem: Deletes/inserts between pages cause skipped or duplicate items.
```

**Cursor pagination** — stable for large or live datasets:
```
GET /posts?after=eyJpZCI6MTAwfQ&limit=20
Response: {"data": [...], "next_cursor": "eyJpZCI6MTIwfQ", "has_more": true}
```

Use cursor for: feeds, infinite scroll, large datasets, real-time data.

---

## 4. Rate Limiting

**Token Bucket** (allows bursts):
*   Refill bucket at a fixed rate (e.g., 100 tokens/min).
*   Each request consumes tokens; reject if bucket empty.
*   Allows short bursts up to bucket capacity.

**Sliding Window** (smooth limiting):
*   Track timestamps of requests in last N seconds.
*   Reject if count exceeds limit.
*   More accurate but more memory per user.

Rate limit headers to return:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1700000060
Retry-After: 30
```

---

## 5. REST vs gRPC vs GraphQL

| | REST | gRPC | GraphQL |
| :--- | :--- | :--- | :--- |
| **Protocol** | HTTP/1.1 | HTTP/2 | HTTP |
| **Payload** | JSON | Protobuf (binary) | JSON |
| **Schema** | OpenAPI (optional) | `.proto` files (required) | Schema required |
| **Streaming** | SSE / WebSocket | Native bi-directional | Subscriptions |
| **Best for** | Public APIs, browsers | Internal microservices | Flexible client queries |
| **Tooling** | Excellent | Good | Good |
