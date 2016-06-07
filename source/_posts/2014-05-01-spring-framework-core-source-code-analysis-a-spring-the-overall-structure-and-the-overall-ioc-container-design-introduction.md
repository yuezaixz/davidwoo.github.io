---
layout: post
title: 'Spring Framework的核心源码解读（一）-Spring整体结构以及IoC容器设计整体介绍'
date: 2014-05-01 15:19
comments: true
categories: [java, spring, application, IoC, 源码分析, beanfactory]
---
#前言
这么久没写博客？我干嘛去了呢？最主要还是项目比较忙，没有大段的时间来看书、研究、写成博文。仅有的时间也只能翻阅下书籍或者技术文章，而自己又不想天天写些心灵鸡汤类的东西。那么就只好罢工这么久了咯。终于遇到五一有空了，安静下来看看书，写写博文。最近一段时间都是Spring 源码相关的，该系列文章内容大多为《Spring技术内幕》这本书的笔记，这本书很不错，推荐阅读。也会发一段Thoughtwork的那个代码招聘的FizzBuzzWhizz相关的内容。
敬请期待！

#1 概述
##1.1 Spring的整体架构
Spring 总共有十几个组件，其中核心组件只有三个：Core、Context 和 Beans。AOP、Web等都属于核心上层的特性功能。本文将对Spring核心内容的设计进行分析。
![](http://i1.tietuku.com/154191d85dc03b0c.jpg)

#2 IoC容器的实现
##2.1 控制反转（IOC）
实际应用的程序基本上都是由多个类彼此合作来实现业务逻辑，这使得每个对象都需要与其合作的对象的引用。但是这样的控制就导致了程序的耦合度非常大，不易于复用，更不易于做单元测试。
Martin Flowler 提出了“那些方面的控制被反转了？”这个问题。他得出的结论是：依赖对象的获得被反转了。基于这个结论，他为控制反转创造了一个更好的名字：依赖注入。
依赖倒置也是设计模式六大原则之一。在Spring中IoC容器就是这个模式的载体，它可以通过容器对对象进行管理，在对象生成或初始化时直接将数据注入到需要其依赖的对象中。这也是Spring的核心。
##2.2 BeanFactory和ApplicationContext的区别
Spring中容器要分两类，一类是实现BeanFactory接口的简单容器系列，这类容器只实现了容器的最基本功能；另一个是ApplicationContext应用上下文，也就是我们最常用的，项目中的配置文件大多数都是ApplicationContext-xxx.xml，它作为容器的高级形态而存在。
用生活中的容器水杯举例，BeanFactory就定义了可以作为水桶的基本功能（装水）。从下图可以看出，Spring容器的基本功能就是获取bean、判断bean是否存在、是否为单例、是否为原型、类型是否匹配、获取bean的类型、获取bean的别名。因此BeanFactory就是Spring的基本接口。
 ![](http://i1.tietuku.com/75daf2c913bddbdd.jpg)
那么BeanFactory是杯子，那么BeanDefinition就是水。BeanDefinition是用来管理基于Spring的应用中的各种对象以及它们之间的相互依赖关系，就Spring中Bean的主要数据结构，是设计出来水的抽象。如下代码。
```java DefaultListableBeanFactory.java
/** Map of bean definition objects, keyed by bean name */
	private final Map<String, BeanDefinitionbeanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>();

	/** List of bean definition names, in registration order */
	private final List<StringbeanDefinitionNames = new ArrayList<String>();
```
而ApplicationContext则提供了很多特殊的特性，从下图ApplicationContext接口的继承关系可以看出，它除了容器所需的基本特性外，还有其他许多附加特性，比如国际化的消息访问、资源访问、事件传播、载入多个上下文等。
 ![](http://i1.tietuku.com/84eaba13257e4c38.jpg)
##2.3 IoC容器的接口设计
下图描述了IoC容器中的主要接口设计（图片拍自Spring技术内幕一书）。
 ![](http://i1.tietuku.com/8f45e01439fe6212.jpg)
###2.3.1 BeanFactory-HierarchicalBeanFactory-ConfigurableBeanFactory
在这条设计路径中，HierarchicalBeanFactory增加了getParentBeanFactory()的功能，是BeanFactory具备了双亲IoC容器的管理功能，在ConfigurableBeanFactory中主要定义了一些对BeanFactory的配置功能，比如通过setParentBeanFactory()设置双亲IoC容器，通过addBeanPostProcessor()定义了Bean后置处理器等。
###2.3.2 ApplicationContext路线
该设计路线以ApplicationContext应用上下文接口为核心的接口设计，这里涉及的主要接口设计有，从BeanFactory到ListableBeanFactory，再到ApplicationContext，再到我们常用的WebApplicationContext或者ConfigurableApplicationContext接口。
在这个接口体系中，ListableBeanFactory和HierarchicalBeanFactory两个接口，连接BeanFactory接口定义和ApplicationConext应用上下文的接口定义。在ListableBeanFactory接口中，细化了许多BeanFactory的接口功能，比如定义了getBeanDefinitionNames()接口方法；对于HierarchicalBeanFactory接口，我们在前文中已经提到过；对于ApplicationContext接口，它通过继承MessageSource、ResourceLoader、ApplicationEventPublisher接口，在BeanFactory简单IoC容器的基础上添加了许多对高级容器的特性的支持，这些特性我们之前也都提到过了。
总体来说 ApplicationContext 必须要完成以下几件事：

*	标识一个应用环境
*	利用 BeanFactory 创建 Bean 对象
*	保存对象关系表
*	能够捕获各种事件

