# Java AbstractList详解

## 继承关系
AbstractList是List接口的抽象实现类，位于java.util包中。

``` 
AbstractCollection ← AbstractList
```

## 设计目的
1. 提供List接口的默认实现，减少子类工作量
2. 实现通用方法如iterator()和listIterator()
3. 定义骨架结构，规范子类实现
4. 实现fail-fast机制，提供并发修改检测

虽然子类需要重写核心方法，但AbstractList仍然提供了：
- 迭代器实现
- 通用算法实现
- 结构规范约束
- 错误检测机制

## 主要特点
1. 提供了List接口的骨架实现
2. 实现了随机访问数据存储结构（如数组）
3. 实现了iterator()和listIterator()方法
4. 是ArrayList和Vector的父类

## 核心方法实现及源码解析

1. `iterator()`: 返回Iterator实现
```java
public Iterator<E> iterator() {
    return new Itr();
}
```

2. `listIterator()`: 返回ListIterator实现
```java
public ListIterator<E> listIterator() {
    return listIterator(0);
}

public ListIterator<E> listIterator(final int index) {
    rangeCheckForAdd(index);
    return new ListItr(index);
}
```

3. `add(int index, E element)`: 抽象方法，需要子类实现
```java
public abstract void add(int index, E element);
```

4. `get(int index)`: 抽象方法，需要子类实现
```java
public abstract E get(int index);
```

5. `size()`: 抽象方法，需要子类实现
```java
public abstract int size();
```

6. `addAll()`: 批量添加实现
```java
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index);
    boolean modified = false;
    for (E e : c) {
        add(index++, e);
        modified = true;
    }
    return modified;
}
```

7. `indexOf()`: 元素查找实现
```java
public int indexOf(Object o) {
    ListIterator<E> it = listIterator();
    if (o==null) {
        while (it.hasNext())
            if (it.next()==null)
                return it.previousIndex();
    } else {
        while (it.hasNext())
            if (o.equals(it.next()))
                return it.previousIndex();
    }
    return -1;
}
```

## 实践中遇到的问题及解决方案
1. 问题：直接使用AbstractList会抛出UnsupportedOperationException
   解决方案：必须继承AbstractList并实现抽象方法

2. 问题：AbstractList的迭代器是fail-fast的
   解决方案：在迭代过程中不要修改集合，或使用CopyOnWriteArrayList

## 面试常见问题
1. Q: AbstractList和AbstractCollection的区别？
   A: AbstractList专门针对List接口实现，提供了基于索引的操作支持

2. Q: 为什么AbstractList是抽象类？
   A: 因为它只实现了部分List接口方法，留关键方法(get/size/add)给子类实现

3. Q: AbstractList的modCount字段有什么作用？
   A: 用于实现fail-fast机制，检测并发修改

4. Q: AbstractList的迭代器是如何实现的？
   A: 通过内部类Itr实现，关键代码如下：
   
```java
private class Itr implements Iterator<E> {
    int cursor = 0;
    int lastRet = -1;
    int expectedModCount = modCount;

    public boolean hasNext() {
        return cursor != size();
    }

    public E next() {
        checkForComodification();
        try {
            E next = get(cursor);
            lastRet = cursor++;
            return next;
        } catch (IndexOutOfBoundsException e) {
            checkForComodification();
            throw new NoSuchElementException();
        }
    }

    public void remove() {
        if (lastRet == -1)
            throw new IllegalStateException();
        checkForComodification();

        try {
            AbstractList.this.remove(lastRet);
            if (lastRet < cursor)
                cursor--;
            lastRet = -1;
            expectedModCount = modCount;
        } catch (IndexOutOfBoundsException e) {
            throw new ConcurrentModificationException();
        }
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

实现特点：
1. cursor记录当前位置，lastRet记录最后访问位置
2. expectedModCount与外部modCount比较实现fail-fast
3. next()方法会检查并发修改并抛出异常
4. remove()操作会回调外部类的remove()方法

5. Q: 所有的List实现类都继承自AbstractList吗？
   A: 不是。虽然大多数标准List实现(如ArrayList、Vector)继承自AbstractList，但也有例外：
   - CopyOnWriteArrayList直接实现List接口
   - 自定义List实现可以选择直接实现List接口
   - 特殊用途的List可能继承其他抽象类
   AbstractList主要提供了通用实现，但不是强制要求。

6. Q: 为什么ArrayList继承AbstractList后还要实现List接口？
   A: 这是Java中的一种良好实践，主要原因包括：
   - 明确表明该类实现了List接口，提高代码可读性
   - 方便IDE和编译器快速识别接口实现关系
   - 在JavaDoc中明确显示接口实现关系
   - 当AbstractList的继承关系改变时，不影响ArrayList的接口契约
   虽然技术上不是必须的，但这是Java集合框架中的常见做法。

7. Q: AbstractList有哪些主要子类？
   A: AbstractList的主要子类包括：
   - ArrayList: 基于动态数组的实现，非线程安全
   - Vector: 线程安全的动态数组实现
   - Stack: 继承自Vector，实现栈数据结构
   - AbstractSequentialList: 为顺序访问数据结构提供的抽象实现
   - 自定义List实现通常也会继承AbstractList
   这些子类都继承了AbstractList提供的通用实现，并根据自身特点实现了核心方法。
