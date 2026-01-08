---
name: "riverpod-v3"
displayName: "Riverpod v3 状态管理"
description: "Flutter 响应式状态管理框架。包含 Provider、Consumer、自动销毁、Family、Mutations、离线持久化、代码生成等核心概念和最佳实践。"
keywords: ["riverpod", "flutter", "state", "provider", "notifier", "状态管理", "flutter状态", "响应式", "asyncnotifier", "futureProvider"]
---

# Riverpod v3 状态管理框架

Riverpod 是 Flutter/Dart 的响应式缓存和数据绑定框架，v3 版本带来了重大改进，包括简化的 API、实验性的 Mutations 和离线持久化支持。

## 核心特性

- **响应式状态管理**: 自动追踪依赖，状态变化时自动重建
- **类型安全**: 编译时检查，无运行时错误
- **可测试性**: 轻松 mock 和覆盖 providers
- **代码生成**: 可选的 riverpod_generator 简化代码
- **自动销毁**: 智能管理资源生命周期
- **自动重试**: 失败时自动重试
- **Mutations (实验性)**: 简化异步操作
- **离线持久化 (实验性)**: 本地缓存支持

## 何时使用此 Power

- 使用 Riverpod v3 进行 Flutter 状态管理
- 创建和使用 Provider、Notifier
- 处理异步数据 (FutureProvider, AsyncNotifier)
- 实现依赖注入和测试
- 从 v2 迁移到 v3
- 使用代码生成简化开发

## 快速开始

### 安装

```bash
# Flutter 项目
flutter pub add flutter_riverpod

# 使用代码生成 (推荐)
flutter pub add flutter_riverpod
flutter pub add riverpod_annotation
flutter pub add dev:riverpod_generator
flutter pub add dev:build_runner

# 运行代码生成
dart run build_runner watch -d
```

### pubspec.yaml

```yaml
dependencies:
  flutter_riverpod: ^3.1.0
  riverpod_annotation: ^4.0.0

dev_dependencies:
  build_runner:
  riverpod_generator: ^4.0.0+1
```

### Hello World 示例

```dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

// 创建一个 Provider
final helloWorldProvider = Provider((_) => 'Hello world');

void main() {
  runApp(
    // 用 ProviderScope 包裹应用
    ProviderScope(child: MyApp()),
  );
}

// 使用 ConsumerWidget 代替 StatelessWidget
class MyApp extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final String value = ref.watch(helloWorldProvider);
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('Example')),
        body: Center(child: Text(value)),
      ),
    );
  }
}
```

## Provider 类型

| 类型 | 用途 | 返回值 |
|------|------|--------|
| `Provider` | 同步只读值 | `T` |
| `FutureProvider` | 异步只读值 | `Future<T>` |
| `StreamProvider` | 流数据 | `Stream<T>` |
| `NotifierProvider` | 可变同步状态 | `T` |
| `AsyncNotifierProvider` | 可变异步状态 | `Future<T>` |
| `StreamNotifierProvider` | 可变流状态 | `Stream<T>` |

### 基础 Provider

```dart
// 简单值
final nameProvider = Provider((ref) => 'John');

// 代码生成
@riverpod
String name(Ref ref) => 'John';
```

### FutureProvider (异步数据)

```dart
final userProvider = FutureProvider<User>((ref) async {
  final response = await http.get(Uri.parse('api/user'));
  return User.fromJson(jsonDecode(response.body));
});

// 代码生成
@riverpod
Future<User> user(Ref ref) async {
  final response = await http.get(Uri.parse('api/user'));
  return User.fromJson(jsonDecode(response.body));
}
```

### Notifier (可变状态)

```dart
// 手动定义
class CounterNotifier extends Notifier<int> {
  @override
  int build() => 0;

  void increment() => state++;
  void decrement() => state--;
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
  void decrement() => state--;
}
```

### AsyncNotifier (异步可变状态)

```dart
@riverpod
class TodoList extends _$TodoList {
  @override
  Future<List<Todo>> build() async {
    return fetchTodos();
  }

  Future<void> addTodo(Todo todo) async {
    state = const AsyncLoading();
    final newTodos = await saveTodo(todo);
    state = AsyncData(newTodos);
  }
}
```

## 在 Widget 中使用

### ConsumerWidget

```dart
class MyWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final count = ref.watch(counterProvider);
    return Text('Count: $count');
  }
}
```

### Consumer Builder

```dart
Consumer(
  builder: (context, ref, child) {
    final count = ref.watch(counterProvider);
    return Text('Count: $count');
  },
)
```

### 处理异步状态

```dart
class UserWidget extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final userAsync = ref.watch(userProvider);

    return switch (userAsync) {
      AsyncData(:final value) => Text('Hello ${value.name}'),
      AsyncError(:final error) => Text('Error: $error'),
      _ => const CircularProgressIndicator(),
    };
  }
}
```

## Ref 方法

| 方法 | 用途 | 使用场景 |
|------|------|----------|
| `ref.watch` | 监听并重建 | build 方法中 |
| `ref.read` | 一次性读取 | 回调/事件中 |
| `ref.listen` | 监听副作用 | 显示 SnackBar 等 |
| `ref.invalidate` | 重置状态 | 刷新数据 |
| `ref.refresh` | 重置并读取 | 刷新并获取新值 |

