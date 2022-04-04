---
title: Spring
date: 2021-08-29 22:07:06
tags: Spring
categories: recycle-bin
---

# 1. Spring 循环依赖及三级缓存

DefaultSingletonBeanRegistry

- singletonObjects【一级缓存】
  - 单例对象的缓存: beanName->bean 实例
- singletonFactories【三级缓存】
  - 单例工厂的缓存: beanName->ObjectFactory 实例
- earlySingletonObjects【二级缓存】
  - 早期单例对象的缓存: beanName->bean 实例

Spring启动过程大致如下：

1. 创建beanFactory，加载配置文件；
2. 解析配置文件转化beanDefination，获取到bean的所有属性、依赖及初始化用到的各类处理器等；
3. 刷新beanFactory容器，初始化所有单例bean；
4. 注册所有的单例bean并返回可用的容器，一般为扩展的applicationContext。

**一级缓存**

在第三步中，所有单例的bean初始化完成后会存放在一个Map(singletonObjects)中，beanName为key，单例bean为value。

第三步单例bean的初始化过程大致如下：

1. 标记bean为创建中；
2. new出bean对象；
3. 如果支持循环依赖则生成三级缓存，可以提前暴露bean；
4. 填充bean属性，解决属性依赖；
5. 初始化bean，处理Aware接口并执行各类bean后处理器，执行初始化方法，如果需要生成aop代理对象；
6. 如果存在循环依赖，解决之 – 这里有点问题，这一步是如果之前解决了aop循环依赖，则缓存中放置了提前生成的代理对象，然后使用原始bean继续执行初始化，所以需要再返回最终bean前，把原始bean置换为代理对象返回；
7. 此时bean已经可以被使用，进行bean注册(标记)并注册销毁方法；
8. 将bean放入容器中(一级缓存)，移除创建中标记及二三级缓存(后面再具体分析)。



### getBean   <-  AbstractBeanFactory

```java
  //Implementation of BeanFactory interface  

  @Override
	public Object getBean(String name) throws BeansException {
		return doGetBean(name, null, null, false);
	}

	@Override
	public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
		return doGetBean(name, requiredType, null, false);
	}

	@Override
	public Object getBean(String name, Object... args) throws BeansException {
		return doGetBean(name, null, args, false);
	}
```

### doGetBean  <-  AbstractBeanFactory

- 返回指定 bean 的一个实例，该实例可以是共享的，也可以是独立的。

