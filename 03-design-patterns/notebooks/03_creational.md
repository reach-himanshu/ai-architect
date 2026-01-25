# Design Patterns 03: Creational Patterns

Creational patterns deal with object creation mechanisms, trying to create objects in a manner suitable to the situation.

## 1. Singleton
Ensures a class has only one instance and provides a global point of access to it.
*   **Use Case**: Database connections, Logging Config, Global App Configuration.
*   **Python Note**: Modules in Python are effectively singletons! Taking advantage of this is often clearer than forcing a Singleton class.

## 2. Factory Method
Defines an interface for creating an object, but lets subclasses decide which class to instantiate.
*   **Use Case**: A unified `ButtonFactory` that returns `WindowsButton` or `MacButton` depending on the OS.

## 3. Builder
Separates the construction of a complex object from its representation.
*   **Use Case**: Building complex SQL queries (`query.select().from().where()`) or configuring a complex `User` object with many optional parameters.
