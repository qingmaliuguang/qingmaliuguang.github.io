---
title: JDK中好用的工具类收集
date: 2021-09-02 08:06:06
tags: Java
---

### 数组复制

- Arrays.copyOf：基于System.arraycopy实现。

  ```java
  // ArrayList::grow
  elementData = Arrays.copyOf(elementData, newCapacity);
  ```

  

- System.arraycopy: native方法

  ```java
  // ArrayList::add(int index, E element)
  System.arraycopy(elementData, index, elementData, index + 1,
                           size - index);
  ```

  

