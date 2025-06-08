# Java AbstractSequentialList详解

## 继承关系
AbstractSequentialList是List接口的抽象实现类，位于java.util包中。

``` 
AbstractList ← AbstractSequentialList
```

## 设计目的
1. 为顺序访问数据结构(如链表)提供骨架实现
2. 减少基于链表实现的List的工作量
3. 规范顺序访问List的实现方式
4. 实现通用的迭代器相关方法

## 主要特点
1. 适合顺序访问而非随机访问的数据结构
2. 实现了基于ListIterator的核心方法
3. 是LinkedList的父类
4. 相比AbstractList，更适合链表类结构

## 核心方法实现及源码解析

1. `get(int index)`: 通过ListIterator遍历实现
```java
public E get(int index) {
    try {
        return listIterator(index).next();
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}
```

2. `set(int index, E element)`: 通过ListIterator实现
```java
public E set(int index, E element) {
    try {
        ListIterator<E> e = listIterator(index);
        E oldVal = e.next();
        e.set(element);
        return oldVal;
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}
```

3. `add(int index, E element)`: 通过ListIterator实现
```java
public void add(int index, E element) {
    try {
        listIterator(index).add(element);
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}
```

4. `remove(int index)`: 通过ListIterator实现
```java
public E remove(int index) {
    try {
        ListIterator<E> e = listIterator(index);
        E outCast = e.next();
        e.remove();
        return outCast;
    } catch (NoSuchElementException exc) {
        throw new IndexOutOfBoundsException("Index: "+index);
    }
}
```

5. `listIterator(int index)`: 抽象方法，需要子类实现
```java
public abstract ListIterator<E> listIterator(int index);
```

6. `iterator()`: 基于listIterator的实现
```java
public Iterator<E> iterator() {
    return listIterator();
}
```

实现特点：
1. 所有随机访问操作都转换为顺序访问
2. 通过try-catch处理边界情况
3. 充分利用ListIterator的功能
4. 体现了适配器模式的思想

## 主要子类
1. LinkedList: 双向链表实现
2. 自定义顺序访问List通常继承此类

## 实践中遇到的问题及解决方案
1. 问题：随机访问性能差
   解决方案：不适合需要频繁随机访问的场景，改用ArrayList

2. 问题：并发修改异常
   解决方案：使用线程安全实现或加锁

## 面试常见问题
1. Q: AbstractSequentialList和AbstractList的区别？
   A: AbstractSequentialList针对顺序访问优化，AbstractList更适合随机访问

2. Q: 为什么LinkedList继承AbstractSequentialList？
   A: 因为链表结构适合顺序访问，AbstractSequentialList提供了更适合的实现骨架

3. Q: AbstractSequentialList的优缺点？
   A: 
   优点：简化顺序访问List的实现
   缺点：随机访问性能差
