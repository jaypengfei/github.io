---
layout: post
title:  Java关键字之synchronized
date:   2018-04-17 16:19:00 +0800
categories: Java
tag: synchronized
---

* content
{:toc}




本文转载自[简书]( https://www.jianshu.com/p/157279e6efdb)

<hr>

1.简介
====================================

在之前的文章中我们已经了解了一些关于Java内存模型的一些知识，并且知道出现线程安全主要来源于JMM的设计，集中在**主内存和线程的工作内存而导致的内存可见性问题，以及重排序导致**的问题，进一步知道了happens-before规则。线程运行时拥有自己的栈空间，会在自己的栈空间运行，如果多线程间没有共享的数据，那么多线程就不能发挥优势。那么共享数据的线程安全问题如何处理？最简单的方法就是每一个线程依次的去读写这个共享变量，这样就不存在任何数据安全的问题。那么Java的关键字synchronized就提供了这样的功能，使得线程依次排队操作共享变量，很显然这种同步的效率很低，但它是也其他并发容器实现的基础。只有理解了它才能大大提升对于并发编程的理解。

# 2.实现原理

在Java代码中使用synchronized可用在代码块和方法中，具体有如下几种使用场景：

<center>表1 synchronized的使用场景</center>

|        分类        |    被锁对象    | 伪代码                                          |
| :----------------: | :------------: | ----------------------------------------------- |
|      实例方法      |  类的实例对象  | public **synchronized** void method() {}        |
|      静态方法      |     类对象     | public **static synchronized** void method() {} |
|      实例对象      |  类的实例对象  | synchronized (this) {}                          |
|     class对象      |     类对象     | synchronized (Demo.class) {}                    |
| 任意实例对象Object | 实例对象Object | String lock = "test"; synchronized(lock) {}     |







<hr>
​最后的最后，老婆我爱你。








