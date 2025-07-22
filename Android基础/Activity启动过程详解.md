# Activity启动过程详解

## 基本流程
1. 调用startActivity()方法
2. 通过Binder IPC机制将请求传递给ActivityManagerService
3. ActivityManagerService处理启动请求
4. 创建目标Activity的进程（如果需要）
5. 执行Activity的生命周期回调

```java
// 示例代码：启动Activity的基本方式
Intent intent = new Intent(this, TargetActivity.class);
startActivity(intent);
```

## 核心步骤详解

### 关键类及其作用
1. **Instrumentation** 
   - 监控应用与系统的交互
   - 负责创建Activity实例
   - 调用Activity的生命周期方法

2. **ActivityTaskManagerService(ATMS)**
   - 管理系统所有Activity的任务和栈
   - 处理Activity启动请求
   - 管理Activity生命周期状态

3. **ActivityThread**
   - 应用的主线程
   - 处理系统消息
   - 执行Activity生命周期回调

4. **ApplicationThread**
   - ATMS与应用进程通信的Binder接口
   - 将系统请求转发给ActivityThread

### 核心源码片段
1. **启动请求入口**
```java
// Activity.java
public void startActivity(Intent intent) {
    mInstrumentation.execStartActivity(
        this, mMainThread.getApplicationThread(), 
        intent, -1);
}
```

2. **ATMS处理请求**
```java
// ActivityTaskManagerService.java
final int startActivity(IApplicationThread caller, 
    Intent intent, String resolvedType, 
    IBinder resultTo, String resultWho, int requestCode,
    int flags, ProfilerInfo profilerInfo, Bundle options) {
    // 验证权限、创建ActivityRecord等
}
```

3. **Activity创建过程**
```java
// ActivityThread.java
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
    // 1. 创建Activity实例
    Activity activity = mInstrumentation.newActivity(
        cl, component.getClassName(), r.intent);
    
    // 2. 调用onCreate()
    mInstrumentation.callActivityOnCreate(activity, r.state);
    
    // 3. 执行其他生命周期方法
}
```

### 详细流程
1. **发起请求阶段**
   - 调用startActivity()
   - 经过Instrumentation处理
   - 通过ActivityTaskManagerService进行跨进程通信

2. **系统处理阶段**
   - ActivityManagerService验证权限和参数
   - 检查目标Activity是否在Manifest中声明
   - 创建或复用目标Activity所在进程

3. **Activity创建阶段**
   - 创建Activity实例
   - 调用onCreate()等生命周期方法
   - 完成视图的创建和显示

## 常见问题与解决方案
1. **ActivityNotFoundException**
   - 原因：未在AndroidManifest.xml中声明Activity
   - 解决：确保所有Activity都在Manifest中正确声明

2. **启动速度慢**
   - 原因：onCreate()中执行耗时操作
   - 解决：优化初始化代码，使用异步加载

## 面试常见问题
1. Activity的启动模式有哪些？各自的应用场景？
2. 简述Activity启动过程中涉及的主要系统服务
3. 如何优化Activity的启动速度？
4. startActivity()和startActivityForResult()的区别？

## Context详解

### Context的产生与持有
1. **创建时机**：
   - Application Context：在应用启动时由ActivityThread创建
   - Activity Context：在Activity创建时由Instrumentation创建

2. **持有关系**：
   - Application持有全局的Base Context
   - 每个Activity持有：
     - 自身的Context（继承自ContextThemeWrapper）
     - Application Context（通过getApplicationContext()获取）

3. **创建过程**：
```java
// ContextImpl创建过程
class ContextImpl extends Context {
    static ContextImpl createAppContext(ActivityThread mainThread, 
        LoadedApk packageInfo) {
        ContextImpl context = new ContextImpl();
        context.init(packageInfo, null, mainThread);
        return context;
    }
}
```

### Context核心成员
1. **资源相关**：
   - Resources mResources
   - Resources.Theme mTheme
   - AssetManager mAssets

2. **组件相关**：
   - ActivityThread mMainThread
   - IBinder mActivityToken
   - ClassLoader mClassLoader

3. **系统服务**：
   - PackageInfo mPackageInfo
   - ContentResolver mContentResolver

### 资源初始化
1. **初始化时机**：
   - Application Context资源在LoadedApk创建时初始化
   - Activity Context资源在Activity创建时初始化

2. **初始化方法**：
```java
// Resources初始化过程
final Resources resources = packageInfo.getResources(mainThread);
if (resources != null) {
    mResources = resources;
    mTheme = resources.newTheme();
    mTheme.applyStyle(theme, true);
}
```

3. **资源类型**：
   - 布局资源（layout）
   - 字符串资源（string）
   - 图片资源（drawable）
   - 样式资源（style）
   - 颜色资源（color）

## 组件关系说明

1. **Activity与Context**
   - Activity继承自ContextThemeWrapper
   - 每个Activity持有基Context(ApplicationContext)和主题资源

2. **Activity与Application**
   - Application是全局单例，早于Activity创建
   - Activity通过getApplication()获取Application实例
   - Application负责管理应用级资源和生命周期

3. **Activity与AMS/ATMS**
   - AMS(ActivityManagerService)管理Activity生命周期
   - ATMS(ActivityTaskManagerService)管理任务栈和ActivityRecord
   - 通过Binder IPC通信

4. **Activity与Instrumentation**
   - Instrumentation监控Activity创建和生命周期调用
   - 每个进程只有一个Instrumentation实例

## 启动时序图

```
+---------+     +---------------+     +-----+     +------+
| Activity|     |Instrumentation|     | ATMS|     |AppThread|
+----+----+     +-------+-------+     +--+--+     +---+---+
     |                  |               |            |
     | startActivity()  |               |            |
     +----------------->|               |            |
     |                  | execStartActivity()        |
     |                  +-------------->|            |
     |                  |               | startActivity()
     |                  |               +------------>|
     |                  |               |            | scheduleLaunchActivity()
     |                  |               |            +------+
     |                  |               |            |      |
     |                  |               |            |<-----+
     |                  |               |            |
     |                  | handleLaunchActivity()     |
     |                  |<---------------------------+
     | performLaunchActivity()          |            |
     |<-----------------+               |            |
     |                  |               |            |
     | onCreate()       |               |            |
     +----------------->|               |            |
```

## 设计模式应用
1. **代理模式**：ActivityManagerService作为代理处理启动请求
2. **工厂模式**：Activity的实例化过程
3. **观察者模式**：生命周期回调机制
