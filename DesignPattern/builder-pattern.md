# 建造者模式(Builder Pattern)——复杂对象的组装与创建

说明：设计模式系列文章是读`刘伟`所著`《设计模式的艺术之道(软件开发人员内功修炼之道)》`一书的阅读笔记。个人感觉这本书讲的不错，有兴趣推荐读一读。详细内容也可以看看此书作者的博客`https://blog.csdn.net/LoveLion/article/details/17517213`

## 模式概述

### 模式定义

没有人买车会只买一个轮胎或者方向盘，大家买的都是一辆包含轮胎、方向盘和发动机等多个部件的完整汽车。如何将这些部件组装成一辆完整的汽车并返回给用户，这是`建造者模式`需要解决的问题。建造者模式又称为生成器模式，它是一种较为复杂、使用频率也相对较低的创建型模式。建造者模式为客户端返回的不是一个简单的产品，而是一个由多个部件组成的复杂产品。

它将客户端与包含多个组成部分（或部件）的复杂对象的创建过程分离，客户端无须知道复杂对象的内部组成部分与装配方式，只需要知道所需建造者的类型即可。它关注如何一步一步创建一个的复杂对象，不同的具体建造者定义了不同的创建过程，且具体建造者相互独立，增加新的建造者非常方便，无须修改已有代码，系统具有较好的扩展性。

> `建造者模式(Builder Pattern)`：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。建造者模式是一种对象创建型模式。

建造者模式一步一步创建一个复杂的对象，它允许用户只通过指定复杂对象的类型和内容就可以构建它们，用户不需要知道内部的具体构建细节。

### 模式结构图

建造者模式结构图如下所示：

