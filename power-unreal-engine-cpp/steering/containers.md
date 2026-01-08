# 虚幻引擎容器

## TArray - 数组

TArray 是虚幻引擎中最常用的容器，用于存储同一类型元素的序列。

### 基本操作

```cpp
TArray<int32> Numbers;

// 添加元素
Numbers.Add(10);
Numbers.Add(20);
Numbers.AddUnique(10);  // 仅在不存在时添加

// 插入元素
Numbers.Insert(15, 1);  // 在索引1处插入

// 访问元素
int32 First = Numbers[0];
int32 Last = Numbers.Last();
int32 Top = Numbers.Top();  // 同 Last()

// 查找
int32 Index = Numbers.Find(10);
bool bContains = Numbers.Contains(10);

// 移除
Numbers.Remove(10);           // 移除第一个匹配项
Numbers.RemoveAt(0);          // 按索引移除
Numbers.RemoveAll([](int32 N) { return N < 15; });

// 大小
int32 Count = Numbers.Num();
bool bEmpty = Numbers.IsEmpty();
Numbers.Empty();              // 清空
Numbers.Reserve(100);         // 预分配
```

### 遍历

```cpp
// 基于范围的 for 循环
for (int32 Number : Numbers)
{
    UE_LOG(LogTemp, Log, TEXT("%d"), Number);
}

// 带引用修改
for (int32& Number : Numbers)
{
    Number *= 2;
}

// 带索引
for (int32 i = 0; i < Numbers.Num(); ++i)
{
    UE_LOG(LogTemp, Log, TEXT("[%d] = %d"), i, Numbers[i]);
}
```

### 排序和过滤

```cpp
// 排序
Numbers.Sort();  // 升序
Numbers.Sort([](int32 A, int32 B) { return A > B; });  // 降序

// 过滤
TArray<int32> Filtered = Numbers.FilterByPredicate(
    [](int32 N) { return N > 10; }
);
```

## TMap - 映射

TMap 以键值对形式存储数据。

### 基本操作

```cpp
TMap<FString, int32> NameToAge;

// 添加
NameToAge.Add(TEXT("Alice"), 25);
NameToAge.Add(TEXT("Bob"), 30);
NameToAge.Emplace(TEXT("Charlie"), 35);

// 访问
int32* AgePtr = NameToAge.Find(TEXT("Alice"));
if (AgePtr)
{
    int32 Age = *AgePtr;
}

// 使用 FindOrAdd
int32& Age = NameToAge.FindOrAdd(TEXT("David"));
Age = 40;

// 检查存在
bool bExists = NameToAge.Contains(TEXT("Alice"));

// 移除
NameToAge.Remove(TEXT("Alice"));

// 大小
int32 Count = NameToAge.Num();
```

### 遍历

```cpp
// 遍历键值对
for (const TPair<FString, int32>& Pair : NameToAge)
{
    UE_LOG(LogTemp, Log, TEXT("%s: %d"), *Pair.Key, Pair.Value);
}

// 仅遍历键
TArray<FString> Keys;
NameToAge.GetKeys(Keys);

// 仅遍历值
TArray<int32> Values;
NameToAge.GenerateValueArray(Values);
```

## TSet - 集合

TSet 存储唯一元素，不保证顺序。

### 基本操作

```cpp
TSet<FString> UniqueNames;

// 添加
UniqueNames.Add(TEXT("Alice"));
UniqueNames.Add(TEXT("Bob"));
UniqueNames.Add(TEXT("Alice"));  // 不会重复添加

// 检查存在
bool bExists = UniqueNames.Contains(TEXT("Alice"));

// 移除
UniqueNames.Remove(TEXT("Alice"));

// 集合操作
TSet<FString> OtherSet;
OtherSet.Add(TEXT("Bob"));
OtherSet.Add(TEXT("Charlie"));

TSet<FString> Union = UniqueNames.Union(OtherSet);
TSet<FString> Intersect = UniqueNames.Intersect(OtherSet);
TSet<FString> Difference = UniqueNames.Difference(OtherSet);
```

### 遍历

```cpp
for (const FString& Name : UniqueNames)
{
    UE_LOG(LogTemp, Log, TEXT("%s"), *Name);
}
```

## TSubclassOf - 类型安全的类引用

```cpp
// 声明
UPROPERTY(EditDefaultsOnly)
TSubclassOf<AActor> ActorClass;

// 使用
if (ActorClass)
{
    AActor* NewActor = GetWorld()->SpawnActor<AActor>(ActorClass);
}
```

## 软引用和硬引用

```cpp
// 硬引用 - 始终加载
UPROPERTY()
UTexture2D* HardRef;

// 软引用 - 按需加载
UPROPERTY()
TSoftObjectPtr<UTexture2D> SoftRef;

// 加载软引用
if (!SoftRef.IsNull())
{
    UTexture2D* Texture = SoftRef.LoadSynchronous();
}
```
