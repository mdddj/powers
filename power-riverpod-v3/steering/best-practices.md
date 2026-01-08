# Riverpod 最佳实践

## DO ✅

### 在 build 中使用 ref.watch

```dart
Widget build(BuildContext context, WidgetRef ref) {
  // ✅ 正确 - 状态变化时自动重建
  final count = ref.watch(counterProvider);
  return Text('$count');
}
```

### 在回调中使用 ref.read

```dart
ElevatedButton(
  onPressed: () {
    // ✅ 正确 - 回调中使用 read
    ref.read(counterProvider.notifier).increment();
  },
  child: Text('Increment'),
)
```

### 使用 select 优化重建

```dart
// ✅ 只在 name 变化时重建
final name = ref.watch(userProvider.select((u) => u.name));

// ✅ 只在 items 数量变化时重建
final itemCount = ref.watch(cartProvider.select((c) => c.items.length));
```

### Provider 自己初始化

```dart
// ✅ 正确 - Provider 内部初始化
final userProvider = FutureProvider<User>((ref) async {
  return await api.fetchCurrentUser();
});
```

### 使用 AsyncValue 处理异步状态

```dart
// ✅ 正确 - 使用 switch 处理所有状态
return switch (asyncValue) {
  AsyncData(:final value) => DataWidget(value),
  AsyncError(:final error) => ErrorWidget(error),
  _ => LoadingWidget(),
};
```

### 在导航前初始化

```dart
// ✅ 正确 - 在导航回调中初始化
ElevatedButton(
  onPressed: () {
    ref.read(someProvider).init();
    Navigator.of(context).push(...);
  },
  child: Text('Navigate'),
)
```

## DON'T ❌

### 不要在 build 中使用 ref.read

```dart
Widget build(BuildContext context, WidgetRef ref) {
  // ❌ 错误 - 不会响应状态变化
  final count = ref.read(counterProvider);
  return Text('$count');
}
```

### 不要在 Widget 中初始化 Provider

```dart
class MyWidget extends ConsumerStatefulWidget {
  @override
  ConsumerState<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends ConsumerState<MyWidget> {
  @override
  void initState() {
    super.initState();
    // ❌ 错误 - 可能导致竞态条件
    ref.read(provider).init();
  }
}
```

### 不要用 Provider 存储临时 UI 状态

```dart
// ❌ 错误 - 临时状态不应该用 Provider
final textFieldProvider = StateProvider((ref) => '');
final isExpandedProvider = StateProvider((ref) => false);

// ✅ 正确 - 使用 StatefulWidget 或 flutter_hooks
class MyWidget extends StatefulWidget {
  @override
  State<MyWidget> createState() => _MyWidgetState();
}

class _MyWidgetState extends State<MyWidget> {
  final _controller = TextEditingController();
  bool _isExpanded = false;
  // ...
}
```

### 不要在 Provider 初始化时执行副作用

```dart
// ❌ 错误 - 初始化时执行副作用
final formProvider = Provider((ref) {
  submitForm(); // 不应该在这里
  return FormState();
});

// ✅ 正确 - 使用 Mutation 或在回调中执行
final submitForm = Mutation<void>();

onPressed: () {
  submitForm.run(ref, (tsx) async {
    await api.submit(formData);
  });
}
```

### 不要动态创建 Provider

```dart
// ❌ 错误 - 动态创建 Provider
Widget build(context, ref) {
  final provider = Provider((ref) => someValue); // 每次 build 都创建新的
  return Text(ref.watch(provider));
}

// ✅ 正确 - Provider 应该是顶级 final 变量
final myProvider = Provider((ref) => someValue);

Widget build(context, ref) {
  return Text(ref.watch(myProvider));
}
```

### 不要忽略 mounted 检查

```dart
// ❌ 错误 - async 后可能 Widget 已卸载
onPressed: () async {
  await someFuture;
  ref.read(...); // 可能抛出异常
}

// ✅ 正确 - 检查 mounted
onPressed: () async {
  await someFuture;
  if (!context.mounted) return;
  ref.read(...);
}
```

## 架构建议

### 分层结构

