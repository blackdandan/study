# Looper（主线程消息循环机制）详解

`Looper` 是 Android 消息循环的核心，负责不断从 `MessageQueue` 取出消息并分发。

## 主要作用

- 管理消息循环
- 关联线程的 `MessageQueue`

## 基本用法

```java
Looper.prepare();
Handler handler = new Handler();
Looper.loop();
```

## 主线程 Looper

- 主线程在应用启动时自动创建 Looper
- 通过 `Looper.getMainLooper()` 获取

## 工作原理

1. `Looper` 关联当前线程的 `MessageQueue`
2. 不断循环，取出消息并分发

// ... 可补充更多内容 ... 