```java
protected <T> T doGetBean(
  String name, @Nullable Class<T> requiredType, @Nullable Object[] args, boolean typeCheckOnly)
  throws BeansException {
 // 将别名解析为规范名称。
 String beanName = transformedBeanName(name);
 Object beanInstance;

 // 检查bean是否已经存在了缓存中
 Object sharedInstance = getSingleton(beanName);
 if (sharedInstance != null && args == null) {
  // 表明之前被创建过
  if (logger.isTraceEnabled()) {
   // 检查该bean单例是否正在创建中（整个工厂范围内）。
   if (isSingletonCurrentlyInCreation(beanName)) {
    logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
      "' that is not fully initialized yet - a consequence of a circular reference");
   }
   else {
    logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
   }
  }
  // 获取给定 bean 实例的对象，如果是 FactoryBean，则是 bean 实例本身或其创建的对象。
  // 这里对于普通的bean，则会直接的返回，
  // 如果是FactoryBean类型的则会创建对应的实例返回
  beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, null);
 }

 else {
  // 指定的原型bean是否正在创建中(在当前线程中)。
  // 如果是正在创建的Prototype类型的bean,无法处理该类型循环依赖的问题，则直接抛出异常信息
  if (isPrototypeCurrentlyInCreation(beanName)) {
   throw new BeanCurrentlyInCreationException(beanName);
  }

  // Check if bean definition exists in this factory.
  // 对IOC容器中是否存在指定名称的BeanDefinition进行检查，首先检查是否能在当前的BeanFactory中获取到所需要的Bean定义，如果不能则委托当前容器的父级容器去查找，如果还是找不到则沿着容器的继承体系向父级容器查找。
  BeanFactory parentBeanFactory = getParentBeanFactory();
  if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
   // Not found -> check parent.
   String nameToLookup = originalBeanName(name);
   if (parentBeanFactory instanceof AbstractBeanFactory) {
    return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
      nameToLookup, requiredType, args, typeCheckOnly);
   }
   else if (args != null) {
    // Delegation to parent with explicit args.
    return (T) parentBeanFactory.getBean(nameToLookup, args);
   }
   else if (requiredType != null) {
    // No args -> delegate to standard getBean method.
    return parentBeanFactory.getBean(nameToLookup, requiredType);
   }
   else {
    return (T) parentBeanFactory.getBean(nameToLookup);
   }
  }
   
  // 到此处，说明当前IOC容器中有该bean的定义。开始创建bean。
   
	// typeCheckOnly: 实例是否为类型检查获取，不用于实际使用。
  if (!typeCheckOnly) {
   // 将指定的 bean 标记为已创建（或即将创建）。这允许 bean 工厂优化其缓存以重复创建指定 bean。
   markBeanAsCreated(beanName);
  }
  // Start：开始创建该beanName对应实例
  StartupStep beanCreation = this.applicationStartup.start("spring.beans.instantiate")
    .tag("beanName", name);
  try {
   if (requiredType != null) {
    beanCreation.tag("beanType", requiredType::toString);
   }
   //根据指定Bean名称获取其父级的Bean定义
   //主要解决Bean继承时子类合并父类公共属性问题
   RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
   checkMergedBeanDefinition(mbd, beanName, args);

   // 获取此bean所依赖的beanNames。
   // 保证对当前bean所依赖的bean进行初始化。
   String[] dependsOn = mbd.getDependsOn();
   if (dependsOn != null) {
    for (String dep : dependsOn) {
     // 如果dep依赖或者间接依赖beanName，则说明发生了环状依赖，即循环引用，抛出异常。
     if (isDependent(beanName, dep)) {
      throw new BeanCreationException(mbd.getResourceDescription(), beanName,
        "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
     }
     // 将beanName注册到dep的依赖和被依赖集合中。
     registerDependentBean(dep, beanName);
     try {
      // 获取beanName的依赖dep的实例。
      getBean(dep);
     }
     catch (NoSuchBeanDefinitionException ex) {
      throw new BeanCreationException(mbd.getResourceDescription(), beanName,
        "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
     }
    }
   }

   // Create bean instance.
   if (mbd.isSingleton()) {
    sharedInstance = getSingleton(beanName, () -> {
     try {
      return createBean(beanName, mbd, args);
     }
     catch (BeansException ex) {
      // Explicitly remove instance from singleton cache: It might have been put there
      // eagerly by the creation process, to allow for circular reference resolution.
      // Also remove any beans that received a temporary reference to the bean.
      destroySingleton(beanName);
      throw ex;
     }
    });
    beanInstance = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
   }
		
   else if (mbd.isPrototype()) {
    // It's a prototype -> create a new instance.
    Object prototypeInstance = null;
    try {
     beforePrototypeCreation(beanName);
     prototypeInstance = createBean(beanName, mbd, args);
    }
    finally {
     afterPrototypeCreation(beanName);
    }
    beanInstance = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
   }

   else {
    String scopeName = mbd.getScope();
    if (!StringUtils.hasLength(scopeName)) {
     throw new IllegalStateException("No scope name defined for bean ´" + beanName + "'");
    }
    Scope scope = this.scopes.get(scopeName);
    if (scope == null) {
     throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
    }
    try {
     Object scopedInstance = scope.get(beanName, () -> {
      beforePrototypeCreation(beanName);
      try {
       return createBean(beanName, mbd, args);
      }
      finally {
       afterPrototypeCreation(beanName);
      }
     });
     beanInstance = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
    }
    catch (IllegalStateException ex) {
     throw new ScopeNotActiveException(beanName, scopeName, ex);
    }
   }
  }
  catch (BeansException ex) {
   beanCreation.tag("exception", ex.getClass().toString());
   beanCreation.tag("message", String.valueOf(ex.getMessage()));
   cleanupAfterBeanCreationFailure(beanName);
   throw ex;
  }
  finally {
   // End：bean 创建结束。
   beanCreation.end();
  }
 }
 // 若beanInstance不是所需类型requiredType，则将其转化为requiredType型。 
 return adaptBeanInstance(name, beanInstance, requiredType);
}
```

### getSingleton  <-  DefaultSingletonBeanRegistry

- 返回在给定名称下注册的（原始）单例对象。检查已经实例化的单例，并允许早期引用当前正在创建的单例（解决循环引用）。

