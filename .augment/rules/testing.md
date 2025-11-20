---
type: "always_apply"
description: "Testing requirements and patterns for TDD, widget testing, and controller testing"
---

#### Testing Strategy

##### Test-Driven Development (TDD)

- **Write tests first** for any new functionality
- **Follow the Red-Green-Refactor (TDD) cycle** when writing code:
  1. **RED**: Write a failing test that describes the desired behavior
  2. **GREEN**: Write the minimal code to make the test pass
  3. **REFACTOR**: Improve the code while keeping tests green
- **Run tests after each step** to ensure they pass/fail as expected
- **Exception**: Spike/exploration work may skip TDD temporarily, but must be refactored with tests before merging

##### Testing Trophy Model

Follow the Testing Trophy for effective test coverage:

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

**Unit Tests (~20%):**
- Pure functions, utilities, models
- Complex business logic in isolation
- Data transformations and calculations
- Edge cases and boundary conditions

**Integration Tests (~50% - PRIMARY FOCUS):**
- Feature flows with stores and real dependencies
- User interactions and state changes together
- Stores with real repositories (mock only external APIs)
- Widget + store + service integration
- Data flows through the app

**E2E Tests (~30%):**
- Critical user journeys end-to-end
- Complete flows: login → dashboard → action → logout
- Real backend (or realistic mocks)
- Happy paths and critical business flows

**Why Integration Tests are Primary:**
- ✅ Test how components work together (closer to real usage)
- ✅ Catch more bugs than isolated unit tests
- ✅ Resilient to refactoring (test behavior, not implementation)
- ✅ Balance between speed and confidence

##### What to Test

**Integration Tests (Primary):**
- User behavior and feature flows
- Store + Repository + Service integration
- State management with real dependencies (mock only external APIs)
- Navigation flows and screen transitions
- Form validation with submission
- Error handling across layers
- Widget + Store integration

**Unit Tests (Secondary):**
- Complex business logic (calculations, algorithms)
- Pure functions (utilities, validators, formatters)
- Data models (serialization, equality, copyWith)
- Edge cases (boundary conditions, null handling)

**E2E Tests (Tertiary):**
- Critical user journeys (login, checkout, payment)
- Happy paths and common user flows
- Cross-feature flows
- Regression prevention

**Don't test:**
- ❌ Third-party packages
- ❌ Flutter framework widgets
- ❌ Generated code (*.g.dart, *.freezed.dart)
- ❌ Simple getters/setters without logic
- ❌ UI styling (colors, fonts)
- ❌ Implementation details

#### When to Apply Testing Trophy

**Use Testing Trophy (50% integration focus) for:**
- Feature-complete screens with stores and services
- Complex user workflows and state management
- Production apps with multiple features
- Teams prioritizing confidence over speed

**Focus on Unit Tests (80%+ unit focus) for:**
- Utility libraries and pure functions
- Complex algorithms or business logic
- Shared components used across features
- Performance-critical code

**Use E2E Tests (30%+ E2E focus) for:**
- Critical user journeys (authentication, payments)
- Multi-screen workflows
- Regression prevention for high-value features

#### Unit Testing

##### Store Testing

**Best Practices:**
- Test all public methods of stores
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

  group('UserStore', () {
    test('should load user successfully', () async {
      // Arrange
      final user = User(id: '1', name: 'John');
      when(() => mockUserService.getUser('1'))
          .thenAnswer((_) async => Success(user));

      // Act
      final store = container.read(UserStore.provider.notifier);
      final result = await store.loadUser('1');

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
      final store = container.read(UserStore.provider.notifier);
      final result = await store.loadUser('1');

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


##### Store Testing Pattern
```dart
import 'package:flutter_test/flutter_test.dart';
import 'package:sqflite_common_ffi/sqflite_ffi.dart';

void main() {
  // Only initialize database if store uses offline storage/persistence
  TestWidgetsFlutterBinding.ensureInitialized();
  sqfliteFfiInit();
  databaseFactory = databaseFactoryFfi;

  group('FeatureStore', () {
    test('should have initial state', () {
      final container = ProviderContainer();
      final store = container.read(FeatureStore.provider);

      expect(store.someValue, 'default');
    });

    test('should update state', () {
      final container = ProviderContainer();
      final store = container.read(FeatureStore.provider.notifier);

      store.updateValue('new value');

      final state = container.read(FeatureStore.provider);
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

// Mock Store for AsyncNotifier
class MockFeatureStore extends AsyncNotifier<FeatureState>
    with Mock
    implements FeatureStore {
  final FeatureState initialState;
  final bool shouldError;

  MockFeatureStore({
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

  // Only needed if the screen's store uses offline storage
  sqfliteFfiInit();
  databaseFactory = databaseFactoryFfi;

  group('FeatureScreen Tests', () {
    late MockFeatureStore mockStore;

    Widget createTestWidget({
      FeatureState? state,
      bool shouldError = false,
    }) {
      mockStore = MockFeatureStore(
        initialState: state ?? const FeatureState(),
        shouldError: shouldError,
      );

      return ProviderScope(
        overrides: [
          FeatureStore.provider.overrideWith(() => mockStore),
        ],
        child: const MaterialApp(home: FeatureScreen()),
      );
    }

    testWidgets('should display loading state', (tester) async {
      await tester.pumpWidget(
        ProviderScope(
          overrides: [
            FeatureStore.provider.overrideWithBuild((ref, notifier) async {
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

    testWidgets('should update when store state changes', (tester) async {
      await tester.pumpWidget(createTestWidget());
      await tester.pumpAndSettle();

      // Mock state change
      when(() => mockStore.updateValue(any())).thenAnswer((_) async {
        mockStore.state = AsyncData(
          const FeatureState(value: 'updated'),
        );
      });

      await mockStore.updateValue('updated');
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