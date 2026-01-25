# Design Patterns: State Machine

A State Machine models an entity that transitions between a finite set of states based on events/inputs.

## 1. Components
- **States**: The possible conditions (e.g., `IDLE`, `RUNNING`, `PAUSED`).
- **Events**: Triggers that cause transitions (e.g., `start`, `pause`, `stop`).
- **Transitions**: Rules for moving between states.

## 2. Use Cases
- **Order Processing**: `PENDING` → `PAID` → `SHIPPED` → `DELIVERED`.
- **Game Characters**: `IDLE` → `WALKING` → `ATTACKING`.
- **UI Components**: `DISABLED` → `ENABLED` → `LOADING`.

## 3. Implementation Approaches
- **Enum + Dictionary**: Simple, Python-native.
- **State Pattern (OOP)**: Each state is a class.
- **Libraries**: `transitions`, `statemachine`.
