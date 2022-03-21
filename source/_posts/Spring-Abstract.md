---
title: Spring 抽象
date: 2021-08-29 14:37:01
tags: Spring
---

# 1. BeanFactory

用于访问Spring bean容器的 root 接口。

这是bean容器的基本客户端视图;其他接口如ListableBeanFactory和org.springframework.beans.factory.config.ConfigurableBeanFactory可用于特定用途。

该接口由包含许多bean定义的对象实现，每个bean定义都由一个String名称唯一标识。根据bean定义，工厂将返回被包含对象的独立实例(Prototype设计模式)，或单个共享实例(Singleton设计模式的更好选择，在该模式中，实例是工厂范围内的一个单例)。返回哪种类型的实例取决于bean工厂配置:API是相同的。从Spring 2.0开始，根据具体的应用程序上下文(例如。web环境中的“请求”和“会话”范围)。

这种方法的要点在于，BeanFactory是应用程序组件的中央注册中心，并集中应用程序组件的配置(例如，单个对象不再需要读取属性文件)。

请注意，依赖依赖项注入(“push”配置)通过setters或constructors配置应用程序对象通常比使用任何形式的“pull”配置(如BeanFactory查找)更好。Spring的依赖注入功能是使用这个BeanFactory接口及其子接口实现的。

**Bean工厂实现应该尽可能支持标准的Bean生命周期接口。完整的初始化方法及其标准顺序为:**

1. BeanNameAware的setBeanName
2. BeanClassLoaderAware的setBeanClassLoader
3. BeanFactoryAware的setBeanFactory
4. EnvironmentAware的setEnvironment
5. EmbeddedValueResolverAware的setEmbeddedValueResolver
6. ResourceLoaderAware的setResourceLoader(仅适用于在应用程序上下文中运行时)
7. ApplicationEventPublisherAware的setApplicationEventPublisher(仅适用于在应用程序上下文中运行时)
8. MessageSourceAware的setMessageSource(仅适用于在应用程序上下文中运行时)
9. ApplicationContextAware的setApplicationContext(仅适用于在应用程序上下文中运行时)
10. ServletContextAware的setServletContext(仅适用于运行在web应用程序上下文时)
11. **BeanPostProcessors**的**postProcessBeforeInitialization**方法
12. **InitializingBean**的**afterPropertiesSet**
13. 自定义的 **init-method** 定义
14. **BeanPostProcessors**的**postProcessAfterInitialization**方法

**在关闭bean工厂时，应用以下生命周期方法:**

1. DestructionAwareBeanPostProcessors的postProcessBeforeDestruction方法
2. DisposableBean 的 destroy
3. 自定义的 destroy-method 定义

