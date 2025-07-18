# Android DataStore 详解

## 演进过程
```
SharedPreferences → DataStore
├── 基于XML → 基于Protocol Buffers
├── 同步API → 协程/Kotlin Flow支持
├── 无类型安全 → 强类型支持
└── 主线程操作 → 异步设计
```

DataStore是Android Jetpack组件中用于替代SharedPreferences的现代化数据存储解决方案，于2020年推出。

## 替代技术
DataStore主要替代了以下技术：
1. SharedPreferences
   - 解决了SP的同步API、主线程阻塞问题
   - 提供了类型安全和错误处理机制
2. 部分SQLite使用场景
   - 对于简单的键值对存储，比SQLite更轻量

## 使用方法

### 1. 添加依赖
```kotlin
implementation "androidx.datastore:datastore-preferences:1.0.0"
// 或使用Proto DataStore
implementation "androidx.datastore:datastore:1.0.0" 
```

### 2. 创建DataStore
```kotlin
// Preferences DataStore
val Context.dataStore by preferencesDataStore(name = "settings")

// Proto DataStore
val Context.userDataStore by dataStore(
    fileName = "user.pb",
    serializer = UserSerializer
)
```

### 3. 读写操作
```kotlin
// 写入
suspend fun saveSettings(enable: Boolean) {
    context.dataStore.edit { preferences ->
        preferences[PreferencesKeys.ENABLED] = enable
    }
}

// 读取
val enabledFlow: Flow<Boolean> = context.dataStore.data
    .map { preferences ->
        preferences[PreferencesKeys.ENABLED] ?: false
    }
```

## 源码原理

### 核心架构
```
DataStore 核心组件
├── DataStoreFactory - 创建DataStore实例
├── Serializer - 数据序列化接口
├── DataMigration - 数据迁移支持
└── DataStore - 核心存储接口
```

### 关键实现细节
1. 基于Kotlin协程和Flow实现异步操作
2. 使用内存缓存减少IO操作
3. 文件操作使用原子提交保证数据一致性
4. 通过拦截器支持数据迁移

## 与其他框架比较

| 特性        | DataStore | SharedPreferences | Room |
|------------|-----------|-------------------|------|
| 异步支持     | ✅        | ❌                | ✅   |
| 类型安全     | ✅        | ❌                | ✅   |
| 关系型数据   | ❌        | ❌                | ✅   |
| 协程/Flow   | ✅        | ❌                | ✅   |
| 数据迁移     | ✅        | ❌                | ✅   |

优势：
1. 专为Kotlin设计，协程支持完善
2. 类型安全，减少运行时错误
3. 支持数据迁移
4. 更现代的API设计

劣势：
1. 不支持Java流式API
2. 学习曲线比SharedPreferences高
3. 对简单场景可能过度设计

## 实践中遇到的问题及解决方案

### 问题1：多进程访问冲突
**现象**：多进程同时写入导致数据不一致  
**解决方案**：
- 使用单例模式管理DataStore实例
- 或者考虑使用Room数据库替代

### 问题2：大数据量性能问题
**现象**：存储大量数据时性能下降  
**解决方案**：
- 将大数据拆分为多个DataStore文件
- 考虑使用Proto DataStore替代Preferences DataStore

## 面试常见问题

### Q1: DataStore与SharedPreferences的主要区别？
**答**：
1. API设计：DataStore基于协程和Flow，SP是同步API
2. 线程安全：DataStore默认线程安全，SP需要手动处理
3. 错误处理：DataStore提供明确错误处理机制
4. 类型安全：DataStore支持类型安全，SP所有值都是Object

### Q2: 如何实现DataStore的数据迁移？
**答**：
1. 实现DataMigration接口
2. 在创建DataStore时添加migrations参数
3. 示例：
```kotlin
val dataStore = context.dataStore(
    migrations = listOf(SharedPreferencesMigration(context, "old_prefs"))
)
```

### Q3: Preferences DataStore和Proto DataStore如何选择？
**答**：
- Preferences DataStore：适合简单键值对，类似SP的使用场景
- Proto DataStore：适合复杂结构化数据，需要类型安全和协议定义
