# Java LinkedList详解

## 1. 类继承关系

LinkedList同时实现了List和Deque接口，支持列表和双端队列操作。

```
   继承AbstractSequenceList，  实现Deque借口 
              └──────────────┘   └──────────────┘
```

## 2. 内部数据结构

### 节点结构
```java
private static class Node<E> {
    E item;         // 存储的元素
    Node<E> next;   // 后继节点
    Node<E> prev;   // 前驱节点
    
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

### 链表结构图示
```
[first] ↔ [Node1] ↔ [Node2] ↔ ... ↔ [NodeN] ↔ [last]
   │        │          │                 │
   └─ item1 └─ item2 └─ ...       └─ itemN
```

### 核心字段
```java
transient int size = 0;          // 元素数量
transient Node<E> first;         // 头节点
transient Node<E> last;          // 尾节点
```

## 3. 核心方法实现

### 添加元素
```java
// 头部添加
private void linkFirst(E e) {
    final Node<E> f = first;
    final Node<E> newNode = new Node<>(null, e, f);
    first = newNode;
    if (f == null)
        last = newNode;  // 空链表情况
    else
        f.prev = newNode;
    size++;
    modCount++;  // 结构修改计数
}

// 尾部添加
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;  // 空链表情况
    else
        l.next = newNode;
    size++;
    modCount++;
}

// 指定位置插入
public void add(int index, E element) {
    checkPositionIndex(index);
    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}

void linkBefore(E e, Node<E> succ) {
    final Node<E> pred = succ.prev;
    final Node<E> newNode = new Node<>(pred, e, succ);
    succ.prev = newNode;
    if (pred == null)
        first = newNode;
    else
        pred.next = newNode;
    size++;
    modCount++;
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

## 4. 性能对比分析

| 特性            | LinkedList          | ArrayList           |
|----------------|---------------------|---------------------|
| 底层结构         | 双向链表             | 动态数组             |
| 随机访问(get/set) | O(n)                | O(1)                |
| 头部插入/删除     | O(1)                | O(n)                |
| 尾部插入/删除     | O(1)                | O(1) 平均           |
| 中间插入/删除     | O(n)(查找)+O(1)(操作)| O(n)                |
| 内存占用         | 更高(每个元素额外12字节)| 更低(连续内存)       |
| 迭代性能         | 快速                 | 快速                 |
| 缓存友好性       | 差                   | 好                   |

## 5. 使用场景

### 适合LinkedList的场景：
1. 频繁在头部/中间插入删除元素
2. 需要实现队列/双端队列/栈
3. 不需要频繁随机访问元素
4. 内存充足的应用

### 适合ArrayList的场景：
1. 频繁随机访问元素
2. 主要在尾部添加元素
3. 内存敏感的应用
4. 需要缓存友好的数据结构

## 6. 线程安全方案

### 同步包装
```java
List<String> syncList = Collections.synchronizedList(new LinkedList<>());
```

### 使用场景
1. 读多写少：CopyOnWriteArrayList
2. 高并发队列：ConcurrentLinkedQueue
3. 阻塞队列：LinkedBlockingQueue

## 7. 使用示例

```java
// 作为List使用
List<String> list = new LinkedList<>();
list.add("A");
list.add(0, "B");  // 头部插入
list.remove(1);     // 删除元素

// 作为Deque使用
Deque<String> deque = new LinkedList<>();
deque.offerFirst("First");
deque.offerLast("Last");
String first = deque.pollFirst();

// 作为Queue使用
Queue<String> queue = new LinkedList<>();
queue.offer("Task1");
queue.offer("Task2");
String task = queue.poll();
```

## 8. 实践中遇到的问题

### 问题1：foreach循环中删除元素
```java
// 错误方式 - 会抛ConcurrentModificationException
for (String item : list) {
    if (item.equals("remove")) {
        list.remove(item);  // 结构性修改
    }
}

// 正确方式 - 使用Iterator
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    if (it.next().equals("remove")) {
        it.remove();  // 安全删除
    }
}
```

### 问题2：随机访问性能差
```java
// 避免这样使用 - O(n^2)时间复杂度
for (int i = 0; i < list.size(); i++) {
    System.out.println(list.get(i));
}

// 推荐使用迭代器 - O(n)时间复杂度
for (String item : list) {
    System.out.println(item);
}
```

## 9. 面试常见问题

1. **LinkedList和ArrayList的主要区别？**
   - 底层实现：链表 vs 数组
   - 访问模式：顺序访问 vs 随机访问
   - 内存布局：非连续 vs 连续
   - 适用场景：频繁插入删除 vs 频繁随机访问

2. **LinkedList如何实现Deque接口？**
   ```java
   // 典型Deque操作实现
   public void addFirst(E e) {
       linkFirst(e);
   }
   
   public void addLast(E e) {
       linkLast(e);
   }
   ```
   - 利用双向链表特性自然支持两端操作
   - 所有Deque操作时间复杂度均为O(1)

3. **LinkedList的迭代器实现？**
   ```java
   // ListIterator关键实现
   private class ListItr implements ListIterator<E> {
       private Node<E> lastReturned;
       private Node<E> next;
       private int nextIndex;
       
       public E next() {
           checkForComodification();
           if (!hasNext())
               throw new NoSuchElementException();
           lastReturned = next;
           next = next.next;
           nextIndex++;
           return lastReturned.item;
       }
   }
   ```
   - 维护当前节点指针
   - 支持双向遍历
   - 实现快速失败机制

4. **LinkedList的内存占用？**
   - 每个元素需要额外12字节(32位JVM)或24字节(64位JVM)开销
   - 计算公式：`总内存 ≈ (元素大小 + 24) * size`
   - 相比ArrayList有显著的对象头开销
