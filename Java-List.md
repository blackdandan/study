# Java List接口详解

## 继承关系
List接口继承自Collection接口，是Java集合框架中的有序集合。

``` 
Collection ← List
```

## List接口新增方法
相比Collection接口，List增加了以下主要方法：

1. 位置访问操作
- `E get(int index)`
- `E set(int index, E element)`
- `void add(int index, E element)`
- `E remove(int index)`

2. 搜索操作
- `int indexOf(Object o)`
- `int lastIndexOf(Object o)`

3. 列表迭代器
- `ListIterator<E> listIterator()`
- `ListIterator<E> listIterator(int index)`

4. 范围操作
- `List<E> subList(int fromIndex, int toIndex)`

## List主要实现类
1. ArrayList
   - 基于动态数组实现
   - 随机访问快，插入删除慢
   - 非线程安全

2. LinkedList
   - 基于双向链表实现
   - 插入删除快，随机访问慢
   - 实现了Deque接口

3. Vector
   - 线程安全的动态数组
   - 方法同步，性能较低
   - 有子类Stack

4. CopyOnWriteArrayList
   - 线程安全变体
   - 写时复制机制
   - 适合读多写少场景

## 实践中遇到的问题及解决方案
1. 问题：ArrayList在并发环境下修改会抛出ConcurrentModificationException
   解决方案：使用Collections.synchronizedList包装或改用CopyOnWriteArrayList

2. 问题：LinkedList的随机访问性能差
   解决方案：对于需要频繁随机访问的场景，改用ArrayList

## 面试常见问题
1. Q: ArrayList和LinkedList的区别？
   A: 主要区别在于底层实现(数组vs链表)和性能特点(随机访问快vs插入删除快)

2. Q: Vector为什么被弃用？
   A: 虽然线程安全但性能差，推荐使用Collections.synchronizedList或CopyOnWriteArrayList替代

3. Q: List的subList方法使用时要注意什么？
   A: subList是原列表的视图，对子列表的修改会影响原列表，且原列表结构变化会导致子列表操作抛出异常
