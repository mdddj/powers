# Riverpod 高级特性

## 自动销毁 (Auto Dispose)

### 启用/禁用

```dart
// 代码生成 - 默认启用
@riverpod
String hello(Ref ref) => 'Hello';

// 禁用自动销毁
@Riverpod(keepAlive: true)
String hello(Ref ref) => 'Hello';

// 手动定义
final provider = Provider(
  isAutoDispose: true,  // 启用
  (ref) => 'Hello',
);
```

### 何时触发销毁？

当 Provider 没有监听者时：
1. `ref.onCancel` 被调用
2. 等待一帧
3. 如果仍无监听者，`ref.onDispose` 被调用，状态销毁

### 生命周期回调

```dart
@riverpod
Stream<int> ticker(Ref ref) {
  final controller = StreamController<int>();
  var count = 0;
  Timer? timer;

  void start() {
    timer = Timer.periodic(Duration(seconds: 1), (_) {
      controller.add(count++);
    });
  }

  // 销毁时清理
  ref.onDispose(() {
    timer?.cancel();
    controller.close();
  });

  // 最后一个监听者移除时
  ref.onCancel(() {
    timer?.cancel();
  });

  // 新监听者添加时
  ref.onResume(() {
    start();
  });

  start();
  return controller.stream;
}
```

### 条件保持存活

```dart
@riverpod
Future<Data> fetchData(Ref ref) async {
  final response = await api.getData();
  
  // 成功后保持存活
  if (response.isSuccess) {
    ref.keepAlive();
  }
  
  return response.data;
}

// 延时保持存活
extension CacheExtension on Ref {
  void cacheFor(Duration duration) {
    final link = keepAlive();
    final timer = Timer(duration, link.close);
    onDispose(timer.cancel);
  }
}

@riverpod
Future<Data> cachedData(Ref ref) async {
  ref.cacheFor(Duration(minutes: 5));
  return await api.getData();
}
```

## Family (参数化 Provider)

### 基本用法

```dart
// 代码生成
@riverpod
Future<User> user(Ref ref, int userId) async {
  return await api.fetchUser(userId);
}

// 使用
final user = ref.watch(userProvider(123));
final anotherUser = ref.watch(userProvider(456));

// 手动定义
final userProvider = FutureProvider.family<User, int>((ref, userId) async {
  return await api.fetchUser(userId);
});
```

### 多参数

```dart
// 代码生成支持多参数
@riverpod
Future<List<Post>> posts(
  Ref ref, {
  required int userId,
  int page = 1,
  int limit = 20,
}) async {
  return await api.fetchPosts(userId, page: page, limit: limit);
}

// 使用
final posts = ref.watch(postsProvider(userId: 123, page: 2));
```

### 参数要求

参数必须有一致的 `==` 实现：

```dart
// ✅ 好 - 基本类型
userProvider(123)

// ✅ 好 - 实现了 == 的类
@freezed
class UserQuery with _$UserQuery {
  factory UserQuery({required int id, String? name}) = _UserQuery;
}
userProvider(UserQuery(id: 123))

// ❌ 差 - 每次创建新对象
userProvider({'id': 123}) // Map 每次都是新实例
```

### 使 Family Provider 失效

```dart
// 使特定参数失效
ref.invalidate(userProvider(123));

// 使所有参数失效
ref.invalidate(userProvider);
```

## 自动重试 (v3 新特性)

Provider 失败时自动重试，默认最多 10 次，指数退避从 200ms 到 6.4s。

### 自定义重试逻辑

```dart
Duration? myRetry(int retryCount, Object error) {
  // 最多重试 5 次
  if (retryCount >= 5) return null;
  
  // 不重试 ProviderException
  if (error is ProviderException) return null;
  
  // 指数退避
  return Duration(milliseconds: 200 * (1 << retryCount));
}

// 全局配置
ProviderScope(
  retry: myRetry,
  child: MyApp(),
)

// 单个 Provider
@Riverpod(retry: myRetry)
Future<Data> myData(Ref ref) async => await api.getData();

// 禁用重试
ProviderScope(
  retry: (_, __) => null,
  child: MyApp(),
)
```

### 等待重试完成

```dart
// 等待 Provider 成功（跳过中间失败）
final data = await ref.watch(myProvider.future);
```

## Mutations (实验性)

Mutations 用于处理表单提交等副作用，自动管理加载/错误状态。

### 定义 Mutation

```dart
// 全局定义
final addTodo = Mutation<Todo>();

// 或作为 Notifier 的静态变量
class TodoNotifier extends Notifier<List<Todo>> {
  static final addTodo = Mutation<Todo>();
  
  @override
  List<Todo> build() => [];
}
```

### 使用 Mutation

