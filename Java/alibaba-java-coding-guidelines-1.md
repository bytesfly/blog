# 阿里巴巴Java开发手册——编程规约


## Java开发手册版本更新说明

|版本号|版本名|更新日期|备注|
|:------:|:------:|:------:|:------:|
|1.3.0|终极版|2017.09.25|单元测试规约，IDE代码规约插件|
|1.3.1|纪念版|2017.11.30|修正部分描述|
|1.4.0|详尽版|2018.05.20|增加设计规约大类，共16条|
|1.5.0|华山版|2019.06.19|详细更新见下面|

本笔记主要基于`华山版`(1.5.0)的总结。华山版具体更新如下：
- 鉴于本手册是社区开发者集体智慧的结晶，本版本移除阿里巴巴`Java开发手册`的限定词`阿里巴巴`
- 新增21条新规约。比如，`switch`的NPE问题、浮点数的比较、无泛型限制、锁的使用方式、判断表达式、日期格式等
- 修改描述112处。比如，`IFNULL`的判断、集合的`toArray`、日志处理等
- 完善若干处示例。比如，命名示例、卫语句示例、`enum`示例、`finally`的`return`示例等。

## 专有名词解释
1. `POJO`（Plain Ordinary Java Object）: 在本手册中，POJO专指只有setter、getter、toString的简单类，包括DO、DTO、BO、VO等。
2. `GAV`（GroupId、ArtifactctId、Version）: Maven坐标，是用来唯一标识jar包。
3. `OOP`（Object Oriented Programming）: 本手册泛指类、对象的编程处理方式。
4. `ORM`（Object Relation Mapping）: 对象关系映射，对象领域模型与底层数据之间的转换，本文泛指ibatis, mybatis等框架。
5. `NPE`（java.lang.NullPointerException）: 空指针异常。
6. `SOA`（Service-Oriented Architecture）: 面向服务架构，它可以根据需求通过网络对松散耦合的粗粒度应用组件进行分布式部署、组合和使用，有利于提升组件可重用性，可维护性。
7. `IDE`（Integrated Development Environment）: 用于提供程序开发环境的应用程序，一般包括代码编辑器、编译器、调试器和图形用户界面等工具，本《手册》泛指 IntelliJ IDEA 和eclipse。
8. `OOM`（Out Of Memory）: 源于java.lang.OutOfMemoryError，当JVM没有足够的内存来为对象分配空间并且垃圾回收器也无法回收空间时，系统出现的严重状况。
9. `一方库`：本工程内部子项目模块依赖的库（jar包）。
10. `二方库`：公司内部发布到中央仓库，可供公司内部其它应用依赖的库（jar包）。
11. `三方库`：公司之外的开源库（jar包）。
## 一、 编程规约
### (一) 命名风格
***正例：***
- 国际通用的名称，可视同英文；`alibaba` / `youku` / `hangzhou` 等
- 类名使用`UpperCamelCase`风格，但以下情形例外：`DO` / `BO` / `DTO` / `VO` / `AO` / `PO` / `UID` 等。如：`UserDO` / `XmlService` / `TcpUdpDeal` 
- 方法名、参数名、成员变量、局部变量都统一使用`lowerCamelCase`风格，必须遵从驼峰形式；`localValue` / `getHttpMessage` / `inputUserId`
- 常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字长。`MAX_STOCK_COUNT` / `CACHE_EXPIRED_TIME`
- 抽象类命名使用`Abstract`或`Base`开头；异常类命名使用`Exception`结尾；测试类命名以它要测试的类的名称开始，以`Test`结尾
- 类型与中括号紧挨相连来表示数组，定义整形数组 ```int[] arrayDemo;```
- `包名`统一使用`小写`，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使用`单数`形式，但是`类名`如果有复数含义，`类名可以使用复数形式`。包名`com.alibaba.ai.util`，类名为`MessageUtils`（此规则参考`spring`的框架结构）
- 为了达到代码自解释的目标，任何自定义编程元素在命名时，使用尽量完整的单词组合来表达其意。在JDK中，表达原子更新的类名为：`AtomicReferenceFieldUpdater` 
- 在常量与变量的命名时，表示类型的名词放在词尾，以提升辨识度。如：`startTime` / `workQueue` / `nameList` / `TERMINATED_THREAD_COUNT`
- 如果模块、接口、类、方法使用了`设计模式`，在命名时需体现出具体模式（将设计模式体现在名字中，有利于阅读者快速理解架构设计理念）。如： `class OrderFactory` / `class LoginProxy` / `class ResourceObserver` 
- 接口类中的方法和属性不要加任何修饰符号（`public`也不要加），保持代码的简洁性,并加上有效的`Javadoc`注释。尽量不要在接口里定义变量，如果一定要定义变量，肯定是与接口方法相关，并且是整个应用的基础常量。接口方法签名`void commit();`，接口基础常量`String COMPANY = "alibaba";`
- 对于`Service`和`DAO`类，基于`SOA`的理念，暴露出来的服务一定是接口，内部的实现类用`Impl`的后缀与接口区别。如`CacheServiceImpl`实现`CacheService`接口
- 如果是`形容能力`的`接口`名称，取对应的形容词为接口名（通常是–`able`的形容词）如 `AbstractTranslator`实现`Translatable`接口
- 枚举类名带上`Enum`后缀，枚举成员名称需要`全大写`，单词间用`下划线`隔开。(说明：枚举其实就是特殊的类，域成员均为常量，且构造方法被默认强制是私有。)枚举名字为`ProcessStatusEnum`的成员名称：`SUCCESS` / `UNKNOWN_REASON`
- 各层命名规约:
> **A) Service/DAO 层方法命名规约**
> 1. 获取单个对象的方法用`get`做前缀。
> 2. 获取多个对象的方法用`list`做前缀，复数形式结尾如：`listObjects`。
> 3. 获取统计值的方法用`count`做前缀。 
> 4. 插入的方法用`save`/`insert`做前缀。
> 5. 删除的方法用`remove`/`delete`做前缀。
> 6. 修改的方法用`update`做前缀。

