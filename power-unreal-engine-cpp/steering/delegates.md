# 虚幻引擎委托

委托是一种泛型但类型安全的方式，可在 C++ 对象上调用成员函数。

## 委托类型

虚幻引擎支持三种类型的委托：
- **单播委托**：绑定单个函数
- **多播委托**：绑定多个函数
- **动态委托**：可序列化，支持蓝图

## 单播委托

### 声明

```cpp
// 无参数
DECLARE_DELEGATE(FSimpleDelegate);

// 带参数
DECLARE_DELEGATE_OneParam(FStringDelegate, FString);
DECLARE_DELEGATE_TwoParams(FTwoParamDelegate, FString, int32);

// 带返回值
DECLARE_DELEGATE_RetVal(bool, FBoolDelegate);
DECLARE_DELEGATE_RetVal_OneParam(bool, FCheckDelegate, FString);
```

### 绑定

```cpp
// 绑定到 UObject 成员函数
MyDelegate.BindUObject(this, &AMyActor::MyFunction);

// 绑定到原始 C++ 对象
MyDelegate.BindRaw(RawPointer, &FMyClass::MyFunction);

// 绑定到共享指针对象
MyDelegate.BindSP(SharedPtr, &FMyClass::MyFunction);

// 绑定到 Lambda
MyDelegate.BindLambda([](FString Str) {
    UE_LOG(LogTemp, Log, TEXT("%s"), *Str);
});

// 绑定到静态函数
MyDelegate.BindStatic(&MyStaticFunction);

// 解绑
MyDelegate.Unbind();
```

### 执行

```cpp
// 检查并执行
if (MyDelegate.IsBound())
{
    MyDelegate.Execute();
}

// 或使用 ExecuteIfBound
MyDelegate.ExecuteIfBound();

// 带返回值
bool Result = MyDelegate.Execute();
```

## 多播委托

多播委托可以绑定多个函数，全部执行。

### 声明

```cpp
DECLARE_MULTICAST_DELEGATE(FOnGameStart);
DECLARE_MULTICAST_DELEGATE_OneParam(FOnHealthChanged, float);
```

### 绑定

```cpp
// 添加绑定
FDelegateHandle Handle = OnHealthChanged.AddUObject(this, &AMyActor::HandleHealthChanged);

// 添加 Lambda
OnHealthChanged.AddLambda([](float NewHealth) {
    UE_LOG(LogTemp, Log, TEXT("Health: %f"), NewHealth);
});

// 移除绑定
OnHealthChanged.Remove(Handle);
OnHealthChanged.RemoveAll(this);
```

### 广播

```cpp
// 广播到所有绑定的函数
OnHealthChanged.Broadcast(100.0f);
```

## 动态委托

动态委托可以序列化，支持蓝图绑定。

### 声明

```cpp
// 单播动态委托
DECLARE_DYNAMIC_DELEGATE(FDynamicSimpleDelegate);
DECLARE_DYNAMIC_DELEGATE_OneParam(FDynamicStringDelegate, FString, Message);

// 多播动态委托（最常用于蓝图事件）
DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnGameStarted);
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnScoreChanged, int32, NewScore);
```

### 在类中使用

```cpp
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

public:
    // 蓝图可分配的事件
    UPROPERTY(BlueprintAssignable, Category = "Events")
    FOnScoreChanged OnScoreChanged;

    void AddScore(int32 Points)
    {
        Score += Points;
        OnScoreChanged.Broadcast(Score);
    }

private:
    int32 Score = 0;
};
```

### 绑定动态委托

```cpp
// 绑定到 UFUNCTION
UFUNCTION()
void HandleScoreChanged(int32 NewScore);

OnScoreChanged.AddDynamic(this, &AMyActor::HandleScoreChanged);

// 移除
OnScoreChanged.RemoveDynamic(this, &AMyActor::HandleScoreChanged);
```

## 载荷数据

绑定时可以传递额外参数：

```cpp
DECLARE_DELEGATE_OneParam(FMessageDelegate, FString);

// 绑定时传递载荷
MyDelegate.BindUObject(this, &AMyActor::HandleMessage, TEXT("Extra Data"));

// 函数签名
void HandleMessage(FString Message, FString ExtraData);
```

## 常见使用场景

### 事件系统

```cpp
// GameMode.h
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnPlayerDied, APlayerController*, Player);

UCLASS()
class AMyGameMode : public AGameModeBase
{
    GENERATED_BODY()

public:
    UPROPERTY(BlueprintAssignable)
    FOnPlayerDied OnPlayerDied;
};

// 触发事件
OnPlayerDied.Broadcast(DeadPlayer);
```

### 回调函数

```cpp
// 异步操作完成回调
DECLARE_DELEGATE_OneParam(FOnLoadComplete, bool);

void LoadAssetAsync(FOnLoadComplete OnComplete)
{
    // 异步加载...
    OnComplete.ExecuteIfBound(true);
}

// 使用
LoadAssetAsync(FOnLoadComplete::CreateUObject(this, &AMyActor::OnAssetLoaded));
```
