---
layout: post
title:  设计模式
date:   2018-04-24 13:19:00 +0800
categories: Java
tag: Design pattern
---

* content
{:toc}
<hr>

> 参考《Java程序性能优化》

设计模式是前人工作的总结和提炼。通常，被人们广泛流传的设计模式都是对某一特定问题的成熟解决方案。如果能合理地使用设计模式，不仅能使系统更容易被他人理解，同时也能使系统拥有更加合理的结构。下面介绍的是一些经典的设计模式。

1.单例模式
------------------------------------

单例模式是设计模式中使用最为普遍的模式之一。它是一种对象创建模式，用于产生一个对象的具体实例，它可以确保系统中一个类只产生一个实例。在Java中，这样的行为可以带来两大好处：

1. 对于频繁使用的对象，可以省略创建对象的时间，这对于那些重量级对象而言，是非常可观的一笔系统开销
2. 由于new操作的次数减少，因而对系统内存的使用频率也会降低，这将减轻GC压力，缩短GC停顿时间

因此对于系统的关键组件和被频繁使用的对象，使用单例模式可以有效地改善系统的性能。单例模式的核心在于通过一个接口返回唯一的的对象实例。一个简单的单例实现如下：

```java
// 代码1 单例模式
public class Singleton {
    private Singleton() {
        System.out.println("Singleton is create"); //创建单例的过程可能会比较慢
    }
    private static Singleton instance = new Singleton();
    public static Singleton getInstance() {
        return instance;
    }
}
```

注意以上的代码，首先单例类必须要有一个private访问级别的构造函数，只有这样，才能确保单例不会在系统中的其他代码处被实例化。其次instance和getInstance()必须是static的。

这种单例模式的实现非常简单，而且十分可靠。唯一不足的仅是无法对instance实现懒加载。假如单例的创建过程很慢，而由于instance成员变量是static定义的，因此JVM在加载单例类时，单例对象就会被创建，如果此时，这个单例类在系统中还扮演其他角色，那么在任何时候使用这个单例类的地方都会初始化这个单例变量，而不管是否会被用到。比如单例类作为String工厂，用于创建一些字符串（该类既用于创建单例，有用于创建tring对象）：

```java
// 代码2
public class Singleton {
    private Singleton() {
        System.out.println("Singleton is create");
    }
    private static Singleton instance = new Singleton();
    public static Singleton getInstance() {
        return instance;
    }
    public static void createString() {
        System.out.println("createString in Singleton");
    }
    
}
```

当使用Singleton.createString()执行任务时，程序输出：

```
Singleton is create
createString in Singleton
```

可以看到，虽然此时并没有使用单例类，但它还是被创建出来。为了解决这个问题，并以此提高系统在相关函数调用时的反应速度，就需要引入懒加载机制。

```java
// 代码3 单例模式的懒加载
public class LazySingleton {
    private LazySingleton() {
        System.out.println("LazySingleton is create");
    }
    private static LazySingleton instance = null;
    public static synchronized LazySinleton getInstance() {
        if(instance == null)
            instance = new LazySinleton();
        return instance;
    }
}
```

首先，对于静态成员变量instance初始值赋予null，确保系统启动时没有额外的负载；其次在getInstance()工厂方法里，判断当前单例是否已经存在，若存在则返回，不存在则创建单例。需要注意的是getInstace()方法必须是同步的，否则在多线程环境下。线程1正新建单例时，完成赋值操作前，线程2判断instance为null，故线程2也将启动新建单例的程序，从而导致多个实例被创建，故需要同步。

使用代码3中的单例实现，虽然实现了懒加载的功能，但是和第一种方法相比，它引入了同步关键字，因此在多线程环境下，它的耗时要远远大于第一种单例模式。

为了使用懒加载而引入的同步关键字反而降低了系统性能，所以还需要继续改进：

```java
// 代码4 静态内部类实现单例模式
public class StaticSingleton {
    private StaticSingleton() {}
    private static class SingletonHolder {
        private static StaticSingleton instance = new StaticSingleton();
    }
    public static StaticSingleton getInstance() {
        return SingletonHolder.instance;
    }
}
```

在这个实现中，单利模式使用内部类来维护单例模式，当StaticSingleton被加载时，其内部类并不会被初始化，故可以确保当StaticSingleton类被载入JVM时，不会初始化单例类，而当getInstance()被调用时，才会加载SingletonHolder，从而初始化instance。同时，**由于实例的建立是在类加载时完成的，故天生对多线程友好**，getInstance()也不需要使用同步关键字。

一般来说，用以上方式实现的单例已经可以确保在系统中只存在唯一实例了。但仍有例外的情况，可能导致系统生成多个实例，比如在代码中，强行使用单例类的私有构造函数，生成多个单例。

> 一个可以被序列化的单例：

```java
// 代码5
public class SerSingleton implements java.io.Serializable {
    String name;
    private SerSingleton() {
        System.out.println("Singleton is create");
        name = "SerSingleton";
    }
    private static SerSingleton instance = new SerSingleton();
    public static SerSingleton getInstace() {
        return instance;
    }
    public static void createString() {
        System.out.println("createString in Singleton");
    }
    private Object readResolve() { //阻止生成新的实例，总是返回当前对象
        return instance;
    }
}
```

测试代码如下：

```java
@Test
public void test() throws Exception {
    SerSIngleton s1 = null;
    SerSingleton s = SerSingleton.getInstance();
    // 先将实例串行化到文件
    FileOutputStream fos = new FileOutputStream("SerSingleton.txt");
    ObjectOutputStream oos = new ObjectOutputStream(fos);
    oos.writeObject(s);
    oos.flush();
    oos.close();
    // 从文件读出原有的单例类
    FileInputStream fis = new FileInputStream("SerSingleton.txt");
    ObjectInputStream ois = new ObjectInputStream(fis);
    s1 = (SerSingleton) ois.readObject();
    Assert.assertEquals(s, s1);
}
```

使用一段测试代码测试单例的序列化化和反序列化，当去掉SerSingleton中的readReslove()时，抛出以下异常：

```java
junit.framework.AssertionFailedError: expected:javatuning.ch2.singleton.serialization.SerSingleton@5224ee
    but was:javatuning.ch2.singleton.serializationSerSingleton@18fe7c3
```

说明测试代码中s和s1指向了不同的实例，在反序列化后，生成多个对象实例。而加上readReslove()函数的，程序正常退出。说明，即便经过反序列化，仍然保持了单例的特征。事实上，在实现了私有的readReslove()后，readObject()已经形同虚设了，它直接使用readReslove()替换了原本的返回值，从而在形式上构造了单例。

> 序列化和反序列化可能会破坏单例。一般来说对单例进行序列化和反序列化的场景并不多见，但是如果存在，就要多加注意。



## 2.代理模式










<hr>
​最后的最后，老婆我爱你。