> **B) 领域模型命名规约**
> 1. 数据对象：`xxxDO`，xxx即为数据表名。
> 2. 数据传输对象：`xxxDTO`，xxx为业务领域相关的名称。
> 3. 展示对象：`xxxVO`，xxx一般为网页名称。
> 4. POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO。

***反例：***
- 不能以下划线或美元符号开始、结束，如：~~_name~~、~~$name~~、~~name_~~
- 严禁使用拼音与英文混合的方式，如：~~DaZhePromotion~~[打折]、 ~~getPingfenByName()~~ [评分]
- 避免在子父类的成员变量之间、或者不同代码块的局部变量之间采用完全相同的命名，使可读性降低。
- 杜绝完全不规范的缩写，避免望文不知义。`AbstractClass`“缩写”命名成~~AbsClass~~，`condition`“缩写”命名成~~condi~~，此类随意缩写严重降低了代码的可阅读性。
- 接口类中的方法和属性不要加任何修饰符号（public也不要加) ~~public abstract void f();~~
- POJO类中`布尔类型`变量都不要加`is`前缀，否则部分框架解析会引起`序列化错误`。
> 定义为基本数据类型`Boolean isDeleted`的属性，它的方法也是`isDeleted()`，RPC框架在反向解析的时候，“误以为”对应的属性名称是`deleted`，导致属性获取不到，进而抛出异常。

### (二) 常量定义
- 不允许任何魔法值（即未经预先定义的常量）直接出现在代码中 ~~String key =``"Id#taobao_"`` + tradeId;~~
- 在`long`或者`Long`赋值时，数值后使用大写的`L`，不能是小写的`l`，小写容易跟数字1混淆，造成误解。~~Long a = 2l;~~
- 不要使用一个常量类维护所有常量，要按常量功能进行归类，分开维护。正例：缓存相关常量放在类`CacheConsts`下；系统配置相关常量放在类`ConfigConsts`下。
- 如果变量值仅在一个固定范围内变化用`enum`类型来定义。

