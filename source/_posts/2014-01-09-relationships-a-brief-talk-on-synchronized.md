---
layout: post
title: '企业的人际关系---浅淡java的synchronized'
date: 2014-01-09 23:12
comments: true
categories: [java, 个人随笔, 多线程, synchronized, 人际关系]
---
#人机关系的故事
##领导和前任领导
最近有朋友给我抱怨说，他们新任领导和上任领导怎么勾心斗角、明争暗斗的。让我突然觉得新任领导和上任领导的关系就和婆媳关系一样。新任领导必然要否认上任领导，这样就会到处给别人说团队业绩、效率是怎么在自己的英明领导下快速提升。而团队成员在和前任领导聊天时候就必然会说“哎呀，那个XXX啊，就是个渣啊，和您在的时候简直就没得比啊，差的多的多了啊”。这样你来我往，这两任领导之前关系再好现在也好不起来了。
##官僚化
又想到之前阿伯伯公司的一个朋友说，你别看我们是世界500强，企业里官僚化的可严重了。进阿伯伯首先就要认准领导，拍好领导马屁。一帮招你进去的那个领导就决定了你在公司的地位。如果他没什么本事，你在公司也就没什么机会了。如果他把你招进去没多久然后他就空降到别的部门去了，而且还不带上你，那你就更惨了，新任的领导一般对于前朝旧臣是不会重用的。
##笑里藏刀
再想到之前，大学同学在公司中的一个比较好的同事，和他表面上好兄弟同甘共苦，但是背地里不断和领导揭他的短，然后好的机会，有油水的大便宜都占了，脏活累活苦活都给了他。
#IT企业
这样的故事还很多，但是至少我工作三年是没碰到这么邪恶的了。所以我就有了这么一个观点，互联网公司甚至IT企业的人际关系简单。为什么这么说呢，我觉得有两个原因，首先就是互联网公司发展快，大家活都没时间干了，哪里有时间吵架啊。还有就是IT企业人员流动快，还没形成官僚化气氛的时候，团队已经更新了。
#总之
当然这些都是我的瞎扯，不过大家进入社会后，在人际关系上一定还是要小心的。我是一个爱开玩笑的人，但是我开玩笑也一定会有分寸。开不起玩笑的人我是万万不会和他开玩笑的。总之，亲君子，远小人吧。


