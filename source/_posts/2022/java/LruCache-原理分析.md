---
title: LruCache-原理分析
date: 2022-03-24 00:26:52
categories:
 - Java
tags:
 - Java
cover: https://s1.ax1x.com/2022/03/24/q34Snx.jpg
---

1.LruCache原理是什么
 LruCache 采用最近最少使用算法，设定一个缓存大小，当缓存达到这个大小之后，会将最老的数据移除，避免图片占用内存过大导致OOM。

2.Lrucache源码分析
 ```java
public class LruCache<K, V> {
	// 数据最终存在 LinkedHashMap 中
    private final LinkedHashMap<K, V> map;
	...
	public LruCache(int maxSize) {
        if (maxSize <= 0) {
            throw new IllegalArgumentException("maxSize <= 0");
        }
        this.maxSize = maxSize;
		// 创建一个LinkedHashMap，accessOrder 传true
        this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
    }
    ...
```
LruCache 构造方法里创建一个LinkedHashMap，accessOrder 参数传true，表示访问时重新排序，accessOrder 参数传false，表示插入时重新排序。（/* LruCache 核心工作原理就在此*/）


插入顺序是什么意思？
按照对象插入的顺序进行排列，可以简单说按照插入的时间先后顺序排列。
依次放入1,2,3,4,5   那么按照插入顺序后LruCache中的对象顺序就是1,2,3,4,5

那么访问顺序是什么意思呢？
访问顺序意思就是LruCache中被访问过的子对象会重新排序，那么意味着我们先依次排序进行对象缓存1,2,3,4,5
现阶段的对象顺序是1,2,3,4,5
如果现在内存使用了2号缓存，那么就表示2号被访问了，重新排序 2,1,3,4,5
如果现在内存再命中了4号缓存，那么就表示4号被访问了，重新排序 4,2,1,3,5
内存再命中了5号缓存，那么就表示4号被访问了，重新排序 5,4,2,1,3
.....
.....

所以每次使用过的缓存都会被重新排到缓存队列的最前面。

那么意味着，队列越后面的缓存就使用得越少，当缓存队列容量达到阀值后，再往队列中添加新缓存时则可以将队列最后面的使用次数最少的缓存依次移除，插入新缓存，达到Lru算法的效果。
