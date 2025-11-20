# AI Agent Rules

Universal rules for AI coding assistants working on this project. These rules apply across all IDEs and tools (Cursor, Windsurf, Claude Desktop, etc.).

## Table of Contents

- [AI Agent Rules](#ai-agent-rules)
  - [Table of Contents](#table-of-contents)
  - [Core Principles](#core-principles)
  - [Testing \& Quality](#testing--quality)
    - [Test-Driven Development (TDD)](#test-driven-development-tdd)
    - [Testing Trophy Model](#testing-trophy-model)
    - [Tidy First: Structural Before Behavioral](#tidy-first-structural-before-behavioral)
    - [Refactoring Guidelines](#refactoring-guidelines)
  - [Documentation](#documentation)
  - [Architecture \& Organization](#architecture--organization)
  - [Flutter \& Dart](#flutter--dart)
  - [State Management](#state-management)
  - [Type Safety](#type-safety)
  - [Meta Rules](#meta-rules)
  - [Configuration Locations](#configuration-locations)

---

## Core Principles

**Always Apply:**
- **Eliminates duplication ruthlessly** - Remove repeated code, logic, and patterns
- **Expresses intent clearly** - Use descriptive names for classes, methods, variables
- **Makes dependencies explicit** - Declare all dependencies clearly
- **Keeps methods small and focused** - Single responsibility principle, aim for < 20 lines
- **Minimizes state and side effects** - Prefer immutability and pure functions
- **Uses the simplest solution** - YAGNI principle, avoid premature optimization

**Context-Aware:**
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

---

## Testing & Quality

### Test-Driven Development (TDD)

**ALWAYS:**
- Write tests first for any new functionality - no exceptions
- Follow the Red-Green-Refactor (TDD) cycle strictly
- Run tests after each step to ensure they pass/fail as expected

**NEVER:**
- Write production code without a failing test first
- Skip the refactor step - always clean up code after making tests pass

### Testing Trophy Model

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

**What to Test:**
- ✅ User behavior and feature flows
- ✅ Store + Repository + Service integration
- ✅ State management with real dependencies
- ✅ Navigation flows
- ✅ Form validation with submission
- ✅ Error handling across layers
- ✅ Widget + Store integration
- ❌ Third-party packages
- ❌ Flutter framework widgets
- ❌ Generated code (*.g.dart, *.freezed.dart)
- ❌ Simple getters/setters without logic
- ❌ UI styling (colors, fonts)
- ❌ Implementation details

### Tidy First: Structural Before Behavioral

**Principles:**
- **Always make structural changes first** before behavioral changes
- **Never mix structural and behavioral changes** - commit them separately

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

---

## Documentation

**README File Length Limit:**
- README.md files should **never exceed 400 lines** (not characters)
- Keep content concise, scannable, and focused on essential information

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

## Architecture & Organization

**Choosing the Right Structure:**
- **Small** (< 10 screens): Flat structure - models/, services/, screens/, widgets/
- **Medium** (10-30 screens): Feature-based - ui/[feature]/ with stores
- **Large** (30+ screens): MVU pattern - app/, data/, domain/, ui/ with state/

**General Principles:**
- Follow existing project structure - don't mix structures
- Group related files together
- Consistent naming: `*_screen.dart`, `*_store.dart`, `*_widget.dart`
- One widget per file (except small private helpers)
- Use `index.dart` for barrel exports
- Features should not import from other features

**MVU (Model-View-Update) Pattern:**
- **Model**: Immutable state (Equatable) - pure data, no logic
- **View**: Pure UI (screen widgets) - renders based on state
- **Update**: Store (Notifier) - manages state transitions and side effects

---

## Flutter & Dart

**Widget Organization:**
- Always organize constructors before properties
- Always implement `const` constructors for widgets when possible
- Always add `super.key` to widget constructors
- **NEVER use widget helper methods** - extract widgets into separate classes
- **NEVER create private `_build*` methods** - use composition with separate widget classes

**Widget State Management:**
- Use `StatelessWidget` for UI components without internal state
- Use `StatefulWidget` only for components that need to maintain UI-related state
- **NEVER store business logic or application state in StatefulWidget**
- Store only ephemeral UI state in StatefulWidget
- For any state that needs to be shared across widgets or persisted, use Riverpod providers

**Error Handling Patterns:**
- Always use `result_dart` package for operations that can fail
- Use `Result<T>` for success type T with Exception as failure type
- Use `ResultDart<T, E>` for custom failure types
- Always handle both `Success` and `Failure` cases using `.isSuccess()` and `.isError()`
- Use `result.exceptionOrNull()` to access error details
- Wrap exceptions in proper Exception types, not raw strings
- Always handle `AsyncValue` states: `data`, `loading`, `error`

**User-Facing Error Handling:**
- Always provide user feedback for async operations (loading states, error messages)
- Use `ScaffoldMessenger.of(context).showSnackBar()` for temporary error messages
- Use proper error boundaries for critical failures
- Provide meaningful error messages, not technical exception details
- Always check `mounted` before showing UI feedback in async operations

**Navigation:**
- **Primary**: Use `AutoRoute` for all navigation when available
- **Fallback**: If AutoRoute is not configured, use `Navigator.of(context).push()`
- Always check existing navigation patterns in the codebase before implementing

**Dependency Management:**
- Always use package managers (flutter pub add) instead of manually editing pubspec.yaml
- Check existing dependencies before adding new ones to avoid conflicts
- Verify package compatibility with existing Flutter/Dart versions

**Code Generation:**
- Run `dart run build_runner build -d` after model changes
- Use `@freezed` for immutable data classes
- Use `@JsonSerializable` for API models
- Always use `part` files for generated code

---

## State Management

**Riverpod State Management:**

**Synchronous (Notifier):**
- Use `Notifier` for synchronous state management
- Use `AutoDisposeNotifierProvider` to prevent memory leaks
- Keep controllers focused on a single feature or domain
- Use `state = state.copyWith()` for immutable state updates
- Never mutate state directly - always create new instances

**Asynchronous (AsyncNotifier):**
- Use `AsyncNotifier` for async operations
- Use `FutureProvider.autoDispose` for one-time async operations
- Use `ref.cacheFor()` for expensive operations that should be cached
- Use `ref.refreshIn()` for data that should auto-refresh

**Provider Selection Decision Tree:**
- Is the data async (Future/Stream)? → Use `AsyncNotifier` or `FutureProvider`
- Does the data need to be cached? → Use `FutureProvider` with `cacheFor()`
- Is this a one-time operation? → Use `FutureProvider.autoDispose`
- Does this need to be watched by widgets? → Use `Notifier` or `AsyncNotifier`

---

## Type Safety

**Exhaustive Type Checking:**

**Core Pattern: Use Sealed Classes for Type Hierarchies:**
- ✅ **GOOD**: Sealed class with exhaustive switch - Compiler enforces all cases are handled
- ❌ **BAD**: Abstract class with runtime checks - Silent bug if new type added

**When to Apply:**
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

---

## Meta Rules

**Rule Writing Best Practices:**

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

**Quality Checklist:**
- [ ] Under 650 lines
- [ ] Includes concrete code examples
- [ ] Uses ❌ BAD / ✅ GOOD comparisons for anti-patterns
- [ ] Scannable with clear headings and bullet points
- [ ] No vague language ("consider", "might", "should probably")
- [ ] References specific files/patterns when applicable
- [ ] All code examples are concise and focused
- [ ] No redundant or duplicate content

---

## Configuration Locations

- **Augment**: `.augment/rules/` (source of truth)
- **Cursor IDE**: `.cursor/rules/`
- **Windsurf IDE**: `.windsurf/rules/`
- **Claude Desktop**: `.claude/CLAUDE.md`
- **Universal**: `agents.md` (this file)

