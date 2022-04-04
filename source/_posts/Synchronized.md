---
title: Synchronized
date: 2022-03-21 15:06:25
tags: Java、subject
categories: in-use
---

表格 表示**字段**访问权限和属性的各个标志（来自Java虚拟机规范4.5）(ACC 简写自 access_flags)

| 标志名        | 值   | 说明                                                         |
| ------------- | ---- | ------------------------------------------------------------ |
| ACC_PUBLIC    |      | 声明为public，可以从包外访问                                 |
| ACC_PRIVATE   |      | 声明为private，只能在定义该字段的类中访问                    |
| ACC_PROTECTED |      | 声明为protected，子类可以访问                                |
| ACC_STATIC    |      | 声明为static                                                 |
| ACC_FINAL     |      | 声明为final，对象构造好之后，就不能直接设置该字段了          |
| ACC_VOLATILE  |      | 声明为volatile，被标识的字段无法缓存                         |
| ACC_TRANSIENT |      | 声明为transient，被标识的字段不会为持久化对象管理器所写入或读取 |
| ACC_SYNTHETIC |      | 被表示的字段由编译器产生，而没有写源代码中                   |
| ACC_ENUM      |      | 该字段声明为某个枚举类型的成员                               |

表格 表示**方法**访问权限及属性的各个标志（来自Java虚拟机规范4.6）(ACC 简写自 access_flags)

| 标志名           | 值   | 说明                                                         |
| ---------------- | ---- | ------------------------------------------------------------ |
| ACC_PUBLIC       |      | 声明为public，可以从包外访问                                 |
| ACC_PRIVATE      |      | 声明为private，只能在定义该方法的类中访问                    |
| ACC_PROTECTED    |      | 声明为protected，子类可以访问                                |
| ACC_STATIC       |      | 声明为static                                                 |
| ACC_FINAL        |      | 声明为final，不能被覆盖                                      |
| ACC_SYNCHRONIZED |      | <font color='00FF00'>声明为synchronized，对该方法的调用，将包装在同步锁（monitor）里</font> |
| ACC_BRIDGE       |      | 声明为bridge方法，由编译器产生                               |
| ACC_VARARGS      |      | 表示方法带有变长参数                                         |
| ACC_NATIVE       |      | 声明为native，该方法不是用Java语言实现的                     |
| ACC_ABSTRACT     |      | 声明为abstract，该方法没有实现代码                           |
| ACC_STRICT       |      | 声明为strictfp，使用FP-strict浮点模式                        |
| ACC_SYNTHETIC    |      | 该方法是由编译器合成的，而不是由源代码编译出来的             |

参见SynchronizedDemo的字节码，同步方法add由一个ACC_SYNCHRONIZED flag。在其调用方main方法中调用add的前后几句指令如下，先通过monitorenter获取同步锁，调用完成同通过monitorexit释放同步锁。（此处静态方法add的监视器对象为SynchronizedDemo的类对象）

```java
4: monitorenter
5: getstatic     #7                  // Field java/lang/System.out:Ljava/io/PrintStream;
8: iconst_1
9: invokestatic  #23                 // Method add:(I)I
12: invokevirtual #27                 // Method java/io/PrintStream.println:(I)V
15: aload_1
16: monitorexit
```

**关于静态同步方法与非静态同步方法的调用：**

- 不同线程 调用 同一个类的静态同步方法 互斥；
- 不同线程 调用 同一个对象的非静态同步方法 互斥；
- 不同线程 调用 同一个类的不同对象的非静态同步方法 不互斥。

**对象头中同步锁相关内容：**

- 对象在堆内存中的存储布局：对象头 + 实例数据 + 对齐填充

- 对象头=Mark Word + Klass Word + （数组长度）？

  > Mark Word：存储对象自身运行时数据
  >
  > Klass Word：类型指针，指向它的类型元数据。Java虚拟机通过该指针确定该对象是哪个类的实例。

- 对象同步锁信息位于Mark Word中。

- hotspot虚拟机对synchronized所做的锁优化：自旋锁与自适应自旋、轻量级锁、偏向锁、锁消除、锁粗化。

  > 这些优化应该只是针对synchronized的，对于并发框架中的Lock并未使用。

- 针对锁优化，同步锁状态分成了四种状态：无锁、偏向锁、轻量级锁、重量级锁。

> 参考《深入理解Java虚拟机》
>
> <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/HotSpot%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%AF%B9%E8%B1%A1%E5%A4%B4Mark%20Word.png" alt="HotSpot虚拟机对象头Mark Word" style="zoom: 50%;" />
>
> 参考：[Java对象头与偏向锁、轻量级锁](https://www.likecs.com/show-203562639.html)

重量级锁

- 通过操作系统互斥量来实现。Linux中提供一把互斥锁mutex（也称之为互斥量）。

  > 参考：[互斥锁mutex](https://blog.csdn.net/qq_39736982/article/details/82348672)
  >
  > 主要应用函数：
  >
  > - pthread_mutex_init()函数          功能：初始化一个互斥锁
  >
  > - pthread_mutex_destroy()函数   功能：销毁一个互斥锁
  >
  > - pthread_mutex_lock()函数        功能：加锁
  >
  > - pthread_mutex_trylock()函数    功能：尝试加锁
  >
  > - pthread_mutex_unlock()函数    功能：解锁

轻量级锁

- 目的：初衷是**在没有多线程竞争的前提下**，减少传统的重量级锁使用操作系统互斥量产生的性能消耗。

  > 互斥同步面临的主要问题是进行线程阻塞和唤醒所带来的性能开销，因此这种同步也被称为阻塞同步（Blocking Synchronization）。

偏向锁

- 目的：消除数据在**无竞争情况下**的同步原语，进一步提高程序的运行性能。

  > 即避免了轻量级锁中Mark Word 的 CAS拷贝等操作的消耗。

- 其他注意点：

  - 当一个对象已经计算过一致性哈希码后，它就再也无法进入偏向锁状态了；而当一个对象当前正处于偏向锁状态，又收到需要计算其一致性哈希码请求[1]时，它的偏向状态会被立即撤销，并且锁会膨胀为重量级锁。在重量级锁的实现中，对象头指向了重量级锁的位置，代表重量级锁的ObjectMonitor类里有字段可以记录非加锁状态（标志位为“01”）下的Mark Word，其中自然可以存储原来的哈希码。

    > 注意，[1]这里说的计算请求应来自于对Object::hashCode()或者System::identityHashCode(Object)方法的调用，如果重写了对象的hashCode()方法，计算哈希码时并不会产生这里所说的请求。

![偏向锁与轻量级锁过程](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/%E5%81%8F%E5%90%91%E9%94%81-%E8%BD%BB%E9%87%8F%E7%BA%A7%E9%94%81%E8%BF%87%E7%A8%8B.jpg)

偏向锁还有写疑问，参考：

- https://www.zhihu.com/question/55075763

- https://www.cnblogs.com/cqqfboy/p/14657900.html

- https://www.itqiankun.com/article/bias-lightweight-synchronized-lock

  > 文中的一幅图：
  >
  > ![synchronized原理_from_itqiankun](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images/synchronized%E5%8E%9F%E7%90%86_from_itqiankun.jpeg)

