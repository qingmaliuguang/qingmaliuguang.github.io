---
title: JVM调优
date: 2022-03-21 15:06:25
tags: Java、subject
---

### 关于锁

- 自适应自旋

  > 默认开启。

- 偏向锁

  > 默认开启。- XX:+UseBiased Locking
  >
  > 如果某一类锁对象的总撤销数超过了一个阈值（对应 Java 虚拟机参数 
  > -XX:BiasedLockingBulkRebiasThreshold，默认为 20），那么 Java 虚拟机会宣布这个类的偏向锁失效。
  >
  > 如果总撤销数超过另一个阈值（对应 Java 虚拟机参数 
  > -XX:BiasedLockingBulkRevokeThreshold，默认值为 40），那么 Java 虚拟机会认为这个类已经不再适合偏向锁。此时，Java 虚拟机会撤销该类实例的偏向锁，并且在之后的加锁过程中直接为该类实例设置轻量级锁。

