# 适配器模式(Adapter Pattern)——不兼容结构的协调


说明：设计模式系列文章是读`刘伟`所著`《设计模式的艺术之道(软件开发人员内功修炼之道)》`一书的阅读笔记。个人感觉这本书讲的不错，有兴趣推荐读一读。详细内容也可以看看此书作者的博客`https://blog.csdn.net/LoveLion/article/details/17517213`。

## 模式概述

### 模式定义

与电源适配器相似，在适配器模式中引入了一个被称为适配器(`Adapter`)的包装类，而它所包装的对象称为适配者(`Adaptee`)，即被适配的类。适配器的实现就是把客户类的请求转化为对适配者的相应接口的调用。也就是说：当客户类调用适配器的方法时，在适配器类的内部将调用适配者类的方法，而这个过程对客户类是透明的，客户类并不直接访问适配者类。因此，适配器让那些由于接口不兼容而不能交互的类可以一起工作。

> 适配器模式(`Adapter Pattern`): 将一个接口转换成期望的另一个接口，使接口不兼容的那些类可以一起工作，其别名为包装器(`Wrapper`)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

*注意：在适配器模式定义中所提及的接口是指广义的接口，它可以表示一个方法或者方法的集合。*

### 模式结构图

在适配器模式中，我们通过增加一个新的适配器类来解决接口不兼容的问题，使得原本没有任何关系的类可以协同工作。根据适配器类与适配者类的关系不同，适配器模式可分为`对象适配器`和`类适配器`两种，在`对象适配器`模式中，适配器与适配者之间是`关联关系`；在`类适配器`模式中，适配器与适配者之间是`继承`关系。在实际开发中，对象适配器的使用频率更高，对象适配器模式结构如图所示

