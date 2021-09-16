# 组合模式(Composite Pattern)——树形结构的处理

说明：设计模式系列文章是读`刘伟`所著`《设计模式的艺术之道(软件开发人员内功修炼之道)》`一书的阅读笔记。个人感觉这本书讲的不错，有兴趣推荐读一读。详细内容也可以看看此书作者的博客`https://blog.csdn.net/LoveLion/article/details/17517213`

## 模式概述

`树形结构`在软件中随处可见，例如操作系统中的目录结构、应用软件中的菜单、办公系统中的公司组织结构等等，如何运用面向对象的方式来处理这种树形结构是`组合模式`需要解决的问题。组合模式通过一种巧妙的设计方案使得用户可以一致性地处理整个树形结构或者树形结构的一部分，也可以一致性地处理树形结构中的叶子节点（不包含子节点的节点）和容器节点（包含子节点的节点）。

### 模式定义

> `组合模式(Composite Pattern)`：组合多个对象形成树形结构以表示具有“整体—部分”关系的层次结构。组合模式对单个对象（即叶子对象）和组合对象（即容器对象）的使用具有一致性，组合模式又可以称为“整体—部分”(Part-Whole)模式，它是一种对象结构型模式。

### 模式结构图

组合模式结构图如下所示：

![组合模式结构图](https://img2020.cnblogs.com/blog/1546632/202006/1546632-20200620095658054-568368664.png)

在组合模式结构图中包含如下几个角色：

- `Component`（抽象构件）：它可以是接口或抽象类，为叶子构件和容器构件对象声明接口，在该角色中可以包含所有子类共有行为的声明和实现。在抽象构件中定义了访问及管理它的子构件的方法，如增加子构件、删除子构件、获取子构件等。

- `Leaf`（叶子构件）：它在组合结构中表示叶子节点对象，叶子节点没有子节点，它实现了在抽象构件中定义的行为。对于那些访问及管理子构件的方法，可以通过异常等方式进行处理。

- `Composite`（容器构件）：它在组合结构中表示容器节点对象，容器节点包含子节点，其子节点可以是叶子节点，也可以是容器节点，它提供一个集合用于存储子节点，实现了在抽象构件中定义的行为，包括那些访问及管理子构件的方法，在其业务方法中可以递归调用其子节点的业务方法。

> 组合模式的关键是定义了一个`抽象构件类`，它既可以代表叶子，又可以代表容器，而客户端针对该抽象构件类进行编程，无须知道它到底表示的是叶子还是容器，可以对其进行统一处理。同时容器对象与抽象构件类之间还建立一个聚合关联关系，在容器对象中既可以包含叶子，也可以包含容器，以此实现递归组合，形成一个树形结构。

### 模式伪代码

对于客户端而言，一般针对抽象构件编程，而无须关心其具体子类是容器构件还是叶子构件。抽象构建类典型代码如下：
```java
public abstract class Component {
    public abstract void add(Component c); //增加成员

    public abstract void remove(Component c); //删除成员

    public abstract Component getChild(int i); //获取成员

    public abstract void operation();  //业务方法
}
```

如果继承抽象构件的是叶子构件，则其典型代码如下所示：

```java
public class Leaf extends Component {
    @Override
    public void add(Component c) {
        //异常处理或错误提示 
    }

    @Override
    public void remove(Component c) {
        //异常处理或错误提示 
    }

    @Override
    public Component getChild(int i) {
        //异常处理或错误提示 
        return null;
    }

    @Override
    public void operation() {
        //叶子构件具体业务方法的实现
    }
}
```

如果继承抽象构件的是容器构件，则其典型代码如下所示:
```java
public class Composite extends Component {

    private List<Component> list = new ArrayList<>();

    @Override
    public void add(Component c) {
        list.add(c);
    }

    @Override
    public void remove(Component c) {
        list.remove(c);
    }

    @Override
    public Component getChild(int i) {
        return (Component) list.get(i);
    }

    @Override
    public void operation() {
        //容器构件具体业务方法的实现
        //递归调用成员构件的业务方法
        for (Object obj : list) {
            ((Component) obj).operation();
        }
    }
}
```

客户端对抽象构件类进行编程
```java
public class Client {
    public static void main(String[] args) {
        Component component;
        component = new Leaf();
        //component = new Composite();

        // 无须知道到底是叶子还是容器
        // 可以对其进行统一处理
        component.operation();
    }
}
```

## 模式简化

### 透明组合模式

> 透明组合模式中，抽象构件Component中声明了所有用于管理成员对象的方法，包括add()、remove()以及getChild()等方法，这样做的好处是确保所有的构件类都有相同的接口。在客户端看来，叶子对象与容器对象所提供的方法是一致的，客户端可以相同地对待所有的对象。透明组合模式也是组合模式的标准形式。

透明组合模式的完整结构图如下：

![透明组合模式结构图](https://img2020.cnblogs.com/blog/1546632/202006/1546632-20200620104846072-1966986837.png)

也可以将叶子构件的`add()`、`remove()`等方法的实现代码移至`Component`中，由`Component`提供统一的默认实现，这样子类就不必强制去实现管理子Component。代码如下所示：
```java
public abstract class Component {
    public void add(Component c) {
        throw new RuntimeException("不支持的操作");
    }

    public void remove(Component c) {
        throw new RuntimeException("不支持的操作");
    }

    public Component getChild(int i) {
        throw new RuntimeException("不支持的操作");
    }

    public abstract void operation();  //业务方法
}
```

`透明组合模式`的缺点是不够安全，因为叶子对象和容器对象在本质上是有区别的。叶子对象不可能有下一个层次的对象，即不可能包含成员对象，因此为其提供add()、remove()以及getChild()等方法是没有意义的，这在编译阶段不会出错，但在运行阶段如果调用这些方法可能会出错（如果没有提供相应的错误处理代码）。

### 安全组合模式

> 安全组合模式中，在抽象构件Component中没有声明任何用于管理成员对象的方法，而是在Composite类中声明并实现这些方法。


安全组合模式的完整结构图如下：

![安全组合模式结构图](https://img2020.cnblogs.com/blog/1546632/202006/1546632-20200620105205351-475574714.png)

此时`Component`就应该这样定义了
```java
public abstract class Component {
    // 业务方法
    public abstract void operation();
}
```

`安全组合模式`的缺点是不够透明，因为叶子构件和容器构件具有不同的方法，且容器构件中那些用于管理成员对象的方法没有在抽象构件类中定义，因此客户端不能完全针对抽象编程，必须有区别地对待叶子构件和容器构件。在实际应用中，安全组合模式的使用频率也非常高，在`Java AWT中`使用的组合模式就是安全组合模式。

## 模式应用

### 模式在JDK中的应用

`Java SE`中的`AWT`和`Swing`包的设计就基于组合模式，在这些界面包中为用户提供了大量的容器构件（如`Container`）和成员构件（如`Checkbox`、`Button`和`TextComponent`等），其结构如下图所示

![AWT组合模式结构图](https://img2020.cnblogs.com/blog/1546632/202006/1546632-20200620111836722-1427412672.png)

`Component`类是抽象构件，`Checkbox`、`Button`和`TextComponent`是叶子构件，而`Container`是容器构件，在`AWT`中包含的叶子构件还有很多。在一个容器构件中可以包含叶子构件，也可以继续包含容器构件，这些叶子构件和容器构件一起组成了复杂的`GUI`界面。除此以外，在`XML解析`、`组织结构树处理`、`文件系统设计`等领域，组合模式都得到了广泛应用。

### 模式在开源项目中的应用

`Spring`中`org.springframework.web.method.support.HandlerMethodArgumentResolver`使用了安全组合模式。提取关键代码如下：
```java
public interface HandlerMethodArgumentResolver {
    
    boolean supportsParameter(MethodParameter parameter);

    Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                           NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception;

}
```
再看下它的一个实现类`org.springframework.web.method.support.HandlerMethodArgumentResolverComposite`
```java
public class HandlerMethodArgumentResolverComposite implements HandlerMethodArgumentResolver {


    private final List<HandlerMethodArgumentResolver> argumentResolvers = new LinkedList<>();

    /**
     * Add the given {@link HandlerMethodArgumentResolver}.
     */
    public HandlerMethodArgumentResolverComposite addResolver(HandlerMethodArgumentResolver resolver) {
        this.argumentResolvers.add(resolver);
        return this;
    }

    /**
     * Add the given {@link HandlerMethodArgumentResolver HandlerMethodArgumentResolvers}.
     */
    public HandlerMethodArgumentResolverComposite addResolvers(
            @Nullable HandlerMethodArgumentResolver... resolvers) {

        if (resolvers != null) {
            Collections.addAll(this.argumentResolvers, resolvers);
        }
        return this;
    }

    /**
     * Clear the list of configured resolvers.
     */
    public void clear() {
        this.argumentResolvers.clear();
    }


    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return getArgumentResolver(parameter) != null;
    }


    @Override
    public Object resolveArgument(MethodParameter parameter, @Nullable ModelAndViewContainer mavContainer,
                                  NativeWebRequest webRequest, @Nullable WebDataBinderFactory binderFactory) throws Exception {

        HandlerMethodArgumentResolver resolver = getArgumentResolver(parameter);
        if (resolver == null) {
            throw new IllegalArgumentException("Unsupported parameter type [" +
                    parameter.getParameterType().getName() + "]. supportsParameter should be called first.");
        }
        return resolver.resolveArgument(parameter, mavContainer, webRequest, binderFactory);
    }
}
```

## 模式总结

### 主要优点

1. 组合模式可以清楚地定义分层次的复杂对象，表示对象的全部或部分层次，它让客户端忽略了层次的差异，方便对整个层次结构进行控制。

2. 客户端可以一致地使用一个组合结构或其中单个对象，不必关心处理的是单个对象还是整个组合结构，简化了客户端代码。

3. 组合模式为树形结构的面向对象实现提供了一种灵活的解决方案，通过叶子对象和容器对象的递归组合，可以形成复杂的树形结构，但对树形结构的控制却非常简单。

### 适用场景

(1) 在具有整体和部分的层次结构中，希望通过一种方式忽略整体与部分的差异，客户端可以一致地对待它们。

(2) 在一个使用面向对象语言开发的系统中需要处理一个树形结构。

(3) 在一个系统中能够分离出叶子对象和容器对象，而且它们的类型不固定，需要增加一些新的类型。