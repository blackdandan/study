# 自定义View重绘性能优化指南

## 优化核心原则
```
优化层次
├── 减少重绘频率
├── 缩小重绘区域
├── 简化绘制操作
└── 合理使用硬件加速
```

## 源代码级优化技术

### 1. 减少无效重绘
```java
// 优化前：频繁触发重绘
public void updateData(Data data) {
    this.data = data;
    invalidate(); // 每次数据变化都重绘
}

// 优化后：条件重绘
public void updateData(Data data) {
    if(!this.data.equals(data)) { // 只有数据确实变化时才重绘
        this.data = data;
        invalidate();
    }
}
```

### 2. 精确指定重绘区域
```java
@Override
protected void onDraw(Canvas canvas) {
    // 只重绘变化部分
    canvas.clipRect(dirtyRect); 
    // 绘制逻辑...
}

public void partialUpdate(Rect dirtyArea) {
    invalidate(dirtyArea); // 只重绘指定区域
}
```

**实现原理**：
1. `invalidate(Rect)`会标记指定矩形区域为脏区域
2. 系统合并多个脏区域形成最小包围矩形
3. `onDraw()`中通过`canvas.clipRect()`限制绘制范围
4. 绘制时系统会应用裁剪区域，只更新指定部分
5. 最终通过SurfaceFlinger合成时只更新变化区域

**注意事项**：
- 需要精确计算变化区域边界
- 多个脏区域会合并，频繁小区域更新可能退化为全屏重绘
- 适用于有明显边界变化的场景（如指针旋转、局部动画）

### 3. 使用Canvas高效API
```java
// 避免使用高开销操作
canvas.drawPath(); // 高开销
canvas.drawCircle(); // 低开销

// 使用批量绘制
canvas.drawLines(); // 优于多次drawLine
```

**性能差异分析**：
1. `drawPath`高开销原因：
   - 需要计算贝塞尔曲线等复杂路径
   - 抗锯齿计算的高消耗：
     ```
     抗锯齿性能消耗
     ├── 采样计算：每个边缘像素需要多次采样
     ├── 混合操作：alpha混合计算增加
     ├── 内存访问：频繁读写帧缓冲区
     └── 同步开销：CPU/GPU数据传输
     ```
   - 路径填充需要更多GPU运算
   - 无法使用硬件加速优化路径

2. 抗锯齿优化建议：
   - 对静态内容预渲染为Bitmap
   - 在滚动/动画时禁用抗锯齿
   - 使用整型坐标减少边缘锯齿
   - 对不重要元素关闭抗锯齿

2. `drawCircle`低开销原因：
   - 有固定数学公式可快速计算
   - 硬件有专门的圆形绘制优化
   - 边界计算简单，易于缓存
   - 支持硬件加速绘制流水线

3. 推荐替代方案：
   - 简单图形优先使用基本绘制方法
   - 复杂路径考虑预渲染为Bitmap
   - 使用`drawRoundRect`替代部分路径绘制

## 性能分析工具

### 1. 使用Systrace分析
```
systrace分析流程
├── 收集trace数据
├── 识别性能瓶颈
└── 针对性优化
```

### 2. 调试标记
```java
// 在自定义View中添加调试标记
private static final boolean DEBUG = false;

if(DEBUG) {
    Log.d("CustomView", "onDraw耗时: " + (endTime - startTime));
}
```

## 高级优化技巧

### 1. 分层绘制与硬件加速
```java
// 使用View.setLayerType
setLayerType(LAYER_TYPE_HARDWARE, null); // 硬件加速层
```

**硬件加速原理**：
1. 基本概念：
   - 将图形计算从CPU转移到GPU
   - 利用GPU并行计算能力加速渲染
   - 通过OpenGL ES实现跨平台支持

2. 核心组件：
   - **显示列表(DisplayList)**：记录绘制命令的中间表示
   - **纹理(Texture)**：离屏缓冲的位图数据
   - **合成器(Compositor)**：管理多个图层的合成

3. 工作流程：
   ```
   硬件加速流程
   ├── 构建阶段
   │   ├── 将View绘制命令转为显示列表
   │   └── 复杂View预渲染为纹理
   ├── 绘制阶段
   │   ├── GPU执行显示列表命令
   │   └── 复用已缓存的纹理
   └── 合成阶段
       ├── 多个图层合成
       └── 最终输出到屏幕
   ```

4. 性能优势：
   - 并行处理图形计算
   - 减少主线程负担
   - 高效处理变换和动画
   - 支持3D图形效果

**详细使用方法**：
1. 三种图层类型：
   - `LAYER_TYPE_NONE`：默认，无额外图层
   - `LAYER_TYPE_SOFTWARE`：软件渲染离屏缓冲
   - `LAYER_TYPE_HARDWARE`：硬件加速离屏缓冲