------------------------------------------------分割线-----------------------------------------
#Synchronized详解
##1 概述
synchronized标示符为java内置了语言级的同步原语，这简化了Java中多线程同步的使用，使用了synchronized关键字就可以轻松地解决多线程共享数据同步问题。
那synchronized如何对共享数据进行同步呢。
本篇文章就不花大篇幅介绍锁的概念，就简单介绍下java对象锁的两个特性，互斥特性和可见特性。
##2 锁
要进行多线程同步操作，就必须要一个锁，常用的方式为对象锁和类锁。
###2.1 对象锁
如果此对象的对象锁已被其他调用者占用，另一个同步方法/同步块要获得该对象锁则需要等待此对象锁被释放。
###2.2 类锁
类似于对象锁。静态方法用synchronized描述符是获取类锁，同步块中也可以使用synchronized(xxx.getClass()/Xxx.class)方法来获取类锁。
其实该锁也就是个对象锁，因为Xxx.class其实就是Xxx类的一个静态对象，只是对于该类所有的对象，这个静态对象是唯一的，所以不管Xxx类有多少个实例对象xxx，都可以用该锁来实现同步。
###2.3 互斥性
互斥锁就是当Synchronized描述的方法或块获取了对象锁，那其他再视图操作该对象锁的同步块或同步方法就会被阻塞，直到对象锁被释放。
###2.4 可见性
可见性就是同步块或同步方法每次使用变量的时候都会读取变量修改后的最新值。
##3 例子
解释太多概念不如直接通过代码来理解下。
###3.1 同步方法，不能被多个线程同时调用
####3.1.1 代码
```java
public synchronized void sayHello(String name) {
        System.out.println("Hello " + name + "!现在时间是:" + getCurrentTime());
        try {
            Thread.sleep(5000);
        }
        catch(InterruptedException e) {
        }
    }
public static void main(String[] args) {
        final TestSynchronized syncObject = new TestSynchronized();
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                syncObject.sayHello(Thread.currentThread().getName());
            }
        }, "线程1号");
        Thread t2 = new Thread(new Runnable() {
            public void run() { 
                syncObject.sayHello(Thread.currentThread().getName());
            }
        }, "线程2号");
t1.start();
        t2.start();
        }
```
####3.1.2 输出
``` Console
Hello 线程1号!现在时间是:2014年01月09日09时50分57秒
Hello 线程2号!现在时间是:2014年01月09日09时51分02秒
```
####3.1.3 分析
两次输出相隔五秒可以看出，synchronized描述的方法，单一个线程占用的时候，别的线程是无法访问该方法的。
###3.2 一个对象的一个同步方法被一个线程占用，该对象的其他同步方法也不能被其他线程调用
####3.2.1 代码
```java
public synchronized void sayHello(String name) {
        System.out.println("Hello " + name + "!现在时间是:" + getCurrentTime());
        try {
            Thread.sleep(5000);
        }
        catch(InterruptedException e) {
        }
    }
public synchronized void sayHi(String name) {
        System.out.println("Hi " + name + "!现在时间是:" + getCurrentTime());
        try {
            Thread.sleep(5000);
        }
        catch(InterruptedException e) {
            e.printStackTrace();
        }
    }
public static void main(String[] args) {
        final TestSynchronized syncObject = new TestSynchronized();
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                syncObject.sayHello(Thread.currentThread().getName());
            }
        }, "线程1号");
        Thread t2 = new Thread(new Runnable() {
            public void run() { 
                syncObject.sayHi (Thread.currentThread().getName());
            }
        }, "线程2号");
		 t1.start();
        t2.start();
        }
```
####3.2.2 输出
```Console
Hi 线程2号!现在时间是:2014年01月09日10时02分36秒
Hello 线程1号!现在时间是:2014年01月09日10时02分41秒
```
####3.2.3 分析
两次输出相隔五秒可以看出，synchronized描述的方法，单一个线程占用的时候，别的线程不但无法访问该方法的还无法调用该对象实例的其他方法。
也说明synchronized描述方法的时候锁的对象就是调用的对象。
###3.3 一个对象的一个同步方法被一个线程占用，该对象的其他同步块也不能被其他线程调用
####3.3.1 代码
```java
public synchronized void sayHello(String name) {
        System.out.println("Hello " + name + "!现在时间是:" + getCurrentTime());
        try {
            Thread.sleep(5000);
        }
        catch(InterruptedException e) {
        }
    }
public synchronized void sayHi(String name) {
        System.out.println("Hi " + name + "!现在时间是:" + getCurrentTime());
        try {
            Thread.sleep(5000);
        }
        catch(InterruptedException e) {
            e.printStackTrace();
        }
    }
public void sayHaha(String name) {

        System.out.println(name + "，哈哈哈，你已经进入了大笑三声方法中了,接下来即将进入同步块！" + "现在时间是:"
                + getCurrentTime());
        synchronized (this) {
            System.out.println("这里是大笑三声同步块 " + name + "!现在时间是:"
                    + getCurrentTime());
            try {
                Thread.sleep(5000);
            }
            catch(InterruptedException e) {
            }
        }
        ;

    }
public static void main(String[] args) {
        final TestSynchronized syncObject = new TestSynchronized();
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                syncObject.sayHello(Thread.currentThread().getName());
            }
        }, "线程1号");
        Thread t2 = new Thread(new Runnable() {
            public void run() { 
                syncObject.sayHi (Thread.currentThread().getName());
            }
        }, "线程2号");
Thread t4 = new Thread(new Runnable() {
            public void run() {
                syncObject.sayHaha(Thread.currentThread().getName());

            }
        }, "线程(5-1)号");
		 t1.start();
		 t2.start();
        t4.start();
        }
```
####3.3.2 输出
```Console
Hello 线程1号!现在时间是:2014年01月09日10时15分27秒
线程(5-1)号，哈哈哈，你已经进入了大笑三声方法中了,接下来即将进入同步块！现在时间是:2014年01月09日10时15分27秒
Hi 线程2号!现在时间是:2014年01月09日10时15分32秒
这里是大笑三声同步块 线程(5-1)号!现在时间是:2014年01月09日10时15分37秒
```
####3.3.3 分析
从提示可以看出，线程4开始时候就已经调用了sayHaha方法，但是无法进入该方法的同步块，因为对象实例的锁已经被线程1占用了。
虽然等到线程1把锁释放了，但是线程2也在等待。所以线程1释放锁后线程2先占用了锁，等线程2释放了锁后，线程4才获得了对象锁，才开始执行sayHaha方法的同步块。
###3.4 不同类实例的对象锁不互斥
####3.4.1 代码
与3.1代码其他部分不变，就加入
```java
……
public void sayWawa(String name) {

        System.out.println(name + "，哈哈哈，你已经进入了大哭三声方法中了,接下来即将进入同步块！" + "现在时间是:"
                + getCurrentTime());
        synchronized (name) {
            System.out.println("这里是大哭三声同步块 " + name + "!现在时间是:"
                    + getCurrentTime());
            try {
                Thread.sleep(5000);
            }
            catch(InterruptedException e) {
            }
        }
        ;

    }
……
Thread t5 = new Thread(new Runnable() {
            public void run() { 
                syncObject.sayWawa(Thread.currentThread().getName());
                
            }
        }, "线程5号");
……
t5.start();
```

