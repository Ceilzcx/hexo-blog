---
title: Java
date: 2021-07-15 15:54:38
tags: java
---

##  Java语言



### Java基础

#### 一、`int` 和 `Integer` 区别和比较

`int` 为基本类型，`Integer`为对象类型。

```java
Integer i01 = 59;
int i02 = 59;
Integer i03 = Integer.valueOf(59);	// 和 Integer i03 = 59一样
Integer i04 = new Integer(59);
System.out.println(i01 == i02);		// true，int类型和Integer类型进行比较时，Integer会自动拆箱，变成int值比较
System.out.println(i01 == i03);		// true
System.out.println(i01 == i04);		// false，i04生成了新的对象
System.out.println(i03 == i04);		// false
```



#### 二、`String` 和 `StringBuffer` 和 `StringBuilder`

+ `String`：**不可改变的字符串**
+ `StringBuffer` ：修改字符串，动态构建字符串。方法添加 `Synchronized` 修饰符
+ `StringBuilder`：线程不安全，速度更快



#### 三、`Synchronized` 和 `Lock`

+ `Synchronized`：保证最多一个线程同时执行该代码块，抛出异常时，主动释放锁。等待锁的线程无法终端，会一直等待响应。

  **修饰静态方法将会锁住类，修饰普通方法会锁住对象的实例。**

+ `Lock`：需要 `finally` 添加 `unlock` 释放锁。



#### 四、六原则和一法则

+ 单一职责原则：类实现高内聚，最实现自己的功能
+ 开闭原则：对外扩展（通过派生新类实现新功能），对内关闭（类本身不进行修改）
+ 依赖倒转原则：面向接口编程。
+ 里氏替换原则：任何时候都可以用子类型替换掉父类型（猫不能继承狗）。
+ 接口隔离原则：接口要小而专，绝不能大而全。
+ 合成聚合复用原则：优先使用聚合或合成关系复用代码。
+ 迪米特法则：一个对象应当对其他对象有尽可能少的了解。



#### 五、请说明一下final, finally, finalize的区别。

+ `final`： 用于声明属性，方法和类，分别表示属性不可变，方法不可覆盖，类不可继承。
+ `finally`：是异常处理语句结构的一部分，表示总是执行。
+ `finalize`：是 `Object` 类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法，可以覆盖此方法提供垃圾收集时的其他资源



### 数据结构

#### 一、`Hashtable`

##### 1）简介

散列表，存储方式：键值对（key-value）映射。

**继承于Dictionary**，实现了**Map接口**

函数同步（操作的函数都有`synchronized `修饰符），线程安全

key和value都不能为 null

`Hashtable` 的实例有两个参数影响其性能：**初始容量 和 加载因子**。

数据通过单项链表的实现保存，所有值存在`Entry[] table`中

```java
class Entry<K, V> {
    final K key;	// 唯一
    V value;
    Entry<K, V> next;
}
```







### Servlet

#### 一、生命周期

+ 初始化 `init()`

  **第一次调用Servlet时调用**

+ `service()`

  执行实际任务的方法。每次服务器接收到一个Servlet请求，产生一个新的线程并调用服务。通过请求的请求类型选择调用对应的方法，例：doGet，doPost

+ `destroy()`

  destroy() 方法只会被调用一次，在 Servlet 生命周期结束时被调用。destroy() 方法可以让您的 Servlet 关闭数据库连接、停止后台线程、把 Cookie 列表或点击计数器写入到磁盘，并执行其他类似的清理活动。

  在调用 destroy() 方法之后，servlet 对象被标记为垃圾回收