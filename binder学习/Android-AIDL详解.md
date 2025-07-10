# Android AIDL详解

## 1. AIDL概述
AIDL(Android Interface Definition Language)是Android接口定义语言，用于实现跨进程通信(IPC)。

```
Android进程通信方式
.
├── Intent           # 简单数据传递
├── Messenger        # 基于消息的轻量级IPC
├── ContentProvider  # 数据共享
└── AIDL             # 复杂接口调用(支持同步/异步)
```

## 2. AIDL工作原理

### 2.1 核心组件
```
AIDL架构
.
├── Client进程      # 调用远程服务
├── Server进程      # 实现服务功能
└── Binder驱动      # 内核层通信桥梁
```

### 2.2 通信流程
1. 定义AIDL接口
2. 实现Stub类
3. 客户端绑定服务获取Proxy
4. 通过Binder驱动完成跨进程调用

## 3. 完整实现步骤

### 3.1 定义AIDL接口
```java
// IRemoteService.aidl
package com.example.aidl;

interface IRemoteService {
    int getPid();
    void basicTypes(int anInt, long aLong, boolean aBoolean, 
                   float aFloat, double aDouble, String aString);
}
```

### 3.2 服务端实现
```java
public class RemoteService extends Service {
    private final IRemoteService.Stub binder = new IRemoteService.Stub() {
        @Override
        public int getPid() {
            return Process.myPid();
        }
        
        @Override
        public void basicTypes(int anInt, long aLong, boolean aBoolean, 
                              float aFloat, double aDouble, String aString) {
            // 实现逻辑
        }
    };

    @Override
    public IBinder onBind(Intent intent) {
        return binder;
    }
}
```

### 3.3 客户端调用

#### 3.3.1 绑定服务流程
```java
// 1. 创建Intent指定要绑定的服务
Intent intent = new Intent(this, RemoteService.class);

// 2. 绑定服务(flags参数说明见下文)
bindService(intent, connection, Context.BIND_AUTO_CREATE);

// 3. 实现ServiceConnection回调
private ServiceConnection connection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        // 4. 将IBinder转换为AIDL接口
        iRemoteService = IRemoteService.Stub.asInterface(service);
        try {
            // 5. 调用远程方法
            int pid = iRemoteService.getPid();
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
    
    @Override
    public void onServiceDisconnected(ComponentName name) {
        // 服务异常断开处理
        iRemoteService = null;
    }
};

// 6. 解绑服务(通常在onDestroy中调用)
unbindService(connection);
```

#### 3.3.2 绑定标志位(FLAG)
```
绑定服务标志位
.
├── BIND_AUTO_CREATE          # 自动创建服务(如果未运行)
├── BIND_DEBUG_UNBIND         # 用于调试解绑问题
├── BIND_NOT_FOREGROUND       # 不将服务提升为前台优先级
├── BIND_ABOVE_CLIENT         # 服务优先级高于客户端
├── BIND_ALLOW_OOM_MANAGEMENT # 允许系统在内存不足时终止服务
└── BIND_WAIVE_PRIORITY       # 不改变服务进程的调度优先级
```

注意事项：
1. 多个flag可以用"|"组合使用
2. BIND_AUTO_CREATE是最常用的flag
3. 绑定/解绑应成对出现，避免内存泄漏

## 4. AIDL支持的数据类型

### 4.1 基本类型
- int, long, boolean, float, double, String

### 4.2 复杂类型
```
AIDL支持的类型
.
├── Parcelable      # 需要实现Parcelable接口
├── List            # 仅支持ArrayList
└── Map             # 仅支持HashMap
```

## 5. 实际应用场景

### 5.1 多进程应用
- 主进程与插件进程通信
- 主进程与后台服务进程通信

### 5.2 系统服务调用
- 访问系统服务如PackageManager
- 自定义系统服务

## 6. 常见问题与解决方案

### 问题1: 类型不匹配错误
解决方案: 确保客户端和服务端使用相同的AIDL文件

### 问题2: 跨进程调用超时
解决方案: 避免在主线程进行耗时跨进程调用

## 7. 面试常见问题

1. Q: AIDL与Messenger的区别?

## 8. Gradle配置

要在Android项目中使用AIDL，需要在模块的build.gradle文件中添加以下配置：

```groovy
android {
    buildFeatures {
        aidl true
    }
}
```

注意：
1. Android Studio 4.0+ 默认已启用AIDL支持
2. 老版本可能需要手动添加此配置
3. 配置后需要同步Gradle项目

A: 
- AIDL支持同步/异步调用
- Messenger基于Handler实现，只能异步
- AIDL适合复杂接口，Messenger适合简单消息

2. Q: 如何传递自定义对象?
A: 实现Parcelable接口并在AIDL文件中声明

3. Q: AIDL调用是同步还是异步?
A: 默认同步，客户端会阻塞直到服务端返回

4. Q: 如何实现异步AIDL调用?
A: 使用oneway关键字或客户端自行创建线程
