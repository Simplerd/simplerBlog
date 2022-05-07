---
title: LinkedHashMap-原理分析
date: 2022-03-23 23:40:39
categories:
 - Java
tags:
 - Java
cover: https://s1.ax1x.com/2022/03/23/q3c9JI.jpg
---
## 前言：

LinkedHashMap 是如何做到LruCache ，下面我们从源码中看一看

## LinkedHashMap构造函数
```java
/**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the specified initial capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public LinkedHashMap(int initialCapacity, float loadFactor) {
        super(initialCapacity, loadFactor);
        accessOrder = false;
    }

    /**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the specified initial capacity and a default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity
     * @throws IllegalArgumentException if the initial capacity is negative
     */
    public LinkedHashMap(int initialCapacity) {
        super(initialCapacity);
        accessOrder = false;
    }

    /**
     * Constructs an empty insertion-ordered <tt>LinkedHashMap</tt> instance
     * with the default initial capacity (16) and load factor (0.75).
     */
    public LinkedHashMap() {
        super();
        accessOrder = false;
    }

    /**
     * Constructs an insertion-ordered <tt>LinkedHashMap</tt> instance with
     * the same mappings as the specified map.  The <tt>LinkedHashMap</tt>
     * instance is created with a default load factor (0.75) and an initial
     * capacity sufficient to hold the mappings in the specified map.
     *
     * @param  m the map whose mappings are to be placed in this map
     * @throws NullPointerException if the specified map is null
     */
    public LinkedHashMap(Map<? extends K, ? extends V> m) {
        super();
        accessOrder = false;
        putMapEntries(m, false);
    }

    /**
     * Constructs an empty <tt>LinkedHashMap</tt> instance with the
     * specified initial capacity, load factor and ordering mode.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @param  accessOrder     the ordering mode - <tt>true</tt> for
     *         access-order, <tt>false</tt> for insertion-order
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public LinkedHashMap(int initialCapacity,
                         float loadFactor,
                         boolean accessOrder) {
        super(initialCapacity, loadFactor);
        this.accessOrder = accessOrder;
    }
```

先分别看下这三个参数：int initialCapacity、float loadFactor, boolean accessOrder

initialCapacity：表示是初始数组长度

loadFactor：表示负载因子，表示数组的元素数量/数组长度超过这个比例，数组就要扩容

accessOrder：false： 基于插入顺序(默认) true： 基于访问顺序

关于插入顺序，访问顺序可以看我的另一篇文章： {% post_link java-链表 java链表(单向，循环、双向)%}

下面看下LinkedHashMap 最常用的get，put 以及afterNodeAccess、afterNodeInsertion方法

## get

LinkedHashMap 重写了get方法，实现了LrucaChe
```java
/**Returns the value to which the specified key is mapped, or null if this map contains no mapping for the key.
More formally, if this map contains a mapping from a key k to a value v such that (key==null ? k==null : key.equals(k)), then this method returns v; otherwise it returns null. (There can be at most one such mapping.)
A return value of null does not necessarily indicate that the map contains no mapping for the key; it's also possible that the map explicitly maps the key to null. The containsKey operation may be used to distinguish these two cases*/
<P>
返回指定键映射到的值，如果此映射不包含该键的映射，则返回null。
更正式地说，如果这个映射包含从键k到值v的映射，使得（键==null？k==null:key.equals（k）），那么这个方法返回v；否则返回null。（最多可以有一个这样的映射。）
空映射不一定包含空映射的返回值；映射也可能显式地将密钥映射为null。containsKey操作可用于区分这两种情况
</p>
public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        // accessOder为true时，被访问的节点被置于双向链表尾部
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }

/**Returns the value to which the specified key is mapped, or defaultValue if this map contains no mapping for the key.*/
<p>返回指定键映射到的值，如果此映射不包含该键的映射，则返回defaultValue。</p>

  public V getOrDefault(Object key, V defaultValue) {
       Node<K,V> e;
       if ((e = getNode(hash(key), key)) == null)
           return defaultValue;
       if (accessOrder)
           afterNodeAccess(e);
       return e.value;
   }
```
可以看到在get 方法中调用了afterNodeAccess方法，下面看下这个方法干了什么

