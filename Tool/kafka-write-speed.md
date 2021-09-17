# 使用脚本+kafka自带命令行工具 统计数据写入kafka速率

## 思路

每隔一段时间(比如说10秒)统计一次某`topic`的所有`partition`的最大`offset`值之和，这便是该`topic`的message总数。
然后除以间隔时间就可以粗略但方便得出  某`topic`的数据增长速率(即相应程序写kafka的速率)

[`Kafka常用topic操作命令汇总`](https://www.cnblogs.com/itwild/p/12287850.html) 中有统计最大offset命令

```bash
# 最大offset
bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list localhost:9092 --topic test_topic --time -1
```

## shell实现

注意：
1. 第一次打印出来的speed请忽略(为了shell更加简单方便没有特殊处理第1次统计的情况)
2. 该脚本需要放入到kafka程序安装根目录，或者把bin/kafka-run-class.sh文件写成绝对路径

```bash
#!/bin/sh

brokers="localhost:9092"
topic="test_topic"

last=0
now=0
speed=0

while :
do
    echo "-------------"
    last=$now
    now=$(bin/kafka-run-class.sh kafka.tools.GetOffsetShell --broker-list ${brokers} --topic ${topic} --time -1 |  awk -F ":" '{sum+=$NF} END {print sum}')
    let speed=(now-last)/10
    echo "now is $now, speed is $speed"
    sleep 10
done

```