![BeanFactoryUML_01](https://gitee.com/qmlg/image-bed/raw/master/images/1731892-20210610131522589-2144305263.png)



## AbstractAutowireCapableBeanFactory

实现默认 bean 创建的抽象 bean 工厂超类，具有RootBeanDefinition类指定的全部功能。 除了 AbstractBeanFactory 的createBean方法之外，还实现AutowireCapableBeanFactory接口。
提供 bean 创建（具有构造函数解析）、属性填充、wiring（包括autowiring）和初始化。 处理运行时 bean 引用、解析托管集合、调用初始化方法等。支持自动装配构造函数、按名称的属性和按类型的属性。

子类要实现的主要**模板方法**是**resolveDependency**(DependencyDescriptor, String, Set, TypeConverter) ，用于按类型自动装配。 如果工厂能够搜索其 bean 定义，匹配 bean 通常将通过这样的搜索来实现。 对于其他工厂样式，可以实现简化的匹配算法。

请注意，此类不承担或实现 bean 定义注册表功能。 有关org.springframework.beans.factory.ListableBeanFactory和BeanDefinitionRegistry接口的实现，请参阅DefaultListableBeanFactory ，它们分别代表此类工厂的 API 和 SPI 视图。

### createBean

- 用于对bean生命周期进行细粒度控制的专用方法。
- 使用非单例bean定义，以避免将bean注册为依赖bean。

- 此类的中心方法：创建 bean 实例、填充 bean 实例、应用后处理器等。

### doCreateBean

- 实际创建指定的bean。 预创建处理此时已经发生，例如检查postProcessBeforeInstantiation回调。
  区分默认 bean 实例化、工厂方法的使用和自动装配构造函数。

```java
  protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {

		// Instantiate the bean.
		BeanWrapper instanceWrapper = null;
		if (mbd.isSingleton()) {
      // 如果RootBeanDefinition是单例的，则移除未完成的FactoryBean实例的缓存。
			instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
		}
		if (instanceWrapper == null) {
      // 使用适当的实例化策略为指定的 bean 创建一个新实例：工厂方法、构造函数自动装配或简单实例化。
			instanceWrapper = createBeanInstance(beanName, mbd, args);
		}
		Object bean = instanceWrapper.getWrappedInstance();
		Class<?> beanType = instanceWrapper.getWrappedClass();
		if (beanType != NullBean.class) {
			mbd.resolvedTargetType = beanType;
		}

		// Allow post-processors to modify the merged bean definition.
		synchronized (mbd.postProcessingLock) {
			if (!mbd.postProcessed) {
				try {
          // 将 MergedBeanDefinitionPostProcessors 应用于指定的 bean 定义，调用它们的postProcessMergedBeanDefinition方法。
					applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
				}
				catch (Throwable ex) {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,
							"Post-processing of merged bean definition failed", ex);
				}
				mbd.postProcessed = true;
			}
		}

		// 主动缓存单例，以便能够解析循环引用，即使是在BeanFactoryAware等生命周期接口触发的情况下。
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}

		// Initialize the bean instance.
		Object exposedObject = bean;
		try {
			populateBean(beanName, mbd, instanceWrapper);
			exposedObject = initializeBean(beanName, exposedObject, mbd);
		}
		catch (Throwable ex) {
			if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
				throw (BeanCreationException) ex;
			}
			else {
				throw new BeanCreationException(
						mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
			}
		}

		if (earlySingletonExposure) {
			Object earlySingletonReference = getSingleton(beanName, false);
			if (earlySingletonReference != null) {
				if (exposedObject == bean) {
					exposedObject = earlySingletonReference;
				}
				else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
					String[] dependentBeans = getDependentBeans(beanName);
					Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
					for (String dependentBean : dependentBeans) {
						if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
							actualDependentBeans.add(dependentBean);
						}
					}
					if (!actualDependentBeans.isEmpty()) {
						throw new BeanCurrentlyInCreationException(beanName,
								"Bean with name '" + beanName + "' has been injected into other beans [" +
								StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
								"] in its raw version as part of a circular reference, but has eventually been " +
								"wrapped. This means that said other beans do not use the final version of the " +
								"bean. This is often the result of over-eager type matching - consider using " +
								"'getBeanNamesForType' with the 'allowEagerInit' flag turned off, for example.");
					}
				}
			}
		}

		// Register bean as disposable.
		try {
			registerDisposableBeanIfNecessary(beanName, bean, mbd);
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanCreationException(
					mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
		}

		return exposedObject;
	}
```



## DefaultListableBeanFactorys

- Spring 对ConfigurableListableBeanFactory和BeanDefinitionRegistry接口的默认实现：一个基于 bean 定义元数据的成熟 bean 工厂，可通过后处理器扩展。
- 典型用法是在访问 bean 之前首先注册所有 bean 定义（可能从 bean 定义文件中读取）。 因此，按名称查找 Bean 是在本地 bean 定义表中的廉价操作，对预先解析的 bean 定义元数据对象进行操作。
- 请注意，特定 bean 定义格式的读取器通常是单独实现的，而不是作为 bean 工厂子类实现：例如参见org.springframework.beans.factory.xml.XmlBeanDefinitionReader 。
- 对于org.springframework.beans.factory.ListableBeanFactory接口的另一种实现，请查看StaticListableBeanFactory ，它管理现有的 bean 实例，而不是根据 bean 定义创建新实例。

## DefaultSingletonBeanRegistry

- singletonObjects【一级缓存】
  - 单例对象的缓存: beanName -> bean 实例
- singletonFactories【三级缓存】
  - 单例工厂的缓存: beanName -> ObjectFactory 实例
- earlySingletonObjects【二级缓存】
  - 早期单例对象（半成品，属性未填充的）的缓存: beanName -> bean 实例
- registeredSingletons
  - 一组已注册的单例，包含按注册顺序排列的bean名称。
- singletonsCurrentlyInCreation
  - 当前正在创建的bean的名称。
- inCreationCheckExclusions
  - 当前在创建检查中排除的bean的名称。
- suppressedExceptions
  - 被抑制异常的集合，可用于关联相关原因。

- singletonsCurrentlyInDestruction: boolean = false;
  - 标志表明我们目前是否在摧毁singleton。
- disposableBeans
  - 可丢弃的bean实例:bean名称到可丢弃的实例。
- containedBeanMap
  - 包含bean名称之间的映射: bean名称与bean包含的bean名称集之间的映射。
- dependentBeanMap
  - 依赖bean名称之间的映射:bean名称到依赖该bean的bean名称集。即“谁依赖我”。
- dependenciesForBeanMap
  - 依赖bean名称之间的映射:bean名称到bean的依赖项的bean名称集。即“我依赖谁”





# 2. API和SPI视图

Java 中区分 API 和 SPI，通俗的讲：API 和 SPI 都是相对的概念，他们的差别只在语义上，API 直接被应用开发人员使用，SPI 被框架扩展人员使用。

API （Application Programming Interface）

- 大多数情况下，都是**实现方**来制定接口并完成对接口的不同实现，**调用方**仅仅依赖却无权选择不同实现。

SPI (Service Provider Interface)

- 而如果是**调用方**来制定接口，**实现方**来针对接口来实现不同的实现。**调用方**来选择自己需要的实现方。

从面向接口编程说起

![这里写图片描述](https://gitee.com/qmlg/image-bed/raw/master/images/20180909205040343.jpeg)

当我们选择在**调用方** 和 **实现方** 中间引入接口。上图没有给出“接口”应该位于哪个“包”中，从纯粹的可能性上考虑，我们有三种选择：

1. 接口位于**实现方**所在的包中；
2. 接口位于**调用方**所在的包中；
3. 接口位于独立的包中。

## 2.1 接口位于【调用方】所在的包中

对于类似这种情况下接口，我们将其称为 SPI, SPI的规则如下：

- 概念上更依赖调用方。
- 组织上位于调用方所在的包中。
- 实现位于独立的包中。

常见的例子是：插件模式的插件。如：

- 数据库驱动 Driver
- 日志 Log
- dubbo扩展点开发

## 2.2 接口位于【实现方】所在的包中

对于类似这种情况下的接口，我们将其称作为API，API的规则如下：

- 概念上更接近实现方。
- 组织上位于实现方所在的包中。

## 2.3 接口位于独立的包中

如果一个“接口”在一个上下文是API，在另一个上下文是SPI，那么你就可以这么组织

需要注意的事项

SPI 和 API 也不一定是接口，这里都是指狭义的具体的接口。

![这里写图片描述](https://gitee.com/qmlg/image-bed/raw/master/images/70-20210829161437067.png)

**Java类库中的实例**

```java
Class.forName("com.mysql.jdbc.Driver");
Connection conn = DriverManager.getConnection(
              "jdbc:mysql://localhost:3306/test", "root", "123456");
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("select * from Users");
```

说明：java.sql.Driver 是 Spi，com.mysql.jdbc.Driver 是 Spi 实现，其它的都是 Api。

# 3. RootBeanDefinition



# 4. ObjectFactory

- @FunctionalInterface
- 定义一个工厂，它可以在调用时返回一个 Object 实例（可能是共享的或独立的）。
  此接口通常用于封装通用工厂，该工厂在每次调用时返回某个目标对象的新实例（原型）。
  此接口类似于FactoryBean ，但后者的实现通常意味着在BeanFactory定义为 SPI 实例，而此类的实现通常意味着作为 API 提供给其他 bean（通过注入）。 因此， getObject()方法具有不同的异常处理行为。

- ```java
  () -> getEarlyBeanReference(beanName, mbd, bean)
  ```

  

  ```java
    protected Object getEarlyBeanReference(String beanName, RootBeanDefinition mbd, Object bean) {
  		Object exposedObject = bean;
  		if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
  			for (SmartInstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().smartInstantiationAware) {
  				exposedObject = bp.getEarlyBeanReference(exposedObject, beanName);
  			}
  		}
  		return exposedObject;
  	}
  ```

  AbstractAutoProxyCreator是SmartInstantiationAwareBeanPostProcessor的一个抽象实现，其getEarlyBeanReference的实现如下：

  ```java
    @Override
  	public Object getEarlyBeanReference(Object bean, String beanName) {
  		Object cacheKey = getCacheKey(bean.getClass(), beanName);
  		this.earlyProxyReferences.put(cacheKey, bean);
      // 必要时包装给定的bean，例如，如果它符合被代理的条件。
  		return wrapIfNecessary(bean, beanName, cacheKey);
  	}
  ```

  它也实现了BeanPostProcessor接口。

# 5. TypeConverter

SimpleTypeConverter











# TypeConverterDelegate

- 用于将属性值转换为目标类型的内部助手类。









------

> 内容来源：
>
> - [Spring源码系列(二)--bean组件的源码分析](https://www.cnblogs.com/ZhangZiSheng001/p/13196228.html)
> - [SPI 与 API的区别](https://blog.csdn.net/jyxmust/article/details/82562242)

