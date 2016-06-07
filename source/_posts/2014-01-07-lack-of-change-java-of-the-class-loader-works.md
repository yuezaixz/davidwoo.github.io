---
layout: post
title: '不足则改之---Java类装载器的工作原理'
date: 2014-01-07 16:14
comments: true
categories: [java, jvm, classloader]
---
很高兴，晚上回来路上捡到了100块钱。我交给了警察叔叔处理，终于拾金不昧了一把。T.T以前都是特么的捡不到钱啊。
#不足
今天和一位技术上的前辈聊了一个小时，除了了解到自己需求等各方面的不足，还发现了自己一个薄弱点，那就是对java底层是如何运行的还不是很了解。早在刚毕业的时候，我还会去了解JVM的运行机制、一个java文件如何编译成字节码文件、如何通过类加载器加载、如何解释运行、JIT，包括JVM实例的堆、程序计数器，包括每个方法的栈。
但是现在，对于这些东西，由于长期专注于web，并且心猿意马，早就关注其他新的技术，导致忘了这些老本行。这些东西早就有所了解而却说不出个所以然来。
因此，晚上回来就痛彻心扉的决定要把这块已经丢失的骨头再啃起来。本来计划今天要把之前写好的javascript实战篇的最后一篇博文整理下发出来，但是，我决定还是先把JVM相关的搞定了把。
#JVM的核心

![1.gif](http://user-image.logdown.io/user/3769/blog/3827/post/174805/kWlOwaDJSbGCe5LDNuxi_1.gif)
#类加载器
可以看到，类加载器在其中占了很重要的一大块，今天就先把类装载器这老骨头给重新啃一啃。
##1 基础概念
###1.1 什么是类加载器
类加载器是一个用来加载类文件的类。
默认的类加载器有三种：Bootstrap类加载器、Extension类加载器和System类加载器。之后章节具体介绍每种类加载器从哪里加载类。
###1.2 类加载器的作用
在Java虚拟机的执行引擎执行的时候加载类。
Java源代码通过javac编译器编译成类文件。然后JVM来执行类文件中的字节码（注意：JIT会把热点代码编译成机器码进行执行，为了提供运行时效率）来执行程序。
类加载器负责加载文件系统、网络或其他来源的类文件。
###1.3 类加载器的三个机制
Java类加载器的工作原理基于以下三个机制：委托、可见性和单一性。
####1.3.1 委托
委托机制是指将加载一个类的请求交给父加载器，如果这个父类加载器不能找到或者加载这个类，那么再加载它。
####1.3.2 可见性
子类的加载器可以看见所有父类的加载器加载的类，而父类加载器看不到子类加载器加载的类。
####1.3.3 单一性
指一个类仅加载一次，该机制是通过委托机制确保子类加载器不会再次加载父类加载器加载过的类。
####1.3.4 例子
通过后面章节介绍了各个类加载器及其工作原理后再结合例子介绍。
###1.4 Bootstrap类加载器
####1.4.1 概念
该类加载器负责加载rt.jar中的JDK类文件，他是所有类加载器的父加载器。该类加载器没有父类加载器，如果你调用String.class.getClassLoader()，会返回null。
```java
        System.out.println(String.class.getClassLoader());
        System.out.println(TestBootstrapCL.class.getClassLoader());
```
``` Console
null
sun.misc.Launcher$AppClassLoader@82ba41
```
因此，bootstrap类加载器被称为初始类加载器。
####1.4.2 加载类文件的地方
JRE/lib/rt.jar
###1.5 Extension类加载器
####1.5.1 概念
Extension类加载器负责加载标准扩展目录下面的类。他加载类之前，会先委托他的父类加载器（Bootstrap）进行加载，没有加载到再从jre/lib/ext目录下或者java.ext.dirs系统属性定义的目录下加载类。
这样就可以使得编写程序变得简单，只需把JAR文件拷贝到扩展目录下面即可，类加载器会自动的在下面查找。
Extension加载器由sun.misc.Launcher$ExtClassLoader实现。
####1.5.2 加载类文件的地方
jre/lib/ext
###1.6 System类加载器
####1.6.1 概念
System类加载器是默认的类加载器，他会从CLASSPATH目录下查找类。CLASSPATH可以是系统CLASSPATH变量中设定的，可以是命令行-classpath –cp后的参数定义的，也可以是jar文件中的Mainfest的classpath属性定义的。SYSTEM类加载器是通过sun.misc.Launcher$AppClassLoader在实现的。
####1.6.2 加载类文件的地方
CLASSPATH
##2 工作原理
把三种类加载器说完了，现在补充下之前没说完的类加载器工作原理的三个机制。
###2.1 委托机制
当一个类加载和初始化的时候，类尽在有需要加载的时候被加载。假设你有一个应用需要的类家Abc.class，首先加载这个类的请求由System类加载器委托给Extension类加载器，然后再委托给Bootstrap类加载器。Bootstrap会先看看rt.jar中有没有这个类，没有的话再把请求返回给extension类加载器，如果这个类被Extension类加载器找到了，那么他将被加载，而System不会加载这个类。如果没有找到，那么System就回去CLASSPATH中寻找。
###2.2 可见性机制
根据可见性机制，子类加载器可以看到父类加载器加载的类，而反之则不行。
看以下例子
```java
try {
            System.out.println("TestBootstrapCL.class.getClassLoader():"+TestBootstrapCL.class.getClassLoader());
            System.out.println("TestBootstrapCL.class.getClassLoader().getParent():"+TestBootstrapCL.class.getClassLoader().getParent());
            Class.forName("com.dragonsoft.test.d140107", true, TestBootstrapCL.class.getClassLoader().getParent());
        }
        catch(ClassNotFoundException e) {
            Logger.getLogger(TestBootstrapCL.class.getName()).log(Level.SEVERE, e.toString());
        }
```

``` Console
TestBootstrapCL.class.getClassLoader():sun.misc.Launcher$AppClassLoader@82ba41
TestBootstrapCL.class.getClassLoader().getParent():sun.misc.Launcher$ExtClassLoader@923e30
2014-1-7 23:58:48 com.dragonsoft.test.d140107.TestBootstrapCL main
严重: java.lang.ClassNotFoundException: com.dragonsoft.test.d140107
```
可以看出TestBootstrapCL的类加载器是SYSTEM类加载器。TestBootstrapCL类加载器的父类加载器是Extension。而Extension类加载器无法加载到TestBootstrapCL类，因为可见性原则限制父类加载器是无法加载子类加载器的。
###2.3 单一性原则
根据该机制，父类加载器加载过的类不能被子类加载器加载第二次。

##3 显示类加载
Class.forName(className)和 Class.forName(className,initialized,classloader)。在JAVA创建JDBC连接的时候，都会要显示的加载数据库驱动类。
类加载是通过调用CLassLoader的loadClass()方法，loadClass()再调用findClass()。如果最后都没找到该类，那就会报错。找到的话就调用defineClass()将字节码转化成类实例，然后返回。


##4 显示类加载的作用
Class.forName(className)在返回一个Class的同时，他会执行该类的静态代码段。
```java
Class clazz=Class.forName(driverClass);
Driver driver=(java.sql.Driver)clazz.newInstance();
DriverManager.registerDriver(driver);
```
比如上面这段代码，我就可以把driverClass写在配置文件中，写在数据库中。只要这些驱动类都是java.sql.Driver实现类的，我就能把他们动态的加载到应用中。是不是有点动态语言的风范？这也就是java用代理事先动态语言的一种方式。


