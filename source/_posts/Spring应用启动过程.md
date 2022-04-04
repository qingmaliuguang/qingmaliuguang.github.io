---
title: Spring应用启动过程
date: 2022-03-21 15:06:25
tags: SpringBoot
categories: in-use
---

# 1. SpringApplication

> 类，可用于从Java主方法引导和启动Spring应用程序。默认情况下，类将执行以下步骤来引导你的应用程序:
>
> - 创建适当的ApplicationContext实例(取决于您的类路径)
> - 注册CommandLinePropertySource以将命令行参数公开为Spring属性
> - 刷新应用程序上下文，加载所有单例beans
> - 触发任何CommandLineRunner beans

# BootstrapContext

引导上下文

# ApplicationContext

应用上下文

> 为应用程序提供配置的中央接口。在应用程序运行时，这是只读的，但如果实现支持，可能会重新加载。
> 一个ApplicationContext提供:
>
> - 用于访问应用程序组件的Bean工厂方法。从ListableBeanFactory继承。
> - 以通用方式加载文件资源的能力。继承自org.springframework.core.io.ResourceLoader接口。
> - 将事件发布到已注册侦听器的能力。继承自ApplicationEventPublisher接口。
> - 解析消息的能力，支持国际化。继承自MessageSource接口。
> - 从父上下文继承。后代上下文中的定义总是优先。这意味着，例如，一个单独的父上下文可以被整个web应用程序使用，而每个servlet都有自己独立于任何其他servlet的子上下文。
>
> 除了标准的org.springframework.beans.factory.BeanFactory生命周期功能外，ApplicationContext实现检测和调用ApplicationContextAware bean以及ResourceLoaderAware、ApplicationEventPublisherAware和MessageSourceAware bean。

# ApplicationContextInitializer

> - 在刷新之前初始化一个Spring ConfigurableApplicationContext的回调接口。
> - 通常用于需要对应用上下文进行编程初始化的web应用程序中。例如，根据上下文的环境注册属性源或激活概要文件。参见ContextLoader和FrameworkServlet 支持，分别声明一个“contextInitializerClasses”上下文参数和初始化参数。
> - ApplicationContextInitializer处理器被鼓励检测Spring的Ordered接口是否已经实现，或者是否存在@Order注解，如果存在，则在调用之前对实例进行排序。
> - 在prepareContext()过程中应用。

# SpringApplicationRunListener

