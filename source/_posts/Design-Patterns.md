---
title: Design Patterns
date: 2021-09-03 20:08:41
tags: 设计模式
---

# 1. 设计模式六大原则

![img](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/design_principle.png)

1.1 单一职责原则

There should never be more than one reason for a class to change.

理解：不同的类具备不同的职责，各司其职。做系统设计是，如果发现有一个类拥有了两种职责，那么就要问一个问题：可以将这个类分成两个类吗？如果真的有必要，那就分开，千万不要让一个类干的事情太多。

总结：一个类只承担一个职责

1.2 开放封闭原则

Software entities like classes,modules and functions should be open for extension but closed for modifications.

理解：类、模块、函数，可以去扩展，但不要去修改。如果要修改代码，尽量用继承或组合的方式来扩展类的功能，而不是直接修改类的代码。当然，如果能保证对整个架构不会产生任何影响，那就没必要搞的那么复杂，直接改这个类吧。

总结：对软件实体的改动，最好用扩展而非修改的方式。

1.3 里式替换原则

Functions that use pointers or references to base classes must be able to use objects of derived classes without knowing it.

理解：父类可被子类替换，但反之不一定成立。也就是说，代码中可以将父类全部替换为子类，程序不会出现异常，但反过来就不一定了。

总结：在继承类是，务必重写（override）父类中所有的方法，尤其需要注意父类的protected方法（它们往往是让你重写的），子类尽量不要暴露自己的public方法供外界调用。

1.4 最少知识原则

Only talk to you immediate friends.

理解：尽量减少对象之间的交互，从而减小类之间的耦合。在做系统设计时，不要让一个类依赖于太多其他的类，需尽量减小依赖关系，否则死都不知道怎么死的。

总结：一定要做到：低耦合、高内聚。

1.5 接口隔离原则

The dependency of one class to another one should depend on the smallest possible interface.

理解：不要对外暴露没有实际意义的接口。也就是说，尽量保证接口的实用性。当需要对外暴露接口时，需要再三斟酌，若没必要对外提供就删了吧，因为一旦提供了就意味着，将来要多做一件事情，何苦给自己找事做呢。

总结：不要对外暴露没有实际意义的接口。

1.6 依赖倒置原则

High level modules should not depends upon low level modules.Both should depend upon abstractions.Abstractions should not depend upon details.Details should depend upon abstractions.

理解：高层模块不应该依赖于底层模块，而应该依赖于抽象。抽象不应依赖于细节，细节应依赖于抽象。应该面向接口编程，不该面向实现类编程。面向实现类编程相当于就事论事，那是正向依赖；面向接口编程，相当于透过现象看本质，抓住事务的共性，那就是反向依赖，即依赖倒置。

总结：面向接口编程，提取出事务的本质和共性。

1.7 合成复用原则

合成复用原则（Composite Reuse Principle，CRP）又叫组合/聚合复用原则（Composition/Aggregate Reuse Principle，CARP）。它要求在软件复用时，要尽量先使用组合或者聚合等关联关系来实现，其次才考虑使用继承关系来实现。

# 2. 分类

- **创建型**

  创建型模式抽象了实例化过程。它们帮助一个系统独立于如何创建、组合和表示它的那些对象。

  一个类创建型模式使用继承改变被实例化的类；而一个对象创建型模式将实例化委托给另一个对象。

- **结构型**

  结构型模式涉及如何组合类和对象以获得更大的结构。

  结构型类模式采用继承机制来组合接口或实现。

  结构型对象模式不是对接口和实现进行组合，而是描述了如何对一些对象进行组合，从而实现新功能的一些方法。因为可以在运行时改变对象组合关系，所以对象组合方式具有更大的灵活性，而这种机制用静态类组合是不可能实现的。

- **行为型**

  行为型模式涉及算法和对象间职责的分配。行为型模式不仅描述对象或类的模式，还描述它们之间的通信模式。这些模式刻画了在运行时难以跟踪的复杂的控制流。它们将你的注意力从控制流转移到对象间的联系方式上来。
  
  类行为型模式使用继承机制在类间分派行为。
  
  对象行为型模式使用对象组合而不是继承。

**三者之间的联系：**

