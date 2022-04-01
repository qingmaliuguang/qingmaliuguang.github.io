---
title: SpringBoot-web-问题记录
date: 2022-03-21 15:06:25
tags: SpringBoot
---

### 1. lib spring-core 无法关联源码？
解决：spring-core依赖中的jar是0字节，需删除重新倒入该依赖。即可。
### 2. 中文编码问题？
参考：https://docs.spring.io/spring-boot/docs/2.6.4/reference/htmlsingle/#web.servlet.spring-mvc.message-converters
中17.4.4。
说明：Spring MVC使用HttpMessageConverter接口来转换HTTP请求和响应。明智的默认值包括在开箱即用。例如，对象可以自动转换为JSON(通过使用Jackson库)或XML(如果Jackson XML扩展可用，则使用Jackson XML扩展;如果Jackson XML扩展不可用，则使用JAXB)。默认情况下，字符串采用UTF-8编码。
如果你需要添加或自定义转换器，你可以使用Spring Boot的HttpMessageConverters类，如下所示:
```java
@Configuration(proxyBeanMethods = false)
public class MyHttpMessageConvertersConfiguration {

  @Bean
  public HttpMessageConverters customConverters() {
    HttpMessageConverter<?> additional = new AdditionalHttpMessageConverter();
    HttpMessageConverter<?> another = new AnotherHttpMessageConverter();
    return new HttpMessageConverters(additional, another);
  }

}
```
上下文中出现的任何HttpMessageConverter bean都被添加到转换器列表中。您也可以用同样的方法重写默认转换器。
Spring使用HttpMessageConverters来渲染@ResponseBody(或@RestController的响应)。您可以通过在Spring Boot上下文中添加适当类型的bean来贡献其他转换器。如果您添加的bean是默认情况下已经包含的类型(例如用于JSON转换的MappingJackson2HttpMessageConverter)，那么它将替换默认值。提供了HttpMessageConverters类型的方便bean，如果您使用默认的MVC配置，它总是可用的。它有一些有用的方法来访问默认的和用户增强的消息转换器(例如，如果您想手动将它们注入到自定义的RestTemplate中，它可能很有用)。
解决：如HttpMessageConverterConfig。

### 3. @Aspect 找不到。

解决：添加依赖spring-boot-starter-aop，它会引入aspectjweaver依赖，@Aspect为其中注解。

### 4. 未生效

解决：@Aspect类要通过添加@Component或@Bean等注解由容器创建其实例才能生效。

### 5. @After逻辑中通过Method对象获取的方法上的注解列表不完整

见：BizLogAspect。

原因：自定义的注解BizLogAnnotation没有添加元注解@Retention(RetentionPolicy.RUNTIME)。

> 扩展：通过代理模式生成代理对象上的方法不会有注解，我们要取注解只能从目标对象中取。
>
> 错误写法：
>
> ```java
> // 此处method获取的是代理对象（由代理模式生成的对象）的方法
> Method method1 = ((MethodSignature) joinPoint.getSignature()).getMethod();
> // 此处annotation==null
> Annotation annotation = method1.getAnnotation(BizLogAnnotation.class);
> ```
>
> 正确写法：
>
> ```java
> targetClass = Class.forName(targetName);
> // 此处是目标对象的原始方法。
> Method[] methods = targetClass.getMethods();
> BizLogAnnotation bizLogAnnotation = methods[0].getAnnotation(BizLogAnnotation.class);
> ```

### 6. @Indexed注解

注释中描述：

> 指示带注解的元素表示索引的构造型。
>
> CandidateComponentsIndex是类路径扫描的替代方法，它使用在编译时生成的元数据文件。索引允许基于原型检索候选组件(即完全限定的名称) 。该注解指示生成器对存在注释元素的元素进行索引，或者对注释元素进行实现或扩展。原型是被注释元素的完全限定名称。
>
> 考虑默认的Component注解，它是用这个注解进行元注解的。如果一个组件用Component来注解，那么该组件的一个条目将会被org.springframework.stereotype.Component模板添加到索引中。
>
> 这个注解也在元注解中得到了尊重。考虑如下定制注解:
>
> ```java
> package com.example;
>   
>    @Target(ElementType.TYPE)
>    @Retention(RetentionPolicy.RUNTIME)
>    @Documented
>    @Indexed
>    @Service
>    public @interface PrivilegedService { ... }
> ```
>
> 如果上面的注释出现在一个类型上，它将被两个原型索引:org.springframework.stereotype.Component和com.example .PrivilegedService。虽然Service不是直接用Indexed注释的，但它是用Component元注释的。
>
> 也可以通过添加@Indexed来索引某个接口的所有实现或者给定类的所有子类。考虑以下基本接口:
>
> ```java
> package com.example;
>   
>    @Indexed
>    public interface AdminService { ... }
> ```
>
> 现在，考虑在某个地方实现这个AdminService:
>
> ```java
> package com.example.foo;
>   
>    import com.example.AdminService;
>   
>    public class ConfigurationAdminService implements AdminService { ... }
> ```
>
> 因为这个类实现了一个被索引的接口，所以它将被自动包含在com.example.AdminService原型中。如果层次结构中有更多的@Indexed接口和/或超类，类将映射到它们所有的原型。

其它参考：[@Indexed注解](https://www.cnblogs.com/54chensongxia/p/14389134.html)、[官方文档](https://docs.spring.io/spring-framework/docs/5.2.6.RELEASE/spring-framework-reference/core.html#beans-scanning-index)，但是6.0.0-M2版本官方文档中无相关介绍。

### 7. @Scope注解是怎么生效的？

ConfigurationClassPostProcessor

> BeanDefinitionRegistryPostProcessor
>
> - 扩展到标准的BeanFactoryPostProcessor SPI，允许在常规BeanFactoryPostProcessor检测开始之前注册更多的bean定义。特别是，BeanDefinitionRegistryPostProcessor可以注册更多的bean定义，这些定义反过来又定义了BeanFactoryPostProcessor实例。
> - 继承自BeanFactoryPostProcessor。

-> **ConfigurationClassBeanDefinitionReader**:loadBeanDefinitions中会检查有无@Scope注解，若有则根据其属性填充bean定义。

### 8. 





# 待整理内容

- DefaultAopProxyFactory

- AbstractAspectJAdvisorFactory -> ReflectiveAspectJAdvisorFactory

  > Spring AOP支持的Aspectj注解：Pointcut.class, Around.class, Before.class, After.class, AfterReturning.class, AfterThrowing.class。

- AnnotationUtils
