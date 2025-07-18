# 自定义ViewGroup事件分发机制

## 事件分发流程概述

### 基本流程
```
事件分发流程
├── dispatchTouchEvent - 事件分发入口
│   ├── onInterceptTouchEvent - 拦截判断
│   └── 子View处理流程
└── onTouchEvent - 最终处理
```

### U型分发机制
```
父ViewGroup                    子View                    父ViewGroup
    │                            │                            │
    │ dispatchTouchEvent         │                            │
    │───────────────────────────>│                            │
    │                            │ dispatchTouchEvent         │
    │                            │───────────────────────────>│
    │                            │                            │ onTouchEvent
    │                            │<───────────────────────────│
    │ onTouchEvent               │                            │
    │<───────────────────────────│                            │
    │                            │                            │
```

## 核心方法详解

### 1. dispatchTouchEvent
```java
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    // 1. 判断是否拦截(onInterceptTouchEvent)
    // 2. 未拦截则分发给子View
    // 3. 无子View处理则调用自身onTouchEvent
}
```

**关键点**：
- 事件分发的入口方法
- 控制事件流向的核心枢纽
- 返回值决定事件是否被消费

### 2. onInterceptTouchEvent
```java
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    // 根据业务逻辑决定是否拦截事件
    return false; // 默认不拦截
}
```

**拦截场景**：
- 需要处理滑动冲突时
- 特定手势识别时
- 业务逻辑需要独占事件时

### 3. onTouchEvent
```java
@Override
public boolean onTouchEvent(MotionEvent event) {
    // 处理触摸事件
    return super.onTouchEvent(event);
}
```

**处理要点**：
- 返回true表示消费事件
- 返回false会触发父View的onTouchEvent
- 通常处理ACTION_DOWN/ACTION_MOVE/ACTION_UP

## 三方法协作流程
```
事件分发协作流程
1. dispatchTouchEvent接收事件
   ├── 调用onInterceptTouchEvent判断是否拦截
   │   ├── 拦截: 转给自身onTouchEvent处理
   │   └── 不拦截: 分发给子View
   └── 若无子View处理，调用自身onTouchEvent
```

## 实践案例

### 案例1：横向滑动的ViewGroup
```java
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
    switch (ev.getAction()) {
        case MotionEvent.ACTION_DOWN:
            mLastX = ev.getX();
            break;
        case MotionEvent.ACTION_MOVE:
            float dx = ev.getX() - mLastX;
            if (Math.abs(dx) > mTouchSlop) {
                return true; // 拦截横向滑动
            }
            break;
    }
    return super.onInterceptTouchEvent(ev);
}
```

### 问题：事件冲突解决
**现象**：ViewGroup和子View都需要处理滑动  
**解决方案**：
1. 外部拦截法：在ViewGroup的onInterceptTouchEvent中判断
2. 内部拦截法：子View通过requestDisallowInterceptTouchEvent控制

## 面试常见问题

### Q1: 三个方法的执行顺序？
**答**：
1. dispatchTouchEvent最先被调用
2. 在dispatchTouchEvent中会调用onInterceptTouchEvent
3. 最后调用onTouchEvent

### Q2: 如何解决滑动冲突？
**答**：
1. 确定冲突类型：同方向/不同方向
2. 外部拦截法：重写父容器的onInterceptTouchEvent
3. 内部拦截法：子View控制父容器拦截

### Q3: onTouchEvent返回false会发生什么？
**答**：
1. 当前View不再接收后续事件
2. 事件会传递给父View的onTouchEvent
3. 如果所有View都不处理，事件会被丢弃
