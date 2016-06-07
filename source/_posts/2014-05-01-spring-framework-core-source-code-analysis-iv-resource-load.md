---
layout: post
title: 'Spring Framework的核心源码解读（四）-Resource加载'
date: 2014-05-01 15:37
comments: true
categories: [java, spring, resource, IoC, 源码分析, Resource加载]
---
##4.2 Resource载入
载入过程其实就是把通过配置文件、注解等方式定义的数据转换成IoC容器所需要的数据结构BeanDefinition。IoC容器对Bean的管理和依赖注入功能的实现，是通过对其持有的BeanDefinition进行各种相关操作来完成的。这些BeanDefinition数据在IoC容器中通过一个HashMap来保持和维护。如果有需要，你也可以换成别的方式对数据进行保持和维护。
上一章节我已经讲到了Resource的定位，这里已经返回了ServletContextResource，这里接着立马调用loadBeanDefinitions(resources)进行Resource的载入。
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
			//Resource的载入
			int loadCount = **loadBeanDefinitions** (resource);
			if (actualResources != null) {
				actualResources.add(resource);
			}
			//……省略部分代码
			return loadCount;
		}
	}
public int loadBeanDefinitions(Resource... resources) throws BeanDefinitionStoreException {
		Assert.notNull(resources, "Resource array must not be null");
		int counter = 0;
		for (Resource resource : resources) {
			//钩子方法
			counter += loadBeanDefinitions(resource);
		}
		return counter;
	}
