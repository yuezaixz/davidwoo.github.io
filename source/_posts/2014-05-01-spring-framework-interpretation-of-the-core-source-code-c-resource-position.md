---
layout: post
title: 'Spring Framework的核心源码解读（三）-IoC容器初始化及Resource定位'
date: 2014-05-01 15:32
comments: true
categories: [java, spring, resource, IoC, 源码分析, Resource定位]
---
#4 IoC容器的初始化过程
本章继续上个章节，以为例子，讲述下IoC容器的初始化。具体来说，这个启动包括BeanDefinition的Resource定位、载入和注册三个基本过程。
Spring把这三个过程分开，使用不同的模块来了完成，如使用相应的ResourceLoader、BeanDefinitionReader等模块，通过这样的设计方式，可以让使用者更灵活地对这三个过程进行剪裁或扩展，定义出最适合自己的IoC容器初始化过程。
##4.1 Resource定位
先回到我们经常使用的ApplicationContext上来，对于不同的实现类，就提供不同的Resource读入功能，比如FileSystemXmlApplicationContext是从文件系统载入Resource，ClassPathXmlApplicationContext是从Class Path载入Resource，而XmlWebApplicationContext则是从Web容器中载入Resource。下UML图表明了context和resource是如何建立关系的。![](http://i1.tietuku.com/42fa2ddb2a984f88.jpg) 
Context 是把资源的加载、解析和描述工作委托给了 ResourcePatternResolver 类来完成，他相当于一个接头人，他把资源的加载、解析和资源的定义整合在一起便于其他组件使用。
不同于FileSystemXmlApplicationContext，XmlWebApplicationContext的refresh不是通过构造函数作为入口的，因为它存在于Web容器中，所以他是从监听器中初始化启动的。
首先先看一个时序图，看看Web应用启动的过程。
 ![](http://i1.tietuku.com/bac88c46e6d1abd4s.jpg)
对容器的启动来说，refresh是一个很重要的方法，在ContextLoader中通过调用refresh()方法启动整个调用。暂时先跳过之前已经说过的XmlWebApplicationContext加载过程和context初始化过程，直接从refresh()开始分析。这段代码主要包含这样几个步骤：

* 构建 BeanFactory，以便于产生所需的“演员”；
* 注册可能感兴趣的事件，比如MessageSource、PostProcessor等；
* 创建 Bean 实例对象
* 触发被监听的事件

```java
//这里是 refresh 也就是刷新配置
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// 准备需要启动的容器上下文，包括启动时间、当前容器状态等
			prepareRefresh();
			// 在子类中创建、启动refreshBeanFactory()的地方
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
			// 初始化BeanFactory，加载容器所需要的行为
			prepareBeanFactory(beanFactory);
			try {
				//设置BeanFactoy的后置处理
				postProcessBeanFactory(beanFactory);
				//调用BeanFactory的后处理器，这些后处理器是在Bean定义中向容器注册的
invokeBeanFactoryPostProcessors(beanFactory);
//注册Bean的后处理器，在Bean创建过程中调用。
				registerBeanPostProcessors(beanFactory);
//对上下文中的消息源进行初始化
				initMessageSource();
//初始化上下文中的事件机制
				initApplicationEventMulticaster();
//初始化其他的特殊Bean
				onRefresh();
				//检查监听Bean并且将这些Bean向容器注册
				registerListeners();
				//实例化所有的(non-lazy-init)单件
				finishBeanFactoryInitialization(beanFactory);
				//发布容器事件，结束Refresh过程
				finishRefresh();
			}
			catch (BeansException ex) {
		//为防止Bean资源占用，在异常处理中，销毁已经在前面过程中生成的单件Bean
				destroyBeans();
				// 重置 'active'标志
				cancelRefresh(ex);
				throw ex;
			}
		}
	}
```

下列代码为AbstractRefreshableApplicationContext的refreshBeanFactory ()方法。这个方法实现了 AbstractApplicationContext 的抽象方法 refreshBeanFactory，这段代码清楚的说明了 IoC容器 的创建过程。注意容器对象的类型的变化，前面介绍了他有很多子类，在什么情况下使用不同的子类这非常关键。在 refreshBeanFactory 方法中有一行 loadBeanDefinitions(beanFactory)，这个方法将开始加载、解析 Bean 的定义，也就是把用户定义的数据结构转化为 IoC 容器中的特定数据结构。
```java AbstractRefreshableApplicationContext.java
@Override
	protected final void refreshBeanFactory() throws BeansException {
	    //这里判断，如果已经建立BeanFactory，则销毁并关闭该BeanFactory
		if (hasBeanFactory()) {
			destroyBeans();
			closeBeanFactory();
		}
		//这里创建并设置持有的DefaultListableBeanFactory的地方同时调用
		//loadBeanDefinitions再载入BeanDefinition的信息
		try {
			DefaultListableBeanFactory beanFactory = createBeanFactory();
			beanFactory.setSerializationId(getId());
			customizeBeanFactory(beanFactory);
	//该方法是钩子方法，直接调用XmlWebApplicationContext中的实现
	**loadBeanDefinitions(beanFactory);**
			synchronized (this.beanFactoryMonitor) {
				this.beanFactory = beanFactory;
			}
		}
		catch (IOException ex) {
			throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);
		}
	}
//这就是在上下文中创建DefaultListableBeanFactory的地方，而getInternalParentBeanFactory的具体实现可以参看AbstractApplicationContext中的实现，会根据容器已有的双亲IoC容器的信息来生成DefaultListableBeanFactory的双亲容器 
	protected DefaultListableBeanFactory createBeanFactory() {
		return new DefaultListableBeanFactory(getInternalParentBeanFactory());
	}
```
前面已经说了有不同子类的实现，那么接着就到具体实现类中去看看。这里因为有许多载入方式，所以通过一个抽象函数（钩子方法）把具体的实现委托给各个子类来实现。这里使用的是xml的载入方式。通过XmlBeanDefinitionReader载入Bean的地方。
```java XmlWebApplicationContext.java
@Override
	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
		// 创建一个XmlBeanDefinitionReader并设置beanFactory.
		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);

		// 通过上下文环境配置reader
		beanDefinitionReader.setEnvironment(this.getEnvironment());
		beanDefinitionReader.setResourceLoader(this);
		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));

		// 钩子方法，子类不重载就不做任何操作
		initBeanDefinitionReader(beanDefinitionReader);
		loadBeanDefinitions(beanDefinitionReader);
	}
protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws IOException {
//调用基类的获取配置路径方法
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			for (String configLocation : configLocations) {
				reader.loadBeanDefinitions(configLocation);
			}
		}
	}
/**
	 * 默认的配置文件为 "/WEB-INF/applicationContext.xml"
	 */
	@Override
	protected String[] getDefaultConfigLocations() {
		if (getNamespace() != null) {
			return new String[] {DEFAULT_CONFIG_LOCATION_PREFIX + getNamespace() + DEFAULT_CONFIG_LOCATION_SUFFIX};
		}
		else {
			return new String[] {DEFAULT_CONFIG_LOCATION};
		}
	}
```
```java AbstractRefreshableConfigApplicationContext.java
protected String[] getConfigLocations() {
//基类中又调用钩子方法
		return (this.configLocations != null ? this.configLocations : getDefaultConfigLocations());
	}
```
具体资源载入是在XmlBeanDefinitionReader读入BeanDefinition时完成的。
```java XmlBeanDefinitionReader.java
public int loadBeanDefinitions(String location, Set<ResourceactualResources) throws BeanDefinitionStoreException {
		ResourceLoader resourceLoader = getResourceLoader();
		if (resourceLoader == null) {
			//……异常，省略
		}

		if (resourceLoader instanceof ResourcePatternResolver) {
			//……这里很少用的，直接跳过
		}
		else {
			// 通过调用DefaultResourceLoader的getResources完成具体的Resource定位	
			Resource resource = resourceLoader.getResource(location);
			int loadCount = loadBeanDefinitions(resource);
			if (actualResources != null) {
				actualResources.add(resource);
			}
			//……省略部分代码
			return loadCount;
		}
	}
```
对于具体取得resource的过程，我们可以看看DefaultResourceLoader是怎么完成的。
```java DefaultResourceLoader.java
public Resource getResource(String location) {
		Assert.notNull(location, "Location must not be null");
		//先处理带有classpath表示的Resource
		if (location.startsWith(CLASSPATH_URL_PREFIX)) {
			return new ClassPathResource(location.substring(CLASSPATH_URL_PREFIX.length()), getClassLoader());
		}
		else {
			try {
				// 再尝试处理URL表示的Resource
				URL url = new URL(location);
				return new UrlResource(url);
			}
			catch (MalformedURLException ex) {
				// 如果既不是classpath，又不是URL标示，则把getResource的任务交给getResourceByPath，这个方法有默认实现，但基本用子类实现
				return getResourceByPath(location);
			}
		}
	}
```
这里使用的是基类AbstractRefreshableWebApplicationContext的实现方式。返回的是ServletContextResource类型的Resource。
```java AbstractRefreshableWebApplicationContext.java
@Override
	protected Resource getResourceByPath(String path) {
		return new ServletContextResource(this.servletContext, path);
	}
```
如果是其他的ApplicationContext，那么会对应生成其他种类的Resource，比如ClassPathResource、ServletContextResource等。从下图可以看出Resource接口的设计。在BeanDefinition定位完成的基础上，就可以通过返回的Resource对象来进行BeanDefinition的载入了。现在BeanDefinition所需要的数据还并未读入，这些读入将在下一章节，Resource载入中说明
![](http://i1.tietuku.com/d1d7292220faf75a.jpg)