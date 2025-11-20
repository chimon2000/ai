---
type: "agent_requested"
description: "Code organization"
---

---
type: "agent_requested"
description: "Code organization"---

#### Code Organization

##### Choosing the Right Structure

**Small** (< 10 screens): Flat structure - models/, services/, screens/, widgets/
**Medium** (10-30 screens): Feature-based - ui/[feature]/ with stores
**Large** (30+ screens): MVU pattern - app/, data/, domain/, ui/ with state/

##### Decision Criteria for Migration

**Migrate Small → Medium when:**
- Project has 10+ screens or 3+ features
- Multiple screens share business logic
- Need to organize related screens together
- Team size growing (2+ developers)
- Complexity of state management increasing

**Migrate Medium → Large when:**
- Project has 30+ screens or 5+ features
- Complex business logic across multiple layers
- Need clear separation between data and presentation
- Team size 3+ developers
- Reusable domain logic across features
- API/repository layer becoming complex

**Signs you're ready to migrate:**
- Current structure feels disorganized
- Hard to find related files
- Duplicate code across features
- Difficult to test business logic
- New team members struggle with structure

**Signs you're not ready to migrate:**
- Project still < 10 screens
- Simple, linear data flow
- Single developer or very small team
- Prototype or MVP phase
- Frequent structural changes expected

##### Small Project Structure

```
lib/
├── main.dart
├── models/              # Data models
│   ├── user.dart
│   └── product.dart
├── services/            # Business logic & API calls
│   ├── auth_service.dart
│   └── api_service.dart
├── screens/             # All screens
│   ├── home_screen.dart
│   ├── login_screen.dart
│   └── profile_screen.dart
└── widgets/             # Shared widgets
    ├── custom_button.dart
    └── loading_indicator.dart
```

**When to use:** < 20 files, simple data flow, prototypes

##### Medium Project Structure

```
lib/
├── main.dart
├── models/              # Shared data models
│   └── user.dart
├── services/            # Shared services
│   └── api_service.dart
├── ui/                  # UI layer with features
│   ├── auth/           # Feature: Authentication
│   │   ├── auth_store.dart
│   │   ├── auth_provider.dart
│   │   ├── login_screen.dart
│   │   └── widgets/
│   │       └── login_form.dart
│   ├── home/           # Feature: Home
│   │   ├── home_store.dart
│   │   ├── home_screen.dart
│   │   └── widgets/
│   │       ├── home_header.dart
│   │       └── home_card.dart
│   ├── profile/        # Feature: Profile
│   │   ├── profile_store.dart
│   │   ├── profile_screen.dart
│   │   └── widgets/
│   │       └── profile_avatar.dart
│   └── widgets/        # Shared UI components
│       ├── buttons/
│       │   └── custom_button.dart
│       └── inputs/
│           └── custom_text_field.dart
└── utils/               # Shared utilities
    └── validators.dart
```

**When to use:** 10-30 screens, multiple features, growing team

##### MVU (Model-View-Update) Pattern

**Model**: Immutable state (Equatable) - pure data, no logic
**View**: Pure UI (screen widgets) - renders based on state
**Update**: Store (Notifier) - manages state transitions and side effects

**Benefits:** Clear separation, predictable state, easy testing, type-safe, unidirectional flow

**When to use:** Large projects, complex state, multiple state variations, testing priority

**Medium vs Large:**
- **Medium**: Store at feature root - `class AuthStore extends AsyncNotifier<User?>`
- **Large**: Store in state/ subdirectory - `class LoginStore extends Notifier<LoginState>` (with separate state file)

##### Large Project Structure (MVU Pattern)

