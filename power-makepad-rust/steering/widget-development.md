# Widget 开发指南

## Widget 核心 Trait

```rust
// 基础 trait
pub trait WidgetNode: LiveApply {
    fn uid_to_widget(&self, uid: WidgetUid) -> WidgetRef;
    fn find_widgets(&self, path: &[LiveId], cached: WidgetCache, results: &mut WidgetSet);
    fn walk(&mut self, cx: &mut Cx) -> Walk;
    fn area(&self) -> Area;
    fn redraw(&mut self, cx: &mut Cx);
}

// Widget trait
pub trait Widget: WidgetNode {
    fn handle_event(&mut self, cx: &mut Cx, event: &Event, scope: &mut Scope);
    fn draw_walk(&mut self, cx: &mut Cx2d, scope: &mut Scope, walk: Walk) -> DrawStep;
}
```

## 自定义 Widget 结构

```rust
#[derive(Live, LiveHook, Widget)]
pub struct MyWidget {
    #[live] draw_bg: DrawQuad,
    #[live] draw_text: DrawText,
    #[layout] layout: Layout,
    #[walk] walk: Walk,
    #[rust] value: f32,
}

impl Widget for MyWidget {
    fn handle_event(&mut self, cx: &mut Cx, event: &Event, scope: &mut Scope) {
        match event.hits(cx, self.draw_bg.area()) {
            Hit::FingerDown(_) => {
                cx.widget_action(uid, &scope.path, MyWidgetAction::Clicked);
            }
            _ => {}
        }
    }
    
    fn draw_walk(&mut self, cx: &mut Cx2d, scope: &mut Scope, walk: Walk) -> DrawStep {
        self.draw_bg.begin(cx, walk, self.layout);
        self.draw_text.draw_walk(cx, walk, Align::default(), "Hello");
        self.draw_bg.end(cx);
        DrawStep::done()
    }
}
```

## Action 定义

```rust
#[derive(Clone, Debug, DefaultNone)]
pub enum MyWidgetAction {
    None,
    Clicked,
    ValueChanged(f32),
}
```

## Live 属性标记

- `#[live]` - UI 相关字段，可在 Live DSL 中配置
- `#[rust]` - 非 UI 字段，仅 Rust 代码使用
- `#[layout]` - 布局配置
- `#[walk]` - 尺寸和位置配置
- `#[deref]` - 继承父组件功能
- `#[animator]` - 动画控制器

## 事件处理

```rust
fn handle_event(&mut self, cx: &mut Cx, event: &Event, scope: &mut Scope) {
    self.view.handle_event(cx, event, scope);
    
    match event.hits(cx, self.view.area()) {
        Hit::FingerDown(_) => {
            // 鼠标按下
        }
        Hit::FingerUp(_) => {
            // 鼠标抬起（点击完成）
            cx.action(MyAction::Clicked);
        }
        Hit::FingerMove(_) => {
            // 鼠标移动
        }
        Hit::FingerHoverIn(_) => {
            // 鼠标进入
        }
        Hit::FingerHoverOut(_) => {
            // 鼠标离开
        }
        _ => {}
    }
}
```

## Action 传递与处理

```rust
// 发送 action
cx.action(MyAction::Clicked(index));
cx.widget_action(uid, &scope.path, MyWidgetAction::Clicked);

// 接收 action (在 App 中)
impl MatchEvent for App {
    fn handle_actions(&mut self, cx: &mut Cx, actions: &Actions) {
        for action in actions {
            if let MyAction::Clicked(idx) = action.cast() {
                // 处理点击事件
            }
        }
    }
}
```

## 继承现有 Widget

```rust
#[derive(Live, LiveHook, Widget)]
pub struct MyButton {
    #[deref]
    button: Button,
    #[rust]
    custom_data: String,
}

impl Widget for MyButton {
    fn handle_event(&mut self, cx: &mut Cx, event: &Event, scope: &mut Scope) {
        self.button.handle_event(cx, event, scope);
        // 添加自定义逻辑
    }
    
    fn draw_walk(&mut self, cx: &mut Cx2d, scope: &mut Scope, walk: Walk) -> DrawStep {
        self.button.draw_walk(cx, scope, walk)
    }
}
```

## LiveHook 生命周期

```rust
impl LiveHook for MyWidget {
    fn after_new_from_doc(&mut self, cx: &mut Cx) {
        // 组件创建后调用
    }
    
    fn before_apply(&mut self, cx: &mut Cx, apply: &mut Apply, index: usize, nodes: &[LiveNode]) {
        // 应用属性前调用
    }
    
    fn after_apply(&mut self, cx: &mut Cx, apply: &mut Apply, index: usize, nodes: &[LiveNode]) {
        // 应用属性后调用
    }
}
```
