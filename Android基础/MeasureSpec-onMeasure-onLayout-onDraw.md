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

### onMeasure 详解

#### 1. 方法签名与参数
```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec)
```
- `widthMeasureSpec`、`heightMeasureSpec`：由父容器传递下来的测量约束，包含测量模式和大小。

#### 2. 调用流程
- 测量流程由父 View（通常是 ViewGroup）发起，依次调用每个子 View 的 measure() 方法。
- measure() 内部最终会调用 onMeasure()。
- onMeasure() 负责根据 MeasureSpec 计算自身的宽高，并通过 setMeasuredDimension(width, height) 设置。

#### 3. MeasureSpec 的作用
- MeasureSpec 是一个 32 位 int，高 2 位表示模式（EXACTLY、AT_MOST、UNSPECIFIED），低 30 位表示大小。
- 通过 MeasureSpec.getMode() 和 MeasureSpec.getSize() 获取模式和大小。

#### 4. 常见实现方式
- 普通自定义 View：
```java
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightMode = MeasureSpec.getMode(heightMeasureSpec);
    int heightSize = MeasureSpec.getSize(heightMeasureSpec);

    int width = ... // 根据需求计算
    int height = ... // 根据需求计算

    setMeasuredDimension(width, height);
}
```
- 自定义 ViewGroup：
  需要遍历所有子 View，调用 measureChild/measureChildren，并根据子 View 的测量结果决定自身大小。

#### 5. 注意事项
- 必须调用 setMeasuredDimension()，否则会抛出异常。
- 不要直接使用传入的 size 作为最终宽高，要结合 mode 进行判断和适配。
- 对于 wrap_content，要根据内容自适应大小。
- 对于 match_parent 或精确值，要遵循父容器的约束。

#### 6. 源码片段
```java
// View.java
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
    if ((mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ||
        widthMeasureSpec != mOldWidthMeasureSpec ||
        heightMeasureSpec != mOldHeightMeasureSpec) {
        onMeasure(widthMeasureSpec, heightMeasureSpec);
        ...
    }
}
```

#### 7. 自定义 ViewGroup 的特殊点
- 需要遍历所有子 View，分别为每个子 View 计算合适的 MeasureSpec 并调用 measure()。
- 最终根据所有子 View 的测量结果，决定自身的宽高。

#### 8. widthMeasureSpec 和 heightMeasureSpec 的本质与理解

- 这两个参数本质上是由父容器根据自身的测量逻辑和布局参数（如 layout_width、layout_height）生成的 32 位 int 值，分别用于约束子 View 的宽和高。
- 每个 MeasureSpec 由两部分组成：
  - **模式（mode）**：高 2 位，表示测量约束类型。
    - `EXACTLY`：精确模式，父容器希望子 View 的大小就是指定的值（如 match_parent 或具体数值）。
    - `AT_MOST`：最大模式，子 View 的大小不能超过指定的最大值（如 wrap_content）。
    - `UNSPECIFIED`：未指定模式，父容器对子 View 没有限制（如 ScrollView 的子 View）。
  - **大小（size）**：低 30 位，表示具体的像素值。

- MeasureSpec 的生成方式：
  - 父容器在 measure 子 View 时，会根据自身的剩余空间、子 View 的 layout_xxx 属性和自身的测量模式，通过 `ViewGroup.getChildMeasureSpec()` 生成对应的 MeasureSpec。

- 如何理解和使用：
  - 在 onMeasure 中，不能直接把 MeasureSpec 的 size 作为最终宽高，需要结合 mode 进行判断：
    - 如果 mode 是 EXACTLY，必须等于 size。
    - 如果 mode 是 AT_MOST，不能超过 size，通常取内容和 size 的较小值。
    - 如果 mode 是 UNSPECIFIED，可以任意大小，通常用于系统内部场景。
  - 通过 `MeasureSpec.getMode(widthMeasureSpec)` 和 `MeasureSpec.getSize(widthMeasureSpec)` 获取具体的模式和大小。

- 示例：
```java
int widthMode = MeasureSpec.getMode(widthMeasureSpec);
int widthSize = MeasureSpec.getSize(widthMeasureSpec);
```

