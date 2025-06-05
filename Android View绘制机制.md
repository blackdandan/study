# Android View绘制机制

## 绘制流程概述
Android View的绘制分为三个主要步骤：
1. Measure (测量)
2. Layout (布局) 
3. Draw (绘制)

## 类图
```
+------------------+
|      View        |
+------------------+
| +onMeasure()     |
| +onLayout()      |
| +onDraw()        |
+------------------+
      ^
      |
+-----+------+
| ViewGroup  |
+------------+
| +dispatchDraw() |
+------------+
```

## 关键方法解析
```java
// View.java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    // 测量View大小
}

protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
    // 确定子View位置
}

protected void onDraw(Canvas canvas) {
    // 绘制View内容
}
```

## 性能优化技巧
1. 减少View层级
2. 使用ViewStub延迟加载
3. 避免在onDraw中创建对象

## 实践中遇到的问题
1. 问题：复杂布局导致卡顿
   解决方案：使用Hierarchy Viewer分析并简化布局层级

2. 问题：过度绘制
   解决方案：开启GPU过度绘制调试，减少不必要的背景绘制

## 面试问题与答案

1. **描述View的绘制流程**
   - 答案：View的绘制分为三个阶段：
     1) Measure：测量View的宽高，通过onMeasure()方法完成
     2) Layout：确定View在父容器中的位置，通过onLayout()方法完成
     3) Draw：实际绘制View内容，通过onDraw()方法完成

2. **onMeasure()中MeasureSpec的作用**
   - 答案：MeasureSpec是一个32位int值，包含测量模式和尺寸信息：
     - EXACTLY：精确尺寸
     - AT_MOST：最大尺寸
     - UNSPECIFIED：未指定
     父View通过MeasureSpec向子View传递布局要求

3. **如何优化自定义View的性能**
   - 答案：
     1) 减少不必要的invalidate()调用
     2) 避免在onDraw()中创建对象
     3) 使用canvas.clipRect()限制绘制区域
     4) 考虑使用硬件加速

4. **invalidate()和requestLayout()的区别**
   - 答案：
     - invalidate()：只触发重绘(onDraw)
     - requestLayout()：会触发重新测量(onMeasure)和重新布局(onLayout)
     - 需要改变View大小时用requestLayout()
     - 只更新显示内容时用invalidate()

5. **实现一个自定义View(不是ViewGroup)也要复写onLayout/onMeasure方法吗**
   - 答案：
     1) onMeasure：通常需要重写，用于确定View的尺寸
     2) onLayout：一般不需要重写，因为非ViewGroup的View没有子View需要布局
     3) 特殊情况：如果View需要特殊测量逻辑(如固定宽高比)才需要重写onMeasure
     4) 简单View可以直接使用默认实现
