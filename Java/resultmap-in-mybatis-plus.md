# MyBatis-Plus中如何使用ResultMap

> [MyBatis-Plus](https://baomidou.com/) （简称`MP`）是一个`MyBatis`的增强工具，在`MyBatis`的基础上只做增强不做改变，为简化开发、提高效率而生。

`MyBatis-Plus`对`MyBatis`基本零侵入，完全可以与`MyBatis`混合使用，这点很赞。

在涉及到关系型数据库增删查改的业务时，我比较喜欢用`MyBatis-Plus`，开发效率极高。具体的使用可以参考官网，或者自己上手摸索感受一下。

下面简单总结一下在`MyBatis-Plus`中如何使用`ResultMap`。

## 问题说明

先看个例子：

有如下两张表：
```sql
create table tb_book
(
    id     bigint primary key,
    name   varchar(32),
    author varchar(20)
);

create table tb_hero
(
    id      bigint primary key,
    name    varchar(32),
    age     int,
    skill   varchar(32),
    bid bigint
);
```
其中，`tb_hero`中的`bid`关联`tb_book`表的`id`。

下面先看`Hero`实体类的代码，如下：

```java
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableId;
import com.baomidou.mybatisplus.annotation.TableName;
import com.fasterxml.jackson.annotation.JsonInclude;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;

@Getter
@Setter
@NoArgsConstructor
@TableName("tb_hero")
@JsonInclude(JsonInclude.Include.NON_NULL)
public class Hero {

    @TableId("id")
    private Long id;

    @TableField(value = "name", keepGlobalFormat = true)
    private String name;

    @TableField(value = "age", keepGlobalFormat = true)
    private Integer age;

    @TableField(value = "skill", keepGlobalFormat = true)
    private String skill;

    @TableField(value = "bid", keepGlobalFormat = true)
    private Long bookId;

    // *********************************
    // 数据库表中不存在以下字段(表join时会用到)
    // *********************************

    @TableField(value = "book_name", exist = false)
    private String bookName;

    @TableField(value = "author", exist = false)
    private String author;
}
```

注意了，我特地把`tb_hero`表中的`bid`字段映射成实体类`Hero`中的`bookId`属性。

- 测试`BaseMapper`中内置的`insert()`方法或者`IService`中的`save()`方法

`MyBatis-Plus`打印出的`SQL`为：

```bash
==> Preparing: INSERT INTO tb_hero ( id, "name", "age", "skill", "bid" ) VALUES ( ?, ?, ?, ?, ? )
==> Parameters: 1589788935356416(Long), 阿飞(String), 18(Integer), 天下第一快剑(String), 1(Long)
```

没毛病， `MyBatis-Plus`会根据`@TableField`指定的映射关系，生成对应的`SQL`。

- 测试`BaseMapper`中内置的`selectById()`方法或者`IService`中的`getById()`方法

`MyBatis-Plus`打印出的`SQL`为：

```bash
==> Preparing: SELECT id,"name","age","skill","bid" AS bookId FROM tb_hero WHERE id=?
==> Parameters: 1(Long)
```

也没毛病，可以看到生成的`SELECT`中把`bid`做了别名`bookId`。


- 测试自己写的SQL

比如现在我想连接`tb_hero`与`tb_book`这两张表，如下：
```java
@Mapper
@Repository
public interface HeroMapper extends BaseMapper<Hero> {
    @Select({"SELECT tb_hero.*, tb_book.name as book_name, tb_book.author" +
            " FROM tb_hero" +
            " LEFT JOIN tb_book" +
            " ON tb_hero.bid = tb_book.id" +
            " ${ew.customSqlSegment}"})
    IPage<Hero> pageQueryHero(@Param(Constants.WRAPPER) Wrapper<Hero> queryWrapper,
                              Page<Hero> page);
}
```

查询`MyBatis-Plus`打印出的`SQL`为：

```bash
==> Preparing: SELECT tb_hero.*, tb_book.name AS book_name, tb_book.author FROM tb_hero LEFT JOIN tb_book ON tb_hero.bid = tb_book.id WHERE ("bid" = ?) ORDER BY id ASC LIMIT ? OFFSET ?
==> Parameters: 2(Long), 1(Long), 1(Long)
```

SQL没啥问题，过滤与分页也都正常，但是此时你会发现`bookId`属性为`null`，如下：

![](https://img2020.cnblogs.com/blog/1546632/202111/1546632-20211122141925905-1394428265.png)

为什么呢？

调用`BaseMapper`中内置的`selectById()`方法并没有出现这种情况啊？？？

回过头来再对比一下在`HeroMapper`中自己定义的查询与`MyBatis-Plus`自带的`selectById()`有啥不同，还记得上面的刚刚的测试吗，生成的SQL有啥不同？

原来，`MyBatis-Plus`为`BaseMapper`中内置的方法生成SQL时，会把`SELECT`子句中`bid`做别名`bookId`，而自己写的查询`MyBatis-Plus`并不会帮你修改`SELECT`子句，也就导致`bookId`属性为`null`。

## 解决方法

- 方案一：表中的字段与实体类的属性严格保持一致(字段有下划线则属性用驼峰表示)

在这里就是`tb_hero`表中的`bid`字段映射成实体类`Hero`中的`bid`属性。这样当然可以解决问题，但不是本篇讲的重点。

- 方案二：把自己写的`SQL`中`bid`做别名`bookId`

- 方案三：使用`@ResultMap`，这是此篇的重点

在`@TableName`设置`autoResultMap = true`

```java
@TableName(value = "tb_hero", autoResultMap = true)
public class Hero {
    
}
```

然后在自定义查询中添加`@ResultMap`注解，如下：

```java
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.ResultMap;
import org.apache.ibatis.annotations.Select;
import org.springframework.stereotype.Repository;

@Mapper
@Repository
public interface HeroMapper extends BaseMapper<Hero> {
    @ResultMap("mybatis-plus_Hero")
    @Select({"SELECT tb_hero.*, tb_book.name as book_name, tb_book.author" +
            " FROM tb_hero" +
            " LEFT JOIN tb_book" +
            " ON tb_hero.bid = tb_book.id" +
            " ${ew.customSqlSegment}"})
    IPage<Hero> pageQueryHero(@Param(Constants.WRAPPER) Wrapper<Hero> queryWrapper,
                              Page<Hero> page);
}
```
这样，也能解决问题。

下面简单看下源码，`@ResultMap("mybatis-plus_实体类名")`怎么来的。

详情见： `com.baomidou.mybatisplus.core.metadata.TableInfo#initResultMapIfNeed()`

```java
/**
 * 自动构建 resultMap 并注入(如果条件符合的话)
 */
void initResultMapIfNeed() {
    if (autoInitResultMap && null == resultMap) {
        String id = currentNamespace + DOT + MYBATIS_PLUS + UNDERSCORE + entityType.getSimpleName();
        List<ResultMapping> resultMappings = new ArrayList<>();
        if (havePK()) {
            ResultMapping idMapping = new ResultMapping.Builder(configuration, keyProperty, StringUtils.getTargetColumn(keyColumn), keyType)
                .flags(Collections.singletonList(ResultFlag.ID)).build();
            resultMappings.add(idMapping);
        }
        if (CollectionUtils.isNotEmpty(fieldList)) {
            fieldList.forEach(i -> resultMappings.add(i.getResultMapping(configuration)));
        }
        ResultMap resultMap = new ResultMap.Builder(configuration, id, entityType, resultMappings).build();
        configuration.addResultMap(resultMap);
        this.resultMap = id;
    }
}
```
注意看上面的字符串`id`的构成，你应该可以明白。

思考： 这种方式的`ResultMap`默认是强绑在一个`@TableName`上的，如果是某个聚合查询或者查询的结果并非对应一个真实的表怎么办呢？有没有更优雅的方式？

## 自定义@AutoResultMap注解

基于上面的思考，我做了下面简单的实现：

- 自定义@AutoResultMap注解

```java
import java.lang.annotation.*;

/**
 * 使用@AutoResultMap注解的实体类
 * 自动生成{auto.mybatis-plus_类名}为id的resultMap
 * {@link MybatisPlusConfig#initAutoResultMap()}
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface AutoResultMap {

}
```

- 启动时扫描@AutoResultMap注解的实体类

```java
package com.bytesfly.mybatis.config;

import cn.hutool.core.util.ClassUtil;
import cn.hutool.core.util.ReflectUtil;
import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.core.metadata.TableInfo;
import com.baomidou.mybatisplus.core.metadata.TableInfoHelper;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import com.baomidou.mybatisplus.extension.toolkit.JdbcUtils;
import com.bytesfly.mybatis.annotation.AutoResultMap;
import lombok.extern.slf4j.Slf4j;
import org.apache.ibatis.builder.MapperBuilderAssistant;
import org.mybatis.spring.SqlSessionTemplate;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import javax.annotation.PostConstruct;
import java.util.Set;

/**
 * 可添加一些插件
 */
@Configuration
@EnableTransactionManagement(proxyTargetClass = true)
@MapperScan(basePackages = "com.bytesfly.mybatis.mapper")
@Slf4j
public class MybatisPlusConfig {

    @Autowired
    private SqlSessionTemplate sqlSessionTemplate;

    /**
     * 分页插件(根据jdbcUrl识别出数据库类型, 自动选择适合该方言的分页插件)
     * 相关使用说明: https://baomidou.com/guide/page.html
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor(DataSourceProperties dataSourceProperties) {

        String jdbcUrl = dataSourceProperties.getUrl();
        DbType dbType = JdbcUtils.getDbType(jdbcUrl);

        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(dbType));
        return interceptor;
    }

    /**
     * @AutoResultMap注解的实体类自动构建resultMap并注入
     */
    @PostConstruct
    public void initAutoResultMap() {
        try {
            log.info("--- start register @AutoResultMap ---");

            String namespace = "auto";

            String packageName = "com.bytesfly.mybatis.model.db.resultmap";
            Set<Class<?>> classes = ClassUtil.scanPackageByAnnotation(packageName, AutoResultMap.class);

            org.apache.ibatis.session.Configuration configuration = sqlSessionTemplate.getConfiguration();

            for (Class clazz : classes) {
                MapperBuilderAssistant assistant = new MapperBuilderAssistant(configuration, "");
                assistant.setCurrentNamespace(namespace);
                TableInfo tableInfo = TableInfoHelper.initTableInfo(assistant, clazz);

                if (!tableInfo.isAutoInitResultMap()) {
                    // 设置 tableInfo的autoInitResultMap属性 为 true
                    ReflectUtil.setFieldValue(tableInfo, "autoInitResultMap", true);
                    // 调用 tableInfo#initResultMapIfNeed() 方法，自动构建 resultMap 并注入
                    ReflectUtil.invoke(tableInfo, "initResultMapIfNeed");
                }
            }

            log.info("--- finish register @AutoResultMap ---");
        } catch (Throwable e) {
            log.error("initAutoResultMap error", e);
            System.exit(1);
        }
    }
}
```

关键代码其实没有几行，耐心看下应该不难懂。

- 使用@AutoResultMap注解

还是用例子来说明更直观。

下面是一个聚合查询：

```java
@Mapper
@Repository
public interface BookMapper extends BaseMapper<Book> {

    @ResultMap("auto.mybatis-plus_BookAgg")
    @Select({"SELECT tb_book.id, max(tb_book.name) as name, array_agg(distinct tb_hero.id order by tb_hero.id asc) as hero_ids" +
            " FROM tb_hero" +
            " INNER JOIN tb_book" +
            " ON tb_hero.bid = tb_book.id" +
            " GROUP BY tb_book.id"})
    List<BookAgg> agg();
}
```

其中`BookAgg`的定义如下，在实体类上使用了`@AutoResultMap`注解：

```java
@Getter
@Setter
@NoArgsConstructor
@AutoResultMap
public class BookAgg {

    @TableId("id")
    private Long bookId;

    @TableField("name")
    private String bookName;

    @TableField("hero_ids")
    private Object heroIds;
}
```

完整代码见： [https://github.com/bytesfly/springboot-demo/tree/master/springboot-mybatis-plus](https://github.com/bytesfly/springboot-demo/tree/master/springboot-mybatis-plus)