2. 性能优势：
   - 复杂View只需渲染一次到纹理
   - 动画时直接复用纹理
   - 避免每帧重新执行绘制代码
   - 支持GPU硬件加速合成

3. 适用场景：
   - 复杂但静态的内容（如渐变背景）
   - 需要应用变换（旋转/缩放）
   - 执行动画时保持平滑

4. 注意事项：
   - 会消耗额外显存（纹理内存）
   - 过度使用可能导致OOM
   - 动态内容需要手动更新图层

### 2. 缓存绘制结果
```java
private Bitmap mCacheBitmap;

@Override
protected void onDraw(Canvas canvas) {
    if(mCacheBitmap == null) {
        mCacheBitmap = Bitmap.createBitmap(width, height, Config.ARGB_8888);
        Canvas cacheCanvas = new Canvas(mCacheBitmap);
        // 绘制到缓存...
    }
    canvas.drawBitmap(mCacheBitmap, 0, 0, null);
}
```

## 实践案例

### 案例1：仪表盘控件优化
**问题**：指针旋转导致频繁全View重绘  
**解决方案**：
```java
// 只重绘指针区域
RectF needleRect = calculateNeedleRect();
invalidate(needleRect);
```

### 案例2：复杂背景处理
**问题**：复杂背景每次重绘耗时  
**解决方案**：
```java
// 预渲染背景到Bitmap
if(backgroundDirty) {
    renderBackgroundToBitmap();
    backgroundDirty = false;
}
canvas.drawBitmap(mBackgroundBitmap, 0, 0, null);
```

## 技术细节补充

### invalidate(Rect)机制详解
1. 参数说明：
   - 接受Rect或RectF指定脏区域
   - 坐标相对于View自身坐标系
   - 支持局部坐标和绝对坐标两种形式
   - 必须在UI线程调用，非UI线程需使用postInvalidate()

2. 底层实现：
   - 通过`ViewRootImpl.invalidateChildInParent()`传递脏区域
   - 最终调用`Surface.lockCanvas()`获取指定区域画布
   - 只更新指定区域的显示缓冲区

3. API版本差异：
   - **API 14以下**：
     - 完全依赖软件渲染
     - 传入的Rect参数可能被修改（破坏性）
     - 精确指定区域能显著提升性能
   - **API 14-20**：
     - 引入硬件加速渲染
     - 脏区域计算重要性降低
     - 系统会自动优化重绘区域
   - **API 21+**：
     - 完全忽略传入的Rect参数
     - 系统内部自动计算最优重绘区域
     - 推荐直接使用invalidate()

4. 最佳实践：
   - 新代码建议直接使用invalidate()
   - 兼容旧代码时可保留invalidate(Rect)调用
   - 仍需避免不必要的全屏重绘
   - 关注整体绘制性能而非局部优化

### scrollTo vs invalidate性能对比
1. `scrollTo`优势：
   - 直接操作显示缓冲区，不触发重绘
   - 只更新显示位置，不重新生成内容
   - 支持硬件加速滚动
   - 适用于内容不变的平移操作

2. 实现原理：
   ```java
   // scrollTo内部实现简析
   public void scrollTo(int x, int y) {
       if (mScrollX != x || mScrollY != y) {
           int oldX = mScrollX;
           int oldY = mScrollY;
           mScrollX = x;
           mScrollY = y;
           invalidateParentCaches();
           onScrollChanged(mScrollX, mScrollY, oldX, oldY);
           if (!awakenScrollBars()) {
               postInvalidateOnAnimation();
           }
       }
   }
   ```

3. 使用场景：
   - 列表/页面滚动
   - 大画布平移
   - 视口移动但内容不变的情况

## 面试常见问题

### Q1: 如何检测自定义View的重绘性能问题？
**答**：
1. 使用Systrace分析绘制耗时
2. 开启GPU过度绘制调试
3. 添加绘制耗时日志
4. 使用Android Profiler监控

### Q2: 什么情况下应该考虑使用硬件加速层？
**答**：
1. 复杂但不频繁变化的绘制内容
2. 需要应用变换(旋转/缩放)时
3. 合成多个绘制元素时
4. 注意：会消耗额外内存

### Q3: 如何优化自定义View的滚动性能？
**答**：
1. 实现onScrollChanged()手动控制重绘
2. 使用scrollTo()代替invalidate()
3. 对静态内容使用缓存Bitmap
4. 禁用滚动时的抗锯齿

## 设计模式应用

### 1. 享元模式(Flyweight)
```java
// 共享绘制资源
private static Paint sSharedPaint = new Paint();

@Override
protected void onDraw(Canvas canvas) {
    sSharedPaint.setColor(Color.RED);
    canvas.drawRect(rect, sSharedPaint);
}
```

### 2. 策略模式(Strategy)
```java
// 根据不同状态选择绘制策略
interface DrawStrategy {
    void draw(Canvas canvas);
}

private DrawStrategy mCurrentStrategy;
