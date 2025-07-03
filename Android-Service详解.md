# Android Service详解

## 1. Service概述
Service是Android四大组件之一，用于在后台执行长时间运行操作。

```
Android组件关系图
.
├── Activity      # 用户界面
├── Service       # 后台服务
├── Broadcast     # 广播接收
└── ContentProvider # 数据共享
```

## 2. Service进程类型

### 2.1 主进程服务
- 默认运行在主线程
- 示例代码：
```java
public class MainService extends Service {
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
}
```

### 2.2 独立进程服务
- 在AndroidManifest.xml中配置：
```xml
<service
    android:name=".RemoteService"
    android:process=":remote" />
```

## 3. 前台服务与后台服务

### 3.1 前台服务
必须显示通知，优先级高，适用于音乐播放、位置跟踪等场景。

#### 完整实现步骤：
1. 创建通知渠道(Android 8.0+必需)
```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
    NotificationChannel channel = new NotificationChannel(
        "channel_id", 
        "Channel Name",
        NotificationManager.IMPORTANCE_DEFAULT);
    getSystemService(NotificationManager.class)
        .createNotificationChannel(channel);
}
```

#### NotificationChannel详解：
- **作用**：Android 8.0引入的通知渠道系统，用于分类管理通知
- **核心功能**：
  - 允许用户按渠道单独控制通知行为
  - 统一管理同一类型的通知
  - 设置通知优先级、声音、震动等属性
- **重要参数**：
  - channel_id: 渠道唯一标识
  - name: 用户可见的渠道名称
  - importance: 通知重要性级别(IMPORTANCE_HIGH/IMPORTANCE_DEFAULT等)
- **最佳实践**：
  - 为不同类型通知创建不同渠道
  - 渠道创建后无法修改部分属性
  - 应提供有意义的渠道名称和描述

2. 构建通知
```java
// 基础通知
Notification notification = new NotificationCompat.Builder(this, "channel_id")
    .setContentTitle("服务运行中")
    .setContentText("正在执行后台任务")
    .setSmallIcon(R.drawable.ic_notification)
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    .build();

// 自定义通知示例
Notification customNotification = new NotificationCompat.Builder(this, "channel_id")
    .setContentTitle("文件下载")
    .setContentText("下载进度: 50%")
    .setSmallIcon(R.drawable.ic_download)
    // 添加大图样式
    .setStyle(new NotificationCompat.BigPictureStyle()
        .bigPicture(BitmapFactory.decodeResource(getResources(), R.drawable.download_big)))
    // 添加进度条
    .setProgress(100, 50, false)
    // 添加操作按钮
    .addAction(R.drawable.ic_pause, "暂停", pausePendingIntent)
    .addAction(R.drawable.ic_cancel, "取消", cancelPendingIntent)
    .setPriority(NotificationCompat.PRIORITY_DEFAULT)
    .build();
```

#### 通知自定义方式：
1. **大图样式**：
   - BigPictureStyle: 显示大图
   - BigTextStyle: 显示长文本
   - InboxStyle: 列表样式

2. **进度通知**：
   - setProgress(max, progress, indeterminate)
   - 适用于文件下载/上传等场景

3. **操作按钮**：
   - addAction(): 添加最多3个操作按钮
   - 需要配合PendingIntent实现点击事件

4. **完全自定义UI(RemoteViews)**：
```java
// 1. 创建自定义布局
RemoteViews remoteViews = new RemoteViews(getPackageName(), R.layout.custom_notification);

// 2. 设置布局元素
remoteViews.setTextViewText(R.id.title, "自定义通知");
remoteViews.setImageViewResource(R.id.icon, R.drawable.custom_icon);
remoteViews.setProgressBar(R.id.progress_bar, 100, 50, false);

// 3. 构建通知
Notification customNotification = new NotificationCompat.Builder(this, "channel_id")
    .setSmallIcon(R.drawable.ic_notification)
    .setCustomContentView(remoteViews)  // 设置自定义视图
    .setStyle(new NotificationCompat.DecoratedCustomViewStyle()) // 保持系统装饰
    .build();

// 4. 更新自定义视图(可选)
remoteViews.setTextViewText(R.id.title, "更新后的标题");
notificationManager.notify(1, customNotification);
```

