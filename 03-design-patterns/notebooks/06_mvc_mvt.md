# Design Patterns 06: MVC & MVT

These architectural patterns separate concerns in an application: Data (Model), Logic (Controller/View), and Interface (View/Template).

## 1. MVC (Model-View-Controller)
*   **Model**: The data structure (e.g., User, Product). Database logic often lives here.
*   **View**: The display (HTML, UI). It shows data to the user.
*   **Controller**: The brain. Accepts input, talks to the Model, and selects the View.

## 2. MVT (Model-View-Template)
Used by frameworks like **Django**.
*   **Model**: Same as MVC.
*   **View**: Handles the logic (like the Controller in MVC).
*   **Template**: Handles the display (like the View in MVC).

## 3. Why Separation Matters?
*   **Parallel Development**: Frontend devs work on Views/Templates, Backend devs work on Models/Controllers.
*   **Maintainability**: Changing the UI doesn't break the database logic.
