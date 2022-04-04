---
title: Spring 事务
date: 2021-08-29 12:13:37
tags: Spring
categories: in-use
---

# 1. Spring事务传播特性

- @Transactional.propagation

- enum Propagation
- interface TransactionDefinition

**REQUIRED**

- 支持当前事务，如果不存在则创建一个新事务。
- 这是事务注解的**默认**设置。

**SUPPORTS**

- 支持当前事务，如果不存在则以非事务方式执行。
- 注意：对于具有事务同步的事务管理器， SUPPORTS与根本没有事务略有不同，因为它定义了同步将应用的事务范围。 因此，相同的资源（JDBC 连接、Hibernate 会话等）将在整个指定范围内共享。 请注意，这取决于事务管理器的实际同步配置。
- See Also:
  org.springframework.transaction.support.AbstractPlatformTransactionManager.setTransactionSynchronization

**MANDATORY**

- 支持当前事务，如果不存在则抛出异常。

**REQUIRES_NEW**

- 创建一个新事务，如果存在，则暂停当前事务。
- 注意：实际的事务暂停不会在所有事务管理器上开箱即用。 这尤其适用于org.springframework.transaction.jta.JtaTransactionManager ，它需要javax.transaction.TransactionManager对其可用（这在标准 Java EE 中是特定于服务器的）。
- See Also:
  org.springframework.transaction.jta.JtaTransactionManager.setTransactionManager
- 场景：比如现在有一段业务代码，方法 A 调用方法 B，我希望的是如果方法 A 出错了，此时仅仅回滚方法 A，不能回滚方法 B，这个时候可以给方法 B 使用 REQUIRES_NEW 传播机制，让他们两的事务是不同的。

**NOT_SUPPORTED**

- 以非事务方式执行，如果存在则暂停当前事务。
- 注意：实际的事务暂停不会在所有事务管理器上开箱即用。 这尤其适用于org.springframework.transaction.jta.JtaTransactionManager ，它需要javax.transaction.TransactionManager对其可用（这在标准 Java EE 中是特定于服务器的）。
- See Also:
  org.springframework.transaction.jta.JtaTransactionManager.setTransactionManager

**NEVER**

- 以非事务方式执行，如果存在事务则抛出异常。

**<font color='FF0000'>NESTED</font>**

- 如果当前事务存在，则在一个嵌套事务中执行，否则行为类似于REQUIRED 。
  - 嵌套事务：是指外层的事务如果回滚，会导致内层的事务也回滚；但是内层的事务如果回滚，仅仅是滚回自己的代码。
- 注意：嵌套事务的实际创建仅适用于特定的事务管理器。 开箱即用，这仅适用于 JDBC DataSourceTransactionManager。 一些 JTA 提供者也可能支持嵌套事务。
- See Also:
  org.springframework.jdbc.datasource.DataSourceTransactionManager
- 场景：如果方法 A 调用方法 B，如果出错，方法 B 只能回滚它自己，方法 A 可以带着方法 B 一起回滚。那这种情况可以给方法 B 加上 NESTED 嵌套事务。

> Y：也许主要使用REQUIRED与NESTED。



除了事务的传播行为外，事务的其他特性Spring是借助底层资源的功能来完成的，Spring无非只充当个代理的角色。但是事务的传播行为却是Spring凭借自身的框架提供的功能。

**在同一个类中，一个方法调用另外一个有注解（比如@Async，@Transational）的方法，注解是不会生效的**

代码示例:

```java
/**
 * 例子中，有两方法，一个有@Transational注解，一个没有。如果调用了有注解的addPerson()方法，会启动一个Transaction；如果调用 
 * updatePersonByPhoneNo()，因为它内部调用了有注解的addPerson()，如果你以为系统也会为它启动一个Transaction，那就错了，实际上是没有的。
 **/
@Service
public class PersonServiceImpl implements PersonService {
 
 @Autowired
 PersonDao personDao;
 
 @Override
 @Transactional
 public boolean addPerson(Person person) {
  boolean result = personDao.insertPerson(person)>0 ? true : false;
  return result;
 }
 
 @Override
 //@Transactional
 public boolean updatePersonByPhoneNo(Person person) {
  boolean result = personDao.updatePersonByPhoneNo(person)>0 ? true : false;
  addPerson(person); //测试同一个类中@Transactional是否起作用
  return result;
 }
}
```

**原因：**

Spring 在扫描bean的时候会扫描方法上是否包含@Transactional注解，如果包含，spring会为这个bean动态地生成一个子类（即代理类，proxy），代理类是继承原来那个bean的。此时，当这个有注解的方法被调用的时候，实际上是由代理类来调用的，**代理类在调用之前就会启动transaction**。然而，**如果这个有注解的方法是被同一个类中的其他方法调用的，那么该方法的调用并没有通过代理类，而是直接通过原来的那个bean，所以就不会启动transaction**，我们看到的现象就是@Transactional注解无效。

**为什么一个方法a()调用同一个类中另外一个方法b()的时候，b()不是通过代理类来调用的呢？**可以看下面的例子（为了简化，用伪代码表示）：

```java
@Service
class A{
    @Transactinal
    method b(){...}
    
    method a(){    //标记1
        b();
    }
}
 
//Spring扫描注解后，创建了另外一个代理类，并为有注解的方法插入一个startTransaction()方法：
class proxy$A{
    A objectA = new A();
    method b(){    //标记2
        startTransaction();
        objectA.b();
    }
 
    method a(){    //标记3
        objectA.a();    //由于a()没有注解，所以不会启动transaction，而是直接调用A的实例的a()方法
    }
}
```

