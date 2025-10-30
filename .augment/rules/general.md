---
type: "always_apply"
description: "General rules to apply in any project"
---

#### Core Principles

**Eliminates duplication ruthlessly:**
- Remove repeated code, logic, and patterns
- Extract common functionality into reusable components
- Use DRY (Don't Repeat Yourself) principle consistently

**Expresses intent clearly through naming and structure:**
- Use descriptive names for classes, methods, variables
- Structure code to reveal intent at first glance
- Avoid cryptic abbreviations or unclear patterns

**Makes dependencies explicit:**
- Declare all dependencies clearly
- Avoid hidden or implicit dependencies
- Use dependency injection where appropriate

**Keeps methods small and focused:**
- Single responsibility principle - one reason to change
- Methods should do one thing well
- Aim for methods under 20 lines when possible

**Minimizes state and side effects:**
- Prefer immutability and pure functions
- Reduce mutable state where possible
- Make side effects explicit and controlled

**Uses the simplest solution that could possibly work:**
- YAGNI (You Aren't Gonna Need It) - don't over-engineer
- Avoid premature optimization
- Choose clarity over cleverness

#### Context-Aware Rules

- **ALWAYS** examine existing codebase patterns before implementing new features
- If project uses different patterns than specified, follow existing patterns for consistency
- Document deviations from standard rules in code comments
- Prefer project consistency over rigid rule adherence when patterns conflict
- Use codebase-retrieval to understand existing architecture before starting

#### Rule Priority System

1. **Critical Priority**: Pre-task analysis, context awareness
2. **High Priority**: Error handling, testing, navigation
3. **Medium Priority**: Code organization, dependency management
4. **Low Priority**: Formatting, naming conventions

#### Conflict Resolution

1. **Project Consistency** > **Rule Adherence** (when patterns conflict)
2. **Existing Patterns** > **New Patterns** (unless refactoring entire codebase)
3. **User Requirements** > **Technical Preferences**