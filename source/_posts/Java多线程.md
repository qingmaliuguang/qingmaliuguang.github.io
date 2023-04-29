---
title: Java多线程
date: 2022-03-21 15:06:25
tags: Java、subject
categories: in-use
---

- 处理器缓存

- 内存屏障

- Java多线程机制

  > - 三个关注点：原子性、可见性、有序性
  >
  >   > 可见性即内存可见性；有序性即禁止重排序
  >
  > - 硬件
  >
  >   > 硬件保证某些从语义上看起来需要多次操作的行为可以只通过一条处理器指令就能完成，这类指令常用的有： 
  >   >
  >   > - 测试并设置（Test-and-Set）； 
  >   > - 获取并增加（Fetch-and-Increment）； 
  >   > - 交换（Swap）； 
  >   > - 比较并交换（Compare-and-Swap，下文称CAS）； 
  >   > - 加载链接/条件储存（Load-Linked/Store-Conditional，下文称LL/SC）。
  >   >
  >   > => Java中最终暴露出来的是CAS指令。
  >
  > - class 文件层面
  >
  >   > - synchronized同步块：monitorenter & monitorexit
  >   >
  >   > - synchronized方法：ACC_SYNCHRONIZED
  >   
  > - JDK 并发包层面
  >
  >   > - Unsafe.park & Unsafe.unpark基于mutex互斥量实现的线程挂起。可参考：[jdk1.8 Unsafe类 park和unpark方法解析](https://blog.csdn.net/a7980718/article/details/83661613)
  >   >
  >   > - Unsafe中的三个内存屏障：loadFence() & storeFence() & fullFence()
  >   > - Unsafe中的CAS指令
  >   >
  >   > - happen-before规则
  >
  > - happen-before 规则总结
  >
  >   > 1. 单线程中的每个操作，happen-before于该线程中任意后续操作。 
  >   > 2. 对volatile变量的写，happen-before于后续对这个变量的读。 
  >   > 3. 对synchronized的解锁，happen-before于后续对这个锁的加锁。 
  >   > 4. 对final变量的写，happen-before于final域对象的读，happen-before于后续对final变量的读。 
  >   >
  >   > 四个基本规则再加上happen-before的传递性，就构成JMM对开发者的整个承诺。在这个承诺以外的部分，程序都可能被重排序，都需要开发者小心地处理内存可见性问题。
  >   >
  >   > Java语言规范17.4.5中描述如下：
  >   >
  >   > - 在监视器上的解锁动作在每个后续在该监视器上的锁定操作之前发生。
  >   > - 对volatile 域（第8.3.1.4节）的写操作在每个后续对该域的读操作之前发生。
  >   > - 在线程上对 start() 的调用在被启动线程的所有动作之前发生。
  >   > - 一个线程中的所有动作在任何其他线程成功地从该线程上的 join() 返回之前发生。
  >   > - 任何对象的缺省初始化在程序中其他任何动作（除了缺省的写操作）之前发生。
  >
  > - JDK8开始，Unsafe中提供的三个内存屏障
  >
  >   - loadFence()
  >   - storeFence()
  >   - fullFence()
  >
  >   > 在理论层面，可以把基本的CPU内存屏障分成四种： 
  >   >
  >   > （1）LoadLoad：禁止读和读的重排序。 
  >   >
  >   > （2）StoreStore：禁止写和写的重排序。 
  >   >
  >   > （3）LoadStore：禁止读和写的重排序。 
  >   >
  >   > （4）StoreLoad：禁止写和读的重排序。
  >   >
  >   > 根据注释：
  >   >
  >   > - loadFence=LoadLoad+LoadStore ，即禁止读-读、读-写重排序
  >   >
  >   > - storeFence=StoreStore+LoadStore ，即禁止读-写、写-写重排序
  >   >
  >   > - fullFence=loadFence+storeFence+StoreLoad，即禁止所有重排序
  >
  > - 内存屏障 + happen-before 规则  =>  synchronized、volatile、final
  >
  >   - synchronized：原子性、可见性、有序性
  >
  >   - volatile：64位写入的原子性、可见性和有序性。
  >
  >   - final：有序性 => 需要避免构造函数溢出问题（示例见FinalReferenceEscapeExample，参考[final引用不能从构造函数内“溢出”](https://bbs.huaweicloud.com/blogs/136261)）。
  >
  >     > ![image.png](https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images-unnamed/1575361630931085.png)
  >
  > - 从底向上看volatile背后的原理：
  >
  > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images-unnamed/%E6%88%AA%E5%B1%8F%202021-10-29%20%E4%B8%8B%E5%8D%882.54.08.png" alt="截屏 2021-10-29 下午2.54.08" style="zoom: 33%;" />
  >
  > - CPU 的 CAS（Compare and Swap）指令 => unsafe中的CAS操作：CAS能够将read-modify-write和check-and-act之类的操作转换为原子操作。
  >
  >   > Y：CAS操作常作用于volatile变量，如此可同时保证对其操作的原子性、可见性、有序性。如Atomic类。

- Java部分多线程相关类的实现

  整个Concurrent包的层次体系：

  <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images-unnamed/%E6%88%AA%E5%B1%8F%202021-10-29%20%E4%B8%8B%E5%8D%882.59.24.png" alt="截屏 2021-10-29 下午2.59.24" style="zoom: 33%;"/>

  - Atomic类

    > volatile + CAS
    >
    > - Unsafe类中提供了int、long、Object三种类型的CAS操作，其它类型可转换成这三种类型，如boolean转成int，double转成long。带版本号的int和long通过与版本号构造Pair对象，转成Object。
    >
    > - ABA问题 => 同时使用“值”和“版本号”，即AtomicStampedReference。
    >
    >   > 目前来说AtomicStampedReference处于相当鸡肋的位置，大部分情况下ABA问题不会影响程序并发的正确性，如果需要解决ABA问题，改用传统的互斥同步可能会比原子类更为高效。
  
  - Lock 与 Condition
  
    > 为了实现一把具有阻塞或唤醒功能的锁，需要几个核心要素：
    >
    > - ① 需要一个state变量，标记该锁的状态。state变量至少有两个值：0、1。对state变量的操作，要确保线程安全，也就是会用到CAS。
    >
    >   > volatile + CAS
    >   >
    >   > state取值不仅可以是0、1，还可以大于1，就是为了支持锁的可重入性。
    >
    > - ② 需要记录当前是哪个线程持有锁。
    >
    >   > 普通成员变量即可。
    >
    > - ③ 需要底层支持对一个线程进行阻塞或唤醒操作。
    >
    >   > 在Unsafe类中，提供了阻塞或唤醒线程的一对操作原语，也就是park/unpark。
    >   >
    >   > LockSupport的工具类，对这一对原语做了简单封装。
    >
    > - ④ 需要有一个队列维护所有阻塞的线程。这个队列也必须是线程安全的无锁队列，也需要用到CAS。
    >
    >   > **AbstractQueuedSynchronizer：队列同步器（AQS）**
    >   >
    >   > 提供一个框架，用于实现依赖先进先出 (FIFO) 等待队列的阻塞锁和相关同步器（信号量、事件等）。 此类旨在成为大多数依赖单个原子int值来表示状态的同步器的有用基础。
    >   >
    >   > 要将此类用作同步器的基础，请根据适用情况重新定义以下方法，方法是使用getState 、 setState和/或compareAndSetState检查和/或修改同步状态：
    >   >
    >   > - tryAcquire
    >   > - tryRelease
    >   > - tryAcquireShared
    >   > - tryReleaseShared
    >   > - isHeldExclusively
    >   >
    >   > Y：模版方法模式。
    >   >
    >   > 基于 volatile + CAS + 双向链表 实现无锁化阻塞队列。
    >
    > Condition将Object监视方法(wait、notify和notifyAll)分解为不同的对象，通过使用任意的Lock实现将它们组合起来，从而实现每个对象拥有多个等待集的效果。**在Lock替换synchronized方法和语句的地方，Condition替换Object监视方法的使用。**
    >
    > Condition类注解中的一个示例：BoundedBuffer
    >
    > ```java
    > // 例如，假设我们有一个支持put和take方法的有界缓冲区。如果在一个空的缓冲区上尝试一个take，那么线程将会阻塞，直到一个项目变得可用;如果尝试在已满的缓冲区上执行put操作，则线程将阻塞，直到有可用空间为止。我们希望在单独的等待集中保持等待的put线程和取线程，这样当缓冲区中的项目或空间可用时，我们可以使用这种优化，即每次只通知单个线程。这可以通过使用两个Condition实例来实现。
    > class BoundedBuffer {
    >      final Lock lock = new ReentrantLock();
    >      final Condition notFull  = lock.newCondition(); 
    >      final Condition notEmpty = lock.newCondition(); 
    >   
    >      final Object[] items = new Object[100];
    >      int putptr, takeptr, count;//PTR 为 指针。
    >   
    >      public void put(Object x) throws InterruptedException {
    >        lock.lock();
    >        try {
    >          // 使用count 而非putptr，因为实现的环形缓冲区。
    >          while (count == items.length)
    >            notFull.await();
    >          items[putptr] = x;
    >          if (++putptr == items.length) putptr = 0;
    >          ++count;
    >          notEmpty.signal();
    >        } finally {
    >          lock.unlock();
    >        }
    >      }
    >   
    >      public Object take() throws InterruptedException {
    >        lock.lock();
    >        try {
    >          while (count == 0)
    >            notEmpty.await();
    >          Object x = items[takeptr];
    >          if (++takeptr == items.length) takeptr = 0;
    >          --count;
    >          notFull.signal();
    >          return x;
    >        } finally {
    >          lock.unlock();
    >        }
    >      }
    >    }
    > ```
    >
    > 
    >
    > **ReentrantLock**
    >
    > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images-unnamed/image-20211030000810955.png" alt="image-20211030000810955" style="zoom: 33%;" />
    >
    > - ReentrantLock.Sync采用的独占模式，其中实现了tryRelease和isHeldExclusively，tryAcquire交由子类实现，FairSync和NonfairSync分别通过该方法实现了公平锁和非公平锁。（非公平锁的tryAcquire逻辑使用了Sync的nonfairTryAcquire，因为ReentrantLock的tryLock方法会直接调用sync.nonfairTryAcquire即使用非公平锁逻辑）。
    >
    > **ReentrantReadWriteLock**
    >
    > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images-unnamed/image-20211030000856693.png" alt="image-20211030000856693" style="zoom: 50%;" />
    >
    > - acquire/release、acquireShared/releaseShared 是AQS里面的两对模板方法。互斥锁和读写锁的写锁都是基于acquire/release模板方法来实现的。读写锁的读锁是基于acquireShared/releaseShared这对模板方法来实现的。
    >
    > **StampedLock**
    >
    > - 乐观读
    >
    >   > - StampedLock引入了“乐观读”策略，读的时候不加读锁，读出来发现数据被修改了，再升级为“悲观读”，相当于降低了“读”的地位，把抢锁的天平往“写”的一方倾斜了一下，避免写线程被饿死。
    >   > - 使用一个volatile 变量 state，表示读锁、写锁、版本号，获取释放锁时对state进行CAS操作。
    >   > - 直接使用了loadFence避免validate()与前边的读操作进行重排序。
    >
    > - 悲观读写：自旋 + 阻塞
    >
    >   - 没有基于AQS，自己实现阻塞队列
    >
    >     > - 在AQS里面，当一个线程CAS state失败之后，会立即加入阻塞队列，并且进入阻塞状态。但在StampedLock中，CAS state失败之后，会不断自旋，自旋足够多的次数之后，如果还拿不到锁，才进入阻塞状态。为此，根据CPU的核数，定义了自旋次数的常量值。如果是单核的CPU，肯定不能自旋，在多核情况下，才采用自旋策略。
    >     >
    >     > - 另外一个不同于AQS的阻塞队列的地方是，在每个WNode里面有一个cowait指针，用于串联起所有的读线程。例如，队列尾部阻塞的是一个读线程 1，现在又来了读线程 2、3，那么会通过cowait指针，把1、2、3串联起来。1被唤醒之后，2、3也随之一起被唤醒，因为读和读之间不互斥。
    >
    > | 锁                     | 并发度                                       | 阻塞队列       |
    > | ---------------------- | -------------------------------------------- | -------------- |
    > | ReentrantLock          | 读与读互斥，读与写互斥，写与写互斥           | 基于AQS        |
    > | ReentrantReadWriteLock | 读与读不互斥，读与写互斥，写与写互斥         | 基于AQS        |
    > | StampedLock            | 读与读不互斥，读与写不一定不互斥，写与写互斥 | 自实现阻塞队列 |
    >
    > **Condition** 
    >
    > - AbstractQueuedSynchronizer.ConditionObject
    >
    >   > - wait（）/notify（）必须和synchronized一起使用，Condition也是如此，必须和Lock一起使用。
    >   >
    >   > - 其作用在于使线程阻塞并添加到阻塞队列中（await），唤醒阻塞队列中第一个或全部线程（signal、signalAll）,基于LockSupport。
    >   > - 在调用 await、signal、signalAll的时候，必须先拿到锁。
    >   > - 因其操作都在获取锁之后进行，所以不需要volatile、CAS等机制。 
  
  - 同步工具
  
    > **Semaphore**
    >
    > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images-unnamed/%E6%88%AA%E5%B1%8F%202021-10-31%20%E4%B8%8B%E5%8D%883.12.49.png" alt="截屏 2021-10-31 下午3.12.49" style="zoom:33%;" />
    >
    > **CountDownLatch**
    >
    > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images-unnamed/%E6%88%AA%E5%B1%8F%202021-10-31%20%E4%B8%8B%E5%8D%883.29.21.png" alt="截屏 2021-10-31 下午3.29.21" style="zoom:33%;" />
    >
    > - 因为是基于 AQS 阻塞队列来实现的，所以可以让多个线程都阻塞在state=0条件上，通过countDown()一直累减state，减到0后一次性唤醒所有线程。如图4-4所示，假设初始总数为M，N个线程await()，M个线程countDown()，减到0之后，N个线程被唤醒。
    >
    >   <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images-unnamed/%E6%88%AA%E5%B1%8F%202021-10-31%20%E4%B8%8B%E5%8D%883.35.29.png" alt="截屏 2021-10-31 下午3.35.29" style="zoom: 50%;" />
    >
    >   Y：
    >
    >   - CountDownLatch的构造参数设置的state的值，即M，调用await方法，只是判断state是否等于0，等于则返回相当于获取锁成功，不等则一直阻塞，不可被中断，因响应中断后会重复进入阻塞。进入阻塞的线程数N与M无关。
    >
    >   - 调用countDown，会修改state的值，仅当state修改后变为0时唤醒阻塞队列中的所有线程，countDown需调用M次。
    >
    >     > 最后一次countDown操作后触发doReleaseShared方法以唤醒队列中线程，阻塞的所有线程不是同时唤醒的，而是从头节点依次唤醒的，即被唤醒的线程继续执行唤醒操作doReleaseShared去唤醒下一个。
    >     >
    >     > 参考：https://www.jianshu.com/p/128476015902
    >
    > **CyclicBarrier**
    >
    > > - CyclicBarrier基于ReentrantLock + Condition实现。
    > > - CyclicBarrier是可以被重用的。
    > > - CyclicBarrier 会响应中断。
    > > - 构造参数中的回调函数barrierAction只会被最后一个线程执行1次（在唤醒其他所有线程之前），而不是所有线程每个都执行1次。
    > >
    > > Y：其与CountDownLatch的区别：
    > >
    > > - 下一步动作的动作实施者是不一样的，CountDownLatch中是await的所有线程，CyclicBarrier是最后一个线程。
    > > - 整个过程的参与者不同，CountDownLatch中有两种角色，await与countDown，两个可能完全无关。CyclicBarrier中只有一种。
    >
    > **Exchanger**
    >
    > > 用于线程间交换数据。
    > >
    > > Exchanger的核心机制和Lock一样，也是CAS+park/unpark。
    >
    > **Phaser**
    >
    > > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images-unnamed/%E6%88%AA%E5%B1%8F%202021-10-31%20%E4%B8%8B%E5%8D%885.05.41.png" alt="截屏 2021-10-31 下午5.05.41" style="zoom: 50%;" />
    > >
    > > 基于volatile state + CAS + 无锁栈 Treiber Stack（等待队列）。
    > >
    > > 主要方法：
    > >
    > > - arrive、awaitAdvance  =>  可替代CountDownLatch
    > > - arriveAndAwaitAdvance  =>  可替代CyclicBarrier
    > >
    > > 特性：
    > >
    > > - 动态调整线程个数；
    > >
    > > - 层次Phaser。
    > >
    > >   > 父Phaser并不用感知子Phaser的存在，当子Phaser中注册的参与者数量大于0时，会把自己向父节点注册；当子Phaser中注册的参与者数量等于0时，会自动向父节点解注册。父Phaser把子Phaser当作一个正常参与的线程就可以了。
  
  - 并发容器
  
    > **BlockingQueue**
    >
    > > 入队提供了add（..）、offer（..）、put （..）3个函数，有什么区别呢？从上面的定义可以看到，add（..）和offer（..）的返回值是布尔类型，而put无返回值，还会抛出中断异常，所以add（..）和offer（..）是无阻塞的，也是Queue本身定义的接口，而put（..）是阻塞式的。add（..）和offer（..）的区别不大，当队列为满的时候，前者会抛出异常，后者则直接返回false。
    > >
    > > 出队列与之类似，提供了remove（）、peek（）、take（）等函数，remove（）和peek（）是非阻塞式的，take（）是阻塞式的。
    >
    > - ArrayBlockingQueue 
    >
    >   > 是一个用数组实现的环形队列，核心为 1 把锁 + 2 个条件，即lock + notEmpty + notFull。
    >
    > - LinkedBlockingQueue 
    >
    >   > LinkedBlockingQueue是一种基于单向链表的阻塞队列。因为队头和队尾是2个指针分开操作的，所以用了2把锁+2个条件，同时有1个AtomicInteger的原子变量记录count数。
    >   >
    >   > 核心 为 2 把锁 + 2 个条件，即 takeLock + notEmpty + putLock + notFull。
    >
    > - PriorityBlockingQueue
    >
    >   > 队列通常是先进先出的，而PriorityQueue是按照元素的优先级从小到大出队列的。正因为如此，PriorityQueue中的2个元素之间需要可以比较大小，并实现Comparable接口。
    >   >
    >   > 核心为 1 把锁 + 1 个条件，即 lock + notEmpty。
    >
    > - DelayQueue
    >
    >   > DelayQueue即延迟队列，也就是一个按延迟时间从小到大出队的PriorityQueue。
    >   >
    >   > 核心为 1 把锁 + 1 个非空条件，即 lock + available。
    >
    > - SynchronousQueue
    >
    >   > SynchronousQueue是一种特殊的BlockingQueue，它本身没有容量。先调put（..），线程会阻塞；直到另外一个线程调用了take（），两个线程才同时解锁，反之亦然。对于多个线程而言，例如3个线程，调用3次put（..），3个线程都会阻塞；直到另外的线程调用3次take（），6个线程才同时解锁，反之亦然。
    >   >
    >   > 两种模式：
    >   >
    >   > - 公平模式（队列模式）：TransferQueue  ->  volatile + CAS
    >   > - 非公平模式（栈模式）：TransferStack  ->  volatile + CAS
    >
    > **BlockingDeque**
    >
    > > BlockingDeque定义了一个阻塞的双端队列接口。该接口在继承了BlockingQueue接口的同时，增加了对应的双端队列操作接口。该接口只有一个实现，就是LinkedBlockingDeque，实现原理与LinkedBlockingQueue一样。
    >
    > **CopyOnWrite**
    >
    > > CopyOnWrite指在“写”的时候，不是直接“写”源数据，而是把数据拷贝一份进行修改，再通过悲观锁或者乐观锁的方式写回。那为什么不直接修改，而是要拷贝一份修改呢？这是为了在“读”的时候不加锁。
    > >
    > > Y：适合于“写”操作很少的情况。
    >
    > - CopyOnWriteArrayList
    >
    >   > CopyOnWriteArrayList的核心数据结构也是一个数组，该数组为volatile。
    >   >
    >   > “读”函数不加锁，“写”函数加锁，且实现逻辑为写时复制一个新数组，将数据写（添加、删除、修改）到新数组中，用新数组替换旧数组。
    >
    > - CopyOnWriteArraySet 
    >
    >   > CopyOnWriteArraySet 就是用 Array 实现的一个 Set，保证所有元素都不重复。其内部是封装的一个CopyOnWriteArrayList。
    >
    > **ConcurrentLinkedQueue/Deque**
    >
    > **ConcurrentHashMap**
    >
    > > volatile + CAS + synchronzied
    > >
    > > - 头节点所在的hash数组为volatile。
    > >
    > > - put：若数组中钙hash位置为null，则不需要使用锁，通过cas操作添加，若已有元素，则通过 synchronized (f) 进行同步，其中f为数组中的头节点。
    > > - get：不加锁
    > > - remove：基于replaceNode
    > > - replaceNode：使用 synchronized (f) 进行同步，其中f为数组中的头节点。
    >
  
  - 线程池
  
    > - ThreadPoolExecutor
    >
    >   > 手动配置和调整此类时，请使用以下指南：（来自其注释）
    >   >
    >   > - <font color='00FF00'> Core and maximum pool sizes </font>
    >   >
    >   >   > ThreadPoolExecutor会根据corePoolSize和maximumPoolSize设置的边界自动调整池的大小(见getPoolSize)。当在方法execute(Runnable)中提交一个新任务时，如果运行的线程少于corePoolSize，则创建一个新线程来处理该请求，即使其他工作线程空闲。否则，如果正在运行的线程少于maximumPoolSize，则只有在队列已满时才会创建一个新线程来处理请求。通过将corePoolSize和maximumPoolSize设置为相同，可以创建一个固定大小的线程池。通过将maximumPoolSize设置为一个本质上无界的值，比如Integer.MAX_VALUE，允许池容纳任意数量的并发任务。最典型的情况是，核心池和最大池大小仅在构造时设置，但它们也可以使用setCorePoolSize和setMaximumPoolSize动态更改。
    >   >
    >   > - On-demand construction
    >   >
    >   >   > <font color='00FF00'>默认情况下，即使是核心线程也只有在新任务到达时才被初始创建和启动，但这可以使用prestartCoreThread或prestartAllCoreThreads方法动态覆盖。</font>如果使用非空队列来构造线程池，您可能希望预先启动线程。
    >   >
    >   > - Creating new threads
    >   >
    >   >   > 使用ThreadFactory创建新线程。如果没有指定，则使用Executors.defaultThreadFactory，它创建的线程都在同一个线程组中，具有相同的NORM_PRIORITY优先级和非守护线程状态。通过提供一个不同的ThreadFactory，你可以改变线程的名称、线程组、优先级、守护进程状态等。如果ThreadFactory通过newThread返回null请求创建线程失败，执行器将继续执行，但可能无法执行任何任务。线程应该拥有“modifyThread”RuntimePermission。如果使用该池的工作线程或其他线程不拥有此权限，则服务可能降级:配置更改可能不会及时生效，并且shutdown线程池可能保持在可能终止但未完成的状态。
    >   >
    >   > - Keep-alive times
    >   >
    >   >   > 如果池当前有超过corePoolSize线程，多余的线程将被终止，如果它们的空闲时间超过了keepAliveTime(参见getKeepAliveTime(TimeUnit))。这提供了一种方法，可以在池没有被积极使用时减少资源消耗。如果以后池变得更活跃，则会构造新的线程。这个参数也可以使用方法setKeepAliveTime(long, TimeUnit)动态更改。使用Long.MAX_VALUE TimeUnit.NANOSECONDS 有效地禁止空闲线程在关闭之前终止。默认情况下，keep-alive策略只适用于多于corePoolSize线程的情况，但是allowCoreThreadTimeOut(boolean)方法也可以用于将这个超时策略应用于核心线程，只要keepAliveTime的值不为零。
    >   >
    >   > - Queuing
    >   >
    >   >   > 任何BlockingQueue都可以用来传输和保存提交的任务。这个队列的使用与池的大小调整相互作用:
    >   >   >
    >   >   > - 如果运行的线程少于corePoolSize, Executor总是倾向于添加一个新线程而不是排队。
    >   >   > - 如果corePoolSize或更多的线程正在运行，Executor总是倾向于将请求排队，而不是添加一个新线程。
    >   >   > - 如果请求不能排队，则创建一个新线程，除非该线程超过maximumPoolSize，在这种情况下，任务将被拒绝。
    >   >   >
    >   >   > 有三种一般的排队策略:
    >   >   >
    >   >   > 1. 直接的传递。对于工作队列来说，一个好的默认选择是SynchronousQueue，它将任务传递给线程，而不持有它们。在这里，如果没有立即可用的线程来运行任务，尝试对任务进行排队将失败，因此将构造一个新线程。当处理可能具有内部依赖关系的请求集时，此策略可以避免锁定。直接切换通常需要无限制的maximumpoolsize，以避免拒绝新提交的任务。反过来，当命令的平均到达速度超过处理速度时，就有可能出现无限制的线程增长。
    >   >   > 2. 无界队列。使用无界队列(例如，没有预定义容量的LinkedBlockingQueue)将导致在所有corePoolSize线程都繁忙时，新的任务在队列中等待。因此，只会创建corePoolSize线程。(因此，maximumPoolSize的值没有任何影响。)当每个任务完全独立于其他任务时，这可能是合适的，因此任务不会影响其他任务的执行;例如，在一个网页服务器中。虽然这种类型的排队在平滑短暂的请求突发方面很有用，但当命令的平均到达速度超过它们的处理速度时，它承认了无限工作队列增长的可能性。
    >   >   > 3. 有界的队列。当使用有限的maximumPoolSizes时，有界队列(例如ArrayBlockingQueue)有助于防止资源耗尽，但可能更难调优和控制。队列大小和最大池大小可以相互权衡:使用大型队列和小型池可以最小化CPU使用、OS资源和上下文切换开销，但可能会人为地降低吞吐量。如果任务经常阻塞(例如，如果它们受I/O限制)，系统可能会为比您所允许的更多的线程调度时间。使用小队列通常需要更大的池大小，这使得cpu更加繁忙，但可能会遇到不可接受的调度开销，这也会降低吞吐量。
    >   >
    >   > - Rejected tasks
    >   >
    >   >   > 在方法execute(Runnable)中提交的新任务将在Executor已经关闭，以及Executor使用了有限的最大线程和工作队列容量，并且饱和时被拒绝。在这两种情况下，execute方法都调用RejectedExecutionHandler.rejectedExecution(Runnable, ThreadPoolExecutor) 方法。提供了四个预定义的处理策略:
    >   >   >
    >   >   > 1. 在默认ThreadPoolExecutor.AbortPolicy，处理程序抛出一个运行时RejectedExecutionException。
    >   >   > 2. 在ThreadPoolExecutor.CallerRunsPolicy，调用execute的线程自身运行该任务。这提供了一个简单的反馈控制机制，可以降低新任务提交的速度。
    >   >   > 3. 在ThreadPoolExecutor.DiscardPolicy，一个不能执行的任务将被简单地删除。此策略仅为那些极少的从不依赖任务完成的情况而设计。
    >   >   > 4. 在ThreadPoolExecutor.DiscardOldestPolicy，如果执行器没有关闭，工作队列头的任务将被删除，然后重试执行(可能再次失败，导致重复执行)。这种政策很难被接受。在几乎所有的情况下，您还应该取消任务，以导致任何组件在等待其完成时出现异常，并/或记录失败，如ThreadPoolExecutor.DiscardOldestPolicy文档中所描述。
    >   >   >
    >   >   > 可以定义和使用其他类型的RejectedExecutionHandler类。这样做需要一些注意，特别是当策略设计为仅在特定容量或排队策略下工作时。
    >   >
    >   > - Hook methods
    >   >
    >   >   > ​	该类提供受保护的可重写beforeExecute(Thread, Runnable)和afterExecute(Runnable, Throwable)方法，它们在每个任务执行之前和之后被调用。这些可以用来操纵执行环境;例如，重新初始化ThreadLocals、收集统计信息或添加日志条目。此外，可以覆盖终止的方法以执行在Executor完全终止后需要执行的任何特殊处理。
    >   >   >
    >   >   > ​	如果hook、callback或BlockingQueue方法抛出异常，内部工作线程可能会依次失败、突然终止，并可能被替换。
    >   >
    >   > - Queue maintenance
    >   >
    >   >   > <font color='00FF00'>方法getQueue()允许访问工作队列，以便进行监视和调试。</font>强烈制止将此方法用于其他目的。当大量排队的任务被取消时，提供的两个方法remove(Runnable)和purge可用来协助存储回收。
    >   >
    >   > - Reclamation
    >   >
    >   >   > 在程序中不再引用且没有剩余线程的池可以被回收(垃圾回收)，而无需显式关闭。您可以通过设置适当的keep-alive时间，使用0个核心线程的下界和/或设置allowCoreThreadTimeOut(boolean)来配置池，以允许所有未使用的线程最终死亡。
    >   >
    >   > 
    >
    > - ForkJoinPool
    >
    >   > - 用于运行ForkJoinTasks的ExecutorService。ForkJoinPool为来自非Forkjointask客户端的提交提供入口点，以及管理和监控操作。
    >   >
    >   > - 任务执行方法总结
    >   >
    >   > - |                       | Call from non-fork/join clients | Call from within fork/join computations       |
    >   >   | --------------------- | ------------------------------- | --------------------------------------------- |
    >   >   | 安排异步执行          | execute(ForkJoinTask)           | ForkJoinTask.fork                             |
    >   >   | 等待并获得结果        | invoke(ForkJoinTask)            | ForkJoinTask.invoke                           |
    >   >   | 安排执行，获取 Future | submit(ForkJoinTask)            | ForkJoinTask.fork (ForkJoinTasks are Futures) |
    >   >
    >   >   用于构造公共池的参数可以通过设置以下系统属性来控制:
    >   >
    >   >   > - java.util.concurrent.ForkJoinPool.common.parallelism - 并行级别，一个非负整数。
    >   >   > - java.util.concurrent.ForkJoinPool.common.threadFactory - ForkJoinPool.ForkJoinWorkerThreadFactory的类名。系统类加载器用于加载该类。
    >   >   > - java.util.concurrent.ForkJoinPool.common.exceptionHandler - Thread.UncaughtExceptionHandler的类名。系统类加载器用于加载该类。
    >   >   > - java.util.concurrent.ForkJoinPool.common.maximumSpares - 用于维护目标并行性而允许的额外线程的最大数量(默认为256)。
    >   >
    >   > - 实现注意事项:该实现将最大运行线程数限制为32767。尝试创建大于最大数量的池会导致IllegalArgumentException异常。
    >   >
    >   > - 只有当池关闭或内部资源耗尽时，该实现才会拒绝提交的任务(即通过抛出RejectedExecutionException)。
    >   >
    >   > - 构造函数参数：
    >   >
    >   >   > - parallelism - 并行度级别。对于默认值，使用Runtime.availableProcessors。
    >   >   > - factory - 创建新线程的工厂。默认值为defaultForkJoinWorkerThreadFactory。
    >   >   > - handler - 内部工作线程的处理程序，在执行任务时遇到不可恢复的错误而终止。对于默认值，使用null。
    >   >   > - asyncMode - 如果为true，为未加入的fork任务建立本地先进先出调度模式。在工作线程只处理事件式异步任务的应用程序中，此模式可能比默认的基于本地堆栈的模式更合适。默认值为false。
    >   >   > - corePoolSize - 要保持在池中的线程数(除非经过保持活动后超时)。通常情况下(默认情况下)这个值与并行级相同，但如果任务经常阻塞，可以设置为更大的值以减少动态开销。使用较小的值(例如0)具有与默认值相同的效果。
    >   >   > - maximumPoolSize -允许的最大线程数。当达到最大值时，尝试替换阻塞的线程会失败。(但是，由于不同线程的创建和终止可能会重叠，并且可能由给定的线程工厂管理，因此这个值可能会暂时超过。)要排列公共池默认使用的相同值，请使用256加上并行级别。(默认情况下，公共池最多允许256个空闲线程。)使用大于实现的总线程限制的值(例如Integer.MAX_VALUE)与使用此限制(这是默认值)具有相同的效果。
    >   >   > - minimumRunnable—不被join或ForkJoinPool.ManagedBlocker阻塞的最小允许的核心线程数。为了确保进度，当未阻塞的线程太少且可能存在未执行的任务时，会构造新的线程，直到给定的maximumPoolSize。对于默认值，请使用1，以确保活动性。在出现阻塞活动时，较大的值可能会提高吞吐量，但也可能不会，因为开销增加了。当提交的任务不能有需要额外线程的依赖项时，可以接受值为0。
    >   >   > - saturate - 如果非空，则在尝试创建大于允许的最大线程总数时调用的谓词。默认情况下，当线程在join或ForkJoinPool.ManagedBlocker上即将阻塞，但因为将超过maximumPoolSize而不能被替换时，抛出RejectedExecutionException。但是，如果这个谓词返回true，则不会抛出异常，因此池继续在少于目标可运行线程数的情况下操作，这可能无法确保进度。
    >   >   > - keepAliveTime—从最后一次使用到线程终止(然后在需要时替换)之前经过的时间。默认值为60,TimeUnit.SECONDS。
    >   >   > - unit - keepAliveTime参数的时间单位
    >   >
    >   > - commonPool()
    >   >
    >   >   > 返回公共池实例。这个池是静态构造的;它的运行状态不受shutdown或shutdownNow尝试的影响。 但是，这个池和任何正在进行的处理在程序System.exit时自动终止。 任何依赖于异步任务处理在程序终止前完成的程序都应该在退出前调用commonPool().awaitQuiescence
    >   >
    >   > - 使用示例：ForkJoinCalculator，参考：[ForkJoinPool线程池的使用以及原理](https://blog.csdn.net/f641385712/article/details/83749798)
    >   >
    >   >   > <img src="https://love-coder-blog-images.oss-cn-beijing.aliyuncs.com/images-unnamed/work%20stealing.png" alt="work stealing" style="zoom: 67%;" />
    >   >   >
    >   >   > - ForkJoinPool 的每个工作线程都维护着一个工作队列（WorkQueue），这是一个双端队列（Deque），里面存放的对象是任务（ForkJoinTask）。
    >   >   > - 每个工作线程在运行中产生新的任务（通常是因为调用了 fork()）时，会放入工作队列的队尾，并且工作线程在处理自己的工作队列时，使用的是 LIFO 方式，也就是说每次从队尾取出任务来执行。
    >   >   > - 每个工作线程在处理自己的工作队列同时，会尝试窃取一个任务（或是来自于刚刚提交到 pool 的任务，或是来自于其他工作线程的工作队列），窃取的任务位于其他线程的工作队列的队首，也就是说工作线程在窃取其他工作线程的任务时，使用的是 FIFO 方式。
    >   >   > - 在遇到 join() 时，如果需要 join 的任务尚未完成，则会先处理其他任务，并等待其完成。
    >   >   > - 在既没有自己的任务，也没有可以窃取的任务时，进入休眠。
    >
    > - Executors 工厂方法
    >
    >   > - newFixedThreadPool(int nThreads) & newFixedThreadPool(int nThreads, ThreadFactory threadFactory)
    >   >
    >   >   > - 固定线程数，未预先启动线程。
    >   >   > - 基于ThreadPoolExecutor。
    >   >   >
    >   >   > - corePoolSize == maximumPoolSize
    >   >   >
    >   >   > - 无界任务队列（Integer.MAX_VALUE）
    >   >
    >   > - newWorkStealingPool(int parallelism) & newWorkStealingPool()
    >   >
    >   >   > - 创建一个线程池，该线程池维护足够的线程以支持给定的并行性级别，并且可以使用多个队列来减少争用。并行度级别对应于参与或可参与任务处理的最大线程数。线程的实际数量可以动态地增长和收缩。工作窃取池不保证所提交任务的执行顺序。
    >   >   > - 基于ForkJoinPool
    >   >   > - newWorkStealingPool()，并行度为Java虚拟机可用cpu核心数：Runtime.getRuntime().availableProcessors()。
    >   >
    >   > - newSingleThreadExecutor() & newSingleThreadExecutor(ThreadFactory threadFactory)
    >   >
    >   >   > - 基于ThreadPoolExecutor。
    >   >   > - corePoolSize == maximumPoolSize == 1
    >   >   > - 无界任务队列（Integer.MAX_VALUE）
    >   >
    >   > - newCachedThreadPool() & newCachedThreadPool(ThreadFactory threadFactory)
    >   >
    >   >   > - 如果没有可用的现有线程，将创建一个新线程并将其添加到池中。60秒内未使用的线程将被终止并从缓存中删除。因此，一个长时间保持空闲的池不会消耗任何资源。
    >   >   > - 基于ThreadPoolExecutor。
    >   >   > - corePoolSize == maximumPoolSize == 1
    >   >   > - 无界任务队列（Integer.MAX_VALUE）
    >   >
    >   > - newSingleThreadScheduledExecutor() & newSingleThreadScheduledExecutor(ThreadFactory threadFactory)
    >   >
    >   >   > - 创建一个单线程执行器，该执行器可以安排命令在给定的延迟后运行或定期执行。
    >   >   > - 基于ScheduledThreadPoolExecutor
    >   >
    >   > - newScheduledThreadPool(int corePoolSize) & newScheduledThreadPool(int corePoolSize, ThreadFactory threadFactory)
    >   >
    >   >   > - 创建一个线程池，该线程池可以调度命令在给定的延迟后运行或定期执行。
    >   >   > - 基于其父类ThreadPoolExecutor。maximumPoolSize == Integer.MAX_VALUE
    >   >
    >   > - unconfigurableExecutorService(ExecutorService executor) & unconfigurableScheduledExecutorService(ScheduledExecutorService executor)
    >   >
    >   >   > - 返回一个对象，该对象将所有定义的ExecutorService方法委托给给定的执行器，而不是任何其他可能使用类型转换访问的方法。这提供了一种安全“冻结”配置的方法，并且不允许对给定的具体实现进行调优。
    >
    > - 