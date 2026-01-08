# Riverpod 核心概念

## Provider 是什么？

Provider 本质上是"带缓存的函数"。它们会缓存函数的返回值，当使用相同参数调用时返回缓存值。

```dart
// 普通函数 - 每次调用都执行
User fetchUser(int id) => api.getUser(id);

// Provider - 缓存结果，多个 Widget 共享
final userProvider = FutureProvider.family<User, int>((ref, id) async {
  return api.getUser(id);
});
```

## Provider 类型选择

### 同步 vs 异步 vs 流

| 返回类型 | 不可变 | 可变 |
|----------|--------|------|
| `T` | `Provider` | `NotifierProvider` |
| `Future<T>` | `FutureProvider` | `AsyncNotifierProvider` |
| `Stream<T>` | `StreamProvider` | `StreamNotifierProvider` |

### 选择指南

1. **数据来源是什么？**
   - 同步计算 → `Provider` / `Notifier`
   - 网络请求 → `FutureProvider` / `AsyncNotifier`
   - 实时数据流 → `StreamProvider` / `StreamNotifier`

2. **需要从 UI 修改吗？**
   - 只读 → `Provider` / `FutureProvider` / `StreamProvider`
   - 可修改 → `Notifier` / `AsyncNotifier` / `StreamNotifier`

## 创建 Provider

### 不可变 Provider

```dart
// 同步值
final configProvider = Provider((ref) => AppConfig());

// 异步值
final userProvider = FutureProvider<User>((ref) async {
  return await api.fetchUser();
});

// 流
final messagesProvider = StreamProvider<List<Message>>((ref) {
  return firestore.collection('messages').snapshots();
});
```

### 可变 Notifier

```dart
// 同步 Notifier
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
  void reset() => state = 0;
}

final counterProvider = NotifierProvider<CounterNotifier, int>(
  CounterNotifier.new,
);

// 异步 Notifier
class TodoListNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    return await api.fetchTodos();
  }

  Future<void> addTodo(String title) async {
    state = const AsyncLoading();
    try {
      final newTodo = await api.createTodo(title);
      state = AsyncData([...state.value ?? [], newTodo]);
    } catch (e, st) {
      state = AsyncError(e, st);
    }
  }
}

final todoListProvider = AsyncNotifierProvider<TodoListNotifier, List<Todo>>(
  TodoListNotifier.new,
);
```

### 代码生成语法

```dart
// 不可变
@riverpod
String greeting(Ref ref) => 'Hello';

@riverpod
Future<User> user(Ref ref) async => await api.fetchUser();

// 可变 - 使用 class
@riverpod
class Counter extends _$Counter {
  @override
  int build() => 0;

  void increment() => state++;
}

@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async => await api.fetchTodos();

  Future<void> add(String title) async {
    // ...
  }
}
```

## Ref 详解

Ref 是与 Provider 交互的主要方式。

### 获取 Ref

```dart
// 在 Provider 中
final myProvider = Provider((ref) {
  // ref 作为参数
  final other = ref.watch(otherProvider);
  return MyValue();
});

// 在 Notifier 中
class MyNotifier extends Notifier<int> {
  @override
  int build() {
    // this.ref 可用
    ref.watch(otherProvider);
    return 0;
  }
}

// 在 Widget 中
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // ref 作为参数
    final value = ref.watch(myProvider);
    return Text('$value');
  }
}
```

### ref.watch - 监听变化

```dart
// 在 build 方法中使用
Widget build(BuildContext context, WidgetRef ref) {
  // 当 counterProvider 变化时，Widget 自动重建
  final count = ref.watch(counterProvider);
  return Text('$count');
}

// 在 Provider 中组合
final fullNameProvider = Provider((ref) {
  final firstName = ref.watch(firstNameProvider);
  final lastName = ref.watch(lastNameProvider);
  return '$firstName $lastName';
});
```

### ref.read - 一次性读取

```dart
// 在回调中使用，不触发重建
ElevatedButton(
  onPressed: () {
    // 读取当前值
    final count = ref.read(counterProvider);
    print('Current: $count');
    
    // 调用 notifier 方法
    ref.read(counterProvider.notifier).increment();
  },
  child: Text('Increment'),
)
```

### ref.listen - 监听副作用

