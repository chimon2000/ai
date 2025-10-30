---
trigger: model_decision
description: "Rules applied to any Flutter or Dart projects"
---

#### Flutter/Dart Specific Rules

##### Widget Organization
- Always organize constructors before properties
- Always implement `const` constructors for widgets when possible
- Always add `super.key` to widget constructors
- **NEVER use widget helper methods** - extract widgets into separate classes instead
- **NEVER create private `_build*` methods** - use composition with separate widget classes
- Always extract complex widget trees into separate widget classes with clear responsibilities
- Use widget composition over widget helper methods for better testability and reusability

##### Widget State Management
- Use `StatelessWidget` for UI components without internal state (e.g., display-only widgets, static layouts)
- Use `StatefulWidget` only for components that need to maintain UI-related state (e.g., animations, form inputs, scroll positions, UI toggles)
- **NEVER store business logic or application state in StatefulWidget** - use Riverpod providers/controllers instead
- Store only ephemeral UI state in StatefulWidget (state that doesn't need to persist or be shared)
- Examples of valid StatefulWidget state: animation controllers, text field focus, local form validation, scroll positions
- Examples of invalid StatefulWidget state: user data, API responses, authentication state, app-wide settings
- For any state that needs to be shared across widgets or persisted, use Riverpod providers

##### Error Handling Patterns
- Always use `result_dart` package for operations that can fail
- Use `Result<T>` for success type T with Exception as failure type
- Use `ResultDart<T, E>` for custom failure types (e.g., `ResultDart<User, AuthError>`)
- Use `Result<Unit>`/`ResultDart<Unit, E>` for operations that don't return data but can fail
- Always handle both `Success` and `Failure` cases using `.isSuccess()` and `.isError()`
- Use `result.exceptionOrNull()` to access error details
- Wrap exceptions in proper Exception types, not raw strings
- Use `NetworkException.handleException()` for API error handling when available
- Implement proper error boundaries with custom error widgets
- Always handle `AsyncValue` states: `data`, `loading`, `error`

##### User-Facing Error Handling
- Always provide user feedback for async operations (loading states, error messages)
- Use `ScaffoldMessenger.of(context).showSnackBar()` for temporary error messages
- Use proper error boundaries for critical failures
- Handle `AsyncValue` states consistently: loading, data, error
- Provide meaningful error messages, not technical exception details
- Always check `mounted` before showing UI feedback in async operations

#### Project-Specific Patterns

##### Adaptive Project Structure
- **Small Projects**: Use flat structure with models/, services/, screens/, widgets/
- **Medium Projects**: Use feature-based structure with controllers/ and providers/
- **Large Projects**: Use the full structure below
- Always follow existing project structure patterns
- Group related files together (controller + screen + widgets)
- Use consistent naming conventions throughout the project

#### Navigation
- **Primary**: Use `AutoRoute` for all navigation when available in the project
- **Fallback**: If AutoRoute is not configured, use `Navigator.of(context).push()` with proper route management
- Always check existing navigation patterns in the codebase before implementing
- Use `@RoutePage()` annotation for all screen widgets when using AutoRoute
- Implement proper route guards and observers for protected routes
- Always use `appRouter.push()` instead of `Navigator` when AutoRoute is available

#### Dependency Management
- Always use package managers (flutter pub add) instead of manually editing pubspec.yaml
- Check existing dependencies before adding new ones to avoid conflicts
- Verify package compatibility with existing Flutter/Dart versions
- Use specific version constraints for critical dependencies
- Document why specific packages were chosen in comments or README
- Always run `flutter pub get` after dependency changes
- Use `flutter pub deps` to check for dependency conflicts

#### API Integration
- Always use Chopper service classes for API calls
- Use `ApiClient` from `get_it` for dependency injection
- Handle responses with `.when(success: ..., failure: ...)` pattern
- Always return fallback data in failure cases for CMS providers

#### Testing Patterns
- Use `mocktail` for mocking, not `mockito`
- Always create `createTestWidget()` helper with `ProviderScope`
- Use `ScreenUtilInit` wrapper in test widgets
- Mock controllers by extending `AutoDisposeNotifier` with `Mock`
- Always use `overrides` parameter for provider testing

#### Code Generation
- Run `dart run build_runner build -d` after model changes
- Use `@freezed` for immutable data classes
- Use `@JsonSerializable` for API models
- Always use `part` files for generated code