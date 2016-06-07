---
layout: post
title: '不靠谱的volatile'
date: 2014-01-09 11:00
comments: true
categories: [java, 多线程, volatile]
---
#1 Volatile关键字的含义
在java线程并发处理时，用volatile修饰的变量，线程在每次使用变量的时候都会读取变量修改后的最新值。
#2 与synchronized的对比
Synchronized对 对象的锁有两个特性：互斥性和可见性。
##2.1 互斥性
互斥锁就是当Synchronized描述的方法或块获取了对象锁，那其他再视图操作该对象锁的同步块或同步方法就会被阻塞，直到对象锁被释放。
##2.2 可见性
可见性就是同步块或同步方法每次使用变量的时候都会读取变量修改后的最新值。
##2.3 结论
很明显，volatile只有具有可见性的特性。但是，他的可见性对于原子性操作又怎么个不靠谱法呢？
#3 例子
##3.1 程序
```java Counter.java
/** 
* @ClassName: Counter 
* @Description: 测试volatile
* @author David 
* @date 2014-1-9 下午04:37:36 
* @version 1.0 
*/
public class Counter {
    
    /** 
     * @Fields count : 不用volatile修饰
     */ 
    public static int count = 0;
    /** 
    * @Fields count : 用volatile修饰
    */ 
//    public volatile static int count = 0;
    
    /** 
     *
     * @Title: inc 
     * @Description: <b>功能：</b>自增长<br
     * @return void    返回类型 
     */
    public static void inc(){
        try {
            Thread.sleep(1);
        }
        catch(InterruptedException e) {
        }
        
        count++;
    }

    /** 
     *
     * @Title: main 
     * @Description: <b>功能：</b>我是一个main<br
     * @return void    返回类型 
     */
    public static void main(String[] args) {
        for (int i = 0; i < 1000; i++) {
            new Thread(new Runnable() {
                public void run() {
                    Counter.inc();
                }
            }).start();
        }
        
        try {
            Thread.sleep(3000);
        }
        catch(InterruptedException e) {
        }
        
        System.out.println("你猜，最后的结果会是多少呢？结果是"+Counter.count);
    }

}
```
##3.2 结果
该函数输出的结果不固定，大概就是在990左右。
那如果给count变量加上volatile会怎么样呢？
结果还是在990多左右，可能会比没加volatile的情况更接近1000了。
但是为什么会出现这个情况呢，不是已经确保线程从内存中获取的count变量始终是最新的了吗。


#4 原因
下图描述了JVM运行时刻的内存分配。右边的内存区域就是JVM虚拟机的堆内存，每一个线程运行时都有一个线程栈，线程栈保存了线程运行时候的变量信息。当线程访问某一个对象值的时候，首先通过对象的引用找到对应在堆内存中的变量值，然后把该变量的具体指加载到线程本地内存中，建立一个变量的副本，之后线程内存和主内存之间就没啥关系了。
知道线程退出之前，自动把线程变量副本写到对象在堆中的变量中。这时候堆中的对象值才发生变化。
因此，线程可见性的特性只保证了从堆中复制的值是最新的，而不能保证线程栈中的值是最新的。
![1.jpg](http://user-image.logdown.io/user/3769/blog/3827/post/175079/Git9mKXwT8yiboZ0JN9X_1.jpg)
 
#5 Volatile的适用场合
既然volatile这么不靠谱，那为什么还会有他呢？其实他还是有一些适用的场合的。
##5.1 读操作远远大于写操作的时候
由于volatile的开销比synchronized小的多的多。所以，出于性能考虑，在读操作远远大于写操作的时候，还是适用volatile吧。
##5.2 用作状态的时候
比如调度引擎表示是否启动状态的时候，执行调度方法的时候都需要判断状态是否为启动才继续。这种时候适合用volatile。
其实这种情况也就是2.1中的一种特殊情况。