> - 用于SpringApplication运行方法的侦听器。SpringApplicationRunListeners是通过SpringFactoriesLoader加载的，它应该声明一个公共构造函数，该构造函数接受一个SpringApplication实例和一个String[]参数。每次运行都会创建一个新的SpringApplicationRunListener实例
>
> - 唯一实现：**EventPublishingRunListener**
>
>   > - 发布SpringApplicationEvents的SpringApplicationRunListener。
>   >
>   > - 使用内部的**ApplicationEventMulticaster**来处理在上下文实际刷新之前触发的事件。
>   >
>   >   > **ApplicationEventMulticaster**
>   >   >
>   >   > - 由对象实现的接口，这些对象可以管理许多**ApplicationListener**对象并向它们发布事件。
>   >   > - 一个org.springframework.context.ApplicationEventPublisher，通常是一个Spring org.springframework.context.ApplicationEventPublisher，可以使用一个ApplicationEventMulticaster作为一个委托来实际发布事件。
>
> - 通过该接口可看出Spring Boot将运行过程分成几个环节（事件），对应spring-boot包context.event package 下的七个SpringApplicationEvent。
>
>   > 1. starting：当run方法第一次启动时立即调用。可以用于非常早期的初始化。
>   >
>   >    -> ApplicationStartingEvent
>   >
>   >    > 事件应该在SpringApplication启动后尽早发布——在环境或ApplicationContext可用之前，但在ApplicationListeners被注册之后。事件的源是SpringApplication本身，但是要注意在这个早期阶段不要过多地使用它的内部状态，因为它可能会在生命周期的后期被修改。
>   >
>   > 2. environmentPrepared：在环境准备好之后，但在ApplicationContext创建之前调用。 
>   >
>   >    -> ApplicationEnvironmentPreparedEvent
>   >
>   >    > 事件发布时，一个SpringApplication正在启动，并且Environment第一次可用来检查和修改。
>   >
>   > 3. contextPrepared：创建并准备好ApplicationContext，但在加载源之前调用。
>   >
>   >    -> ApplicationContextInitializedEvent
>   >
>   >    > 事件发布时，SpringApplication启动，ApplicationContext已经准备好，ApplicationContextInitializers已经被调用，但在任何bean定义被加载之前。
>   >
>   > 4. contextLoaded：加载应用程序上下文后，在刷新应用程序上下文之前调用。
>   >
>   >    -> ApplicationPreparedEvent
>   >
>   >    > 事件发布时，一个SpringApplication正在启动，ApplicationContext已经完全准备好，但还没有刷新。bean定义将被加载，并且在这个阶段Environment已经可以使用了。
>   >
>   > 5. started：上下文已经刷新，应用程序已经启动，但是CommandLineRunners和ApplicationRunners还没有被调用。
>   >
>   >    -> ApplicationStartedEvent
>   >
>   >    > 事件在刷新应用程序上下文之后，但在调用任何应用程序和命令行运行程序之前发布。
>   >
>   > 6. ready：在run方法结束之前，当应用程序上下文已经被刷新并且所有的CommandLineRunners和ApplicationRunner已经被调用时，立即调用。
>   >
>   >    -> ApplicationReadyEvent
>   >
>   >    > 事件尽可能晚地发布，以表明应用程序已准备好为请求提供服务。事件的源是SpringApplication本身，但是要谨慎修改它的内部状态，因为那时所有的初始化步骤都已经完成了
>   >
>   > 7. failed：当运行应用程序时发生故障时调用。
>   >
>   >    -> ApplicationFailedEvent
>   >
>   >    > SpringApplication启动失败时发布的事件。
>   >
>   > 

# ApplicationListener

> - 由应用程序事件侦听器实现的接口。
> - 基于观察者设计模式的标准java.util.EventListener接口。
> - 从Spring 3.0开始，ApplicationListener可以通用地声明它感兴趣的事件类型。当在Spring ApplicationContext中注册时，事件将被相应地过滤，只有匹配的事件对象才会调用侦听器。
> - 不同于SpringApplicationRunListener，有一点关联，其关联可阅读EventPublishingRunListener。

# ApplicationContextFactory

> - 用于创建用于SpringApplication的ConfigurableApplicationContext的策略接口。创建的上下文应该以默认的形式返回，由SpringApplication负责配置和刷新上下文。 
>
> - 其有一个功能接口create(WebApplicationType webApplicationType)，根据给定的webApplicationType为SpringApplication创建应用上下文。
>
> - 默认实现：ApplicationContextFactory.DEFAULT
>
>   > - SERVLET -> **AnnotationConfigServletWebServerApplicationContext**
>   > - REACTIVE -> **AnnotationConfigReactiveWebServerApplicationContext**
>   > - default -> AnnotationConfigApplicationContext

# AnnotationConfigServletWebServerApplicationContext

