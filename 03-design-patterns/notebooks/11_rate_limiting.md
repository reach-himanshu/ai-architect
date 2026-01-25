# Design Patterns 11: Rate Limiting (Throttling)

Rate limiting controls the rate of traffic sent or received by a network interface controller. It prevents DoS attacks and resource exhaustion.

## 1. Algorithms
1.  **Token Bucket**: Tokens are added to a bucket at a fixed rate. Each request consumes a token. If bucket is empty, request is denied. Allows "bursts" up to bucket capacity.
2.  **Leaky Bucket**: Requests enter a queue and are processed at a constant rate. Smooths out traffic. No bursts.
3.  **Fixed Window**: "100 requests per minute". Reset counter at the top of the minute. Can have boundary issues (e.g. 100 requests at 10:00:59 and 100 at 10:01:01).

## 2. Implementation
We often use Redis or similar in-memory stores to keep global counters in distributed systems.
