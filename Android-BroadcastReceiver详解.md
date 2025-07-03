# Android BroadcastReceiver详解

## 1. BroadcastReceiver概述
BroadcastReceiver是Android四大组件之一，用于接收系统或应用发出的广播。

```
Android广播类型
.
├── 系统广播        # 如开机、充电、时区变更
├── 自定义广播      # 应用内或跨应用通信
└── 有序广播        # 可被拦截和修改的广播
```

## 2. 注册方式对比

### 2.1 动态注册
```java
// 创建广播接收器
BroadcastReceiver receiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        // 处理广播
    }
};

// 注册广播
IntentFilter filter = new IntentFilter();
filter.addAction("com.example.MY_ACTION");
registerReceiver(receiver, filter);

// 取消注册(通常在onDestroy中)
unregisterReceiver(receiver);
```

#### 特点：
- 生命周期与注册组件绑定
- 灵活性高，可随时注册/取消
- 必须手动取消注册，否则会导致内存泄漏

### 2.2 静态注册
```xml
<receiver android:name=".MyReceiver">
    <intent-filter>
        <action android:name="com.example.MY_ACTION" />
    </intent-filter>
</receiver>
```

#### 特点：
- 在AndroidManifest.xml中声明
- 应用未运行也能接收广播(系统限制)
- 无需手动取消注册
- Android 8.0+对静态注册有限制

## 3. LocalBroadcastManager

### 3.1 使用场景
- 应用内通信
- 避免系统广播的安全和性能问题

### 3.2 基本用法
```java
// 获取实例
LocalBroadcastManager localBroadcastManager = 
    LocalBroadcastManager.getInstance(context);

// 注册接收器
localBroadcastManager.registerReceiver(receiver, filter);

// 发送广播
Intent intent = new Intent("local-action");
localBroadcastManager.sendBroadcast(intent);

// 取消注册
localBroadcastManager.unregisterReceiver(receiver);
```

## 4. 实践问题与解决方案

### 问题1: 接收不到广播
解决方案：
- 检查action是否匹配
- Android 8.0+需使用显式广播或动态注册
- 检查权限声明

### 问题2: ANR(Application Not Responding)
解决方案：
- 避免在onReceive()中执行耗时操作
- 使用goAsync()或启动Service处理

## 5. 面试常见问题

1. Q: 动态注册和静态注册的区别？
A: 
- 动态注册：代码中注册，生命周期与组件绑定
- 静态注册：Manifest声明，可接收系统广播

2. Q: Android 8.0对广播的限制？
A: 
- 禁止大部分隐式静态注册
- 需使用动态注册或显式Intent

3. Q: LocalBroadcastManager的优点？
A: 
- 只在应用内传播，更安全
- 效率高于系统广播
- 无需权限声明

4. Q: 如何避免广播导致的ANR？
A: 
- onReceive()执行时间不超过10秒
- 使用goAsync()延长处理时间
- 启动Service处理耗时任务
