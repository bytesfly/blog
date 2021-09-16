# 单例模式(Singleton Pattern)——确保对象的唯一性


说明：设计模式系列文章是读`刘伟`所著`《设计模式的艺术之道(软件开发人员内功修炼之道)》`一书的阅读笔记。个人感觉这本书讲的不错，有兴趣推荐读一读。详细内容也可以看看此书作者的博客`https://blog.csdn.net/LoveLion/article/details/17517213`。

## 模式概述

### 模式定义

实际开发中，我们会遇到这样的情况，为了节约系统资源或者数据的一致性(比如说全局的`Config`、携带上下文信息的`Context`等等)，有时需要确保系统中某个类只有唯一一个实例，当这个唯一实例创建成功之后，我们无法再创建一个同类型的其他对象，所有的操作都只能基于这个唯一实例。为了确保对象的唯一性，我们可以通过单例模式来实现，这就是单例模式的动机所在。

> 单例模式(`Singleton Pattern`): 确保某一个类只有一个实例，而且自己实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。单例模式是一种对象创建型模式。

单例模式有三个要点：
1. 某个类只能有一个实例
2. 它必须自行创建这个实例
3. 它必须自行向整个系统提供这个实例

### 模式结构图

单例模式是结构最简单的设计模式一，在它的核心结构中只包含一个被称为单例类的特殊类。单例模式结构图如下所示：