## afterNodeAccess
```java
void afterNodeAccess(Node<K,V> e) { // move node to last
    // 用 last 表示插入 e 前的尾节点
    // 插入 e 后 e 是尾节点, 所以也是表示 e 的前一个节点
    LinkedHashMap.Entry<K,V> last;
    //如果是访问序，且当前节点并不是尾节点
    //将该节点置为双向链表的尾部
    if (accessOrder && (last = tail) != e) {
        // p: 当前节点
        // b: 前一个节点
        // a: 后一个节点
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        // 因为要移动到链尾，所以先至尾指针为空
        p.after = null;

        //如果前面没有元素，则p之前为头结点，直接让a成为头结点
        if (b == null)
            head = a;
        // 否则b的尾指针指向a
        else
            b.after = a;

        if (a != null)
        //如果a不为空，则a的头指针指向b
            a.before = b;
        else
        //否则 p之前就为尾指针，则另b成为尾指针
            last = b;

        if (last == null)
            head = p;
        //如果双向链表中只有p一个节点，则令p即为头结点，也为尾节点
        else {
        //否则 将p插入链尾
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```
## put

LinkedHashMap 并没有重写HashMap的 put 方法，调用的还是HashMap得put方法
```java
/**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

    /**
     * Implements Map.put and related methods.
     *
     * @param hash hash for key
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
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
此外HashMap的putVal()方法，还调用了afterNodeInsertion()方法，

## afterNodeInsertion

当插入数据时，将双向链表的头结点移除，这几个方法让LinkedHashMap实现了LRU算法。不过removeEldestEntry()默认是返回false的，需要子类继承重写removeEldestEntry()方法。
```java
void afterNodeInsertion(boolean evict) { // possibly remove eldest
        LinkedHashMap.Entry<K,V> first;
        if (evict && (first = head) != null && removeEldestEntry(first)) {
            K key = first.key;
            removeNode(hash(key), key, null, false, true);
        }
    }
```

LinkedHashMap的remove()方法也是调用的HashMap的remove()方法，
```java
/**
     * Removes the mapping for the specified key from this map if present.
     *
     * @param  key key whose mapping is to be removed from the map
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V remove(Object key) {
        Node<K,V> e;
        return (e = removeNode(hashLkey), key, null, false, true)) == null ?
            null : e.value;
    }


/**
     * Implements Map.remove and related methods.
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to match if matchValue, else ignored
     * @param matchValue if true only remove if value is equal
     * @param movable if false do not move other nodes while removing
     * @return the node, or null if none
     */
    final Node<K,V> removeNode(int hash, Object key, Object value,
                               boolean matchValue, boolean movable) {
        Node<K,V>[] tab; Node<K,V> p; int n, index;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (p = tab[index = (n - 1) & hash]) != null) {
            Node<K,V> node = null, e; K k; V v;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                node = p;
            else if ((e = p.next) != null) {
                if (p instanceof TreeNode)
                    node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
                else {
                    do {
                        if (e.hash == hash &&
                            ((k = e.key) == key ||
                             (key != null && key.equals(k)))) {
                            node = e;
                            break;
                        }
                        p = e;
                    } while ((e = e.next) != null);
                }
            }
            if (node != null && (!matchValue || (v = node.value) == value ||
                                 (value != null && value.equals(v)))) {
                if (node instanceof TreeNode)
                    ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
                else if (node == p)
                    tab[index] = node.next;
                else
                    p.next = node.next;
                ++modCount;
                --size;
                afterNodeRemoval(node);
                return node;
            }
        }
        return null;
    }
```

总结：

*   LinkedHashMap 继承 HashMap，拥有 HashMap 的功能
*   LinkedHashMap 维护了两个数据结构，一是 HashMap 的结构，二是用来做迭代的双向链表
*   LinkedHashMap 独特的迭代器设计和一些函数的重写，导致迭代器按双向链表迭代，并且若没有设置 accessOrder，则按插入顺序迭代，否则，按访问顺序迭代
*   通过重写removeEldestEntry可以实现 LruCache 的功能
