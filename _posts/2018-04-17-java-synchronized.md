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
|      实例对象      |  类的实例对象  | **synchronized (this)** {}                      |
|     class对象      |     类对象     | **synchronized (Demo.class)** {}                |
| 任意实例对象Object | 实例对象Object | String lock = "test"; **synchronized(lock)** {} |

需要注意的是如果锁住的是类对象的话，尽管new多个实例对象，但他们仍然属于同一个类，依然会被锁住，即线程之间保证同步关系。

## 2.1 对象锁机制

现在我们来看看synchronized的具体底层实现，一个简单的例子：

```java
public class SynchronizedDemo {
    public static void main(String[] args) {
        synchronized(SynchronizedDemo.class) {
        }
        method();
    }
    public static void method() {}
}
```

上面的代码块有一个同步代码块，锁住的是类对象，并且还有一个同步静态方法，锁住的依然是该类的类对象。编译以后，切换到其Main.class文件的同级目录以后，用：

javap -v Main.class

查看字节码文件，如下：

![synchronized]({{ '/styles/images/java/synchronized/pic1.png' | prepend: site.baseurl }})

<center>图1 Main.class的字节码文件</center>

如上图所示，注意其中的第4，6以及12行，这也是添加了synchronized以后独有的。执行同步代码块后首先要执行monitorenter指令，退出的时候执行monitorexit指令。通过分析以后可以看出，使用synchronized进行同步，其关键是必须要对对象的监视器monitor进行获取，当咸亨获取monitor后才能继续往下执行，否则就只能等待。而很显然这个获取的过程是互斥的，即同一时刻只有一个线程能够获取到monitor。上面的例子在执行完同步代码块之后紧接着会再去执行一个静态同步方法，而这个方法锁的对象依然是这个类对象，那么正在执行的线程还需要获取该锁吗？答案是不必，从上图中就可以看出，执行静态同步方法时就只有一条monitorexit指令并没有monitorenter指令来获取锁。这也就是**锁的重入性**，即在同一线程中，线程不需要再次获取同一把锁。synchronized先天具有重入性，每个对象拥有一个计数器，当咸亨获取该对象锁后，计数器加一，释放锁后计数器减一。

任意一个对象都拥有自己的监视器，当这个对象由同步块或者这个对象的同步方法调用时，执行方法的线程必须先获取该对象的监视器才能进入同步块和同步方法，如果没有获取到监视器的线程将会被阻塞在同步块和同步方法的入口处，进入到**BLOCKED**状态（关于线程的状态请参考[这里](https://www.jianshu.com/p/f65ea68a4a7f)）。

下图表示了对象，对象监视器，同步队列以及执行线程状态之间的关系：

![synchronized]({{ '/styles/images/java/synchronized/pic2.png' | prepend: site.baseurl }})

<center>图2 对象，对象监视器，同步队列以及执行线程状态之间的关系</center>

该图可以看出，任意线程对Object的访问，首先要获得Object的监视器，如果获取失败，该线程就进入同步状态，线程状态变为BLOCKED，当Object的监视器占有者释放后，在同步队列中得线程就会有机会重新获取该监视器。



<hr>
​最后的最后，老婆我爱你。








