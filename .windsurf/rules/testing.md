---
trigger: always_on
description: "Testing requirements and patterns for TDD, widget testing, and controller testing"
---

#### Testing Strategy

##### Test-Driven Development (TDD)

- **ALWAYS write tests first** for any new functionality - no exceptions
- **ALWAYS follow the Red-Green-Refactor (TDD) cycle** strictly when writing code:
  1. **RED**: Write a failing test that describes the desired behavior
  2. **GREEN**: Write the minimal code to make the test pass
  3. **REFACTOR**: Improve the code while keeping tests green
- **NEVER write production code without a failing test first**
- **NEVER skip the refactor step** - always clean up code after making tests pass
- **ALWAYS run tests after each step** to ensure they pass/fail as expected

##### Testing Trophy

Follow the Testing Trophy model for effective test coverage:

```
       ___
      /   \
     /  E2E \        30% - End-to-End Tests
    /_______\
   /         \
  / Integration \    50% - Integration Tests (PRIMARY FOCUS)
 /_______________\
/                 \
|   Unit Tests    |  20% - Unit Tests
|_________________|
|_Static_Analysis_| Base - Linting, Type Checking, Compiler
```

**Static Analysis (Foundation):**
- Linting with `flutter analyze`
- Type checking with strict mode
- Compiler errors caught before runtime
- Custom lint rules (riverpod_lint, custom_lint)
- Zero cost - catches issues immediately

**Unit Tests (~20%):**
- Test pure functions, utilities, and models
- Test complex business logic in isolation
- Test data transformations and calculations
- Fast, isolated, no external dependencies
- Focus on edge cases and corner cases

**Integration Tests (~50% - PRIMARY FOCUS):**
- Test feature flows with stores and real dependencies
- Test user interactions and state changes together
- Test stores with real repositories (mock only external APIs)
- Test widget + store + service integration
- Verify data flows through the app correctly
- Balance between speed and confidence

**E2E Tests (~30%):**
- Test critical user journeys end-to-end
- Test complete flows: login → dashboard → action → logout
- Test with real backend (or realistic mocks)
- Verify app works as users experience it
- Focus on happy paths and critical business flows

##### What to Test

**Primary Focus (Integration Tests):**
- ✅ **User behavior and feature flows** - Test complete user interactions
- ✅ **Store + Repository + Service integration** - Test data flows through layers
- ✅ **State management with real dependencies** - Mock only external APIs
- ✅ **Navigation flows** - Test routing and screen transitions
- ✅ **Form validation with submission** - Test entire form flow
- ✅ **Error handling across layers** - Test how errors propagate and display
- ✅ **Widget + Store integration** - Test UI updates when state changes

**Secondary Focus (Unit Tests):**
- ✅ **Complex business logic** - Calculations, algorithms, transformations
- ✅ **Pure functions** - Utilities, validators, formatters
- ✅ **Data models** - Serialization, equality, copyWith
- ✅ **Edge cases** - Boundary conditions, null handling, error cases

**Tertiary Focus (E2E Tests):**
- ✅ **Critical user journeys** - Login, checkout, payment, signup
- ✅ **Happy paths** - Most common user flows
- ✅ **Cross-feature flows** - Features working together
- ✅ **Regression prevention** - Ensure critical paths don't break

**Don't test:**
- ❌ Third-party packages (trust they're tested)
- ❌ Flutter framework widgets
- ❌ Generated code (*.g.dart, *.freezed.dart)
- ❌ Simple getters/setters without logic
- ❌ UI styling (colors, fonts) - use visual regression testing instead
- ❌ Implementation details - Test behavior, not internals