```dart
Widget build(BuildContext context, WidgetRef ref) {
  // 监听变化执行副作用
  ref.listen(authProvider, (previous, next) {
    if (next == null) {
      // 用户登出，导航到登录页
      Navigator.of(context).pushReplacementNamed('/login');
    }
  });

  ref.listen(errorProvider, (_, error) {
    if (error != null) {
      ScaffoldMessenger.of(context).showSnackBar(
        SnackBar(content: Text(error.message)),
      );
    }
  });

  return Container();
}
```

### ref.invalidate / ref.refresh

```dart
// invalidate - 标记为过期，下次读取时重新计算
ref.invalidate(userProvider);

// refresh - invalidate + read，立即获取新值
final newUser = ref.refresh(userProvider);

// 常用于下拉刷新
RefreshIndicator(
  onRefresh: () => ref.refresh(dataProvider.future),
  child: ListView(...),
)
```

## Consumer Widget

### ConsumerWidget

```dart
class MyWidget extends ConsumerWidget {
  const MyWidget({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final value = ref.watch(myProvider);
    return Text('$value');
  }
}
```

### ConsumerStatefulWidget

```dart
class MyWidget extends ConsumerStatefulWidget {
  const MyWidget({super.key});

  @override
  ConsumerState<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends ConsumerState<MyWidget> {
  @override
  void initState() {
    super.initState();
    // 可以在这里使用 ref
  }

  @override
  Widget build(BuildContext context) {
    final value = ref.watch(myProvider);
    return Text('$value');
  }
}
```

### Consumer Builder

```dart
// 局部使用 Consumer
Scaffold(
  appBar: AppBar(title: Text('App')),
  body: Consumer(
    builder: (context, ref, child) {
      final data = ref.watch(dataProvider);
      return Text('$data');
    },
  ),
)
```

## AsyncValue 处理

```dart
Widget build(BuildContext context, WidgetRef ref) {
  final asyncUser = ref.watch(userProvider);

  // 方式 1: switch 表达式 (推荐)
  return switch (asyncUser) {
    AsyncData(:final value) => UserCard(user: value),
    AsyncError(:final error) => ErrorWidget(error: error),
    _ => const LoadingWidget(),
  };

  // 方式 2: when 方法
  return asyncUser.when(
    data: (user) => UserCard(user: user),
    error: (error, stack) => ErrorWidget(error: error),
    loading: () => const LoadingWidget(),
  );

  // 方式 3: 属性访问
  if (asyncUser.isLoading) return const LoadingWidget();
  if (asyncUser.hasError) return ErrorWidget(error: asyncUser.error);
  return UserCard(user: asyncUser.value!);
}
```

### AsyncValue 属性

```dart
final async = ref.watch(myProvider);

async.value        // T? - 当前值（可能为 null）
async.valueOrNull  // T? - 同上
async.requireValue // T - 当前值（无值时抛异常）

async.isLoading    // bool - 是否加载中
async.isRefreshing // bool - 是否刷新中（有旧数据）
async.isReloading  // bool - 是否重新加载中

async.hasValue     // bool - 是否有值
async.hasError     // bool - 是否有错误
async.error        // Object? - 错误对象
async.stackTrace   // StackTrace? - 错误堆栈
```

## Select 优化

使用 `select` 只监听状态的一部分，减少不必要的重建：

```dart
// 只在 name 变化时重建
final name = ref.watch(userProvider.select((user) => user.name));

// 只在 items 数量变化时重建
final count = ref.watch(cartProvider.select((cart) => cart.items.length));

// 多个 select
final isAdult = ref.watch(userProvider.select((u) => u.age >= 18));
final hasEmail = ref.watch(userProvider.select((u) => u.email != null));
```

## Provider 依赖

Provider 可以依赖其他 Provider：

```dart
final authProvider = Provider<AuthService>((ref) => AuthService());

final userProvider = FutureProvider<User>((ref) async {
  // 依赖 authProvider
  final auth = ref.watch(authProvider);
  final token = await auth.getToken();
  return api.fetchUser(token);
});

final profileProvider = Provider<Profile>((ref) {
  // 依赖 userProvider
  final user = ref.watch(userProvider);
  return user.when(
    data: (u) => Profile(name: u.name),
    error: (_, __) => Profile.empty(),
    loading: () => Profile.loading(),
  );
});
```

当依赖的 Provider 变化时，依赖它的 Provider 会自动重新计算。