```
lib/
├── providers/           # Provider 定义
│   ├── auth_provider.dart
│   └── user_provider.dart
├── models/              # 数据模型
│   └── user.dart
├── services/            # 业务逻辑/API
│   └── api_service.dart
└── ui/                  # Widget
    └── screens/
```

### Provider 组织

```dart
// auth_provider.dart
final authServiceProvider = Provider((ref) => AuthService());

final authStateProvider = StreamProvider<User?>((ref) {
  final auth = ref.watch(authServiceProvider);
  return auth.authStateChanges();
});

final isLoggedInProvider = Provider((ref) {
  final authState = ref.watch(authStateProvider);
  return authState.valueOrNull != null;
});
```

### 依赖注入

```dart
// 抽象服务
abstract class UserRepository {
  Future<User> getUser(int id);
}

// 实现
class ApiUserRepository implements UserRepository {
  @override
  Future<User> getUser(int id) => api.fetchUser(id);
}

// Provider
final userRepositoryProvider = Provider<UserRepository>((ref) {
  return ApiUserRepository();
});

// 使用
final userProvider = FutureProvider.family<User, int>((ref, id) {
  final repo = ref.watch(userRepositoryProvider);
  return repo.getUser(id);
});

// 测试时覆盖
ProviderScope(
  overrides: [
    userRepositoryProvider.overrideWithValue(MockUserRepository()),
  ],
  child: MyApp(),
)
```

### 错误处理

```dart
// 全局错误处理
class ErrorObserver extends ProviderObserver {
  @override
  void providerDidFail(
    ProviderObserverContext context,
    Object error,
    StackTrace stackTrace,
  ) {
    // 记录错误
    logger.error('Provider failed', error, stackTrace);
  }
}

// 使用
ProviderScope(
  observers: [ErrorObserver()],
  child: MyApp(),
)
```

## 性能优化

### 使用 select 减少重建

```dart
// ❌ 任何 user 属性变化都重建
final user = ref.watch(userProvider);
return Text(user.name);

// ✅ 只在 name 变化时重建
final name = ref.watch(userProvider.select((u) => u.name));
return Text(name);
```

### 拆分大型 Provider

```dart
// ❌ 一个大 Provider
final appStateProvider = Provider((ref) => AppState(
  user: ...,
  settings: ...,
  cart: ...,
));

// ✅ 拆分为多个小 Provider
final userProvider = Provider((ref) => ...);
final settingsProvider = Provider((ref) => ...);
final cartProvider = Provider((ref) => ...);
```

### 合理使用 keepAlive

```dart
// 频繁访问的数据保持存活
@Riverpod(keepAlive: true)
Future<Config> appConfig(Ref ref) async {
  return await loadConfig();
}

// 临时数据使用自动销毁（默认）
@riverpod
Future<SearchResults> search(Ref ref, String query) async {
  return await api.search(query);
}
```

### 避免不必要的 Provider 链

```dart
// ❌ 过长的 Provider 链
final a = Provider((ref) => 1);
final b = Provider((ref) => ref.watch(a) + 1);
final c = Provider((ref) => ref.watch(b) + 1);
final d = Provider((ref) => ref.watch(c) + 1);

// ✅ 直接计算
final result = Provider((ref) {
  final base = ref.watch(baseProvider);
  return base + 3;
});
```

## 测试建议

### 单元测试 Provider

```dart
test('counter increments', () {
  final container = ProviderContainer.test();
  addTearDown(container.dispose);

  expect(container.read(counterProvider), 0);
  container.read(counterProvider.notifier).increment();
  expect(container.read(counterProvider), 1);
});
```

### Mock 依赖

```dart
test('user provider fetches user', () async {
  final container = ProviderContainer.test(
    overrides: [
      apiProvider.overrideWithValue(MockApi()),
    ],
  );

  final user = await container.read(userProvider.future);
  expect(user.name, 'Mock User');
});
```

### Widget 测试

```dart
testWidgets('shows user name', (tester) async {
  await tester.pumpWidget(
    ProviderScope(
      overrides: [
        userProvider.overrideWith(
          (ref) async => User(name: 'Test User'),
        ),
      ],
      child: MaterialApp(home: UserScreen()),
    ),
  );

  await tester.pumpAndSettle();
  expect(find.text('Test User'), findsOneWidget);
});
```