**解决的方法（两种）：**

1. 把这两个方法分开到不同的类中；
2. 把注解加到类名上面。

**结论:**

在一个Service内部，事务方法之间的嵌套调用，普通方法和事务方法之间的嵌套调用，都不会开启新的事务。

1. Spring采用动态代理机制来实现事务控制，而动态代理最终都是要调用原始对象的，而原始对象在去调用方法时，是不会再触发代理了！

2. Spring的事务管理是通过AOP实现的，其AOP的实现对于非final类是通过cglib这种方式，即生成当前类的一个子类作为代理类，然后在调用其下的方法时，会判断这个方法有没有@Transactional注解，如果有的话，则通过动态代理实现事务管理(拦截方法调用，执行事务等切面)。当b()中调用a()时，发现b()上并没有@Transactional注解，所以整个AOP代理过程(事务管理)不会发生。

**使用AOP 代理后的方法调用执行流程：**

![img](https://gitee.com/qmlg/image-bed/raw/master/images/%E4%BD%BF%E7%94%A8AOP%20%E4%BB%A3%E7%90%86%E5%90%8E%E7%9A%84%E6%96%B9%E6%B3%95%E8%B0%83%E7%94%A8%E6%89%A7%E8%A1%8C%E6%B5%81%E7%A8%8B.png)



## 1.2 理解Spring框架事务抽象

Spring事务抽象的关键是事务策略的概念。事务策略是由一个TransactionManager定义的，特别是

- 用于Servlet事务管理的org.springframework.transaction.PlatformTransactionManager接口
- 用于响应式事务管理的org.springframework.transaction.ReactiveTransactionManager接口

下面的清单显示了PlatformTransactionManager API的定义:

```java
public interface PlatformTransactionManager extends TransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

```java
public interface ReactiveTransactionManager extends TransactionManager {

    Mono<ReactiveTransaction> getReactiveTransaction(TransactionDefinition definition) throws TransactionException;

    Mono<Void> commit(ReactiveTransaction status) throws TransactionException;

    Mono<Void> rollback(ReactiveTransaction status) throws TransactionException;
}
```

## 1.3 声明式事务管理

Spring框架的TransactionInterceptor为命令式和响应式编程模型提供了事务管理。拦截器通过检查方法返回类型来检测所需的事务管理风格。返回响应类型(如Publisher或Kotlin Flow)的方法(或它们的子类型)符合响应事务管理的条件。包括void在内的所有其他返回类型都使用命令事务管理的代码路径。

事务管理风格会影响需要哪个事务管理器。命条式事务需要一个PlatformTransactionManager，而响应式事务使用ReactiveTransactionManager实现。

> @Transactional通常使用PlatformTransactionManager管理的线程绑定事务，将事务暴露给当前执行线程中的所有数据访问操作。注意:这不会传播到方法中新启动的线程。
>
> `ReactiveTransactionManager`管理的反应式事务使用Reactor上下文而不是线程本地属性。因此，所有参与的数据访问操作都需要在同一响应管道中的同一Reactor上下文中执行。

下图展示了在事务代理上调用方法的概念视图:

![tx](https://gitee.com/qmlg/image-bed/raw/master/images/tx.png)



## 1.4 事务传播

在Spring管理的事务中，注意物理事务和逻辑事务之间的差异，以及传播设置如何适用于此差异。

##### 理解`PROPAGATION_REQUIRED`

![需要tx道具](https://gitee.com/qmlg/image-bed/raw/master/images/tx_prop_required.png)

`PROPAGATION_REQUIRED`执行物理事务，如果不存在事务，则在本地为当前范围执行，或参与为更大范围定义的现有“外”事务。这是同一线程中常见调用堆栈安排中的良好默认值（例如，将服务立面委托给多个存储库方法，所有基础资源都必须参与服务级事务）。

##### 理解`PROPAGATION_REQUIRES_NEW`

![tx道具需要新的](https://gitee.com/qmlg/image-bed/raw/master/images/tx_prop_requires_new.png)

# 2. @Transactional的其它属性

**isolation 属性**

事务的隔离级别，默认值为 Isolation.DEFAULT。可选的值有：

- Isolation.DEFAULT：使用底层数据库默认的隔离级别
- Isolation.READ_UNCOMMITTED：读取未提交数据（会出现脏读，不可重复读）基本不使用
- Isolation.READ_COMMITTED：读取已提交数据（会出现不可重复读和幻读）
- Isolation.REPEATABLE_READ：可重复读（会出现幻读）
- Isolation.SERIALIZABLE：串行化

**timeout 属性**

事务的超时时间，默认值为 -1。如果超过该时间限制但事务还没有完成，则自动回滚事务。

**readOnly 属性**

指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true。

**rollbackFor 属性**

用于指定能够触发事务回滚的异常类型，可以指定多个异常类型。

**noRollbackFor 属性**

抛出指定的异常类型，不回滚事务，也可以指定多个异常类型。

------

> 内容来源：
>
> - [Spring 的事务实现原理和传播机制](https://zhuanlan.zhihu.com/p/157646604)
> - [Spring框架文档](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#spring-data-tier)
> - [Spring事务管理嵌套事务详解 : 同一个类中，一个方法调用另外一个有事务的方法](https://blog.csdn.net/levae1024/article/details/82998386)

