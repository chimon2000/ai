---
type: "always_apply"
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

**Why Testing Trophy over Test Pyramid:**

The Testing Trophy emphasizes **integration tests** because they:
- ✅ Test how components work together (closer to real usage)
- ✅ Catch more bugs than isolated unit tests
- ✅ Give confidence that features actually work
- ✅ Are resilient to refactoring (test behavior, not implementation)
- ✅ Provide good balance of speed and confidence

**Traditional pyramid over-emphasizes unit tests**, which:
- ❌ Often test implementation details
- ❌ Break when refactoring (even if behavior unchanged)
- ❌ Don't catch integration bugs
- ❌ Give false confidence

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

#### Unit Testing

##### Controller Testing

**Best Practices:**
- Test all public methods of controllers
- Use `mocktail` for mocking external services
- Register fallback values with `registerFallbackValue()` for complex types
- Test Result<T> return types with `.isSuccess()` and `.isError()`
- Test error handling paths
- Verify analytics calls are made correctly
- Test navigation calls with mock router

**Example:**

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:mocktail/mocktail.dart';

class MockUserService extends Mock implements UserService {}

void main() {
  late MockUserService mockUserService;
  late ProviderContainer container;

  setUp(() {
    mockUserService = MockUserService();
    container = ProviderContainer(
      overrides: [
        userServiceProvider.overrideWithValue(mockUserService),
      ],
    );
  });

  tearDown(() {
    container.dispose();
  });

  group('UserController', () {
    test('should load user successfully', () async {
      // Arrange
      final user = User(id: '1', name: 'John');
      when(() => mockUserService.getUser('1'))
          .thenAnswer((_) async => Success(user));

      // Act
      final controller = container.read(UserController.provider.notifier);
      final result = await controller.loadUser('1');

      // Assert
      expect(result.isSuccess(), true);
      expect(result.getOrNull(), user);
      verify(() => mockUserService.getUser('1')).called(1);
    });

    test('should handle error when loading user fails', () async {
      // Arrange
      when(() => mockUserService.getUser('1'))
          .thenAnswer((_) async => Failure(Exception('Network error')));

      // Act
      final controller = container.read(UserController.provider.notifier);
      final result = await controller.loadUser('1');

      // Assert
      expect(result.isError(), true);
      expect(result.exceptionOrNull()?.toString(), contains('Network error'));
    });
  });
}
```

##### Service Testing

**Best Practices:**
- Test all service methods
- Mock HTTP clients and external dependencies
- Test error handling and edge cases
- Verify correct data transformations

**Example:**

```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';

class MockHttpClient extends Mock implements HttpClient {}

void main() {
  late MockHttpClient mockHttpClient;
  late UserService userService;

  setUp(() {
    mockHttpClient = MockHttpClient();
    userService = UserService(mockHttpClient);
  });

  group('UserService', () {
    test('should return user when API call succeeds', () async {
      // Arrange
      when(() => mockHttpClient.get('/users/1'))
          .thenAnswer((_) async => {'id': '1', 'name': 'John'});

      // Act
      final result = await userService.getUser('1');

      // Assert
      expect(result.isSuccess(), true);
      expect(result.getOrNull()?.name, 'John');
    });

    test('should return failure when API call fails', () async {
      // Arrange
      when(() => mockHttpClient.get('/users/1'))
          .thenThrow(Exception('Network error'));

      // Act
      final result = await userService.getUser('1');

      // Assert
      expect(result.isError(), true);
    });
  });
}
```

#### Widget Testing

##### Best Practices

- Use `ProviderScope` wrapper for all Riverpod widget tests
- Always test widget rendering with `findsOneWidget`
- Test user interactions with `tester.tap()` and `tester.enterText()`
- Mock all external dependencies using provider overrides
- Test both success and error states for async widgets
- Use `tester.pump()` for simple tests, `tester.pumpAndSettle()` only when animations/async operations are expected
- Add keys to widgets for easier testing
- Test accessibility features (semantics, labels)

#### Integration Testing

##### Best Practices

- Test complete user flows end-to-end
- Use real providers where possible, mock external services
- Test offline scenarios and error recovery
- Verify data persistence across app restarts
- Test navigation flows
- Test authentication flows
- Verify data synchronization

##### Widget Testing Tips
```dart
// Add keys to widgets for testing
class MyScreen extends StatelessWidget {
  static const Key appBarKey = Key('my_screen_app_bar');
  static const Key listViewKey = Key('my_screen_list_view');
  
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        key: appBarKey,
        title: Text('Title'),
      ),
      body: ListView(
        key: listViewKey,
        children: [...],
      ),
    );
  }
}

