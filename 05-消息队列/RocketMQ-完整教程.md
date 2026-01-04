# RocketMQ 完整教程

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
- **版本**: 5.1.x
- **官方文档**: [https://rocketmq.apache.org](https://rocketmq.apache.org)
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、分布式系统基础、消息队列基础
- **文档来源**: Apache RocketMQ官方文档
- **更新时间**: 2024-12-31

### 什么是RocketMQ
Apache RocketMQ是阿里巴巴开源的分布式消息中间件，具有低延迟、高可靠、万亿级容量和灵活的可扩展性。RocketMQ是阿里巴巴双11购物节的核心消息系统，经过大规模生产环境验证。

### 核心价值
- **高性能**: 单机支持万级TPS
- **高可用**: 支持主从同步和异步复制
- **海量消息堆积**: 支持亿级消息堆积
- **丰富特性**: 支持事务消息、延迟消息、顺序消息
- **运维友好**: 提供完善的管理控制台

## 🎯 学习目标
- [ ] 理解RocketMQ核心概念和架构
- [ ] 掌握普通消息、顺序消息的使用
- [ ] 理解并实现事务消息
- [ ] 掌握延迟消息的使用场景
- [ ] 理解消息过滤和消息轨迹
- [ ] 掌握集群部署和高可用配置


## 📖 基础概念

### 1.1 核心组件

#### NameServer（名称服务器）
- 类似注册中心，管理Broker的路由信息
- 无状态，可集群部署
- Broker定期向NameServer上报状态

#### Broker（消息服务器）
- 存储消息、转发消息
- 分为Master和Slave
- Master负责读写，Slave负责备份

#### Producer（生产者）
- 发送消息到Broker
- 支持同步、异步、单向发送
- 支持负载均衡

#### Consumer（消费者）
- 从Broker拉取消息
- 支持Push和Pull两种模式
- 支持集群消费和广播消费

#### Topic（主题）
- 消息的逻辑分类
- 一个Topic可以有多个Queue

#### Message Queue（消息队列）
- Topic的物理分区
- 类似Kafka的Partition

### 1.2 消息类型

1. **普通消息**: 最基本的消息类型
2. **顺序消息**: 保证消息顺序消费
3. **延迟消息**: 延迟一定时间后才能被消费
4. **事务消息**: 支持分布式事务
5. **批量消息**: 批量发送提高吞吐量

### 1.3 应用场景
- **应用解耦**: 订单系统与库存系统解耦
- **流量削峰**: 秒杀场景削峰填谷
- **数据分发**: 数据同步到多个系统
- **异步处理**: 注册后发送邮件、短信
- **分布式事务**: 保证数据最终一致性

## 🔥 核心特性 (重点)

### 2.1 普通消息 🔥

#### 同步发送
```java
/**
 * 同步发送消息
 * @author erik.zhou
 */
public class SyncProducer {
    
    public void sendSync() throws Exception {
        // 创建生产者
        DefaultMQProducer producer = new DefaultMQProducer("producer_group");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();
        
        try {
            // 创建消息
            Message msg = new Message(
                "TopicTest",        // Topic
                "TagA",             // Tag
                "Hello RocketMQ".getBytes(StandardCharsets.UTF_8)
            );
            
            // 同步发送
            SendResult sendResult = producer.send(msg);
            System.out.printf("发送结果: %s%n", sendResult);
            
        } finally {
            producer.shutdown();
        }
    }
}
```

#### 异步发送
```java
/**
 * 异步发送消息
 * @author erik.zhou
 */
public class AsyncProducer {
    
    public void sendAsync() throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("producer_group");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();
        
        try {
            Message msg = new Message("TopicTest", "TagA", 
                "Hello RocketMQ".getBytes(StandardCharsets.UTF_8));
            
            // 异步发送
            producer.send(msg, new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.println("发送成功: " + sendResult);
                }
                
                @Override
                public void onException(Throwable e) {
                    System.err.println("发送失败: " + e.getMessage());
                }
            });
            
            // 等待异步发送完成
            Thread.sleep(1000);
            
        } finally {
            producer.shutdown();
        }
    }
}
```

#### 消费者
```java
/**
 * Push模式消费者
 * @author erik.zhou
 */
public class PushConsumer {
    
    public void consume() throws Exception {
        // 创建消费者
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_group");
        consumer.setNamesrvAddr("localhost:9876");
        
        // 订阅Topic
        consumer.subscribe("TopicTest", "*");
        
        // 注册消息监听器
        consumer.registerMessageListener(new MessageListenerConcurrently() {
            @Override
            public ConsumeConcurrentlyStatus consumeMessage(
                    List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                
                for (MessageExt msg : msgs) {
                    System.out.printf("接收消息: %s%n", 
                        new String(msg.getBody(), StandardCharsets.UTF_8));
                }
                
                // 返回消费状态
                return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
            }
        });
        
        // 启动消费者
        consumer.start();
        System.out.println("消费者启动成功");
    }
}
```

### 2.2 顺序消息 🔥

**全局顺序**: 整个Topic只有一个Queue
**分区顺序**: 相同业务ID的消息发送到同一个Queue

```java
/**
 * 顺序消息生产者
 * @author erik.zhou
 */
public class OrderedProducer {
    
    public void sendOrdered() throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("producer_group");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();
        
        try {
            String orderId = "ORDER_001";
            
            // 发送订单相关的多条消息
            String[] messages = {"创建订单", "支付订单", "发货", "完成"};
            
            for (String msg : messages) {
                Message message = new Message("OrderTopic", "TagA", 
                    msg.getBytes(StandardCharsets.UTF_8));
                
                // 使用MessageQueueSelector选择队列
                SendResult sendResult = producer.send(message, 
                    new MessageQueueSelector() {
                        @Override
                        public MessageQueue select(List<MessageQueue> mqs, 
                                Message msg, Object arg) {
                            // 根据orderId选择队列，保证相同订单的消息进入同一队列
                            String orderId = (String) arg;
                            int index = Math.abs(orderId.hashCode()) % mqs.size();
                            return mqs.get(index);
                        }
                    }, orderId);  // orderId作为选择队列的参数
                
                System.out.println("发送顺序消息: " + msg + ", " + sendResult);
            }
            
        } finally {
            producer.shutdown();
        }
    }
}

/**
 * 顺序消息消费者
 * @author erik.zhou
 */
public class OrderedConsumer {
    
    public void consume() throws Exception {
        DefaultMQPushConsumer consumer = new DefaultMQPushConsumer("consumer_group");
        consumer.setNamesrvAddr("localhost:9876");
        consumer.subscribe("OrderTopic", "*");
        
        // 注册顺序消息监听器
        consumer.registerMessageListener(new MessageListenerOrderly() {
            @Override
            public ConsumeOrderlyStatus consumeMessage(
                    List<MessageExt> msgs, ConsumeOrderlyContext context) {
                
                for (MessageExt msg : msgs) {
                    System.out.printf("顺序消费: %s%n", 
                        new String(msg.getBody(), StandardCharsets.UTF_8));
                }
                
                return ConsumeOrderlyStatus.SUCCESS;
            }
        });
        
        consumer.start();
        System.out.println("顺序消费者启动成功");
    }
}
```

### 2.3 延迟消息 🔥

RocketMQ支持18个延迟级别：1s 5s 10s 30s 1m 2m 3m 4m 5m 6m 7m 8m 9m 10m 20m 30m 1h 2h

```java
/**
 * 延迟消息示例
 * @author erik.zhou
 */
public class DelayMessageProducer {
    
    public void sendDelayMessage() throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("producer_group");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();
        
        try {
            Message msg = new Message("DelayTopic", "TagA", 
                "延迟消息".getBytes(StandardCharsets.UTF_8));
            
            // 设置延迟级别：3表示延迟10秒
            msg.setDelayTimeLevel(3);
            
            SendResult sendResult = producer.send(msg);
            System.out.println("发送延迟消息: " + sendResult);
            
        } finally {
            producer.shutdown();
        }
    }
}
```

**应用场景**:
- 订单超时自动取消
- 定时任务触发
- 延迟通知


### 2.4 事务消息 🔥 (⚠️ 难点)

事务消息用于保证本地事务和消息发送的原子性。

**执行流程**:
1. 发送半消息（Half Message）
2. 执行本地事务
3. 提交或回滚事务消息
4. 如果超时未确认，Broker回查事务状态

```java
/**
 * 事务消息生产者
 * @author erik.zhou
 */
public class TransactionProducer {
    
    public void sendTransactional() throws Exception {
        // 创建事务监听器
        TransactionListener transactionListener = new TransactionListener() {
            
            // 执行本地事务
            @Override
            public LocalTransactionState executeLocalTransaction(Message msg, Object arg) {
                System.out.println("执行本地事务: " + new String(msg.getBody()));
                
                try {
                    // 执行本地业务逻辑
                    // 例如：数据库操作
                    boolean success = doLocalTransaction(arg);
                    
                    if (success) {
                        return LocalTransactionState.COMMIT_MESSAGE;
                    } else {
                        return LocalTransactionState.ROLLBACK_MESSAGE;
                    }
                    
                } catch (Exception e) {
                    System.err.println("本地事务执行失败: " + e.getMessage());
                    return LocalTransactionState.ROLLBACK_MESSAGE;
                }
            }
            
            // 回查本地事务状态
            @Override
            public LocalTransactionState checkLocalTransaction(MessageExt msg) {
                System.out.println("回查事务状态: " + msg.getTransactionId());
                
                // 根据业务逻辑查询事务状态
                // 例如：查询数据库记录是否存在
                boolean exists = checkTransactionStatus(msg.getTransactionId());
                
                if (exists) {
                    return LocalTransactionState.COMMIT_MESSAGE;
                } else {
                    return LocalTransactionState.ROLLBACK_MESSAGE;
                }
            }
            
            private boolean doLocalTransaction(Object arg) {
                // 模拟本地事务
                return true;
            }
            
            private boolean checkTransactionStatus(String transactionId) {
                // 模拟查询事务状态
                return true;
            }
        };
        
        // 创建事务生产者
        TransactionMQProducer producer = new TransactionMQProducer("transaction_group");
        producer.setNamesrvAddr("localhost:9876");
        
        // 设置事务监听器
        producer.setTransactionListener(transactionListener);
        
        // 设置线程池
        ExecutorService executorService = new ThreadPoolExecutor(
            2, 5, 100, TimeUnit.SECONDS,
            new ArrayBlockingQueue<>(2000),
            new ThreadFactory() {
                @Override
                public Thread newThread(Runnable r) {
                    Thread thread = new Thread(r);
                    thread.setName("transaction-thread");
                    return thread;
                }
            });
        producer.setExecutorService(executorService);
        
        producer.start();
        
        try {
            // 发送事务消息
            Message msg = new Message("TransactionTopic", "TagA", 
                "事务消息".getBytes(StandardCharsets.UTF_8));
            
            SendResult sendResult = producer.sendMessageInTransaction(msg, null);
            System.out.println("发送事务消息: " + sendResult);
            
            Thread.sleep(10000);  // 等待事务确认
            
        } finally {
            producer.shutdown();
            executorService.shutdown();
        }
    }
}
```

**应用场景**:
- 订单创建后发送通知
- 账户扣款后发送消息
- 库存扣减后发送消息

### 2.5 批量消息

```java
/**
 * 批量消息示例
 * @author erik.zhou
 */
public class BatchProducer {
    
    public void sendBatch() throws Exception {
        DefaultMQProducer producer = new DefaultMQProducer("producer_group");
        producer.setNamesrvAddr("localhost:9876");
        producer.start();
        
        try {
            List<Message> messages = new ArrayList<>();
            
            for (int i = 0; i < 10; i++) {
                Message msg = new Message("BatchTopic", "TagA", 
                    ("批量消息-" + i).getBytes(StandardCharsets.UTF_8));
                messages.add(msg);
            }
            
            // 批量发送
            SendResult sendResult = producer.send(messages);
            System.out.println("批量发送结果: " + sendResult);
            
        } finally {
            producer.shutdown();
        }
    }
}
```

## 💻 实战应用

### 3.1 环境搭建

#### Docker部署
```bash
# 启动NameServer
docker run -d --name rmqnamesrv \
  -p 9876:9876 \
  apache/rocketmq:5.1.0 \
  sh mqnamesrv

# 启动Broker
docker run -d --name rmqbroker \
  --link rmqnamesrv:namesrv \
  -p 10911:10911 -p 10909:10909 \
  -e "NAMESRV_ADDR=namesrv:9876" \
  apache/rocketmq:5.1.0 \
  sh mqbroker
```

#### Maven依赖
```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-client</artifactId>
    <version>5.1.0</version>
</dependency>
```

### 3.2 Spring Boot集成

#### 依赖
```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.2.3</version>
</dependency>
```

#### 配置
```yaml
rocketmq:
  name-server: localhost:9876
  producer:
    group: spring-producer-group
    send-message-timeout: 3000
    retry-times-when-send-failed: 2
```

#### 生产者
```java
/**
 * Spring Boot生产者
 * @author erik.zhou
 */
@Service
public class OrderProducer {
    
    @Autowired
    private RocketMQTemplate rocketMQTemplate;
    
    public void sendOrder(Order order) {
        // 同步发送
        SendResult sendResult = rocketMQTemplate.syncSend(
            "order-topic", order);
        System.out.println("发送结果: " + sendResult);
    }
    
    public void sendOrderAsync(Order order) {
        // 异步发送
        rocketMQTemplate.asyncSend("order-topic", order, 
            new SendCallback() {
                @Override
                public void onSuccess(SendResult sendResult) {
                    System.out.println("发送成功: " + sendResult);
                }
                
                @Override
                public void onException(Throwable e) {
                    System.err.println("发送失败: " + e.getMessage());
                }
            });
    }
}
```

#### 消费者
```java
/**
 * Spring Boot消费者
 * @author erik.zhou
 */
@Service
@RocketMQMessageListener(
    topic = "order-topic",
    consumerGroup = "order-consumer-group"
)
public class OrderConsumer implements RocketMQListener<Order> {
    
    @Override
    public void onMessage(Order order) {
        System.out.println("接收订单: " + order);
        // 处理订单逻辑
        processOrder(order);
    }
    
    private void processOrder(Order order) {
        // 业务处理
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

1. **批量发送**: 提高吞吐量
2. **异步发送**: 提高发送效率
3. **消息压缩**: 减少网络传输
4. **合理设置线程数**: 提高并发处理能力

### 4.2 常见陷阱

#### ⚠️ 陷阱1: 消息丢失
**解决方案**: 
- 同步发送并检查返回结果
- 开启持久化
- 配置主从同步

#### ⚠️ 陷阱2: 消息重复
**解决方案**: 业务层实现幂等性

#### ⚠️ 陷阱3: 消息堆积
**解决方案**:
- 增加消费者数量
- 优化消费逻辑
- 设置消息过期时间

## ❓ 常见问题

### Q1: RocketMQ和Kafka的区别？
**A**: 
- **RocketMQ**: 支持事务消息、延迟消息，更适合业务场景
- **Kafka**: 更高吞吐量，更适合日志收集和流处理

### Q2: 如何保证消息不丢失？
**A**: 
1. 生产者使用同步发送
2. Broker开启持久化
3. 消费者手动提交offset

### Q3: 事务消息如何使用？
**A**: 实现TransactionListener接口，处理本地事务和事务回查

## 🔗 相关资源

- [RocketMQ官方文档](https://rocketmq.apache.org/docs/quick-start/)
- [RocketMQ GitHub](https://github.com/apache/rocketmq)
- [RocketMQ控制台](https://github.com/apache/rocketmq-dashboard)

## 📝 学习检查清单

- [ ] 理解RocketMQ核心概念
- [ ] 掌握普通消息使用
- [ ] 掌握顺序消息使用
- [ ] 理解事务消息原理
- [ ] 掌握延迟消息使用
- [ ] 能够集成Spring Boot
- [ ] 了解性能优化方法

---

**@author erik.zhou**  
**文档版本**: 1.0  
**最后更新**: 2024-12-31
