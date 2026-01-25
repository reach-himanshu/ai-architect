# Design Patterns 07: Repository & Registry

These patterns manage how we access data and services properly.

## 1. Repository Pattern
Abstracts the Data Layer. The application core should not know if data comes from SQL, an API, or a text file.
*   **Benefit**: You can swap the database or use in-memory storage for unit tests.

## 2. Registry Pattern
A centralized place to store and retrieve global objects (services, configurations).
*   **Comparison**: Similar to a "Service Locator" or a simple Dependency Injection container.
*   **Usage**: `Registry.get('database')` typically returns the singleton database connection.
