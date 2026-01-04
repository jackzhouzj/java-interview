# Kafka 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [相关资源](#相关资源)

## 📚 技术概述
- **版本**: 3.6.x
- **官方文档**: [https://kafka.apache.org](https://kafka.apache.org)
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、分布式系统基础、网络编程
- **文档来源**: Apache Kafka官方文档
- **更新时间**: 2024-12-31

### 什么是Kafka
Apache Kafka是一个分布式流处理平台和消息系统，最初由LinkedIn开发并开源。Kafka被设计为高吞吐量、可持久化、可水平扩展的分布式发布-订阅消息系统。它能够处理数万亿级别的事件，被广泛应用于实时数据管道、流式处理、日志聚合等场景。

### 核心价值
- **高吞吐量**: 单机可达百万级TPS
- **可扩展性**: 支持水平扩展，轻松应对数据增长
- **持久化**: 消息持久化到磁盘，支持数据回溯
- **容错性**: 分布式架构，支持副本机制
- **实时性**: 毫秒级延迟，支持实时流处理

## 🎯 学习目标
- [ ] 理解Kafka的核心概念（Topic、Partition、Replica）
- [ ] 掌握生产者和消费者的使用
- [ ] 理解分区机制和负载均衡
- [ ] 掌握副本机制和高可用配置
- [ ] 理解消息顺序性保证机制
- [ ] 掌握幂等性和事务特性
- [ ] 能够进行性能调优和监控


## 📖 基础概念

### 1.1 核心组件

#### Broker（代理服务器）
- Kafka集群由多个Broker组成
- 每个Broker是一个独立的Kafka服务器
- Broker负责接收、存储和转发消息
- 通过Broker ID唯一标识

#### Topic（主题）
- 消息的逻辑分类
- 生产者发送消息到Topic
- 消费者从Topic订阅消息
- 一个Topic可以有多个分区

#### Partition（分区）
- Topic的物理分组
- 每个分区是一个有序的、不可变的消息序列
- 分区内消息有序，分区间无序
- 分区是Kafka并行处理的基本单位

#### Replica（副本）
- 每个分区可以有多个副本
- 副本分为Leader和Follower
- Leader处理所有读写请求
- Follower同步Leader的数据，提供容错

#### Producer（生产者）
- 向Kafka发送消息的客户端
- 决定消息发送到哪个分区
- 支持同步和异步发送

#### Consumer（消费者）
- 从Kafka读取消息的客户端
- 消费者通过Consumer Group实现负载均衡
- 支持自动和手动提交offset

#### Consumer Group（消费者组）
- 多个消费者组成一个消费者组
- 同一消费者组内的消费者共同消费Topic
- 每个分区只能被组内一个消费者消费
- 不同消费者组可以独立消费同一Topic

#### Offset（偏移量）
- 消息在分区中的唯一标识
- 消费者通过offset追踪消费进度
- offset存储在Kafka内部Topic中

### 1.2 架构图

```
Producer1 ──┐
Producer2 ──┼──> Broker1 (Leader P0, Follower P1)
Producer3 ──┘    Broker2 (Follower P0, Leader P1)
                 Broker3 (Follower P0, Follower P1)
                      │
                      ├──> Consumer Group 1
                      │    ├─ Consumer1 (P0)
                      │    └─ Consumer2 (P1)
                      │
                      └──> Consumer Group 2
                           ├─ Consumer3 (P0)
                           └─ Consumer4 (P1)
```

### 1.3 应用场景
- **消息系统**: 解耦系统组件，异步处理
- **日志聚合**: 收集分布式系统的日志
- **流式处理**: 实时数据处理和分析
- **事件溯源**: 记录系统状态变更
- **数据管道**: 在不同系统间传输数据
- **指标监控**: 收集和处理监控指标


## 🔥 核心特性 (重点)

### 2.1 生产者（Producer）🔥

#### 基本使用

**代码示例**:
```java
/**
 * Kafka生产者示例
 * @author erik.zhou
 */
public class KafkaProducerExample {
    
    public void sendMessage() {
        // 1. 配置生产者属性
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        
        // 2. 创建生产者
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        
        try {
            // 3. 创建消息记录
            ProducerRecord<String, String> record = 
                new ProducerRecord<>("my-topic", "key", "value");
            
            // 4. 发送消息（异步）
            producer.send(record, (metadata, exception) -> {
                if (exception == null) {
                    System.out.println("消息发送成功: " + 
                        "topic=" + metadata.topic() + 
                        ", partition=" + metadata.partition() + 
                        ", offset=" + metadata.offset());
                } else {
                    System.err.println("消息发送失败: " + exception.getMessage());
                }
            });
            
        } finally {
            // 5. 关闭生产者
            producer.close();
        }
    }
    
    // 同步发送
    public void sendMessageSync() throws Exception {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        
        try {
            ProducerRecord<String, String> record = 
                new ProducerRecord<>("my-topic", "key", "value");
            
            // 同步发送，等待结果
            RecordMetadata metadata = producer.send(record).get();
            System.out.println("消息发送成功: offset=" + metadata.offset());
            
        } finally {
            producer.close();
        }
    }
}
```

#### 分区策略

**默认分区器**:
- 如果指定了partition，直接使用
- 如果指定了key，根据key的hash值选择分区
- 如果都没指定，轮询选择分区

**自定义分区器**:
```java
/**
 * 自定义分区器
 * @author erik.zhou
 */
public class CustomPartitioner implements Partitioner {
    
    @Override
    public int partition(String topic, Object key, byte[] keyBytes, 
                        Object value, byte[] valueBytes, Cluster cluster) {
        List<PartitionInfo> partitions = cluster.partitionsForTopic(topic);
        int numPartitions = partitions.size();
        
        if (key == null) {
            // 没有key，随机选择分区
            return ThreadLocalRandom.current().nextInt(numPartitions);
        }
        
        // 根据业务逻辑自定义分区策略
        // 例如：按用户ID分区
        if (key instanceof String) {
            String userId = (String) key;
            return Math.abs(userId.hashCode()) % numPartitions;
        }
        
        return Math.abs(key.hashCode()) % numPartitions;
    }
    
    @Override
    public void close() {}
    
    @Override
    public void configure(Map<String, ?> configs) {}
}

// 使用自定义分区器
props.put(ProducerConfig.PARTITIONER_CLASS_CONFIG, CustomPartitioner.class.getName());
```

### 2.2 消费者（Consumer）🔥

#### 基本使用

**代码示例**:
```java
/**
 * Kafka消费者示例
 * @author erik.zhou
 */
public class KafkaConsumerExample {
    
    public void consumeMessage() {
        // 1. 配置消费者属性
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, 
                  StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, 
                  StringDeserializer.class.getName());
        props.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        
        // 2. 创建消费者
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        
        try {
            // 3. 订阅主题
            consumer.subscribe(Collections.singletonList("my-topic"));
            
            // 4. 拉取消息
            while (true) {
                ConsumerRecords<String, String> records = 
                    consumer.poll(Duration.ofMillis(100));
                
                for (ConsumerRecord<String, String> record : records) {
                    System.out.printf("接收消息: topic=%s, partition=%d, offset=%d, key=%s, value=%s%n",
                        record.topic(), record.partition(), record.offset(), 
                        record.key(), record.value());
                }
            }
            
        } finally {
            // 5. 关闭消费者
            consumer.close();
        }
    }
}
```

#### 手动提交Offset

**代码示例**:
```java
/**
 * 手动提交Offset示例
 * @author erik.zhou
 */
public class ManualCommitExample {
    
    public void consumeWithManualCommit() {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, 
                  StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, 
                  StringDeserializer.class.getName());
        // 关闭自动提交
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
        
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Collections.singletonList("my-topic"));
        
        try {
            while (true) {
                ConsumerRecords<String, String> records = 
                    consumer.poll(Duration.ofMillis(100));
                
                for (ConsumerRecord<String, String> record : records) {
                    try {
                        // 处理消息
                        processRecord(record);
                        
                        // 同步提交当前offset
                        consumer.commitSync(Collections.singletonMap(
                            new TopicPartition(record.topic(), record.partition()),
                            new OffsetAndMetadata(record.offset() + 1)
                        ));
                        
                    } catch (Exception e) {
                        System.err.println("处理消息失败: " + e.getMessage());
                        // 可以选择跳过或重试
                    }
                }
            }
            
        } finally {
            consumer.close();
        }
    }
    
    // 异步提交
    public void consumeWithAsyncCommit() {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, 
                  StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, 
                  StringDeserializer.class.getName());
        props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");
        
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(props);
        consumer.subscribe(Collections.singletonList("my-topic"));
        
        try {
            while (true) {
                ConsumerRecords<String, String> records = 
                    consumer.poll(Duration.ofMillis(100));
                
                for (ConsumerRecord<String, String> record : records) {
                    processRecord(record);
                }
                
                // 异步提交offset
                consumer.commitAsync((offsets, exception) -> {
                    if (exception != null) {
                        System.err.println("提交offset失败: " + exception.getMessage());
                    } else {
                        System.out.println("提交offset成功: " + offsets);
                    }
                });
            }
            
        } finally {
            // 关闭前同步提交，确保offset提交成功
            try {
                consumer.commitSync();
            } finally {
                consumer.close();
            }
        }
    }
    
    private void processRecord(ConsumerRecord<String, String> record) {
        // 处理消息逻辑
        System.out.println("处理消息: " + record.value());
    }
}
```


### 2.3 分区与副本机制 🔥

#### 分区（Partition）

**分区的作用**:
1. **负载均衡**: 数据分散到多个分区，提高并行度
2. **水平扩展**: 增加分区数量可以提高吞吐量
3. **顺序保证**: 同一分区内消息有序

**分区分配策略**:
- **Range**: 按分区范围分配（默认）
- **RoundRobin**: 轮询分配
- **Sticky**: 粘性分配，尽量保持原有分配

**代码示例**:
```java
/**
 * 分区配置示例
 * @author erik.zhou
 */
public class PartitionExample {
    
    // 创建多分区Topic
    public void createTopic() {
        Properties props = new Properties();
        props.put(AdminClientConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        
        try (AdminClient adminClient = AdminClient.create(props)) {
            NewTopic newTopic = new NewTopic("my-topic", 3, (short) 2);
            // 3个分区，2个副本
            
            adminClient.createTopics(Collections.singleton(newTopic));
            System.out.println("Topic创建成功");
        }
    }
    
    // 指定分区发送消息
    public void sendToPartition() {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        
        // 指定发送到分区0
        ProducerRecord<String, String> record = 
            new ProducerRecord<>("my-topic", 0, "key", "value");
        
        producer.send(record);
        producer.close();
    }
}
```

#### 副本（Replica）

**副本机制**:
- **Leader副本**: 处理所有读写请求
- **Follower副本**: 同步Leader数据，不处理客户端请求
- **ISR（In-Sync Replicas）**: 与Leader保持同步的副本集合

**副本同步流程**:
```
1. Producer发送消息到Leader
2. Leader写入本地日志
3. Follower从Leader拉取消息
4. Follower写入本地日志
5. Follower发送ACK给Leader
6. Leader收到所有ISR的ACK后，更新HW（High Watermark）
7. Leader返回ACK给Producer
```

### 2.4 消息顺序性 🔥 (⚠️ 难点)

#### 顺序保证规则
1. **分区内有序**: 同一分区内的消息严格有序
2. **分区间无序**: 不同分区之间的消息无序
3. **Key相同**: 相同key的消息会发送到同一分区

#### 保证顺序性的方法

**方法1: 单分区**
```java
/**
 * 单分区保证全局顺序
 * @author erik.zhou
 */
public class SinglePartitionOrder {
    
    public void sendOrdered() {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        // 设置为1，保证消息顺序
        props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 1);
        
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        
        // 所有消息发送到同一分区
        for (int i = 0; i < 10; i++) {
            ProducerRecord<String, String> record = 
                new ProducerRecord<>("my-topic", 0, null, "message-" + i);
            producer.send(record);
        }
        
        producer.close();
    }
}
```

**方法2: 相同Key**
```java
/**
 * 使用相同Key保证局部顺序
 * @author erik.zhou
 */
public class KeyBasedOrder {
    
    public void sendOrderedByKey(String userId, List<String> messages) {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        
        // 相同userId的消息会发送到同一分区，保证顺序
        for (String message : messages) {
            ProducerRecord<String, String> record = 
                new ProducerRecord<>("my-topic", userId, message);
            producer.send(record);
        }
        
        producer.close();
    }
}
```

### 2.5 幂等性与事务 🔥 (⚠️ 难点)

#### 幂等性（Idempotence）

**作用**: 防止消息重复发送

**原理**:
- Producer为每条消息分配唯一的序列号
- Broker检测到重复序列号时，丢弃重复消息
- 只保证单个Producer单个分区的幂等性

**配置**:
```java
/**
 * 幂等性生产者
 * @author erik.zhou
 */
public class IdempotentProducer {
    
    public void sendIdempotent() {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        
        // 开启幂等性
        props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        // 幂等性要求以下配置
        props.put(ProducerConfig.ACKS_CONFIG, "all");
        props.put(ProducerConfig.RETRIES_CONFIG, Integer.MAX_VALUE);
        props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 5);
        
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        
        ProducerRecord<String, String> record = 
            new ProducerRecord<>("my-topic", "key", "value");
        producer.send(record);
        
        producer.close();
    }
}
```

#### 事务（Transaction）

**作用**: 保证多条消息的原子性

**特性**:
- **原子性**: 多条消息要么全部成功，要么全部失败
- **跨分区**: 支持跨多个分区的事务
- **Exactly-Once**: 结合幂等性实现精确一次语义

**代码示例**:
```java
/**
 * 事务生产者
 * @author erik.zhou
 */
public class TransactionalProducer {
    
    public void sendTransactional() {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        
        // 开启事务
        props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        props.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "my-transactional-id");
        
        KafkaProducer<String, String> producer = new KafkaProducer<>(props);
        
        // 初始化事务
        producer.initTransactions();
        
        try {
            // 开始事务
            producer.beginTransaction();
            
            // 发送多条消息
            producer.send(new ProducerRecord<>("topic1", "key1", "value1"));
            producer.send(new ProducerRecord<>("topic2", "key2", "value2"));
            producer.send(new ProducerRecord<>("topic3", "key3", "value3"));
            
            // 提交事务
            producer.commitTransaction();
            System.out.println("事务提交成功");
            
        } catch (Exception e) {
            // 回滚事务
            producer.abortTransaction();
            System.err.println("事务回滚: " + e.getMessage());
        } finally {
            producer.close();
        }
    }
    
    // 事务消费-转换-生产模式
    public void consumeTransformProduce() {
        // 消费者配置
        Properties consumerProps = new Properties();
        consumerProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        consumerProps.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group");
        consumerProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, 
                         StringDeserializer.class.getName());
        consumerProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, 
                         StringDeserializer.class.getName());
        consumerProps.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);
        consumerProps.put(ConsumerConfig.ISOLATION_LEVEL_CONFIG, "read_committed");
        
        // 生产者配置
        Properties producerProps = new Properties();
        producerProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        producerProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
                         StringSerializer.class.getName());
        producerProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
                         StringSerializer.class.getName());
        producerProps.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
        producerProps.put(ProducerConfig.TRANSACTIONAL_ID_CONFIG, "my-tx-id");
        
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProps);
        KafkaProducer<String, String> producer = new KafkaProducer<>(producerProps);
        
        consumer.subscribe(Collections.singletonList("input-topic"));
        producer.initTransactions();
        
        try {
            while (true) {
                ConsumerRecords<String, String> records = 
                    consumer.poll(Duration.ofMillis(100));
                
                if (!records.isEmpty()) {
                    producer.beginTransaction();
                    
                    try {
                        for (ConsumerRecord<String, String> record : records) {
                            // 转换消息
                            String transformedValue = transform(record.value());
                            
                            // 发送到输出Topic
                            producer.send(new ProducerRecord<>(
                                "output-topic", record.key(), transformedValue));
                        }
                        
                        // 提交消费者offset到事务
                        Map<TopicPartition, OffsetAndMetadata> offsets = new HashMap<>();
                        for (ConsumerRecord<String, String> record : records) {
                            offsets.put(
                                new TopicPartition(record.topic(), record.partition()),
                                new OffsetAndMetadata(record.offset() + 1)
                            );
                        }
                        producer.sendOffsetsToTransaction(offsets, 
                            consumer.groupMetadata());
                        
                        // 提交事务
                        producer.commitTransaction();
                        
                    } catch (Exception e) {
                        producer.abortTransaction();
                        System.err.println("事务失败: " + e.getMessage());
                    }
                }
            }
        } finally {
            consumer.close();
            producer.close();
        }
    }
    
    private String transform(String value) {
        // 转换逻辑
        return value.toUpperCase();
    }
}
```


## 💻 实战应用

### 3.1 环境搭建

#### Docker Compose部署
```yaml
version: '3.8'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:7.5.0
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - "2181:2181"

  kafka:
    image: confluentinc/cp-kafka:7.5.0
    depends_on:
      - zookeeper
    ports:
      - "9092:9092"
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

#### Maven依赖
```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>3.6.0</version>
</dependency>
```

### 3.2 Spring Boot集成

#### 配置文件
```yaml
spring:
  kafka:
    bootstrap-servers: localhost:9092
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
      acks: all
      retries: 3
    consumer:
      group-id: my-group
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      auto-offset-reset: earliest
      enable-auto-commit: false
      properties:
        spring.json.trusted.packages: "*"
```

#### 生产者
```java
/**
 * Spring Kafka生产者
 * @author erik.zhou
 */
@Service
@Slf4j
public class OrderProducer {
    
    @Autowired
    private KafkaTemplate<String, Order> kafkaTemplate;
    
    public void sendOrder(Order order) {
        ListenableFuture<SendResult<String, Order>> future = 
            kafkaTemplate.send("order-topic", order.getOrderId(), order);
        
        future.addCallback(
            result -> log.info("发送成功: {}", result.getRecordMetadata()),
            ex -> log.error("发送失败", ex)
        );
    }
}
```

#### 消费者
```java
/**
 * Spring Kafka消费者
 * @author erik.zhou
 */
@Component
@Slf4j
public class OrderConsumer {
    
    @KafkaListener(topics = "order-topic", groupId = "order-group")
    public void consume(ConsumerRecord<String, Order> record,
                       Acknowledgment ack) {
        try {
            log.info("接收订单: {}", record.value());
            processOrder(record.value());
            ack.acknowledge();  // 手动提交
        } catch (Exception e) {
            log.error("处理失败", e);
            // 不提交offset，消息会重新消费
        }
    }
    
    private void processOrder(Order order) {
        // 业务处理
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

#### 生产者优化
```java
/**
 * 生产者性能优化
 * @author erik.zhou
 */
public class ProducerOptimization {
    
    public KafkaProducer<String, String> createOptimizedProducer() {
        Properties props = new Properties();
        props.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        props.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, 
                  StringSerializer.class.getName());
        
        // 批量发送优化
        props.put(ProducerConfig.BATCH_SIZE_CONFIG, 16384);  // 16KB
        props.put(ProducerConfig.LINGER_MS_CONFIG, 10);      // 等待10ms
        
        // 压缩
        props.put(ProducerConfig.COMPRESSION_TYPE_CONFIG, "lz4");
        
        // 缓冲区大小
        props.put(ProducerConfig.BUFFER_MEMORY_CONFIG, 33554432);  // 32MB
        
        // 并发请求数
        props.put(ProducerConfig.MAX_IN_FLIGHT_REQUESTS_PER_CONNECTION, 5);
        
        return new KafkaProducer<>(props);
    }
}
```

#### 消费者优化
```java
/**
 * 消费者性能优化
 * @author erik.zhou
 */
public class ConsumerOptimization {
    
    public KafkaConsumer<String, String> createOptimizedConsumer() {
        Properties props = new Properties();
        props.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
        props.put(ConsumerConfig.GROUP_ID_CONFIG, "my-group");
        props.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, 
                  StringDeserializer.class.getName());
        props.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, 
                  StringDeserializer.class.getName());
        
        // 批量拉取
        props.put(ConsumerConfig.FETCH_MIN_BYTES_CONFIG, 1024);     // 1KB
        props.put(ConsumerConfig.FETCH_MAX_WAIT_MS_CONFIG, 500);    // 500ms
        props.put(ConsumerConfig.MAX_POLL_RECORDS_CONFIG, 500);     // 每次最多500条
        
        // 会话超时
        props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 30000);  // 30s
        props.put(ConsumerConfig.HEARTBEAT_INTERVAL_MS_CONFIG, 3000); // 3s
        
        return new KafkaConsumer<>(props);
    }
}
```

### 4.2 常见陷阱

#### ⚠️ 陷阱1: 消息丢失
**原因**: acks配置不当、未开启幂等性
**解决方案**:
```java
props.put(ProducerConfig.ACKS_CONFIG, "all");
props.put(ProducerConfig.ENABLE_IDEMPOTENCE_CONFIG, true);
props.put(ProducerConfig.RETRIES_CONFIG, Integer.MAX_VALUE);
```

#### ⚠️ 陷阱2: 消息重复
**原因**: 消费者重复消费、未实现幂等性
**解决方案**: 使用Redis或数据库实现消费幂等性

#### ⚠️ 陷阱3: 消费者Rebalance
**原因**: 消费者处理时间过长、心跳超时
**解决方案**:
```java
props.put(ConsumerConfig.MAX_POLL_INTERVAL_MS_CONFIG, 300000);  // 5分钟
props.put(ConsumerConfig.SESSION_TIMEOUT_MS_CONFIG, 30000);     // 30秒
```

## ❓ 常见问题

### Q1: Kafka如何保证高吞吐量？
**A**: 
1. **顺序写磁盘**: 利用磁盘顺序写的高性能
2. **零拷贝**: 使用sendfile系统调用
3. **批量处理**: 批量发送和批量拉取
4. **压缩**: 支持多种压缩算法
5. **分区并行**: 多分区并行处理

### Q2: Kafka和RabbitMQ如何选择？
**A**: 
- **Kafka**: 大数据量、日志收集、流处理、需要消息回溯
- **RabbitMQ**: 复杂路由、消息优先级、延迟队列、中小数据量

### Q3: 如何保证Exactly-Once？
**A**: 
1. 生产者开启幂等性和事务
2. 消费者使用事务消费模式
3. 业务层实现幂等性

### Q4: 分区数如何设置？
**A**: 
- 考虑吞吐量需求
- 考虑消费者并行度
- 一般设置为消费者数量的整数倍
- 建议不超过1000个分区

## 🔗 相关资源

- [Kafka官方文档](https://kafka.apache.org/documentation/)
- [Kafka GitHub](https://github.com/apache/kafka)
- [Confluent文档](https://docs.confluent.io/)

## 📝 学习检查清单

- [ ] 理解Kafka核心概念
- [ ] 掌握生产者和消费者API
- [ ] 理解分区和副本机制
- [ ] 掌握消息顺序性保证
- [ ] 理解幂等性和事务
- [ ] 掌握性能优化方法
- [ ] 能够解决常见问题

---

**@author erik.zhou**  
**文档版本**: 1.0  
**最后更新**: 2024-12-31
