# MeasureSpec、onMeasure、onLayout、onDraw 详解

Android 自定义 View 的绘制流程主要包括测量（onMeasure）、布局（onLayout）、绘制（onDraw）三个阶段，而 MeasureSpec 是测量阶段的核心参数。

---

## 1. MeasureSpec

MeasureSpec 是 View 测量过程中的一个重要参数，决定了 View 的测量模式和大小。

- **组成**：32 位 int，高 2 位是模式（mode），低 30 位是大小（size）。
- **三种模式**：
  - `EXACTLY`：精确值（如 match_parent、具体数值）
  - `AT_MOST`：最大值（如 wrap_content）
  - `UNSPECIFIED`：未指定，父容器不对 View 有任何限制

**常用方法：**
```java
int mode = MeasureSpec.getMode(measureSpec);
int size = MeasureSpec.getSize(measureSpec);
```

---

## 2. onMeasure

onMeasure 是 View 测量自身大小的方法。

- 作用：根据父容器传递的 MeasureSpec，计算并设置自身的宽高。
- 典型实现：
```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    // 或自定义测量逻辑
    setMeasuredDimension(width, height);
}
```
- 注意：自定义 ViewGroup 时需遍历子 View 并调用 measureChild/measureChildren。

---

## 3. onLayout

onLayout 是 ViewGroup 用于布局子 View 的方法。

- 作用：确定每个子 View 的位置（left, top, right, bottom）。
- 典型实现：
```java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    // 遍历子 View，调用 child.layout(left, top, right, bottom)
}
```
- 注意：只有 ViewGroup 需要重写 onLayout，普通 View 不需要。

---

## 4. onDraw

onDraw 是 View 的绘制方法。

- 作用：负责将 View 的内容绘制到屏幕上。
- 典型实现：
```java
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    // 绘制内容，如 canvas.drawXXX()
}
```
- 注意：只有设置了背景或需要自定义内容时才需重写 onDraw。

---

## 5. 三者关系与流程

1. **onMeasure**：确定 View 和子 View 的尺寸。
2. **onLayout**：确定 ViewGroup 内部每个子 View 的具体位置。
3. **onDraw**：将 View 的内容绘制到屏幕。

```
父容器 measure → 子 View onMeasure → ViewGroup onLayout → View onDraw
```

---

## 6. 常见问题

- 为什么自定义 ViewGroup 必须实现 onMeasure 和 onLayout？
- onDraw 什么时候会被调用？
- 如何理解 MeasureSpec 的三种模式？

---

如需更详细的源码分析或自定义 View 实践，欢迎继续提问！ 