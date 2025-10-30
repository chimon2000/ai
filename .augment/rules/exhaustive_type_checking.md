---
type: "agent_requested"
description: "Exhaustive type checking patterns in Dart/Flutter code"
---

#### Exhaustive Type Checking with Sealed Classes and Enums

##### Principle

When you have a known, finite set of subtypes or variants, always use patterns that enable compile-time exhaustive checking rather than runtime type checks or open inheritance hierarchies. This prevents silent bugs when new types are added but not handled everywhere they should be.

##### Core Pattern: Use Sealed Classes for Type Hierarchies

**✅ GOOD: Sealed class with exhaustive switch**
```dart
sealed class Result<T> {}

class Success<T> extends Result<T> {
  final T data;
  Success(this.data);
}

class Failure<T> extends Result<T> {
  final Exception error;
  Failure(this.error);
}

// Compiler enforces all cases are handled
String handleResult<T>(Result<T> result) {
  return switch (result) {
    Success(:final data) => 'Success: $data',
    Failure(:final error) => 'Error: $error',
    // Compiler error if new subtype added without handling it here
  };
}
```

**❌ BAD: Abstract class with runtime checks**
```dart
abstract class Result<T> {}

class Success<T> extends Result<T> {
  final T data;
  Success(this.data);
}

class Failure<T> extends Result<T> {
  final Exception error;
  Failure(this.error);
}

// Silent bug: if new subtype added, this code still compiles
String handleResult<T>(Result<T> result) {
  if (result is Success) return 'Success: ${result.data}';
  if (result is Failure) return 'Error: ${result.error}';
  return 'Unknown'; // Hides bugs
}
```

##### Pattern 1: Use Enums for Simple Value Sets

**✅ GOOD: Enum for known values**
```dart
enum Status {
  pending,
  approved,
  rejected,
  cancelled,
}

// Exhaustive switch - compiler error if case missing
String statusLabel(Status status) {
  return switch (status) {
    Status.pending => 'Pending Review',
    Status.approved => 'Approved',
    Status.rejected => 'Rejected',
    Status.cancelled => 'Cancelled',
  };
}
```

**❌ BAD: String constants**
```dart
class Status {
  static const String pending = 'pending';
  static const String approved = 'approved';
  static const String rejected = 'rejected';
  // Easy to forget values, no compile-time checking
}

// No way to verify all values are handled
String statusLabel(String status) {
  if (status == Status.pending) return 'Pending Review';
  if (status == Status.approved) return 'Approved';
  return 'Unknown'; // Silent bug if new status added
}
```

##### Pattern 2: Use Sealed Classes for Complex Type Hierarchies

**✅ GOOD: Sealed class for state machine**
```dart
sealed class LoadingState {}

class Idle extends LoadingState {}

class Loading extends LoadingState {}

class Success extends LoadingState {
  final List<Item> items;
  Success(this.items);
}

class Error extends LoadingState {
  final String message;
  Error(this.message);
}

// Compiler verifies all states are handled
Widget buildUI(LoadingState state) {
  return switch (state) {
    Idle() => const SizedBox.shrink(),
    Loading() => const CircularProgressIndicator(),
    Success(:final items) => ListView(children: items.map((i) => Text(i.name)).toList()),
    Error(:final message) => ErrorWidget(message: message),
  };
}
```

**❌ BAD: Abstract class with if-else chains**
```dart
abstract class LoadingState {}

class Idle extends LoadingState {}
class Loading extends LoadingState {}
class Success extends LoadingState {
  final List<Item> items;
  Success(this.items);
}

// Easy to miss cases, no compile-time verification
Widget buildUI(LoadingState state) {
  if (state is Idle) return const SizedBox.shrink();
  if (state is Loading) return const CircularProgressIndicator();
  if (state is Success) return ListView(children: state.items.map((i) => Text(i.name)).toList());
  return Container(); // Silent bug if new state added
}
```

##### Pattern 3: Use Switch Expressions for Pattern Matching

**✅ GOOD: Exhaustive switch expression**
```dart
sealed class ApiResponse<T> {}

class ApiSuccess<T> extends ApiResponse<T> {
  final T data;
  ApiSuccess(this.data);
}

class ApiError<T> extends ApiResponse<T> {
  final int statusCode;
  final String message;
  ApiError(this.statusCode, this.message);
}

class ApiNetworkError<T> extends ApiResponse<T> {
  final String reason;
  ApiNetworkError(this.reason);
}

// Compiler error if any case is missing
Future<void> handleResponse<T>(ApiResponse<T> response) async {
  return switch (response) {
    ApiSuccess(:final data) => _handleSuccess(data),
    ApiError(:final statusCode, :final message) => _handleApiError(statusCode, message),
    ApiNetworkError(:final reason) => _handleNetworkError(reason),
  };
}
```

**❌ BAD: Switch with default case**
```dart
// Default case hides missing cases
Future<void> handleResponse<T>(ApiResponse<T> response) async {
  switch (response) {
    case ApiSuccess():
      _handleSuccess(response.data);
      break;
    case ApiError():
      _handleApiError(response.statusCode, response.message);
      break;
    default:
      // Silent bug: new ApiNetworkError case not handled
      break;
  }
}
```

##### Pattern 4: Sealed Classes with Generics

**✅ GOOD: Generic sealed class for flexible type hierarchies**
```dart
sealed class Either<L, R> {}

class Left<L, R> extends Either<L, R> {
  final L value;
  Left(this.value);
}

class Right<L, R> extends Either<L, R> {
  final R value;
  Right(this.value);
}

// Exhaustive pattern matching with generics
T fold<T>(Either<L, R> either, T Function(L) onLeft, T Function(R) onRight) {
  return switch (either) {
    Left(:final value) => onLeft(value),
    Right(:final value) => onRight(value),
  };
}
```