### (三) 代码格式
- 采用4个空格缩进，禁止使用`tab`字符。
- 注释的双斜线与注释内容之间有且仅有一个空格。
- 在进行类型强制转换时，右括号与强制转换值之间不需要任何空格隔开`int second = (int)first + 2;`
- IDE的text file encoding设置为UTF-8;IDE中文件的换行符使用Unix格式，不要使用Windows格式。
- 单个方法的总行数不超过80行
- 不同逻辑、不同语义、不同业务的代码之间插入`一个空行`分隔开来以提升可读性(说明：`任何情形，没有必要插入多个空行进行隔开`。)
### (四) OOP 规约
- 避免通过一个类的`对象引用`访问此类的`静态变量`或`静态方法`，无谓增加编译器解析成本，直接用类名来访问即可
- 所有的覆写方法，必须加`@Override`注解
- 外部正在调用或者二方库依赖的接口，不允许修改方法签名，避免对接口调用方产生影响。接口过时必须加`@Deprecated`注解，并清晰地说明采用的新接口或者新服务是么。
- 不能使用过时的类或方法。
- Object的`equals`方法容易抛空指针异常，应使用常量或确定有值的对象来调用equals，``"test".equals(object);``【推荐使用 java.util.Objects#equals（JDK7 引入的工具类】
- 所有整型包装类对象之间值的比较，全部使用`equals`方法比较。【在-128至127这个区间之外的所有数据，都会在堆上产生，并不会复用已有对象，这是一个大坑，推荐使用equals方法进行判断】
- 定义数据对象`DO`类时，属性类型要与数据库字段类型相匹配。数据库字段的`bigint`必须与类属性的`Long`类型相对应。
- 为了防止精度损失，禁止使用构造方法`BigDecimal(double)`的方式把`double`值转化为`BigDecimal`对象(在精确计算或值比较的场景中可能会导致业务逻辑异常)。`BigDecimal g = new BigDecimal(0.1f);`实际的存储值为：`0.10000000149`。正例：优先推荐入参为`String`的构造方法，或使用`BigDecimal`的`valueOf`方法，此方法内部其实执行了`Double`的`toString`，而Double的`toString`按`double`的实际能表达的精度对尾数进行了截断。
```java
BigDecimal recommend1 = new BigDecimal("0.1");
BigDecimal recommend2 = BigDecimal.valueOf(0.1);
```
- 关于基本数据类型与包装数据类型的使用标准如下：`1）【强制】所有的POJO类属性必须使用包装数据类型。2）【强制】RPC方法的返回值和参数必须使用包装数据类型。3） 【推荐】所有的局部变量使用基本数据类型。`【说明：POJO类属性没有初值是提醒使用者在需要使用时，必须自己显式地进行赋值，任何NPE问题，或者入库检查，都由使用者来保证。`正例：数据库的查询结果可能是null，因为自动拆箱，用基本数据类型接收有NPE风险。反例：比如显示成交总额涨跌情况，即正负x%，x为基本数据类型，调用的RPC服务，调用不成功时，返回的是默认值，页面显示为0%，这是不合理的，应该显示成中划线。所以包装数据类型的null值，能够表示额外的信息，如：远程调用失败，异常退出。`】 
- 定义 DO/DTO/VO等`POJO`类时，不要设定任何属性`默认值`。【反例：POJO 类的 createTime 默认值为 new Date()，但是这个属性在数据提取时并没有置入具体值，在更新其它字段时又附带更新了此字段，导致创建时间被修改成当前时间。】
- 序列化类新增属性时，请不要修改`serialVersionUID`字段，避免反序列失败；如果完全不兼容升级，避免反序列化混乱，那么请修改`serialVersionUID`值。(说明：注意serialVersionUID不一致会抛出序列化运行时异常。)
- 构造方法里面禁止加入任何业务逻辑，如果有初始化逻辑，请放在`init`方法中。
- `POJO`类必须写`toString`方法。使用IDE中的工具：source> generate toString时，如果继承了另一个POJO类，注意在前面加一下`super.toString`。【说明：在方法执行抛出异常时，可以直接调用 POJO 的 `toString()`方法打印其属性值，便于排查问题】
- 禁止在`POJO`类中，同时存在对应属性xxx的`isXxx()`和`getXxx()`方法。【说明：框架在调用属性 xxx 的提取方法时，并不能确定哪个方法一定是被优先调用到】
- 使用索引访问用String的`split`方法得到的数组时，需做最后一个分隔符后有无内容的检查，否则会有抛`IndexOutOfBoundsException`的风险
- 当一个类有多个构造方法，或者多个同名方法，这些方法应该按顺序放置在一起，便于阅读，此条规则优先于下一条
- 类内方法定义的顺序依次是：公有方法或保护方法 > 私有方法 > getter/setter方法。
- 在`getter/setter`方法中，不要增加业务逻辑，增加排查问题的难度
- 循环体内，字符串的连接方式，使用`StringBuilder`的`append`方法进行扩展
- `final`可以声明类(不允许被继承的类，如`String`类)、成员变量(不允许修改引用的域对象)、方法、以及本地变量(不允许运行过程中重新赋值的局部变量),避免上下文重复使用一个变量，使用final可以强制重新定义一个变量，方便更好地进行重构
- 慎用`Object`的`clone`方法来拷贝对象，对象`clone`方法默认是浅拷贝，若想实现深拷贝需覆写`clone`方法实现域对象的深度遍历式拷贝。
- 类成员与方法访问控制从严。`1）如果不允许外部直接通过new来创建对象，那么构造方法必须是private。2）工具类不允许有public或default构造方法。3）类非static 成员变量并且与子类共享，必须是protected。 4）类非static成员变量并且仅在本类使用，必须是private。5）类static成员变量如果仅在本类使用，必须是private。 6）若是static成员变量，考虑是否为final。7）类成员方法只供类内部调用，必须是 private。8）类成员方法只对继承类公开，那么限制为protected。`【说明：任何类、方法、参数、变量，严控访问范围。过于宽泛的访问范围，不利于模块解耦】
- `浮点数`之间的等值判断，基本数据类型不能用==来比较，包装数据类型不能用equals 来判断。【浮点数采用“尾数+阶码”的编码方式，类似于科学计数法的“有效数字+指数”的表示方式】
```java
// 反例
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
if (a == b) { // 预期进入此代码快，执行其它业务逻辑
// 但事实上 a==b 的结果为 false
}
Float x = Float.valueOf(a);
Float y = Float.valueOf(b);
if (x.equals(y)) { // 预期进入此代码快，执行其它业务逻辑
// 但事实上 equals 的结果为 false
}

// 正例

// (1)指定一个误差范围，两个浮点数的差值在此范围之内，则认为是相等的
float a = 1.0f - 0.9f;
float b = 0.9f - 0.8f;
float diff = 1e-6f;

if (Math.abs(a - b) < diff) {
    System.out.println("true");
}

// (2)使用BigDecimal来定义值，再进行浮点数的运算操作
// BigDecimal构造的时候注意事项 见上文
BigDecimal a = new BigDecimal("1.0");
BigDecimal b = new BigDecimal("0.9");
BigDecimal c = new BigDecimal("0.8");

BigDecimal x = a.subtract(b);
BigDecimal y = b.subtract(c);

if (x.equals(y)) {
    System.out.println("true");
}
```
### (五) 集合处理
- 关于`hashCode`和`equals`的处理，遵循如下规则:1）只要覆写`equals`，就必须覆写`hashCode`。2）因为`Set`存储的是不重复的对象，依据`hashCode`和`equals`进行判断，所以`Set`存储的对象必须覆写这两个方法。3）如果自定义对象作为`Map`的键，那么必须覆写`hashCode`和`equals`。【说明：`String`已覆写`hashCode`和`equals`方法，所以我们可以愉快地使用`String`对象作为`key`来使用】
- `ArrayList`的`subList`结果不可强转成`ArrayList`，否则会抛出`ClassCastException`异常，即java.util.RandomAccessSubList cannot be cast to java.util.ArrayList【说明：`subList`返回的是`ArrayList`的内部类`SubList`，并不是`ArrayList`而是`ArrayList`的一个视图，对于`SubList`子列表的所有操作最终会反映到原列表上】
- 使用`Map`的方法`keySet()/values()/entrySet()`返回集合对象时，不可以对其进行添加元素操作，否则会抛出`UnsupportedOperationException`异常
- `Collections`类返回的对象，如：`emptyList()/singletonList()`等都是immutable list，不可对其进行添加或者删除元素的操作【反例：如果查询无结果，返回 `Collections.emptyList()`空集合对象，调用方一旦进行了添加元素的操作，就会触发`UnsupportedOperationException`异常。】
- 在`subList`场景中，高度注意对原集合元素的增加或删除，均会导致子列表的遍历、增加、删除产生`ConcurrentModificationException`异常
- 使用集合转数组的方法，必须使用集合的`toArray(T[] array)`，传入的是类型完全一致、长度为0的空数组【反例：直接使用toArray无参方法存在问题，此方法返回值只能是Object[]类，若强转其它类型数组将出现ClassCastException错误。】
```java
// 正例
List<String> list = new ArrayList<>(2);
list.add("行无际");
list.add("itwild");
String[] array = list.toArray(new String[0]);
/*
说明：
使用toArray带参方法，数组空间大小的length：
1）等于0，动态创建与size相同的数组，性能最好
2）大于0但小于size，重新创建大小等于size的数组，增加GC负担
3）等于size，在高并发情况下，数组创建完成之后，size正在变大的情况下，负面影响与上相同
4）大于size，空间浪费，且在size处插入null值，存在NPE隐患
*/
```
- 在使用`Collection`接口任何实现类的`addAll()`方法时，都要对输入的集合参数进行NPE判断 【说明：在`ArrayList#addAll`方法的第一行代码即`Object[] a = c.toArray();`其中c为输入集合参数，如果为null，则直接抛出异常。】
- 使用工具类`Arrays.asList()`把数组转换成集合时，不能使用其修改集合相关的方法，它的`add/remove/clear`方法会抛出`UnsupportedOperationException`异常【说明：`asList`的返回对象是一个`Arrays`内部类，并没有实现集合的修改方法。`Arrays.asList`体现的是适配器模式，只是转换接口，后台的数据仍是数组。】
```java
String[] str = new String[] { "it", "wild" };
List list = Arrays.asList(str);

// 第一种情况：list.add("itwild"); 运行时异常
// 第二种情况：str[0] = "changed1"; 也会随之修改
// 反之亦然 list.set(0, "changed2");
```
- 泛型通配符`<? extends T>`来接收返回的数据，此写法的泛型集合不能使用add方法，而`<? super T>`不能使用get方法，作为接口调用赋值时易出错。【说明：扩展说一下`PECS`(Producer Extends Consumer Super)原则：第一、频繁往外读取内容的，适合用`<? extends T>`。第二、经常往里插入的，适合用`<? super T>`】
> 这个地方我觉得有必要简单解释一下(`行无际`本人的个人理解哈，有不对的地方欢迎指出)，上面的说法可能有点官方或者难懂。其实我们一直也是这么干的，不过没注意而已。举个最简单的例子，用`泛型`的时候，如果你`遍历`(`read`)一个List，你是不是希望List里面装的越具体越好啊，你希望里面装的是`Object`吗，如果里面装的是`Object`那么你想想你会有多痛苦，每个对象都用`instanceof`判断一下再`类型强转`，所以这个方法的参数List主要用于`遍历`(`read`)的时候，大多数情况你可能会要求里面的元素最大是`T`类型，即用`<? extends T>`限制一下。再看你往List里面`插入`(`write`)数据又会怎么样，为了灵活性和可扩展性，你马上可能就要说我当然希望List里面装的是`Object`了，这样我什么类型的对象都能往List里面写啊，这样设计出来的接口的灵活性和可扩展性才强啊，如果里面装的类型太靠下(假定`继承层次从上往下`，父类在上，子孙类在下)，那么位于上级的很多类型的数据你就无法写入了，这个时候用`<? super T>`来限制一下最小是`T`类型。下面我们来看`Collections.copy()`这个例子。

