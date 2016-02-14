---
layout: post
title: kafka学习记录
description: 接触一个新的消息中间件
category: blog
---

顺序读写文件比随机读写内存都要快

## kafka基本使用

考虑一下场景:

### 启动三个broker

启动命令如下：

```
kafka-server-start.sh /usr/local/etc/kafka/server1.properties
```

broker的简单配置如下:

```
broker.id=0
port=9092
log.dirs=/usr/local/var/lib/kafka-logs
zookeeper.connect=localhost:2181
```

broker的启动配置文件有很多参数，这里先简单关注4个参数:

* id: 每个broker的id唯一
* port: 每个broker的监听端口号唯一
* log目录: 给不同的broker分配不同的Log存储位置，kafka中所有的数据都统称为log
* zookeeper: broker启动时必须向zookeeper报告，由zookeeper统一调度管理

### 注册一个topic

注册命令如下：

```
kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 3 --partition 1 --topic my-replicated-topi
```

这是一个注册有2个备份节点和1个partition的topic, 默认必须有1个replication-factor和1个partition

注意以下几点

* 注册时必须指定zookeeper地址和端口号，说明是向zookeeper汇报topic信息，而不是broker
* 注册时可以指定备份实例数目和分区数目
* --topic指定topic名字, 相同的topic不允许创建两次
* replication的数量不能大于broker的数量

### 获取一个topic的描述

```
kafka-topics.sh --describe --zookeeper localhost:2181 --topic my_test

Topic:my_test1 PartitionCount:1 ReplicationFactor:1 Configs:
  Topic: my_test1 Partition: 0 Leader: 0 LeaderReplicas: 0 LeaderIsr: 0
```

### producer

启动一个producer生产者

```
kafka-console-producer.sh --broker-list localhost:9092 --topic my_test1
```

这里需要指定broker的位置而不是zookeeper的位置

### consumer

注册一个消费者

```
kafka-console-consumer.sh --zookeeper localhost:2181 --topic my_test1
```

### partition

create topic时可以指定多个partition
每个topic包含一个或者多个partition
当一个topic有多个partition时，向topic写入消息时，只有partition被写入

```
Topic:my_test6PartitionCount:2.ReplicationFactor:2.Configs:
Topic: my_test6PartitionCountPartition: 0 Leader: 1 Replicas: 1,2 Isr: 1,2
Topic: my_test6PartitionCountPartition: 1 Leader: 2 Replicas: 2,0 LeaderIsr: 2,0
```

从上面可以看出：当有多个partition时，每个partition的replication都是随机的

多个partition可以做负载均衡，避免所有的数据都落在一个partition上,它会根据消息的key
和partition策略将消息落在指定的partition上

在发送一条消息时，可以指定这条消息的key，Producer根据这个key和Partition机制来判断应该将这条消息发送到哪个Parition。
Paritition机制可以通过指定Producer的paritition. class这一参数来指定，该class必须实现kafka.producer.Partitioner接口。
本例中如果key可以被解析为整数则将对应的整数与Partition总数取余，该消息会被发送到该数对应的Partition。
（每个Parition都会有个序号,序号从0开始）

创建partition时，可以要求partition的数量比broker要多

partition一般都是按照机器分配，尽量分散；当partition数量比broker数量多时才会出现一个broker中有一个topic的不同partition
，大部分情况下,不同的partition是分布在不同的broker上的

### 消息单播和多播模型

使用Consumer high level API时，同一Topic的一条消息只能被同一个Consumer Group内的一个Consumer消费，
但多个Consumer Group可同时消费这一消息。

这是Kafka用来实现一个Topic消息的广播（发给所有的Consumer）和单播（发给某一个Consumer）的手段。
一个Topic可以对应多个Consumer Group。如果需要实现广播，只要每个Consumer有一个独立的Group就可以了。
要实现单播只要所有的Consumer在同一个Group里。用Consumer Group还可以将Consumer进行自由的分组而不需要多次发送消息到不同的Topic.

kafka consumer group可以理解为抽象层次更高的kafka consumer, 使用group机制来管理消息的传播机制

### 高效

