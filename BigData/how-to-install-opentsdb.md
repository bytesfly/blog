# centos7安装部署opentsdb2.4.0


## 写在前面

最近因为项目需要在读opentsdb的一部分源码，后面会做个小结分享出来。本人是不大喜欢写这种安装部署的文章，考虑到opentsdb安装部署对于初次接触者来说不太友好，另外对公司做测试的同事可能有些帮助作用，方便他们快速安装部署，就把OpenTSDB 2.4.0安装部署文档写在这里。

对于opentsdb是什么，应用领域这里就不说了，不了解的请看官网`http://opentsdb.net/`。

这里只提一点，`opentsdb`的后端数据存储依赖于`HBase`。

所以安装步骤就分为了三步 (我们也可以类比安装传统的依赖关系型数据库比如mysql作为后端数据存储的软件)

1. 安装HBase (类比传统软件我们要安装mysql)

2. 创建表结构 (类比我们在mysql中创建database以及table)

3. 安装配置并启动opentsdb (类比一些springboot的应用)

## 安装HBase

如果已经有HBase环境了，那么请跳过这一步(大多数使用`HBase`集群环境应该都是用`CDH`管理)

官网地址： `http://hbase.apache.org/`

其实`HBase`的存储又是依赖`HDFS`，当然了如果只是本地测试用，可以直接用`本地文件系统`代替`HDFS`，这样就不需要部署一套`HDFS`集群了

搭建`Standalone HBase`，官方文档`http://hbase.apache.org/book.html#quickstart`，一步一步都有，请仔细读

这里简单梳理一下关键的地方：

- `conf/hbase-env.sh`文件

```bash
# The java implementation to use.
export JAVA_HOME=/usr/jdk64/jdk1.8.0_112
```
- `conf/hbase-site.xml`文件
```xml
<configuration>
  <property>
    <!-- hbase实际存放数据地方，这里是本地文件系统，生产环境一般HDFS地址，例如hdfs://namenode.example.org:8020/hbase -->
    <name>hbase.rootdir</name>
    <value>file:///home/itwild/hbase</value>
  </property>
  <property>
    <!--
指定zookeeper的data目录
目前hbase需要依赖zookeeper，HBase通过Zookeeper来做Master的高可用、RegionServer的监控、元数据的入口以及
集群配置的维护等工作
因为是Standalone，为了降低部署复杂度，启动的时候也会启zookeeper，指定zk data存储目录，实际使用大多用单独的zk集群，一般不使用内置的zk
    -->
    <name>hbase.zookeeper.property.dataDir</name>
    <value>/home/itwild/zookeeper</value>
  </property>
</configuration>
```
然后执行`bin/start-hbase.sh`, 启动成功了`jps`命令可以看到`HMaster`进程。

在standalone模式下，虽然看到的是一个`JVM`实例，实际上启了`HMaster`、`HRegionServer`、`ZooKeeper`

启动成功，可打开HBase Web UI, `http://localhost:16010`

## 在HBase中创建表结构

1. 执行`bin/hbase shell`进入一个交互的界面 (这里我们也可以类比执行`mysql -uXXX -pXXX`后进入)

2. 在交互界面里我们依次执行下面4条建表(table)语句
```bash
# opentsdb中那些metric数据就存在这张表中
# 这张表数据会很大，考虑到读写效率，我们注意到这张表就一个列族
create 'tsdb',{NAME => 't', VERSIONS => 1, BLOOMFILTER => 'ROW'}

# opentsdb中建立metric name、tagK、tagV字面量与uid一一对应的表
# opentsdb不会存储实际的字符串字面值
# 比如system.cpu.util的metric，会将system.cpu.util转化为id(默认自增，后面介绍部分源码的时候会有讲到)后，存入HBase
# 这张表有id、name两个列族，可通过id找到name，也可以通过name找到id
create 'tsdb-uid',{NAME => 'id', BLOOMFILTER => 'ROW'},{NAME => 'name', BLOOMFILTER => 'ROW'}

# 下面两张表暂时可不必太关心，先创建出来就好
create 'tsdb-tree',{NAME => 't', VERSIONS => 1, BLOOMFILTER => 'ROW'}
create 'tsdb-meta',{NAME => 'name', BLOOMFILTER => 'ROW'}
```

