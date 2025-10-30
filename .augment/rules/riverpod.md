---
type: "agent_requested"
description: "Rules applied to a Flutter project that uses Riverpod for state management."
---

#### State Management with Riverpod

##### Synchronous (Notifier):
```dart
class FeatureController extends Notifier<FeatureState> {
  @override
  FeatureState build() => const FeatureState();

  void updateValue(String value) => state = state.copyWith(value: value);

  static final provider = AutoDisposeNotifierProvider<FeatureController, FeatureState>(
    FeatureController.new,
  );
}

// Usage
final state = ref.watch(FeatureController.provider);
ref.read(FeatureController.provider.notifier).updateValue('new');
```

##### Asynchronous (AsyncNotifier):
```dart
class FeatureController extends AsyncNotifier<FeatureState> {
  @override
  Future<FeatureState> build() async => FeatureState(data: await _loadData());

  Future<void> updateValue(String value) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      return state.value!.copyWith(value: value);
    });
  }

  static final provider = AutoDisposeAsyncNotifierProvider<FeatureController, FeatureState>(
    FeatureController.new,
  );
}

// Usage
final stateAsync = ref.watch(FeatureController.provider);
stateAsync.when(
  data: (state) => Text(state.value),
  loading: () => CircularProgressIndicator(),
  error: (error, stack) => Text('Error: $error'),
);
```

**Guidelines:**
- Use `FutureProvider.autoDispose` for one-time async operations
- Use `ref.cacheFor()` for expensive operations that should be cached
- Use `ref.refreshIn()` for data that should auto-refresh

#### Controller Pattern

- Separate business logic into controller classes
- Use `AutoDisposeAsyncNotifier` for async controllers
- Controllers handle business logic, navigation, analytics
- Widgets handle UI rendering and user interactions only
- Use `Result<T>` for controller methods (Exception as failure type)
- Use `ResultDart<T, E>` for custom failure types
- Use `Result<Unit>` for operations that don't return data

#### Provider Organization

- Group providers by feature in dedicated folders
- Consistent naming: `*Provider`, `*Controller`, `*Service`
- Always use `autoDispose` unless global state is needed
- Implement proper provider dependencies with `ref.watch()`

#### Patterns

- Prefer `ConsumerWidget` over `StatefulWidget`
- Use `ConsumerStatefulWidget` only for lifecycle methods
- **In build()**: Use `ref.watch()` (rebuilds on state change)
- **In callbacks**: Use `ref.read()` (one-time access)
- Never use `ref.watch()` in callbacks or lifecycle methods
- Never use `ref.read()` in build() - won't rebuild on state change

#### Common Anti-Patterns

**❌ BAD: Business logic in widgets**
```dart
class UserScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () async {
        final response = await http.get(Uri.parse('/user'));
        final user = User.fromJson(jsonDecode(response.body));
        ref.read(userProvider.notifier).state = user;
      },
      child: Text('Load User'),
    );
  }
}
```

**✅ GOOD: Business logic in controller**
```dart
class UserController extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async => null;

  Future<Result<User>> loadUser() async {
    state = const AsyncLoading();
    final result = await ref.read(userServiceProvider).getUser();
    state = result.when(
      success: (user) => AsyncData(user),
      failure: (error) => AsyncError(error, StackTrace.current),
    );
    return result;
  }

  static final provider = AsyncNotifierProvider<UserController, User?>(
    UserController.new,
  );
}

class UserScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () => ref.read(UserController.provider.notifier).loadUser(),
      child: Text('Load User'),
    );
  }
}
```

**❌ BAD: ref.read in build() - won't rebuild**
```dart
class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.read(counterProvider);
    return Text('$count');
  }
}
```

**✅ GOOD: ref.watch in build() - rebuilds on change**
```dart
class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('$count');
  }
}
```

**❌ BAD: ref.watch in callbacks - throws error**
```dart
class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () => ref.watch(counterProvider.notifier).increment(),
      child: Text('Increment'),
    );
  }
}
```

**✅ GOOD: ref.read in callbacks**
```dart
class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    return ElevatedButton(
      onPressed: () => ref.read(counterProvider.notifier).increment(),
      child: Text('Increment'),
    );
  }
}
```

**❌ BAD: Not using autoDispose - stays in memory**
```dart
final searchResultsProvider = FutureProvider<List<Result>>((ref) async {
  final query = ref.watch(searchQueryProvider);
  return await searchService.search(query);
});
```

**✅ GOOD: Using autoDispose - disposed when unused**
```dart
final searchResultsProvider = FutureProvider.autoDispose<List<Result>>((ref) async {
  final query = ref.watch(searchQueryProvider);
  return await searchService.search(query);
});
```

**❌ BAD: Rebuilds entire widget on any property change**
```dart
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final user = ref.watch(userProvider);
    return Column(
      children: [
        Text(user.name),
        Text(user.email),
        ExpensiveWidget(), // Rebuilds unnecessarily
      ],
    );
  }
}
```

**✅ GOOD: Use select() to watch specific properties**
```dart
class UserProfile extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userName = ref.watch(userProvider.select((user) => user.name));
    return Column(
      children: [
        Text(userName),
        UserEmail(),
        const ExpensiveWidget(),
      ],
    );
  }
}

class UserEmail extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final email = ref.watch(userProvider.select((user) => user.email));
    return Text(email);
  }
}
```

**❌ BAD: Mutating state directly - won't rebuild**
```dart
class TodoController extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];

  void addTodo(Todo todo) => state.add(todo); // Won't trigger rebuild
}
```