```dart
// watch - 在 build 中监听
final value = ref.watch(myProvider);

// read - 在回调中读取
onPressed: () {
  ref.read(counterProvider.notifier).increment();
}

// listen - 监听副作用
ref.listen(myProvider, (previous, next) {
  ScaffoldMessenger.of(context).showSnackBar(
    SnackBar(content: Text('Value: $next')),
  );
});

// invalidate - 重置
ref.invalidate(myProvider);

// refresh - 重置并读取
final newValue = ref.refresh(myProvider);
```

## 自动销毁 (Auto Dispose)

```dart
// 代码生成默认启用自动销毁
@riverpod
String hello(Ref ref) => 'Hello';

// 禁用自动销毁
@Riverpod(keepAlive: true)
String hello(Ref ref) => 'Hello';

// 手动定义
final provider = Provider(
  isAutoDispose: true,
  (ref) => 'Hello',
);

// 条件保持存活
@riverpod
Future<String> example(Ref ref) async {
  final response = await http.get(Uri.parse('api/data'));
  ref.keepAlive(); // 成功后保持存活
  return response.body;
}
```

### 生命周期回调

```dart
@riverpod
Stream<int> example(Ref ref) {
  final controller = StreamController<int>();

  ref.onDispose(controller.close);      // 销毁时
  ref.onCancel(() => print('No listeners')); // 无监听者时
  ref.onResume(() => print('Listener added')); // 新监听者时

  return controller.stream;
}
```

## Family (参数化 Provider)

```dart
// 代码生成 - 直接添加参数
@riverpod
Future<User> user(Ref ref, int userId) async {
  return fetchUser(userId);
}

// 使用
final user = ref.watch(userProvider(123));

// 手动定义
final userProvider = FutureProvider.family<User, int>((ref, userId) async {
  return fetchUser(userId);
});
```

## 测试

### 单元测试

```dart
void main() {
  test('counter increments', () {
    final container = ProviderContainer.test();

    expect(container.read(counterProvider), 0);
    container.read(counterProvider.notifier).increment();
    expect(container.read(counterProvider), 1);
  });
}
```

### Mock Provider

```dart
final container = ProviderContainer.test(
  overrides: [
    userProvider.overrideWith((ref) async => User(name: 'Test')),
  ],
);
```

### Widget 测试

```dart
testWidgets('displays user name', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        userProvider.overrideWith((ref) async => User(name: 'John')),
      ],
      child: const MyApp(),
    ),
  );

  await tester.pumpAndSettle();
  expect(find.text('Hello John'), findsOneWidget);
});
```

## 最佳实践

### DO ✅

```dart
// 使用 ref.watch 在 build 中监听
final value = ref.watch(provider);

// 使用 ref.read 在回调中读取
onPressed: () => ref.read(provider.notifier).doSomething()

// 使用 select 优化重建
final name = ref.watch(userProvider.select((u) => u.name));
```

### DON'T ❌

```dart
// 不要在 build 中使用 ref.read
Widget build(context, ref) {
  final value = ref.read(provider); // ❌ 不会响应变化
}

// 不要在 initState 中使用 ref
void initState() {
  ref.read(provider); // ❌ 可能导致问题
}

// 不要用 Provider 存储临时 UI 状态
final textFieldProvider = StateProvider((ref) => ''); // ❌
```

## v3 新特性

### 自动重试

```dart
// 全局禁用
ProviderScope(
  retry: (retryCount, error) => null,
  child: MyApp(),
)

// 单个 Provider 禁用
@Riverpod(retry: myRetry)
class TodoList extends _$TodoList { ... }
```

### Mutations (实验性)

```dart
final addTodo = Mutation<Todo>();

// 在 Widget 中使用
final addTodoState = ref.watch(addTodo);

// 触发 mutation
addTodo.run(ref, (tsx) async {
  final todo = await createTodo();
  return todo;
});
```

### 离线持久化 (实验性)

```dart
class TodoList extends AsyncNotifier<List<Todo>> {
  @override
  Future<List<Todo>> build() async {
    persist(
      ref.watch(storageProvider.future),
      key: 'todo_list',
      encode: (todos) => todos.map((t) => t.toJson()).toList(),
      decode: (json) => (json as List).map(Todo.fromJson).toList(),
    );
    return fetchTodosFromServer();
  }
}
```

## Steering 文件

详细文档位于 `steering/` 目录：

- `concepts.md` - Provider、Consumer、Ref 等核心概念
- `advanced.md` - Mutations、离线持久化、自动重试
- `migration.md` - 版本迁移指南
- `best-practices.md` - DO/DON'T 最佳实践
- `codegen.md` - 代码生成使用指南

## 资源链接

- 官网: https://riverpod.dev/
- GitHub: https://github.com/rrousselGit/riverpod
- pub.dev: https://pub.dev/packages/flutter_riverpod
- riverpod_lint: https://pub.dev/packages/riverpod_lint
