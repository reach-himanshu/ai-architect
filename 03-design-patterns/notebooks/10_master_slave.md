# Design Patterns 10: Master-Slave (Leader-Follower)

This pattern is fundamental to Database Scalability.

## 1. The Concept
*   **Master (Leader)**: Handles all **Writes** (INSERT, UPDATE, DELETE). There is typically only one Master to ensure consistency.
*   **Slaves (Followers)**: Handle **Reads** (SELECT). There can be many slaves. They replicate data from the Master.

## 2. Benefits
*   **Read Scalability**: Most apps read way more than they write (e.g., Twitter: 1 tweet per user vs 1000 views). Adding Slaves scales reads linearly.
*   **Backup**: If Master dies, a Slave can be promoted to Master.

## 3. Trade-offs
*   **Replication Lag**: Data takes time to copy from Master to Slave. A user might write data and not see it immediately if they read from a slow Slave (Eventual Consistency).
