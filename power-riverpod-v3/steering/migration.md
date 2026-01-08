# Riverpod 迁移指南

## 从 v2 迁移到 v3

### 主要变化

1. **自动重试** - Provider 失败时默认自动重试
2. **暂停/恢复** - 不可见的 Provider 会被暂停
3. **StateProvider 等移至新导入** - 不推荐使用
4. **所有 Provider 使用 `==` 过滤更新**
5. **Ref 简化** - 移除 Ref 子类
6. **AutoDispose 接口移除** - 统一为单一接口
7. **Family Notifier 移除** - 统一为 Notifier
8. **Provider 失败抛出 ProviderException**

### 自动重试

```dart
// 禁用全局自动重试
ProviderScope(
  retry: (retryCount, error) => null,
  child: MyApp(),
)

// 禁用单个 Provider
@Riverpod(retry: (_, __) => null)
Future<Data> myData(Ref ref) async => ...;
```

### StateProvider 等迁移

```dart
// 旧导入
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 新导入（如果仍要使用）
import 'package:flutter_riverpod/legacy.dart';
```

推荐迁移到 Notifier：

```dart
// 旧 StateProvider
final counterProvider = StateProvider((ref) => 0);

// 新 Notifier
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
  void set(int value) => state = value;
}
```

### ProviderObserver 变化

```dart
// 旧
class MyObserver extends ProviderObserver {
  @override
  void didUpdateProvider(
    ProviderBase provider,
    Object? previousValue,
    Object? newValue,
    ProviderContainer container,
  ) { ... }
}

// 新
class MyObserver extends ProviderObserver {
  @override
  void didUpdateProvider(ProviderObserverContext context) {
    final provider = context.provider;
    final container = context.container;
    // ...
  }
}
```

### Ref 简化

```dart
// 旧 - 不同的 Ref 类型
final myProvider = Provider<int>((ProviderRef<int> ref) => 0);
final autoProvider = Provider.autoDispose<int>((AutoDisposeProviderRef<int> ref) => 0);

// 新 - 统一使用 Ref
final myProvider = Provider<int>((Ref ref) => 0);

// 代码生成
@riverpod
int myProvider(Ref ref) => 0;  // 不再是 MyProviderRef
```

### AutoDispose 接口移除

```dart
// 旧
class MyNotifier extends AutoDisposeNotifier<int> { ... }
class MyNotifier extends AutoDisposeFamilyNotifier<int, String> { ... }

// 新 - 统一使用 Notifier
class MyNotifier extends Notifier<int> { ... }

// 代码生成自动处理
@riverpod  // 默认 autoDispose
class MyNotifier extends _$MyNotifier { ... }

@Riverpod(keepAlive: true)  // 禁用 autoDispose
class MyNotifier extends _$MyNotifier { ... }
```

### Family Notifier 移除

```dart
// 旧
class UserNotifier extends FamilyNotifier<User, int> {
  @override
  User build(int userId) => ...;
}

// 新 - 使用代码生成
@riverpod
class User extends _$User {
  @override
  User build(int userId) => ...;  // 参数直接在 build 中
}

// 或手动定义
class UserNotifier extends Notifier<User> {
  @override
  User build() {
    final userId = ref.watch(userIdProvider);  // 从其他 Provider 获取参数
    return ...;
  }
}
```

### ProviderException 处理

```dart
// 旧 - 直接捕获原始错误
try {
  ref.read(myProvider);
} on MyCustomError catch (e) {
  // 处理错误
}

// 新 - 错误包装在 ProviderException 中
try {
  ref.read(myProvider);
} on ProviderException catch (e) {
  final originalError = e.error;
  if (originalError is MyCustomError) {
    // 处理错误
  }
}

// AsyncValue 不受影响
final async = ref.watch(myProvider);
if (async.hasError) {
  final error = async.error;  // 原始错误
}
```

## 从 StateNotifier 迁移

### 基本迁移

