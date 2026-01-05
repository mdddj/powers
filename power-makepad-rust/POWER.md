---
name: "makepad-rust"
displayName: "Makepad Rust UI Framework"
description: "Makepad - 用 Rust 编写的开源跨平台 UI 框架，支持 Windows、Linux、macOS、iOS、Android 和 Web。基于 GPU 着色器渲染，提供实时样式化和高性能 UI 开发体验。"
keywords: ["makepad", "rust", "ui", "gui", "cross-platform", "gpu", "shader", "live dsl", "sdf2d", "widget", "跨平台", "界面", "着色器", "矢量绘图"]
---

# Makepad Rust UI 框架

Makepad 是一个用 Rust 编写的开源跨平台 UI 框架，支持所有主流平台（Windows、Linux、macOS、iOS、Android、Web）。它基于 GPU 着色器渲染，提供超快编译速度和实时样式化功能。

## 核心特性

- **跨平台**: 一套代码运行于所有主流平台
- **GPU 加速**: 基于着色器的高性能 UI 渲染
- **实时样式化**: UI 变化即时反映，无需重新编译
- **混合即时模式**: 结合即时模式和保留模式的优势
- **Live DSL**: 声明式 UI 语法，简化样式定义

## 快速开始

### 创建新项目

```bash
cargo new my_makepad_app
```

### Cargo.toml 依赖

```toml
[dependencies]
makepad-widgets = { git = "https://github.com/makepad/makepad" }
```

### 基本应用结构

```rust
use makepad_widgets::*;

live_design! {
    use link::theme::*;
    use link::widgets::*;

    App = {{App}} {
        ui: <Root> {
            main_window = <Window> {
                show_bg: true,
                width: Fill,
                height: Fill,
                draw_bg: {
                    fn pixel(self) -> vec4 {
                        return #2a2a2a;
                    }
                }
                body = <View> {
                    flow: Down,
                    spacing: 10,
                    align: { x: 0.5, y: 0.5 }
                    
                    <Label> { text: "Hello Makepad!" }
                    <Button> { text: "Click Me" }
                }
            }
        }
    }
}

#[derive(Live, LiveHook)]
pub struct App {
    #[live] ui: WidgetRef,
    #[rust] counter: i32,
}

impl LiveRegister for App {
    fn live_register(cx: &mut Cx) {
        makepad_widgets::live_design(cx);
    }
}

impl MatchEvent for App {
    fn handle_actions(&mut self, cx: &mut Cx, actions: &Actions) {
        if self.ui.button(id!(button)).clicked(&actions) {
            self.counter += 1;
        }
    }
}

impl AppMain for App {
    fn handle_event(&mut self, cx: &mut Cx, event: &Event) {
        self.match_event(cx, event);
        self.ui.handle_event(cx, event, &mut Scope::empty());
    }
}

app_main!(App);
```

# When to Load Steering Files

- 学习 Live DSL 布局和样式语法 → `live-dsl.md`
- 使用 Sdf2d 进行矢量图形绘制 → `sdf2d-api.md`
- 开发自定义 Widget 组件 → `widget-development.md`
- 集成 Moly Kit AI 组件 → `moly-kit.md`

## 资源链接

- 官网: https://makepad.nl/
- GitHub: https://github.com/makepad/makepad
- 在线体验: https://makepad.dev/
- Makepad Book: https://book.makepad.rs/zh/
