# MongoDB -> kafka 高性能实时同步方案

## 写这篇博客的目的
让更多的人了解 阿里开源的MongoShake可以很好满足mongodb到kafka高性能高可用实时同步需求（项目地址：`https://github.com/alibaba/MongoShake`，下载地址：`https://github.com/alibaba/MongoShake/releases`）。至此博客就结束了，你可以愉快地啃这个项目了。还是一起来看一下官方的描述：
> MongoShake is a universal data replication platform based on MongoDB's oplog. Redundant replication and active-active replication are two most important functions. 基于mongodb oplog的集群复制工具，可以满足迁移和同步的需求，进一步实现灾备和多活功能。

## 没有标题的标题

哈哈，有兴趣听我啰嗦的可以往下。最近，有个实时增量采集mongodb数据(`数据量在每天10亿条左右`)的需求，需要先调研一下解决方案。我分别百度、google了`mongodb kafka sync 同步 采集 实时`等 关键词，写这篇博客的时候排在最前面的当属`kafka-connect`(官方有实现`https://github.com/mongodb/mongo-kafka`，其实也有非官方的实现)那一套方案，我对kafka-connect相对熟悉一点(不熟悉的话估计编译部署都要花好一段时间)，没测之前就感觉可能不满足我的采集性能需求，测下来果然也是不满足需求。后来，也看到了`https://github.com/rwynn/route81`，编译部署也较为麻烦，同样不满足采集性能需求。我搜索东西的时候一般情况下不会往下翻太多，没找到所需的，大多会尝试换关键词(包括中英文)搜搜，`这次可能也提醒我下次要多往下找找，说不定有些好东西未必排在最前面几个`。

之后在github上搜`in:readme mongodb kafka sync`，让我眼前一亮。

![github上搜索mongodb、kafka、sync关键词结果](https://img2018.cnblogs.com/blog/1546632/202002/1546632-20200218230148278-198252635.png)

点进去快速读了一下readme，正是我想要的(后面自己实际测下来确实高性能、高可用，满足我的需求)，官方也提供了MongoShake的[性能测试报告](https://github.com/alibaba/MongoShake/wiki/MongoShake-Performance-Document)。

这篇博客不讲(`也很大可能是笔者技术太渣，无法参透领会(●´ω｀●)`)MongoShake的架构、原理、实现，如何高性能的，如何高可用的等等。就一个目的，希望其他朋友在搜索`mongodb kafka`时候，`MongoShake`的解决方案可以排在最前面。

## 初次使用MongoShake值得注意的地方

### 数据处理流程
***v2.2.1之前的MongoShake版本处理数据的流程：***

MongoDB(数据源端，待同步的数据)
`-->`MongoShake(对应的是`collector.linux`进程，作用是采集)
`-->`Kafka(raw格式，未解析的带有header+body的数据)
`-->`receiver(对应的是`receiver.linux`进程，作用是解析，这样下游组件就能拿到比如解析好的一条一条的json格式的数据)
`-->`下游组件(拿到mongodb中的数据用于自己的业务处理)

***v2.2.1之前MongoShake的版本解析入kafka，需要分别启collector.linux和receiver.linux进程，而且receiver.linux需要自己根据你的业务逻辑填充完整,然后编译出来，默认只是把解析出来的数据打个log而已***

`src/mongoshake/receiver/replayer.go`中的代码如图：

![需要自己填充receiver逻辑的地方](https://img2018.cnblogs.com/blog/1546632/202002/1546632-20200219005031066-1197176266.png)

详情见：[https://github.com/alibaba/MongoShake/wiki/FAQ#q-how-to-connect-to-different-tunnel-except-direct](https://github.com/alibaba/MongoShake/wiki/FAQ#q-how-to-connect-to-different-tunnel-except-direct)

***v2.2.1版本MongoShake的`collector.conf`有一个配置项`tunnel.message`***
```sh
# the message format in the tunnel, used when tunnel is kafka.
# "raw": batched raw data format which has good performance but encoded so that users
# should parse it by receiver.
# "json": single oplog format by json.
# "bson": single oplog format by bson.
# 通道数据的类型，只用于kafka和file通道类型。
# raw是默认的类型，其采用聚合的模式进行写入和
# 读取，但是由于携带了一些控制信息，所以需要专门用receiver进行解析。
# json以json的格式写入kafka，便于用户直接读取。
# bson以bson二进制的格式写入kafka。
tunnel.message = json
```
- 如果选择的`raw`格式，那么数据处理流程和上面之前的一致(MongoDB->MongoShake->Kafka->receiver->下游组件)
- 如果选择的是`json`、`bson`,处理流程为MongoDB->MongoShake->Kafka->下游组件

**v2.2.1版本设置为json处理的优点就是把以前需要由receiver对接的格式，改为直接对接，从而少了一个receiver，也不需要用户额外开发，降低开源用户的使用成本。**

简单总结一下就是：
***raw格式能够最大程度的提高性能，但是需要用户有额外部署receiver的成本。json和bson格式能够降低用户部署成本，直接对接kafka即可消费，相对于raw来说，带来的性能损耗对于大部分用户是能够接受的。***

### 高可用部署方案

我用的是v2.2.1版本，高可用部署非常简单。`collector.conf`开启master的选举即可：
```sh
# high availability option.
# enable master election if set true. only one mongoshake can become master
# and do sync, the others will wait and at most one of them become master once
# previous master die. The master information stores in the `mongoshake` db in the source
# database by default.
# 如果开启主备mongoshake拉取同一个源端，此参数需要开启。
master_quorum = true

# checkpoint存储的地址，database表示存储到MongoDB中，api表示提供http的接口写入checkpoint。
context.storage = database
```
同时我checkpoint的存储地址默认用的是database，会默认存储在`mongoshake`这个db中。我们可以查询到checkpoint记录的一些信息。
```sh
rs0:PRIMARY> use mongoshake
switched to db mongoshake
rs0:PRIMARY> show collections;
ckpt_default
ckpt_default_oplog
election
rs0:PRIMARY> db.election.find()
{ "_id" : ObjectId("5204af979955496907000001"), "pid" : 6545, "host" : "192.168.31.175", "heartbeat" : NumberLong(1582045562) }
```
我在192.168.31.174，192.168.31.175，192.168.31.176上总共启了3个MongoShake实例，可以看到现在工作的是192.168.31.175机器上进程。自测过程，高速往mongodb写入数据，手动kill掉192.168.31.175上的collector进程，等192.168.31.174成为master之后，我又手动kill掉它，最终只保留192.168.31.176上的进程工作，最后统计数据发现，有重采数据现象，猜测有实例还没来得及checkpoint就被kill掉了。