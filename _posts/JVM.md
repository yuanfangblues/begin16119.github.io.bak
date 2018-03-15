---
layout:     post
title:      Jvm基础知识
subtitle:   
date:       2018-01-05
author:     BY
header-img: img/home-bg-o.jpg
catalog: true
tags:
    - Java
    - 基础
    - JVM
---

# JVM结构和其模块作用

- 程序计数器：线程私有，用来保存当前执行的地址。

- 虚拟机栈：用于程序执行。包含局部变量表，操作数栈，返回地址。每一个函数都是一个栈帧结构,方法结束，则栈帧回收。

- 堆:用于存放对象的地方。每个对象都有对象头，对象头里面有三个字段，MarkWord里面存放对象的hashcode, 是否加锁，偏向锁，线程持有等信息。
lengthArray字段（如果是数组则有）。

- 方法区：也是非堆，用于存放类的信息，静态数据，常量，方法等信息。

# 对象的创建、定位、回收

java中生成对象，主要存放在堆中。

对象如何创建过程时怎样？

当调用new关键字时，会在堆中开辟一块内存，这时候静态数据再类加载的时候已经初始化了，而成员变量还未初始化。当调用init()方法，也就是构造器时，这时候才初始化成员变量。

如何定位对象呢?

- 一般有两种方式，第一种类似在堆中保存一张表，栈帧中直接引用表的index，val存放对象的地址。好处显而易见，当val也就是对象的地址变化时，index不必改变，只需要修改val.

- 第二种就是直接饮用，java spothot虚拟机采用的这一种方式。栈帧中直接引用堆中对象的地址。

要回收哪些对象？

没有被引用的对象。

有两种策略：引用计数法，对象被引用，则计数器加一，反之减一。为0则会回收。

可达性分析：==树根是什么？==
1.本地变量表中对象
2.方法区静态属性引用的对象
3.方法区中常量引用的对象

# 年轻代如何进入老年代？

1.阈值默认15.每次经过MinorGC年龄+1.
2.如果在S空间中相同年龄的对象总和大于S空间的一半，则大于等于该年龄的进入老年代。

如何回收对象？

有些像花一样，朝升夕落。有些像海龟一样，万寿无疆。

所以要区分对待。

基本操作是，把堆空间存放想象成一张8*8的表格，每一个小格子里都有对象。

Mark sweep : 标记-清理算法。

执行过程-> 先遍历堆中所有对象，如果没有被引用，则打上标记。下次遍历时清理掉标记过的对象。

缺点：每次都要先标记,而且会产生内存碎片，空间利用率不高。

于是为了弥补缺点：

分块-复制 ：将内存分为两块，每次只使用一块。假设A、B两块，将对象先放在A中，当回收时，将A中存活的对象全部移到B中，再清除A中所有对象。但是每次只能利用一半的空间。很浪费。

在年轻代里：采用分块-复制的改进版本，经过数据分析，java中90%的对象都是朝生夕死，将这一块分为Edan_suer1_ser2,比例8:1:1每次都使用伊甸园和一块S部分，也就是90%的空间，回收时，将对象移动到s2中，再次后面对象s2不够则放到伊甸园中。如此反复。

在老年代里：对象经常很难被撤销引用，例如占空间寿命长的数组。采用标记-整理算法，因为不经常回收，标记后清楚没有引用的对象，在移动新对象，减少内存碎片。

# 内存溢出和内存泄漏

内存溢出：OOM.

程序计数器不考虑。

虚拟机栈：虚拟机栈中，超过一定量的栈帧会导致StackOverFlow.当线程创建过多时，会产生OOM错误。因为jvm内存，忽略pc,减去堆，减去方法区，本地内存，就只有虚拟机栈，而虚拟机栈又是线程隔离的，每个线程产生都会产生内存消耗，最终OOM.

堆: 堆中容器or数组等大对象，不断增加元素，最终OOM.因为引用一直都在，所以无法回收。

方法区：方法区主要存放类的信息，静态，常量，方法等信息。当加载过多的类的时候就会OOM.例如，cglib,bytecode,各种字节码框架，会加载过多的信息到方法区，最终OOM.

本地内存:NIO方面buffer申请的本地内存。

以上的解决方案：调参数。栈-Xss 20M ,对-Xms最小堆内存 -Xmm 最大堆内存，-XX:PermSize -XX:MaxPermSize.

==内存泄漏==

# GC和内存回收策略

GC回收器一般成对兑现，新生代和老年代。

一般出现全局安全点才运行gc,即stop the world.

串行回收器：Serial:用户线程停止，gc线程执行，且单线程。

并行回收器：Paller：用户线程停止,gc多线程执行。

并发回收器：CMS：用户线程不停止和gc线程并发执行。
