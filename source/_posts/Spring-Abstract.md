---
title: Spring 抽象
date: 2021-08-29 14:37:01
tags: Spring
categories: recycle-bin
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





## 1. AbstractAutowireCapableBeanFactory

- initializeBean -> 
  - applyBeanPostProcessorsBeforeInitialization
  - applyBeanPostProcessorsAfterInitialization

## 2. DefaultSingletonBeanRegistry

getSingleton

> 处理循环引用。

## PropertyAccessorFactory（spring-beans）

- 获取PropertyAccessor实例的简单工厂facade，特别是对于BeanWrapper实例。隐藏实际的目标实现类及其扩展的公共签名。

## HandlerInterceptor（spring-webmvc）

> HandlerInterceptor基本上类似于Servlet Filter，但与后者相比，它只允许自定义预处理（可以选择禁止处理程序本身的执行），以及自定义后处理。过滤器功能更强大，例如，它们允许交换传递给链的请求和响应对象。请注意，过滤器是在web .xml中配置的，它是应用程序上下文中的HandlerInterceptor。

## WebRequestInterceptor

> 接口，一般的web请求拦截。允许通过构建WebRequest抽象来应用于Servlet请求。
> 该接口采用**mvc风格的请求处理:执行一个处理程序，公开一组模型对象，然后根据该模型呈现视图。**另外，处理程序也可以完全处理请求，而不呈现视图。

## RequestMappingHandlerMapping

> 从@Controller类的类型和方法级的@RequestMapping注释中创建RequestMappingInfo实例。
> 弃用注意:
> 在5.2.4中，useSuffixPatternMatch和useRegisteredSuffixPatternMatch被弃用，以阻止使用路径扩展来进行请求映射和内容协商(与ContentNegotiationManagerFactoryBean中类似的弃用)。有关更多内容，请阅读第24719期。

## BeanNameUrlHandlerMapping

> 实现了org.springframe.web.servlet.HandlerMapping接口，该接口将url映射到名称以斜杠(“/”)开头的bean，类似于Struts将url映射到动作名称的方式。
> 这是org.springframework.web.servlet使用的默认实现。DispatcherServlet，以及org.springframework.web.servlet.mvc.method.annotation .RequestMappingHandlerMapping。另外，SimpleUrlHandlerMapping允许以声明的方式自定义处理程序映射。
> 映射是从URL到bean名。因此，一个传入的URL“/foo”将映射到一个名为“/foo”的处理程序，或者映射到“/foo /foo2”，如果多个映射到一个单独的处理程序。
> 支持直接匹配(给定"/test" ->注册"/test")和"*"匹配(给定"/test" ->注册"/t*")。注意，默认情况下，如果适用，映射到当前servlet映射中;详见"alwaysUseFullPath"属性。关于模式选项的详细信息，请参见org.springframework.util.AntPathMatcher javadoc。

## ProxyFactory

> 用于编程使用的AOP代理的工厂，而不是通过bean工厂中的声明性设置。这个类提供了一种在自定义用户代码中获取和配置AOP代理实例的简单方法。

## AopProxyFactory -> DefaultAopProxyFactory

> **DefaultAopProxyFactory**
>
> 默认的AopProxyFactory实现，创建CGLIB代理或JDK动态代理。
>
> 创建一个CGLIB代理，如果一个给定的AdvisedSupport实例如下所示:
>
> - 设置了 optimize 标志
> - 设置了 proxyTargetClass 标志
> - 没有指定代理接口
>
> 通常，指定proxyTargetClass来强制使用CGLIB代理，或者指定一个或多个接口来使用JDK动态代理

- createAopProxy





## BeanDefinitionRegistry

- 保存bean定义的注册中心的接口，例如RootBeanDefinition和ChildBeanDefinition实例。 通常由内部使用AbstractBeanDefinition层次结构的Beanfactories实现。
- 这是Spring bean工厂包中封装bean定义注册的唯一接口。标准的BeanFactory接口只包括对完全配置的工厂实例的访问。
- Spring bean定义的读者希望使用这个接口的实现。Spring core 中已知的实现者有DefaultListableBeanFactory和GenericApplicationContext。









LazyInitializationBeanFactoryPostProcessor + LazyInitializationExcludeFilter

ImportBeanDefinitionRegistrar

ForwardedHeaderFilter & FilterRegistrationBean





------

> 内容来源：
>
> - [Spring源码系列(二)--bean组件的源码分析](https://www.cnblogs.com/ZhangZiSheng001/p/13196228.html)
> - [SPI 与 API的区别](https://blog.csdn.net/jyxmust/article/details/82562242)

