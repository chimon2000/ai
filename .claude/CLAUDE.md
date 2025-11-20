# Claude Desktop Agent Rules

This file consolidates all development rules for Claude Desktop integration. Rules are organized by type and domain.

## Table of Contents

- [Always Apply Rules](#always-apply-rules)
  - [General Principles](#general-principles)
  - [Testing Strategy](#testing-strategy)
  - [Tidy First](#tidy-first)
  - [Refactoring Guidelines](#refactoring-guidelines)
  - [Documentation Standards](#documentation-standards)
- [Agent Requested Rules](#agent-requested-rules)
  - [Code Organization](#code-organization)
  - [Flutter/Dart Specific](#flutterdart-specific)
  - [Riverpod State Management](#riverpod-state-management)
  - [Exhaustive Type Checking](#exhaustive-type-checking)
  - [Rule Writing](#rule-writing)

---

## Always Apply Rules

### General Principles

**Core Principles:**
- **Eliminates duplication ruthlessly** - Remove repeated code, logic, and patterns
- **Expresses intent clearly** - Use descriptive names for classes, methods, variables
- **Makes dependencies explicit** - Declare all dependencies clearly
- **Keeps methods small and focused** - Single responsibility principle, aim for < 20 lines
- **Minimizes state and side effects** - Prefer immutability and pure functions
- **Uses the simplest solution** - YAGNI principle, avoid premature optimization

**Context-Aware Rules:**
- **ALWAYS** examine existing codebase patterns before implementing new features
- If project uses different patterns than specified, follow existing patterns for consistency
- Prefer project consistency over rigid rule adherence when patterns conflict

**Rule Priority System:**
1. **Critical Priority**: Pre-task analysis, context awareness
2. **High Priority**: Error handling, testing, navigation
3. **Medium Priority**: Code organization, dependency management
4. **Low Priority**: Formatting, naming conventions

**Conflict Resolution:**
1. **Project Consistency** > **Rule Adherence** (when patterns conflict)
2. **Existing Patterns** > **New Patterns** (unless refactoring entire codebase)
3. **User Requirements** > **Technical Preferences**

### Testing Strategy

**Test-Driven Development (TDD):**
- **Write tests first** for any new functionality
- **Follow the Red-Green-Refactor (TDD) cycle** when writing code
- **Run tests after each step** to ensure they pass/fail as expected
- **Exception**: Spike/exploration work may skip TDD temporarily, but must be refactored with tests before merging

**Testing Trophy Model:**
- **Static Analysis (Foundation)**: Linting, type checking, compiler errors
- **Unit Tests (~20%)**: Pure functions, utilities, models, complex business logic
- **Integration Tests (~50% - PRIMARY FOCUS)**: Feature flows, store + repository integration, widget + store integration
- **E2E Tests (~30%)**: Critical user journeys, complete flows, happy paths

**Why Integration Tests are Primary:**
- ✅ Test how components work together (closer to real usage)
- ✅ Catch more bugs than isolated unit tests
- ✅ Resilient to refactoring (test behavior, not implementation)
- ✅ Balance between speed and confidence

**What to Test:**
- ✅ User behavior and feature flows
- ✅ Store + Repository + Service integration
- ✅ State management with real dependencies (mock only external APIs)
- ✅ Navigation flows and screen transitions
- ✅ Form validation with submission
- ✅ Error handling across layers
- ✅ Widget + Store integration
- ❌ Third-party packages
- ❌ Flutter framework widgets
- ❌ Generated code (*.g.dart, *.freezed.dart)
- ❌ Simple getters/setters without logic
- ❌ UI styling (colors, fonts)
- ❌ Implementation details

**When to Apply Testing Trophy:**
- **Use Testing Trophy (50% integration focus) for**: Feature-complete screens with stores and services, complex user workflows, production apps, teams prioritizing confidence
- **Focus on Unit Tests (80%+ unit focus) for**: Utility libraries, complex algorithms, shared components, performance-critical code
- **Use E2E Tests (30%+ E2E focus) for**: Critical user journeys (authentication, payments), multi-screen workflows, regression prevention

### Tidy First

**Principles:**
- **Always make structural changes first** before behavioral changes
- **Never mix structural and behavioral changes** - commit them separately
- See `general.md` for core principles

**Workflow:**
1. Identify needed structural changes
2. Make structural changes first - Refactor, rename, reorganize
3. Run tests - Ensure structural changes don't break anything
4. Commit structural changes - Separate commit
5. Implement behavioral changes - Add new feature with TDD
6. Commit behavioral changes - Separate commit

**Structural Changes:**
- Renaming variables, methods, classes
- Extracting methods or classes
- Moving code to different files
- Reorganizing imports
- Removing dead code
- Simplifying complex conditionals
- Breaking up large methods

**Behavioral Changes:**
- Adding new features
- Fixing bugs
- Changing business logic
- Modifying API responses
- Updating UI behavior

### Refactoring Guidelines

**Guidelines:**
- **Red-Green-Refactor cycle** - Refactor only when tests are passing
- **One refactoring at a time** - Make a single, focused change per commit
- **Run tests after each step** - Verify tests still pass after each refactoring
- **Use IDE refactoring tools** - Leverage automated refactoring when available
- **Leave code better than you found it** - Boy Scout Rule

**Common Patterns:**
- **Extract Method** - Remove duplication by extracting repeated code
- **Rename for Clarity** - Use clear, descriptive names that reveal intent
- **Simplify Conditionals** - Replace complex conditionals with guard clauses

**When NOT to Refactor:**
- Don't refactor code you don't understand - Add tests first
- Don't refactor without tests - Tests verify refactoring didn't break functionality
- Don't refactor during feature development - Separate structural and behavioral changes
- Don't refactor code that works and isn't causing problems - YAGNI principle

### Documentation Standards

**README File Length Limit:**
- README.md files should **never exceed 400 lines** (not characters)
- Keep content concise, scannable, and focused on essential information
- Remove verbose explanations, redundant examples, and excessive prose

**When Additional Context is Needed:**
1. **Split into separate markdown files** - Create focused docs (DEPLOYMENT.md, DEVELOPMENT.md, etc.)
2. **Use visual diagrams** - Replace lengthy text with Mermaid diagrams
3. **Link to detailed docs** - Keep README as high-level overview

**README Best Practices:**
- **Quick-start first** - Get users running in < 5 minutes
- **Table of contents** - For READMEs > 100 lines
- **Sections in priority order**: Description, Quick start, Use cases, Configuration, Links to docs, Troubleshooting
- ✅ Prioritize quick-start information
- ✅ Use tables, bullet points, inline code for scannability
- ✅ Include only essential examples (one per concept)
- ✅ Link to external docs for deep dives
- ✅ Use diagrams to replace multi-paragraph explanations
- ❌ Avoid verbose prose
- ❌ Avoid redundant examples
- ❌ Avoid copy-pasting

---

## Agent Requested Rules

### Code Organization

**Choosing the Right Structure:**
- **Small** (< 10 screens): Flat structure - models/, services/, screens/, widgets/
- **Medium** (10-30 screens): Feature-based - ui/[feature]/ with stores
- **Large** (30+ screens): MVU pattern - app/, data/, domain/, ui/ with state/

**Decision Criteria for Migration:**
- **Small → Medium**: 10+ screens, 3+ features, multiple screens share logic, team growing, state complexity increasing
- **Medium → Large**: 30+ screens, 5+ features, complex business logic, team 3+, reusable domain logic, complex API layer
- **Signs ready to migrate**: Structure feels disorganized, hard to find files, duplicate code, difficult to test, new members struggle
- **Signs not ready**: < 10 screens, simple data flow, single developer, prototype/MVP, frequent structural changes

**Small Project Structure:**
- Use flat structure with models/, services/, screens/, widgets/
- When to use: < 20 files, simple data flow, prototypes

**Medium Project Structure:**
- Use feature-based structure with ui/[feature]/ directories
- When to use: 10-30 screens, multiple features, growing team

**Large Project Structure (MVU Pattern):**
- **Model**: Immutable state (Equatable) - pure data, no logic
- **View**: Pure UI (screen widgets) - renders based on state
- **Update**: Store (Notifier) - manages state transitions and side effects
- When to use: 30+ screens, complex business logic, large team, clean architecture
- **Medium vs Large**: Medium has store at feature root; Large has store in state/ subdirectory with separate state file

**General Principles:**
- Follow existing project structure - don't mix structures
- Group related files together
- Consistent naming: `*_screen.dart`, `*_store.dart`, `*_widget.dart`
- One widget per file (except small private helpers)
- Use `index.dart` for barrel exports
- Features should not import from other features

### Flutter/Dart Specific

**Widget Organization:**
- Always organize constructors before properties
- Always implement `const` constructors for widgets when possible
- Always add `super.key` to widget constructors
- **NEVER use widget helper methods** - extract widgets into separate classes
- **NEVER create private `_build*` methods** - use composition with separate widget classes
- Always extract complex widget trees into separate widget classes
- Use widget composition over widget helper methods for better testability

**Widget State Management:**
- Use `StatelessWidget` for UI components without internal state
- Use `StatefulWidget` only for components that need to maintain UI-related state
- **NEVER store business logic or application state in StatefulWidget**
- Store only ephemeral UI state in StatefulWidget
- Valid StatefulWidget state: animation controllers, text field focus, local form validation, scroll positions
- Invalid StatefulWidget state: user data, API responses, authentication state, app-wide settings
- For any state that needs to be shared across widgets or persisted, use Riverpod providers

**Error Handling Patterns:**
- Always use `result_dart` package for operations that can fail
- Use `Result<T>` for success type T with Exception as failure type
- Use `ResultDart<T, E>` for custom failure types
- Use `Result<Unit>`/`ResultDart<Unit, E>` for operations that don't return data but can fail
- Always handle both `Success` and `Failure` cases using `.isSuccess()` and `.isError()`
- Use `result.exceptionOrNull()` to access error details
- Wrap exceptions in proper Exception types, not raw strings
- Use `NetworkException.handleException()` for API error handling when available
- Implement proper error boundaries with custom error widgets
- Always handle `AsyncValue` states: `data`, `loading`, `error`

**User-Facing Error Handling:**
- Always provide user feedback for async operations (loading states, error messages)
- Use `ScaffoldMessenger.of(context).showSnackBar()` for temporary error messages
- Use proper error boundaries for critical failures
- Handle `AsyncValue` states consistently: loading, data, error
- Provide meaningful error messages, not technical exception details
- Always check `mounted` before showing UI feedback in async operations

**Project-Specific Patterns:**
- **Small Projects**: Use flat structure with models/, services/, screens/, widgets/
- **Medium Projects**: Use feature-based structure with controllers/ and providers/
- **Large Projects**: Use the full MVU structure
- Always follow existing project structure patterns
- Group related files together (controller + screen + widgets)
- Use consistent naming conventions throughout the project

**Navigation:**
- **Primary**: Use `AutoRoute` for all navigation when available
- **Fallback**: If AutoRoute is not configured, use `Navigator.of(context).push()`
- Always check existing navigation patterns in the codebase before implementing
- Use `@RoutePage()` annotation for all screen widgets when using AutoRoute
- Implement proper route guards and observers for protected routes
- Always use `appRouter.push()` instead of `Navigator` when AutoRoute is available

**Dependency Management:**
- Always use package managers (flutter pub add) instead of manually editing pubspec.yaml
- Check existing dependencies before adding new ones to avoid conflicts
- Verify package compatibility with existing Flutter/Dart versions
- Use specific version constraints for critical dependencies
- Document why specific packages were chosen in comments or README
- Always run `flutter pub get` after dependency changes
- Use `flutter pub deps` to check for dependency conflicts

**API Integration:**
- Always use Chopper service classes for API calls
- Use `ApiClient` from `get_it` for dependency injection
- Handle responses with `.when(success: ..., failure: ...)` pattern
- Always return fallback data in failure cases for CMS providers

**Testing Patterns:**
- Use `mocktail` for mocking, not `mockito`
- Always create `createTestWidget()` helper with `ProviderScope`
- Use `ScreenUtilInit` wrapper in test widgets
- Mock stores by extending `AutoDisposeNotifier` with `Mock`
- Always use `overrides` parameter for provider testing

**Code Generation:**
- Run `dart run build_runner build -d` after model changes
- Use `@freezed` for immutable data classes
- Use `@JsonSerializable` for API models
- Always use `part` files for generated code

### Riverpod State Management

**State Management with Riverpod:**

**Synchronous (Notifier):**
- Use `Notifier` for synchronous state management
- Use `AutoDisposeNotifierProvider` to prevent memory leaks
- Keep stores focused on a single feature or domain
- Use `state = state.copyWith()` for immutable state updates
- Never mutate state directly - always create new instances

**Asynchronous (AsyncNotifier):**
- Use `AsyncNotifier` for async operations
- Use `FutureProvider.autoDispose` for one-time async operations
- Use `ref.cacheFor()` for expensive operations that should be cached
- Use `ref.refreshIn()` for data that should auto-refresh

**Store Pattern:**
- Separate business logic into store classes
- Use `AutoDisposeAsyncNotifier` for async stores
- Stores handle business logic, navigation, analytics
- Widgets handle UI rendering and user interactions only
- Use `Result<T>` for store methods (Exception as failure type)
- Use `ResultDart<T, E>` for custom failure types
- Use `Result<Unit>` for operations that don't return data

**Provider Selection Decision Tree:**
- Is the data async (Future/Stream)? → Use `AsyncNotifier` or `FutureProvider`
- Does the data need to be cached? → Use `FutureProvider` with `cacheFor()`
- Is this a one-time operation? → Use `FutureProvider.autoDispose`
- Does this need to be watched by widgets? → Use `Notifier` or `AsyncNotifier`

**Common Patterns:**
- **Combining Multiple Providers** - Use `.when()` to combine multiple providers
- **Invalidating Providers** - Use `ref.refresh()` or `ref.invalidate()`
- **Family Providers** - Use `FutureProvider.family` for parameterized providers

### Exhaustive Type Checking

**Principle:**
When you have a known, finite set of subtypes or variants, always use patterns that enable compile-time exhaustive checking rather than runtime type checks or open inheritance hierarchies.

**Core Pattern: Use Sealed Classes for Type Hierarchies:**
- ✅ **GOOD**: Sealed class with exhaustive switch - Compiler enforces all cases are handled
- ❌ **BAD**: Abstract class with runtime checks - Silent bug if new type added

**Pattern 1: Use Enums for Simple Value Sets:**
- Use enums for simple value sets (Status, Priority, etc.)
- Use exhaustive switch expressions for pattern matching

**Pattern 2: Use Sealed Classes for Complex Type Hierarchies:**
- Use sealed classes for type hierarchies with known, finite subtypes
- Use pattern matching with switch expressions

**Pattern 3: Use Switch Expressions for Pattern Matching:**
- Use switch expressions instead of if-else chains
- More concise and readable than traditional switch statements

**When to Apply This Rule:**
- **Always use sealed classes** for type hierarchies with known, finite subtypes
- **Always use enums** for simple value sets
- **Always use switch expressions** for pattern matching on sealed types
- **Never use abstract classes** when you want exhaustive checking
- **Never use runtime type checks** (is/as) when sealed classes are available
- **Never use if-else chains** for type checking when switch is available

**Benefits:**
- ✅ Compile-time safety - Compiler catches missing cases
- ✅ Refactoring safety - Adding new types forces updates everywhere
- ✅ Self-documenting - Code clearly shows all possible cases
- ✅ No silent bugs - Can't accidentally miss a case
- ✅ Better performance - No runtime type checks needed
- ✅ Cleaner code - Pattern matching is more readable

### Rule Writing

**Rule Quality Standards:**
- **Focused and Actionable** - Each rule addresses a single domain or concern
- **Scoped Appropriately** - Keep rules under 650 lines maximum
- **Concrete and Specific** - Always include code examples showing ❌ BAD and ✅ GOOD patterns
- **Clear and Scannable** - Use bullet points, tables, and code blocks for scannability
- **Composability** - Rules should be composable and reference each other when needed

**When to Create New Rules:**
- You find yourself repeating the same guidance across multiple conversations
- A domain has grown complex enough to warrant dedicated documentation (>100 lines)
- You need to establish patterns for a new technology or framework
- Existing rules exceed 650 lines and can be logically split

**Rule Maintenance:**
- When simplifying rules, aim for 30-50% reduction while preserving all unique patterns
- Remove verbose explanations and redundant examples
- Keep all decision trees, anti-pattern examples (❌/✅), and migration guides
- Consolidate duplicate content across sections
- Verify all code examples still compile and follow current patterns

**Quality Checklist:**
- [ ] Under 650 lines
- [ ] Includes concrete code examples
- [ ] Uses ❌ BAD / ✅ GOOD comparisons for anti-patterns
- [ ] Scannable with clear headings and bullet points
- [ ] No vague language ("consider", "might", "should probably")
- [ ] Frontmatter metadata present
- [ ] References specific files/patterns when applicable
- [ ] All code examples are concise and focused
- [ ] No redundant or duplicate content

---

## Cross-References

- **General Principles** → Referenced by all other rules
- **Testing Strategy** → Referenced by Tidy First, Refactoring
- **Tidy First** → Works with Testing Strategy and Refactoring
- **Refactoring Guidelines** → Works with Tidy First and Testing
- **Code Organization** → Referenced by Flutter/Dart and Riverpod
- **Flutter/Dart Specific** → Works with Code Organization and Riverpod
- **Riverpod State Management** → Works with Flutter/Dart and Testing
- **Exhaustive Type Checking** → Applies to Flutter/Dart and Riverpod
- **Rule Writing** → Meta-rule for maintaining all other rules

