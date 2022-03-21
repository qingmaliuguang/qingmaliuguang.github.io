---
title: JVM-ReadNote
date: 2021-09-06 12:54:04
tags: JVM、阅读笔记
---

# 1. 散点记录

- -Xms & -Xmx

- -XX:+HeapDumpOnOutOfMemoryError

  当抛出java.lang.OutOfMemoryError异常时，Dump出当前的**内存堆转储快照**以便进行事后分析。

- 从Java7开始字符串常量池从方法区移到了Java堆中，通过String.intern方法填充字符串常量池因此的OutOfMemoryError: Java heap space。

- 从Java8开始，永久代已没有了，方法区中剩余信息移到通过直接内存创建的元空间。字符串常量池仍在堆中不在方法区中。

  - 元空间相关参数

    - -XX:MaxMetaspaceSize=size

      元空间最大值，默认为-1，即不限制。

    - -XX:MetaspaceSize=size

      元空间初始大小

- 字符串常量池虽然物理上存放在堆中，但逻辑上仍属于方法区。

- -XX:MaxDirectMemorySize=size

  直接内存容量最大值，默认与Java堆最大值（-Xmx指定）一致。

- 垃圾回收

  - 对象存活判定算法
    - 引用计数算法
    - 可达性分析算法

  - 分代收集理论
    - 基于三条经验假说：弱分代假说、强分代假说、跨代引用假说

  - 垃圾收集算法
  - 算法实现——垃圾收集器
    - 可达性分析算法
      - 根节点枚举
        - HotSpot：使用一组称为OopMap的数据结构。
      - 安全点
        - 安全点的选定：“长时间执行”的最明显特征就是指令序列的复用，如方法调用、循环跳转、异常跳转等。
        - 在垃圾收集发生时让所有线程都跑到最近的安全点：主动式中断
          - HotSpot使用内存保护陷阱的方式，把轮询操作精简至只有一条汇编指令的程度。
      - 安全区
        - 执行到安全区时，标识自己已经进入了安全区。
        - 离开安全区域前，检查虚拟机是否已经完成了根节点枚举。
      - 跨代引用及部分区域收集（Partial GC）行为的跨区域引用问题
        - Remembered Set 记忆集  -> 常用实现：卡表（Card Table）
          - 卡表元素维护问题：HotSpot虚拟机里是通过写后屏障技术（非并发中的内存屏障）



