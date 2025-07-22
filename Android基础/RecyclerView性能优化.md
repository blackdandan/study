# RecyclerView性能优化指南

## 核心优化方向
```
优化维度
├── 缓存策略优化
├── 数据更新优化
├── 布局优化
└── 线程优化
```

## 缓存策略深度解析

### 1. 四级缓存机制及源码分析
```
RecyclerView缓存层级
├── Scrap缓存 (mAttachedScrap/mChangedScrap)
│   ├── 存储当前屏幕可见Item
│   └── 复用时不需重新绑定数据
├── Cache缓存 (mCachedViews)
│   ├── 默认大小2
│   └── 保存最近离开屏幕的Item
├── ViewPool (RecycledViewPool)
│   ├── 全局共享
│   └── 按ViewType分类存储
└── 自定义缓存
    ├── 扩展RecycledViewPool
    └── 实现自定义复用逻辑
```

**源码关键点**：
1. 缓存获取顺序：
```java
// RecyclerView.Recycler.getViewForPosition()
ViewHolder tryGetViewHolderForPositionByDeadline() {
    // 1. 从Scrap缓存获取
    // 2. 从Cache缓存获取  
    // 3. 从ViewPool获取
    // 4. 创建新ViewHolder
}
```

2. 缓存回收逻辑：
```java
// LayoutManager.scrapOrRecycleView()
void scrapOrRecycleView(Recycler recycler, View view) {
    if (viewHolder.isInvalid() && !viewHolder.isRemoved()) {
        recycler.recycleViewHolderInternal(viewHolder); // 进入Cache/ViewPool
    } else {
        recycler.scrapView(view); // 进入Scrap
    }
}
```

### 2. 缓存配置实践

#### 何时增大itemViewCacheSize
```java
// 适用场景：
// 1. 需要快速来回滚动的列表
// 2. Item布局复杂耗时
// 3. 有动画效果的场景

// 示例：电商商品列表
recyclerView.setItemViewCacheSize(10); // 默认2

// 监控缓存命中率（示例）
recyclerView.setRecyclerListener(new RecyclerView.RecyclerListener() {
    @Override
    public void onViewRecycled(RecyclerView.ViewHolder holder) {
        // 统计各缓存层级的命中情况
    }
});
```

#### 缓存命中率统计实现
```java
// 自定义RecycledViewPool扩展统计功能
class StatsRecycledViewPool extends RecyclerView.RecycledViewPool {
    private Map<Integer, Integer> hitCount = new HashMap<>();
    private Map<Integer, Integer> missCount = new HashMap<>();

    @Override
    public ViewHolder getRecycledView(int viewType) {
        ViewHolder holder = super.getRecycledView(viewType);
        if (holder != null) {
            hitCount.put(viewType, getHitCount(viewType) + 1);
        } else {
            missCount.put(viewType, getMissCount(viewType) + 1); 
        }
        return holder;
    }

    public float getHitRate(int viewType) {
        return (float)hitCount.getOrDefault(viewType, 0) / 
               (hitCount.getOrDefault(viewType, 0) + missCount.getOrDefault(viewType, 0));
    }
}
```

#### setItemPrefetchEnabled变更说明
```
预加载机制演进
├── Android 5.0: 引入基础预取
├── Android 7.0: 默认启用预取
├── Android 10: 移除setItemPrefetchEnabled API
└── 当前建议：
    ├── 使用setInitialPrefetchItemCount
    └── 确保LayoutManager支持预取
```
```java
// 设置缓存大小
recyclerView.setItemViewCacheSize(20); // 默认2
recyclerView.setRecycledViewPool(customPool);

// 预加载配置
recyclerView.setInitialPrefetchItemCount(10);
```

## DiffUtil异步优化

### 1. 基础用法
```java
// 实现DiffUtil.Callback
class MyDiffCallback extends DiffUtil.Callback {
    // 实现比较方法...
}

// 同步计算差异
DiffUtil.DiffResult result = DiffUtil.calculateDiff(new MyDiffCallback(oldList, newList));
result.dispatchUpdatesTo(adapter);
```

### 2. 异步优化方案
```java
// 使用RxJava或协程异步计算
Observable.fromCallable(() -> 
    DiffUtil.calculateDiff(new MyDiffCallback(oldList, newList))
    .subscribeOn(Schedulers.computation())
    .observeOn(AndroidSchedulers.mainThread())
    .subscribe(result -> 
        result.dispatchUpdatesTo(adapter));
```

## 高级优化技巧

### 1. 预加载优化
```java
// 启用Item预取
recyclerView.setItemPrefetchEnabled(true);

// 自定义LayoutManager预加载
layoutManager.setInitialPrefetchItemCount(5);
```

### 2. 视图类型优化
```java
// 减少getItemViewType变化
@Override
public int getItemViewType(int position) {
    // 返回稳定的类型值
    return data.get(position).getStableType();
}
```

## 实践案例

### 案例1：复杂列表卡顿优化
**问题**：多种Item类型导致频繁创建ViewHolder  
**解决方案**：
```java
// 扩大RecycledViewPool
RecyclerView.RecycledViewPool pool = new RecyclerView.RecycledViewPool();
pool.setMaxRecycledViews(TYPE_A, 10);
pool.setMaxRecycledViews(TYPE_B, 10);
recyclerView.setRecycledViewPool(pool);
```

### 案例2：数据更新闪烁
**问题**：全量notifyDataSetChanged导致UI闪烁  
**解决方案**：
```java
// 使用DiffUtil局部更新
AsyncListDiffer<Item> differ = new AsyncListDiffer<>(this, new MyDiffCallback());
differ.submitList(newList); // 自动异步计算差异
```

## 面试常见问题

### Q1: RecyclerView缓存机制是怎样的？
**答**：
1. 四级缓存体系：Scrap、Cache、ViewPool、自定义
2. Scrap缓存当前屏幕Item，快速复用
3. Cache缓存最近离开屏幕的Item
4. ViewPool全局共享，减少对象创建

### Q2: 如何优化DiffUtil的性能？
**答**：
1. 异步计算差异（RxJava/协程）
2. 实现高效equals()和hashCode()
3. 使用AsyncListDiffer自动管理
4. 避免在主线程执行calculateDiff

### Q3: 什么情况下需要自定义RecycledViewPool？
**答**：
1. 多RecyclerView共享相同Item类型
2. 需要扩大特定类型的缓存数量
3. 有特殊复用需求的场景
4. 需要统计缓存命中率时

## 设计模式应用

### 1. 享元模式
```java
// 通过RecycledViewPool实现视图复用
recyclerView.setRecycledViewPool(sharedPool);
```

### 2. 观察者模式
```java
// ListAdapter自动处理数据更新
listAdapter.submitList(newData); // 内部使用AsyncListDiffer