```
lib/
├── main.dart
├── app/                    # App-level configuration
│   ├── router.dart        # Route definitions
│   ├── app.dart           # Root app widget
│   └── theme/
│       ├── app_theme.dart
│       └── colors.dart
├── core/                   # Core utilities & constants
│   ├── constants/
│   │   └── api_constants.dart
│   ├── extensions/
│   │   └── string_extensions.dart
│   └── utils/
│       └── logger.dart
├── data/                   # Data layer
│   ├── models/            # Data models
│   │   ├── user.dart
│   │   └── product.dart
│   ├── repositories/      # Data access
│   │   ├── user_repository.dart
│   │   └── product_repository.dart
│   └── services/          # External services
│       ├── api_service.dart
│       └── storage_service.dart
├── domain/                 # Business logic layer
│   ├── entities/          # Domain entities
│   │   └── user_entity.dart
│   └── usecases/          # Business use cases
│       └── login_usecase.dart
├── i18n/                   # Internationalization
│   ├── en.dart
│   └── es.dart
└── ui/                     # Presentation layer (MVU)
    ├── auth/              # Feature: Authentication
    │   ├── login/
    │   │   ├── login_screen.dart           # View - pure UI function
    │   │   ├── state/                      # Model - immutable state & domain logic
    │   │   │   ├── login_state.dart        # Screen state (Equatable)
    │   │   │   ├── login_store.dart        # Store - state management & updates
    │   │   │   └── login_event.dart        # User events/actions (optional)
    │   │   └── widgets/                    # Reusable view components
    │   │       ├── login_form.dart
    │   │       └── login_header.dart
    │   └── register/
    │       ├── register_screen.dart        # View - pure UI function
    │       ├── state/                      # Model - immutable state & domain logic
    │       │   ├── register_state.dart     # Screen state (Equatable)
    │       │   ├── register_store.dart     # Store - state management & updates
    │       │   └── register_event.dart     # User events/actions (optional)
    │       └── widgets/                    # Reusable view components
    │           └── register_form.dart
    ├── home/              # Feature: Home
    │   ├── home_screen.dart                # View - pure UI function
    │   ├── state/                          # Model - immutable state & domain logic
    │   │   ├── home_state.dart             # Screen state (Equatable)
    │   │   └── home_store.dart             # Store - state management & updates
    │   └── widgets/                        # Reusable view components
    │       ├── home_card.dart
    │       └── home_header.dart
    └── widgets/           # Reusable non-feature components
        ├── buttons/
        │   ├── primary_button.dart
        │   └── secondary_button.dart
        ├── inputs/
        │   └── custom_text_field.dart
        └── loading/
            └── loading_indicator.dart
```

**When to use:** 30+ screens, complex business logic, large team, clean architecture

**Example:**
```dart
// State (Model)
class ProfileState extends Equatable {
  final UserEntity? user;
  final bool isLoading;
  final String? error;

  const ProfileState({this.user, this.isLoading = false, this.error});

  ProfileState copyWith({UserEntity? user, bool? isLoading, String? error}) {
    return ProfileState(
      user: user ?? this.user,
      isLoading: isLoading ?? this.isLoading,
      error: error ?? this.error,
    );
  }

  @override
  List<Object?> get props => [user, isLoading, error];
}

// Store (Update) - manages state transitions and side effects
class ProfileStore extends Notifier<ProfileState> {
  @override
  ProfileState build() => const ProfileState();

  Future<void> loadUser(String userId) async {
    state = state.copyWith(isLoading: true, error: null);
    final result = await ref.read(userRepositoryProvider).getUser(userId);

    result.when(
      success: (user) => state = state.copyWith(user: user, isLoading: false),
      failure: (error) => state = state.copyWith(error: error.toString(), isLoading: false),
    );
  }
}

// Screen (View)
class ProfileScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(ProfileStore.provider);

    if (state.isLoading) return Center(child: CircularProgressIndicator());
    if (state.error != null) return Center(child: Text('Error: ${state.error}'));
    if (state.user == null) return Center(child: Text('No user'));

    return Scaffold(
      appBar: AppBar(title: Text(state.user!.name)),
      body: Text('Email: ${state.user!.email}'),
    );
  }
}
```

**Event Pattern (Optional):**

For complex features with many user interactions:

```dart
// Events
sealed class LoginEvent {}
class LoginSubmitted extends LoginEvent {
  final String email, password;
  LoginSubmitted(this.email, this.password);
}
class LoginEmailChanged extends LoginEvent {
  final String email;
  LoginEmailChanged(this.email);
}

// Store with event handler
class LoginStore extends Notifier<LoginState> {
  @override
  LoginState build() => const LoginState();

  void handleEvent(LoginEvent event) {
    switch (event) {
      case LoginSubmitted():
        _handleSubmit(event.email, event.password);
      case LoginEmailChanged():
        state = state.copyWith(email: event.email);
    }
  }

  Future<void> _handleSubmit(String email, String password) async {
    state = state.copyWith(isLoading: true);
    final result = await ref.read(authServiceProvider).login(email, password);
    result.when(
      success: (user) => state = state.copyWith(user: user, isLoading: false),
      failure: (error) => state = state.copyWith(error: error.toString(), isLoading: false),
    );
  }
}
```

