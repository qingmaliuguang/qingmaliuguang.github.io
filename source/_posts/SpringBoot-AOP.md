---
title: SpringBoot-AOP
date: 2022-03-21 15:06:25
tags: SpringBoot
categories: 整理-来自代码
---

# 1. 关于Spring AOP

## 1.1 @EnableAspectJAutoProxy

如下来自其注释。

> 支持处理用AspectJ的@Aspect注解标记的组件，类似于在Spring的  aop:aspectj-autoproxy XML元素中找到的功能。
> 在@Configuration类上使用如下:
>
> ```java
> @Configuration
>    @EnableAspectJAutoProxy
>    public class AppConfig {
>   
>        @Bean
>        public FooService fooService() {
>            return new FooService();
>        }
>   
>        @Bean
>        public MyAspect myAspect() {
>            return new MyAspect();
>        }
>    }
> ```
>
> FooService是一个典型的POJO组件，而MyAspect是一个@Aspect风格的方面:
>
> ```java
> public class FooService {
>   
>        // various methods
>    }
>    @Aspect
>    public class MyAspect {
>   
>        @Before("execution(* FooService+.*(..))")
>        public void advice() {
>            // advise FooService methods as appropriate
>        }
>    }
> ```
>
> 在上面的场景中，@EnableAspectJAutoProxy确保了MyAspect将被正确处理，并且FooService将在它提供的通知中被代理混合。
> 用户可以使用proxyTargetClass()属性来控制为FooService创建的代理类型。下面启用cglib风格的“子类”代理，而不是默认的基于接口的JDK代理方法。
>
> ```java
> @Configuration
>    @EnableAspectJAutoProxy(proxyTargetClass=true)
>    public class AppConfig {
>        // ...
>    }
> ```
>
> 请注意@Aspect bean可以像其他任何bean一样被组件扫描。简单地用@Aspect和@Component标记切面:
>
> ```java
> package com.foo;
>   
>    @Component
>    public class FooService { ... }
>   
>    @Aspect
>    @Component
>    public class MyAspect { ... }
> ```
>
> 注意；@EnableAspectJAutoProxy只适用于它的本地应用程序上下文，允许在不同级别上选择性地代理bean。 请在每个单独的上下文中重新声明@EnableAspectJAutoProxy，例如公共的根web应用程序上下文和任何单独的DispatcherServlet应用程序上下文，如果你需要在多个级别应用它的行为。
> 这个特性需要aspectjweaver出现在类路径上。虽然这个依赖通常对于spring-aop是可选的，但是对于@EnableAspectJAutoProxy及其底层工具来说是必需的。

**@EnableAspectJAutoProxy通过添加@Import注解引入了AspectJAutoProxyRegistrar.class。**

## 1.2 AspectJAutoProxyRegistrar

- 根据给定的@EnableAspectJAutoProxy注解，在当前的BeanDefinitionRegistry中注册一个**AnnotationAwareAspectJAutoProxyCreator**。
- 基于导入的@Configuration类上的@EnableAspectJAutoProxy.proxyTargetClass()属性的值，注册、升级和配置AspectJ自动代理创建器
- AspectJAutoProxyRegistrar实现自ImportBeanDefinitionRegistrar接口。

> **ImportBeanDefinitionRegistrar**
>
> 接口，由在处理@Configuration类时注册额外bean定义的类型实现。当在bean定义级别(与@Bean方法/实例级别相反)操作时，需要或必须使用。
>
> 与@Configuration和ImportSelector一起，这种类型的类可以提供给@Import注解(也可以从ImportSelector返回)。
>
> <font color='FF0000'>是在什么时候被调用的？</font>
>
> ImportBeanDefinitionRegistrar可以实现以下任何一个Aware接口，它们各自的方法将在registerBeanDefinitions之前被调用:
>
> - EnvironmentAware
> - BeanFactoryAware
> - BeanClassLoaderAware
> - ResourceLoaderAware
>
> 或者，该类可以提供唯一的构造函数，其中包含一个或多个受支持的形参类型:
>
> - Environment
> - BeanFactory
> - ClassLoader
> - ResourceLoader
>
> 有关使用示例，请参阅实现和相关的单元测试

## 1.3 AnnotationAwareAspectJAutoProxyCreator

> 处理当前应用上下文中所有AspectJ注解切面的**AspectJAwareAdvisorAutoProxyCreator**子类，以及Spring Advisor。
>
> 任何带有AspectJ注解的类都将被自动识别，如果Spring AOP的基于代理的模型能够应用它们，那么它们的通知将被应用。这包括了方法执行连接点。
>
> 如果使用了aop:include元素，只有名称与包含模式匹配的@AspectJ bean才会被认为定义了用于Spring自动代理的方面。
>
> Spring Advisors的处理遵循在org.springframework.aop.framework.autoproxy.**AbstractAdvisorAutoProxyCreator**中建立的规则。
>
> - 继承结构
>
>   > ![AnnotationAwareAspectJAutoProxyCreator](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/AnnotationAwareAspectJAutoProxyCreator-9320885.jpg)

