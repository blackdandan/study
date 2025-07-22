# Android架构组件详解

## Android架构演进史

### 1. MVC时代 (2010年前)
```
Activity ──控制─▶ Model
   │
   └─更新─▶ View
```
问题：Activity过于臃肿，承担了Controller和View双重角色

### 2. MVP时代 (2010-2017)
```
Activity(View) ──委托─▶ Presenter ──操作─▶ Model
```
改进：分离视图逻辑，但Presenter仍持有View引用

### 3. MVVM + 组件化 (2017-至今)
```
ViewModel ──提供─▶ LiveData ──绑定─▶ View
   │
   └─操作─▶ Repository ──访问─▶ 本地/远程数据
```
特点：
- 数据驱动UI
- 生命周期感知
- 单向数据流

### 架构组件出现背景
- 解决生命周期管理难题
- 统一应用架构指南
- 提升代码可测试性

## 1. ViewModel

### 基本概念
ViewModel是设计用来以生命周期意识的方式存储和管理UI相关数据的组件。

### 核心特点
- 生命周期感知：与Activity/Fragment生命周期关联
- 数据持久化：配置更改时保留数据
- UI数据分离：将业务逻辑与UI分离

### 典型用法
```kotlin
class MyViewModel : ViewModel() {
    private val _data = MutableLiveData<String>()
    val data: LiveData<String> = _data
    
    fun loadData() {
        _data.value = "Loaded data"
    }
}
```

### 实践问题与解决方案
**问题1：ViewModel中持有Context导致内存泄漏**
解决方案：使用AndroidViewModel获取Application Context

**问题2：ViewModel数据在屏幕旋转后丢失**
解决方案：确保使用ViewModelProvider正确初始化

### 面试问题
1. ViewModel与onSaveInstanceState有什么区别？
   - ViewModel适合存储大量数据，而onSaveInstanceState适合存储简单数据
   - ViewModel在进程被杀时会被销毁，而onSaveInstanceState会保留

2. ViewModel如何实现生命周期感知？
   通过LifecycleOwner和ViewModelStore机制

### 相关设计模式
- MVVM模式
- 观察者模式

---

## 2. LiveData

### 基本概念
LiveData是一个可观察的数据持有者类，具有生命周期感知能力。

### 核心特点
- 数据更新自动通知
- 自动取消订阅
- 避免内存泄漏

### 数据流示意图
```
ViewModel ──持有─▶ LiveData ──观察─▶ Activity/Fragment
```

### 典型用法
```kotlin
viewModel.data.observe(this) { data ->
    // 更新UI
}
```

### 实践问题与解决方案
**问题1：LiveData数据倒灌详解**

现象：当新观察者订阅时，会立即收到最后一次的数据更新

原因：LiveData的"粘性"特性设计

示例场景：
1. 从Activity A跳转到Activity B
2. Activity B订阅LiveData
3. 立即收到了Activity A触发的最新数据（可能不需要）

解决方案：
1. SingleLiveEvent方案：
```kotlin
class SingleLiveEvent<T> : MutableLiveData<T>() {
    private val pending = AtomicBoolean(false)

    override fun observe(owner: LifecycleOwner, observer: Observer<in T>) {
        super.observe(owner) { t ->
            if (pending.compareAndSet(true, false)) {
                observer.onChanged(t)
            }
        }
    }

    override fun setValue(t: T) {
        pending.set(true)
        super.setValue(t)
    }
}
```

2. Event Wrapper方案：
```kotlin
class Event<T>(private val content: T) {
    var hasBeenHandled = false
        private set

    fun getContentIfNotHandled(): T? {
        return if (hasBeenHandled) null else {
            hasBeenHandled = true
            content
        }
    }
}
```

**问题2：LiveData在后台线程更新**
解决方案：使用postValue()方法

### LiveData转换操作

#### Transformations工具类
提供两个核心方法：
1. map：数据转换
```kotlin
val userLiveData: LiveData<User> = ...
val userNameLiveData: LiveData<String> = 
    Transformations.map(userLiveData) { user ->
        user.name
    }
```