```java
  @Override
	@Nullable
	public Object getSingleton(String beanName) {
		return getSingleton(beanName, true);
	}
	/**
	 * 返回在给定名称下注册的(原始)单例对象。
	 * 检查已经实例化的单例对象，并允许对当前创建的单例对象的早期引用(解析循环引用)。
	 * 早起引用即对未完全实例化（如未完成属性设置）的引用。
	 **/
  @Nullable
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		// Quick check for existing instance without full singleton lock
		Object singletonObject = this.singletonObjects.get(beanName);
    // 若 一级缓存中没有，且该bean单例正在被创建
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
      // 检查二级缓存
			singletonObject = this.earlySingletonObjects.get(beanName);
			if (singletonObject == null && allowEarlyReference) {
				synchronized (this.singletonObjects) {
					// Consistent creation of early reference within full singleton lock
          // 在完整的单例锁中一致地创建早期引用。
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
						singletonObject = this.earlySingletonObjects.get(beanName);
            // 若二级缓存中没有，查询三级缓存。
						if (singletonObject == null) {
							ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
              // 若三级缓存中存在。
							if (singletonFactory != null) {
                // 使用三级缓存中其对应的ObjectFactory实例，获取单例实例。
								singletonObject = singletonFactory.getObject();
                // 将单例实例添加到二级缓存中。
								this.earlySingletonObjects.put(beanName, singletonObject);
                // 将三级缓存中的该beanName映射移除。
								this.singletonFactories.remove(beanName);
							}
						}
					}
				}
			}
		}
		return singletonObject;
	}
```

### getSingleton <- DefaultSingletonBeanRegistry

- 返回以给定名称注册的（原始）单例对象，如果尚未注册，则创建并注册一个新对象。

```java
  public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(beanName, "Bean name must not be null");
		synchronized (this.singletonObjects) {
			Object singletonObject = this.singletonObjects.get(beanName);
			if (singletonObject == null) {
				if (this.singletonsCurrentlyInDestruction) {
					throw new BeanCreationNotAllowedException(beanName,
							"Singleton bean creation not allowed while singletons of this factory are in destruction " +
							"(Do not request a bean from a BeanFactory in a destroy method implementation!)");
				}
				if (logger.isDebugEnabled()) {
					logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
				}
        // 单例创建之前的回调。默认实现将单例注册为当前正在创建中。
				beforeSingletonCreation(beanName);
				boolean newSingleton = false;
				boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
				if (recordSuppressedExceptions) {
					this.suppressedExceptions = new LinkedHashSet<>();
				}
				try {
          // 生成实例对象。
					singletonObject = singletonFactory.getObject();
					newSingleton = true;
				}
				catch (IllegalStateException ex) {
					// Has the singleton object implicitly appeared in the meantime ->
					// if yes, proceed with it since the exception indicates that state.
					singletonObject = this.singletonObjects.get(beanName);
					if (singletonObject == null) {
						throw ex;
					}
				}
				catch (BeanCreationException ex) {
					if (recordSuppressedExceptions) {
						for (Exception suppressedException : this.suppressedExceptions) {
							ex.addRelatedCause(suppressedException);
						}
					}
					throw ex;
				}
				finally {
					if (recordSuppressedExceptions) {
						this.suppressedExceptions = null;
					}
          // 创建单例后回调，默认实现将单例标记为不再创建。
					afterSingletonCreation(beanName);
				}
				if (newSingleton) {
          // 将给定的单例对象添加到该工厂的单例缓存中。要求对单例进行紧急注册。
					addSingleton(beanName, singletonObject);
				}
			}
			return singletonObject;
		}
	}
```

### addSingleton <- DefaultSingletonBeanRegistry

- 将给定的单例对象添加到该工厂的单例缓存中。要求对单例进行紧急注册。

```java
protected void addSingleton(String beanName, Object singletonObject) {
		synchronized (this.singletonObjects) {
      // 添加到一级缓存中
			this.singletonObjects.put(beanName, singletonObject);
      // 从三级缓存中移除
			this.singletonFactories.remove(beanName);
      // 从二级缓存中移除
			this.earlySingletonObjects.remove(beanName);
      // 添加到registeredSingletons
			this.registeredSingletons.add(beanName);
		}
	}
```

### getObjectForBeanInstance <- AbstractAutowireCapableBeanFactory

```java
@Override
	protected Object getObjectForBeanInstance(
			Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {

		String currentlyCreatedBean = this.currentlyCreatedBean.get();
		if (currentlyCreatedBean != null) {
      // 为给定bean注册一个依赖bean，以便在给定bean销毁之前销毁它。
			registerDependentBean(beanName, currentlyCreatedBean);
		}

		return super.getObjectForBeanInstance(beanInstance, name, beanName, mbd);
	}
```

