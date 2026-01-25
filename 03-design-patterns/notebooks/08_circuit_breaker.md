# Design Patterns 08: Circuit Breaker

In distributed systems, if a service fails (e.g., a Database or API), making more requests to it will only start a "cascade of failure". The Circuit Breaker pattern prevents this.

## 1. States
1.  **Closed**: Everything is normal. Requests go through.
2.  **Open**: The service failed too many times. Requests are blocked immediately (fail fast) to give the system time to recover.
3.  **Half-Open**: After a timeout, we let *one* request through to test if the service is back online.

## 2. Analogy
Just like an electrical circuit breaker in your house protects functionality from a power surge.