```java
// 这里就要求dest的List里面的元素类型 不能在src的List元素类型 之下
// 如果dest的List元素类型位于src的List元素类型之下，就会出现写不进dest
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    //....省略具体的copy代码
}

// 下面再看我写的测试代码就更容易理解了
static class Animal {}

static class Dog extends Animal {}

static class BlackDog extends Dog {}

@Test
public void test() throws Exception {

    List<Dog> dogList = new ArrayList<>(2);
    dogList.add(new BlackDog());
    dogList.add(new BlackDog());

    List<Animal> animalList = new ArrayList<>(2);
    animalList.add(new Animal());
    animalList.add(new Animal());

    // 错误，无法编译通过
    Collections.copy(dogList, animalList);

    // 正确
    Collections.copy(animalList, dogList);
    
    // Collections.copy()的泛型参数就起作到了很好的限制作用
    // 编译期就能发现类型不对
}
```
- 在无泛型限制定义的集合赋值给泛型限制的集合时，在使用集合元素时，需要进行`instanceof`判断，避免抛出`ClassCastException`异常。
```java
// 反例
List<String> generics = null;

List notGenerics = new ArrayList(10);
notGenerics.add(new Object());
notGenerics.add(new Integer(1));

generics = notGenerics;

// 此处抛出 ClassCastException 异常
String string = generics.get(0);
```
- 不要在`foreach`循环里进行元素的`remove/add`操作。`remove`元素请使用`Iterator`方式，如果并发操作，需要对Iterator对象`加锁`
```java
// 正例
List<String> list = new ArrayList<>(); 
list.add("1"); 
list.add("2"); 

Iterator<String> iterator = list.iterator(); 
while (iterator.hasNext()) { 
    String item = iterator.next(); 
    if (删除元素的条件) {
        iterator.remove(); 
    } 
}

// 反例
for (String item : list) { 
  if ("1".equals(item)) { 
    list.remove(item); 
  } 
}
```
- 在JDK7版本及以上，`Comparator`实现类要满足如下三个条件，不然`Arrays.sort`，`Collections.sort`会抛`IllegalArgumentException`异常【说明：三个条件如下 1）x，y 的比较结果和 y，x 的比较结果相反。2）x>y，y>z，则x>z。 3） x=y，则x，z比较结果和y，z 比较结果相同。】
```java
// 反例：下例中没有处理相等的情况
new Comparator<Student>() { 
  @Override 
  public int compare(Student o1, Student o2) { 
    return o1.getId() > o2.getId() ? 1 : -1; 
  } 
};
```
- 集合泛型定义时，在JDK7及以上，使用diamond语法或全省略。【说明：菱形泛型，即 diamond，直接使用<>来指代前边已经指定的类型】
```java
// 正例
// diamond 方式，即<>
HashMap<String, String> userCache = new HashMap<>(16);
// 全省略方式
ArrayList<User> users = new ArrayList(10);
```
- 集合初始化时，指定集合初始值大小【说明：`HashMap`使用`HashMap(int initialCapacity)`初始化。】
> 正例：`initialCapacity` = (需要存储的元素个数 / 负载因子) + 1。注意负载因子（即`loader factor`）默认为0.75，如果暂时无法确定初始值大小，请设置为16（即默认值）。