上列代码的loadBeanDefinitions(resource)是钩子方法，是通过子类实现的，这里我们直接跳到XmlBeanDefinitionReader类去看看具体实现
```java XmlBeanDefinitionReader.java
//这里是调用的入口
public int loadBeanDefinitions(Resource resource) throws BeanDefinitionStoreException {
		return loadBeanDefinitions(new EncodedResource(resource));
	}
//这里是载入XML形式的BeanDefinition的地方
public int loadBeanDefinitions(EncodedResource encodedResource) throws BeanDefinitionStoreException {
	//……省略部分日志代码，前置监测等
		try {
//这里是载入XML形式的BeanDefinition的地方
			InputStream inputStream = encodedResource.getResource().getInputStream();
			try {
				InputSource inputSource = new InputSource(inputStream);
				if (encodedResource.getEncoding() != null) {
					inputSource.setEncoding(encodedResource.getEncoding());
				}
				return doLoadBeanDefinitions(inputSource, encodedResource.getResource());
			}
			finally {
				inputStream.close();
			}
		}
//省略异常处理
	}
//具体的读取过程可以在doLoadBeanDefinitions方法中找到
//这是从特定的XML文件中实际载入BeanDefinition的地方
protected int doLoadBeanDefinitions(InputSource inputSource, Resource resource) throws BeanDefinitionStoreException {
		try {
			int validationMode = getValidationModeForResource(resource);
		//这里取得XML文件的Document对象，这个解析过程是由 documentLoader完成的这个documentLoader是DefaultDocumentLoader,在定义documentLoader的地方创建
			Document doc = this.documentLoader.loadDocument(
					inputSource, getEntityResolver(), this.errorHandler, validationMode, isNamespaceAware());
			//这里启动的是对BeanDefinition解析的详细过程，这个解析会使用到Spring的Bean配置规则，是我们下面需要详细讲解的内容
			return registerBeanDefinitions(doc, resource);
		}
//省略异常处理
	}
public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {
//这里得到 BeanDefinitionDocumentReader来对XML的BeanDefinition进行解析
		BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();
		documentReader.setEnvironment(this.getEnvironment());
		int countBefore = getRegistry().getBeanDefinitionCount();
//具体的解析过程在这个registerBeanDefinitions中完成
documentReader.registerBeanDefinitions(doc, createReaderContext(resource));
//统计载入bean的数量
		return getRegistry().getBeanDefinitionCount() - countBefore;
	}
```
BeanDefinition的载入分成两部分，首先通过调用XML的解析器得到document对象，但这些document对象并没有按照Spring的Bean规则进行解析。这里不研究Document是怎么解析的，完成解析后按照Spring的Bean规则进行解析，这个过程是在 documentReader的registerBeanDefinitions()中实现的。
```java DefaultBeanDefinitionDocumentReader.java
public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		//省略过程
		doRegisterBeanDefinitions(root);
	}
protected void doRegisterBeanDefinitions(Element root) {
		//省略过程
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createHelper(readerContext, root, parent);
		preProcessXml(root);
		parseBeanDefinitions(root, this.delegate);
		//省略过程
	}
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
		//省略部分代码做两件事，1、遍历root，得到的ele对应在Spring BeanDefinition中定义的XML元素。2、判断delegate的命名空间是否匹配
		parseDefaultElement(ele, delegate);
	}
	private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
		//省略部分代码，大概就是根据节点类型的不同，调用不同的处理方法，我们只研究bean是如何加载的。
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
	} 
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
		// BeanDefinitionHolder是BeanDefinition对象的封装类，封装了BeanDefinition，Bean的名字和别名。用它来完成向IoC容器注册。得到这个BeanDefinitionHolder就意味着BeanDefinition是通过BeanDefinitionParserDelegate对XML元素的信息按照Spring的Bean规则进行解析得到的
		BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
		if (bdHolder != null) {
			bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
			try {
				// 这里是向IoC容器注册解析得到BeanDefinition的地方			BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());
			}
			catch (BeanDefinitionStoreException ex) {
				//异常处理
			}
			// 在BeanDefinition向IoC容器注册完以后，发送消息
			getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
		}
	}
```
DefaultBeanDefinitionDocumentReader类里基本是对各种Spring Bean定义规则的处理。具体的处理（比如找到bean元素，将id、name、alaise等信息读取出来存到BeanDefinitionHolder中去）委托给BeanDefinitionParserDelegate来完成。
```java BeanDefinitionParserDelegate.java
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
		return parseBeanDefinitionElement(ele, null);
	}
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
		//这里取得在<bean>元素中定义的id、name和aliase属性的值
		String id = ele.getAttribute(ID_ATTRIBUTE);
		String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

		List<Stringaliases = new ArrayList<String>();
		if (StringUtils.hasLength(nameAttr)) {
			String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
			aliases.addAll(Arrays.asList(nameArr));
		}

		String beanName = id;
		if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
			beanName = aliases.remove(0);
			if (logger.isDebugEnabled()) {
				logger.debug("No XML 'id' specified - using '" + beanName +
						"' as bean name and " + aliases + " as aliases");
			}
		}

		if (containingBean == null) {
			checkNameUniqueness(beanName, aliases, ele);
		}
//这个方法会引发对Bean元素的详细解析,这个数据对象中封装的数据大多都是与<bean>定义相关的，也有很多就是我们在定义Bean时看到的那些Spring标记，比如常见的init-method、destroy-method、factory-method，等等
		AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
		if (beanDefinition != null) {
			if (!StringUtils.hasText(beanName)) {
				try {
					if (containingBean != null) {
						beanName = BeanDefinitionReaderUtils.generateBeanName(
								beanDefinition, this.readerContext.getRegistry(), true);
					}
					else {
						beanName = this.readerContext.generateBeanName(beanDefinition);
						String beanClassName = beanDefinition.getBeanClassName();
						if (beanClassName != null &&
								beanName.startsWith(beanClassName) && beanName.length() beanClassName.length() &&
								!this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
							aliases.add(beanClassName);
						}
					}
					if (logger.isDebugEnabled()) {
						logger.debug("Neither XML 'id' nor 'name' specified - " +
								"using generated bean name [" + beanName + "]");
					}
				}
				catch (Exception ex) {
					error(ex.getMessage(), ele);
					return null;
				}
			}
			String[] aliasesArray = StringUtils.toStringArray(aliases);
			return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
		}

		return null;
	}
```
AbstractBeanDefinition是BeanDefinition非常重要的一个实现类，beanClass、description、lazyInit这些属性都是在配置bean时经常碰到的，也都集中在这里。我们来看看他是如何解析的。
```java BeanDefinitionParserDelegate.java
public AbstractBeanDefinition parseBeanDefinitionElement(
			Element ele, String beanName, BeanDefinition containingBean) {

		this.parseState.push(new BeanEntry(beanName));
		//这里只读取定义的<bean>中设置的class名字，然后载入到BeanDefinition中去，只是做个
		//记录，并不涉及对象的实例化过程，对象的实例化实际上是在依赖注入时完成的
		String className = null;
		if (ele.hasAttribute(CLASS_ATTRIBUTE)) {
			className = ele.getAttribute(CLASS_ATTRIBUTE).trim();
		}

		try {
			String parent = null;
			if (ele.hasAttribute(PARENT_ATTRIBUTE)) {
				parent = ele.getAttribute(PARENT_ATTRIBUTE);
			}
			//这里生成需要的BeanDefinition对象，为Bean定义信息的载入做准备
			AbstractBeanDefinition bd = createBeanDefinition(className, parent);
			//这里对当前的Bean元素进行属性解析，并设置description的信息
			parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
			bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));
			//从名字可以清楚地看到，这里是对各种<bean>元素的信息进行解析的地方
			parseMetaElements(ele, bd);
			parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
			parseReplacedMethodSubElements(ele, bd.getMethodOverrides());
			//解析<bean>的构造函数设置
			parseConstructorArgElements(ele, bd);
			//解析<bean>的property设置
			parsePropertyElements(ele, bd);
			parseQualifierElements(ele, bd);

			bd.setResource(this.readerContext.getResource());
			bd.setSource(extractSource(ele));

			return bd;
		}
		//下面这些异常是在配置Bean出现问题时经常会看到的，在createBeanDefinition时进行的，
		//会检查Bean的class设置是否正确，比如这个类是否能找到
		catch (ClassNotFoundException ex) {
			error("Bean class [" + className + "] not found", ele, ex);
		}
		catch (NoClassDefFoundError err) {
			error("Class that bean class [" + className + "] depends on not found", ele, err);
		}
		catch (Throwable ex) {
			error("Unexpected failure during bean definition parsing", ele, ex);
		}
		finally {
			this.parseState.pop();
		}

		return null;
	}
```
这里就不详细对每个属性如何载入的进行分析了，如果有兴趣的可以自己去研究下，特别是Property的解析还是挺有意思的，因为他可能是值，可能是ref，可能有子元素，子元素可能是Array、List、Set、Map、Prop等。
经过这样逐层地解析，我们在XML文件中定义的BeanDefinition就被整个载入到了IoC容器中，并在容器中建立了数据映射。在IoC容器中建立了对应的数据结构，或者说可以看成是POJO对象在IoC容器中的抽象，这些数据结构可以以AbstractBeanDefinition为入口，让IoC容器执行索引、查询和操作。简单的POJO操作背后其实蕴含着一个复杂的抽象过程，经过以上的载入过程，IoC容器大致完成了管理Bean对象的数据准备工作（或者说是初始化过程）。但是，重要的依赖注入实际上在这个时候还没有发生，现在，在IoC容器BeanDefinition中存在的还只是一些静态的配置信息。严格地说，这时候的容器还没有完全起作用，要完全发挥容器的作用，还需完成数据向容器的注册。
