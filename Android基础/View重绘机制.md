# Android View重绘机制：invalidate vs requestLayout

## 核心概念对比
```
重绘机制对比
├── invalidate()
│   ├── 触发重绘(onDraw)
│   ├── 不触发重新布局
│   └── 仅标记脏区域
└── requestLayout()
    ├── 触发重新布局(onMeasure/onLayout)
    └── 可能触发重绘
```

## 方法详解

### 1. invalidate()
```java
public void invalidate() {
    // 标记当前View为脏区域
    // 触发onDraw()重绘
}
```

**特点**：
- 只重绘不重新布局
- 适用于内容变化但尺寸不变的情况
- 效率较高，只重绘脏区域

**使用场景**：
- 修改View的显示内容
- 动画效果实现
- 自定义View的视觉更新

### 2. requestLayout()
```java
public void requestLayout() {
    // 请求重新测量和布局
    // 可能触发onDraw()
}
```

**特点**：
- 触发完整的measure/layout流程
- 可能导致整个视图树更新
- 开销比invalidate大

**使用场景**：
- View尺寸发生变化
- 子View数量或位置变化
- 需要重新计算布局参数时

## 源码流程分析

### invalidate调用链
```
invalidate()
└── ViewRootImpl.invalidateChild()
    └── scheduleTraversals()
        └── performTraversals()
            └── performDraw()
                └── draw()
                    └── onDraw()
```

### requestLayout调用链
```
requestLayout()
└── ViewRootImpl.scheduleTraversals()
    └── performTraversals()
        ├── performMeasure()
        ├── performLayout() 
        └── performDraw()
```

## 性能优化建议

1. **减少不必要的requestLayout**：
   - 批量修改布局参数
   - 使用ViewPropertyAnimator处理动画

2. **合理使用invalidate**：
   - 指定重绘区域(invalidate(Rect))
   - 避免在draw方法中调用invalidate

3. **区别使用场景**：
   - 仅视觉变化 → invalidate
   - 布局变化 → requestLayout

## 实践案例

### 案例1：自定义进度条更新
```java
// 只需重绘
public void setProgress(int progress) {
    this.progress = progress;
    invalidate(); // 只需重绘不需要重新布局
}
```

### 案例2：动态添加子View
```java
public void addChildView(View child) {
    addView(child);
    requestLayout(); // 需要重新测量和布局
}
```

## 面试常见问题

### Q1: invalidate和requestLayout的区别？
**答**：
1. invalidate只触发重绘，requestLayout触发完整布局流程
2. invalidate效率更高，requestLayout开销更大
3. invalidate适用于内容变化，requestLayout适用于尺寸变化

### Q2: 什么情况下会触发整个视图树的重绘？
**答**：
1. 调用根View的requestLayout
2. 窗口尺寸变化
3. 配置变更(如横竖屏切换)
4. 添加/移除View时

### Q3: 如何优化自定义View的重绘性能？
**答**：
1. 使用clipRect限制绘制区域
2. 避免在draw方法中分配对象
3. 对静态内容使用setWillNotDraw
4. 合理选择invalidate和requestLayout