####3.4.2 输出
```Console
线程5号，哈哈哈，你已经进入了大哭三声方法中了,接下来即将进入同步块！现在时间是:2014年01月09日10时35分50秒
Hello 线程1号!现在时间是:2014年01月09日10时35分50秒
这里是大哭三声同步块 线程5号!现在时间是:2014年01月09日10时35分50秒
```
####3.4.3 分析
从提示可以看出，线程5调用了sayHaha方法后立马执行了同步块，因为同步块占据的是方法的参数变量name的锁，线程1获取的是syncObject对象锁，所以和线程1的不互斥.

###3.5 同一个类的不同实例的对象锁不同
####3.5.1 代码
```java
public synchronized void sayHello(String name) {
        System.out.println("Hello " + name + "!现在时间是:" + getCurrentTime());
        try {
            Thread.sleep(5000);
        }
        catch(InterruptedException e) {
        }
    }
public static void main(String[] args) {
        final TestSynchronized syncObject = new TestSynchronized();
        final TestSynchronized otherSyncObject = new TestSynchronized();
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                syncObject.sayHello(Thread.currentThread().getName());
            }
        }, "线程1号");
        Thread t3 = new Thread(new Runnable() {
            public void run() { 
                otherSyncObject.sayHello(Thread.currentThread().getName());

            }
        }, "不是线程3号");
t1.start();
        t3.start();
        }
```
####3.5.2 输出
``` Console
Hello 线程1号!现在时间是:2014年01月09日10时30分51秒
Hello 不是线程3号!现在时间是:2014年01月09日10时30分51秒
```
####3.5.3 分析
和线程一号同时执行可以看出，同一个类不同实例对象的synchronized描述的方法不会互斥。
也说明了，对象锁无法解决所有同步的问题。这时候就需要类锁。
###3.6 类锁
####3.6.1 代码
```java
public static synchronized void sayStaticClass(String name){
        System.out.println("StaticClass " + name + "!现在时间是:" + getCurrentTime());
        try {
            Thread.sleep(5000);
        }
        catch(InterruptedException e) {
        }
    }
    public void sayClass(String name){
        synchronized (TestSynchronized.class) {
            System.out.println("Class " + name + "!现在时间是:" + getCurrentTime());
            try {
                Thread.sleep(5000);
            }
            catch(InterruptedException e) {
            }
        }
    }
    public void sayGetClass(String name){
        synchronized (this.getClass()) {
            System.out.println("GetClass " + name + "!现在时间是:" + getCurrentTime());
            try {
                Thread.sleep(5000);
            }
            catch(InterruptedException e) {
            }
        }
    }
public static void main(String[] args) {
        final TestSynchronized syncObject = new TestSynchronized();
        Thread t6 = new Thread(new Runnable() {
            public void run() {
                TestSynchronized.sayStaticClass(Thread.currentThread().getName());
                
            }
        }, "线程静态方法类锁");
        Thread t7 = new Thread(new Runnable() {
            public void run() {
                syncObject.sayClass(Thread.currentThread().getName());
                
            }
        }, "线程类锁");
        Thread t8 = new Thread(new Runnable() {
            public void run() {
                syncObject.sayGetClass(Thread.currentThread().getName());

            }
        }, "对象getClass方法线程类锁");
        t6.start();
        t7.start();
        t8.start();
    }
```
####3.6.2 输出
``` Console
StaticClass 线程静态方法类锁!现在时间是:2014年01月09日10时40分10秒
Class 线程类锁!现在时间是:2014年01月09日10时40分15秒
GetClass 对象getClass方法线程类锁!现在时间是:2014年01月09日10时40分20秒
```
####3.6.3 分析
可以看出，加了synchronized的静态方法与synchronized(Xxx.class/xxx.getClass())这三种方式互斥。所以他们获取的是同一个锁，也就是Xxx.class这个Xxx类的静态成员的对象锁。
由于他可以对该类的所有实例进行同步，所以我们称之为类锁。
##4 总结
搞清楚synchronized锁定的是哪个对象，对我们设计更安全的多线程程序是很有帮助的。
通过本文我们可以知道synchronized有两种锁的方式，一种是对象锁，一种是类锁。利用这两种锁，我们可以很好的实现对共享数据的同步操作。
