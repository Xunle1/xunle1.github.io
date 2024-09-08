---
title: HashMap 源码阅读
date: 2024-07-10 21:16:34
categories: 技术
tags:
  - HashMap
  - 源码
  - Java
---

本文为阅读 HashMap 源码的笔记，主要了解 HashMap 增删改查的过程。

文中代码和描述基于 openjdk 17，HashMap 底层数据结构采用 `Node` 数组 + 链表 + 红黑树。

```
openjdk 17.0.11 2024-04-16 LTS
OpenJDK Runtime Environment Corretto-17.0.11.9.1 (build 17.0.11+9-LTS)
OpenJDK 64-Bit Server VM Corretto-17.0.11.9.1 (build 17.0.11+9-LTS, mixed mode, sharing)
```

# 插入元素

调用 `put` 方法插入一个键值对，流程为：

1. 调用 `putVal` 方法

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

2. 如果为空表，则进行初始化

```java
/**
 * Implements Map.put and related methods. * * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    if ((tab = table) == null || (n = tab.length) == 0)
	    // 如果当前表为空表，初始化表。
        n = (tab = resize()).length;
    ...
}
```

3. 如果当前的哈希桶（桶数组）为空，直接插入

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    ...
    // 当前桶数组为空。
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
	...
}
```

4. 如果当前的哈希桶有值，即产生哈希冲突，首先判断链表头，即哈希桶第一个 `Node` 键值对和将要插入的键值对是否相同，相同则覆盖。

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
	...
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 当前桶数组不为空。
    else {
        Node<K,V> e; K k;
        // 重复的键值对
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        ...
    }
    // 覆盖键值对的值
    if (e != null) { // existing mapping for key
        V oldValue = e.value;
        if (!onlyIfAbsent || oldValue == null)
            e.value = value;
        afterNodeAccess(e);
        return oldValue;
    }
	...
}
```

5. 如果哈希桶第一个键值对不重复，判断是否为树形节点，如果为树形节点，调用 `putTreeval`

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
	...
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            e = p;
        // 如果是树形节点则调用树节点插入方法
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
		...
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            afterNodeAccess(e);
            return oldValue;
        }
    }
	...
}
```

6. 如果不是树形节点，则遍历哈希桶，即遍历链表，在链表尾部插入，如果链表长度大于树化的阈值，则将当前哈希桶的节点树化。

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
	...
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    else {
		...
        else {
	        // 遍历链表节点
            for (int binCount = 0; ; ++binCount) {
	            // 在尾部插入新键值对
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 链表节点超过阈值，进行树化
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 存在重复节点
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }
		...
    }
	...
}
```

7. 如果表中元素数量大于阈值，则进行扩容。

```java
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
	...
    ++modCount;
    // 元素数量大于阈值进行扩容
    if (++size > threshold)
        resize();
    afterNodeInsertion(evict);
    return null;
}
```

