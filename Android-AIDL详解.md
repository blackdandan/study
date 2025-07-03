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
```java
// 绑定服务
Intent intent = new Intent(this, RemoteService.class);
bindService(intent, connection, Context.BIND_AUTO_CREATE);

// ServiceConnection实现
private ServiceConnection connection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName name, IBinder service) {
        iRemoteService = IRemoteService.Stub.asInterface(service);
        try {
            int pid = iRemoteService.getPid();
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
};
```

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
