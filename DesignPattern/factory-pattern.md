# 工厂模式三兄弟(Factory Pattern)

说明：设计模式系列文章是读`刘伟`所著`《设计模式的艺术之道(软件开发人员内功修炼之道)》`一书的阅读笔记。个人感觉这本书讲的不错，有兴趣推荐读一读。详细内容也可以看看此书作者的博客`https://blog.csdn.net/LoveLion/article/details/17517213`

`工厂模式`是最常用的一类创建型设计模式，通常我们所说的工厂模式是指`工厂方法模式`，它也是使用频率最高的工厂模式。`简单工厂模式`是工厂方法模式的“小弟”，它不属于GoF23种设计模式，但在软件开发中应用也较为频繁，通常将它作为学习其他工厂模式的入门。此外，工厂方法模式还有一位“大哥”——`抽象工厂模式`。这三种工厂模式各具特色，难度也逐个加大，在软件开发中它们都得到了广泛的应用，成为面向对象软件中常用的创建对象的工具。

## 简单工厂模式
简单工厂模式并不属于GoF 23个经典设计模式，但通常将它作为学习其他工厂模式的基础，它的设计思想很简单。

### 模式定义

> `简单工厂模式(Simple Factory Pattern)`：定义一个工厂类，它可以根据参数的不同返回不同类的实例，被创建的实例通常都具有共同的父类。因为在简单工厂模式中用于创建实例的方法是静态(static)方法，因此简单工厂模式又被称为静态工厂方法(Static Factory Method)模式，它属于类创建型模式。

简单工厂模式的要点在于：当你需要什么，只需要传入一个正确的参数，就可以获取你所需要的对象，而无须知道其创建细节。

### 模式结构图

简单工厂模式结构图如下所示：

