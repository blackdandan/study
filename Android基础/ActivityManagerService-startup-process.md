# ActivityManagerService (AMS) 启动流程详解

## 1. AMS概述
ActivityManagerService是Android系统中核心服务之一：
- 运行在SystemServer进程内
- 整个系统只有一个AMS实例（单例模式）
- 不是独立进程，而是SystemServer持有的服务对象
- 负责管理所有应用进程的生命周期
- 协调四大组件(Activity/Service/Broadcast/ContentProvider)的运行

## 2. 启动流程

### 2.0 启动背景
Android系统启动流程关键步骤：
1. init进程(PID=1)启动，解析init.rc配置文件
2. init启动zygote进程(Android特有进程)
3. zygote fork出SystemServer进程
4. SystemServer启动AMS等核心服务

SystemServer是Android系统启动过程中的核心进程，由Zygote进程fork产生。它负责创建和运行所有系统核心服务，包括AMS。

> 注：
> - init进程(PID=1)：
>   - 系统第一个用户空间进程
>   - 整个系统生命周期唯一存在
>   - 负责维护其他关键进程
> - zygote进程：
>   - Android特有进程孵化器
>   - 系统唯一存在（主zygote）
>   - 持续运行用于快速创建应用进程
> - SystemServer进程：
>   - 由zygote fork产生
>   - 系统唯一实例
>   - 崩溃会导致系统重启
> - 进程创建方式：
>   - init通过exec启动zygote（完全替换）
>   - zygote通过fork创建SystemServer（复制自身）

### 2.1 系统服务全面列表

#### 1. Bootstrap Services（引导服务）
- ActivityManagerService (AMS)
- PowerManagerService
- PackageManagerService
- DisplayManagerService
- UserManagerService
- SensorService
- LightsService

#### 2. Core Services（核心服务）
- BatteryService
- UsageStatsService
- WebViewUpdateService
- NetworkManagementService
- NetworkStatsService
- NetworkPolicyManagerService

#### 3. Other Services（其他服务）
- WindowManagerService
- InputManagerService
- StatusBarManagerService
- NotificationManagerService
- DevicePolicyManagerService
- LocationManagerService

> 注：
> - AOSP(Android Open Source Project)：Android开源项目，包含Android系统完整源代码
> - 不同Android版本服务可能有增减
> - 厂商定制ROM可能添加额外服务
> - 总服务数通常在20-30个左右
> - 完整列表可查看AOSP中SystemServer.java源码

#### 关键核心服务（加★标记）：
- ★ ActivityManagerService (AMS) - 应用生命周期管理
- ★ PackageManagerService - 应用包管理
- ★ WindowManagerService - 窗口管理
- ★ PowerManagerService - 电源管理
- ★ BatteryService - 电池状态管理

### 2.2 详细步骤
1. **创建AMS实例**：
   - SystemServer.main()启动
   - 调用startBootstrapServices()创建AMS实例
   - 关键代码：
     ```java
     mActivityManagerService = new ActivityManagerService(context);
     ```

2. **设置系统进程**：
   - 调用setSystemProcess()注册AMS到ServiceManager
   - 关键服务注册：
     ```java
     ServiceManager.addService(Context.ACTIVITY_SERVICE, this);
     ```

3. **系统准备就绪**：
   - 调用systemReady()通知AMS系统已启动完成
   - 启动Home Activity

## 3. 关键类和方法
- `ActivityManagerService`：核心实现类(Java)
- `SystemServer`：系统服务启动入口(Java实现)
- `ActivityStackSupervisor`：Activity栈管理
- `ProcessRecord`：进程信息记录

> 注：
> - Android系统服务(如AMS)运行在SystemServer进程内
> - 这些服务是SystemServer中的Java对象，不是独立进程
> - 主要使用Java实现，部分功能通过JNI调用Native代码(C/C++)

## 4. 实践问题与解决方案

### 问题1：AMS启动超时
**现象**：系统启动时AMS初始化卡住
**解决方案**：
- 检查依赖服务是否正常启动
- 增加启动超时监控

### 问题2：Service注册失败
**现象**：AMS无法注册到ServiceManager
**解决方案**：
- 检查ServiceManager状态
- 确保有足够权限

## 5. 面试题

### Q1: AMS的主要职责是什么？
A1: 
- 管理所有应用进程的生命周期
- 协调四大组件的运行
- 系统唯一实例，运行在SystemServer进程内

### Q2: AMS启动过程中systemReady()的作用？
A2: 通知系统服务启动完成，准备启动应用进程。

### Q3: AMS如何管理Activity栈？
A3: 通过ActivityStackSupervisor管理多个Activity栈。