1. 创建型模式为其他两种模式使用提供了环境，好比VS软件提供了.net环境和操作平台，是各种编程语言能随心所欲地在这个平台上编译执行；
2. 结构型模式侧重于接口的使用，它做的一切工作都是对象或是类之间的交互，提供一个门，成就一个你来我往，协同合作的地球村；
3. 行为型模式顾名思义，侧重于具体行为，所以概念中才会出现职责分配和算法通信等内容。

将三者结合起来成为故事，中美合作的故事——创建型模式提供国际环境，无战争，求发展；结构型模式为中美合作提供理由，即和平时代的互利共赢，行为型模式就具体到两个大国之间是如何合作，比如经济合作、文化合作等。

- 创建型（5种）
  - Factory Method（工厂方法）
  - Abstract Factory（抽象工厂）
  - Builder（生成器）
  - Prototype（原型）
  - Singleton（单例）

- 结构型（7种）
  - Adapter Class/Object（适配器）
  - Bridge（桥接）
  - Composite（组合）
  - Decorator（装饰）
  - Facade（外观）
  - Flyweight（享元）
  - Proxy（代理）

- 行为型（11种）
  - Interpreter（解释器）
  - Template Method（模板方法）
  - Chain of Responsibility（责任链）
  - Command（命令）
  - Iterator（迭代器）
  - Mediator（中介者）
  - Memento（备忘录）
  - Observer（观察者）
  - State（状态）
  - Strategy（策略）
  - Visitor（访问者）

## 2.2 GOF 结构图

### 2.2.1 创建型（5种）

1. Factory Method（工厂方法）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Factory%20Method%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Factory Method 模式结构图" style="zoom: 50%;" />

2. Abstract Factory（抽象工厂）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Abstract%20Factory%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Abstract Factory模式结构图" style="zoom:50%;" />

3. Builder（生成器）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Builder%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Builder 模式结构图" style="zoom:50%;" />

4. Prototype（原型）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Prototype%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Prototype 模式结构图" style="zoom: 50%;" />

5. Singleton（单例）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Singleton%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Singleton 模式结构图" style="zoom: 50%;" />

### 2.2.2 结构型（7种）

1. Adapter Class/Object（适配器）

  > 类适配器使用多重继承对一个接口与另一个接口进行匹配：
  >
  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Adaptor%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Adaptor 模式结构图" style="zoom:50%;" />
  >
  > 对象匹配器依赖于对象组合：
  >
  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Adaptor%20%E5%AF%B9%E8%B1%A1%E9%80%82%E9%85%8D%E5%99%A8%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Adaptor 对象适配器模式结构图" style="zoom:50%;" />

2. Bridge（桥接）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Bridge%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Bridge 模式结构图" style="zoom:50%;" />

3. Composite（组合）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Composite%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Composite 模式结构图" style="zoom:50%;" />
  >
  > 典型的Composite对象结构图如下：
  >
  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/%E5%85%B8%E5%9E%8B%20Composite%20%E5%AF%B9%E8%B1%A1%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="典型 Composite 对象结构图" style="zoom: 33%;" />

4. Decorator（装饰）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Decorator%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Decorator 模式结构图" style="zoom:50%;" />

5. Facade（外观）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Facade%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Facade 模式结构图" style="zoom: 50%;" />

6. Flyweight（享元）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Flyweight%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Flyweight 模式结构图" style="zoom:50%;" />
  >
  > 下面的对象图说明了如何共享flyweight：
  >
  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/%E5%85%B1%E4%BA%ABflyweight%20%E5%AF%B9%E8%B1%A1%E5%9B%BE.png" alt="共享flyweight 对象图" style="zoom:50%;" />

7. Proxy（代理）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Proxy%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Proxy模式结构图" style="zoom:50%;" />
  >
  > Y：这种应该是基于接口的代理；还可以基于继承实现。

### 2.2.3 行为型（11种）

1. Interpreter（解释器）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Interpreter%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Interpreter 模式结构图" style="zoom: 50%;" />

2. Template Method（模板方法）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Template%20Method%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Template Method 模式结构图" style="zoom: 50%;" />