2. switchMap：LiveData切换
```kotlin
fun getUser(id: String): LiveData<User> {
    return Transformations.switchMap(userIdLiveData) { id ->
        repository.getUser(id)
    }
}
```

原理：内部使用MediatorLiveData实现观察链

### 面试问题
1. LiveData与RxJava有什么区别？
   - LiveData更轻量，专为Android设计
   - RxJava功能更强大，学习曲线更陡
   - LiveData自动处理生命周期，RxJava需要手动dispose

2. Transformations的实现原理？
   基于MediatorLiveData构建观察链，每个转换都会创建一个新的LiveData

### 相关设计模式
- 观察者模式
- 发布-订阅模式

---

## 3. Room

### 基本概念
Room是在SQLite上提供了一个抽象层，简化数据库操作。

### 核心组件
```
Database ──包含─▶ Entity ──对应─▶ 表结构
         ──包含─▶ DAO    ──定义─▶ 数据库操作
```

### 典型用法
```kotlin
@Database(entities = [User::class], version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}

@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    fun getAll(): LiveData<List<User>>
}
```

### 实践问题与解决方案
**问题1：数据库升级导致数据丢失详解**

典型场景：
1. 新增表字段
2. 修改表结构
3. 表重命名
4. 新增表

示例：从v1升级到v2时新增字段
```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE user ADD COLUMN age INTEGER NOT NULL DEFAULT 0")
    }
}

Room.databaseBuilder(..., AppDatabase::class.java, "app.db")
    .addMigrations(MIGRATION_1_2)
    .build()
```

预防措施：
1. 始终提供Migration实现
2. 测试所有版本升级路径
3. 使用exportSchema生成版本历史

**问题2：主线程数据库操作导致ANR**
解决方案：使用协程或RxJava进行异步操作

### 面试问题
1. Room与GreenDAO有什么区别？
   - Room是官方组件，与LiveData集成更好
   - GreenDAO性能更好但需要更多配置

2. 如何实现Room数据库迁移？
   使用addMigrations()方法并提供Migration实现

### Room底层实现

Room架构：
```
Room ──生成─▶ 实现类
   │
   └─基于─▶ SQLiteOpenHelper
      │
      └─封装─▶ SQLiteDatabase
```

关键点：
1. 编译时生成_Impl类
2. 运行时通过SupportSQLiteOpenHelper.Factory创建实例
3. 最终仍使用Android原生的SQLite API

性能优化：
1. 使用预编译语句
2. 事务批处理
3. 类型安全查询验证

### 相关设计模式
- DAO模式
- 仓库模式
- 门面模式（封装SQLiteOpenHelper）

---

## 4. Navigation

### 基本概念
Navigation组件用于简化应用内导航的实现。

### 核心组件
```
NavGraph ──包含─▶ Destination ──表示─▶ 目标Fragment/Activity
         ──包含─▶ Action      ──定义─▶ 导航动作
```

### 典型用法
```xml
<navigation>
    <fragment 
        android:id="@+id/fragmentA"
        android:name="com.example.FragmentA">
        <action 
            android:id="@+id/action_to_b"
            app:destination="@id/fragmentB"/>
    </fragment>
</navigation>
```

### 实践问题与解决方案
**问题1：导航参数类型安全**
解决方案：使用Safe Args插件

**问题2：深层链接处理**
解决方案：配置nav-graph中的deepLink属性

### 面试问题
1. Navigation组件如何实现Fragment间通信？
   通过ViewModel共享或传递Bundle参数

2. 如何处理导航回退栈？
   使用popUpTo和popUpToInclusive属性

### 相关设计模式
- 单Activity架构模式
- 中介者模式

---

## 组件关系图

```
ViewModel ──提供数据─▶ LiveData ──观察─▶ UI
   │
   └─使用─▶ Room ──持久化─▶ 数据库
   
Navigation ──控制─▶ 界面跳转
```

## 最佳实践组合
1. ViewModel + LiveData：UI数据管理
2. ViewModel + Room：数据持久化
3. Navigation + ViewModel：界面间数据共享
