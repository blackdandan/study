# Java集合框架(Java Collections Framework)

Java集合框架是Java中用于存储和操作数据集合的一组接口和类。

## 核心组件

```
                   ┌────────────────────────┐
                   │     Java集合框架       │
                   └──────────┬─────────────┘
                              │
                   ┌──────────▼─────────────┐
                   │     Collection<E>      │
                   └─────┬───────────┬──────┘
                         │           │
              ┌──────────▼───┐   ┌───▼──────────┐
              │   List<E>    │   │   Set<E>     │
              └────┬────┬────┘   └────┬─────────┘
                   │    │             │
        ┌──────────▼┐ ┌─▼────────┐ ┌──▼────────────┐
        │ ArrayList │ │ LinkedList│ │   HashSet     │
        └───────────┘ └──────────┘ └────┬──────────┘
                   │                    │
        ┌──────────▼┐        ┌─────────▼────────────┐
        │   Stack   │        │     LinkedHashSet    │
        └───────────┘        └──────────────────────┘
                                       │
                          ┌────────────▼────────────┐
                          │        TreeSet          │
                          └─────────────────────────┘

                   ┌────────────────────────┐
                   │        Queue<E>        │
                   └─────┬───────────┬──────┘
                         │           │
              ┌──────────▼───┐   ┌───▼──────────┐
              │ PriorityQueue│   │  ArrayDeque  │
              └──────────────┘   └──────────────┘

                   ┌────────────────────────┐
                   │        Map<K,V>        │
                   └─────┬───────────┬──────┘
                         │           │
                ┌────────▼────┐   ┌──▼─────────────┐
                │  HashMap    │   │   TreeMap      │
                └────┬────────┘   └────────────────┘
                     │
        ┌────────────▼────────────┐
        │    LinkedHashMap        │
        └─────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │  ConcurrentHashMap      │
        └─────────────────────────┘
                     │
        ┌────────────▼────────────┐
        │    WeakHashMap         │
        └─────────────────────────┘
```

## 1. 接口(Interfaces)

- **Collection**: 所有集合类的根接口
- **List**: 有序集合，允许重复元素
- **Set**: 不允许重复元素的集合
- **Map**: 键值对映射

## 2. 实现类(Implementations)

| 类型 | 实现类 | 特点 |
|------|--------|------|
| List | ArrayList | 基于数组，随机访问快 |
| List | LinkedList | 基于链表，插入删除快 |
| Set  | HashSet | 基于哈希表，无序 |
| Set  | TreeSet | 基于红黑树，有序 |
| Map  | HashMap | 基于哈希表，键无序 |
| Map  | TreeMap | 基于红黑树，键有序 |

## 3. 算法(Algorithms)

- 排序(Sorting)
- 搜索(Searching)
- 混排(Shuffling)
- 常规数据操作

Java集合框架提供了高性能、线程安全和类型安全的数据结构实现。
