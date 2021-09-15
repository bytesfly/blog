# 换种思路写Mock，让单元测试更简单


## 开篇引入


> 单元测试中的Mock方法，通常是为了绕开那些依赖外部资源或无关功能的方法调用，使得测试重点能够集中在需要验证和保障的代码逻辑上。在定义Mock方法时，开发者真正关心的只有一件事："这个调用，在测试的时候要换成那个假的Mock方法"。



然而当下主流的Mock框架在实现Mock功能时，需要开发者操心的事情实在太多：Mock框架如何初始化、与所用的单元测试框架是否兼容、要被Mock的方法是不是私有的、是不是静态的、被Mock对象是new出来的还是注入的、怎样把被测对象送回被测类里...这些非关键的额外工作极大分散了使用Mock工具应有的乐趣。


周末，在翻github上alibaba的开源项目时，无意间看到了下面这个特立独行的轻量Mock工具。当前知道这个工具的人应该很少，star人数28(包括本人在内)，另外我留意了一下该项目在github上第一次提交代码时间是2020年5月9日。


项目地址：[https://github.com/alibaba/testable-mock](https://github.com/alibaba/testable-mock)
文档：[https://alibaba.github.io/testable-mock/](https://alibaba.github.io/testable-mock/)


> 换种思路写Mock，让单元测试更简单。无需初始化，不挑测试框架，甭管要换的方法是被测类的私有方法、静态方法还是其他任何类的成员方法，也甭管要换的对象是怎么创建的。写好Mock方法，加个@TestableMock注解，一切统统搞定。



这是 `README` 上的描述。扫了一眼项目描述与目录结构后，就抵制不住诱惑，快速上手玩了一下。于是，就有了这篇划水博客，让看到的朋友也心痒一下(●´ω｀●)。当然，最重要的是如果确实好用的话，可以在实际项目中用起来，这样就不再反感需要Mock的单元测试了。


## 快速上手


完整代码见本人github：[https://github.com/bytesfly/less/tree/master/less-alibaba/less-testable](https://github.com/bytesfly/less/tree/master/less-alibaba/less-testable)


这里有一个 `WeatherApi` 的接口，通过调用第三方接口查询天气情况，如下：
```java
import com.github.itwild.less.base.http.feign.WeatherExample;
import feign.Param;
import feign.RequestLine;

public interface WeatherApi {

    @RequestLine("GET /api/weather/city/{city_code}")
    WeatherExample.Response query(@Param("city_code") String cityCode);
}
```


`CityWeather` 查询具体城市的天气，如下：
```java
import cn.hutool.core.map.MapUtil;
import com.github.itwild.less.base.http.feign.WeatherExample;
import feign.Feign;
import feign.jackson.JacksonDecoder;
import feign.jackson.JacksonEncoder;

import java.util.HashMap;
import java.util.Map;

public class CityWeather {

    private static final String API_URL = "http://t.weather.itboy.net";

    private static final String BEI_JING = "101010100";
    private static final String SHANG_HAI = "101020100";
    private static final String HE_FEI = "101220101";

    public static final Map<String, String> CITY_CODE = MapUtil.builder(new HashMap<String, String>())
            .put(BEI_JING, "北京市")
            .put(SHANG_HAI, "上海市")
            .put(HE_FEI, "合肥市")
            .build();

    private static WeatherApi weatherApi = Feign.builder()
            .encoder(new JacksonEncoder())
            .decoder(new JacksonDecoder())
            .target(WeatherApi.class, API_URL);

    public String queryShangHaiWeather() {
        WeatherExample.Response response = weatherApi.query(SHANG_HAI);
        return response.getCityInfo().getCity() + ": " + response.getData().getYesterday().getNotice();
    }

    private String queryHeFeiWeather() {
        WeatherExample.Response response = weatherApi.query(HE_FEI);
        return response.getCityInfo().getCity() + ": " + response.getData().getYesterday().getNotice();
    }

    public static String queryBeiJingWeather() {
        WeatherExample.Response response = weatherApi.query(BEI_JING);
        return response.getCityInfo().getCity() + ": " + response.getData().getYesterday().getNotice();
    }

    public static void main(String[] args) {
        CityWeather cityWeather = new CityWeather();

        String shanghai = cityWeather.queryShangHaiWeather();
        String hefei = cityWeather.queryHeFeiWeather();
        String beijing = CityWeather.queryBeiJingWeather();

        System.out.println(shanghai);
        System.out.println(hefei);
        System.out.println(beijing);
    }
```
运行 `main` 方法，输出如下：
```bash
上海市: 不要被阴云遮挡住好心情
合肥市: 不要被阴云遮挡住好心情
北京市: 阴晴之间，谨防紫外线侵扰
```
相信大多数人编写单元测试时，遇到这种依赖第三方资源时，可能就有点反感写单元测试了。
下面看看有了 `testable-mock` 工具，如何编写单元测试？
`CityWeatherTest` 文件如下：
```java
import com.alibaba.testable.core.accessor.PrivateAccessor;
import com.alibaba.testable.core.annotation.TestableMock;
import com.alibaba.testable.processor.annotation.EnablePrivateAccess;
import com.github.itwild.less.base.http.feign.WeatherExample;
import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.*;

@EnablePrivateAccess
public class CityWeatherTest {

    @TestableMock(targetMethod = "query")
    public WeatherExample.Response query(WeatherApi self, String cityCode) {
        WeatherExample.Response response = new WeatherExample.Response();
        // mock天气接口调用返回的结果
        response.setCityInfo(new WeatherExample.CityInfo().setCity(
                CityWeather.CITY_CODE.getOrDefault(cityCode, cityCode)));
        response.setData(new WeatherExample.Data().setYesterday(
                new WeatherExample.Forecast().setNotice("this is from mock")));
        return response;
    }

    CityWeather cityWeather = new CityWeather();

    /**
     * 测试 public方法调用
     */
    @Test
    public void test_public() {
        String shanghai = cityWeather.queryShangHaiWeather();

        System.out.println(shanghai);
        assertEquals("上海市: this is from mock", shanghai);
    }

    /**
     * 测试 private方法调用
     */
    @Test
    public void test_private() {
        String hefei = (String) PrivateAccessor.invoke(cityWeather, "queryHeFeiWeather");

        System.out.println(hefei);
        assertEquals("合肥市: this is from mock", hefei);
    }

    /**
     * 测试 静态方法调用
     */
    @Test
    public void test_static() {
        String beijing = CityWeather.queryBeiJingWeather();

        System.out.println(beijing);
        assertEquals("北京市: this is from mock", beijing);
    }
}
```
运行单元测试，输出如下：
```bash
合肥市: this is from mock
上海市: this is from mock
北京市: this is from mock
```
从运行结果不难发现，依赖第三方接口的 `query` 方法已经被仅仅加了个 `TestableMock` 注解的方法Mock了。也就是说达到了预期的Mock效果，而且代码优雅易读。
## 实现原理


那么，这优雅易读的背后到底隐藏着什么秘密呢？


相信对这方面有些了解的朋友或多或少也猜到了，没错，正是字节码增强技术！！！
```java
package com.alibaba.testable.agent;

import com.alibaba.testable.agent.transformer.TestableClassTransformer;
import java.lang.instrument.Instrumentation;

/**
 * Agent entry, dynamically modify the byte code of classes under testing
 * @author flin
 */
public class PreMain {
    
    public static void premain(String agentArgs, Instrumentation inst) {
        parseArgs(agentArgs);
        inst.addTransformer(new TestableClassTransformer());
    }
}
```
```java
package com.alibaba.testable.agent.handler;

import org.objectweb.asm.ClassReader;
import org.objectweb.asm.ClassWriter;
import org.objectweb.asm.Opcodes;
import org.objectweb.asm.tree.ClassNode;

import java.io.IOException;

/**
 * @author flin
 */
abstract public class BaseClassHandler implements Opcodes {

    public byte[] getBytes(byte[] classFileBuffer) throws IOException {
        ClassReader cr = new ClassReader(classFileBuffer);
        ClassNode cn = new ClassNode();
        cr.accept(cn, 0);
        transform(cn);
        ClassWriter cw = new ClassWriter( 0);
        cn.accept(cw);
        return cw.toByteArray();
    }

    /**
     * Transform class byte code
     * @param cn original class node
     */
    abstract protected void transform(ClassNode cn);

}
```
追一下源码，可见，该Mock工具借助了ASM Core API来修改字节码。上面也提到了，该项目在github上开源出来的时间并不长，核心代码并不多，认真看应该能看懂，主要是有些朋友可能从来没有了解过字节码增强技术。这里推荐美团技术团队的一篇字节码增强技术相关的文章，[https://tech.meituan.com/2019/09/05/java-bytecode-enhancement.html](https://tech.meituan.com/2019/09/05/java-bytecode-enhancement.html)，相信有了这样的基础，回过头来再看看 `TestableMock` 的源码会轻松许多。


本篇博客并不会过多探究字节码增强技术的细节，顶多算是抛砖引玉，目的是让读者知道有这么一个优雅的Mock工具，另外字节码增强技术相当于是一把打开运行时JVM的钥匙，利用它可以动态地对运行中的程序做修改，也可以跟踪JVM运行中程序的状态，这样就能在开发中减少冗余代码，提高开发效率。顺便提一句，我们平时使用的AOP(Cglib就是基于ASM的)也与字节码增强密切相关，它们实质上还是利用各种手段生成符合规范的字节码文件。


虽然这篇不讲修改字节码的操作细节，但我还是想让读者直观地看到增强后的字节码(class文件)是什么样子的，说白了就是到底把我写的代码在运行时修改成了啥？？？于是，我把运行时增强过的字节码重新写入了文件，然后使用反编译工具(拖到IDEA中即可)观察被修改后的源码。


运行时(即增强后的)CityWeatherTest.class反编译后如下：
```java
import com.alibaba.testable.core.accessor.PrivateAccessor;
import com.alibaba.testable.core.annotation.TestableMock;
import com.alibaba.testable.core.util.InvokeRecordUtil;
import com.alibaba.testable.processor.annotation.EnablePrivateAccess;
import com.github.itwild.less.base.http.feign.WeatherExample.CityInfo;
import com.github.itwild.less.base.http.feign.WeatherExample.Data;
import com.github.itwild.less.base.http.feign.WeatherExample.Forecast;
import com.github.itwild.less.base.http.feign.WeatherExample.Response;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

@EnablePrivateAccess
public class CityWeatherTest {
    CityWeather cityWeather = new CityWeather();
    public static CityWeatherTest _testableInternalRef;
    public static CityWeatherTest _testableInternalRef;

    public CityWeatherTest() {
    }

    @TestableMock(
        targetMethod = "query"
    )
    public Response query(WeatherApi var1, String cityCode) {
        InvokeRecordUtil.recordMockInvoke(new Object[]{var1, cityCode}, false);
        InvokeRecordUtil.recordMockInvoke(new Object[]{var1, cityCode}, false);
        Response response = new Response();
        response.setCityInfo((new CityInfo()).setCity((String)CityWeather.CITY_CODE.getOrDefault(cityCode, cityCode)));
        response.setData((new Data()).setYesterday((new Forecast()).setNotice("this is from mock")));
        return response;
    }

    @Test
    public void test_public() {
        _testableInternalRef = this;
        _testableInternalRef = this;
        String shanghai = this.cityWeather.queryShangHaiWeather();
        System.out.println(shanghai);
        Assertions.assertEquals("上海市: this is from mock", shanghai);
    }

    @Test
    public void test_private() {
        _testableInternalRef = this;
        _testableInternalRef = this;
        String hefei = (String)PrivateAccessor.invoke(this.cityWeather, "queryHeFeiWeather", new Object[0]);
        System.out.println(hefei);
        Assertions.assertEquals("合肥市: this is from mock", hefei);
    }

    @Test
    public void test_static() {
        _testableInternalRef = this;
        _testableInternalRef = this;
        String beijing = CityWeather.queryBeiJingWeather();
        System.out.println(beijing);
        Assertions.assertEquals("北京市: this is from mock", beijing);
    }
}
```


运行时(即增强后的)CityWeather.class反编译后如下：
```java
import cn.hutool.core.map.MapUtil;
import com.github.itwild.less.base.http.feign.WeatherExample.Response;
import feign.Feign;
import feign.jackson.JacksonDecoder;
import feign.jackson.JacksonEncoder;
import java.util.HashMap;
import java.util.Map;

public class CityWeather {
    private static final String API_URL = "http://t.weather.itboy.net";
    private static final String BEI_JING = "101010100";
    private static final String SHANG_HAI = "101020100";
    private static final String HE_FEI = "101220101";
    public static final Map<String, String> CITY_CODE = MapUtil.builder(new HashMap()).put("101010100", "北京市").put("101020100", "上海市").put("101220101", "合肥市").build();
    private static WeatherApi weatherApi = (WeatherApi)Feign.builder().encoder(new JacksonEncoder()).decoder(new JacksonDecoder()).target(WeatherApi.class, "http://t.weather.itboy.net");

    public CityWeather() {
    }

    public String queryShangHaiWeather() {
        Response response = CityWeatherTest._testableInternalRef.query(weatherApi, "101020100");
        return response.getCityInfo().getCity() + ": " + response.getData().getYesterday().getNotice();
    }

    private String queryHeFeiWeather() {
        Response response = CityWeatherTest._testableInternalRef.query(weatherApi, "101220101");
        return response.getCityInfo().getCity() + ": " + response.getData().getYesterday().getNotice();
    }

    public static String queryBeiJingWeather() {
        Response response = CityWeatherTest._testableInternalRef.query(weatherApi, "101010100");
        return response.getCityInfo().getCity() + ": " + response.getData().getYesterday().getNotice();
    }

    public static void main(String[] args) {
        CityWeather cityWeather = new CityWeather();
        String shanghai = cityWeather.queryShangHaiWeather();
        String hefei = cityWeather.queryHeFeiWeather();
        String beijing = queryBeiJingWeather();
        System.out.println(shanghai);
        System.out.println(hefei);
        System.out.println(beijing);
    }
}
```
原来，运行时把调用到 `query` 方法的实现都换成了自己Mock的代码。