### registerDependentBean <- DefaultSingletonBeanRegistry

- 维护dependentBeanMap和dependenciesForBean

```java
public void registerDependentBean(String beanName, String dependentBeanName) {
		String canonicalName = canonicalName(beanName);

		synchronized (this.dependentBeanMap) {
			Set<String> dependentBeans =
					this.dependentBeanMap.computeIfAbsent(canonicalName, k -> new LinkedHashSet<>(8));
      // 若已存在则直接返回。
			if (!dependentBeans.add(dependentBeanName)) {
				return;
			}
		}

		synchronized (this.dependenciesForBeanMap) {
			Set<String> dependenciesForBean =
					this.dependenciesForBeanMap.computeIfAbsent(dependentBeanName, k -> new LinkedHashSet<>(8));
			dependenciesForBean.add(canonicalName);
		}
	}
```

### getObjectForBeanInstance <- AbstractBeanFactory

```java
/**
 * 获取给定 bean 实例的对象，如果是 FactoryBean，则是 bean 实例本身或其创建的对象。
 * */
protected Object getObjectForBeanInstance(
			Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {

		// Don't let calling code try to dereference the factory if the bean isn't a factory.
		if (BeanFactoryUtils.isFactoryDereference(name)) {
      // NullBean是空bean实例的内部表示。
			if (beanInstance instanceof NullBean) {
				return beanInstance;
			}
      //name的前缀是&,但是类型不是FactoryBean，就报错
			if (!(beanInstance instanceof FactoryBean)) {
				throw new BeanIsNotAFactoryException(beanName, beanInstance.getClass());
			}
			if (mbd != null) {
				mbd.isFactoryBean = true;
			}
			return beanInstance;
		}

		// 现在我们有了bean实例，它可以是普通bean或FactoryBean。如果它是FactoryBean，我们使用它来创建bean实例，除非调用者实际上想要一个对工厂的引用。
		if (!(beanInstance instanceof FactoryBean)) {
			return beanInstance;
		}

		Object object = null;
		if (mbd != null) {
			mbd.isFactoryBean = true;
		}
		else {
      // 从给定的FactoryBean获取要公开的对象（如果缓存中有的话）。
			object = getCachedObjectForFactoryBean(beanName);
		}
		if (object == null) {
			// Return bean instance from factory.
			FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
			// Caches object obtained from FactoryBean if it is a singleton.
			if (mbd == null && containsBeanDefinition(beanName)) {
        // 返回合并的 RootBeanDefinition，如果指定的 bean 对应于子 bean 定义，则遍历其父 bean 定义。
				mbd = getMergedLocalBeanDefinition(beanName);
			}
      // 此 bean 定义是否是“合成的”，即不是由应用程序本身定义的
			boolean synthetic = (mbd != null && mbd.isSynthetic());
      // 从给定的 FactoryBean 获取要公开的对象。
			object = getObjectFromFactoryBean(factory, beanName, !synthetic);
		}
		return object;
	}
```

### isFactoryDereference <- BeanFactoryUtils

- 返回给定的名称是否为工厂的间接引用

```java
  public static boolean isFactoryDereference(@Nullable String name) {
    // BeanFactory.FACTORY_BEAN_PREFIX = "&";
		return (name != null && name.startsWith(BeanFactory.FACTORY_BEAN_PREFIX));
	}
```

### createBean <- AbstractAutowireCapableBeanFactory

- 在AbstractBeanFactory中该方法为抽象方法。

```java
  /**
   * 此类的中心方法：创建 bean 实例、填充 bean 实例、应用后处理器等。
   **/
  @Override
	protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
			throws BeanCreationException {

		if (logger.isTraceEnabled()) {
			logger.trace("Creating instance of bean '" + beanName + "'");
		}
		RootBeanDefinition mbdToUse = mbd;

    // 确保此时实际解析了bean class，并在动态解析class的情况下克隆无法存储在共享合并bean定义中的bean定义。
    // 解析指定bean definition的bean class，将bean class name 解析为Class引用(如果需要)，并将解析后的Class存储在bean definition中以供进一步使用。
		Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
		if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
			mbdToUse = new RootBeanDefinition(mbd);
			mbdToUse.setBeanClass(resolvedClass);
		}

		// Prepare method overrides.
		try {
			mbdToUse.prepareMethodOverrides();
		}
		catch (BeanDefinitionValidationException ex) {
			throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
					beanName, "Validation of method overrides failed", ex);
		}

		try {
			// 让BeanPostProcessors有机会返回一个代理而不是目标bean实例。（InstantiationAwareBeanPostProcessor）
			Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
			if (bean != null) {
				return bean;
			}
		}
		catch (Throwable ex) {
			throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
					"BeanPostProcessor before instantiation of bean failed", ex);
		}

		try {
      // 不同于默认的bean实例化，使用工厂方法和自动装配构造函数。
			Object beanInstance = doCreateBean(beanName, mbdToUse, args);
			if (logger.isTraceEnabled()) {
				logger.trace("Finished creating instance of bean '" + beanName + "'");
			}
			return beanInstance;
		}
		catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
			// A previously detected exception with proper bean creation context already,
			// or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
			throw ex;
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
		}
	}
```

