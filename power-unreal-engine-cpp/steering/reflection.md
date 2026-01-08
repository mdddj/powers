# 虚幻引擎反射系统

虚幻引擎反射系统使用宏为类提供引擎和编辑器功能。

## 核心概念

- **UObject**：虚幻中对象的基类
- **UCLASS**：标记从 UObject 派生的类
- **UPROPERTY**：标记属性以支持反射
- **UFUNCTION**：标记函数以支持反射
- **USTRUCT**：标记结构体以支持反射

## UCLASS

### 基本声明

```cpp
UCLASS()
class MYPROJECT_API AMyActor : public AActor
{
    GENERATED_BODY()
    
public:
    AMyActor();
};
```

### 常用说明符

```cpp
// 可在蓝图中创建
UCLASS(Blueprintable)

// 蓝图可继承
UCLASS(BlueprintType)

// 抽象类，不能实例化
UCLASS(Abstract)

// 不在编辑器中显示
UCLASS(NotBlueprintable)

// 组合使用
UCLASS(Blueprintable, BlueprintType, ClassGroup = "MyGame")
```

## UPROPERTY

### 编辑器可见性

```cpp
// 任何地方可见（只读）
UPROPERTY(VisibleAnywhere)
float ReadOnlyValue;

// 任何地方可编辑
UPROPERTY(EditAnywhere)
float EditableValue;

// 仅在默认值中可编辑
UPROPERTY(EditDefaultsOnly)
float DefaultOnlyValue;

// 仅在实例中可编辑
UPROPERTY(EditInstanceOnly)
float InstanceOnlyValue;
```

### 蓝图访问

```cpp
// 蓝图只读
UPROPERTY(BlueprintReadOnly)
float ReadOnlyInBP;

// 蓝图可读写
UPROPERTY(BlueprintReadWrite)
float ReadWriteInBP;
```

### 分类和显示

```cpp
UPROPERTY(EditAnywhere, Category = "Stats|Health")
float MaxHealth;

UPROPERTY(EditAnywhere, meta = (DisplayName = "最大生命值"))
float MaxHP;

UPROPERTY(EditAnywhere, meta = (ClampMin = "0", ClampMax = "100"))
float Percentage;
```

### 复制

```cpp
// 网络复制
UPROPERTY(Replicated)
int32 ReplicatedValue;

UPROPERTY(ReplicatedUsing = OnRep_Health)
float Health;

UFUNCTION()
void OnRep_Health();
```

## UFUNCTION

### 蓝图集成

```cpp
// 蓝图可调用
UFUNCTION(BlueprintCallable, Category = "MyFunctions")
void DoSomething();

// 纯函数（无副作用）
UFUNCTION(BlueprintPure, Category = "MyFunctions")
float GetValue() const;

// 蓝图实现事件
UFUNCTION(BlueprintImplementableEvent, Category = "Events")
void OnGameStart();

// C++ 默认实现，蓝图可覆盖
UFUNCTION(BlueprintNativeEvent, Category = "Events")
void OnDamageReceived(float Damage);
void OnDamageReceived_Implementation(float Damage);
```

### 网络

```cpp
// 服务器执行
UFUNCTION(Server, Reliable)
void ServerDoAction();

// 客户端执行
UFUNCTION(Client, Reliable)
void ClientShowMessage(const FString& Message);

// 多播
UFUNCTION(NetMulticast, Unreliable)
void MulticastPlayEffect();
```

## USTRUCT

### 基本声明

```cpp
USTRUCT(BlueprintType)
struct FMyStruct
{
    GENERATED_BODY()
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    FString Name;
    
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    int32 Value;
    
    // 构造函数
    FMyStruct()
        : Name(TEXT(""))
        , Value(0)
    {}
};
```

### 在类中使用

```cpp
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()
    
public:
    UPROPERTY(EditAnywhere)
    FMyStruct MyData;
    
    UPROPERTY(EditAnywhere)
    TArray<FMyStruct> DataArray;
};
```

## UENUM

```cpp
UENUM(BlueprintType)
enum class EMyEnum : uint8
{
    None        UMETA(DisplayName = "无"),
    Option1     UMETA(DisplayName = "选项1"),
    Option2     UMETA(DisplayName = "选项2"),
    MAX         UMETA(Hidden)
};

// 使用
UPROPERTY(EditAnywhere)
EMyEnum MyEnumValue;
```

## 接口

```cpp
// 接口声明
UINTERFACE(MinimalAPI, Blueprintable)
class UMyInterface : public UInterface
{
    GENERATED_BODY()
};

class IMyInterface
{
    GENERATED_BODY()
    
public:
    // 纯虚函数
    virtual void DoAction() = 0;
    
    // 带默认实现
    virtual FString GetName() { return TEXT("Default"); }
    
    // 蓝图可实现
    UFUNCTION(BlueprintNativeEvent, BlueprintCallable)
    void BlueprintAction();
};

// 实现接口
UCLASS()
class AMyActor : public AActor, public IMyInterface
{
    GENERATED_BODY()
    
public:
    virtual void DoAction() override;
    virtual void BlueprintAction_Implementation() override;
};

// 检查接口
if (Actor->Implements<UMyInterface>())
{
    IMyInterface::Execute_BlueprintAction(Actor);
}
```

## TSubclassOf

提供类型安全的类引用：

```cpp
UPROPERTY(EditDefaultsOnly)
TSubclassOf<AWeapon> WeaponClass;

// 生成
if (WeaponClass)
{
    AWeapon* Weapon = GetWorld()->SpawnActor<AWeapon>(WeaponClass);
}
```