##### Pattern 5: Sealed Classes with Mixins for Shared Behavior

**✅ GOOD: Sealed class with mixin for shared functionality**
```dart
mixin Retryable {
  int get retryCount;
  bool get canRetry => retryCount < 3;
}

sealed class TaskState {}

class Pending extends TaskState {}

class Running extends TaskState with Retryable {
  @override
  final int retryCount;
  Running(this.retryCount);
}

class Failed extends TaskState with Retryable {
  final String error;
  @override
  final int retryCount;
  Failed(this.error, this.retryCount);
}

class Completed extends TaskState {
  final String result;
  Completed(this.result);
}

// Exhaustive handling with mixin access
void processTask(TaskState state) {
  switch (state) {
    case Pending():
      print('Task pending');
    case Running(:final retryCount):
      print('Running, retry count: $retryCount');
    case Failed(:final error, :final retryCount):
      print('Failed: $error, retries: $retryCount');
    case Completed(:final result):
      print('Completed: $result');
  }
}
```

##### When to Apply This Rule

- **State machines** with known states (Loading, Success, Error, etc.)
- **Result/Either types** for error handling
- **API response types** with known variants
- **UI state representations** (Idle, Loading, Content, Error)
- **Navigation routes** with known destinations
- **Domain models** with fixed subtypes
- **Event types** in event-driven architectures
- **Permission states** (Granted, Denied, NotDetermined)
- **Authentication states** (Unauthenticated, Authenticating, Authenticated, Error)

##### Benefits

- ✅ **Compile-time safety**: Compiler error when new subtypes added but not handled
- ✅ **No silent bugs**: Can't accidentally miss a case with a default clause
- ✅ **Better IDE support**: Autocomplete shows all available cases
- ✅ **Self-documenting**: All variants visible in one place
- ✅ **Refactoring safety**: Adding new type forces updates everywhere it's used
- ✅ **Pattern matching**: Destructure data directly in switch cases

##### Migration Guide: From Abstract to Sealed

**Step 1: Change abstract to sealed**
```dart
// Before
abstract class Result<T> {}

// After
sealed class Result<T> {}
```

**Step 2: Update switch statements to be exhaustive**
```dart
// Before
switch (result) {
  case Success():
    handleSuccess();
    break;
  case Failure():
    handleFailure();
    break;
  default:
    break;
}

// After
switch (result) {
  case Success():
    handleSuccess();
  case Failure():
    handleFailure();
}
```

**Step 3: Convert if-else chains to switch expressions**
```dart
// Before
String message;
if (result is Success) {
  message = 'Success: ${result.data}';
} else if (result is Failure) {
  message = 'Error: ${result.error}';
} else {
  message = 'Unknown';
}

// After
final message = switch (result) {
  Success(:final data) => 'Success: $data',
  Failure(:final error) => 'Error: $error',
};
```

##### Common Patterns in Ava Project

**Result Type (Already in use)**
```dart
sealed class Result<T> {}

class Success<T> extends Result<T> {
  final T data;
  Success(this.data);
}

class Failure<T> extends Result<T> {
  final Exception error;
  Failure(this.error);
}
```

**AsyncValue States (Riverpod)**
```dart
// Already uses exhaustive checking with AsyncValue
// Always handle: data, loading, error
state.when(
  data: (data) => DataWidget(data),
  loading: () => LoadingWidget(),
  error: (error, stack) => ErrorWidget(error),
);
```

**Screen States**
```dart
sealed class HomeScreenState {}

class HomeLoading extends HomeScreenState {}

class HomeLoaded extends HomeScreenState {
  final List<Card> cards;
  final CreditScore creditScore;
  HomeLoaded(this.cards, this.creditScore);
}

class HomeError extends HomeScreenState {
  final String message;
  HomeError(this.message);
}
```

##### Linting and Tooling

**Enable exhaustiveness checking in analysis_options.yaml:**
```yaml
linter:
  rules:
    - exhaustive_cases  # Warns about non-exhaustive switch statements
```

**Use sealed classes with Freezed (if needed):**
```dart
@freezed
sealed class Result<T> with _$Result<T> {
  const factory Result.success(T data) = Success<T>;
  const factory Result.failure(Exception error) = Failure<T>;
}
```

##### Anti-Patterns to Avoid

**❌ AVOID: Open inheritance with runtime checks**
```dart
abstract class Animal {}
class Dog extends Animal {}
class Cat extends Animal {}
// Anyone can add new subtypes anywhere
```

**❌ AVOID: String/int constants instead of enums**
```dart
class Color {
  static const int red = 0xFF0000;
  static const int green = 0xFF00FF00;
  // Easy to use invalid values
}
```

**❌ AVOID: Default cases that hide missing cases**
```dart
switch (state) {
  case Loading:
    break;
  default:
    // Hides missing cases
    break;
}
```

**❌ AVOID: Nullable types instead of sealed classes**
```dart
class Result<T> {
  final T? data;
  final Exception? error;
  // Can't tell which is valid, no exhaustive checking
}
```

##### References

- [Dart 3.0 Sealed Classes](https://dart.dev/language/class-modifiers#sealed)
- [Pattern Matching in Dart](https://dart.dev/language/patterns)
- [Switch Expressions](https://dart.dev/language/branches#switch-expressions)
- [Exhaustiveness Checking](https://dart.dev/language/patterns#exhaustiveness-checking)

