---
layout: post
title:  Java Concurrency
date:   2017-08-06 15:52:00 +0800
categories: Java
tag: Concurrency
---

* content
{:toc}


本文参考《Java Concurrency in Practice》。
<hr>

4.对象的组合
====================================

我们并不希望对每次一的内存访问都进行分析以确保程序是线程安全的，而是希望将一些现有的线程安全组件组合为更大规模的组件或程序。这一章将介绍一些组合模式，这些模式能够使一个类更容易称为线程安全的，并且在维护这些类时不会无意中破坏类的安全保证。<br>

4.1设计线程安全的类
-----------------------------------
在线程安全的程序中，虽然可以将程序的所有状态都保存在公有的静态域中，但与那些将状态封装起来的程序相比，这些程序的线程安全性更难以得到验证，并且在修改时也更难以确保其线程安全性。通过使用封装技术，可以使得在不对整个程序进行分析的情况下就可以判断一个类是否是线程安全的。<br>
在设计线程安全类的过程中，需要包含以下三个基本要素：
+ 找出构成对象状态的所有变量。

+ 找出约束状态变量的不变性条件。

+ 建立对象状态的并发访问管理策略。

要分析对象的状态，首先从对象的域开始。如果对象中的所有域都是基本类型的变量，那么这些域将构成对象的全部状态。程序4-1中的Counter只有一个域value，因此这个域就是Counter的全部状态。对于含有n个基本类型域的对象，其状态就是这些域构成的n元组。例如，二维点的状态就是它的坐标值（x，y）。如果在对象的域中引用了其他对象，那么该对象的状态将包含被引用对象的域`。例如LinkedList的状态就包括该链表中所有节点对象的状态。

  ``` java
//程序4-1 使用Java监视器模式的线程安全计数器  
@ThreadSafe
  public final class Counter {
  	@GuardedBy("this") private long value = 0;
    	public synchronized long getValue() {
        return value;
    	}
    	public synchronized long increment() {
        if(value == long.MAX_VALUE)
          throw new illegalStateException("counter overflow");
        return ++value;
    	} 
  }
  ```



<hr>
​最后的最后，老婆我爱你。