### resolveBeforeInstantiation <- AbstractAutowireCapableBeanFactory

- 应用before-instantiation post-processors，解析指定bean是否有实例化之前的shortcut。

```java
  @Nullable
	protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
		Object bean = null;
		if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
			// 确保bean class在此时已实际解析。
			if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        // 确定bean的目标类型
				Class<?> targetType = determineTargetType(beanName, mbd);
				if (targetType != null) {
          // 实例化前的后处理器，这里生成bean或其代理后交给 实例化后的后处理器加工。
					bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
					if (bean != null) {
            // 实例化后的后处理器   加工完之后 这个bean会用来替换原本正常逻辑中的bean
						bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
					}
				}
			}
      // 设置beforeInstantiationResolved，其为true表示实例化前的后处理器已经启动。
			mbd.beforeInstantiationResolved = (bean != null);
		}
		return bean;
	}
```

### doCreateBean <- AbstractAutowireCapableBeanFactory

- 实际创建指定的bean。此时已经进行了创建前处理，例如检查postProcessBeforeInstantiation回调。
- 区别默认bean实例化、使用工厂方法和自动装配构造函数。

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
    // 是否暴露早期单例 early singleton。当需要创建单例，且允许循环引用，且该beanName已在创建之中，则暴露其early singleton。
		boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
				isSingletonCurrentlyInCreation(beanName));
		if (earlySingletonExposure) {
			if (logger.isTraceEnabled()) {
				logger.trace("Eagerly caching bean '" + beanName +
						"' to allow for resolving potential circular references");
			}
			addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
		}

		// 初始化bean实例。
		Object exposedObject = bean;
		try {
      // 使用 bean 定义中的属性值填充给定 BeanWrapper 中的 bean 实例。
			populateBean(beanName, mbd, instanceWrapper);
      // 初始化给定的 bean 实例，应用工厂回调以及 init 方法和 bean 后处理器。
      // Called from createBean for traditionally defined beans, and from initializeBean for existing bean instances.
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
		// 如果允许公开early singleton
		if (earlySingletonExposure) {
      // 获取early singleton
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
            // 存在beans引用了该bean的原始版本，而没有使用其最终版本。
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

### createBeanInstance <- AbstractAutowireCapableBeanFactory

- 使用适当的实例化策略为指定的 bean 创建一个新实例：工厂方法、构造函数自动装配或简单实例化。

```java
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
		// Make sure bean class is actually resolved at this point.
		Class<?> beanClass = resolveBeanClass(mbd, beanName);

		if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
			throw new BeanCreationException(mbd.getResourceDescription(), beanName,
					"Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
		}

		Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
		if (instanceSupplier != null) {
			return obtainFromSupplier(instanceSupplier, beanName);
		}

		if (mbd.getFactoryMethodName() != null) {
			return instantiateUsingFactoryMethod(beanName, mbd, args);
		}

		// Shortcut when re-creating the same bean...
		boolean resolved = false;
		boolean autowireNecessary = false;
		if (args == null) {
			synchronized (mbd.constructorArgumentLock) {
				if (mbd.resolvedConstructorOrFactoryMethod != null) {
					resolved = true;
					autowireNecessary = mbd.constructorArgumentsResolved;
				}
			}
		}
		if (resolved) {
			if (autowireNecessary) {
				return autowireConstructor(beanName, mbd, null, null);
			}
			else {
				return instantiateBean(beanName, mbd);
			}
		}

		// Candidate constructors for autowiring?
		Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
		if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
				mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
			return autowireConstructor(beanName, mbd, ctors, args);
		}

		// Preferred constructors for default construction?
		ctors = mbd.getPreferredConstructors();
		if (ctors != null) {
			return autowireConstructor(beanName, mbd, ctors, null);
		}

		// No special handling: simply use no-arg constructor.
		return instantiateBean(beanName, mbd);
	}
```

### addSingletonFactory <- DefaultSingletonBeanRegistry

- 如果需要的话，添加给定的单例工厂来构建指定的单例。用于单例的紧急注册，例如能够解析循环引用。

```java
  protected void addSingletonFactory(String beanName, ObjectFactory<?> singletonFactory) {
		Assert.notNull(singletonFactory, "Singleton factory must not be null");
		synchronized (this.singletonObjects) {
			if (!this.singletonObjects.containsKey(beanName)) {
        // 添加三级缓存
				this.singletonFactories.put(beanName, singletonFactory);
        // 移除二级缓存
				this.earlySingletonObjects.remove(beanName);
        // 注册该单例
				this.registeredSingletons.add(beanName);
			}
		}
	}
```

### getEarlyBeanReference <- AbstractAutowireCapableBeanFactory

- 获取对指定 bean 的早期访问的引用，通常用于解析循环引用。
- SmartInstantiationAwareBeanPostProcessor

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



### initializeBean <- AbstractAutowireCapableBeanFactory

- 初始化给定的 bean 实例，应用工厂回调以及 init 方法和 bean 后处理器。
- Called from createBean for traditionally defined beans, and from initializeBean for existing bean instances.

```java
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
 // Aware 方法调用：BeanNameAware、BeanClassLoaderAware、BeanFactoryAware
 if (System.getSecurityManager() != null) {
  AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
   invokeAwareMethods(beanName, bean);
   return null;
  }, getAccessControlContext());
 }
 else {
  invokeAwareMethods(beanName, bean);
 }

 Object wrappedBean = bean;
 if (mbd == null || !mbd.isSynthetic()) {
  // 对wrappedBean应用所有已注册的BeanPostProcessor的postProcessBeforeInitialization方法。
  wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
 }

 try {
  // 现在，所有属性都设置好了，让bean有机会作出反应，并有机会了解其所属的bean工厂(此对象)。这意味着检查bean是否实现了InitializingBean或定义了自定义的init方法，如果实现了，则调用必要的回调。
  invokeInitMethods(beanName, wrappedBean, mbd);
 }
 catch (Throwable ex) {
  throw new BeanCreationException(
    (mbd != null ? mbd.getResourceDescription() : null),
    beanName, "Invocation of init method failed", ex);
 }
 if (mbd == null || !mbd.isSynthetic()) {
  // 对wrappedBean应用所有已注册的BeanPostProcessor的 postProcessAfterInitialization 方法。
  wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
 }

 return wrappedBean;
}
```

### invokeInitMethods <- AbstractAutowireCapableBeanFactory

- 现在，所有属性都设置好了，让bean有机会作出反应，并有机会了解其所属的bean工厂(此对象)。这意味着检查bean是否实现了InitializingBean或定义了自定义的init方法，如果实现了，则调用必要的回调。

```java
protected void invokeInitMethods(String beanName, Object bean, @Nullable RootBeanDefinition mbd)
  throws Throwable {

 boolean isInitializingBean = (bean instanceof InitializingBean);
 if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
  // 调用 afterPropertiesSet方法。此方法允许 bean 实例在设置所有 bean 属性后执行其整体配置和最终初始化的验证。
  if (logger.isTraceEnabled()) {
   logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
  }
  if (System.getSecurityManager() != null) {
   try {
    AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
     ((InitializingBean) bean).afterPropertiesSet();
     return null;
    }, getAccessControlContext());
   }
   catch (PrivilegedActionException pae) {
    throw pae.getException();
   }
  }
  else {
   ((InitializingBean) bean).afterPropertiesSet();
  }
 }

 if (mbd != null && bean.getClass() != NullBean.class) {
  String initMethodName = mbd.getInitMethodName();
  if (StringUtils.hasLength(initMethodName) &&
    !(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
    !mbd.isExternallyManagedInitMethod(initMethodName)) {
   // 在给定的bean上调用指定的自定义init方法。叫invokeInitMethods。可以在子类中重写，以便自定义带有参数的init方法解析。
   invokeCustomInitMethod(beanName, bean, mbd);
  }
 }
}
```



------

> 内容来源：
>
> [Spring系列第56篇：一文搞懂spring到底为什么要用三级缓存](https://itsoku.blog.csdn.net/article/details/113977261)

