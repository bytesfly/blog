# kafka启动报错"A broker is already registered on the path /brokers/ids/1"排查

## 问题
kafka挂掉后，启动报错日志如下
```sh
[2020-03-19 17:50:58,123] FATAL Fatal error during KafkaServerStartable startup. Prepare to shutdown (kafka.server.KafkaServerStartable)
java.lang.RuntimeException: A broker is already registered on the path /brokers/ids/1. This probably indicates that you either have configured a brokerid that is already in use, or else you have shutdown this broker and restarted it faster than the zookeeper timeout so it appears to be re-registering.
        at kafka.utils.ZkUtils.registerBrokerInZk(ZkUtils.scala:408)
        at kafka.utils.ZkUtils.registerBrokerInZk(ZkUtils.scala:394)
        at kafka.server.KafkaHealthcheck.register(KafkaHealthcheck.scala:71)
        at kafka.server.KafkaHealthcheck.startup(KafkaHealthcheck.scala:51)
        at kafka.server.KafkaServer.startup(KafkaServer.scala:269)
        at kafka.server.KafkaServerStartable.startup(KafkaServerStartable.scala:39)
        at kafka.Kafka$.main(Kafka.scala:67)
        at kafka.Kafka.main(Kafka.scala)
[2020-03-19 17:50:58,123] INFO [Kafka Server 1], shutting down (kafka.server.KafkaServer)
```
## 分析
从`This probably indicates that you either have configured a brokerid that is already in use`提示可知，zookeeper中可能已经注册了此broker id，正常情况下，你应该不会启动两个相同broker id的kafka server(除非你没注意弄错了使得两个kafka server用了相同的broker id)

于是我用`$KAFKA`安装包下带有的`zookeeper` client 连接了zk server，看一下kafka broker的注册情况

```sh
$KAFKA/bin/zookeeper-shell.sh 192.168.0.1:2181 ls /brokers/ids
```

执行后显示
```sh
Connecting to 192.168.0.1:2181

WATCHER::

WatchedEvent state:SyncConnected type:None path:null
[1, 2, 3]
```
然而实际情况是，broker id为1的kafka server并没有启动起来。原因是这台机器之前因为卡死被`物理重启`，kafka broker没有正常下线，zk上还保留着它的broker id。

## 解决方案
找到原因后，解决就很简单了，把注册在`zookeeper`上的这个broker id `delete`掉就行了

```sh
$KAFKA/bin/zookeeper-shell.sh 192.168.0.1:2181 delete  /brokers/ids/1
```

然后再启动观察日志就正常了。