![简单工厂模式结构图](https://img2020.cnblogs.com/blog/1546632/202006/1546632-20200610222117573-1259462893.png)

### 模式伪代码

在使用`简单工厂模式`时，首先需要对`产品类`进行重构，不能设计一个包罗万象的产品类，而需根据实际情况设计一个产品层次结构，将所有产品类公共的代码移至抽象产品类，并在抽象产品类中声明一些抽象方法，以供不同的具体产品类来实现，典型的抽象产品类代码如下所示：
```java
public abstract class Product {
    // 所有产品的公共属性

    // 所有产品类的公共业务方法
    public void methodSame() {
        //公共方法的实现
    }

    // 声明抽象业务方法
    public abstract void methodDiff();
}
```

在`具体产品类`中实现了抽象产品类中声明的抽象业务方法。
```java
public class ConcreteProduct extends Product {
    @Override
    public void methodDiff() {
        // 具体产品业务方法的实现
    }
}
```  

简单工厂模式的核心是`工厂类`，在没有工厂类之前，客户端一般会使用`new`关键字来直接创建产品对象，而在引入工厂类之后，客户端可以通过工厂类来创建产品，在简单工厂模式中，工厂类提供了一个静态工厂方法供客户端使用，根据所传入的参数不同可以创建不同的产品对象，典型的工厂类代码如下所示:
```java
public class Factory {
    //静态工厂方法
    public static Product getProduct(String arg) {
        Product product = null;
        if (arg.equalsIgnoreCase("A")) {
            product = new ConcreteProductA();
            //初始化设置product
        } else if (arg.equalsIgnoreCase("B")) {
            product = new ConcreteProductB();
            //初始化设置product
        }
        return product;
    }
}
```
在`客户端`代码中，我们通过调用工厂类的工厂方法即可得到产品对象，典型代码如下所示：
```java
public class Client {
    public static void main(String[] args) {
        Product product;
        product = Factory.getProduct("A"); //通过工厂类创建产品对象
        product.methodSame();
        product.methodDiff();
    }
}
```

### 模式简化
有时候，为了简化简单工厂模式，我们可以将`抽象产品类`和`工厂类`合并，将静态工厂方法移至抽象产品类中，如下图所示。

![简化的简单工厂模式](https://img2020.cnblogs.com/blog/1546632/202006/1546632-20200614180924146-772071609.png)

客户端可以通过产品父类的静态工厂方法，根据参数的不同创建不同类型的产品子类对象，这种做法在JDK等类库和框架中也广泛存在。  
比如：`java.nio.charset.Charset`
```java
public abstract class Charset {

    /**
     * Returns a charset object for the named charset.
     */
    public static Charset forName(String charsetName) {
        java.nio.charset.Charset cs = lookup(charsetName);
        if (cs != null)
            return cs;
        throw new UnsupportedCharsetException(charsetName);
    }
}
```

### 模式小结

简单工厂模式提供了专门的工厂类用于创建对象，将对象的创建和对象的使用分离开，它作为一种最简单的工厂模式在软件开发中得到了较为广泛的应用。

使用场景：
1. 工厂类负责创建的对象比较少，由于创建的对象较少，不会造成工厂方法中的业务逻辑太过复杂。
2. 客户端只知道传入工厂类的参数，对于如何创建对象并不关心。

## 工厂方法模式

`简单工厂模式`虽然简单，但存在一个很严重的问题。当系统中需要引入新产品时，由于静态工厂方法通过所传入参数的不同来创建不同的产品，这必定要修改工厂类的源代码，将违背“开闭原则”，如何实现增加新产品而不影响已有代码？工厂方法模式应运而生。

### 模式定义
在`工厂方法模式`中，我们不再提供一个统一的工厂类来创建所有的产品对象，而是针对不同的产品提供不同的工厂，系统提供一个与产品等级结构对应的工厂等级结构。工厂方法模式定义如下：

> `工厂方法模式(Factory Method Pattern)`：定义一个用于创建对象的接口，让子类决定将哪一个类实例化。工厂方法模式让一个类的实例化延迟到其子类。工厂方法模式又简称为工厂模式(Factory Pattern)，又可称作虚拟构造器模式(Virtual Constructor Pattern)或多态工厂模式(Polymorphic Factory Pattern)。工厂方法模式是一种类创建型模式。

### 模式结构图
工厂方法模式提供一个抽象工厂接口来声明抽象工厂方法，而由其子类来具体实现工厂方法，创建具体的产品对象。工厂方法模式结构如图所示:

![工厂方法模式结构图](https://img2020.cnblogs.com/blog/1546632/202006/1546632-20200614183211331-269191986.png)

在工厂方法模式结构图中包含如下几个角色：
- Product（抽象产品）：它是定义产品的接口，是工厂方法模式所创建对象的超类型，也就是产品对象的公共父类。
- ConcreteProduct（具体产品）：它实现了抽象产品接口，某种类型的具体产品由专门的具体工厂创建，具体工厂和具体产品之间一一对应。
- Factory（抽象工厂）：在抽象工厂类中，声明了工厂方法(Factory Method)，用于返回一个产品。抽象工厂是工厂方法模式的核心，所有创建对象的工厂类都必须实现该接口。
- ConcreteFactory（具体工厂）：它是抽象工厂类的子类，实现了抽象工厂中定义的工厂方法，并可由客户端调用，返回一个具体产品类的实例。

### 模式伪代码
与简单工厂模式相比，工厂方法模式最重要的区别是引入了抽象工厂角色，抽象工厂可以是接口，也可以是抽象类或者具体类，其典型代码如下所示：
```java
public interface Factory {
    Product factoryMethod();
}
```
在抽象工厂中声明了工厂方法但并未实现工厂方法，具体产品对象的创建由其子类负责，`客户端针对抽象工厂编程`，可在运行时再指定具体工厂类，具体工厂类实现了工厂方法，不同的具体工厂可以创建不同的具体产品，其典型代码如下所示：
```java
public class ConcreteFactory implements Factory {
    @Override
    public Product factoryMethod() {
        return new ConcreteProduct();
    }
}
```
在客户端代码中，只需关心工厂类即可，不同的具体工厂可以创建不同的产品，典型的客户端类代码片段如下所示:
```java
public class Client {
    public static void main(String[] args) {
        // 确定是哪个工厂可得到产品
        Factory factory = new ConcreteFactory();
        // 获取产品
        Product product = factory.factoryMethod();
    }
}
```
### 模式简化
有时候，为了进一步简化客户端的使用，还可以对客户端隐藏工厂方法，此时，`在工厂类中将直接调用产品类的业务方法，客户端无须调用工厂方法创建产品`，直接通过工厂即可使用所创建的对象中的业务方法。
```java
// 改为抽象类
public class AbstractFactory {
    // 在工厂类中直接调用产品类的业务方法
    public void productMethod() {
        Product product = this.createProduct();
        product.method();
    }

    public abstract Product createProduct();
}
```
通过将业务方法的调用移入工厂类，可以直接使用工厂对象来调用产品对象的业务方法，客户端无须直接调用工厂方法，在客户端并不关心Product细节的情况下使用这种设计方案会更加方便。
### 模式小结

工厂方法模式能够让工厂可以自主确定创建何种产品对象，而如何创建这个对象的细节则完全封装在具体工厂内部，用户只需要关心所需产品对应的工厂，无须关心创建细节，甚至无须知道具体产品类的类名。基于`工厂角色`和`产品角色`的多态性设计是工厂方法模式的关键。

## 抽象工厂模式

`工厂方法模式`通过引入工厂等级结构，解决了简单工厂模式中工厂类职责太重的问题，但由于工厂方法模式中的每个工厂只生产一类产品，可能会导致系统中存在大量的工厂类，势必会增加系统的开销。此时，我们可以考虑将一些相关的产品组成一个`产品族`，由同一个工厂来统一生产，这就是我们本文将要学习的抽象工厂模式的基本思想。

这里我斗胆举个例子来说明一下吧，如果不恰当欢迎指出。

众所周知，国内知名的电器厂有海尔、海信(姑且就认为是2个)，电器厂会生产电视机、电冰箱、空调(姑且就认为是3种产品)。
- 使用工厂方法模式：工厂方法模式中每个工厂只生产一类产品，那么就必须要有`海尔电视机厂`、`海尔电冰箱厂`、`海尔空调厂`、`海信电视机厂`、`海信电冰箱厂`、`海信空调厂`
- 使用抽象工厂模式：抽象工厂中每个工厂生产由多种产品组成的"产品族"，那么就只需要有`海尔工厂`、`海信工厂`就够了，每个工厂可生产自家的电视机、电冰箱、空调。

由此看出使用`抽象工厂模式`极大地减少了系统中类的个数。

### 模式定义

抽象工厂模式为创建一组对象提供了一种解决方案。与工厂方法模式相比，抽象工厂模式中的具体工厂不只是创建一种产品，它负责创建一族产品。抽象工厂模式定义如下：

> 抽象工厂模式(Abstract Factory Pattern)：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式，它是一种对象创建型模式。

### 模式结构图
在抽象工厂模式中，每一个具体工厂都提供了多个工厂方法用于产生多种不同类型的产品，这些产品构成了一个产品族，抽象工厂模式结构如图所示：

![抽象工厂模式结构图](https://img2020.cnblogs.com/blog/1546632/202006/1546632-20200614194007106-1386712790.png)

在抽象工厂模式结构图中包含如下几个角色：
- AbstractFactory（抽象工厂）：它声明了一组用于创建一族产品的方法，每一个方法对应一种产品。
- ConcreteFactory（具体工厂）：它实现了在抽象工厂中声明的创建产品的方法，生成一组具体产品，这些产品构成了一个产品族，每一个产品都位于某个产品等级结构中。
- AbstractProduct（抽象产品）：它为每种产品声明接口，在抽象产品中声明了产品所具有的业务方法。
- ConcreteProduct（具体产品）：它定义具体工厂生产的具体产品对象，实现抽象产品接口中声明的业务方法。

### 模式伪代码

在抽象工厂中声明了多个工厂方法，用于创建不同类型的产品，抽象工厂可以是接口，也可以是抽象类或者具体类，其典型代码如下所示：
```java
public abstract class AbstractFactory {

    public abstract AbstractProductA createProductA();

    public abstract AbstractProductB createProductB();

    public abstract AbstractProductC createProductC();
}
```

具体工厂实现了抽象工厂，每一个具体的工厂方法可以返回一个特定的产品对象，而同一个具体工厂所创建的产品对象构成了一个产品族。对于每一个具体工厂类，其典型代码如下所示：
```java
public class ConcreteFactory1 extends AbstractFactory {
    @Override
    public AbstractProductA createProductA() {
        return new ConcreteProductA1();
    }

    @Override
    public AbstractProductB createProductB() {
        return new ConcreteProductB1();
    }

    @Override
    public AbstractProductC createProductC() {
        return new ConcreteProductC1();
    }
}
```

### 模式小结

如果一开始就学习`抽象工厂模式`估计很难理解为什么这样设计，按次序学习分析`简单工厂模式`、`工厂方法模式`、`抽象工厂模式`基本就顺理成章了。实际开发中，可能并不是照搬照套工厂模式三兄弟的伪代码，大多会简化其中的部分实现。本来学习设计模式就是重思想，学习如何用抽象类、接口、拆分、组合等将软件解耦合，并增强系统可扩展性，这才是最关键的。