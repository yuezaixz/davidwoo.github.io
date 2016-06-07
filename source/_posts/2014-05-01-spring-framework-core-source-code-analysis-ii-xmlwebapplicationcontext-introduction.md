---
layout: post
title: 'Spring Framework的核心源码解读（二）-XmlWebApplicationContext介绍'
date: 2014-05-01 15:29
comments: true
categories: [java, spring, IoC, XmlWebApplicationContext]
---
#3 XmlWebApplicationContext
XmlWebApplicationContext是开发web应用最常用的上下文环境。
##3.1 XmlWebApplicationContext的特性
XmlWebApplicationContext提供了IoC容器对Web环境ServletAPI的集成，并提供Xml解析器，从而可以从xml文件中获取BeanDefinition数据。
 ![](http://i1.tietuku.com/2ba01b6872beb1db.jpg)

##3.2 XmlWebApplicationContext的加载
在系统中通过配置servlet或者listener（如下代码）就能让web系统从xml中加载BeanDefinition数据。
```xml web.xml
    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>
```
那再让我们看看ContextLoaderListener的源码片段。从下面的代码可以看出，该监听会判断上下载加载器是否存在，不存在就进行初始化。
```java ContextLoaderListener.java
this.contextLoader = createContextLoader();
		if (this.contextLoader == null) {
			this.contextLoader = this;
		}
		this.contextLoader.initWebApplicationContext(event.getServletContext());
```
而initWebApplicationContext()方法中，又通过BeanUtils.instantiateClass(contextClass)来获取ApplicationContext，我们再继续寻找源头。在ContextLoader类中我们找到了determineContextClass()方法。
```java ContextLoader.java
protected Class<?determineContextClass(ServletContext servletContext) {
		String contextClassName = servletContext.getInitParameter(CONTEXT_CLASS_PARAM);
		if (contextClassName != null) {
			//通过类加载器加载该类并返回
		}
		else {
			contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());
			//通过类加载器加载该类并返回
		}
	}
```
servletContext.getInitParameter是取web.xml里的配置参数，意思就是先看看我们有没有自己定义context的类，如果我们定义了，直接返回这个class。否则，去defaultStrategies里去取,WebApplicationContext的类名作为key。这里我Ctry+O搜了下defaultStrategies，发现了源头。
```java ContextLoader.java
private static final String DEFAULT_STRATEGIES_PATH = "ContextLoader.properties";
private static final Properties defaultStrategies;

	static {
		try {
			ClassPathResource resource = new ClassPathResource(DEFAULT_STRATEGIES_PATH, ContextLoader.class);
			defaultStrategies = PropertiesLoaderUtils.loadProperties(resource);
		}
		catch (IOException ex) {
			throw new IllegalStateException("Could not load 'ContextLoader.properties': " + ex.getMessage());
		}
	}
```
这里定义的就是我们需要的类。
![](http://i1.tietuku.com/5e8030512d81ede8.jpg)