> 反例：`HashMap`需要放置1024个元素，由于没有设置容量初始大小，随着元素不断增加，容量7次被迫扩大，`resize`需要重建`hash`表，严重影响性能。
- 使用`entrySet`遍历`Map`类集合KV，而不是`keySet`方式进行遍历。【说明：`keySet`其实是遍历了2次，一次是转为`Iterator`对象，另一次是从`hashMap`中取出 `key`所对应的`value`。而`entrySet`只是遍历了一次就把`key`和`value`都放到了entry中，效率更高。如果是JDK8，使用`Map.forEach`方法。】
> 正例：`values()`返回的是V值集合，是一个list集合对象；`keySet()`返回的是K值集合，是一个Set集合对象；`entrySet()`返回的是`K-V`值组合集合。
- 高度注意`Map`类集合`K/V`能不能存储`null`值的情况，如下表格：

|集合类|Key|Value|Super|说明|
|-----|----|-----|-----|----|
|Hashtable|不允许为null|不允许为null|Dictionary|线程安全|
|ConcurrentHashMap|不允许为null|不允许为null|AbstractMap|锁分段技术（JDK8:CAS）|
|TreeMap|不允许为null|允许为null|AbstractMap|线程不安全|
|HashMap|允许为null|允许为null|AbstractMap|线程不安全|
> 反例：由于`HashMap`的干扰，很多人认为`ConcurrentHashMap`是可以置入`null`值，而事实上，存储`null`值时会抛出`NPE`异常。
- 合理利用好集合的有序性(`sort`)和稳定性(`order`)，避免集合的无序性(`unsort`)和不稳定性(`unorder`)带来的负面影响。【说明：有序性是指遍历的结果是按某种比较规则依次排列的。稳定性指集合每次遍历的元素次序是一定的。如：`ArrayList`是 `order/unsort`；`HashMap`是`unorder/unsort`；`TreeSet`是`order/sort`。】
- 利用`Set`元素唯一的特性，可以快速对一个集合进行去重操作，避免使用`List`的`contains`方法进行遍历、对比、去重操作

### (六) 并发处理
- 获取单例对象需要保证线程安全，其中的方法也要保证线程安全【说明：资源驱动类、工具类、单例工厂类都需要注意】
- 创建线程或线程池时请指定有意义的线程名称，方便出错时回溯
```java
// 正例：自定义线程工厂，并且根据外部特征进行分组，比如机房信息
public class UserThreadFactory implements ThreadFactory {
    private final String namePrefix;
    private final AtomicInteger nextId = new AtomicInteger(1);
    // 定义线程组名称，在 jstack 问题排查时，非常有帮助
    UserThreadFactory(String whatFeaturOfGroup) {
        namePrefix = "From UserThreadFactory's " + whatFeaturOfGroup + "-Worker-"; 
    }
    
    @Override
    public Thread newThread(Runnable task) {
        String name = namePrefix + nextId.getAndIncrement();
        Thread thread = new Thread(null, task, name, 0, false);
        System.out.println(thread.getName());
        return thread; 
    } 
}
```
- 线程资源必须通过线程池提供，不允许在应用中自行显式创建线程【说明：线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。如果不使用线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题】
- 线程池不允许使用`Executors`去创建，而是通过`ThreadPoolExecutor`的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险
> 说明：`Executors`返回的线程池对象的弊端如下： 

> 1）`FixedThreadPool`和`SingleThreadPool`：
允许的请求队列长度为`Integer.MAX_VALUE`，可能会堆积大量的请求，从而导致`OOM`。