> - ServletWebServerApplicationContext，它接受带注解的类作为输入——特别是带@Configuration注解的类，但也接受普通的@Component类和使用javax兼容JSR-330 的javax.inject注解。允许逐个注册类(将类名指定为配置位置)以及类路径扫描(将基包指定为配置位置)。
>
> - 注意:在有多个@Configuration类的情况下，后面的@Bean定义将覆盖前面加载文件中定义的定义。可以利用这一点，通过额外的Configuration类有意覆盖某些bean定义。
>
> - register(Class<?>... annotatedClasses)
>
>   > 注册一个或多个要处理的带注解类。请注意，为了让上下文完全处理新类，必须调用**refresh()**。
>   > 调用#register是**幂等**的;多次添加同一个带注解的类不会产生额外的效果。
>
> - scan(String... basePackages)
>
>   > 在指定的基包中执行扫描。请注意，为了让上下文完全处理新类，必须调用**refresh()**。 
>
> - **refresh()**
>
>   > 继承自**ServletWebServerApplicationContext**的实现：在超类（**AbstractApplicationContext**）的refresh外层加了异常处理，发生运行时异常时关闭web服务器。
>
> - 其supertypes hierarchy：
>
>   > ![AnnotationConfigServletWebServerApplicationContext](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/AnnotationConfigServletWebServerApplicationContext.jpg)
>   >
>   > - BeanFactory：用于访问Spring bean容器的根接口。
>   >
>   > - **AbstractApplicationContext**
>   >
>   >   > - ApplicationContext接口的抽象实现。不强制要求用于配置的存储类型;简单地实现公共上下文功能。使用模板方法设计模式，需要具体的子类来实现抽象方法。
>   >   >
>   >   > - 与普通的BeanFactory相反，ApplicationContext应该检测在其内部bean工厂中定义的特殊bean:因此，这个类自动注册了在上下文中定义为bean的BeanFactoryPostProcessors、BeanPostProcessors和ApplicationListeners。
>   >   >
>   >   > - MessageSource也可以在上下文中作为bean提供，命名为“messageSource”;否则，消息解析被委托给父上下文。此外，应用事件的多播器可以作为ApplicationEventMulticaster类型的bean在上下文中提供;否则，将使用SimpleApplicationEventMulticaster类型的默认multicaster。
>   >   >
>   >   > - 通过扩展DefaultResourceLoader实现资源加载。因此，将非url资源路径视为类路径资源(支持包含包路径的完整类路径资源名，例如。"mypackage/myresource.dat")，除非getResourceByPath方法在子类中被重写。
>   >   >
>   >   > - **refresh()**
>   >   >
>   >   >   > ![AbstractApplicationContext-refresh](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/AbstractApplicationContext-refresh.jpg)
>   >
>   > - GenericApplicationContext
>   >
>   >   > - 通用的ApplicationContext实现，保存单个内部DefaultListableBeanFactory实例，并且不采用特定的bean定义格式。实现BeanDefinitionRegistry 接口，以便对其应用任何bean定义读取器。
>   >   > - 典型的用法是通过BeanDefinitionRegistry接口注册各种bean定义，然后调用refresh()来用应用上下文语义初始化这些bean(处理ApplicationContextAware，自动检测BeanFactoryPostProcessors，等等)。
>   >   > - 与为每次刷新创建一个新的内部BeanFactory实例的其他ApplicationContext实现不同，此上下文的内部BeanFactory从一开始就可用，以便能够在其上注册bean定义。refresh() 只能被调用一次。
>   >   > - 对于自定义应用程序上下文实现，这些实现应该以可刷新的方式读取特殊的bean定义格式，考虑从AbstractRefreshableApplicationContext基类派生。
>   >   > - 该类无参构造中创建的**DefaultListableBeanFactory**的实例。
>   >
>   > - GenericWebApplicationContext
>   >
>   >   > - GenericApplicationContext的子类，适用于web环境。
>   >   > - 实现了ConfigurableWebApplicationContext，但不是用于web.xml中的声明式设置。相反，它是为编程设置而设计的，例如用于构建嵌套上下文或在WebApplicationInitializer中使用。
>   >
>   > - ServletWebServerApplicationContext
>   >
>   >   > - 一个WebApplicationContext，可以用来从包含的ServletWebServerFactory bean中引导自己。
>   >   >
>   >   >   > ServletWebServerFactory: 工厂接口，可以用来创建一个WebServer。
>   >   >
>   >   > - 这个上下文将通过在ApplicationContext本身中搜索单个ServletWebServerFactory bean来创建、初始化和运行一个WebServer。ServletWebServerFactory可以自由使用标准的Spring概念(比如依赖注入、生命周期回调和属性占位符变量)。
>   >   >
>   >   > - 此外，任何在上下文中定义的Servlet或Filter bean都将自动注册到web服务器。在单个Servlet bean的情况下，将使用'/'映射。如果发现多个Servlet bean，则小写的bean名称将用作映射前缀。任何名为'dispatcherServlet'的Servlet都会被映射到'/'。筛选器bean将被映射到所有的url('/*')。
>   >   >
>   >   > - 对于更高级的配置，上下文可以定义实现ServletContextInitializer接口的bean(通常是ServletRegistrationBeans和/或FilterRegistrationBeans) 。为了防止重复注册，ServletContextInitializer bean的使用将禁用自动Servlet和Filter bean注册。
>   >   >
>   >   > - 虽然这个上下文可以直接使用，但是大多数开发人员应该考虑使用AnnotationConfigServletWebServerApplicationContext或XmlServletWebServerApplicationContext的变体。
>
> - 

