---
layout: post
title: 'ConcurrentHashMap源码试读'
date: 2014-11-27 00:50
comments: true
categories: [同步, 线程安全, ConcurrentHashMap, 桶分割远离]
---
#背景
接昨天的文章，继续深入探索ConcurrentHashMap是如何进行分段加锁的。今天的代码比较多，也不是那么理解，只当自己的源码试读记录。
#桶分割
看下面这段代码，是ConcurrentHashMap的一些默认值，定义了默认桶的个数，索引位移数（segmentShift，如桶为16个，即2的4次方，那么该值为32-4=28），桶数组等
```java
    static final int DEFAULT_SEGMENTS = 16;
    static final int MAX_SEGMENTS = 1 << 16;
    final int segmentMask;
    final int segmentShift;
    final Segment[] segments;
```
再来看看桶的数据结构，其成员变量的意义：

* count：Segment中元素的数量
* modCount：对table的大小造成影响的操作的数量（比如put或者remove操作）
* threshold：阈值，Segment里面元素的数量超过这个值依旧就会对Segment进行扩容
* table：链表数组，数组中的每一个元素代表了一个链表的头部
* loadFactor：负载因子，用于确定threshold

```java
static final class Segment<K,V> extends ReentrantLock implements Serializable {  
  transient volatile int count;  
  transient int modCount;  
  transient int threshold;  
  transient volatile HashEntry<K,V>[] table;  
  final float loadFactor;  
}  
```
再看看代码是如何找到key值对应的桶，再在桶中获取相应的HashEnty的，可以看出，这里要进行两次hash的索引，所以速度会比HashMap、HashTable慢很多。
```java ConcurrentHashMap.java片段
final Segment<K,V> segmentFor(int hash) {
	return (Segment<K,V>) segments[(hash >>> segmentShift) & segmentMask];
}
```

```java Segment类片段
V get(Object key, int hash) {  
		//count定义为volatile，因为put、remove都会改变该值，竞争非常激烈
    if (count != 0) { // read-volatile  
    		//HashEntry相比Entry，它把hash和key都定义成了final，是防止链表结构被破坏，出现ConcurrentModification的情况
        HashEntry<K,V> e = getFirst(hash);  
        while (e != null) {  
        		//在确定了链表的头部以后，就可以对整个链表进行遍历，取出key对应的value的值
            if (e.hash == hash && key.equals(e.key)) {  
                V v = e.value;  
                //如果拿出的value的值是null，则可能这个key，value对正在put的过程中
                if (v != null)  
                    return v;  
                //那么就加锁来保证取出的value是完整的，如果不是null，则直接返回value
                return readValueUnderLock(e); // recheck  
            }  
            e = e.next;  
        }  
    }  
    return null;  
}
```
这里也是用位操作来确定链表的头部，与HashMap中的定位方式相同，具体为什么这样减一再与，不是很理解？
```java Segment类片段
HashEntry<K,V> getFirst(int hash) {
    HashEntry[] tab = table;
    return (HashEntry<K,V>) tab[hash & (tab.length - 1)];
}
```

#put操作
put操作一开始就锁定了整个segment，这保证并发的安全，修改数据是不能并发进行的，必须得有个判断是否超限的语句以确保容量不足时能够rehash。
仔细研究后发现，其实segment很像一个HashTable，只是在ConcurrentHashMap中定义了16个HashTable的桶，用来根据hashCode分段存储数据。
```java ConcurrentHashMap.java片段
public V put(K key, V value) {
        if (value == null)
            throw new NullPointerException();
        int hash = hash(key);
        return segmentFor(hash).put(key, hash, value, false);
    }
```
```java segment片段
				V put(K key, int hash, V value, boolean onlyIfAbsent) {
            lock();
            try {
                int c = count;
                if (c++ > threshold) // ensure capacity
                    rehash();
                HashEntry[] tab = table;
                int index = hash & (tab.length - 1);
                HashEntry<K,V> first = (HashEntry<K,V>) tab[index];
                HashEntry<K,V> e = first;
                while (e != null && (e.hash != hash || !key.equals(e.key)))
                    e = e.next;

                V oldValue;
                if (e != null) {
                    oldValue = e.value;
                    if (!onlyIfAbsent)
                        e.value = value;
                }
                else {
                    oldValue = null;
                    ++modCount;
                    tab[index] = new HashEntry<K,V>(key, hash, first, value);
                    count = c; // write-volatile
                }
                return oldValue;
            } finally {
                unlock();
            }
        }
        
```
#remove
remove大多数代码都差不多，就是移除需要移除的元素的操作有些特殊，看代码的注释部分。

```java
        V remove(Object key, int hash, Object value) {
            lock();
            try {
                int c = count - 1;
                HashEntry[] tab = table;
                int index = hash & (tab.length - 1);
                HashEntry first = (HashEntry)tab[index];
                HashEntry e = first;
                while (e != null && (e.hash != hash || !key.equals(e.key)))
                    e = e.next;

                V oldValue = null;
                //e为找到的要删除的元素
                if (e != null) {
                    V v = e.value;
                    if (value == null || value.equals(v)) {
                        oldValue = v;
                        ++modCount;
                        HashEntry newFirst = e.next;
                        //从链表头开始往下扫，扫到e就跳过
                        for (HashEntry p = first; p != e; p = p.next)
                        		//不为e则重新new一个，加到链表中，所以每次remove都会颠倒链表的顺序
                            newFirst = new HashEntry(p.key, p.hash, 
                                                          newFirst, p.value);
                        tab[index] = newFirst;
                        count = c; // write-volatile
                    }
                }
                return oldValue;
            } finally {
                unlock();
            }
        }
```
为什么要都重新new一个HashEntry出来呢？这个因为之前说过，他的hash、next和key都是final的，这意味着在第一次设置了next域之后便不能再改变它，取而代之的是将它之前的节点全都克隆一次。至于entry为什么要设置为不变性，这跟不变性的访问不需要同步从而节省时间有关。