**通知元素与Service通信**：
```java
// 1. 创建PendingIntent
Intent intent = new Intent(this, MyService.class);
intent.setAction("ACTION_UPDATE");
intent.putExtra("command", "pause");
PendingIntent pendingIntent = PendingIntent.getService(
    this, 
    0, 
    intent, 
    PendingIntent.FLAG_UPDATE_CURRENT | PendingIntent.FLAG_IMMUTABLE
);

// 2. 在RemoteViews中设置点击事件
remoteViews.setOnClickPendingIntent(R.id.btn_pause, pendingIntent);

// 3. Service中处理Intent
@Override
public int onStartCommand(Intent intent, int flags, int startId) {
    if (intent != null && "ACTION_UPDATE".equals(intent.getAction())) {
        String command = intent.getStringExtra("command");
        if ("pause".equals(command)) {
            // 处理暂停逻辑
        }
    }
    return super.onStartCommand(intent, flags, startId);
}
```

**注意事项**：
- 自定义布局必须使用RemoteViews支持的元素
- 布局文件应尽量简单，避免复杂视图
- Android 12+对自定义通知有更多限制
- 不同厂商ROM可能有兼容性问题
- 更新频率不宜过高，避免性能问题
- 使用FLAG_IMMUTABLE保证安全性(Android 12+要求)

3. 启动前台服务
```java
startForeground(1, notification);
```

#### 注意事项：
- Android 9.0+需要请求FOREGROUND_SERVICE权限
- 必须设置小图标，否则会崩溃
- 服务停止时需要调用stopForeground(true)
- 不同Android版本通知样式可能有差异

#### 实际应用场景：
1. 音乐播放器
2. 文件下载/上传
3. 位置跟踪
4. 即时通讯消息接收

### 3.2 后台服务
默认类型，Android 8.0+限制较多

## 4. AIDL进程通信

### 4.1 定义AIDL接口
```java
// IMyService.aidl
interface IMyService {
    int getPid();
}
```

### 4.2 服务端实现
```java
private final IMyService.Stub binder = new IMyService.Stub() {
    public int getPid() {
        return Process.myPid();
    }
};水水水水水水水水水水水水水水顶顶顶顶顶                                                                                                                                                                                                                                                                                                                                                               
```

## 5. 实践问题与解决方案

### 问题1：Service被系统回收
解决方案：使用startForeground()提升优先级

### 问题2：跨进程通信失败
解决方案：检查AIDL文件包名是否一致

## 6. 面试常见问题

1. Q: Service生命周期？
A: onCreate() → onStartCommand()/onBind() → onDestroy()

2. Q: 如何保证Service不被杀死？
A: 使用前台服务 + START_STICKY

3. Q: Service和IntentService的区别？
A: 
- Service默认在主线程运行
- IntentService自动创建工作线程
- IntentService任务执行完后自动停止

4. Q: 如何实现跨进程Service通信？
A:
- 使用AIDL定义接口
- 服务端实现Stub
- 客户端绑定服务获取Proxy

5. Q: 前台服务的通知如何更新？
A: 使用NotificationManager.notify()更新同一ID的通知

6. Q: Service的启动方式有哪些区别？
A:
- startService(): 长期后台运行
- bindService(): 与其他组件绑定生命周期

7. Q: 如何检测Service是否正在运行？
A: 使用ActivityManager.getRunningServices()

8. Q: Service的onStartCommand返回值含义？
A:
- START_STICKY: 系统会重建服务
- START_NOT_STICKY: 系统不会重建服务
- START_REDELIVER_INTENT: 重建服务并重发Intent
