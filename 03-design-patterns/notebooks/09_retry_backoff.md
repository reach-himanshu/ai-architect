# Design Patterns 09: Retry & Backoff

Network failures are often "transient"—they last for a few milliseconds (blips). Just blindly failing is bad user experience. We should retry.

## 1. Retry Strategy
Trying the request again. Usually limited to `N` attempts.

## 2. Exponential Backoff
Instead of retrying immediately (which might hammer a struggling server), wait longer between each retry.
*   Attempt 1: Wait 1s
*   Attempt 2: Wait 2s
*   Attempt 3: Wait 4s
*   Attempt 4: Wait 8s

## 3. Jitter
Adding randomness to the wait time to prevent "Thundering Herd" problems where all clients retry at the exact same millisecond.