**When to use:** Many user interactions, action logging/tracking, replay/undo, complex transitions

##### Testing Considerations

**Small**: Test screens/services directly, minimal mocking, integration tests
**Medium**: Test stores with mocked services, widget tests with mocked providers
**Large (MVU)**: Unit test state (copyWith, equality), unit test stores (mock repos), widget test screens (mock stores), integration test flows

**Example:**
```dart
// State test
test('ProfileState copyWith updates fields', () {
  const initial = ProfileState(isLoading: false);
  final updated = initial.copyWith(isLoading: true);
  expect(updated.isLoading, true);
});

// Store test
test('ProfileStore loads user', () async {
  final mockRepo = MockUserRepository();
  when(() => mockRepo.getUser('123')).thenAnswer((_) async => Success(testUser));

  final container = ProviderContainer(
    overrides: [userRepositoryProvider.overrideWithValue(mockRepo)],
  );

  final store = container.read(ProfileStore.provider.notifier);
  await store.loadUser('123');
  final state = container.read(ProfileStore.provider);

  expect(state.user, testUser);
  expect(state.isLoading, false);
});
```

##### Migration Paths

**Small → Medium (when criteria met):**

1. Create `ui/` directory for all UI components
2. Group related screens into feature folders under `ui/`
3. Move screen-specific widgets into feature `widgets/` folders
4. Add stores for business logic
5. Keep shared models and services at root level
6. Move shared widgets to `ui/widgets/`
7. Update imports and test all features work

```bash
# Before (Small)
lib/screens/login_screen.dart
lib/screens/home_screen.dart
lib/widgets/login_form.dart
lib/widgets/custom_button.dart

# After (Medium)
lib/ui/auth/login_screen.dart
lib/ui/auth/widgets/login_form.dart
lib/ui/auth/auth_store.dart
lib/ui/home/home_screen.dart
lib/ui/home/home_store.dart
lib/ui/widgets/buttons/custom_button.dart
```

**Medium → Large (when criteria met):**

1. Create `app/`, `data/`, `domain/` directories (keep existing `ui/`)
2. Reorganize features within `ui/` to add sub-feature structure
3. Refactor stores to MVU pattern:
   - Create `state/` directory in each feature
   - Split store into `[feature]_state.dart` (Equatable) and `[feature]_store.dart` (Notifier)
   - Add `[feature]_event.dart` if using event-driven updates
4. Extract data models to `data/models/`
5. Create repositories in `data/repositories/`
6. Add domain entities in `domain/entities/`
7. Move app config to `app/`
8. Extract shared utilities to `core/`
9. Shared widgets already in `ui/widgets/` (no change needed)
10. Migrate incrementally - one feature at a time

```bash
# Before (Medium)
lib/ui/auth/login_screen.dart
lib/ui/auth/auth_store.dart
lib/ui/auth/widgets/login_form.dart
lib/models/user.dart
lib/services/api_service.dart

# After (Large with MVU)
lib/ui/auth/login/login_screen.dart
lib/ui/auth/login/state/login_state.dart
lib/ui/auth/login/state/login_store.dart
lib/ui/auth/login/state/login_event.dart
lib/ui/auth/login/widgets/login_form.dart
lib/ui/widgets/buttons/primary_button.dart
lib/data/models/user.dart
lib/domain/entities/user_entity.dart
lib/data/repositories/user_repository.dart
lib/data/services/api_service.dart
```

#### General Principles

- Follow existing project structure - don't mix structures
- Group related files together
- Consistent naming: `*_screen.dart`, `*_store.dart`, `*_widget.dart`
- One widget per file (except small private helpers)
- Use `index.dart` for barrel exports
- Features should not import from other features

#### See Also

- **`flutter.md`** - Flutter/Dart specific rules for widgets and state management
- **`riverpod.md`** - Riverpod state management patterns and best practices
- **`testing.md`** - Testing strategies for different project structures
- **`general.md`** - Core principles (DRY, intent clarity, small methods, minimal state)