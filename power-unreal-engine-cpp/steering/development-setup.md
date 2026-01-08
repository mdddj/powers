# 虚幻引擎开发环境设置

## Visual Studio 设置

### 推荐版本
- Visual Studio 2022（推荐）
- Visual Studio 2019

### 必需组件
- 使用 C++ 的桌面开发
- 使用 C++ 的游戏开发
- Windows 10/11 SDK

### 推荐扩展
- UnrealVS 扩展（随引擎安装）
- Visual Assist（可选，付费）

### 配置优化

```
工具 → 选项 → 文本编辑器 → C/C++ → 高级
- 禁用 IntelliSense（使用 UnrealVS 或 Visual Assist）
- 禁用数据库回退
```

## VS Code 设置

### 必需扩展
- C/C++ (Microsoft)
- C/C++ Extension Pack

### 推荐扩展
- Unreal Engine 4 Snippets
- C++ Intellisense

### 配置文件

`.vscode/c_cpp_properties.json`:
```json
{
    "configurations": [
        {
            "name": "UE5",
            "includePath": [
                "${workspaceFolder}/**",
                "C:/Program Files/Epic Games/UE_5.7/Engine/Source/**"
            ],
            "defines": [
                "UE_BUILD_DEVELOPMENT=1",
                "WITH_EDITOR=1"
            ],
            "compilerPath": "cl.exe",
            "cStandard": "c17",
            "cppStandard": "c++20",
            "intelliSenseMode": "windows-msvc-x64"
        }
    ]
}
```

## Rider 设置

JetBrains Rider 对虚幻引擎有原生支持。

### 优势
- 更快的索引和代码补全
- 内置蓝图支持
- 更好的重构工具

### 配置
1. 安装 Rider
2. 打开 .uproject 文件
3. Rider 自动检测 UE 项目

## Xcode 设置 (macOS)

### 生成项目文件
```bash
cd /path/to/project
/path/to/UE5/Engine/Build/BatchFiles/Mac/GenerateProjectFiles.sh -project="/path/to/MyProject.uproject" -game
```

### 推荐设置
- 使用 Metal 调试器
- 启用地址消毒器进行调试

## Live Coding

Live Coding 允许在编辑器运行时重新编译代码。

### 启用
编辑器 → 编辑 → 编辑器偏好设置 → Live Coding → 启用

### 快捷键
- `Ctrl + Alt + F11`：编译并重新加载

### 限制
- 不能修改 UPROPERTY 或 UFUNCTION 签名
- 不能添加/删除反射属性
- 构造函数更改需要重启编辑器

## 编译配置

### 配置类型
| 配置 | 用途 |
|------|------|
| Debug | 完整调试信息，最慢 |
| DebugGame | 引擎优化，游戏调试 |
| Development | 开发用，平衡性能和调试 |
| Shipping | 发布版本，最优化 |
| Test | 测试版本 |

### 编译命令

```bash
# 从命令行编译
Engine/Build/BatchFiles/Build.bat MyProject Win64 Development -Project="C:/Projects/MyProject/MyProject.uproject"
```

## 项目结构

```
MyProject/
├── Config/                 # 配置文件
│   ├── DefaultEngine.ini
│   ├── DefaultGame.ini
│   └── DefaultInput.ini
├── Content/               # 资产文件
├── Source/                # C++ 源代码
│   └── MyProject/
│       ├── MyProject.Build.cs
│       ├── MyProject.h
│       ├── MyProject.cpp
│       └── MyProjectGameMode.cpp
├── MyProject.uproject     # 项目文件
└── Binaries/              # 编译输出
```

## Build.cs 文件

```csharp
using UnrealBuildTool;

public class MyProject : ModuleRules
{
    public MyProject(ReadOnlyTargetRules Target) : base(Target)
    {
        PCHUsage = PCHUsageMode.UseExplicitOrSharedPCHs;
        
        PublicDependencyModuleNames.AddRange(new string[] {
            "Core",
            "CoreUObject",
            "Engine",
            "InputCore"
        });
        
        PrivateDependencyModuleNames.AddRange(new string[] {
            "Slate",
            "SlateCore"
        });
    }
}
```

## 常见问题

### 编译错误：找不到头文件
- 检查 Build.cs 中的模块依赖
- 确保包含路径正确

### IntelliSense 不工作
- 重新生成项目文件
- 刷新 Visual Studio 解决方案

### 热重载失败
- 检查是否修改了反射属性
- 尝试完全重新编译
