# Epic C++ 代码规范

Epic Games 的官方 C++ 代码规范。

## 命名规范

### 类型前缀

| 前缀 | 类型 | 示例 |
|------|------|------|
| `T` | 模板类 | `TArray`, `TMap` |
| `U` | UObject 派生类 | `UActorComponent` |
| `A` | AActor 派生类 | `ACharacter` |
| `S` | SWidget 派生类 | `SButton` |
| `I` | 接口类 | `IInputDevice` |
| `E` | 枚举 | `ECollisionChannel` |
| `F` | 其他结构体/类 | `FVector`, `FString` |
| `b` | 布尔变量 | `bIsEnabled` |

### 命名示例

```cpp
// 正确
float TeaWeight;
int32 TeaCount;
bool bDoesTeaStink;
FName TeaName;
FString TeaFriendlyName;
UClass* TeaClass;
USoundCue* TeaSound;
UTexture* TeaTexture;

// 错误
float lastMouseCoordinates;  // 应使用 PascalCase
float delta_coordinates;     // 不使用下划线
```

### 函数命名

```cpp
// 布尔函数应发起询问
bool IsVisible();
bool ShouldClearBuffer();
bool CheckTea(FTea Tea);      // 错误：True 的意义不明确
bool IsTeaFresh(FTea Tea);    // 正确：命名说明返回值含义

// 输出参数使用 Out 前缀
void GetResult(FResult& OutResult);
```

## 代码格式

### 大括号

```cpp
// 正确：大括号在新行
if (bThing)
{
    return;
}

// 错误：大括号在同一行
if (bThing) {
    return;
}

// 单语句也使用大括号
if (bHaveUnrealLicense)
{
    InsertYourGameHere();
}
else
{
    CallMarkRein();
}
```

### 缩进

- 使用制表符，不使用空格
- 制表符设为 4 字符

### Switch 语句

```cpp
switch (condition)
{
    case 1:
        // ...
        // 落入
    case 2:
        // ...
        break;
    case 3:
        // ...
        return;
    case 4:
    case 5:
        // ...
        break;
    default:
        break;
}
```

## 常量正确性

```cpp
// 常量参数
void SomeMutatingOperation(FThing& OutResult, const TArray<int32>& InArray);

// 常量方法
void FThing::SomeNonMutatingOperation() const;

// 常量迭代
for (const FString& Str : StringArray)
{
    // 不修改 StringArray
}
```

## 现代 C++ 特性

### nullptr

```cpp
// 正确
if (Pointer == nullptr)

// 错误
if (Pointer == NULL)
```

### auto 使用限制

```cpp
// 允许：Lambda
auto Lambda = [](int32 X) { return X * 2; };

// 允许：迭代器类型冗长时
for (auto It = Map.CreateIterator(); It; ++It)

// 不允许：普通变量
auto Value = GetValue();  // 错误：类型不明确
float Value = GetValue(); // 正确
```

### 基于范围的 for 循环

```cpp
// 新样式
for (TPair<FString, int32>& Kvp : MyMap)
{
    UE_LOG(LogCategory, Log, TEXT("Key: %s, Value: %d"), *Kvp.Key, Kvp.Value);
}

// 旧样式（避免）
for (auto It = MyMap.CreateIterator(); It; ++It)
{
    UE_LOG(LogCategory, Log, TEXT("Key: %s, Value: %d"), It.Key(), *It.Value());
}
```

### Lambda

```cpp
// 简短 Lambda
Thing* HelloThing = ArrayOfThings.FindByPredicate(
    [](const Thing& Th) { return Th.GetName().Contains(TEXT("Hello")); }
);

// 使用显式捕获，避免 [&] 和 [=]
int32 LocalValue = 10;
auto Lambda = [LocalValue]() { return LocalValue * 2; };
```

### 强类型枚举

```cpp
// 新样式
UENUM()
enum class EThing : uint8
{
    Thing1,
    Thing2
};

// 旧样式（避免）
UENUM()
namespace EThing
{
    enum Type
    {
        Thing1,
        Thing2
    };
}
```

## API 设计

### 避免布尔参数

```cpp
// 错误
FCup* MakeCupOfTea(FTea* Tea, bool bAddSugar, bool bAddMilk);
FCup* Cup = MakeCupOfTea(Tea, false, true);  // 含义不明

// 正确：使用枚举标志
enum class ETeaFlags
{
    None  = 0x00,
    Milk  = 0x01,
    Sugar = 0x02,
    Honey = 0x04
};
ENUM_CLASS_FLAGS(ETeaFlags)

FCup* MakeCupOfTea(FTea* Tea, ETeaFlags Flags = ETeaFlags::None);
FCup* Cup = MakeCupOfTea(Tea, ETeaFlags::Milk | ETeaFlags::Honey);
```

### 避免过长参数列表

```cpp
// 错误
TUniquePtr<FCup[]> MakeTeaForParty(
    const FTeaFlags* TeaPreferences,
    uint32 NumCupsToMake,
    FKettle* Kettle,
    ETeaType TeaType,
    float BrewingTimeInSeconds
);

// 正确：使用结构体
struct FTeaPartyParams
{
    const FTeaFlags* TeaPreferences = nullptr;
    uint32 NumCupsToMake = 0;
    FKettle* Kettle = nullptr;
    ETeaType TeaType = ETeaType::EnglishBreakfast;
    float BrewingTimeInSeconds = 120.0f;
};

TUniquePtr<FCup[]> MakeTeaForParty(const FTeaPartyParams& Params);
```

## 注释

```cpp
// 错误：无用注释
// 增加叶子
++Leaves;

// 正确：解释意图
// 我们知道还有另一种茶叶
++Leaves;

// 错误：注释低质量代码
// 大叶子和小叶子之和减去兼具两者的叶子数量
t = s + l - b;

// 正确：重写代码
TotalLeaves = SmallLeaves + LargeLeaves - SmallAndLargeLeaves;
```

## 包容性选词

避免使用：
- `Blacklist` → 使用 `DenyList`, `BlockList`
- `Whitelist` → 使用 `AllowList`, `TrustList`
- `Master` → 使用 `Primary`, `Main`, `Leader`
- `Slave` → 使用 `Secondary`, `Replica`, `Worker`