// Find widgets by key instead of type
expect(find.byKey(MyScreen.appBarKey), findsOneWidget);
expect(find.byKey(MyScreen.listViewKey), findsOneWidget);

// Only use scrollUntilVisible when testing scrollable widgets
// For non-scrollable screens, widgets should be found directly
```


##### Controller Testing Pattern
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:sqflite_common_ffi/sqflite_ffi.dart';

void main() {
  // Only initialize database if controller uses offline storage/persistence
  TestWidgetsFlutterBinding.ensureInitialized();
  sqfliteFfiInit();
  databaseFactory = databaseFactoryFfi;

  group('FeatureController', () {
    test('should have initial state', () {
      final container = ProviderContainer();
      final controller = container.read(FeatureController.provider);
      
      expect(controller.someValue, 'default');
    });

    test('should update state', () {
      final container = ProviderContainer();
      final controller = container.read(FeatureController.provider.notifier);
      
      controller.updateValue('new value');
      
      final state = container.read(FeatureController.provider);
      expect(state.someValue, 'new value');
    });
  });
}
```

##### Screen Testing Pattern
```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:mocktail/mocktail.dart';
import 'package:sqflite_common_ffi/sqflite_ffi.dart';

// Mock Controller for AsyncNotifier
class MockFeatureController extends AsyncNotifier<FeatureState>
    with Mock
    implements FeatureController {
  final FeatureState initialState;
  final bool shouldError;

  MockFeatureController({
    this.initialState = const FeatureState(),
    this.shouldError = false,
  });

  @override
  Future<FeatureState> build() async {
    if (shouldError) {
      throw Exception('Failed to load');
    }
    return initialState;
  }
}

void main() {
  TestWidgetsFlutterBinding.ensureInitialized();
  
  // Only needed if the screen's controller uses offline storage
  sqfliteFfiInit();
  databaseFactory = databaseFactoryFfi;

  group('FeatureScreen Tests', () {
    late MockFeatureController mockController;

    Widget createTestWidget({
      FeatureState? state,
      bool shouldError = false,
    }) {
      mockController = MockFeatureController(
        initialState: state ?? const FeatureState(),
        shouldError: shouldError,
      );

      return ProviderScope(
        overrides: [
          FeatureController.provider.overrideWith(() => mockController),
        ],
        child: const MaterialApp(home: FeatureScreen()),
      );
    }

    testWidgets('should display loading state', (tester) async {
      await tester.pumpWidget(
        ProviderScope(
          overrides: [
            FeatureController.provider.overrideWithBuild((ref, notifier) async {
              await Future.delayed(const Duration(seconds: 1));
              return const FeatureState();
            }),
          ],
          child: const MaterialApp(home: FeatureScreen()),
        ),
      );
      
      // Initial pump shows loading
      await tester.pump();
      expect(find.byType(CircularProgressIndicator), findsOneWidget);
      
      // Complete loading
      await tester.pumpAndSettle();
      expect(find.byType(CircularProgressIndicator), findsNothing);
    });

    testWidgets('should display error state', (tester) async {
      await tester.pumpWidget(createTestWidget(shouldError: true));
      await tester.pumpAndSettle();

      expect(find.text('Error: Exception: Failed to load'), findsOneWidget);
    });

    testWidgets('should display content with scrolling', (tester) async {
      await tester.pumpWidget(createTestWidget());
      await tester.pumpAndSettle();

      // Find visible widget
      expect(find.byKey(const Key('visible_widget')), findsOneWidget);

      // Only use scrollUntilVisible if the screen has scrollable content
      await tester.scrollUntilVisible(
        find.byKey(const Key('hidden_widget')),
        100,
      );
      expect(find.byKey(const Key('hidden_widget')), findsOneWidget);
    });

    testWidgets('should update when controller state changes', (tester) async {
      await tester.pumpWidget(createTestWidget());
      await tester.pumpAndSettle();

      // Mock state change
      when(() => mockController.updateValue(any())).thenAnswer((_) async {
        mockController.state = AsyncData(
          const FeatureState(value: 'updated'),
        );
      });

      await mockController.updateValue('updated');
      await tester.pumpAndSettle();

      expect(find.text('updated'), findsOneWidget);
    });
  });
}
```

#### See Also

- **`riverpod.md`** - Riverpod-specific patterns for controllers and providers
- **`flutter.md`** - Flutter/Dart specific rules for error handling and state management
- **`tidy_first.md`** - Red-Green-Refactor cycle and refactoring guidelines
- **`general.md`** - Core principles (DRY, intent clarity, small methods, minimal state)