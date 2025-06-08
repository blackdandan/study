# Java LinkedList详解

## 继承关系
LinkedList是List和Deque接口的双向链表实现，位于java.util包中。

``` 
AbstractSequentialList ← LinkedList
(实现List和Deque接口)
```

## 数据结构
1. 基于双向链表实现
2. 每个节点包含：
```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```
3. 维护first和last指针

## 核心方法实现

### 添加元素
```java
// 头部添加
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;
}

// 尾部添加
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
}
```

### 删除元素
```java
E unlink(Node<E> x) {
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    return element;
}
```

## 与ArrayList对比

| 特性        | LinkedList | ArrayList |
|------------|------------|-----------|
| 底层结构     | 双向链表    | 动态数组   |
| 随机访问性能 | O(n)       | O(1)      |
| 插入删除性能 | O(1)       | O(n)      |
| 内存占用     | 更高(节点开销)| 更低      |

## 使用场景
1. 频繁插入删除操作
2. 需要实现队列/双端队列
3. 不需要频繁随机访问

## 实践中遇到的问题
1. 问题：foreach循环中删除元素抛ConcurrentModificationException
   解决方案：使用Iterator的remove()方法

2. 问题：随机访问性能差
   解决方案：改用ArrayList或使用ListIterator

## 面试常见问题
1. Q: LinkedList和ArrayList的区别？
   A: 主要区别在数据结构(链表vs数组)和性能特点

2. Q: LinkedList为什么能实现Deque接口？
   A: 因为双向链表天然支持两端操作

3. Q: LinkedList是线程安全的吗？
   A: 不是，需要外部同步或使用Collections.synchronizedList

4. Q: ArrayList和LinkedList的remove()方法性能比较？
   A: 
   - ArrayList的remove():
     * 平均时间复杂度O(n)
     * 需要移动后续元素
     * 最坏情况(删除第一个元素)需要移动n-1个元素
   - LinkedList的remove():
     * 已知节点引用时O(1)
     * 按索引查找时O(n)(需要先遍历找到节点)
     * 删除操作本身只需修改相邻节点的引用
   结论：
   - 随机位置删除：LinkedList性能更好
   - 尾部删除：两者性能接近
   - 需要根据具体使用场景选择
