---
name: "unreal-engine-cpp"
displayName: "Unreal Engine C++ 编程"
description: "虚幻引擎 5.7 C++ 编程完整文档（中文版）。包含核心概念、容器、委托、反射系统、智能指针、代码规范和开发环境设置。"
keywords: ["unreal", "ue5", "c++", "游戏开发", "虚幻引擎", "game development", "uclass", "uproperty", "ufunction", "actor", "blueprint", "蓝图"]
---

# Unreal Engine C++ 编程

虚幻引擎 5.7 C++ 编程完整文档（中文版）。

## 何时使用此 Power

当你需要：
- 为虚幻引擎编写 C++ 代码
- 理解 UE 的容器类（TArray、TMap、TSet）
- 使用委托和事件系统
- 使用反射系统（UCLASS、UPROPERTY、UFUNCTION）
- 遵循 Epic 的代码规范
- 设置开发环境（VS、VS Code、Xcode、Rider）

## 快速参考

### 核心类型前缀

| 前缀 | 类型 |
|------|------|
| `U` | 继承自 UObject 的类 |
| `A` | 继承自 AActor 的类 |
| `S` | 继承自 SWidget 的类 |
| `F` | 其他结构体和类 |
| `T` | 模板类 |
| `I` | 接口类 |
| `E` | 枚举 |
| `b` | 布尔变量前缀 |

### 常用宏

```cpp
// 类声明
UCLASS()
class MYPROJECT_API AMyActor : public AActor
{
    GENERATED_BODY()
public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float MyProperty;
    
    UFUNCTION(BlueprintCallable)
    void MyFunction();
};

// 结构体声明
USTRUCT(BlueprintType)
struct FMyStruct
{
    GENERATED_BODY()
    
    UPROPERTY()
    int32 Value;
};
```

### 容器类

```cpp
// 数组
TArray<int32> Numbers;
Numbers.Add(10);
Numbers.Remove(10);

// 映射
TMap<FString, int32> NameToAge;
NameToAge.Add(TEXT("John"), 25);

// 集合
TSet<FString> UniqueNames;
UniqueNames.Add(TEXT("Alice"));
```

### 委托

```cpp
// 声明委托
DECLARE_DELEGATE(FSimpleDelegate);
DECLARE_DELEGATE_OneParam(FStringDelegate, FString);
DECLARE_DELEGATE_RetVal(bool, FBoolDelegate);

// 绑定和执行
MyDelegate.BindUObject(this, &AMyActor::MyFunction);
MyDelegate.ExecuteIfBound();
```

## 何时加载 Steering 文件

- 编写 C++ 代码基础 → `programming-basics.md`
- 使用容器类（TArray、TMap、TSet）→ `containers.md`
- 使用委托和事件 → `delegates.md`
- 使用反射系统 → `reflection.md`
- 设置开发环境 → `development-setup.md`
- 遵循代码规范 → `coding-standards.md`

## 可移植类型

| 类型 | 描述 |
|------|------|
| `bool` | 布尔值 |
| `TCHAR` | 字符 |
| `uint8` / `int8` | 8位整数 |
| `uint16` / `int16` | 16位整数 |
| `uint32` / `int32` | 32位整数 |
| `uint64` / `int64` | 64位整数 |
| `float` | 单精度浮点 |
| `double` | 双精度浮点 |

## 资源

- [官方文档](https://dev.epicgames.com/documentation/zh-cn/unreal-engine)
- [API 参考](https://docs.unrealengine.com/API)
- 版本：5.7