- 它有两个AspectJAdvisorFactory和BeanFactoryAspectJAdvisorsBuilder类型的成员：aspectJAdvisorFactory和aspectJAdvisorsBuilder。aspectJAdvisorFactory为ReflectiveAspectJAdvisorFactory的实例，并用aspectJAdvisorFactory构造了aspectJAdvisorsBuilder。

- findCandidateAdvisors方法

  > -> 调用了aspectJAdvisorsBuilder.buildAspectJAdvisors()
  >
  > -> 调用了aspectJAdvisorFactory.getAdvisors(...)   【ReflectiveAspectJAdvisorFactory::getAdvisors(...)】

## 1.4 AbstractAdvisorAutoProxyCreator

> 通用的自动代理生成器，根据检测到的每个bean的advisor为特定bean构建AOP代理。
>
> **子类可以重写findCandidateAdvisors()方法来返回应用于任何对象的自定义advisor列表。 子类还可以重写继承的shouldSkip方法，以排除某些对象的自动代理。**
>
> 需要排序的顾问或建议应该用@Order注解，或者实现org.springframework.core.Ordered接口。 这个类使用AnnotationAwareOrderComparator对顾问进行排序。 没有使用@Order注释或没有实现Ordered接口的advisor将被认为是无序的;它们将以未定义的顺序出现在advisor链的末尾。

- 继承自**AbstractAutoProxyCreator**。

## 1.5 AbstractAutoProxyCreator