注意一下，这里的建表语句我有意把`压缩`(`COMPRESSION`)选项去掉了，因为存储用的是本地文件系统，有些压缩可能是不支持的，生产环境使用`HDFS`的建表语句可能是这样的
```bash
create 'tsdb',{NAME => 't', VERSIONS => 1, BLOOMFILTER => 'ROW', COMPRESSION => 'SNAPPY'}

create 'tsdb-uid',{NAME => 'id', BLOOMFILTER => 'ROW', COMPRESSION => 'SNAPPY'},{NAME => 'name', BLOOMFILTER => 'ROW', COMPRESSION => 'SNAPPY'}

create 'tsdb-tree',{NAME => 't', VERSIONS => 1, BLOOMFILTER => 'ROW', COMPRESSION => 'SNAPPY'}

create 'tsdb-meta',{NAME => 'name', BLOOMFILTER => 'ROW', COMPRESSION => 'SNAPPY'}
```

-  在hbase shell的交互界面中执行`list`，即可看到上面创建的4张表
```bash
hbase(main):004:0> list
TABLE
tsdb
tsdb-meta
tsdb-tree
tsdb-uid
```
## 安装配置并启动opentsdb

下载地址：`https://github.com/OpenTSDB/opentsdb/releases`

这里`centos7`系统，选择下载`opentsdb-2.4.0.noarch.rpm`包

- 执行`yum -y  localinstall opentsdb-2.4.0.noarch.rpm`

如果这里安装报错，可能需要`vi /usr/bin/yum`临时改一下python解析器的版本`#!/usr/bin/python2.7`，
改过之后安装过程中又有报错，可能需要`vi /usr/libexec/urlgrabber-ext-down`同样临时改一下python解析器的版本`#!/usr/bin/python2.7`

- 把opentsdb的服务注册为系统服务，即可以用`systemctl status/start/stop/restart opentsdb`来查看控制

`vi  /usr/lib/systemd/system/opentsdb.service`添加以下内容
```bash
[Unit]

Description=OpenTSDB Service

[Service]

Type=forking

PrivateTmp=yes

ExecStart=/usr/share/opentsdb/etc/init.d/opentsdb start

ExecStop=/usr/share/opentsdb/etc/init.d/opentsdb stop

Restart=on-abort
```
然后你就发现可以用`systemctl status opentsdb`了，不过现在服务还是`dead`状态

> 注意一下，默认opentsdb配置文件目录：/etc/opentsdb/opentsdb.conf，默认opentsdb日志目录：/var/log/opentsdb

- 修改`/etc/opentsdb/opentsdb.conf`配置文件
```bash
tsd.network.port = 4242
tsd.http.staticroot = /usr/share/opentsdb/static/
tsd.http.cachedir = /tmp/opentsdb
tsd.core.auto_create_metrics = true
tsd.core.plugin_path = /usr/share/opentsdb/plugins
# zookeeper的地址，即hbase依赖的zookeeper的地址，localhost:2181,localhost:2182,localhost:2183
tsd.storage.hbase.zk_quorum = localhost:2181
tsd.storage.fix_duplicates = true
tsd.http.request.enable_chunked = true
tsd.http.request.max_chunk = 4096000
tsd.storage.max_tags = 16
# 这里看到了我们上面在hbase中创建的4张表
tsd.storage.hbase.data_table = tsdb
tsd.storage.hbase.uid_table = tsdb-uid
tsd.storage.hbase.tree_table = tsdb-tree
tsd.storage.hbase.meta_table = tsdb-meta
# 下面几个配置项到部分源码解析的时候会有介绍，暂时可以先忽略
# tsd.query.skip_unresolved_tagvs = true
# hbase.rpc.timeout = 120000
```
- 启动opentsdb，`systemctl start opentsdb`，成功的话，就可以打开opentsdb的界面了`http://localhost:4242/`

至此，有关opentsdb的安装部署就完成了。后面我会结合opentsdb部分源码分享一些探究性问题。祝君好运！
