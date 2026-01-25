# Design Patterns 04: Structural Patterns

Structural patterns explain how to assemble objects and classes into larger structures while keeping these structures flexible and efficient.

## 1. Adapter
Allows objects with incompatible interfaces to work together. It wraps an existing class with a new interface.
*   **Analogy**: A travel power adapter. The wall socket is different, but the adapter makes your plug fit.

## 2. Decorator
Attaches additional responsibilities to an object dynamically.
*   **Python**: Python has built-in support via the `@decorator` syntax, which is an implementation of this pattern.

## 3. Facade
Provides a simplified interface to a library, a framework, or any other complex set of classes.
*   **Use Case**: Wrapping a complex 3rd party API (with 20 arguments) into a simple helper class with 2 arguments.