> 2）`CachedThreadPool`：
允许的创建线程数量为`Integer.MAX_VALUE`，可能会创建大量的线程，从而导致`OOM`。
- `SimpleDateFormat`是线程不安全的类，一般不要定义为`static`变量，如果定义为`static`，必须加锁。【说明：如果是JDK8的应用，可以使用`Instant`代替`Date`，`LocalDateTime`代替`Calendar`，`DateTimeFormatter`代替`SimpleDateFormat`，官方给出的解释：`simple beautiful strong immutable thread-safe`。】
```java
// 正例：注意线程安全。亦推荐如下处理
private static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>() { 
    @Override 
    protected DateFormat initialValue() { 
        return new SimpleDateFormat("yyyy-MM-dd"); 
    } 
};
```
- 必须回收自定义的`ThreadLocal`变量，尤其在线程池场景下，线程经常会被复用，如果不清理自定义的`ThreadLocal`变量，可能会影响后续业务逻辑和造成内存泄露等问题。尽量使用`try-finally`块进行回收
```java
// 正例
objectThreadLocal.set(userInfo);
try {
    // ...
} finally {
    objectThreadLocal.remove();
}
```
- 高并发时，同步调用应该去考量锁的性能损耗。能用无锁数据结构，就不要用锁；能锁区块，就不要锁整个方法体；能用对象锁，就不要用类锁【说明：尽可能使加锁的代码块工作量尽可能的小，避免在锁代码块中调用`RPC`方法。】
- 对多个资源、数据库表、对象同时加锁时，需要保持一致的加锁顺序，否则可能会造成死锁。【说明：线程一需要对表A、B、C依次全部加锁后才可以进行更新操作，那么线程二的加锁顺序也必须是A、B、C，否则可能出现死锁。】
- 在使用阻塞等待获取锁的方式中，必须在`try`代码块之外，并且在加锁方法与`try`代码块之间没有任何可能抛出异常的方法调用，避免加锁成功后，在`finally`中无法解锁
> 说明一：如果在`lock`方法与`try`代码块之间的方法调用抛出异常，那么无法解锁，造成其它线程无法成功获取锁。

> 说明二：如果`lock`方法在`try`代码块之内，可能由于其它方法抛出异常，导致在 `finally`代码块中，`unlock`对未加锁的对象解锁，它会调用`AQS`的`tryRelease`方法（取决于具体实现类），抛出`IllegalMonitorStateException`异常。

> 说明三：在`Lock`对象的`lock`方法实现中可能抛出`unchecked`异常，产生的后果与说明二相同
```java
// 正例
Lock lock = new XxxLock();
// ...
lock.lock();
try {
    doSomething();
    doOthers();
} finally {
    lock.unlock();
}

// 反例
Lock lock = new XxxLock();
// ...
try {
    // 如果此处抛出异常，则直接执行 finally 代码块
    doSomething();
    // 无论加锁是否成功，finally 代码块都会执行
    lock.lock();
    doOthers();
} finally {
    lock.unlock();
}
```
- 在使用尝试机制来获取锁的方式中，进入业务代码块之前，必须先判断当前线程是否持有锁。锁的释放规则与锁的阻塞等待方式相同
```java
// 正例
Lock lock = new XxxLock();
// ...
boolean isLocked = lock.tryLock();
if (isLocked) {
    try {
      doSomething();
      doOthers();
    } finally {
      lock.unlock();
    } 
}
```
- 并发修改同一记录时，避免更新丢失，需要加锁。要么在应用层加锁，要么在缓存加锁，要么在数据库层使用乐观锁，使用`version`作为更新依据【说明：如果每次访问冲突概率小于20%，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次数不得小于3次。】
- 多线程并行处理定时任务时，`Timer`运行多个`TimeTask`时，只要其中之一没有捕获抛出的异常，其它任务便会自动终止运行，如果在处理定时任务时使用`ScheduledExecutorService`则没有这个问题
- 资金相关的金融敏感信息，使用悲观锁策略。【说明：乐观锁在获得锁的同时已经完成了更新操作，校验逻辑容易出现漏洞，另外，乐观锁对冲突的解决策略有较复杂的要求，处理不当容易造成系统压力或数据异常，所以资金相关的金融敏感信息不建议使用乐观锁更新。】
- 使用`CountDownLatch`进行异步转同步操作，每个线程退出前必须调用`countDown`方法，线程执行代码注意catch异常，确保`countDown`方法被执行到，避免主线程无法执行至`await`方法，直到超时才返回结果【说明：注意，子线程抛出异常堆栈，不能在主线程`try-catch`到。】
- 避免`Random`实例被多线程使用，虽然共享该实例是线程安全的，但会因竞争同一`seed`导致的性能下降【说明：`Random`实例包括`java.util.Random`的实例或者`Math.random()`的方式。正例：在JDK7之后，可以直接使用API`ThreadLocalRandom`，而在JDK7之前，需要编码保证每个线程持有一个实例】
- 在并发场景下，通过双重检查锁`（double-checked locking）`实现延迟初始化的优化问题隐患(可参考 The "Double-Checked Locking is Broken" Declaration)，推荐解决方案中较为简单一种（适用于JDK5及以上版本），将目标属性声明为`volatile`型。
```java
// 注意 这里的代码并非出自官方的《java开发手册》
// 参考 https://blog.csdn.net/lovelion/article/details/7420886
public class LazySingleton { 
    // volatile除了保证内容可见性还有防止指令重排序
    // 对象的创建实际上是三条指令：
    // 1、分配内存地址 2、内存地址初始化 3、返回内存地址句柄
    // 其中2、3之间可能发生指令重排序
    // 重排序可能导致线程A创建对象先执行1、3两步，
    // 结果线程B进来判断句柄已经不为空，直接返回给上层方法
    // 此时对象还没有正确初始化内存，导致上层方法发生严重错误
    private volatile static LazySingleton instance = null; 
 
    private LazySingleton() { } 
    
    public static LazySingleton getInstance() { 
        // 第一重判断
        if (instance == null) {
            synchronized (LazySingleton.class) {
                // 第二重判断
                if (instance == null) {
                    // 创建单例实例
                    instance = new LazySingleton(); 
                }
            }
        }
        return instance; 
    }
}

// 既然这里提到 单例懒加载，还有这样写的
// 参考 https://blog.csdn.net/lovelion/article/details/7420888
class Singleton {
  private Singleton() { }
  
  private static class HolderClass {
  // 由Java虚拟机来保证其线程安全性，确保该成员变量只能初始化一次
    final static Singleton instance = new Singleton();
  }

  public static Singleton getInstance() {
      return HolderClass.instance;
  }
}
```
- `volatile`解决多线程内存不可见问题。对于一写多读，是可以解决变量同步问题，但是如果多写，同样无法解决线程安全问题。【说明：如果是`count++`操作，使用如下类实现：`AtomicInteger count = new AtomicInteger();` `count.addAndGet(1);`如果是JDK8，推荐使用`LongAdder`对象，比`AtomicLong`性能更好（减少乐观锁的重试次数）。】
- `HashMap`在容量不够进行`resize`时由于高并发可能出现死链，导致CPU飙升，在开发过程中可以使用其它数据结构或加锁来规避此风险
- `ThreadLocal`对象使用`static`修饰，`ThreadLocal`无法解决共享对象的更新问题【说明：这个变量是针对一个线程内所有操作共享的，所以设置为静态变量，所有此类实例共享此静态变量，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只要是这个线程内定义的)都可以操控这个变量】

