---
title: Spring应用启动过程-2
date: 2022-04-07 18:16:25
tags: SpringBoot
categories: 整理-来自代码
---

# 关于注解注入

- CommonAnnotationBeanPostProcessor

  > - 支持开箱即用的公共Java注解的BeanPostProcessor实现，特别是jakarta.annotation包中的公共注解。许多Jakarta EE技术(例如JSF和JAX-RS)都支持这些公共Java注解。 
  > - 这个后处理程序包括对PostConstruct和PreDestroy注解的支持——分别作为init注解和destroy注解——通过从InitDestroyAnnotationBeanPostProcessor 继承预配置的注解类型。
  > -  中心元素是Resource注解，用于命名beans的注解驱动注入，默认情况下来自包含的Spring BeanFactory，在JNDI中只解析了映射的名称引用。“alwaysUseJndiLookup”标志强制JNDI查找等同于标准的Jakarta EE资源注入，用于名称引用和默认名称。目标bean可以是简单的POJOs，除了必须匹配的类型外，没有其他特殊需求。 
  > -  注意:“context:annotation-config”和“context:component-scan”XML标签将会注册一个默认的CommonAnnotationBeanPostProcessor。如果您打算指定一个自定义的CommonAnnotationBeanPostProcessor bean定义，请删除或关闭那里的默认注解配置! 
  > -  注:**注解注入将在XML注入之前执行**;因此，对于通过两种方法连接的属性，后一种配置将覆盖前一种配置。 