![单例模式结构图](https://img2020.cnblogs.com/blog/1546632/202005/1546632-20200523104505352-451913605.png)

单例模式结构图中只包含一个单例角色：
- `Singleton（单例）`：在单例类的内部实现只生成一个实例，同时它提供一个静态的getInstance()工厂方法，让客户可以访问它的唯一实例；为了防止在外部对其实例化，将其构造函数设计为私有；在单例类内部定义了一个Singleton类型的静态对象，作为外部共享的唯一实例。

### 饿汉式单例与懒汉式单例

#### 饿汉式单例

`饿汉式单例`类是实现起来最简单的单例类。由于在定义静态变量的时候实例化单例类，因此在类加载的时候就已经创建了单例对象，典型代码如下：

```java
public class EagerSingleton { 
    private static final EagerSingleton instance = new EagerSingleton(); 
    private EagerSingleton() { } 
 
    public static EagerSingleton getInstance() {
        return instance; 
    }   
}
```

#### 懒汉式单例

`懒汉式单例`在第一次调用`getInstance()`方法时实例化，在类加载时并不自行实例化，这种技术又称为延迟加载(`Lazy Load`)或者懒加载技术，即需要的时候再加载实例，为避免`多线程`环境下同时调用`getInstance()`方法从而生成多个实例，需要确保线程安全，相应实现也就有多种方式。

`第一种`方法可以使用关键字`synchronized`，代码实现如下：
```java
public class LazySingleton { 
    private static LazySingleton instance = null; 
 
    private LazySingleton() { } 
 
    public synchronized static LazySingleton getInstance() { 
        if (instance == null) {
            instance = new LazySingleton(); 
        }
        return instance; 
    }
}
```
在`getInstance()`方法前面增加了关键字`synchronized`进行同步，以处理多线程同时访问的安全问题。我们知道使用`synchronized`关键字最好是在离共享资源最近的位置加锁，这样同步带来的性能影响会减小。所以`让人感觉`上面的实现可以优化为如下代码：

```java
public static LazySingleton getInstance() { 
    if (instance == null) {
        synchronized (LazySingleton.class) {
            instance = new LazySingleton(); 
        }
    }
    return instance; 
}
```
问题貌似得以解决，事实并非如此。如果使用以上代码来实现单例，还是会存在单例对象不唯一。原因如下：  
假如在某一瞬间`线程A`和`线程B`都在调用`getInstance()`方法，此时`instance`对象为`null`值，均能通过`instance == null`的判断。由于实现了`synchronized`加锁机制，`线程A`进入`synchronized`修饰的代码块中执行实例创建代码，`线程B`处于排队等待状态，必须等待`线程A`执行完毕后才可以进入`synchronized`修饰的代码块。但当`A`执行完毕时，`线程B`并不知道实例已经创建，将继续创建新的实例，导致产生多个单例对象，违背单例模式的设计思想，因此需要进行进一步改进，在`synchronized`中再进行一次`(instance == null)`判断，这种方式称为`双重检查锁定(Double-Check Locking)`。使用双重检查锁定实现的懒汉式单例类典型代码如下所示：

```java
public class LazySingleton { 
    private volatile static LazySingleton instance = null; 
 
    private LazySingleton() { } 
 
    public static LazySingleton getInstance() { 
        // 第一重判断
        if (instance == null) {
            // 使用synchronized关键字加锁
            synchronized (LazySingleton.class) {
                //第二重判断
                if (instance == null) {
                    instance = new LazySingleton(); //创建单例实例
                }
            }
        }
        return instance; 
    }
}
```
需要注意的是，如果使用`双重检查锁定`来实现懒汉式单例类，最好在静态成员变量`instance`之前增加修饰符`volatile`，被`volatile`修饰的变量可以保证多线程环境下的可见性以及禁止指令重排序。由于`volatile`关键字会屏蔽Java虚拟机所做的一些优化，可能对执行效率稍微有些影响，因此使用双重检查锁定来实现单例模式也不一定是最完美的实现方式。

如果是`java`语言的程序，还可以使用`静态内部类`的方式实现。代码如下：
```java
public class Singleton {
    private Singleton() {
    }

    private static class HolderClass {
        final static Singleton instance = new Singleton();
    }

    public static Singleton getInstance() {
        return HolderClass.instance;
    }
}
```
由于静态单例对象没有作为`Singleton`的成员变量直接实例化，因此`类加载`时不会实例化`Singleton`，第一次调用`getInstance()`时将加载内部类`HolderClass`，在该内部类中定义了一个`static`类型的变量`instance`，此时会首先初始化这个变量，由`Java虚拟机`来保证其线程安全性，确保该成员变量只初始化一次。

## 模式应用

### 模式在JDK中的应用
在JDK中，`java.lang.Runtime`使用了`饿汉式单例`，如下：
```java
public class Runtime {
    private static Runtime currentRuntime = new Runtime();
    
    public static Runtime getRuntime() {
        return currentRuntime;
    }

    /** Don't let anyone else instantiate this class */
    private Runtime() {}
}
```

### 模式在开源项目中的应用

`Spring`框架中许多地方使用了`单例模式`，这里随便举个例子，如`org.springframework.aop.framework.ProxyFactoryBean`中的部分代码如下：
```java
/**
 * Return the singleton instance of this class's proxy object,
 * lazily creating it if it hasn't been created already.
 * @return the shared singleton proxy
 */
private synchronized Object getSingletonInstance() {
  if (this.singletonInstance == null) {
    this.targetSource = freshTargetSource();
    if (this.autodetectInterfaces && getProxiedInterfaces().length == 0 && !isProxyTargetClass()) {
      // Rely on AOP infrastructure to tell us what interfaces to proxy.
      Class<?> targetClass = getTargetClass();
      if (targetClass == null) {
        throw new FactoryBeanNotInitializedException("Cannot determine target class for proxy");
      }
      setInterfaces(ClassUtils.getAllInterfacesForClass(targetClass, this.proxyClassLoader));
    }
    // Initialize the shared singleton instance.
    super.setFrozen(this.freezeProxy);
    this.singletonInstance = getProxy(createAopProxy());
  }
  return this.singletonInstance;
}
```

## 模式总结

单例模式作为一种目标明确、结构简单、理解容易的设计模式，在软件开发中使用频率相当高，在很多应用软件和框架中都得以广泛应用。

### 主要优点

(1) 单例模式提供了对唯一实例的受控访问。因为单例类封装了它的唯一实例，所以它可以严格控制客户怎样以及何时访问它。

(2) 由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象单例模式无疑可以提高系统的性能。

(3) 允许可变数目的实例。基于单例模式我们可以进行扩展，使用与单例控制相似的方法来获得指定个数的对象实例，既节省系统资源，又解决了单例单例对象共享过多有损性能的问题。

### 适用场景

在以下情况下可以考虑使用单例模式：

(1) 系统只需要一个实例对象，如系统要求提供一个唯一的序列号生成器或资源管理器，或者需要考虑资源消耗太大而只允许创建一个对象。

(2) 客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例。