**✅ GOOD: Create new state objects**
```dart
class TodoController extends Notifier<List<Todo>> {
  @override
  List<Todo> build() => [];

  void addTodo(Todo todo) => state = [...state, todo];

  void removeTodo(String id) => state = state.where((t) => t.id != id).toList();

  void updateTodo(String id, Todo updated) {
    state = [for (final t in state) if (t.id == id) updated else t];
  }
}
```

**❌ BAD: Circular dependency**
```dart
final userProvider = FutureProvider<User>((ref) async {
  final user = ref.watch(userProvider); // Error!
  return user;
});
```

**✅ GOOD: Clear dependency chain**
```dart
final userIdProvider = StateProvider<String?>((ref) => null);

final userProvider = FutureProvider<User?>((ref) async {
  final userId = ref.watch(userIdProvider);
  if (userId == null) return null;
  return await ref.read(userServiceProvider).getUser(userId);
});
```

**❌ BAD: Assumes data is always available**
```dart
class UserScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);
    final user = userAsync.value!; // Can throw!
    return Text(user.name);
  }
}
```

**✅ GOOD: Handle all AsyncValue states**
```dart
class UserScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);
    return userAsync.when(
      data: (user) => Text(user.name),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

#### Migration Guides

**StatefulWidget → Riverpod:**

```dart
// BEFORE
class CounterScreen extends StatefulWidget {
  @override
  State<CounterScreen> createState() => _CounterScreenState();
}

class _CounterScreenState extends State<CounterScreen> {
  int _count = 0;
  void _increment() => setState(() => _count++);

  @override
  Widget build(BuildContext context) {
    return Column(children: [
      Text('Count: $_count'),
      ElevatedButton(onPressed: _increment, child: Text('Increment')),
    ]);
  }
}

// AFTER
class CounterController extends Notifier<int> {
  @override
  int build() => 0;
  void increment() => state++;
  static final provider = NotifierProvider<CounterController, int>(CounterController.new);
}

class CounterScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(CounterController.provider);
    return Column(children: [
      Text('Count: $count'),
      ElevatedButton(
        onPressed: () => ref.read(CounterController.provider.notifier).increment(),
        child: Text('Increment'),
      ),
    ]);
  }
}
```

**Provider (old) → Riverpod:**

```dart
// BEFORE
class Counter with ChangeNotifier {
  int _count = 0;
  int get count => _count;
  void increment() { _count++; notifyListeners(); }
}

// main.dart: ChangeNotifierProvider(create: (_) => Counter(), child: MyApp())
// widget: final counter = Provider.of<Counter>(context); Text('${counter.count}')

// AFTER
class CounterController extends Notifier<int> {
  @override
  int build() => 0;
  void increment() => state++;
  static final provider = NotifierProvider<CounterController, int>(CounterController.new);
}

// main.dart: ProviderScope(child: MyApp())
// widget: final count = ref.watch(CounterController.provider); Text('$count')
```

**FutureBuilder → AsyncNotifier:**

```dart
// BEFORE
class UserScreen extends StatefulWidget {
  @override
  State<UserScreen> createState() => _UserScreenState();
}

class _UserScreenState extends State<UserScreen> {
  late Future<User> _userFuture;

  @override
  void initState() {
    super.initState();
    _userFuture = fetchUser();
  }

  Future<User> fetchUser() async {
    final response = await http.get(Uri.parse('/user'));
    return User.fromJson(jsonDecode(response.body));
  }

  @override
  Widget build(BuildContext context) {
    return FutureBuilder<User>(
      future: _userFuture,
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) return CircularProgressIndicator();
        if (snapshot.hasError) return Text('Error: ${snapshot.error}');
        return Text(snapshot.data!.name);
      },
    );
  }
}

// AFTER
class UserController extends AsyncNotifier<User?> {
  @override
  Future<User?> build() async {
    final result = await ref.read(userServiceProvider).getUser();
    return result.getOrNull();
  }

  Future<void> refresh() async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      final result = await ref.read(userServiceProvider).getUser();
      return result.getOrThrow();
    });
  }

  static final provider = AsyncNotifierProvider<UserController, User?>(UserController.new);
}

class UserScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userState = ref.watch(UserController.provider);
    return userState.when(
      data: (user) => user != null ? Text(user.name) : Text('No user'),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}
```

**StreamBuilder → StreamProvider:**

```dart
// BEFORE
class MessagesScreen extends StatelessWidget {
  final Stream<List<Message>> messagesStream;

  @override
  Widget build(BuildContext context) {
    return StreamBuilder<List<Message>>(
      stream: messagesStream,
      builder: (context, snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) return CircularProgressIndicator();
        if (snapshot.hasError) return Text('Error: ${snapshot.error}');
        final messages = snapshot.data ?? [];
        return ListView(children: messages.map((m) => Text(m.text)).toList());
      },
    );
  }
}

// AFTER
final messagesProvider = StreamProvider.autoDispose<List<Message>>((ref) {
  return ref.read(messageServiceProvider).watchMessages();
});

class MessagesScreen extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final messagesState = ref.watch(messagesProvider);
    return messagesState.when(
      data: (messages) => ListView(children: messages.map((m) => Text(m.text)).toList()),
      loading: () => CircularProgressIndicator(),
      error: (error, stack) => Text('Error: $error'),
    );
  }
}

#### See Also

- **`testing.md`** - Riverpod-specific testing patterns for controllers, services, and widgets
- **`flutter.md`** - Flutter/Dart specific rules for error handling and state management
- **`general.md`** - Core principles (DRY, intent clarity, small methods, minimal state)
```