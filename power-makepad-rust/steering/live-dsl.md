# Live DSL 语法指南

## 布局属性

```rust
live_design! {
    MyView = <View> {
        // 尺寸
        width: Fill,        // 填充父容器
        height: Fit,        // 适应内容
        width: 200,         // 固定像素
        
        // 布局方向
        flow: Down,         // 垂直排列
        flow: Right,        // 水平排列
        flow: Overlay,      // 叠加
        
        // 间距和内边距
        spacing: 10,
        padding: { left: 10, top: 5, right: 10, bottom: 5 }
        
        // 对齐 (0.0-1.0 归一化坐标)
        align: { x: 0.5, y: 0.5 }  // 居中
        align: { x: 1.0, y: 1.0 }  // 右下角
    }
}
```

## 背景绘制

```rust
live_design! {
    MyView = <View> {
        show_bg: true,
        draw_bg: {
            color: #ff0000,  // 纯色背景
            
            // 或自定义着色器
            fn pixel(self) -> vec4 {
                // 渐变效果
                return mix(#f00, #00f, self.pos.x);
            }
        }
    }
}
```

## 动画系统

```rust
live_design! {
    MyButton = <Button> {
        animator: {
            hover = {
                default: off,
                off = {
                    from: { all: Forward { duration: 0.2 } }
                    apply: { draw_bg: { color: #333 } }
                }
                on = {
                    from: { all: Forward { duration: 0.2 } }
                    apply: { draw_bg: { color: #555 } }
                }
            }
            pressed = {
                default: off,
                off = {
                    from: { all: Forward { duration: 0.1 } }
                    apply: { draw_bg: { color: #333 } }
                }
                on = {
                    from: { all: Snap } }
                    apply: { draw_bg: { color: #222 } }
                }
            }
        }
    }
}
```

## 动画 from 类型

```rust
from: {
    all: Forward { duration: 0.2 }    // 正向播放一次
    all: Reverse { duration: 0.2 }    // 反向播放一次
    all: Loop { duration: 0.2 }       // 循环播放
    all: BounceLoop { duration: 0.2 } // 来回循环播放
    all: Snap                         // 瞬间切换，无动画
}
```

## 缓动函数

```rust
ease: Linear          // 线性
ease: InQuad          // 二次方加速
ease: OutQuad         // 二次方减速
ease: InOutQuad       // 二次方加速减速
ease: Bezier {        // 贝塞尔曲线
    cp0: 0.0, cp1: 0.0, cp2: 1.0, cp3: 1.0
}
```

## 属性继承

```rust
// 基础组件
Card = {{MyView}} {
    background_color: #fff,
    corner_radius: 4.0,
    flow: Down,
    spacing: 10
}

// 继承并扩展
ProductCard = <Card> {
    background_color: #f5f5f5,  // 覆盖属性
    corner_radius: 8.0,
    width: 200,                  // 新增属性
    
    // 添加子组件
    <Label> { text: "Product" }
}
```

## 变量定义

```rust
live_design! {
    // 定义资源变量
    ICON = dep("crate://self/resources/icon.svg");
    PRIMARY_COLOR = #007bff;
    
    MyButton = <Button> {
        draw_icon: { svg_file: (ICON) }
        draw_bg: { color: (PRIMARY_COLOR) }
    }
}
```