> org.springframework.beans.factory.config.**BeanPostProcessor**的实现， 它用AOP代理包装每个合格的bean，在调用bean本身之前委托给指定的拦截器。
>
> - BeanPostProcessor
>
>   > - 工厂钩子，允许自定义修改新bean实例——例如，检查标记接口或用代理包装bean。
>   >
>   > - 通常，通过标记接口或类似的方式填充bean的后处理器将实现postProcessBeforeInitialization， 而用代理包装bean的后处理器通常将实现postProcessAfterInitialization。
>   >
>   > - 注册
>   >
>   >   ApplicationContext可以在它的bean定义中自动检测BeanPostProcessor bean，并将这些后处理器应用到随后创建的任何bean。 普通的BeanFactory允许以编程方式注册后处理程序，将它们应用于通过bean工厂创建的所有bean。
>   >
>   > - 排序
>   >
>   >   在ApplicationContext中被自动检测到的BeanPostProcessor bean将根据 org.springframework.core.prioritordered和org.springframework.core.Ordered语义进行排序。 相反，以编程方式注册到BeanFactory的BeanPostProcessor bean将按照注册的顺序应用; 对于以编程方式注册的后处理器，任何通过实现prioritorderordered或Ordered接口表示的排序语义都将被忽略。 而且，对于BeanPostProcessor bean， @Order注解没有被考虑在内
>
> - **postProcessBeforeInstantiation** 【实例化之前】
>
>   > - 流程图
>   >
>   >   <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/AnnotationAwareAspectJAutoProxyCreator-postProcessBeforeInstantiation.jpg" alt="AnnotationAwareAspectJAutoProxyCreator-postProcessBeforeInstantiation" style="zoom: 33%;" />
>
>   -> createProxy
>
>   > 为给定的bean创建AOP代理。
>   >
>   > - 流程图
>   >
>   >   ![AnnotationAwareAspectJAutoProxyCreator-createProxy](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/AnnotationAwareAspectJAutoProxyCreator-createProxy.jpg)
>   >
>   > 
>
>   -> ProxyFactory::getProxy
>
>   > 根据该工厂中的设置创建一个新的代理。
>
>   -> ProxyCreatorSupport::createAopProxy
>
>   > - ProxyCreatorSupport
>   >
>   >   > 代理工厂的基类。提供对可配置的AopProxyFactory的方便访问
>   >   >
>   >   > - AopProxyFactory
>   >   >
>   >   >   > 接口，由工厂实现，这些工厂能够基于AdvisedSupport配置对象创建AOP代理。
>   >   >   >
>   >   >   > 代理人须遵守以下合约:
>   >   >   >
>   >   >   > - 它们应该实现配置指出应该代理的所有接口。
>   >   >   > - 它们应该实现被建议的接口。
>   >   >   > - 它们应该实现equals方法来比较代理接口、通知和目标。
>   >   >   > - 如果所有的advisor和target都是可序列化的，它们应该是可序列化的。
>   >   >   > - 如果advisor和target是线程安全的，那么它们应该是线程安全的。
>   >   >   >
>   >   >   > 代理可能允许也可能不允许更改通知。如果它们不允许更改通知(例如，因为配置被冻结)，代理应该在尝试更改通知时抛出AopConfigException。
>
>   -> DefaultAopProxyFactory::createAopProxy
>
>   > - DefaultAopProxyFactory
>   >
>   >   > 默认的AopProxyFactory实现，创建CGLIB代理或JDK动态代理。
>   >   >
>   >   > <font color='00FF00'>创建一个CGLIB代理，如果一个给定的AdvisedSupport实例如下所示:</font>
>   >   >
>   >   > - 设置了 optimize 标志
>   >   > - 设置了 proxyTargetClass 标志
>   >   > - 没有指定代理接口
>   >   >
>   >   > 通常，指定proxyTargetClass来强制使用CGLIB代理，或者指定一个或多个接口来使用JDK动态代理。
>
>   -> AopProxy::getProxy
>
>   > - AopProxy
>   >
>   >   > 用于已配置的AOP代理的委托接口，允许创建实际的代理对象。
>   >   >
>   >   > JDK动态代理和CGLIB代理都有现成的实现，由DefaultAopProxyFactory应用。
>   >   >
>   >   > - 类图：
>   >   >
>   >   > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/AopProxy.jpg" alt="AopProxy" style="zoom: 50%;" />
>   >
>   > - JdkDynamicAopProxy::getProxy
>   >
>   >   > <font color='FF0000'>JdkDynamicAopProxy的invoke方法是在哪里调用的？</font>
>   >
>   >   -> Proxy.newProxyInstance  【jdk java.lang.reflect】
>   >
>   >   > 返回指定接口的代理实例，该接口将方法调用分派给指定的调用处理程序。
>   >   >
>   >   > 如果违反了以下任何一个限制，将会抛出IllegalArgumentException:
>   >   >
>   >   > ......
>   >   >
>   >   > 请注意，指定的代理接口的顺序是重要的:对具有相同接口组合但顺序不同的代理类的两个请求将导致两个不同的代理类。
>   >
>   >   -> Constructor::newInstance
>   >
>   > - ObjenesisCglibAopProxy::getProxy
>   >
>   >   > 基于对象的CglibAopProxy扩展，在不调用类构造函数的情况下创建代理实例。在Spring 4中默认使用。
>   >   >
>   >   > - CglibAopProxy
>   >   >
>   >   >   > 面向Spring AOP框架的基于cglib的AopProxy实现。
>   >   >   >
>   >   >   > 这种类型的对象应该通过代理工厂获得，由AdvisedSupport对象配置。这个类是Spring AOP框架的内部类，客户机代码不需要直接使用它。
>   >   >   >
>   >   >   > 如果需要，DefaultAopProxyFactory将自动创建基于cglib的代理，例如在代理一个目标类的情况下(有关详细信息，请参阅代理javadoc)。
>   >   >   >
>   >   >   > 如果底层(目标)类是线程安全的，则使用该类创建的代理是线程安全的。
>   >
>   >   -> SpringObjenesis::newInstance  ||  Constructor::newInstance
>   >
>   >   > - SpringObjenesis
>   >   >
>   >   >   > 特定于Spring的ObjenesisStd / ObjenesisBase的变体，提供基于Class键而不是类名的缓存，并允许选择性地使用缓存。
>
> - **postProcessAfterInitialization** 【初始化之后】
>
>   > 如果bean被子类标识为代理，则使用配置的拦截器创建代理
>   >
>   > - 流程图
>   >
>   >   ![AnnotationAwareAspectJAutoProxyCreator-postProcessAfterInitialization](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/AnnotationAwareAspectJAutoProxyCreator-postProcessAfterInitialization.jpg)
>   
>   -> wrapIfNecessary
>   
>   -> createProxy
>
> 这个类区分“公共”拦截器:为它创建的所有代理共享，和“特定”拦截器:每个bean实例唯一。 不需要任何通用的拦截器。如果有，则使用interceptorNames属性设置它们。

# 2. AnnotationAwareAspectJAutoProxyCreator是什么时候怎么起作用的？

对照继承结构图《AnnotationAwareAspectJAutoProxyCreator》和流程图《AbstractAutowireCapableBeanFactory\-createBean》。

AnnotationAwareAspectJAutoProxyCreator继承了超类AbstractAutoProxyCreator对如下三个关键接口的实现：

- postProcessBeforeInstantiation

  > - 在AbstractAutowireCapableBeanFactory\#resolveBeforeInstantiation处触发调用。
  >
  >   applyBeanPostProcessorsBeforeInstantiation

- postProcessProperties

  > - 在AbstractAutowireCapableBeanFactory\#populateBean处触发调用。

- postProcessAfterInitialization

  > - 在AbstractAutowireCapableBeanFactory\#resolveBeforeInstantiation处触发调用。
  >
  >   applyBeanPostProcessorsAfterInitialization
  >
  > - 在AbstractAutowireCapableBeanFactory\#initializeBean处触发调用。
  >
  >   applyBeanPostProcessorsAfterInitialization

- 另外两个后处理方法postProcessAfterInstantiation和postProcessBeforeInitialization均继承的接口中的默认实现，无特殊处理。