![建造者模式结构图](https://img2020.cnblogs.com/blog/1546632/202005/1546632-20200531203645693-665857209.png)

建造者模式结构图中包含如下几个角色:
- `Builder（抽象建造者）`：它为创建一个产品Product对象的各个部件指定抽象接口，在该接口中一般声明两类方法，一类方法是buildPartX()，它们用于创建复杂对象的各个部件；另一类方法是getResult()，它们用于返回复杂对象。Builder既可以是抽象类，也可以是接口。

- `ConcreteBuilder（具体建造者）`：它实现了Builder接口，实现各个部件的具体构造和装配方法，定义并明确它所创建的复杂对象，也可以提供一个方法返回创建好的复杂产品对象。

- `Product（产品角色）`：它是被构建的复杂对象，包含多个组成部件，具体建造者创建该产品的内部表示并定义它的装配过程。

- `Director（指挥者）`：指挥者又称为导演类，它负责安排复杂对象的建造次序，指挥者与抽象建造者之间存在关联关系，可以在其construct()建造方法中调用建造者对象的部件构造与装配方法，完成复杂对象的建造。客户端一般只需要与指挥者进行交互，在客户端确定具体建造者的类型，并实例化具体建造者对象（也可以通过配置文件和反射机制），然后通过指挥者类的构造函数或者Setter方法将该对象传入指挥者类中。

在建造者模式的定义中提到了复杂对象，那么什么是`复杂对象`？简单来说，复杂对象是指那些包含多个非简单类型的成员属性，这些成员属性也称为部件或零件，如汽车包括方向盘、发动机、轮胎等部件，电子邮件包括发件人、收件人、主题、内容、附件等部件。

### 模式伪代码

定义`产品角色`，典型代码如下：
```java
public class Product {
    // 定义部件，部件可以是任意类型，包括值类型和引用类型
    private String partA;

    private String partB;

    private String partC;

    // getter、setter方法省略
}
```

在`抽象建造者`中定义了产品的创建方法和返回方法，其典型代码如下
```java
public abstract class Builder {
    // 创建产品对象
    protected Product product = new Product();

    public abstract void buildPartA();

    public abstract void buildPartB();

    public abstract void buildPartC();

    // 返回产品对象
    public Product getResult() {
        return product;
    }
}
```

在抽象类`Builder`中声明了一系列抽象的`buildPartX()`方法用于创建复杂产品的各个部件，具体建造过程在`ConcreteBuilder`中实现，此外还提供了工厂方法`getResult()`，用于返回一个建造好的完整产品。

在`ConcreteBuilder`中实现了`buildPartX()`方法，通过调用`Product`的`setPartX()`方法可以给产品对象的成员属性设值。不同的具体建造者在实现`buildPartX()`方法时将有所区别。

在建造者模式的结构中还引入了一个指挥者类`Director`，该类主要有两个作用：一方面它隔离了客户与创建过程；另一方面它控制产品的创建过程，包括某个`buildPartX()`方法是否被调用以及多个`buildPartX()`方法调用的先后次序等。指挥者针对`抽象建造者`编程，客户端只需要知道具体建造者的类型，即可通过指挥者类调用建造者的相关方法，返回一个完整的产品对象。在实际生活中也存在类似指挥者一样的角色，如一个客户去购买电脑，电脑销售人员相当于指挥者，只要客户确定电脑的类型，电脑销售人员可以通知电脑组装人员给客户组装一台电脑。指挥者类的代码示例如下：
```java
public class Director {

    private Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    public void setBuilder(Builder builder) {
        this.builder = builer;
    }

    //产品构建与组装方法
    public Product construct() {

        builder.buildPartA();
        builder.buildPartB();
        builder.buildPartC();

        return builder.getResult();
    }
}
```

在指挥者类中可以注入一个`抽象建造者`类型的对象，其核心在于提供了一个建造方法`construct()`，在该方法中调用了builder对象的构造部件的方法，最后返回一个产品对象。

对于客户端而言，只需关心具体的建造者即可，代码片段如下所示：
```java
public static void main(String[] args) {
    Builder builder = new ConcreteBuilder();

    Director director = new Director(builder);

    Product product = director.construct();
}
```

### 模式简化
在有些情况下，为了简化系统结构，可以将`Director`和抽象建造者`Builder`进行合并，在`Builder`中提供逐步构建复杂产品对象的`construct()`方法。
```java
public abstract class Builder {
    // 创建产品对象
    protected Product product = new Product();

    public abstract void buildPartA();

    public abstract void buildPartB();

    public abstract void buildPartC();

    // 返回产品对象
    public Product construct() {
        buildPartA();
        buildPartB();
        buildPartC();
        return product;
    }
}
```
## 模式应用

### 模式在JDK中的应用
`java.util.stream.Stream.Builder`
```java
public interface Builder<T> extends Consumer<T> {
    /**
     * Adds an element to the stream being built.
     */
    default Builder<T> add(T t) {
        accept(t);
        return this;
    }

    /**
     * Builds the stream, transitioning this builder to the built state
     */
    Stream<T> build();
}
```
### 模式在开源项目中的应用

看下`Spring`是如何构建`org.springframework.web.servlet.mvc.method.RequestMappingInfo`
```java
/**
 * Defines a builder for creating a RequestMappingInfo.
 * @since 4.2
 */
public interface Builder {
  /**
   * Set the path patterns.
   */
  Builder paths(String... paths);

  /**
   * Set the request method conditions.
   */
  Builder methods(RequestMethod... methods);

  /**
   * Set the request param conditions.
   */
  Builder params(String... params);

  /**
   * Set the header conditions.
   * <p>By default this is not set.
   */
  Builder headers(String... headers);

  /**
   * Build the RequestMappingInfo.
   */
  RequestMappingInfo build();
}
```

`Builder`接口的默认实现，如下：
```java
private static class DefaultBuilder implements Builder {

  private String[] paths = new String[0];

  private RequestMethod[] methods = new RequestMethod[0];

  private String[] params = new String[0];

  private String[] headers = new String[0];

  public DefaultBuilder(String... paths) {
    this.paths = paths;
  }

  @Override
  public Builder paths(String... paths) {
    this.paths = paths;
    return this;
  }

  @Override
  public DefaultBuilder methods(RequestMethod... methods) {
    this.methods = methods;
    return this;
  }

  @Override
  public DefaultBuilder params(String... params) {
    this.params = params;
    return this;
  }

  @Override
  public DefaultBuilder headers(String... headers) {
    this.headers = headers;
    return this;
  }
  
  @Override
  public RequestMappingInfo build() {
    ContentNegotiationManager manager = this.options.getContentNegotiationManager();

    PatternsRequestCondition patternsCondition = new PatternsRequestCondition(
        this.paths, this.options.getUrlPathHelper(), this.options.getPathMatcher(),
        this.options.useSuffixPatternMatch(), this.options.useTrailingSlashMatch(),
        this.options.getFileExtensions());

    return new RequestMappingInfo(this.mappingName, patternsCondition,
        new RequestMethodsRequestCondition(this.methods),
        new ParamsRequestCondition(this.params),
        new HeadersRequestCondition(this.headers),
        new ConsumesRequestCondition(this.consumes, this.headers),
        new ProducesRequestCondition(this.produces, this.headers, manager),
        this.customCondition);
  }
}
```
`Spring`框架中许多`构建类`的实例化使用了类似上面方式，总结有以下特点：
1. `Builder`大多是`构建类`的内部类，`构建类`提供了一个静态创建`Builder`的方法
2. `Builder`返回`构建类的实例`，大多通过`build()`方法
3. 构建过程有`大量参数`，除了几个必要参数，用户可根据自己所需选择设置其他参数实例化对象
## 模式总结

建造者模式的核心在于如何一步步构建一个包含多个组成部件的完整对象，使用相同的构建过程构建不同的产品，在软件开发中，如果我们需要创建复杂对象并希望系统具备很好的灵活性和可扩展性可以考虑使用建造者模式。

`建造者模式`与`抽象工厂模式`有点相似，但是建造者模式返回一个完整的复杂产品，而抽象工厂模式返回一系列相关的产品；在抽象工厂模式中，客户端通过选择具体工厂来生成所需对象，而在建造者模式中，客户端通过指定具体建造者类型并指导Director类如何去生成对象，侧重于一步步构造一个复杂对象，然后将结果返回。**如果将抽象工厂模式看成一个汽车配件生产厂，生成不同类型的汽车配件，那么建造者模式就是一个汽车组装厂，通过对配件进行组装返回一辆完整的汽车。**

### 主要优点

1. 将产品本身与产品的创建过程解耦，使得相同的创建过程可以创建不同的产品对象
2. 可以更加精细地控制产品的创建过程。将复杂产品的创建步骤分解在不同的方法中，使得创建过程更加清晰，也更方便使用程序来控制创建过程。

### 适用场景

(1) 需要生成的产品对象有复杂的内部结构，这些产品对象通常包含多个成员属性。

(2) 需要生成的产品对象的属性相互依赖，需要指定其生成顺序。

(3) 隔离复杂对象的创建和使用，并使得相同的创建过程可以创建不同的产品。