![对象适配器结构图](https://img2020.cnblogs.com/blog/1546632/202005/1546632-20200517093507246-1923277557.png)

在对象适配器模式结构图中包含如下几个角色：
- `Target（目标抽象类）`：目标抽象类定义客户所需接口，可以是一个抽象类或接口，也可以是具体类。
- `Adapter（适配器类）`：适配器可以调用另一个接口，作为一个转换器，对`Adaptee`和`Target`进行适配，适配器类是适配器模式的核心，在对象适配器中，它通过继承(或者实现)`Target`并关联一个`Adaptee`对象使二者产生联系。
- `Adaptee（适配者类）`：适配者即被适配的角色，它定义了一个已经存在的接口，这个接口需要适配，适配者类一般是一个具体类，包含了客户希望使用的业务方法，在某些情况下可能没有适配者类的源代码。

### 模式伪代码

在`对象适配器`中，客户端需要调用`request()`方法，而适配者类`Adaptee`没有该方法，但是它所提供的`specificRequest()`方法却是客户端所需要的。为了使客户端能够使用适配者类，需要提供一个包装类`Adapter`，即适配器类。这个包装类包装了一个适配者的实例，从而将客户端与适配者衔接起来，在适配器的`request()`方法中调用适配者的`specificRequest()`方法。因为适配器类与适配者类是`关联关系`（也可称之为委派关系），所以这种适配器模式称为`对象适配器模式`。典型的对象适配器代码如下所示：
```java
public class Adapter implements Target {
    // 维持一个对适配者对象的引用
    private Adaptee adaptee;

    // 构造注入适配者
    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
        // 转发调用
        adaptee.specificRequest();
    }
}
```

### 类适配器，双向适配器，缺省适配器

#### 类适配器
`类适配器`模式和`对象适配器`模式最大的区别在于适配器和适配者之间的关系不同，对象适配器模式中适配器和适配者之间是`关联关系`，而类适配器模式中适配器和适配者是`继承关系`

![类适配器结构图](https://img2020.cnblogs.com/blog/1546632/202005/1546632-20200517095252755-1268131042.png)

适配器类实现了抽象目标类接口`Target`，并继承了适配者类，在适配器类的`request()`方法中调用所继承的适配者类的`specificRequest()`方法，实现了适配。  
典型代码实现如下：
```java
public class Adapter extends Adaptee implements Target {
    @Override
    public void request() {
        specificRequest();
    }
}
```
由于`Java`、`C#`等语言不支持多重类继承，因此`类适配器`的使用受到很多限制，例如如果目标抽象类`Target`不是接口，而是一个类，就无法使用类适配器；此外，如果适配者`Adaptee`为最终(`final`)类，也无法使用类适配器。在Java等面向对象编程语言中，大部分情况下我们使用的是对象适配器，`类适配器`较少使用。

#### 双向适配器

`双向适配器`: 在`对象适配器`的使用过程中，如果在适配器中同时包含对目标类和适配者类的引用，适配者可以通过它调用目标类中的方法，目标类也可以通过它调用适配者类中的方法，那么该适配器就是一个双向适配器。

![双向适配器结构图](https://img2020.cnblogs.com/blog/1546632/202005/1546632-20200517100001199-2025299022.png)

典型代码实现如下：
```java
public class Adapter implements Target, Adaptee {
    //同时维持对抽象目标类和适配者的引用
    private Target target;
    private Adaptee adaptee;

    public Adapter(Target target) {
        this.target = target;
    }

    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
        adaptee.specificRequest();
    }

    @Override
    public void specificRequest() {
        target.request();
    }
}
```
在实际开发中，我们很少使用`双向适配器`。违背了`单一职责`原则，相当于一个适配器承担了两个适配器的职责。

#### 缺省适配器

`缺省适配器`模式是适配器模式的一种变体，其应用也较为广泛。
> `缺省适配器模式(Default Adapter Pattern)`：当不需要实现一个接口所提供的所有方法时，可先设计一个抽象类实现该接口，并为接口中每个方法提供一个默认实现（空方法），那么该抽象类的子类可以选择性地覆盖父类的某些方法来实现需求，它适用于不想使用一个接口中的所有方法的情况，又称为`单接口适配器模式`。

![缺省适配器结构图](https://img2020.cnblogs.com/blog/1546632/202005/1546632-20200517101208598-111312955.png)

典型代码实现如下：
```java
public abstract class Adapter implements Target {
    @Override
    public void request1() {
        // 空实现，让具体实现类去有选择地实现
    }

    @Override
    public void request2() {
        // 空实现，让具体实现类去有选择地实现
    }

    @Override
    public void request3() {
        // 空实现，让具体实现类去有选择地实现
    }
}

public class ConcreteAdapter extends Adapter {
    // 维持一个对适配者对象的引用
    private Adaptee adaptee;

    // 构造注入适配者
    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request1() {
        // 只实现request1
        adaptee.specificRequest();
    }
}
```

## 模式应用

### 模式在JDK中的应用
在JDK中，`IO`类中也大量使用到了适配器模式。比如说`StringReader`将`String`适配到`Reader`，`InputStreamReader`将`InputStream`适配到`Reader`等等。

这里用`StringReader`来说明。这里的`StringReader`相当于上述的`Adapter`，`Reader`相当于上述的`Target`，`String`相当于上述的`Adaptee`
```java
public class StringReader extends Reader {

   // 维持对adaptee对象的引用
    private String str;
    
    private int length;
    private int next = 0;
    private int mark = 0;

    /**
     * 构造注入一个String用于之后的read操作
     */
    public StringReader(String s) {
        this.str = s;
        this.length = s.length();
    }
  
  // 这里相当于是在做适配操作，转为目标对象所期望的请求
    public int read() throws IOException {
        synchronized (lock) {
            ensureOpen();
            if (next >= length)
                return -1;
            return str.charAt(next++);
        }
    }
}
```

### 模式在开源项目中的应用

其实不只是开源项目，我们自己写的项目很多地方都是隐含着适配器模式，只是有时候这种特性表现的不是很明显(因为我们很自然去使用)，以至于我们都没有给类名命成`XxxAdapter`，比如说我们使用第三方库，第三方库某方法名太长或者参数过多，或者调用过于复杂了，我们可能会对第三方库再次做个封装，把适合自己项目当前业务逻辑的默认参数，默认实现补充完整，让其他地方很方便调用，举个具体例子，项目中可能经常会用到`HttpClient`，大多数情况下，对现有的`HttpClient`再次封装(比如client的创建、http响应结果的统一处理等等)，封装成方便自己项目使用的`SpecialHttpClient`，如果你还想切换不同的底层`HttpClient`实现，还可以对`SpecialHttpClient`抽出来一个接口，通过不同的`Adapter`来注入不同的`HttpClient`(比如apache的`HttpClient`、`OkHttpClient`、Spring的`RestTemplate`以及`WebClient`等等)来实现，这种很自然的思想 个人觉得本质上也用到了`适配器模式`，相当于是把第三方的`HttpClient`适配成了自己的`SpecialHttpClient`。

当转换的源不是单一的时候，这种`适配器`思想就凸显出来了(对应上面的例子就是说 项目中需要同时用到apache的`HttpClient`、Spring的`RestTemplate`以及`WebClient`等)。

这里举个`Spring`中的例子。在`Spring`的`AOP`中，由于`Advisor`需要的是`MethodInterceptor`对象，所以每一个`Advisor`中的`Advice`都要适配成对应的`MethodInterceptor`对象
```java
public interface AdvisorAdapter {

	boolean supportsAdvice(Advice advice);
  
	MethodInterceptor getInterceptor(Advisor advisor);
}


class ThrowsAdviceAdapter implements AdvisorAdapter, Serializable{

	@Override
	public boolean supportsAdvice(Advice advice) {
		return (advice instanceof ThrowsAdvice);
	}

	@Override
	public MethodInterceptor getInterceptor(Advisor advisor) {
		return new ThrowsAdviceInterceptor(advisor.getAdvice());
	}

}
```

## 模式总结

适配器模式将现有接口转化为客户类所期望的接口，实现了对现有类的复用，它是一种使用频率非常高的设计模式，在软件开发中得以广泛应用。

### 主要优点

无论是对象适配器模式还是类适配器模式都具有如下优点：

(1) 将目标类和适配者类解耦，通过引入一个适配器类来重用现有的适配者类，无须修改原有结构，提高了扩展性，符合“开闭原则”  
(2) 增加了类的透明性和复用性，将具体的业务实现过程封装在适配者类中，对于客户端类而言是透明的，而且提高了适配者的复用性，同一个适配者类可以在多个不同的系统中复用。

具体来说，`类适配器`模式还有如下优点：

由于适配器类是适配者类的子类，因此可以在适配器类中置换一些适配者的方法，使得适配器的灵活性更强。

`对象适配器`模式还有如下优点：

(1) 一个对象适配器可以把多个不同的适配者适配到同一个目标；

(2) 可以适配一个适配者的子类，由于适配器和适配者之间是关联关系，根据“里氏代换原则”，适配者的子类也可通过该适配器进行适配。

### 主要缺点

`类适配器`模式的缺点如下：

(1) 对于Java、C#等不支持多重类继承的语言，一次最多只能适配一个适配者类，不能同时适配多个适配者；

(2) 适配者类不能为最终类，如在Java中不能为final类，C#中不能为sealed类；

(3) 在Java、C#等语言中，类适配器模式中的目标抽象类只能为接口，不能为类，其使用有一定的局限性。

对象适配器模式的缺点如下：

与类适配器模式相比，要在适配器中置换适配者类的某些方法比较麻烦(比如说适配者类中的某些方法是`protected`,而我们做适配的时候刚好需要用到)。如果一定要置换掉适配者类的一个或多个方法，可以先做一个适配者类的子类，将适配者类的方法置换掉，然后再把适配者类的子类当做真正的适配者进行适配，实现过程较为复杂。

### 适用场景

在以下情况下可以考虑使用适配器模式：

系统需要使用(复用)一些现有的类，而这些类的接口（如方法名）不符合系统的需要，甚至没有这些类的源代码等等，可使用适配器模式协调诸多不兼容结构的场景。
