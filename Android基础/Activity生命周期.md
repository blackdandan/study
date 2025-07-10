# Activity生命周期（onSaveInstanceState、launchMode）

## 完整生命周期方法
### 核心方法
1. onCreate() - Activity创建时调用
2. onStart() - Activity可见时调用
3. onResume() - Activity获取焦点时调用
4. onPause() - Activity失去焦点时调用
5. onStop() - Activity不可见时调用
6. onDestroy() - Activity销毁时调用
7. onRestart() - Activity从停止状态恢复时调用

### 方法调用顺序
```
启动Activity: onCreate → onStart → onResume
返回键退出: onPause → onStop → onDestroy
Home键退出: onPause → onStop
重新进入: onRestart → onStart → onResume
```

## 生命周期图示
```
Activity生命周期状态机：
┌─────────────┐       ┌─────────────┐
│  创建中     │──────▶│ 启动中      │
└─────────────┘       └─────────────┘
      ▲                     │
      │                     ▼
┌─────────────┐       ┌─────────────┐
│  销毁中     │◀──────│ 运行中      │
└─────────────┘       └─────────────┘
      ▲                     │
      │                     ▼
┌─────────────┐       ┌─────────────┐
│  停止中     │◀──────│ 暂停中      │
└─────────────┘       └─────────────┘
```

## onSaveInstanceState
### 作用
在Activity可能被系统销毁前保存临时数据

### 使用场景
1. 屏幕旋转
2. 内存不足时后台Activity被回收
3. 多窗口模式下切换

### 代码示例
```java
@Override
protected void onSaveInstanceState(Bundle outState) {
    super.onSaveInstanceState(outState);
    outState.putString("KEY", "需要保存的数据");
}
```

## launchMode
### 四种启动模式
1. standard - 默认模式，每次启动创建新实例
2. singleTop - 栈顶复用
3. singleTask - 栈内复用
4. singleInstance - 单独任务栈

### 对比表
| 模式          | 新实例 | 任务栈       | 适用场景           |
|---------------|--------|--------------|--------------------|
| standard      | 总是   | 当前栈       | 常规Activity       |
| singleTop     | 可能   | 当前栈       | 通知点击处理       |
| singleTask    | 可能   | 可能新建栈   | 应用主界面         |
| singleInstance| 总是   | 新建独立栈   | 系统级单独功能     |

### 代码中使用启动模式
#### 1. 在AndroidManifest.xml中声明
```xml
<activity 
    android:name=".MainActivity"
    android:launchMode="singleTask">
</activity>
```

#### 2. 通过Intent Flags设置
```java
Intent intent = new Intent(this, MainActivity.class);
// 设置singleTop模式
intent.addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP);
// 或者设置singleTask模式
// intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
startActivity(intent);
```

#### 3. 创建新实例(NewInstance)
- standard模式默认每次都会创建新实例
- 其他模式下也可以通过以下方式强制创建新实例：
```java
Intent intent = new Intent(this, MainActivity.class);
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK | 
                Intent.FLAG_ACTIVITY_MULTIPLE_TASK);
startActivity(intent);
```

#### 4. taskAffinity详解
- **概念**：Activity的归属任务栈标识，默认与包名相同
- **作用**：
  - 决定singleTask Activity的宿主任务栈
  - 影响FLAG_ACTIVITY_NEW_TASK的行为
- **代码示例**：
```xml
<activity 
    android:name=".MainActivity"
    android:launchMode="singleTask"
    android:taskAffinity="com.example.customtask"/>
```
- **使用场景**：
  - 将不同模块的Activity分配到独立任务栈
  - 实现类似浏览器的多标签页效果
  - 分离后台服务Activity和前台UI Activity

#### 5. 注意事项
- Manifest声明是静态设置，适用于整个应用生命周期
- Intent Flags是动态设置，优先级高于Manifest声明
- singleTask和FLAG_ACTIVITY_NEW_TASK通常配合使用
- 使用FLAG_ACTIVITY_CLEAR_TOP可以清除栈顶Activity
- FLAG_ACTIVITY_MULTIPLE_TASK可以强制创建新实例
- taskAffinity需与launchMode配合使用才有效果

## 实践中遇到的问题
### 问题1：onSaveInstanceState未被调用
**解决方案**：
- 确认是在用户主动退出(如按返回键)时不会调用
- 只在系统回收时调用
- 测试时可用开发者选项"不保留活动"模拟

### 问题2：launchMode不生效
**解决方案**：
- 检查AndroidManifest.xml中的配置
- 确保没有通过Intent.FLAG_ACTIVITY_*标记覆盖
- 清除任务栈历史后测试

## 面试常见问题
### Q1：onSaveInstanceState和onPause的区别？
**回答要点**：
- onPause总是会被调用，是生命周期的一部分
- onSaveInstanceState只在可能被系统销毁时调用
- onPause适合持久化保存，onSaveInstanceState适合临时状态

### Q2：singleTask和singleInstance的区别？
**回答要点**：
- singleTask允许其他Activity进入它的任务栈
- singleInstance的任务栈只允许它自己
- singleInstance更适合完全独立的功能(如来电界面)

### Q3：Activity生命周期方法的执行顺序？
**回答要点**：
- 完整顺序：onCreate → onStart → onResume → onPause → onStop → onDestroy
- 部分场景：onRestart会在从后台返回时调用(onRestart → onStart → onResume)
- 横竖屏切换：会销毁重建(onPause → onStop → onDestroy → onCreate → onStart → onResume)

### Q4：onCreate和onResume的区别？
**回答要点**：
- onCreate只调用一次(创建时)，onResume可能多次调用(从后台返回时)
- onCreate适合初始化操作，onResume适合恢复UI状态
- onCreate有Bundle参数可恢复数据，onResume没有

### Q5：什么情况下Activity会被销毁重建？
**回答要点**：
- 配置改变(屏幕旋转、语言切换等)
- 内存不足时后台Activity被回收
- 开发者手动调用recreate()
- 多窗口模式切换

### Q6：如何避免Activity因配置改变而重建？
**回答要点**：
- 使用android:configChanges属性声明处理配置变更
- 重写onConfigurationChanged()方法手动处理
- 适用于简单的UI调整，复杂情况仍需重建

### Q7：onSaveInstanceState和onRestoreInstanceState的区别？
**回答要点**：
- onSaveInstanceState在可能被销毁前调用
- onRestoreInstanceState在重建后恢复数据时调用
- 两者都接收Bundle参数，但调用时机不同

### Q8：Activity A启动Activity B时的生命周期变化？
**回答要点**：
- A.onPause() → B.onCreate() → B.onStart() → B.onResume() → A.onStop()
- 返回时：B.onPause() → A.onRestart() → A.onStart() → A.onResume() → B.onStop() → B.onDestroy()

### Q9：后台Activity被回收后如何恢复数据？
**回答要点**：
- 使用onSaveInstanceState保存临时数据
- 在onCreate或onRestoreInstanceState中恢复
- 持久化数据应使用数据库或SharedPreferences

### Q10：Activity的onNewIntent方法何时调用？
**回答要点**：
- singleTop或singleTask模式下已有实例被复用时
- 可用于处理新的Intent数据
- 需要手动调用setIntent更新当前Intent
