# Design Patterns 05: Behavioral Patterns

Behavioral patterns are concerned with algorithms and the assignment of responsibilities between objects.

## 1. Observer
Defines a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.
*   **Use Case**: Event Listeners, UI Updates, Pub/Sub systems.

## 2. Strategy
Defines a family of algorithms, encapsulates each one, and makes them interchangeable. Strategy lets the algorithm vary independently from clients that use it.
*   **Use Case**: Payment methods (CreditCard, PayPal, Bitcoin) in a shopping cart. The cart just sees `pay()`.

## 3. Command
Encapsulates a request as an object, thereby letting you parameterize clients with different requests, queue or log requests, and support undoable operations.
*   **Use Case**: Text editor commands (Copy, Paste, Undo).