3. Chain of Responsibility（责任链）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Chain%20of%20Responsibility%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Chain of Responsibility 模式结构图" style="zoom:50%;" />

4. Command（命令）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Command%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Command 模式结构图" style="zoom:50%;" />

5. Iterator（迭代器）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Iterator%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Iterator 模式结构图" style="zoom:50%;" />

6. Mediator（中介者）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Mediator%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Mediator 模式结构图" style="zoom:50%;" />
  >
  > 典型对象结构：
  >
  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Mediator%20%E5%85%B8%E5%9E%8B%E5%AF%B9%E8%B1%A1%E7%BB%93%E6%9E%84.png" alt="Mediator 典型对象结构" style="zoom: 50%;" />

7. Memento（备忘录）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Memento%20%E7%BB%93%E6%9E%84%E6%A8%A1%E5%BC%8F%E5%9B%BE.png" alt="Memento 结构模式图" style="zoom:50%;" />

8. Observer（观察者）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Observer%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Observer 模式结构图" style="zoom:50%;" />

9. State（状态）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/State%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="State 模式结构图" style="zoom: 50%;" />

10. Strategy（策略）

  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Strategy%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Strategy 模式结构图" style="zoom:50%;" />

11. Visitor（访问者）

   > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/Visitor%20%E6%A8%A1%E5%BC%8F%E7%BB%93%E6%9E%84%E5%9B%BE.png" alt="Visitor 模式结构图"/>



## 2.3 Some Think

> Y：
>
> 创建型：
>
> - 单例侧重于共享行为；
>
> - 原型模式侧重以内存复制的方式替代对象创建过程，提高效率；
>
> - 生成器模式侧重对一个复杂的包含多个组成部分或环节的创建过程进行定制化拼装；
>
> - 工厂方法模式侧重封装对象的实例化过程和屏蔽实例化的子类；
>
> - 抽象工厂侧重一系列对象的整体变更，如产品线或主题变更。
>
> 结构型：
>
> 行为型：

**AbstractFactory**

> Y：强调灵活变更一系列产品的创建，即灵活而统一的变更一条产品线。在Spring中通过@Configuration和@Condition注解来实现的产品线切换（如AspectJ代理、webFlux方式等）是否可以看作AbstractFactory的一种实现形式  
>
> GOF：一个具体的工厂通常是一个Singleton。

**FactoryMethod**

> Y：Creator 的抽象意在支持同一产品的多种创建方式的灵活替换；ConcreateCreator来表示该产品具体的某种创建方式，如通过某个Prototype实例的复制，或者通过Redis数据反序列化而来等。

**Builder（生成器）**

> Y：Builder提供复杂对象X各组成部分的创建接口，其子类负责X各组成部分的具体创建的实现。Director即使用方，基于Builder提供的X各组成部分的创建接口灵活拼装X的组成部分及顺序。
>
> Y：若把该复杂产品看作同一系列产品，那么Builder及其实现与AbstractFactory模式中的AbstractFactory及其实现相似。而X每个部分的创建也都是基于FactoryMethod。
>
> GOF：Builder在最后一步返回产品，而对于Abstract Factory来说，产品是立即返回的。

**Singleton**

**Prototype**

> Y：注意是通过原型实例拷贝自身来创建新的对象。所以说原型实例是Singleton。多个原型实例可作为一个对象组，替代Abstract Factory完成一系列产品的创建。

> Y：引申深拷贝、浅拷贝？
>
> - 浅拷贝：仅对象本身及其值类型的成员，引用类型的成员拷贝的地址，指向原成员实例。
> - 深拷贝：对向本身及其所有成员全部拷贝，成员也是深拷贝。引用型成员指向新拷贝的实例。
>
> 通过实现Cloneable接口，调用super.clone()方法实现某对象的拷贝。最终调用的Object的clone()方法，native方法。

**创建型模式讨论**

> GOF：通常设计以使用Factory Method开始，并且当设计者发现需要更大的灵活性时，设计便会向其它创建型模式演化。

**Adapter**

> 又名包装器（Wrapper）。通过包装，使一个原本不兼容的接口可以一同工作。
>
> Y：类似转接头？强调利用已有接口或对象。

**Bridge**

