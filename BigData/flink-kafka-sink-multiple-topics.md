# Flink把数据sink到kafka多个topic


## 需求与场景

上游某业务数据量特别大，进入到kafka一个topic中(当然了这个topic的partition数必然多，有人肯定疑问为什么非要把如此庞大的数据写入到1个topic里，历史留下的问题，现状就是如此庞大的数据集中在一个topic里)。这就需要根据一些业务规则把这个大数据量的topic数据分发到多个(成百上千)topic中，以便下游的多个job去消费自己topic的数据，这样上下游之间的耦合性就降低了，也让下游的job轻松了很多，下游的job只处理属于自己的数据，避免成百上千的job都去消费那个大数据量的topic。数据被分发之后再让下游job去处理 对网络带宽、程序性能、算法复杂性都有好处。

这样一来就需要 这么一个分发程序，把上下游job连接起来。

## 分析与思考

1. Flink中有connect算子，可以连接2个流，在这里1个就是上面数据量庞大的业务数据流，另外1个就是规则流(或者叫做配置流，也就是决定根据什么样的规则分发业务数据)

2. 但是问题来了，根据规则分发好了，如何把这些数据sink到kafka多个(成百上千)topic中呢？

3. 首先想到的就是添加多个sink，每分发到一个topic，就多添加1个addSink操作，这对于如果只是分发到2、3个topic适用的，我看了一下项目中有时候需要把数据sink到2个topic中，同事中就有人添加了2个sink，完全ok，但是在这里要分发到几十个、成百上千个topic，就肯定不现实了，不需要解释吧。

4. sink到kafka中，其实本质上就是用`KafkaProducer`往kafka写数据，那么不知道有没有想起来，用`KafkaProducer`写数据的时候api是怎样的，`public Future<RecordMetadata> send(ProducerRecord<K, V> record);` 显然这里需要一个ProducerRecord对象，再看如何实例化`ProducerRecord`对象，`public ProducerRecord(String topic, V value)`, 也就是说每一个message都指定topic，标明是写到哪一个topic的，而不必说  我们要写入10个不同的topic中，我们就一定new 10 个 KafkaProducer

5. 到上面这一步，如果懂的人就会豁然开朗了，我本来想着可能需要稍微改改flink-connector-kafka实现，让我惊喜的是flink-connector-kafka已经留有了接口，只要实现`KeyedSerializationSchema`这个接口的`String getTargetTopic(T element);`就行

## 代码实现

先看一下`KeyedSerializationSchema`接口的定义，我们知道kafka中存储的都是byte[],所以由我们自定义序列化key、value

```java
/**
 * The serialization schema describes how to turn a data object into a different serialized
 * representation. Most data sinks (for example Apache Kafka) require the data to be handed
 * to them in a specific format (for example as byte strings).
 *
 * @param <T> The type to be serialized.
 */
@PublicEvolving
public interface KeyedSerializationSchema<T> extends Serializable {

	/**
	 * Serializes the key of the incoming element to a byte array
	 * This method might return null if no key is available.
	 *
	 * @param element The incoming element to be serialized
	 * @return the key of the element as a byte array
	 */
	byte[] serializeKey(T element);

	/**
	 * Serializes the value of the incoming element to a byte array.
	 *
	 * @param element The incoming element to be serialized
	 * @return the value of the element as a byte array
	 */
	byte[] serializeValue(T element);

	/**
	 * Optional method to determine the target topic for the element.
	 *
	 * @param element Incoming element to determine the target topic from
	 * @return null or the target topic
	 */
	String getTargetTopic(T element);
}
```
重点来了，实现这个`String getTargetTopic(T element);`就可以决定这个message写入到哪个topic里。

于是 我们可以这么做，拿到业务数据(我们用的是json格式)，然后根据规则分发的时候，就在这条json格式的业务数据里添加一个写到哪个topic的字段，比如说叫`topicKey`，
然后我们实现`getTargetTopic()`方法的时候，从业务数据中取出`topicKey`字段就行了。

实现如下(这里我是用scala写的，java类似)：
```scala
class OverridingTopicSchema extends KeyedSerializationSchema[Map[String, Any]] {

  override def serializeKey(element: Map[String, Any]): Array[Byte] = null

  override def serializeValue(element: Map[String, Any]): Array[Byte] = JsonTool.encode(element) //这里用JsonTool指代json序列化的工具类

  /**
    * kafka message value 根据 topicKey字段 决定 往哪个topic写
    * @param element
    * @return
    */
  override def getTargetTopic(element: Map[String, Any]): String = {
    if (element != null && element.contains(“topicKey”)) {
      element(“topicKey”).toString
    } else null
  }
}
```

之后在new `FlinkKafkaProducer`对象的时候 把上面我们实现的这个`OverridingTopicSchema`传进去就行了。

```java
public FlinkKafkaProducer(
		String defaultTopicId,  // 如果message没有指定写往哪个topic，就写入这个默认的topic
		KeyedSerializationSchema<IN> serializationSchema,//传入我们自定义的OverridingTopicSchema
		Properties producerConfig,
		Optional<FlinkKafkaPartitioner<IN>> customPartitioner,
		FlinkKafkaProducer.Semantic semantic,
		int kafkaProducersPoolSize) {
                    //....
}
```

**至此，我们只需要把上面new 出来的`FlinkKafkaProducer`添加到addSink中就能实现把数据sink到kafka多个(成百上千)topic中。**


*下面简单追踪一下`FlinkKafkaProducer`源码，看看flink-connector-kafka是如何将我们自定义的`KeyedSerializationSchema`作用于最终的`ProducerRecord`*

```java
        /**  这个是用户可自定义的序列化实现
	 * (Serializable) SerializationSchema for turning objects used with Flink into.
	 * byte[] for Kafka.
	 */
	private final KeyedSerializationSchema<IN> schema;

        @Override
	public void invoke(FlinkKafkaProducer.KafkaTransactionState transaction, IN next, Context context) throws FlinkKafkaException {
		checkErroneous();
// 调用我们自己的实现的schema序列化message中的key
		byte[] serializedKey = schema.serializeKey(next);

// 调用我们自己的实现的schema序列化message中的value
		byte[] serializedValue = schema.serializeValue(next);
                
// 调用我们自己的实现的schema取出写往哪个topic
		String targetTopic = schema.getTargetTopic(next);

		if (targetTopic == null) {
// 如果没有指定写往哪个topic，就写往默认的topic
// 这个默认的topic是我们new  FlinkKafkaProducer时候作为第一个构造参数传入（见上面的注释）
			targetTopic = defaultTopicId;
		}

		Long timestamp = null;
		if (this.writeTimestampToKafka) {
			timestamp = context.timestamp();
		}
		ProducerRecord<byte[], byte[]> record;
		int[] partitions = topicPartitionsMap.get(targetTopic);
		if (null == partitions) {
			partitions = getPartitionsByTopic(targetTopic, transaction.producer);
			topicPartitionsMap.put(targetTopic, partitions);
		}
		if (flinkKafkaPartitioner != null) {
			record = new ProducerRecord<>(
				targetTopic, // 这里看到了我们上面一开始分析的ProducerRecord
				flinkKafkaPartitioner.partition(next, serializedKey, serializedValue, targetTopic, partitions),
				timestamp,
				serializedKey,
				serializedValue);
		} else {
			record = new ProducerRecord<>(targetTopic, null, timestamp, serializedKey, serializedValue);
		}
		pendingRecords.incrementAndGet();
		transaction.producer.send(record, callback);
	}
```
