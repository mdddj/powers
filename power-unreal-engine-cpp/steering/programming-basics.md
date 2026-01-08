# 虚幻引擎 C++ 编程基础

## 概述

虚幻引擎为 C++ 程序员提供了一个健壮的框架。主要功能包括：

- 在 C++ 中创建新的 Gameplay 类
- 使用虚幻反射系统封装类
- 使用 Gameplay 架构构建项目
- 创建委托进行类型安全的函数调用

## 创建 Actor

```cpp
// MyActor.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyActor.generated.h"

UCLASS()
class MYPROJECT_API AMyActor : public AActor
{
    GENERATED_BODY()

public:
    AMyActor();

protected:
    virtual void BeginPlay() override;

public:
    virtual void Tick(float DeltaTime) override;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "My Properties")
    float Health = 100.0f;

    UFUNCTION(BlueprintCallable, Category = "My Functions")
    void TakeDamage(float DamageAmount);
};
```

```cpp
// MyActor.cpp
#include "MyActor.h"

AMyActor::AMyActor()
{
    PrimaryActorTick.bCanEverTick = true;
}

void AMyActor::BeginPlay()
{
    Super::BeginPlay();
}

void AMyActor::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);
}

void AMyActor::TakeDamage(float DamageAmount)
{
    Health -= DamageAmount;
    if (Health <= 0)
    {
        Destroy();
    }
}
```

## 属性说明符

### 常用 UPROPERTY 说明符

```cpp
// 编辑器可见性
UPROPERTY(VisibleAnywhere)      // 只读
UPROPERTY(EditAnywhere)         // 可编辑
UPROPERTY(VisibleDefaultsOnly)  // 仅在类默认值中可见
UPROPERTY(EditDefaultsOnly)     // 仅在类默认值中可编辑
UPROPERTY(VisibleInstanceOnly)  // 仅在实例中可见
UPROPERTY(EditInstanceOnly)     // 仅在实例中可编辑

// 蓝图访问
UPROPERTY(BlueprintReadOnly)    // 蓝图只读
UPROPERTY(BlueprintReadWrite)   // 蓝图可读写

// 组合使用
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Stats")
float MaxHealth;
```

### 常用 UFUNCTION 说明符

```cpp
UFUNCTION(BlueprintCallable)           // 蓝图可调用
UFUNCTION(BlueprintPure)               // 纯函数，无副作用
UFUNCTION(BlueprintImplementableEvent) // 蓝图实现
UFUNCTION(BlueprintNativeEvent)        // C++ 默认实现，蓝图可覆盖
```

## 字符串处理

### FString

```cpp
FString MyString = TEXT("Hello World");
FString Combined = MyString + TEXT(" UE5");
int32 Length = MyString.Len();
bool bContains = MyString.Contains(TEXT("Hello"));
```

### FName

```cpp
FName MyName = FName(TEXT("MyAsset"));
// FName 用于资产引用，不区分大小写，比较快速
```

### FText

```cpp
FText MyText = FText::FromString(TEXT("显示文本"));
// FText 用于本地化文本
```

## 日志输出

```cpp
// 基本日志
UE_LOG(LogTemp, Log, TEXT("普通日志"));
UE_LOG(LogTemp, Warning, TEXT("警告日志"));
UE_LOG(LogTemp, Error, TEXT("错误日志"));

// 带参数
UE_LOG(LogTemp, Log, TEXT("Health: %f"), Health);
UE_LOG(LogTemp, Log, TEXT("Name: %s"), *PlayerName);
```

## 断言

```cpp
check(Pointer != nullptr);           // 发布版本中移除
checkf(Value > 0, TEXT("Value must be positive"));
verify(SomeFunction());              // 发布版本中保留表达式
ensure(Pointer != nullptr);          // 仅触发一次
ensureMsgf(Value > 0, TEXT("Invalid value"));
```

## 内存管理

### UObject 自动垃圾回收

```cpp
// UObject 由引擎自动管理
UPROPERTY()
UMyObject* MyObject;

// 创建 UObject
MyObject = NewObject<UMyObject>(this);
```

### 智能指针

```cpp
// 共享指针
TSharedPtr<FMyClass> SharedPtr = MakeShared<FMyClass>();

// 弱指针
TWeakPtr<FMyClass> WeakPtr = SharedPtr;

// 唯一指针
TUniquePtr<FMyClass> UniquePtr = MakeUnique<FMyClass>();
```
