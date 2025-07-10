# Java集合-Collections接口

`Collections`是Java集合框架中的一个工具类，提供了一系列静态方法来操作或返回集合。

## 主要方法及用途

### 排序相关
- `sort(List<T> list)`: 根据元素的自然顺序对指定列表进行升序排序
  - 默认使用TimSort算法(改进的归并排序)
  - 时间复杂度: O(n log n)
  - 空间复杂度: O(n)
  - 稳定排序算法
- `sort(List<T> list, Comparator<? super T> c)`: 根据指定比较器对列表进行排序
  - Android平台实现细节：
    - 针对API级别>25直接调用list.sort(c)
    - 针对API级别≤25的特殊兼容处理：
      1. 如果是ArrayList，直接对其内部数组进行排序
      2. 其他List类型转换为数组排序后再写回
    - 目的：保持旧版本Android的行为兼容性
- `reverse(List<?> list)`: 反转列表中元素的顺序

### 查找相关
- `binarySearch(List<? extends Comparable<? super T>> list, T key)`: 使用二分搜索法搜索指定列表中的指定对象
- `binarySearch(List<? extends T> list, T key, Comparator<? super T> c)`: 使用指定比较器进行二分查找

### 修改操作
- `swap(List<?> list, int i, int j)`: 交换列表中指定位置的元素
- `fill(List<? super T> list, T obj)`: 使用指定元素替换列表中的所有元素
- `copy(List<? super T> dest, List<? extends T> src)`: 将所有元素从一个列表复制到另一个列表

### 不可变集合
- `emptyList()`: 返回空的不可变列表
- `singletonList(T o)`: 返回只包含指定对象的不可变列表
- `unmodifiableList(List<? extends T> list)`: 返回指定列表的不可修改视图

### 同步控制
- `synchronizedList(List<T> list)`: 返回由指定列表支持的同步(线程安全)列表
- `synchronizedSet(Set<T> s)`: 返回由指定集合支持的同步集合
- `synchronizedMap(Map<K,V> m)`: 返回由指定映射支持的同步映射

### 其他实用方法
- `shuffle(List<?> list)`: 使用默认随机源对列表进行置换
- `max(Collection<? extends T> coll)`: 根据自然顺序返回集合中的最大元素
- `min(Collection<? extends T> coll)`: 根据自然顺序返回集合中的最小元素
- `frequency(Collection<?> c, Object o)`: 返回指定集合中等于指定对象的元素数
