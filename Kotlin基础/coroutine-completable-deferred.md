# Kotlin协程中的CompletableDeferred与嵌套Launch分析

## CompletableDeferred解析

### 定义与作用
CompletableDeferred是Deferred接口的可完成版本，允许手动设置结果值。与普通Deferred不同，它提供了`complete()`和`completeExceptionally()`方法。

```kotlin
val deferred = CompletableDeferred<String>()
launch {
    delay(1000)
    deferred.complete("Done")
}
```

### 同类概念比较

#### Deferred vs CompletableDeferred
```
.
├── Deferred                # 只能通过async协程构建器创建
└── CompletableDeferred      # 可手动完成的结果容器
```

#### Channel vs CompletableDeferred
```
.
├── Channel                 # 多值通信管道
└── CompletableDeferred      # 单值结果容器
```

## 嵌套Launch行为分析

### 结构化并发示例
```kotlin
fun main() = runBlocking {
    // 外层launch
    launch(CoroutineName("parent")) {
        // 内层launch
        launch(CoroutineName("child")) {
            delay(100)
            println("Child coroutine")
        }
        println("Parent coroutine")
    }
}
```

### 执行流程
```
.
├── runBlocking
│   └── parent
│       └── child
```

## 实践问题与解决方案

### 问题1: 取消传播
当父协程被取消时，所有子协程会自动取消。

解决方案：
```kotlin
val job = launch {
    val child = launch {
        try {
            delay(Long.MAX_VALUE)
        } finally {
            println("Child cancelled")
        }
    }
    delay(100)
    job.cancel()
}
```

## 面试题解析

### "There is more than one label with such a name in this scope"

#### 问题原因
当在嵌套launch中使用相同标签名时会出现此错误：
```kotlin
launch(label@ {
    launch(label@ { // 错误！重复标签
        // ...
    })
})
```

#### 解决方案
1. 使用不同标签名：
```kotlin
launch(parentLabel@ {
    launch(childLabel@ {
        // ...
    })
})
```

2. 使用结构化并发替代：
```kotlin
launch {
    launch { // 自动继承父协程上下文
        // ...
    }
}
```

#### 最佳实践
- 避免在嵌套协程中重复使用标签
- 优先使用结构化并发机制
- 必要时使用唯一协程名进行区分