### (七) 控制语句
- 当`switch`括号内的变量类型为`String`并且此变量为外部参数时，必须先进行`null`判断。
```java
public class SwitchString {
    
    public static void main(String[] args) {
        // 这里会抛异常 java.lang.NullPointerException
        method(null);
    }
    
    public static void method(String param) {
        switch (param) {
            // 肯定不是进入这里
            case "sth":
                System.out.println("it's sth");
                break;
            // 也不是进入这里
            case "null":
                System.out.println("it's null");
                break;
            // 也不是进入这里
            default:
                System.out.println("default");
        } 
    }
}
```
- 在`if/else/for/while/do`语句中必须使用大括号。【说明：即使只有一行代码，避免采用单行的编码方式：~~`if (condition) statements;`~~】
- 在`高并发场景`中，避免使用”等于”判断作为中断或退出的条件。【说明：如果并发控制没有处理好，容易产生等值判断被“击穿”的情况，使用大于或小于的区间判断条件来代替。】
> 反例：判断剩余奖品数量等于 0 时，终止发放奖品，但因为并发处理错误导致奖品数量瞬间变成了负数，这样的话，活动无法终止。
- 表达异常的分支时，少用`if-else`方式，这种方式可以改写成下面代码：【说明：如果非使用`if()...else if()...else...`方式表达逻辑，避免后续代码维护困难，请勿超过`3`层】
```java
if (condition) { 
    ...
    return obj; 
} 
// 接着写 else 的业务逻辑代码;
```
> 超过3层的`if-else`的逻辑判断代码可以使用`卫语句`、`策略模式`、`状态模式`等来实现。其中卫语句即代码逻辑先考虑失败、异常、中断、退出等直接返回的情况，以方法多个出口的方式，解决代码中判断分支嵌套的问题，这是逆向思维的体现。
```java
// 示例代码
public void findBoyfriend(Man man) {
  if (man.isUgly()) {
    System.out.println("本姑娘是外貌协会的资深会员");
    return;
  }
  if (man.isPoor()) {
    System.out.println("贫贱夫妻百事哀");
    return;
  }
  if (man.isBadTemper()) {
    System.out.println("银河有多远，你就给我滚多远");
    return;
  }
  System.out.println("可以先交往一段时间看看");
}
```
- 除常用方法（如`getXxx/isXxx`）等外，不要在条件判断中执行其它复杂的语句，将复杂逻辑判断的结果赋值给一个有意义的布尔变量名，以提高可读性。【说明：很多`if`语句内的逻辑表达式相当复杂，与、或、取反混合运算，甚至各种方法纵深调用，理解成本非常高。如果赋值一个非常好理解的布尔变量名字，则是件令人爽心悦目的事情。】
```java
// 正例
// 伪代码如下
final boolean existed = (file.open(fileName, "w") != null) && (...) || (...);
if (existed) {
  ...
}

// 反例
// 哈哈，这好像是ReentrantLock里面有类似风格的代码
// 连Doug Lea的代码都拿来当做反面教材啊
// 早前就听别人说过“编程不识Doug Lea,写尽Java也枉然!!!”
public final void acquire(long arg) {
  if (!tryAcquire(arg) &&
    acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) {
      selfInterrupt();
  } 
}
```
- 不要在其它表达式（尤其是条件表达式）中，插入赋值语句【说明：赋值点类似于人体的穴位，对于代码的理解至关重要，所以赋值语句需要清晰地单独成为一行。】
```java
// 反例
public Lock getLock(boolean fair) {
  // 算术表达式中出现赋值操作，容易忽略 count 值已经被改变
  threshold = (count = Integer.MAX_VALUE) - 1;
  // 条件表达式中出现赋值操作，容易误认为是 sync==fair
  return (sync = fair) ? new FairSync() : new NonfairSync();
}
```
- 循环体中的语句要考量性能，以下操作尽量移至循环体外处理，如定义对象、变量、获取数据库连接，进行不必要的`try-catch`操作（这个`try-catch`是否可以移至循环体外）
- 避免采用取反逻辑运算符。【说明：取反逻辑不利于快速理解，并且取反逻辑写法必然存在对应的正向逻辑写法。正例：使用`if (x < 628)`来表达x小于628。反例：使用 if (!(x >= 628))来表达x小于628。】
- 下列情形，需要进行参数校验
> 1） 调用频次低的方法。2）执行时间开销很大的方法。此情形中，参数校验时间几乎可以忽略不计，但如果因为参数错误导致中间执行回退，或者错误，那得不偿失。3）需要极高稳定性和可用性的方法。4）对外提供的开放接口，不管是`RPC/API/HTTP`接口。5）敏感权限入口。
- 下列情形，不需要进行参数校验：
> 1）极有可能被循环调用的方法。但在方法说明里必须注明外部参数检查要求。 2）底层调用频度比较高的方法。毕竟是像纯净水过滤的最后一道，参数错误不太可能到底层才会暴露问题。一般`DAO`层与`Service`层都在同一个应用中，部署在同一台服务器中，所以`DAO`的参数校验，可以省略。3）被声明成`private`只会被自己代码所调用的方法，如果能够确定调用方法的代码传入参数已经做过检查或者肯定不会有问题，此时可以不校验参数。