JVM规范中定义的指令集：[2.11. Instruction Set Summary](https://docs.oracle.com/javase/specs/jvms/se16/html/jvms-2.html#jvms-2.11)

Java虚拟机指令集中的类型支持：

| pcode       | `byte`    | `short`   | `int`       | `long`    | `float`   | `double`  | `char`    | `reference` |
| ----------- | --------- | --------- | ----------- | --------- | --------- | --------- | --------- | ----------- |
| *Tipush*    | *bipush*  | *sipush*  |             |           |           |           |           |             |
| *Tconst*    |           |           | *iconst*    | *lconst*  | *fconst*  | *dconst*  |           | *aconst*    |
| *Tload*     |           |           | *iload*     | *lload*   | *fload*   | *dload*   |           | *aload*     |
| *Tstore*    |           |           | *istore*    | *lstore*  | *fstore*  | *dstore*  |           | *astore*    |
| *Tinc*      |           |           | *iinc*      |           |           |           |           |             |
| *Taload*    | *baload*  | *saload*  | *iaload*    | *laload*  | *faload*  | *daload*  | *caload*  | *aaload*    |
| *Tastore*   | *bastore* | *sastore* | *iastore*   | *lastore* | *fastore* | *dastore* | *castore* | *aastore*   |
| *Tadd*      |           |           | *iadd*      | *ladd*    | *fadd*    | *dadd*    |           |             |
| *Tsub*      |           |           | *isub*      | *lsub*    | *fsub*    | *dsub*    |           |             |
| *Tmul*      |           |           | *imul*      | *lmul*    | *fmul*    | *dmul*    |           |             |
| *Tdiv*      |           |           | *idiv*      | *ldiv*    | *fdiv*    | *ddiv*    |           |             |
| *Trem*      |           |           | *irem*      | *lrem*    | *frem*    | *drem*    |           |             |
| *Tneg*      |           |           | *ineg*      | *lneg*    | *fneg*    | *dneg*    |           |             |
| *Tshl*      |           |           | *ishl*      | *lshl*    |           |           |           |             |
| *Tshr*      |           |           | *ishr*      | *lshr*    |           |           |           |             |
| *Tushr*     |           |           | *iushr*     | *lushr*   |           |           |           |             |
| *Tand*      |           |           | *iand*      | *land*    |           |           |           |             |
| *Tor*       |           |           | *ior*       | *lor*     |           |           |           |             |
| *Txor*      |           |           | *ixor*      | *lxor*    |           |           |           |             |
| *i2T*       | *i2b*     | *i2s*     |             | *i2l*     | *i2f*     | *i2d*     |           |             |
| *l2T*       |           |           | *l2i*       |           | *l2f*     | *l2d*     |           |             |
| *f2T*       |           |           | *f2i*       | *f2l*     |           | *f2d*     |           |             |
| *d2T*       |           |           | *d2i*       | *d2l*     | *d2f*     |           |           |             |
| *Tcmp*      |           |           |             | *lcmp*    |           |           |           |             |
| *Tcmpl*     |           |           |             |           | *fcmpl*   | *dcmpl*   |           |             |
| *Tcmpg*     |           |           |             |           | *fcmpg*   | *dcmpg*   |           |             |
| *if_TcmpOP* |           |           | *if_icmpOP* |           |           |           |           | *if_acmpOP* |
| *Treturn*   |           |           | *ireturn*   | *lreturn* | *freturn* | *dreturn* |           | *areturn*   |

Java虚拟机实际类型与Java虚拟机计算类型的对应关系如表所示：

| Actual type     | Computational type | Category |
| --------------- | ------------------ | -------- |
| `boolean`       | `int`              | 1        |
| `byte`          | `int`              | 1        |
| `char`          | `int`              | 1        |
| `short`         | `int`              | 1        |
| `int`           | `int`              | 1        |
| `float`         | `float`            | 1        |
| `reference`     | `reference`        | 1        |
| `returnAddress` | `returnAddress`    | 1        |
| `long`          | `long`             | 2        |
| `double`        | `double`           | 2        |



Java里面final关键字通常在以下几种情况下被使用

- Final Variables
- Final Methods
- Final Classes

在Java里面final常量不能被改变，final类不能被继承，final方法不能被重写。

# 2. JVM内存模型

![JVM Memory](https://gitee.com/qmlg/image-bed/raw/master/images/JVM%20Memory.jpg)

## 2.1 局部变量表

**局部变量表(Local Variable Table)**是一组变量值存储空间，用于存放方法参数和方法内定义的局部变量。局部变量表的容量以变量槽(Variable Slot)为最小单位，Java虚拟机规范并没有定义一个槽所应该占用内存空间的大小，但是规定了一个槽应该可以存放一个32位以内的数据类型。

> 在Java程序编译为Class文件时,就在方法的Code属性中的max_locals数据项中确定了该方法所需分配的局部变量表的最大容量。(最大Slot数量)

一个局部变量可以保存一个类型为boolean、byte、char、short、int、float、reference和returnAddress类型的数据。reference类型表示对一个对象实例的引用。returnAddress类型是为jsr、jsr_w和ret指令服务的，目前已经很少使用了。

虚拟机通过索引定位的方法查找相应的局部变量，索引的范围是从0~局部变量表最大容量。如果Slot是32位的，则遇到一个64位数据类型的变量(如long或double型)，则会连续使用两个连续的Slot来存储。

## 2.2 操作数栈

**操作数栈(Operand Stack)**也常称为操作栈，它是一个后入先出栈(LIFO)。同局部变量表一样，操作数栈的最大深度也在编译的时候写入到方法的Code属性的max_stacks数据项中。

操作数栈的每一个元素可以是任意Java数据类型，32位的数据类型占一个栈容量，64位的数据类型占2个栈容量,且在方法执行的任意时刻，操作数栈的深度都不会超过max_stacks中设置的最大值。

当一个方法刚刚开始执行时，其操作数栈是空的，随着方法执行和字节码指令的执行，会从局部变量表或对象实例的字段中复制常量或变量写入到操作数栈，再随着计算的进行将栈中元素出栈到局部变量表或者返回给方法调用者，也就是出栈/入栈操作。一个完整的方法执行期间往往包含多个这样出栈/入栈的过程。

## 2.3 动态连接

在一个class文件中，一个方法要调用其他方法，需要将这些方法的符号引用转化为其在内存地址中的直接引用，而符号引用存在于方法区中的运行时常量池。

Java虚拟机栈中，每个栈帧都包含一个指向运行时常量池中该栈所属方法的符号引用，持有这个引用的目的是为了支持方法调用过程中的**动态连接(Dynamic Linking)**。

这些符号引用一部分会在类加载阶段或者第一次使用时就直接转化为直接引用，这类转化称为**静态解析**。另一部分将在每次运行期间转化为直接引用，这类转化称为动态连接。

## 2.4 方法返回

**当一个方法开始执行时，可能有两种方式退出该方法：**

- **正常完成出口**
- **异常完成出口**

**正常完成出口**是指方法正常完成并退出，没有抛出任何异常(包括Java虚拟机异常以及执行时通过throw语句显示抛出的异常)。如果当前方法正常完成，则根据当前方法返回的字节码指令，这时有可能会有返回值传递给方法调用者(调用它的方法)，或者无返回值。具体是否有返回值以及返回值的数据类型将根据该方法返回的字节码指令确定。

**异常完成出口**是指方法执行过程中遇到异常，并且这个异常在方法体内部没有得到处理，导致方法退出。

> 无论是Java虚拟机抛出的异常还是代码中使用athrow指令产生的异常，只要在本方法的异常表中没有搜索到相应的异常处理器，就会导致方法退出。

无论方法采用何种方式退出，在方法退出后都需要返回到方法被调用的位置，程序才能继续执行，方法返回时可能需要在当前栈帧中保存一些信息，用来帮他恢复它的上层方法执行状态。

> 方法退出过程实际上就等同于把当前栈帧出栈，因此退出可以执行的操作有：恢复上层方法的局部变量表和操作数栈，把返回值(如果有的话)压如调用者的操作数栈中，调整PC计数器的值以指向方法调用指令后的下一条指令。

一般来说，方法正常退出时，调用者的PC计数值可以作为返回地址，栈帧中可能保存此计数值。而方法异常退出时，返回地址是通过异常处理器表确定的，栈帧中一般不会保存此部分信息。

## 2.5 附加信息

虚拟机规范允许具体的虚拟机实现增加一些规范中没有描述的信息到栈帧之中，例如和调试相关的信息，这部分信息完全取决于不同的虚拟机实现。在实际开发中，一般会把动态连接，方法返回地址与其他附加信息一起归为一类，称为栈帧信息。

## 2.6 方法区

- 全局字符串池（string pool也有叫做string literal pool）
- class文件常量池（class constant pool）
- 运行时常量池（runtime constant pool）

**物理存储位置变更**：

- 在**JDK1.7之前**运行时常量池逻辑包含字符串常量池存放在方法区, 此时hotspot虚拟机对方法区的实现为永久代。
- 在**JDK1.7** 字符串常量池被从方法区拿到了堆中, 这里没有提到运行时常量池,也就是说字符串常量池被单独拿到堆,运行时常量池剩下的东西还在方法区, 也就是hotspot中的永久代。
- 在**JDK1.8** hotspot移除了永久代用元空间(Metaspace)取而代之, 这时候字符串常量池还在堆, 运行时常量池还在方法区, 只不过方法区的实现从永久代变成了元空间(Metaspace) 。

# 3. 可能与并发相关的点

## 3.1 对象创建过程 new指令

1. 当Java虚拟机遇到一条字节码new指令时，首先将去检查这个指令的参数是否能在常量池中定位到一个类的符号引用，并且检查这个符号引用代表的类是否已被加载、解析和初始化过。如果没有，那必须先执行相应的类加载过程。

2. 在类加载检查通过后，接下来虚拟机将为新生对象分配内存。  

   > 【此处涉及-XX +/- UseTLAB】

3. 内存分配完成之后，虚拟机必须将分配到的内存空间（但不包括对象头）都初始化为零值，如果使用了TLAB的话，这一项工作也可以提前至TLAB分配时顺便进行。这步操作保证了对象的实例字段在Java代码中可以不赋初始值就直接使用，使程序能访问到这些字段的数据类型所对应的零值。

4. 接下来，Java虚拟机还要对对象进行必要的设置，例如这个对象是哪个类的实例、如何才能找到类的元数据信息、对象的哈希码（实际上对象的哈希码会延后到真正调用Object::hashCode()方法时才计算）、对象的GC分代年龄等信息。这些信息存放在对象的对象头（Object Header）之中。根据虚拟机当前运行状态的不同，如是否启用偏向锁等，对象头会有不同的设置方式。  

   > 【此处涉及是否启用偏向锁】

5. 在上面工作都完成之后，从虚拟机的视角来看，一个新的对象已经产生了。但是从Java程序的视角看来，对象创建才刚刚开始——构造函数，即Class文件中的<init>()方法还没有执行，所有的字段都为默认的零值，对象需要的其他资源和状态信息也还没有按照预定的意图构造好。new指令之后会接着执行<init>()方法，按照程序员的意愿对对象进行初始化，这样一个真正可用的对象才算完全被构造出来。

> 此过程描述见《深入理解Java虚拟机 第三版》2.3.1。

## 3.2  对象内存布局

- 在HotSpot虚拟机里，对象在堆内存中的存储布局可以划分为三个部分：对象头（Header）、实例数据（Instance Data）和对齐填充（Padding）。
- HotSpot虚拟机对象的对象头部分包括两类信息。第一类是用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32个比特和64个比特，官方称它为“Mark Word”。
- 接下来实例数据部分是对象真正存储的有效信息，即我们在程序代码里面所定义的各种类型的字段内容，无论是从父类继承下来的，还是在子类中定义的字段都必须记录起来。这部分的存储顺序会受到虚拟机分配策略参数（-XX：FieldsAllocationStyle参数）和字段在Java源码中定义顺序的影响。HotSpot虚拟机默认的分配顺序为longs/doubles、ints、shorts/chars、bytes/booleans、oops（Ordinary Object Pointers，OOPs），从以上默认的分配策略中可以看到，相同宽度的字段总是被分配到一起存放，在满足这个前提条件的情况下，在父类中定义的变量会出现在子类之前。如果HotSpot虚拟机的+XX：CompactFields参数值为true（默认就为true），那子类之中较窄的变量也允许插入父类变量的空隙之中，以节省出一点点空间。

> 涉及-XX：FieldsAllocationStyle 和 +XX：CompactFields

- 对象的第三部分是对齐填充，由于HotSpot虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说就是任何对象的大小都必须是8字节的整数倍。对象头部分已经被精心设计成正好是8字节的倍数（1倍或者2倍），因此，如果对象实例数据部分没有对齐的话，就需要通过对齐填充来补全。

HotSpot虚拟机对象头Mark Word

![截屏 2021-10-12 下午2.30.21](https://gitee.com/qmlg/image-bed/raw/master/images/%E6%88%AA%E5%B1%8F%202021-10-12%20%E4%B8%8B%E5%8D%882.30.21.png)

- 此外，如果对象是一个Java数组，那在对象头中还必须有一块用于记录数组长度的数据，因为虚拟机可以通过普通Java对象的元数据信息确定Java对象的大小，但是如果数组的长度是不确定的，将无法通过元数据中的信息推断出数组的大小。

# 4. JVM工具 与 调优

## 4.1 JVM 异常

JVM只会抛出两种异常：OutOfMemoryError、StackOverflowError。

- 程序计数器：不会产生异常。

- **方法区**

  - OutOfMemoryError

    > Y：可能常出现的是字符串常量池发生OutOfMemoryError，如大量使用String.intern方法向字符串常量池添加字符串。
    >
    > 在大量使用反射、动态代理、CGLib等字节码框架，动态生成JSP以及OSGi这类频繁自定义类加载器的场景中，若加载大量类型，而无法下载的情况下也会内存溢出。

- **堆**

  - OutOfMemoryError

    > **java.lang.OutOfMemoryError: Java heap space**
    >
    > -Xms 和 -Xmx 设置堆大小限制。
    >
    > -XX:+HeapDumpOnOutOfMemoryError  让虚拟机在发生内存溢出时Dump出当前的内存堆转储快照以便进行事后分析。
    >
    > 测试：-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
    >
    > ![image-20211012153546322](https://gitee.com/qmlg/image-bed/raw/master/images/image-20211012153546322.png)
    >
    > ![image-20211012153650569](https://gitee.com/qmlg/image-bed/raw/master/images/image-20211012153650569.png)
    >
    > ![image-20211012153730127](https://gitee.com/qmlg/image-bed/raw/master/images/image-20211012153730127.png)

- **虚拟机栈和本地方法栈**

  > - Hotspot中共用虚拟机栈和本地方法栈
  > - Hotspot中不支持栈的动态扩展

  - OutOfMemoryError：只可能是在**创建线程**申请内存时无法获得足够内存而出现OutOfMemoryError。

    > 通过工具分析堆转储文件：
    >
    > 1. 首先确认是出现了内存溢出（Memory Overflow）还是内存泄漏（Memory Leak）。
    > 2. 如果是内存泄漏，可进一步通过工具查看泄漏对象到GC Roots的引用链，找到泄漏对象是通过怎样的引用路径、与哪些GC Roots相关联，才导致垃圾收集器无法回收它们，根据泄漏对象的类型信息以及它到GC Roots引用链的信息，一般可以比较准确地定位到这些对象创建的位置，进而找出产生内存泄漏的代码的具体位置。

  - StackOverflowError：栈容量不足以容纳新的栈帧。

    > -Xss 参数设定栈容量。
    >
    > Y：很可能是方法调用链过深，如递归程序层次过深或未定义终止条件等。

- **本机直接内存**

  - OurOfMemoryError

    > java.lang.OutOfMemoryError
    >     at sun.misc.Unsafe.allocateMemory(Native Method)
    >
    > -XX：MaxDirectMemorySize  直接内存的容量大小可通过该参数来指定，如果不去指定，则默认与Java堆最大值（由-Xmx指定）一致。

## 4.2 工具

### jmap

生成指定进程当前的内存堆转储文件。

```shell
jmap -dump:file=javaDump.hprof,format=b pid
# 如下，其中3614为进程ID
jmap -dump:file=./javaDump.hprof,format=b 3614
```

> Y：配合-XX:+HeapDumpOnOutOfMemoryError使用，-XX:+HeapDumpOnOutOfMemoryError在系统发生内存溢出时自动生成堆转储文件，jmap可在平时生成以时常观察有没有隐患。

命令详情jmap -h 或者参考：https://www.cnblogs.com/sxdcgaq8080/p/11089664.html

### jhat

分析内存堆转储文件，提供一个本地服务，可通过浏览器查看。

> Y：基本不会使用。

```shell
# 如下，通过http://localhost:7000查看分析结果。
$ jhat java_pid97001.hprof
Reading from java_pid97001.hprof...
Dump file created Tue Oct 12 15:22:00 CST 2021
Snapshot read, resolving...
Resolving 818254 objects...
WARNING:  Failed to resolve object id 0x7bf3aab60 for field type (signature L)
Chasing references, expect 163 dots...................................................................................................................................................................
Eliminating duplicate references...................................................................................................................................................................
Snapshot resolved.
Started HTTP server on port 7000
Server is ready.
```

> ![image-20211012155934401](https://gitee.com/qmlg/image-bed/raw/master/images/image-20211012155934401.png)
>
> 点击Show heap histogram
>
> ![image-20211012160026138](https://gitee.com/qmlg/image-bed/raw/master/images/image-20211012160026138.png)
>
> Y：也可使用JProfiler等工具打开堆转储文件，进行查看。

### jstat

参考：https://blog.csdn.net/zhaozheng7758/article/details/8623549

### jstack

参考：https://www.jianshu.com/p/8d5782bc596e

参考：https://www.cnblogs.com/kongzhongqijing/articles/3630264.html

### jinfo

jinfo 是 JDK 自带的命令，可以用来查看正在运行的 java 应用程序的扩展参数，包括Java System属性和JVM命令行参数；也可以动态的修改正在运行的 JVM 一些参数。当系统崩溃时，jinfo可以从core文件里面知道崩溃的Java应用程序的配置信息。

参考：https://www.jianshu.com/p/8d8aef212b25

### jps

jps是查看java进程信息的命令。

格式  jps [ options ] [ hostid ]
参数说明
     options
          -q   只输出java进程的进程id
          -l    输出java进程的进程id和main方法的类全名
          -m  输出java进程的进程id和main方法的入参
          -v   输出java进程的进程id和jvm的入参
          -V   输出java进程的进程id和通过flag文件传入jvm的参数
     hostid
          命令对应的服务器ip，默认不加参数，代码查看本机

```shell
# 用来查看Java进程，及其jvm入参，比如查看IDEA、Hive等应用进程的jvm设置。
jps -v
```

### JProfiler

### jvisualvm

## 4.3 垃圾收集器与内存分配策略

**回收区域**：堆、方法区。

### 4.3.1 堆的回收

**判断对象是否需要回收**

- 引用计数算法  ->  存在对象循环引用问题

- 可达性分析算法  -> JVM 采用该算法

  - 起始节点集：<font color="FF0000">GC Roots</font>

    1. 在**虚拟机栈**（栈帧中的**本地变量表**）中引用的对象，譬如各个线程被调用的方法堆栈中使用到的参数、局部变量、临时变量等。

    2. 在**方法区**中**类静态属性**引用的对象，譬如Java类的**引用类型静态变量**。

    3. 在**方法区**中**常量引用**的对象，譬如字符串常量池（String Table）里的引用。

    4. 在**本地方法栈**中JNI（即通常所说的**Native方法**）引用的对象。

       > Y：如在调用Unsafe.compareAndSwapInt方法时传入的参数。

    5. **Java虚拟机内部的引用**，如基本数据类型对应的Class对象，一些常驻的异常对象（比如NullPointExcepiton、OutOfMemoryError）等，还有系统类加载器。

    6. 所有被**同步锁（synchronized关键字）持有的对象**。

    7. 反映Java虚拟机内部情况的JMXBean、JVMTI中注册的回调、本地代码缓存等。

    8. 除了这些固定的GC Roots集合以外，根据用户所选用的垃圾收集器以及当前回收的内存区域不同，还可以有其他对象“临时性”地加入，共同构成完整GC Roots集合。

       > Y：比如基于分代收集的垃圾收集器在mirror GC时，RemberSet中的老年代对象也会算作GC Roots。

**分代收集理论**

- 基于三条经验假说：弱分代假说、强分代假说、跨代引用假说
  1. 弱分代假说：绝大多数对象都是朝生夕灭的。 
  2. 强分代假说：熬过越多次垃圾收集过程的对象就越难以消亡。
  3. 跨代引用假说：跨代引用相对于同代引用来说仅占极少数。

**垃圾收集算法**

- 标记清除

- 标记复制

  > 新生代采用的Appel式回收采用的该方法。（通常Eden：Survivor1：Survivor=8:1:1，老年代进行分配担保。）

- 标记整理

**对象进入老年代的情况**

- 新生代回收率不足90%，当Survivor空间满了以后，剩余存活对象进入老年代。【老年代对新生代的分配担保】

- 大对象直接进入老年代

  - G1认为只要大小超过了一个Region容量一半的对象即可判定为大对象。-XX:G1HeapRegionSize可设置Region大小。

  - 对于Serial和ParNew，-XX:PretenureSizeThreshold设置大对象的阈值。

    > Y：在jdk文档中没有找到该参数。https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html#BGBCIEFC

- 长期存活的对象进入老年代

  - 新生代的对象熬过一次minor GC，年龄加1（对象头中记录年龄），增加到一定程度，进入老年代。

    > -XX:MaxTenuringThreshold 来设定年龄阈值。并行(吞吐量)收集器的默认值是15,CMS收集器的默认值是6。

- 如果在Survivor空间中相同年龄所有对象大小的总和大于Survivor空间的一半，年龄大于或等于该年龄的对象就可以直接进入老年代，无须等到-XX：MaxTenuringThreshold中要求的年龄。【动态年龄判定】

**HotSpot的算法实现细节**

- OopMap -> 解决根节点枚举 “Stop The World”的问题

- 安全点 -> 解决OopMap更新频率与时机的问题（解决如何停顿用户线程让虚拟机进入垃圾回收状态的问题）

- 安全区域 -> 对安全点机制的补充

- 记忆集与卡表 -> 解决对象跨代引用或者跨区域引用（对于跨区域收集的GC行为）的问题

- 写屏障 -> 解决卡表元素如何维护的问题

  > 卡表在高并发场景下还面临着“伪共享”（False Sharing）问题。
  >
  > XX：+UseCondCardMark，用来决定是否开启卡表更新的条件判断。开启会增加一次额外判断的开销，但能够避免伪共享问题，

- 三色标记 && ( 增量更新 || 原始快照 ) -> 并发的可达性分析

  - CMS 增量更新
  - G1 原始快照

**HotSpot 的经典垃圾收集器**

<img src="https://gitee.com/qmlg/image-bed/raw/master/images/%E6%88%AA%E5%B1%8F%202021-10-12%20%E4%B8%8B%E5%8D%885.54.50.png" alt="截屏 2021-10-12 下午5.54.50" style="zoom: 25%;" />

- ParNew/CMS/Serial Old【JDK8默认的组合方式】

  > -XX:+UseConcMarkSweepGC，启用CMS之后新生代默认使用ParNew。
  >
  > -XX:SurvivorRatio

  CMS：

  ![截屏 2021-10-12 下午8.03.22](https://gitee.com/qmlg/image-bed/raw/master/images/%E6%88%AA%E5%B1%8F%202021-10-12%20%E4%B8%8B%E5%8D%888.03.22.png)

  优点：并发收集、低停顿

  缺点：

  - 处理器资源敏感；
  - 无法处理“浮动垃圾”，有可能出现“Concurrent Mode Failure”失败进而导致另一次完全“Stop The World” 的Full GC

  > -XX:CMSInitiatingOccupancyFraction，默认值-1，即无效值，则使用-XX:CMSTriggerRatio的值。
  >
  > -XX:CMSTriggerRatio 默认值80%

- G1【JDK9到目前的JDK17的默认GC】

  > -XX:G1HeapRegionSize -> 设置Region容量大小，该值是2的幂，范围从1mb到32mb。默认的区域大小是根据堆大小决定的，目标大约是2048个区域。
  >
  > -XX:MaxGCPauseMillis -> 设置最大G1收集器GC暂停时间的目标(以毫秒为单位)。默认200ms，

  <img src="https://gitee.com/qmlg/image-bed/raw/master/images/%E6%88%AA%E5%B1%8F%202021-10-12%20%E4%B8%8B%E5%8D%888.43.09.png" alt="截屏 2021-10-12 下午8.43.09" style="zoom:25%;" />

  <img src="https://gitee.com/qmlg/image-bed/raw/master/images/%E6%88%AA%E5%B1%8F%202021-10-12%20%E4%B8%8B%E5%8D%888.42.12.png" alt="截屏 2021-10-12 下午8.42.12" style="zoom: 25%;" />

  - Region中还有一类特殊的**Humongous区域，专门用来存储大对象**。G1认为只要大小超过了一个Region容量一半的对象即可判定为大对象。每个Region的大小可以通过参数-XX：G1HeapRegionSize设定，取值范围为1MB～32MB，且应为2的N次幂。而对于那些超过了整个Region容量的超级大对象，将会被存放在N个连续的Humongous Region之中，G1的大多数行为都把**Humongous Region作为老年代的一部分来进行看待**。

  - G1收集器除了并发标记外，其余阶段也是要完全暂停用户线程的，换言之，它并非纯粹地追求低延迟，官方给它设定的目标是**在延迟可控的情况下获得尽可能高的吞吐量**，所以才能担当起“全功能收集器”的重任与期望。

  - 通常把期望停顿时间设置为一两百毫秒或者两三百毫秒会是比较合理的。

  - 如果内存回收的速度赶不上内存分配的速度，G1收集器也要被迫冻结用户线程执行，导致**Full GC**而产生长时间“Stop The World”。

  - G1从**整体**来看是基于“**标记-整理**”算法实现的收集器，但从**局部**（两个Region之间）上看又是基于“**标记-复制**”算法实现，无论如何，这两种算法都意味着G1运作期间**不会产生内存空间碎片**。

  - 按照《深入理解JVM》作者的实践经验，目前在**小内存应用**上**CMS**的表现大概率仍然要会优于G1，而在**大内存**应用上**G1**则大多能发挥其优势，这个优劣势的**Java堆容量平衡点**通常在**6GB至8GB**之间。

    > Y：若堆为8G，则默认Region的大小约为4M。

- ZGC

  - 我们可以给ZGC下一个这样的定义来概括它的主要特征：ZGC收集器是一款基于Region内存布局的，（暂时）不设分代的，使用了读屏障、染色指针和内存多重映射等技术来实现可**并发的标记-整理**算法的，**以低延迟为首要目标**的一款垃圾收集器。

  - 将会成为服务端、大内存、低延迟应用的首选收集器的有力竞争者。

    > Y：可见内存要求也比较高。

### 4.3.2 方法区的回收

方法区的垃圾收集主要回收两部分内容：废弃的常量和不再使用的类型。

> Y：判断类型不再需要的条件苛刻，包括三个方面，实例、加载它的类加载器已被回收，Class对象不再被引用。

关于是否要对类型进行回收，HotSpot虚拟机提供了-Xnoclassgc参数进行控制，还可以使用-verbose：class以及-XX：+TraceClass-Loading、-XX：+TraceClassUnLoading查看类加载和卸载信息，其中-verbose：class和-XX：+TraceClassLoading可以在Product版的虚拟机中使用，-XX：+TraceClassUnLoading参数需要FastDebug版[1]的虚拟机支持。

在大量使用反射、动态代理、CGLib等字节码框架，动态生成JSP以及OSGi这类频繁自定义类加载器的场景中，通常都需要Java虚拟机具备类型卸载的能力，以保证不会对方法区造成过大的内存压力。

## 4.4 调优

一般的项目应该都是不需要调优的：https://www.zhihu.com/question/362201242

- -Xms & -Xmx 最大堆 最小堆

- GC算法：jdk8默认CMS，jdk9到目前的jdk17默认G1

- -Xmn

  为分代收集器中的年轻代设置堆的初始大小和最大大小(以字节为单位)。堆的年轻代区域用于新对象。与其他区域相比，该区域执行GC的频率更高。如果年轻代的大小太小，那么将执行大量minor GC。如果大小太大，则只执行full GC，这可能需要很长时间才能完成。建议您不要为G1收集器设置年轻代的大小，并且对于其它收集器保持年轻代的大小大于总体堆大小的25%而小于50%。

- ![image-20211012212915394](https://gitee.com/qmlg/image-bed/raw/master/images/image-20211012212915394.png)

------

> 内容来源：
>
> - 《深入理解Java虚拟机》
> - [The Java® Virtual Machine Specification-*Java SE 16 Edition*](https://docs.oracle.com/javase/specs/jvms/se16/html/index.html)
> - [Java虚拟机—栈帧、操作数栈和局部变量表](https://zhuanlan.zhihu.com/p/45354152)
> - [Java中几种常量池的区分](http://tangxman.github.io/2015/07/27/the-difference-of-java-string-pool/)
> - [JDK1.8关于运行时常量池, 字符串常量池的要点](https://blog.csdn.net/q5706503/article/details/84640762)

