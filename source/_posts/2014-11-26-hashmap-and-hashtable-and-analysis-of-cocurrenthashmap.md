---
layout: post
title: 'HashMap、HashTable、CocurrentHashMap的几点自问自答'
date: 2014-11-26 01:29
comments: true
categories: [多线程, HashMap, HashTable, CocurrentHashMap, 同步, 线程安全]
---
#背景
最近项目线程安全的问题比较严重，因此也对HashMap、HashTable、CocurrentHashMap、ArrayList、CopyOnWriteArrayList做了点浅析。以下是几点自问自答。
#自问自答
##HashMap与HashTable区别？

* HashMap可以接受null键值和值，而Hashtable则不能
* 前置非synchronized，后者是
* 前者的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的，区别见“把HashMap用在多线程环境会怎么样？”章节

##HashMap的元素次序不变吗？
随着时间的推移，HashMap中的元素次序可能会变。

##如何让Has和Map同步?
 Map m = Collections.synchronizeMap(hashMap);
**但有些因素使得它们不适合高并发的系统。它们仅有单个锁，对整个集合加锁，以及为了防止ConcurrentModificationException异常经常要在迭代的时候要将集合锁定一段时间，这些特性对可扩展性来说都是障碍。**

##那用什么集合类进行多线程既安全又提供高并发性?
ConcurrentHashMap和CopyOnWriteArrayList保留了线程安全的同时，也提供了更高的并发性。 ConcurrentHashMap和CopyOnWriteArrayList并不是处处都需要用，大部分时候你只需要用到HashMap和 ArrayList，它们用于应对一些普通的情况。
 

##把HashMap用在多线程环境会怎么样？
HashMap的iterator是fail-fast迭代器，别的线程改变了map的结构， （增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。 但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。

##性能都如何？
HashMap是非synchronized,CocurrentHashMap和 Hashtable 是，所以HashMap快

##CocurrentHashMap和 Hashtable区别？
用于多线程的环境，但是当Hashtable的大小增加到一定的时候，性能会急剧下降，因为迭代时需要被锁定很长的时间。
ConcurrentHashMap同步性能更好，因为它仅仅根据同步级别对map的一部分进行上锁 ， 它引入了分割(segmentation)，不论它变得多么大，仅仅需要锁定map的某个部分，而其它的线程不需要等到迭代完成才能访问map。但是HashTable会锁整个map提供更强的线程安全性

##HashMap内部实现原理？
HashMap在Put的时候，先对key进行hashCode()，返回的值用于查找bucket位置来存储对象，因此它是在bucket中储存键对象和值对象，作为Map.Entry。

```java
public V put(K key, V value) {
	if (key == null)
	    return putForNullKey(value);
  //计算key的hashCode
  int hash = hash(key.hashCode());
  //找hashCode在bucket中的索引值
  int i = indexFor(hash, table.length);
  //找相应索引值的Entry（链表结构）中是否有key相同的值，如果有则覆盖
  for (Entry<K,V> e = table[i]; e != null; e = e.next) {
      Object k;
      if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
        V oldValue = e.value;
        e.value = value;
        e.recordAccess(this);
        return oldValue;
      }
  }
  //没找到就在该位置的链表增加一个Entry到头部，这里的addEntry还会检查容量，超过负载指数就扩容
  modCount++;
  addEntry(hash, key, value, i);
  return null;
}
```
当我们调用get()方法，HashMap会使用键对象的hashcode找到bucket位置，这个位置是一个链表，包含有键值对的Map.Entry对象 ，存储的是键值对，当有两个值存在于同一个bucket时候， 在调用get时，会先找到bucket位置之后，会调用keys.equals()方法去找到链表中正确的节点，最终找到要找的值对象
```java
public V get(Object key) {
		if (key == null)
    	return getForNullKey();
    int hash = hash(key.hashCode());
    for (Entry<K,V> e = table[indexFor(hash, table.length)];e != null;e = e.next) {
    	Object k;
    	if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
    		return e.value;
    }
    return null;
}
```
所以， 采用合适的equals()和hashCode()方法的话，将会减少碰撞的发生，提高效率。所以，用不可变性使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择

##HashMap如何扩容？
 默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类(如ArrayList等)一样，将会创建原来 HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因 为它调用hash方法找到新的bucket位置。

