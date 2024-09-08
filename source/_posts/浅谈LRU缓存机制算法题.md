---
title: 浅谈 LRU 缓存机制算法题
author: 荀乐
categories: 算法题
tags:
  - LRU
  - leetcode
  - 算法
date: 2021-07-23 20:03:02
---

# [146. LRU 缓存机制](https://leetcode-cn.com/problems/lru-cache/)

运用你所掌握的数据结构，设计和实现一个 [LRU (最近最少使用) 缓存机制](https://baike.baidu.com/item/LRU) 。

实现 `LRUCache` 类：

- `LRUCache(int capacity)` 以正整数作为容量 `capacity` 初始化 LRU 缓存
- `int get(int key)` 如果关键字 `key` 存在于缓存中，则返回关键字的值，否则返回 `-1` 。
- `void put(int key, int value)` 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字-值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。
- 要求在`O(1)`时间复杂度内完成`get`和`put`操作

---

## 实现思路及参考代码

要在`O(1)`时间内完成`get`和`put`操作，很自然的想到哈希表结构，但是很明显页面在置换时对顺序是有要求的。如果使用链表结构，在`get`操作上是`O(n)`的复杂度。故可以考虑采用哈希表加上双向链表的结构，哈希表存储对应的结点。

**原生实现**

先定义双向链表节点类

```java
class Node{
    //定义key的原因是需要key对哈希表进行操作
    public int key, val;
    public Node next, prev;
    public Node(int k, int v) {
        this.key = k;
        this.val = v;
    }
}
```

使用`Node`构建双向链表

```java
class DoubleList{
    //头尾节点
	private Node head, tail;
    //链表节点数
    private int size;

    //构造方法，初始化头尾节点
    public DoubleList() {
        this.head = new Node(0,0);
        this.tail = new Node(0,0);
        head.next = tail;
        tail.prev = head;
        size = 0;
    }

    //从链表尾部添加节点
    public void addLast(Node node) {
        node.prev = tail.prev;
        node.next = tail;
        tail.prev.next = node;
        tail.prev = node;
        size++;
    }

    //删除链表指定节点
    public void remove(Node node) {
        node.prev.next = node.next;
        node.next.prev = node.prev;
        size--;
    }

    //删除链表第一个节点
    public Node removeFirst() {
        if (head.next == tail) return null;
        Node node = head.next;
        remove(node);
        return node;
    }

    //返回链表节点数
    public int size(){
        return size;
    }
}
```

其中删除操作的时间复杂度是`O(1)`，因为是双向链表所以可以直接找到前驱节点，如果是单链表删除操作会需要`O(n)`时间复杂度。

双链表结合哈希表实现`LRUCache`类

```java
class LRUCache{
    private HashMap<Integer, Node> map;
    private DoubleList cache;
    //缓存容量
    private int capacity;

    public LRUCache(int capacity) {
        this.map = new HashMap<>();
        this.cache = new DoubleList();
        this.capacity = capacity;
    }

    //get操作
    public int get(int key) {
        //先查看是否存在该节点
        if (!map.containsKey(key)) return -1;
        //如果存在，将该节点移动到链表尾部成为最近最常使用节点
        int val = map.get(key).val;
        makeRecently(key);
        return val;
    }

    //put操作
    public void put(int key, int value) {
        //如果链表中已存在该节点，将该节点直接移动到尾部
        if (map.containsKey(key)) {
            map.put(key, value);
            makeRecently(key);
            return ;
        }
        //如果不存在该节点，则首先要判断缓存是否已满；
        if (cache.size() == this.capacity) {
            //如果满了删除第一个节点
            Node oldestNode = cache.removeFirst();
            map.remove(oldestNode.key);
        }

        Node newNode = new Node(key, value);
        map.put(key, newNode);
        cache.addLast(newNode);
    }

    //更新最近使用节点
    public void makeRecently(int key) {
        Node newestNode = map.get(key);
        cache.remove(newestNode);
        cache.addLast(newestNode);
    }
}
```

---

**使用`LinkedHashMap`**

Java 内置集合类型`LinkedHashMap`是一个哈希链表。

```java
class LRUCache{
    LinkedHashMap<Integer, Integer> cache;
    int capacity;

    public LRUCache(int capacity) {
        cache = new LinkedHashMap<>();
        this.capacity = capacity;
    }

    //get操作
    public int get(int key) {
        if (!cache.containsKey(key)) return -1;
        makeRecently(key);
       	return cache.get(key);
    }

    //put操作
    public void put(int key, int value) {
        if (cache.containsKey(key)) {
            cache.put(key, value);
            makeRecently(key);
            return ;
        }
        if (this.capacity == cache.size()) {
            int oldestNode = cache.keySet().iterator().next();
            cache.remove(oldestNode);
        }
        cache.put(key, value);
        makeRecently(key);
    }

    //更新节点操作
    public void makeRecently(int key) {
        int val = cache.get(key);
        cache.remove(key);
        cache.put(key, val);
    }
}
```

**最简实现**

```java
class LRUCache extends LinkedHashMap<Integer, Integer>{
    private int capacity;

    public LRUCache(int capacity) {
        super(capacity, 0.75F, true);
        this.capacity = capacity;
    }

    public int get(int key) {
        return super.getOrDefault(key, -1);
    }

    public void put(int key, int value) {
        super.put(key, value);
    }

    //会在put调用时自动调用
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, Integer> eldest) {
        return size() > capacity;
    }
}
```

---

本文参考：[labuladong 的算法小抄](https://github.com/labuladong/fucking-algorithm#labuladong-的算法小抄)