# BeanFactoryPostProcessor

> - 工厂钩子，允许自定义修改应用程序上下文的bean定义，调整上下文的底层bean工厂的bean属性值。 适用于针对系统管理员的定制配置文件，这些配置文件覆盖了在应用程序上下文中配置的bean属性。请参阅PropertyResourceConfigurer及其具体实现，了解满足此类配置需求的开箱即用解决方案。 **BeanFactoryPostProcessor可以与bean定义进行交互和修改，但不能与bean实例进行交互。**这样做可能会导致bean过早实例化，违反容器并导致意想不到的副作用。**如果需要bean实例交互，可以考虑实现BeanPostProcessor**。
>
> - 注册
>
>   ApplicationContext在它的bean定义中自动检测BeanFactoryPostProcessor bean，并在创建任何其他bean之前应用它们。一个BeanFactoryPostProcessor也可以通过编程的方式注册一个ConfigurableApplicationContext。

# BeanPostProcessor

> - 工厂钩子，允许自定义修改新bean实例——例如，检查标记接口或用代理包装bean。
>
> - 通常，通过标记接口或类似的方式填充bean的后处理器将实现postProcessBeforeInitialization， 而用代理包装bean的后处理器通常将实现postProcessAfterInitialization。
>
> - 注册
>
>   ApplicationContext可以在它的bean定义中自动检测BeanPostProcessor bean，并将这些后处理器应用到随后创建的任何bean。 普通的BeanFactory允许以编程方式注册后处理程序，将它们应用于通过bean工厂创建的所有bean。
>
> - postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)





# BeanFactory

