# Moly Kit 集成指南

Moly Kit 是用于 AI 应用开发的 Makepad 组件库。

## 安装

```toml
[dependencies]
moly-kit = { git = "https://github.com/moly-ai/moly-ai.git", features = ["full"], branch = "main" }
```

## 注册组件

```rust
impl LiveRegister for App {
    fn live_register(cx: &mut Cx) {
        makepad_widgets::live_design(cx);
        moly_kit::widgets::live_design(cx);
    }
}
```

## 使用 Chat 组件

```rust
live_design! {
    use moly_kit::widgets::chat::Chat;
    
    MyApp = {{MyApp}} {
        chat = <Chat> {}
    }
}
```

## 核心组件

### Chat
主聊天界面组件，需要绑定 ChatController。

```rust
let controller = Arc::new(Mutex::new(ChatController::new(/* ... */)));
let mut chat = view.chat(ids!(chat));
chat.write().set_chat_controller(cx, Some(controller.clone()));
```

### Messages
消息列表组件，处理消息显示和交互。

```rust
if let Some(msg_action) = action.as_widget_action().and_then(|wa| wa.cast::<MessagesAction>()) {
    match msg_action {
        MessagesAction::Copy(idx) => { /* 复制到剪贴板 */ }
        MessagesAction::Delete(idx) => { /* 删除消息 */ }
        _ => {}
    }
}
```

### PromptInput
输入框组件，处理用户输入。

```rust
let mut input = view.prompt_input(ids!(prompt));
if input.read().submitted(event.actions()) {
    let text = input.text();
    // 发送消息
    input.write().reset(cx);
}
```

### MessageMarkdown
Markdown 渲染组件。

```rust
live_design! {
    <MessageMarkdown> { text: "Hello **Markdown**" }
}
```

### Avatar
头像组件。

```rust
let mut avatar = widget.avatar(ids!(avatar)).borrow_mut().unwrap();
avatar.avatar = Some(EntityAvatar::Text("B".into()));
// 或使用图片
avatar.avatar = Some(EntityAvatar::Image("res/avatar.png".into()));
```

### AttachmentList
附件列表组件。

```rust
let mut list = view.attachment_list(ids!(attachments));
list.write().attachments.push(Attachment::from_text("note.txt", "content").unwrap());
```

### MolyModal
模态对话框组件。

```rust
let modal = view.moly_modal(ids!(settings_modal));
modal.open_as_dialog(cx);
if modal.dismissed(event.actions()) {
    // 关闭后逻辑
}
```

## 实时语音 (Realtime)

需要启用 `realtime` 特性（非 wasm）。

```rust
match upgrade {
    Upgrade::Realtime(channel) => {
        let mut realtime = chat.realtime(ids!(realtime));
        realtime.set_bot_entity_id(cx, EntityId::Bot(bot_id.clone()));
        realtime.set_realtime_channel(channel.clone());
        chat.moly_modal(ids!(audio_modal)).open_as_dialog(cx);
    }
    _ => {}
}
```

## 版本兼容性

建议使用相同版本的 makepad-widgets 和 moly-kit：

```toml
makepad-widgets = { git = "https://github.com/wyeworks/makepad", rev = "b8b65f4fa" }
moly-kit = { git = "https://github.com/moxin-org/moly.git", features = ["full"], branch = "main" }
```
