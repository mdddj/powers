# Sdf2d 矢量绘图 API

Sdf2d 是 Makepad 的 2D 矢量绘图 API，基于 Signed Distance Field 技术。

## 基本用法

```rust
fn pixel(self) -> vec4 {
    let sdf = Sdf2d::viewport(self.pos * self.rect_size);
    
    // 绘制形状
    sdf.circle(50.0, 50.0, 30.0);
    sdf.fill(#f00);
    
    return sdf.result;
}
```

## 基本形状

```rust
// 圆形
sdf.circle(cx, cy, radius);

// 矩形
sdf.rect(x, y, w, h);

// 圆角矩形
sdf.box(x, y, w, h, radius);

// 不同圆角的矩形
sdf.box_x(x, y, w, h, r_left, r_right);
sdf.box_y(x, y, w, h, r_top, r_bottom);
sdf.box_all(x, y, w, h, r_lt, r_rt, r_rb, r_lb);

// 六边形
sdf.hexagon(cx, cy, radius);

// 水平线
sdf.hline(y, half_thickness);
```

## 路径绘制

```rust
fn pixel(self) -> vec4 {
    let sdf = Sdf2d::viewport(self.pos * self.rect_size);
    
    // 绘制星形
    sdf.move_to(30.0, 30.0);
    sdf.line_to(50.0, 70.0);
    sdf.line_to(70.0, 30.0);
    sdf.line_to(10.0, 50.0);
    sdf.line_to(90.0, 50.0);
    sdf.close_path();
    
    sdf.fill(#f00);
    return sdf.result;
}
```

## 布尔运算

```rust
fn pixel(self) -> vec4 {
    let sdf = Sdf2d::viewport(self.pos * self.rect_size);
    
    // 并集
    sdf.circle(40.0, 50.0, 30.0);
    sdf.union();
    sdf.circle(60.0, 50.0, 30.0);
    sdf.fill(#f00);
    
    // 交集
    sdf.circle(40.0, 50.0, 30.0);
    sdf.intersect();
    sdf.circle(60.0, 50.0, 30.0);
    sdf.fill(#0f0);
    
    // 差集
    sdf.circle(50.0, 50.0, 40.0);
    sdf.subtract();
    sdf.circle(50.0, 50.0, 20.0);
    sdf.fill(#00f);
    
    return sdf.result;
}
```

## 填充和描边

```rust
// 填充
sdf.fill(#f00);
sdf.fill_keep(#f00);        // 保留形状状态
sdf.fill_premul(vec4(...)); // 预乘 alpha

// 描边
sdf.stroke(#fff, 2.0);
sdf.stroke_keep(#fff, 2.0); // 保留形状状态

// 发光效果
sdf.glow(#0ff, 10.0);
```

## 变换

```rust
fn pixel(self) -> vec4 {
    let sdf = Sdf2d::viewport(self.pos * self.rect_size);
    
    sdf.translate(10.0, 10.0);           // 平移
    sdf.rotate(PI * 0.25, 50.0, 50.0);   // 旋转 (弧度, 中心点)
    sdf.scale(1.5, 50.0, 50.0);          // 缩放
    
    sdf.rect(20.0, 20.0, 60.0, 60.0);
    sdf.fill(#f00);
    
    return sdf.result;
}
```

## 常用内置函数

### 数学函数
- `abs(x)` - 绝对值
- `sin(x)`, `cos(x)`, `tan(x)` - 三角函数
- `pow(x, y)` - 幂运算
- `sqrt(x)` - 平方根
- `min(x, y)`, `max(x, y)` - 最小/最大值
- `clamp(x, min, max)` - 限制范围
- `mix(a, b, t)` - 线性插值
- `step(edge, x)` - 阶跃函数
- `smoothstep(e0, e1, x)` - 平滑阶跃
- `fract(x)` - 小数部分

### 向量函数
- `length(v)` - 向量长度
- `distance(a, b)` - 两点距离
- `normalize(v)` - 归一化
- `dot(a, b)` - 点积
- `cross(a, b)` - 叉积

### 角度转换
- `radians(degrees)` - 度转弧度
- `degrees(radians)` - 弧度转度
