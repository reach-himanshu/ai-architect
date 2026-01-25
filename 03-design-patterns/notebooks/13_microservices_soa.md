# Design Patterns 13: Microservices & SOA

Service-Oriented Architecture (SOA) and Microservices decompose a large application into smaller, independent services.

## 1. Core Components
*   **Service**: A standalone unit of functionality (e.g., "Payment Service", "User Service").
*   **API Gateway**: The single entry point for clients. It routes requests to the appropriate service. It also handles Authentication, Rate Limiting, and SSL Termination.

## 2. Benefits
*   **Independent Deployment**: You can update the User Service without redeploying the Payment Service.
*   **Scalability**: You can run 5 instances of the Payment Service and only 1 User Service if payments are heavy.

## 3. Communication
*   **Synchronous**: HTTP/REST or gRPC.
*   **Asynchronous**: Message Queues (Kafka, RabbitMQ) for event-driven flows.
