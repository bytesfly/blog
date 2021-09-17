# Kafka常用topic操作命令


topic 工具
[https://cwiki.apache.org/confluence/display/KAFKA/Replication+tools](https://cwiki.apache.org/confluence/display/KAFKA/Replication+tools)

## offset相关
```bash
# 最大offset
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic test_topic --time -1

# 最小offset
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic test_topic --time -2

# offset
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic test_topic
```
## topic相关
```bash 
# 列出当前kafka所有的topic
bin/kafka-topics.sh --zookeeper localhost:2181 --list

# 创建topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --topic test_topic --replication-factor 1 --partitions 1 

bin/kafka-topics.sh --create --zookeeper localhost:2181 --topic test_topic --replication-factor 3 --partitions 10 --config cleanup.policy=compact

bin/kafka-topics.sh --create --zookeeper localhost:2181  --topic test_topic --partitions 1   --replication-factor 1 --config max.message.bytes=64000 --config flush.messages=1

# 查看某topic具体情况
bin/kafka-topics.sh --zookeeper localhost:2181 --describe --topic test_topic

# 修改topic(分区数、特殊配置如compact属性、数据保留时间等)
bin/kafka-topics.sh --zookeeper localhost:2181 --alter --partitions 3  --config cleanup.policy=compact --topic test_topic

# 修改topic(也可以用这种)
bin/kafka-configs.sh --alter --zookeeper localhost:2181 --entity-name test_topic --entity-type topics --add-config cleanup.policy=compact
 
bin/kafka-configs.sh --alter --zookeeper localhost:2181 --entity-name test_topic --entity-type topics --delete-config cleanup.policy
```
## consumer-group相关
```bash
# 查看某消费组(consumer_group)具体消费情况(活跃的消费者以及lag情况等等)
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test_group  --describe

# 列出当前所有的消费组
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --list

# 旧版
bin/kafka-consumer-groups.sh --zookeeper 127.0.0.1:2181 --group test_group --describe
```
## consumer相关
```bash
# 消费数据(从latest消费)
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_topic

# 消费数据(从头开始消费)
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_topic --from-beginning

# 消费数据(最多消费多少条就自动退出消费)
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_topic --max-messages 1

# 消费数据(同时把key打印出来)
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test_topic --property print.key=true

# 旧版
bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic test_topic
```
## producer相关
```bash
# 生产数据
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test_topic

# 生产数据(写入带有key的message)
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test_topic --property "parse.key=true" --property "key.separator=:"
```
## producer-golang
```bash
# golang实现的kafka客户端
https://github.com/Shopify/sarama/tree/master/tools

# Minimum invocation
kafka-console-producer -topic=test -value=value -brokers=kafka1:9092

# It will pick up a KAFKA_PEERS environment variable
export KAFKA_PEERS=kafka1:9092,kafka2:9092,kafka3:9092
kafka-console-producer -topic=test -value=value

# It will read the value from stdin by using pipes
echo "hello world" | kafka-console-producer -topic=test

# Specify a key:
echo "hello world" | kafka-console-producer -topic=test -key=key

# Partitioning: by default, kafka-console-producer will partition as follows:
# - manual partitioning if a -partition is provided
# - hash partitioning by key if a -key is provided
# - random partioning otherwise.
#
# You can override this using the -partitioner argument:
echo "hello world" | kafka-console-producer -topic=test -key=key -partitioner=random

# Display all command line options
kafka-console-producer -help
```