### (八) 注释规约
- 类、类属性、类方法的注释必须使用`Javadoc`规范，使用`/**内容*/`格式，不得使用`// xxx`方式。【说明：在IDE编辑窗口中，`Javadoc`方式会提示相关注释，生成 `Javadoc`可以正确输出相应注释；在IDE中，工程调用方法时，不进入方法即可悬浮提示方法、参数、返回值的意义，提高阅读效率。】
- 所有的抽象方法（包括接口中的方法）必须要用Javadoc注释、除了返回值、参数、异常说明外，还必须指出该方法做什么事情，实现什么功能【说明：对子类的实现要求，或者调用注意事项，请一并说明】
- 所有的类都必须添加创建者和创建日期。
- 方法内部单行注释，在被注释语句上方另起一行，使用//注释。方法内部多行注释使用`/* */`注释，注意与代码对齐
- 所有的`枚举类型`字段必须要有注释，说明每个数据项的用途
- 与其“半吊子”英文来注释，不如用中文注释把问题说清楚。专有名词与关键字保持英文原文即可【反例：“TCP 连接超时”解释成“传输控制协议连接超时”，理解反而费脑筋。】
- 代码修改的同时，注释也要进行相应的修改，尤其是参数、返回值、异常、核心逻辑等的修改。【代码与注释更新不同步，就像路网与导航软件更新不同步一样，如果导航软件严重滞后，就失去了导航的意义】
- 谨慎注释掉代码。在上方详细说明，而不是简单地注释掉。如果无用，则删除。【说明：代码被注释掉有两种可能性：1）后续会恢复此段代码逻辑。2）永久不用。前者如果没有备注信息，难以知晓注释动机。后者建议直接删掉（代码仓库已然保存了历史代码）】
- 对于注释的要求：第一、能够准确反映设计思想和代码逻辑；第二、能够描述业务含义，使别的程序员能够迅速了解到代码背后的信息。完全没有注释的大段代码对于阅读者形同天书，注释是给自己看的，即使隔很长时间，也能清晰理解当时的思路；注释也是给继任者看的，使其能够快速接替自己的工作。
- 好的命名、代码结构是自解释的，注释力求精简准确、表达到位。避免出现注释的一个极端：过多过滥的注释，代码的逻辑一旦修改，修改注释是相当大的负担【语义清晰的代码不需要额外的注释。】
- 特殊注释标记，请注明标记人与标记时间。注意及时处理这些标记，通过标记扫描，经常清理此类标记。线上故障有时候就是来源于这些标记处的代码。
> 1）待办事宜（`TODO`）:（标记人，标记时间，[预计处理时间]）表示需要实现，但目前还未实现的功能。这实际上是一个`Javadoc`的标签，目前的`Javadoc`还没
有实现，但已经被广泛使用。只能应用于类，接口和方法（因为它是一个Javadoc标签）。2）错误，不能工作（`FIXME`）:（标记人，标记时间，[预计处理时间]）
在注释中用`FIXME`标记某代码是错误的，而且不能工作，需要及时纠正的情况。

### (九) 其它
- 在使用正则表达式时，利用好其预编译功能，可以有效加快正则匹配速度【说明：不要在方法体内定义：`Pattern pattern = Pattern.compile("规则");`】
- 注意`Math.random()`这个方法返回是`double`类型，注意取值的范围`0≤x<1`（能够取到零值，注意除零异常），如果想获取整数类型的随机数，不要将x放大10的若干倍然后取整，直接使用`Random`对象的`nextInt`或者`nextLong`方法。
- 获取当前毫秒数`System.currentTimeMillis();`而不是`new Date().getTime();`【说明：如果想获取更加精确的纳秒级时间值，使用`System.nanoTime()`的方式。在JDK8中，针对统计时间等场景，推荐使用`Instant`类。】
- 日期格式化时，传入`pattern`中表示年份统一使用`小写的y`。【说明：日期格式化时，`yyyy`表示当天所在的年，而大写的`YYYY`代表是 week in which year（JDK7之后引入的概念），意思是当天所在的周属于的年份，一周从周日开始，周六结束，只要本周跨年，返回的`YYYY`就是下一年。另外需要注意：**表示月份是大写的M，表示分钟则是小写的m，24小时制的是大写的H，12小时制的则是小写的h** 。正例：`new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");`】
- 任何数据结构的构造或初始化，都应指定大小，避免数据结构无限增长吃光内存
- 及时清理不再使用的代码段或配置信息【说明：对于垃圾代码或过时配置，坚决清理干净，避免程序过度臃肿，代码冗余。正例：对于暂时被注释掉，后续可能恢复使用的代码片断，在注释代码上方，统一规定使用三个斜杠(`///`)来说明注释掉代码的理由】
