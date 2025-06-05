# Collection接口和AbstractCollection类

## 1. Collection接口

### 接口层次结构
```
                   ┌────────────────────────┐
                   │     Collection<E>      │
                   └─────┬───────────┬──────┘
                         │           │
              ┌──────────▼───┐   ┌───▼──────────┐
              │   List<E>    │   │   Set<E>     │
              └──────────────┘   └──────────────┘
```

### 核心方法
#### 基本操作
- `int size()`: 返回集合中元素数量
- `boolean isEmpty()`: 判断集合是否为空
- `boolean contains(Object o)`: 判断是否包含指定元素
- `boolean add(E e)`: 添加元素到集合
- `boolean remove(Object o)`: 移除指定元素
- `Iterator<E> iterator()`: 返回迭代器

#### 批量操作
- `boolean containsAll(Collection<?> c)`: 判断是否包含指定集合所有元素
- `boolean addAll(Collection<? extends E> c)`: 添加指定集合所有元素
- `boolean removeAll(Collection<?> c)`: 移除与指定集合相同的元素
- `boolean retainAll(Collection<?> c)`: 仅保留与指定集合相同的元素
- `void clear()`: 清空集合

#### 数组转换
- `Object[] toArray()`: 返回包含所有元素的数组
- `<T> T[] toArray(T[] a)`: 返回指定类型的数组

### Collection接口特点
1. 是集合框架的根接口
2. 不直接提供实现，通过子接口实现
3. 允许重复元素(具体由实现类决定)
4. 元素顺序取决于具体实现

## 2. AbstractCollection类

### 类定义
```java
public abstract class AbstractCollection<E> implements Collection<E> {
    // 骨架实现
}
```

### 关键默认实现

#### 1. isEmpty()
```java
public boolean isEmpty() {
    return size() == 0;
}
```

#### 2. contains(Object o)
```java
public boolean contains(Object o) {
    Iterator<E> it = iterator();
    if (o == null) {
        while (it.hasNext())
            if (it.next() == null) return true;
    } else {
        while (it.hasNext())
            if (o.equals(it.next())) return true;
    }
    return false;
}
```

#### 3. toArray()
```java
public Object[] toArray() {
    Object[] r = new Object[size()];
    Iterator<E> it = iterator();
    for (int i = 0; i < r.length; i++) {
        if (!it.hasNext()) 
            return Arrays.copyOf(r, i);
        r[i] = it.next();
    }
    return it.hasNext() ? finishToArray(r, it) : r;
}
```

### 设计特点
1. 基于迭代器实现大部分方法
2. 需要子类实现iterator()和size()方法
3. 提供了合理的默认实现
4. 部分方法标记为可选操作

## 3. 面试问题与答案

### Collection相关问题

1. **Collection接口的主要作用是什么？**
   - 提供统一的集合操作接口
   - 定义集合框架的基本行为规范
   - 实现多态性，允许以统一方式处理不同集合类型

2. **Collection和Collections有什么区别？**
   - Collection是集合接口，定义集合基本操作
   - Collections是工具类，提供操作集合的静态方法
   - Collections包含排序、查找、同步化等方法

3. **为什么Collection接口没有get(int index)方法？**
   - 不是所有集合都有索引概念(如Set)
   - 随机访问功能由子接口List提供
   - 保持接口最小化原则

4. **Collection接口的contains()方法如何判断元素相等？**
   - 使用equals()方法比较
   - 对于null元素，使用==比较
   - 依赖对象的equals()实现

### AbstractCollection相关问题

1. **AbstractCollection的设计目的是什么？**
   - 提供Collection接口的骨架实现
   - 减少实现Collection接口的工作量
   - 为具体集合类提供通用实现

2. **AbstractCollection必须实现哪些方法？**
   ```java
   public abstract Iterator<E> iterator();
   public abstract int size();
   ```

3. **AbstractCollection如何实现contains()方法？**
   - 通过迭代器遍历所有元素
   - 对null元素特殊处理
   - 时间复杂度O(n)

4. **AbstractCollection的toArray()实现有什么特点？**
   - 先创建size大小的数组
   - 处理集合大小变化的情况
   - 返回数组可能小于集合大小

5. **为什么AbstractCollection的remove()方法效率可能不高？**
   ```java
   // 需要先查找元素再删除
   public boolean remove(Object o) {
       Iterator<E> it = iterator();
       // 遍历查找代码...
       it.remove(); // 找到后删除
   }
   ```

6. **AbstractCollection的toString()实现有什么特点？**
   - 使用迭代器遍历元素
   - 处理循环引用(this)
   - 格式化为[element1, element2]

### 更多实现细节

1. **add()方法的默认实现**
   ```java
   public boolean add(E e) {
       throw new UnsupportedOperationException();
   }
   ```
   - 默认抛出异常，子类需重写
   - 体现可选操作设计

2. **addAll()实现分析**
   ```java
   public boolean addAll(Collection<? extends E> c) {
       boolean modified = false;
       for (E e : c)
           if (add(e)) // 依赖add()实现
               modified = true;
       return modified;
   }
   ```

3. **removeAll实现原理**
   - 使用iterator().remove()
   - 批量删除匹配元素
   - 性能取决于iterator实现

4. **retainAll实现分析**
   - 保留与指定集合相同的元素
   - 通过contains()判断保留条件
   - 同样依赖iterator().remove()

### Collection子接口详解

1. **List接口新增方法**
   ```java
   // 索引相关操作
   E get(int index);
   E set(int index, E element);
   void add(int index, E element);
   E remove(int index);
   int indexOf(Object o);
   int lastIndexOf(Object o);
   ```
   - 原因：提供有序集合的随机访问能力
   - 特点：维护元素插入顺序

2. **Set接口特点**
   - 不新增方法，但强化语义约束
   - 要求元素唯一性(equals()判断)
   - 原因：数学集合概念的实现

3. **Queue接口新增方法**
   ```java
   // 队列特定操作
   boolean offer(E e);
   E poll();
   E peek();
   ```
   - 原因：实现FIFO队列行为
   - 特点：提供安全的获取/移除操作

4. **Deque接口新增方法**
   ```java
   // 双端队列操作
   void addFirst(E e);
   void addLast(E e);
   E getFirst();
   E getLast();
   ```
   - 原因：支持两端操作的队列
   - 特点：同时具备栈和队列特性

### 设计思想分析
1. **接口隔离原则**
   - 最小化接口职责
   - 通过子接口扩展功能

2. **方法设计考量**
   - 通用功能放在Collection
   - 特殊功能由子接口提供

3. **演化过程**
   - Java 1.2: Collection, List, Set
   - Java 1.5: Queue
   - Java 1.6: Deque
