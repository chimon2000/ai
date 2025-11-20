---
trigger: model_decision
description: "Rules applied to a Flutter project that uses Riverpod for state management."
---

#### State Management with Riverpod

##### Synchronous (Notifier):
```dart
class FeatureStore extends Notifier<FeatureState> {
  @override
  FeatureState build() => const FeatureState();

  void updateValue(String value) => state = state.copyWith(value: value);

  static final provider = AutoDisposeNotifierProvider<FeatureStore, FeatureState>(
    FeatureStore.new,
  );
}

// Usage
final state = ref.watch(FeatureStore.provider);
ref.read(FeatureStore.provider.notifier).updateValue('new');
```

##### Asynchronous (AsyncNotifier):
```dart
class FeatureStore extends AsyncNotifier<FeatureState> {
  @override
  Future<FeatureState> build() async => FeatureState(data: await _loadData());

  Future<void> updateValue(String value) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      return state.value!.copyWith(value: value);
    });
  }

  static final provider = AutoDisposeAsyncNotifierProvider<FeatureStore, FeatureState>(
    FeatureStore.new,
  );
}

// Usage
final stateAsync = ref.watch(FeatureStore.provider);
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

#### Store Pattern

- Separate business logic into store classes
- Use `AutoDisposeAsyncNotifier` for async stores
- Stores handle business logic, navigation, analytics
- Widgets handle UI rendering and user interactions only
- Use `Result<T>` for store methods (Exception as failure type)
- Use `ResultDart<T, E>` for custom failure types
- Use `Result<Unit>` for operations that don't return data

#### Provider Selection Decision Tree

**Is the data async (Future/Stream)?**
- Yes → Use `AsyncNotifier` or `FutureProvider`
- No → Use `Notifier` or `StateNotifier`

**Does the data need to be cached?**
- Yes → Use `FutureProvider` with `cacheFor()`
- No → Use `FutureProvider.autoDispose`

**Is this a one-time operation?**
- Yes → Use `FutureProvider.autoDispose`
- No → Use `FutureProvider` with caching

**Does this need to be watched by widgets?**
- Yes → Use `Notifier` or `AsyncNotifier`
- No → Use `Provider` for computed values

#### Common Patterns

**Combining Multiple Providers:**
```dart
final combinedProvider = ref.watch(provider1).when(
  data: (data1) => ref.watch(provider2).when(
    data: (data2) => CombinedData(data1, data2),
    loading: () => null,
    error: (e, st) => null,
  ),
  loading: () => null,
  error: (e, st) => null,
);
```

**Invalidating Providers:**
```dart
// Invalidate single provider
ref.refresh(myProvider);

// Invalidate all providers
ref.invalidate(myProvider);

// Invalidate family providers
ref.invalidate(myProvider(id));
```

**Family Providers:**
```dart
final userProvider = FutureProvider.family<User, String>((ref, userId) async {
  return await userService.getUser(userId);
});

// Usage
ref.watch(userProvider('123'));
```