> - 默认创建的**DefaultListableBeanFactory**的实例。
>
>   > - AnnotationConfigServletWebServerApplicationContext无参构造 -> GenericApplicationContext无参构造。
>   >
>   > - DefaultListableBeanFactory
>   >
>   >   > - Spring对ConfigurableListableBeanFactory和BeanDefinitionRegistry接口的默认实现:一个基于bean定义元数据的成熟的bean工厂，可以通过后处理器进行扩展。
>   >   > - 典型的用法是在访问bean之前，首先注册所有bean定义(可能是从bean定义文件中读取)。因此，按名称查找Bean是本地Bean定义表中的一种成本较低的操作，它对预先解析的Bean定义元数据对象进行操作。
>   >
>   > - DefaultListableBeanFactory的supertypes hierarchy
>   >
>   >   > ![DefaultListableBeanFactory](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/DefaultListableBeanFactory.jpg)
>   >   >
>   >   > - SingletonBeanRegistry
>   >   >
>   >   >   > 为**共享bean实例**（即单例）定义注册表的接口。注意是注册bean实例而非bean定义。
>   >   >
>   >   > - BeanDefinitionRegistry
>   >   >
>   >   >   > - 保存**bean定义**的注册中心的接口。注意是注册的bean定义。
>   >   >   > - DefaultListableBeanFactory对其方法的实现大多在其超类中，如SimpleAliasRegistry和AbstractBeanFactory。
>   >   >
>   >   > - SimpleAliasRegistry
>   >   >
>   >   >   > AliasRegistry接口的简单实现。作为org.springframework.beans.factory.support.BeanDefinitionRegistry实现的基类。
>   >   >
>   >   > - DefaultSingletonBeanRegistry
>   >   >
>   >   >   > - 共享bean实例的通用注册中心，实现了**SingletonBeanRegistry**。允许注册应该为注册表的所有调用者共享的单例实例，通过bean名获得。
>   >   >   > - 还支持注册可销毁的bean实例(它可能与已注册的单例对应，也可能不对应)，在注册表关闭时销毁。可以注册bean之间的依赖项以强制执行适当的关闭顺序。
>   >   >   > - 这个类主要作为org.springframework.beans.factory.BeanFactory实现的基类，分解出单例bean实例的公共管理。注意，org.springframework.beans.factory .config.ConfigurableBeanFactory接口扩展了SingletonBeanRegistry接口。
>   >   >   > - 请注意，与AbstractBeanFactory和DefaultListableBeanFactory(继承自该类)相比，该类既不假设bean定义概念，也不假设bean实例的特定创建过程。也可以作为可委托的嵌套助手使用。
>   >   >
>   >   > - FactoryBeanRegistrySupport
>   >   >
>   >   >   > - 支持需要处理FactoryBean实例的单例注册表的基类，集成了DefaultSingletonBeanRegistry的单例管理。 作为AbstractBeanFactory的基类。
>   >   >   >
>   >   >   > - 注意：getObjectFromFactoryBean
>   >   >
>   >   > - HierarchicalBeanFactory
>   >   >
>   >   >   > - 由bean工厂实现的子接口，可以是层次结构的一部分。
>   >   >   > - 允许以可配置方式设置父类的bean工厂的相应setParentBeanFactory方法可以在ConfigurableBeanFactory接口中找到。
>   >   >
>   >   > - ConfigurableBeanFactory
>   >   >
>   >   >   > - 由大多数bean工厂实现的配置接口。除了BeanFactory接口中的bean工厂客户端方法外，还提供配置bean工厂的工具。
>   >   >   > - 这个bean工厂接口并不打算用于普通的应用程序代码:对于典型的需求，请坚持使用BeanFactory或ListableBeanFactory。这个扩展的接口只是为了允许框架内部即插即用，以及对bean工厂配置方法的特殊访问。
>   >   >
>   >   > - AbstractBeanFactory
>   >   >
>   >   >   > - BeanFactory实现的抽象基类，提供了ConfigurableBeanFactory SPI的全部功能。不假设有一个listable bean工厂:因此也可以用作从后端资源(其中bean定义访问是一项开销很大的操作) 获取bean定义的bean工厂实现的基类。
>   >   >   > - 这个类提供了一个单例缓存(通过它的基类DefaultSingletonBeanRegistry、单例/原型确定、FactoryBean处理、别名、子bean定义的bean定义合并和bean销毁（可丢弃的bean接口、自定义销毁方法））。此外，它还可以通过实现HierarchicalBeanFactory接口来管理bean工厂层次结构(在未知bean的情况下委托给父类)。
>   >   >   > - **子类要实现的主要模板方法是getBeanDefinition和createBean**，分别为给定的bean名检索bean定义和为给定的bean定义创建bean实例。这些操作的**默认实现**可以在**DefaultListableBeanFactory**和**AbstractAutowireCapableBeanFactory**中找到。
>   >   >
>   >   > - AbstractAutowireCapableBeanFactory
>   >   >
>   >   >   > - 抽象bean工厂超类，实现了默认的bean创建，具有由RootBeanDefinition类指定的全部功能。除了AbstractBeanFactory的createBean方法之外，还实现了AutowireCapableBeanFactory接口。
>   >   >   > - 提供bean创建(通过构造函数解析)、属性填充、连接(包括自动装配)和初始化。 处理运行时bean引用、解析托管集合、调用初始化方法等。支持自动装配构造函数、按名称的属性和按类型的属性。
>   >   >   > - **子类要实现的主要模板方法是resolveDependency(DependencyDescriptor, String, Set, TypeConverter)**，用于按类型自动装配。 对于能够搜索其bean定义的工厂，匹配的bean通常通过这样的搜索来实现。对于其他工厂样式，可以实现简化的匹配算法。
>   >   >   > - 请注意，该类不假设或实现bean定义注册表功能。ListableBeanFactory和BeanDefinitionRegistry接口的实现请参见DefaultListableBeanFactory，它们分别表示这样一个工厂的API和SPI视图。
>   >   >
>   >   > - ListableBeanFactory
>   >   >
>   >   >   > - BeanFactory接口的扩展，由**可以枚举其所有bean实例**的bean工厂来实现，而不是按照客户端的要求，逐个尝试按名称查找bean实例。预加载所有bean定义(例如基于xml的工厂)的BeanFactory实现可以实现这个接口。
>   >   >   > - 如果这是一个HierarchicalBeanFactory，返回值将**不考虑任何BeanFactory层次结构，而是只与当前工厂中定义的bean相关**。也可以使用BeanFactoryUtils helper类来考虑祖先工厂中的bean。
>   >   >   > - 这个接口中的方法将只尊重这个工厂的bean定义。它们会忽略任何已经通过其他方式注册的单例bean，比如ConfigurableBeanFactory的registerSingleton方法，除了getBeanNamesForType和getBeansOfType，它们也会检查手动注册的单例bean。当然，BeanFactory的getBean也允许对这些特殊的bean进行透明访问。但是，在典型的场景中，所有bean都将由外部bean定义，因此大多数应用程序不需要担心这种区别。
>   >   >   > - 注意:除了getBeanDefinitionCount和containsBeanDefinition之外，此接口中的方法不是为频繁调用而设计的。实现可能会很慢。
>
> 

# FactoryBean

> - 由BeanFactory中使用的对象实现的接口，BeanFactory本身就是单个对象的工厂。如果bean实现了这个接口，那么它将被用作要公开的对象的工厂，而不是直接用作将自身公开的bean实例。
> - 注意:实现此接口的bean不能作为普通bean使用。FactoryBean是以bean样式定义的，但是为bean引用公开的对象(getObject())始终是它创建的对象。

# WebServer

# ServletWebServerFactory

> - 工厂接口，可以用来创建一个WebServer。
>
> - servlet web 应用中的实现 -> **TomcatServletWebServerFactory**
>
>   >  **TomcatServletWebServerFactory**
>   >
>   > ![TomcatServletWebServerFactory](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/TomcatServletWebServerFactory-8906994.jpg)
>   >
>   > - 可以用来创建TomcatWebServers的AbstractServletWebServerFactory。可以使用Spring的servletcontextinitializer或Tomcat 的LifecycleListeners来初始化。
>   >   除非显式配置，否则该工厂将创建在端口8080上侦听HTTP请求的容器。
>   >
>   > - spring-boot-autoconfigure -> spring.factory
>   >
>   >   -> EnableAutoConfiguration=ServletWebServerFactoryAutoConfiguration
>   >
>   >   -> @ConditionalOnWebApplication(type = Type.SERVLET) & @Import({ServletWebServerFactoryConfiguration.**EmbeddedTomcat**.class})
>   >
>   >   -> @ConditionalOnClass({ Servlet.class, Tomcat.class, UpgradeProtocol.class })决定了激活EmbeddedTomcat。
>   >
>   >   通过@Bean生成TomcatServletWebServerFactory实例。
>   >
>   > 





LazyInitializationBeanFactoryPostProcessor + LazyInitializationExcludeFilter

ImportBeanDefinitionRegistrar

ForwardedHeaderFilter & FilterRegistrationBean



