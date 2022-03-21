---
title: Spring-AOP
date: 2021-08-26 08:44:07
tags: Spring
---



@EnableAspectJAutoProxy -> @Import(AspectJAutoProxyRegistrar.class) -> 注册一个适当的AnnotationAwareAspectJAutoProxyCreator 的BeanDefinition。

-> 经过IOC



advice：通知，即增强。

advisor：advice的载体。

spring实现AOP思路:

1： 创建`AnnotationAwareAspectJAutoProxyCreator`对象

2： 扫描容器中的切面，创建`PointcutAdvisor`对象
3： 生成代理类

![img](https://gitee.com/qmlg/image-bed/raw/master/images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hjMTIzX2phdmE=,size_16,color_FFFFFF,t_70.png)

AbstractAutoProxyCreator中实现的postProcessBeforeInstantiation与postProcessAfterInitialization。

postProcessBeforeInstantiation

-> createProxy

postProcessAfterInitialization：如果 bean 被子类标识为代理，则使用配置的拦截器创建一个代理。

-> wrapIfNecessary

-> createProxy





ProxyFactory：AOP 代理工厂，以编程的方式使用，而不是通过 bean 工厂中的声明式设置。 此类提供了一种在自定义用户代码中获取和配置 AOP 代理实例的简单方法。

- Spring并没有提供其它的实现类。



AopProxyFactory

由能够基于AdvisedSupport配置对象创建 AOP 代理的工厂实现的接口。
代理人应遵守以下合同：

- 它们应该实现配置指示应该被代理的所有接口。
- 他们应该实现Advised接口。
- 他们应该实现 equals 方法来比较代理接口、通知和目标。
- 如果所有顾问程序和目标都是可序列化的，则它们应该是可序列化的。
- 如果顾问和目标是线程安全的，它们应该是线程安全的。
  代理可能允许也可能不允许更改建议。 如果他们不允许建议更改（例如，因为配置被冻结），代理应该在尝试更改建议时抛出AopConfigException 。

Spring中只有一个实现类DefaultAopProxyFactory。

**DefaultAopProxyFactory**

默认AopProxyFactory实现，创建 CGLIB 代理或 JDK 动态代理。
如果对于给定的AdvisedSupport实例满足以下任一条件，则创建 CGLIB 代理：

- optimize标志已设置
- 设置了proxyTargetClass标志
- 没有指定代理接口
  通常，指定proxyTargetClass以强制执行 CGLIB 代理，或指定一个或多个接口以使用 JDK 动态代理。

createAopProxy

AopProxy

![image-20210826222938687](https://gitee.com/qmlg/image-bed/raw/master/images/image-20210826222938687.png)

Proxy

Proxy提供了创建动态代理类和实例的静态方法，它也是由这些方法创建的所有动态代理类的超类。



java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

而cglib动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

1、如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP 
2、如果目标对象实现了接口，可以强制使用CGLIB实现AOP 

3、如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

JDK动态代理和CGLIB字节码生成的区别？
 （1）JDK动态代理只能对实现了接口的类生成代理，而不能针对类
 （2）CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法
  因为是继承，所以该类或方法最好不要声明成final 



创建的代理类是怎么使用上的呢？

AbstractAutowireCapableBeanFactory

- initializeBean
  - applyBeanPostProcessorsBeforeInitialization
  - invokeInitMethods
  - applyBeanPostProcessorsAfterInitialization

```java
@Override
	public Object applyBeanPostProcessorsBeforeInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
      // 返回的对象替换了原对象，代理类是在这时候完成的替换。
			Object current = processor.postProcessBeforeInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}

@Override
	public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
			throws BeansException {

		Object result = existingBean;
		for (BeanPostProcessor processor : getBeanPostProcessors()) {
			Object current = processor.postProcessAfterInitialization(result, beanName);
			if (current == null) {
				return result;
			}
			result = current;
		}
		return result;
	}
```







Filter 与 Interceptor 的区别。

HandlerInterceptor基本上类似于Servlet Filter，但与后者相比，它只允许自定义预处理(禁止处理程序本身的执行)和自定义后处理。过滤器更强大，例如，它们允许交换链下传递的请求和响应对象。注意，过滤器是在web.xml中配置的，这是应用程序上下文中的HandlerInterceptor。

[filter和interceptor的区别](https://www.cnblogs.com/jichi/p/12805719.html)

[拦截器（Interceptor）和过滤器（Filter）的执行顺序和区别](https://blog.csdn.net/zxd1435513775/article/details/80556034)

filter的执行在interceptor之前。

该图仅供参考，感觉并不是很准确。

![这里写图片描述](https://gitee.com/qmlg/image-bed/raw/master/images/70.png)





```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
        // 获取当前request的handler
				mappedHandler = getHandler(processedRequest);
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

        // 应用所有注册拦截器的 preHandle 方法，若其中一个返回false则中断返回。
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

				applyDefaultViewName(processedRequest, mv);
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
```

Filter什么时候调用的？

嵌入服务器，如tomcat的ApplicationDispatcher。

doDispatch

->invoke

-> 

```java
ApplicationFilterChain filterChain =
                ApplicationFilterFactory.createFilterChain(request, wrapper, servlet);
                ......
                if ((servlet != null) && (filterChain != null)) {
               filterChain.doFilter(request, response);
             }
```

```
internalDoFilter
filter.doFilter(request, response, this);


......
servlet.service(request, response);
```

- Filter & HandlerInterceptoer & MethodInterceptor

参考：https://blog.csdn.net/guyue35/article/details/104658234
