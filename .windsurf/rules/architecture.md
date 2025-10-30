---
trigger: model_decision
description: "Code organization"
---

#### Code Organization

##### Choosing the Right Structure

**Small** (< 10 screens): Flat structure - models/, services/, screens/, widgets/
**Medium** (10-30 screens): Feature-based - ui/[feature]/ with stores
**Large** (30+ screens): MVU pattern - app/, data/, domain/, ui/ with state/

##### Small Project Structure

```
lib/
├── main.dart
├── models/              # Data models
├── services/            # Business logic & API calls
├── screens/             # All screens
└── widgets/             # Shared widgets
```

**When to use:** < 20 files, simple data flow, prototypes

##### Medium Project Structure

```
lib/
├── main.dart
├── models/              # Shared data models
├── services/            # Shared services
├── ui/                  # UI layer with features
│   ├── auth/           # Feature: Authentication
│   ├── home/           # Feature: Home
│   └── widgets/        # Shared UI components
└── utils/               # Shared utilities
```

**When to use:** 10-30 screens, multiple features, growing team

##### MVU (Model-View-Update) Pattern

**Model**: Immutable state (Equatable) - pure data, no logic
**View**: Pure UI (screen widgets) - renders based on state
**Update**: Store (Notifier) - manages state transitions and side effects

**Benefits:** Clear separation, predictable state, easy testing, type-safe, unidirectional flow

**When to use:** Large projects, complex state, multiple state variations, testing priority

##### Large Project Structure (MVU Pattern)

```
lib/
├── main.dart
├── app/                    # App-level configuration
├── core/                   # Core utilities & constants
├── data/                   # Data layer
├── domain/                 # Business logic & entities
└── ui/                     # Presentation layer
    ├── auth/              # Feature: Authentication
    ├── home/              # Feature: Home
    └── widgets/           # Reusable non-feature components
```

**When to use:** 30+ screens, complex business logic, large team, clean architecture

#### Testing Considerations

**Small**: Test screens/services directly, minimal mocking, integration tests
**Medium**: Test stores with mocked services, widget tests with mocked providers
**Large (MVU)**: Unit test state (copyWith, equality), unit test stores (mock repos), widget test screens (mock stores), integration test flows

#### Migration Paths

**Small → Medium:**

1. Create `ui/` directory for all UI components
2. Group related screens into feature folders under `ui/`
3. Move screen-specific widgets into feature `widgets/` folders
4. Add stores for business logic
5. Keep shared models and services at root level
6. Move shared widgets to `ui/widgets/`

**Medium → Large:**

1. Create `data/` layer with repositories and models
2. Create `domain/` layer with entities and use cases
3. Reorganize `ui/` with feature-based structure
4. Move business logic from stores to domain layer
5. Update stores to use repositories from data layer
6. Create `core/` for shared utilities and constants

#### General Principles

- Follow existing project structure - don't mix structures
- Group related files together
- Consistent naming: `*_screen.dart`, `*_store.dart`, `*_widget.dart`
- One widget per file (except small private helpers)
- Use `index.dart` for barrel exports
- Features should not import from other features