> 又名Handle/Body。
>
> Y：分离接口与实现部分。通过对底层差异化实现的封装，对外提供统一而易于使用的接口。就好像Java native方法屏蔽了平台的差异，对外提供了统一的接口。

**Composite**

> Y：从clone方法角度看，Cloneable接口即可看作Component，它的实现类则可能为Leaf或Composite。TreeNode也应该算是。
>
> GOF：几乎在所有面向对象系统中都有Composite模式的应用实例。

**Decorator**

> 又名包装器（wrapper）。
>
> Y：通过包装，添加功能而不改变接口。
>
> GOF：
>
> - Composite（4.3）：可以将装饰视为一个退化的、仅有一个组件的组合。然而，装饰仅给对象添加一些额外的职责——它的目的不在于对象聚集。

**Facade**   [fəˈsɑːd]

> Y：Gateway 接口路由应该是吧？可能不是，但算不算是中介者模式呢？

**Flyweight**

> Y：注意内部状态、外部状态的概念。UnsharedConcreateFlyweight对象由Client创建。
>
> GOF：
>
> -  Flyweight模式通常和Composite（4.3）模式结合起来，用共享叶结点的有向无环图实现一个逻辑上的层次结构。 
> - 通常，最好用flyweight实现State（5.8）和Strategy（5.9）对象。

**Proxy**

> Y：基于Fegin的远程接口调用，就是通过一个代理来实现远程接口的访问。
>
> GOF：
>
> - Decorator（4.4）：尽管装饰的实现部分与代理相似，但装饰的目的不一样。装饰为对象添加一个或多个功能，而代理则控制对对象的访问。

**结构型模式讨论**

**Chain of Responsibility**

> GOF: 职责链常与Composite（4.3）一起使用。这种情况下，一个构件的父构件可作为它的后继。
>
> Y：这样的话，Spring BeanFactory中查询给定类的BeanDefinition时，也是先使用当前工厂，查不到会向上递归使用父工厂查询，那这也是Composite与责任链模式结合使用了？
>
> Y：定义一个list chains存储所有handler，遍历处理request，算是责任链模式吗？应该是的。

**Command**

> Y：领域驱动设计中会使用到。

**Interpreter**

> GOF：一般只有在用一个类层次来定义某个语言时，才强调使用解释器模式。

**Iterator**

> GOF：提供一种方法顺序访问一个聚合对象中的各个元素，而又不需要暴露该对象的内部表示。
>
> Y：迭代器类ConcreateIterator应该是作为聚合类ConcreateAggregate的内部类，以保证ConcreateAggregate不需要暴露其内部表示，而ConcreateIterator又可以访问ConcreateAggregate的内部结构以完成遍历。

**Mediator**

**Memento**

> Y：常与Command一起使用，Command用于回退，而Memento则用来备份Command执行前的状态。
>
> GOF：
>
> - 1）语言支持　备忘录有两个接口：一个为原发器所使用的宽接口，一个为其他对象所使用的窄接口。理想的实现语言应可支持两级的静态保护。
>
> Y：Java中可以使用内部类来实现，Memento作为Originator的内部类。

**Observer**

**State**

**Strategy**

**Template Method**

**Visitor**

> Y：Lambada表达式，如map()。

# 3. 源码中的应用

## 3.1 创建型

## 3.2 结构型

### 3.2.5 Facade

- tomcat-embed-core：ApplicationContextFacade

  > ApplicationContextFacade对象即为一个Facade对象，它从web应用程序中屏蔽了tomcat-embed-core内部的ApplicationContext对象。
  >
  > ```java
  > private final ApplicationContext context;
  > ```
  >
  >  

### 3.2.6 Flyweight

- **JDK：Integer.IntegerCache**

  > ```java
  > // 存储了 -128 and 127 范围的Integer包装类实例。在类加载时初始化。
  > static final Integer cache[];
  > ```







------

> 内容来源：
>
> - [GOF23种设计模式精解](https://www.cnblogs.com/lqmblog/p/8549833.html)
> - [创建型、行为型、结构型有什么区别和联系？](https://blog.csdn.net/zhanduo0118/article/details/85602983)
> - 《设计模式 可复用面向对象软件的基础》

