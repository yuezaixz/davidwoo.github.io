---
layout: post
title: 'Spring Framework的核心源码解读（五）-Resource注册'
date: 2014-05-01 15:39
comments: true
categories: [java, spring, resource, IoC, 源码分析, Resource注册]
---
##4.3 Resource注册
到目前为止，我们已经准备好了Spring需要的数据结构BeanDefinition了，但此时这些数据还不能供IoC容器直接使用，需要在IoC容器中对这些BeanDefinition数据进行注册。
前面提到的BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry())调用就是进行BeanDefinition注册。
```java BeanDefinitionReaderUtils.java
public static void registerBeanDefinition(
			BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
			throws BeanDefinitionStoreException {
		String beanName = definitionHolder.getBeanName();
		registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
		String[] aliases = definitionHolder.getAliases();
		if (aliases != null) {
			for (String aliase : aliases) {
				registry.registerAlias(beanName, aliase);
			}
		}
	}
```
这里又调用了registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition())方法，我们这里的registry就是DefaultListableBeanFactory，他实现了BeanDefinitionRegistry接口的该方法。DefaultListableBeanFactory类中通过一个HashMap来持有载入的BeanDefinition的。
```java DefaultListableBeanFactory.java
private final Map<String, BeanDefinitionbeanDefinitionMap = new ConcurrentHashMap<String, BeanDefinition>();
	private final List<StringbeanDefinitionNames = new ArrayList<String>();
//…
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
			throws BeanDefinitionStoreException {

		Assert.hasText(beanName, "Bean name must not be empty");
		Assert.notNull(beanDefinition, "BeanDefinition must not be null");

		if (beanDefinition instanceof AbstractBeanDefinition) {
			try {
				((AbstractBeanDefinition) beanDefinition).validate();
			}
			catch (BeanDefinitionValidationException ex) {
				throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
						"Validation of bean definition failed", ex);
			}
		}
		//注册的过程需要synchronized，保证数据的一致性
		synchronized (this.beanDefinitionMap) {
		    //这里检查是不是有相同名字的BeanDefinition已经在IoC容器中注册了，如果有相同名字的
		    //BeanDefinition，但又不允许覆盖，那么会抛出异常
			Object oldBeanDefinition = this.beanDefinitionMap.get(beanName);
			if (oldBeanDefinition != null) {
				if (!this.allowBeanDefinitionOverriding) {
					throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
							"Cannot register bean definition [" + beanDefinition + "] for bean '" + beanName +
							"': There is already [" + oldBeanDefinition + "] bound.");
				}
				else {
//如果遇到同名的BeanDefinition，但配置了allowBeanDefinitionOverriding，则覆盖
					if (this.logger.isInfoEnabled()) {
						this.logger.info("Overriding bean definition for bean '" + beanName +
								"': replacing [" + oldBeanDefinition + "] with [" + beanDefinition + "]");
					}
				}
			}
			//这是正常注册BeanDefinition的过程，把Bean的名字存入到beanDefinitionNames的同时，把
			//beanName作为Map的key,把beanDefinition作为value存入到IoC容器持有的beanDefinitionMap中去
			else {
				this.beanDefinitionNames.add(beanName);
				this.frozenBeanDefinitionNames = null;
			}
			this.beanDefinitionMap.put(beanName, beanDefinition);

			resetBeanDefinition(beanName);
		}
	}
```
完成了BeanDefinition的注册，就完成了IoC容器的整个初始化过程。依赖注入并不输入IoC容器的初始化过程。