```dart
// 旧 StateNotifier
class CounterNotifier extends StateNotifier<int> {
  CounterNotifier() : super(0);

  void increment() => state++;
}

final counterProvider = StateNotifierProvider<CounterNotifier, int>(
  (ref) => CounterNotifier(),
);

// 新 Notifier
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
}

final counterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);

// 代码生成
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}
```

### 异步 StateNotifier 迁移

```dart
// 旧
class TodosNotifier extends StateNotifier<AsyncValue<List<Todo>>> {
  TodosNotifier() : super(const AsyncLoading()) {
    _init();
  }

  Future<void> _init() async {
    state = await AsyncValue.guard(() async {
      return await api.fetchTodos();
    });
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      return await api.addTodo(title);
    });
  }
}

// 新 AsyncNotifier
class TodosNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    return await api.fetchTodos();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();
    state = await AsyncValue.guard(() async {
      return await api.addTodo(title);
    });
  }
}

final todosProvider = AsyncNotifierProvider<TodosNotifier, List<Todo>>(
  TodosNotifier.new,
);
```

### 主要区别

| StateNotifier | Notifier/AsyncNotifier |
|---------------|------------------------|
| 构造函数初始化 | `build()` 方法初始化 |
| `super(initialState)` | `return initialState` |
| 手动处理 `AsyncValue` | 自动处理异步状态 |
| `dispose()` 方法 | `ref.onDispose()` |
| `mounted` 属性 | `ref.mounted` (v3) |

### 生命周期迁移

```dart
// 旧 StateNotifier
class MyNotifier extends StateNotifier<int> {
  MyNotifier(this.ref) : super(0) {
    _subscription = ref.listen(otherProvider, (_, value) {
      // ...
    });
  }

  final Ref ref;
  StreamSubscription? _subscription;

  @override
  void dispose() {
    _subscription?.cancel();
    super.dispose();
  }
}

// 新 Notifier
class MyNotifier extends Notifier<int> {
  @override
  int build() {
    ref.listen(otherProvider, (_, value) {
      // 自动清理
    });

    final controller = StreamController<int>();
    ref.onDispose(controller.close);

    return 0;
  }
}
```

## 从 ChangeNotifier 迁移

```dart
// 旧 ChangeNotifier
class CartNotifier extends ChangeNotifier {
  final List<Item> _items = [];
  List<Item> get items => _items;

  void addItem(Item item) {
    _items.add(item);
    notifyListeners();
  }
}

final cartProvider = ChangeNotifierProvider((ref) => CartNotifier());

// 新 Notifier（使用不可变状态）
@riverpod
class Cart extends _$Cart {
  @override
  List<Item> build() => [];

  void addItem(Item item) {
    state = [...state, item];
  }

  void removeItem(Item item) {
    state = state.where((i) => i != item).toList();
  }
}
```

## 迁移检查清单

### v2 → v3

- [ ] 检查自动重试是否影响你的应用
- [ ] 更新 `StateProvider`/`StateNotifierProvider`/`ChangeNotifierProvider` 导入
- [ ] 更新 `ProviderObserver` 实现
- [ ] 将 `AutoDisposeRef` 等替换为 `Ref`
- [ ] 将 `AutoDisposeNotifier` 等替换为 `Notifier`
- [ ] 将 `FamilyNotifier` 迁移到代码生成或重构
- [ ] 更新错误处理以处理 `ProviderException`
- [ ] 测试所有 Provider 行为

### StateNotifier → Notifier

- [ ] 将 `extends StateNotifier<T>` 改为 `extends Notifier<T>`
- [ ] 将构造函数逻辑移到 `build()` 方法
- [ ] 将 `super(initialState)` 改为 `return initialState`
- [ ] 将 `dispose()` 改为 `ref.onDispose()`
- [ ] 更新 Provider 定义
- [ ] 测试所有功能

### ChangeNotifier → Notifier

- [ ] 将可变状态改为不可变状态
- [ ] 移除 `notifyListeners()` 调用
- [ ] 使用 `state = newState` 更新状态
- [ ] 更新 Provider 定义
- [ ] 测试所有功能
