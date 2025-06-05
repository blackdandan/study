# ArrayList详解

ArrayList是Java集合框架中最常用的List实现类，基于动态数组实现。

## 1. 类继承关系

```
                   ┌────────────────────────┐
                   │        List<E>        │
                   └──────────┬─────────────┘
                              │
                   ┌──────────▼─────────────┐
                   │     AbstractList<E>    │
                   └─────┬───────────┬──────┘
                         │           │
              ┌──────────▼───┐   ┌───▼──────────┐
              │ ArrayList<E> │   │  Vector<E>   │
              └──────────────┘   └──────────────┘
```

## 2. 核心特性

- **动态扩容**：默认初始容量10，扩容时增加50%
- **随机访问**：通过索引访问元素时间复杂度O(1)
- **非线程安全**：多线程环境下需要外部同步
- **允许null元素**：可以存储null值
- **快速失败(Fail-Fast)**：迭代时检测到并发修改会抛出ConcurrentModificationException

## 3. 内部结构

```java
transient Object[] elementData;  // 存储元素的数组
private int size;                // 实际元素数量
private static final int DEFAULT_CAPACITY = 10;
```

## 4. 常用方法

| 方法 | 时间复杂度 | 描述 |
|------|------------|------|
| add(E e) | O(1) 平均 | 添加元素到末尾 |
| add(int index, E e) | O(n) | 在指定位置插入元素 |
| get(int index) | O(1) | 获取指定位置元素 |
| remove(int index) | O(n) | 删除指定位置元素 |
| remove(Object o) | O(n) | 删除指定元素 |
| contains(Object o) | O(n) | 检查是否包含元素 |
| size() | O(1) | 返回元素数量 |
| trimToSize() | O(n) | 缩减数组容量到实际大小 |

## 5. 性能分析

- **优点**：
  - 随机访问速度快
  - 尾部添加元素效率高
  - 内存占用比LinkedList少

- **缺点**：
  - 中间插入/删除元素效率低
  - 扩容时会产生额外开销：
    - 创建新数组：需要分配更大的内存空间
    - 数组拷贝：使用System.arraycopy()复制原数组元素
    - 旧数组GC：旧数组成为垃圾等待回收
    - 扩容时机：添加元素时发现容量不足才会触发
    - 实践中的坑：
      - 频繁扩容导致性能下降（特别是大数据量时）
      - 预估容量不准确导致内存浪费
      - 扩容期间可能引发OOM异常

## 6. 使用示例

```java
// 创建ArrayList
List<String> list = new ArrayList<>();

// 添加元素
list.add("Java");
list.add("Python");
list.add(1, "C++");  // 在索引1处插入

// 访问元素
String first = list.get(0);

// 遍历
for (String lang : list) {
    System.out.println(lang);
}

// 删除元素
list.remove("Python");
list.remove(0);
```

## 7. 线程安全替代方案

- 使用Collections.synchronizedList包装：
  ```java
  List<String> syncList = Collections.synchronizedList(new ArrayList<>());
  ```

- 使用CopyOnWriteArrayList(适合读多写少场景)

## 8. 最佳实践

1. 预估容量避免频繁扩容：
   ```java
   new ArrayList<>(100);  // 指定初始容量
   ```

2. 批量操作使用addAll：
   ```java
   list.addAll(anotherList);
   ```

3. 遍历时避免结构性修改

## 9. 面试常见问题

1. **ArrayList和LinkedList的区别？**
   - 底层结构：数组 vs 双向链表
   - 访问效率：O(1) vs O(n)
   - 插入删除：尾部O(1)，中间O(n) vs 头尾O(1)，中间O(n)
   - 内存占用：连续内存 vs 节点额外内存

2. **ArrayList的扩容机制？**
   - 默认初始容量10
   - 扩容公式：newCapacity = oldCapacity + (oldCapacity >> 1)
   - 最大容量Integer.MAX_VALUE - 8：
     ```java
     // JDK源码片段
     private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
     
     private void grow(int minCapacity) {
         int oldCapacity = elementData.length;
         int newCapacity = oldCapacity + (oldCapacity >> 1);
         if (newCapacity - minCapacity < 0)
             newCapacity = minCapacity;
         if (newCapacity - MAX_ARRAY_SIZE > 0)
             newCapacity = hugeCapacity(minCapacity);
         elementData = Arrays.copyOf(elementData, newCapacity);
     }
     ```
     - 减8是因为某些JVM实现会在数组头部存储元数据
     - 超过此限制会抛出OutOfMemoryError
   - 扩容时数组拷贝使用System.arraycopy()

3. **线程安全相关**
   - **ArrayList线程安全**：
     - 非线程安全
     - 解决方案：
       ```java
       // 方法1：Collections.synchronizedList
       List<String> syncList = Collections.synchronizedList(new ArrayList<>());
       
       // 方法2：CopyOnWriteArrayList
       List<String> cowList = new CopyOnWriteArrayList<>();
       ```
     - 需要手动同步的场景：
       ```java
       synchronized(list) {
           list.add(item);
       }
       ```
   
   - **Vector的区别**：
     ```java
     // Vector的同步方法示例
     public synchronized boolean add(E e) {
         modCount++;
         ensureCapacityHelper(elementCount + 1);
         elementData[elementCount++] = e;
         return true;
     }
     ```
     - Java 1.2后被标记为"legacy"，原因：
       - 同步粒度太粗(方法级)，性能差
       - 现代应用更倾向于细粒度同步控制
     - 推荐替代方案：
       - ArrayList + 外部同步
       - CopyOnWriteArrayList
     - 扩容策略不同：Vector增长100%，ArrayList增长50%

4. **快速失败(Fail-Fast)机制？**
   ```java
   // ArrayList中的modCount声明
   protected transient int modCount = 0;
   
   // 迭代器检查实现
   final void checkForComodification() {
       if (modCount != expectedModCount)
           throw new ConcurrentModificationException();
   }
   ```
   - 实现原理：
     - modCount记录结构修改次数(add/remove等操作会递增)
     - 创建迭代器时记录expectedModCount=modCount
     - 每次迭代前调用checkForComodification()检查
   - 注意：
     - 非原子操作，不能完全保证线程安全
     - 仅用于检测单线程下的并发修改

5. **ArrayList的toArray()方法？**
   ```java
   // toArray(T[] a)源码关键部分
   if (a.length < size)
       return (T[]) Arrays.copyOf(elementData, size, a.getClass());
   System.arraycopy(elementData, 0, a, 0, size);
   if (a.length > size)
       a[size] = null;
   ```
   - 行为说明：
     - 数组大小充足时：拷贝元素到该数组，剩余位置保持原内容
     - 数组大小不足时：创建新数组
     - 最佳实践：`list.toArray(new String[0])`

6. **ArrayList的subList()方法注意事项？**
   ```java
   // SubList构造方法
   SubList(AbstractList<E> parent, int offset, int fromIndex, int toIndex) {
       this.parent = parent;
       this.parentOffset = fromIndex;
       this.modCount = ArrayList.this.modCount;
       // ...
   }
   ```
   - 实现原理：
     - 基于原列表的视图(存储偏移量，不拷贝数据)
     - 共享原列表的modCount
   - 异常原因：
     - 任何结构性修改都会改变原列表modCount
     - 子列表操作时会检查modCount是否一致
   - 解决方案：
     ```java
     new ArrayList<>(list.subList(from, to))
     ```