- 总结：
  widthMeasureSpec 和 heightMeasureSpec 是父容器对子 View 的测量约束，决定了子 View 在测量阶段能有多大空间和如何适配父容器的要求。理解这两个参数对于自定义 View 和 ViewGroup 的测量逻辑至关重要。

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

#### 9. 示例：View 放在 FrameLayout 中 onMeasure 参数分析

假设有如下布局：

```xml
<FrameLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <View
        android:layout_width="wrap_content"
        android:layout_height="100dp" />
</FrameLayout>
```

- FrameLayout 的父容器（如 Activity 的根布局）会先给 FrameLayout 一个 MeasureSpec（假设为 EXACTLY 1080px）。
- FrameLayout 在测量子 View 时，会根据子 View 的 layout_width/layout_height 和自身的 MeasureSpec，调用 getChildMeasureSpec 生成子 View 的 MeasureSpec。

对于上面 View 的 onMeasure：
- widthMeasureSpec：mode = AT_MOST，size = 1080（父容器剩余空间，wrap_content）
- heightMeasureSpec：mode = EXACTLY，size = 100dp（layout_height 指定了具体值）

即 View 的 onMeasure 可能收到如下参数：
- widthMeasureSpec = MeasureSpec(AT_MOST, 1080)
- heightMeasureSpec = MeasureSpec(EXACTLY, 100dp)

**解释：**
- 父容器 FrameLayout 的宽度是 match_parent，子 View 的宽度是 wrap_content，所以父容器会给子 View 一个最大宽度限制（AT_MOST，size 为父容器宽度）。
- 子 View 的高度是 100dp，父容器会给它一个精确高度（EXACTLY，size 为 100dp）。

通过这个例子可以理解，onMeasure 的参数是由父容器根据自身和子 View 的布局参数共同决定的。 

#### 10. View 的 Margin 需要自己计算吗？

- **普通 View（非 ViewGroup）**：
  - 在 onMeasure/onLayout 中，margin 的处理通常由父容器（ViewGroup）负责。
  - 普通 View 在 onMeasure/onLayout 时，传入的 MeasureSpec 已经考虑了 margin 的影响，自己无需再手动处理 margin。

- **自定义 ViewGroup**：
  - 需要在测量和布局子 View 时，手动处理每个子 View 的 margin。
  - 例如在 onMeasure 中，测量子 View 时要加上 margin；在 onLayout 中，布局子 View 的位置时也要加上 margin。
  - 示例：
    ```java
    MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
    int childWidthSpec = getChildMeasureSpec(parentWidthSpec, getPaddingLeft() + getPaddingRight() + lp.leftMargin + lp.rightMargin, lp.width);
    int childHeightSpec = getChildMeasureSpec(parentHeightSpec, getPaddingTop() + getPaddingBottom() + lp.topMargin + lp.bottomMargin, lp.height);
    child.measure(childWidthSpec, childHeightSpec);
    ```

- **总结**：
  - 普通 View 不需要关心 margin，ViewGroup 需要在测量和布局子 View 时手动处理 margin。 

#### 11. 示例：扑克牌叠放效果的 ViewGroup onLayout 实现

假设我们要实现一个自定义 ViewGroup，将多张扑克牌水平部分重叠地排列（如手牌效果），可以这样实现 onLayout：

```java
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    int childCount = getChildCount();
    int left = getPaddingLeft();
    int overlap = 40; // 每张牌重叠的像素宽度
    for (int i = 0; i < childCount; i++) {
        View child = getChildAt(i);
        if (child.getVisibility() != GONE) {
            int childWidth = child.getMeasuredWidth();
            int childHeight = child.getMeasuredHeight();
            int top = getPaddingTop();
            child.layout(left, top, left + childWidth, top + childHeight);
            left += (childWidth - overlap); // 下一个 child 的起始位置
        }
    }
}
```

**说明：**
- 每个子 View（扑克牌）都从上一个牌的右侧偏移一定像素（overlap），实现部分重叠的视觉效果。
- 可以通过调整 overlap 的值来控制重叠程度。
- 这种布局常用于卡牌游戏的手牌展示。 