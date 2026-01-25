# Design Patterns 01: SOLID Principles

SOLID is an acronym for five design principles intended to make software designs more understandable, flexible, and maintainable.

## 1. Single Responsibility Principle (SRP)
*   **Definition**: A class should have only one reason to change.
*   **Bad**: A `User` class that handles authentication, database saving, and email sending.
*   **Good**: Separate classes for `UserAuth`, `UserRepository`, and `EmailService`.

## 2. Open/Closed Principle (OCP)
*   **Definition**: Software entities should be open for extension, but closed for modification.
*   **Concept**: You should be able to add new functionality without changing existing code (often achieved via Inheritance or Interfaces).

## 3. Liskov Substitution Principle (LSP)
*   **Definition**: Objects of a superclass should be replaceable with objects of its subclasses without breaking the application.
*   **Key**: If it looks like a Duck and quacks like a Duck, but needs batteries... you probably have the wrong abstraction.

## 4. Interface Segregation Principle (ISP)
*   **Definition**: Clients should not be forced to depend upon interfaces that they do not use.
*   **Concept**: Many client-specific interfaces are better than one general-purpose interface.

## 5. Dependency Inversion Principle (DIP)
*   **Definition**: Depend upon abstractions, not concretions.
*   **Key**: High-level modules should not depend on low-level modules. Both should depend on abstractions (e.g., Abstract Classes or Protocols).