Kafka会为每一个Consumer Group保留一些metadata信息——当前消费的消息的position，也即offset。
在同一个group中的consumer会共享该metadata
这个offset由Consumer控制。正常情况下Consumer会在消费完一条消息后递增该offset。
当然，Consumer也可将offset设成一个较小的值，重新消费一些消息。因为offet由Consumer控制，所以Kafka broker是无状态的，
它不需要标记哪些消息被哪些消费过，也不需要通过broker去保证同一个Consumer Group只有一个Consumer能消费某一条消息，
因此也就不需要锁机制，这也为Kafka的高吞吐率提供了有力保障。

### kafka consumer高级和低级接口



### MessageStream

## 测试

1. 测试是否可以向随意一个broker写数据

启动多个broker, 新建一个无备份，只有一个partition的topic
启动一个consumer，分别启动多个producer，每个producer可以连接任意的broker，但是topic都一致

首先，在create topic的时候，通过describe发现zookeeper会随机挑选一个broker来作为该
topic的数据保存节点，
而在producer写数据的时候，发现不管是哪个producer产生数据，consumer都可以消费, 而且
最终数据都在zookeeper分配的broker节点的log文件中。


2. 测试多个replication实例的作用

当create topic时，如果指定多个replication，则zookeeper会随机挑选两个broker，并推举
某一个为leader节点，另一个为备份节点。
此时可以查看这两个broker的log目录，会发现有相同的topic消息目录.
当某个节点失效后，只要有一个节点工作，就不会影响数据的生产和消费

Kafka分配Replica的算法如下：

将所有Broker（假设共n个Broker）和待分配的Partition排序
将第i个Partition分配到第（i mod n）个Broker上
将第i个Partition的第j个Replica分配到第（(i + j) mode n）个Broker上


3. 当一个replication停止后再重启，是否会同步备份数据? 

可以, 它在重启后回同步replication节点的数据

4. 多线程consumer, 单partition

```
Map<String, Integer> topicCountMap = new HashMap<String, Integer>();
topicCountMap.put(TOPIC, THREAD_NUMBER);

Map<String, List<KafkaStream<byte[], byte[]>>> consumerMap =
consumerConnector.createMessageStreams(topicCountMap);
List<KafkaStream<byte[], byte[]>> streams = consumerMap.get(TOPIC);
```

与zookeeper建立connector之后，在创建消息流时，告知每个消息有多少个线程来处理

同一个groupId的consumer在停止重启多次后，每次都是从上次consumer消费终止的地方开始
重新消费topic的message
如果启动一个新的groupId的consumer，他只会从当前的topic的尾部开始消费message, 而不是从头
开始消费consumer
从这里可以看出同一个groupId的多个consumer共享同一个offset
一个groupId中有多个consumer时，同时只能一个consumer可用，当当前可用consumer关掉后,
则其他的consumer会推举个可用的

5. 多consumer，多partition

如何，让消息落在不同的partition上?

6. 单consumer, 多partition

7. 同一个Consumer Group只有一个Consumer能消费某一条消息

启动处于同一个consumer group的多个consumer进程，producer向topic发送消息，发现只有
一个consumer能收到数据，另外的consumer没有任何数据提醒。

## 疑问

1. 为什么producer需要连接broker, 而consumer需要连接zookeeper, 为什么producer可以连接一个随意的broker

2. 如果一个topic有多个consumer,这是每个consumer有自己的offset，还是只维护一个offset

3. kafkaStream

在一个kafka consumer server中，一般对应一个topic有N个partition就需要定义N个kafkaStream消费，每个kafkaStream需要单独建立一个线程工作

这里探索启动多个kafka consumer server会出现什么情况?

如果某个topic有N个partition，consumer server开启的线程不足N，此时代表这个consumer线程消费所有的partition
这时如果又开启另外的consumer server，则zookeeper会合理分配partition给所有的consumer线程
如果第一次开启的consumer有足够的线程来消费partition，则如果新开启consumer线程，则zookeeper不会额外调整消费线程


4. Java的kafka_producer采用的是多线程做的异步发送消息

不同的topic会开启不同的消息通信线程
同一个topic即使有不同的producer对象，也会公用一个消息通信线程



