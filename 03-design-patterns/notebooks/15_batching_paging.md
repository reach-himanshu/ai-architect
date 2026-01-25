# Design Patterns 15: Batching & Paging

Techniques to reduce network overhead and handle large datasets.

## 1. Batching
Instead of sending 100 requests for 1 item each, send 1 request for 100 items. This reduces the HTTP overhead (handshake, headers) dramatically.

## 2. Pagination
How to fetch 1 million users?
1.  **Offset Pagination**: `LIMIT 10 OFFSET 1000`.
    *   **Problem**: DB must scan and skip 1000 rows. Slow for deep pages.
2.  **Cursor (Keyset) Pagination**: `WHERE id > 1000 LIMIT 10`.
    *   **Pros**: $O(1)$ efficient if index exists.
    *   **Cons**: Cannot jump to specific page "Page 500".
