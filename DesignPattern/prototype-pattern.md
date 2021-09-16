# 原型模式(Prototype Pattern)——对象的克隆


说明：设计模式系列文章是读`刘伟`所著`《设计模式的艺术之道(软件开发人员内功修炼之道)》`一书的阅读笔记。个人感觉这本书讲的不错，有兴趣推荐读一读。详细内容也可以看看此书作者的博客`https://blog.csdn.net/LoveLion/article/details/17517213`。

## 模式概述

### 模式定义

我们平时经常进行的两个电脑基本操作：复制和粘贴，快捷键通常为`Ctrl+C`和`Ctrl+V`，通过对已有对象的复制和粘贴，我们可以创建大量的相同对象。如何在一个面向对象系统中实现对象的复制和粘贴呢？`原型模式`正为解决此类问题而诞生。

> `原型模式(Prototype  Pattern)`：使用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。原型模式是一种对象创建型模式。

需要注意的是通过克隆方法所创建的对象是全新的对象，它们在内存中拥有新的地址，通常对克隆所产生的对象进行修改对原型对象不会造成任何影响，每一个克隆对象都是相互独立的。通过不同的方式修改可以得到一系列相似但不完全相同的对象。

### 模式结构图

原型模式结构图如下所示：

![原型模式结构图](https://img2020.cnblogs.com/blog/1546632/202005/1546632-20200530123209513-298312089.png)

原型模式结构图中包含如下几个角色:
- `Prototype（抽象原型类）`：它是声明克隆方法的接口，是所有具体原型类的公共父类，可以是抽象类也可以是接口，甚至还可以是具体实现类。

- `ConcretePrototype（具体原型类）`：它实现在抽象原型类中声明的克隆方法，在克隆方法中返回自己的一个克隆对象。

- `Client（客户类）`：让一个原型对象克隆自身从而创建一个新的对象，在客户类中只需要直接实例化或通过工厂方法等方式创建一个原型对象，再通过调用该对象的克隆方法即可得到多个相同的对象。由于客户类针对抽象原型类Prototype编程，因此用户可以根据需要选择具体原型类，系统具有较好的可扩展性，增加或更换具体原型类都很方便。

原型模式的核心在于如何实现克隆方法。

### 模式伪代码
定义接口的`clone()`方法，具体原型类(`ConcretePrototype`)实现克隆方法
```java
public interface Prototype {
    Prototype clone();
}

public class ConcretePrototype implements Prototype {
    //成员属性
    private String attr;

    @Override
    public Prototype clone() {
        ConcretePrototype prototype = new ConcretePrototype();
        prototype.setAttr(this.attr);
        return prototype;
    }

    // 无参构造
    public ConcretePrototype() {

    }

    // 带参构造
    public ConcretePrototype(String attr) {
        this.attr = attr;
    }

    // getter and setter
    public String getAttr() {
        return attr;
    }

    public void setAttr(String attr) {
        this.attr = attr;
    }
}
```
在客户类中我们只需要创建一个`ConcretePrototype`对象作为原型对象，然后调用其`clone()`方法即可得到对应的克隆对象
```java
public static void main(String[] args) {
  Prototype obj1 = new ConcretePrototype("行无际");
  Prototype obj2 = obj1.clone();
}
```
这是`原型模式`的通用实现，它与编程语言特性无关，任何面向对象语言都可以使用这种形式来实现对原型的克隆。

## 模式应用

### 模式在JDK中的应用
`Java`语言提供了`clone()`方法。`Cloneable`接口是一个标记接口,也就是没有任何内容,定义如下:
```java
/**
 * A class implements the <code>Cloneable</code> interface to
 * indicate to the {@link java.lang.Object#clone()} method that it
 * is legal for that method to make a
 * field-for-field copy of instances of that class.
 * @see     java.lang.CloneNotSupportedException
 * @see     java.lang.Object#clone()
 */
public interface Cloneable {
}
```
`clone()`方法是在`Object`中定义的,而且是`protected`型的,只有实现了这个`Cloneable`接口，才可以在该类的实例上调用`clone`方法,否则会抛出`CloneNotSupportException`。`Object`中默认的实现是一个浅拷贝,如果需要深拷贝的话,需要自己重写`clone`方法或者把`对象序列化再反序列化得到新对象`或借助第三方的库实现深拷贝。
```java
public class Object {
  protected native Object clone() throws CloneNotSupportedException;
}

public class Item implements Cloneable {

    private String name;

    private Object obj;
    
    public static void main(String[] args) throws Exception {
        Item item = new Item("行无际", new Object());
        Item replicaItem = (Item) item.clone();

        // true，表明默认浅拷贝
        System.out.println(item.obj == replicaItem.obj);
    }
}
```

### 模式在开源项目中的应用

项目中我们可能会结合一些工具库，如`BeanUtils.copyProperties()`来实现对象的克隆。这里也就没必要举例子，其实就是把对象的属性等拷贝一份，但是要根据实际需求来决定是深拷贝还是浅拷贝。另外`Spring`容器中的`Bean`的`scope`有点这里的味道。
1. 当bean的scope为`singleton`时，Spring容器仅创建类的一个实例对象，当下次获取的时候直接返回已创建出来的对象，即单例

2. 当bean的scope为`prototype`时，用户每次获取对象时都创建一个全新的对象返回。

## 模式总结

原型模式作为一种快速创建大量相同或相似对象的方式，在软件开发中应用较为广泛，很多软件提供的复制(`Ctrl+C`)和粘贴(`Ctrl+V`)操作就是原型模式的典型应用，下面对该模式的使用效果和适用情况进行简单的总结。

### 主要优点

当创建新的对象实例较为复杂时，使用原型模式可以简化对象的创建过程，通过复制一个已有实例可以提高新实例的创建效率。

### 适用场景

(1) 创建新对象成本较大（如初始化需要占用较长的时间，占用太多的CPU资源或网络资源），新的对象可以通过原型模式对已有对象进行复制来获得，如果是相似对象，则可以对其成员变量稍作修改。

(2) 如果系统要保存对象的状态，而对象的状态变化很小，或者对象本身占用内存较少时，可以使用原型模式配合备忘录模式来实现。

(3) 需要避免使用分层次的工厂类来创建分层次的对象，并且类的实例对象只有一个或很少的几个组合状态，通过复制原型对象得到新实例可能比使用构造函数创建一个新实例更加方便。
