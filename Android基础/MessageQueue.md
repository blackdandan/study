# MessageQueue 详解

`MessageQueue` 是消息队列，存储即将被处理的消息（Message）。

## 主要作用

- 存储消息和任务
- 按顺序分发给 `Handler` 处理

## 工作流程

1. `Handler` 发送消息到 `MessageQueue`
2. `Looper` 轮询 `MessageQueue`，取出消息
3. 交给对应的 `Handler` 处理

## 特点

- 每个线程只有一个 `MessageQueue`
- 主线程的 `MessageQueue` 由 `Looper.prepareMainLooper()` 创建

// ... 可补充更多内容 ...