```dart
class AddTodoButton extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final addTodoState = ref.watch(addTodo);

    return ElevatedButton(
      style: ButtonStyle(
        backgroundColor: switch (addTodoState) {
          MutationError() => WidgetStatePropertyAll(Colors.red),
          _ => null,
        },
      ),
      onPressed: () {
        addTodo.run(ref, (tsx) async {
          final todo = await api.createTodo(title: 'New Todo');
          return todo;
        });
      },
      child: Row(
        mainAxisSize: MainAxisSize.min,
        children: [
          Text('Add Todo'),
          if (addTodoState is MutationPending)
            Padding(
              padding: EdgeInsets.only(left: 8),
              child: SizedBox(
                width: 16,
                height: 16,
                child: CircularProgressIndicator(strokeWidth: 2),
              ),
            ),
        ],
      ),
    );
  }
}
```

### Mutation 状态

```dart
switch (mutationState) {
  MutationIdle() => // 空闲
  MutationPending() => // 进行中
  MutationSuccess(:final value) => // 成功，value 是结果
  MutationError(:final error) => // 失败
}
```

### 带参数的 Mutation

```dart
final removeTodo = Mutation<void>();

// 使用不同的 key
final removeTodo1 = removeTodo(todo1.id);
final removeTodo2 = removeTodo(todo2.id);

// 触发
removeTodo(todoId).run(ref, (tsx) async {
  await api.deleteTodo(todoId);
});
```

### 重置 Mutation

```dart
// 手动重置
addTodo.reset();

// 自动重置条件：
// - Provider 被销毁
// - 没有监听者
```

## 离线持久化 (实验性)

### 设置 Storage

```dart
// 使用 riverpod_sqflite
import 'package:riverpod_sqflite/riverpod_sqflite.dart';

final storageProvider = FutureProvider<Storage<String, String>>((ref) async {
  return JsonSqFliteStorage.open(
    join(await getDatabasesPath(), 'riverpod.db'),
  );
});
```

### 持久化 Notifier

```dart
class TodoListNotifier extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    // 设置持久化
    persist(
      ref.watch(storageProvider.future),
      key: 'todo_list',
      encode: (todos) => todos.map((t) => t.toJson()).toList(),
      decode: (json) => (json as List)
          .map((item) => Todo.fromJson(item as Map<String, dynamic>))
          .toList(),
    );

    // 仍然从服务器获取最新数据
    return await fetchTodosFromServer();
  }
}
```

### 代码生成简化

```dart
@riverpod
@JsonPersist(key: 'todo_list')
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async {
    return await fetchTodosFromServer();
  }
}
```

### 缓存时长

```dart
persist(
  storage,
  key: 'my_data',
  options: PersistOptions(
    // 默认 2 天
    cacheDuration: Duration(days: 7),
    // 或永久
    // cacheDuration: Duration.infinite,
  ),
  encode: ...,
  decode: ...,
);
```

### 数据迁移

使用 `destroyKey` 在数据结构变化时清除旧缓存：

```dart
persist(
  storage,
  key: 'todo_list',
  destroyKey: 'v2', // 改变此值会清除旧缓存
  encode: ...,
  decode: ...,
);
```

### 等待持久化解码

```dart
@override
Future<List<Todo>> build() async {
  // 等待持久化完成
  await persist(...);
  
  // 如果有缓存，使用缓存
  if (state.hasValue) {
    return state.value!;
  }
  
  // 否则从服务器获取
  return await fetchTodosFromServer();
}
```

### 测试持久化

```dart
testWidgets('test with persistence', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        // 使用内存存储
        storageProvider.overrideWith((_) async => Storage.inMemory()),
      ],
      child: MyApp(),
    ),
  );
});
```

## 取消和防抖

### 取消请求

```dart
@riverpod
Future<Activity> activity(Ref ref) async {
  final client = http.Client();
  
  // 销毁时取消请求
  ref.onDispose(client.close);
  
  final response = await client.get(
    Uri.https('api.example.com', '/activity'),
  );
  
  return Activity.fromJson(jsonDecode(response.body));
}
```

### 防抖

```dart
@riverpod
Future<List<SearchResult>> search(Ref ref, String query) async {
  // 防抖 500ms
  var cancelled = false;
  ref.onDispose(() => cancelled = true);
  
  await Future.delayed(Duration(milliseconds: 500));
  
  if (cancelled) {
    throw Exception('Cancelled');
  }
  
  return await api.search(query);
}
```

### 可复用的防抖扩展

```dart
extension DebounceExtension on Ref {
  Future<void> debounce(Duration duration) async {
    var cancelled = false;
    onDispose(() => cancelled = true);
    
    await Future.delayed(duration);
    
    if (cancelled) {
      throw Exception('Cancelled');
    }
  }
}

@riverpod
Future<List<SearchResult>> search(Ref ref, String query) async {
  await ref.debounce(Duration(milliseconds: 500));
  return await api.search(query);
}
```
