# 1.3 单例模式

## 引子

通过new关键字创建一个对象实例是我们在学习Java时的第一课，这背后Java做了哪些操作是我们学习JVM时的第一课。

## 介绍
当使用new创建一个对象实例时，JVM会在**堆区域**分配相应的内存，让这块内存也就是这个实例对象还有引用指向它时它便会一直存在，如果没有引用指向它此时便会对它进行GC（可达性分析）。也就是说我们在不断new一个对象时会带来两个问题：
* 占用堆内存空间。
* GC带来的影响。

很多类在无状态（通过传递参数只做逻辑处理）的情况下重复利用就可以了，并不需要每次都new一个对象，例如在Spring中对于bean对象的创建默认都是singleton单例模式。

单例模式可以按不同维度对其进行分类

* 线程安全维度：线程安全的单例模式、线程不安全的单例模式
* 对象创建时机：饿汉式的单例模式、懒汉式的单例模式

对于线程不安全的单例模式在多线程环境下对象并不是真正的单例，它还是极有可能被创建为“多例”，所以这种形式的单例模式实际当中并不常见，甚至不会有。接下来的单例模式均是基于**线程安全的单例模式**。

所谓的**饿汉式单例模式**，指的是急不可耐的创建一个对象，也就是在类被JVM加载时就会被创建。

```java
/**
 * 饿汉式单例模式
 */
public class Demo {
    private static Demo instance = new Demo();

    private Demo() {}

    public static Demo getInstance() {
        return instance;
    }
}
```

**饱汉式单例模式**指的是不必急着创建对象，而是用到的时候再去创建。

```java
/**
 * 饱汉式单例模式
 */
public class Demo {
    private static Demo instance;

    private Demo() {}

    public static synchronized Demo getInstance() {
        if (instance == null) {
            instance = new Demo();
        }
        return instance;
    }
}
```

可以看到在饱汉式的单例模式中，`getInstance`方法加上了`synchronized`关键字，这确保了对象只会被创建一次。显然在多线程的环境下这会带来锁的争夺，也就是造成性能下降。

## Java源码中的单例模式

Java源码中的单例模式并不常见，Java作为一个基础语言的支撑，将类设置为单例模式显然它是作为一种工具存在。

Java中有一个类在一些特定需求例如性能监控上一定会被用到——Runtime。这个类就是使用了单例模式。

```java
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    public static Runtime getRuntime() {
        return currentRuntime;
    }
    ······
```

对于获取Java应用程序中的一些状态，并不需要创建很多实例，使用单例模式就是很好的解决方案，Java源码在Runtime类上使用的饿汉式单例模式。

## 扩展

前面提到Spring框架中默认产生的bean对象是一个单例，如果想要它是多例可以将bean对象scope属性改为“prototype”表示每次使用这个bean时都会创建一个新的对象实例。另外还有“request”，每一次http request请求都会创建一个新的对象实例。当然还有“session”等。

由于Spring比较复杂，bean对象的创建和初始化过程也会横跨多个类。在这里只摘取出Spring创建单例bean的部分代码，有兴趣可以查看Spring源码。

```java
//AbstractBeanFactory#doGetBean
if (mbd.isSingleton()) {
    sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
        @Override
        public Object getObject() throws BeansException {
            try {
                return createBean(beanName, mbd, args);
            }
            catch (BeansException ex) {
                // Explicitly remove instance from singleton cache: It might have been put there
                // eagerly by the creation process, to allow for circular reference resolution.
                // Also remove any beans that received a temporary reference to the bean.
                destroySingleton(beanName);
                throw ex;
            }
        }
    });
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
}
```


# Tips

在**1.1工厂模式**中提到过**静态工厂方法**和工厂模式的区别，并提到了使用静态工厂的好处，以及它能应用在单例设计模式中。在上面的单例模式中返回对象和私立的方法就是一个静态工厂方法。

<div align=center>
    <img src="../../ad.png"/>
</div>
