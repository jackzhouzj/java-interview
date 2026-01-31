# RabbitMQ 完整教程

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
- **版本**: 4.2.0
- **官方文档**: [https://www.rabbitmq.com](https://www.rabbitmq.com)
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、多线程编程、网络编程基础
- **文档来源**: RabbitMQ官方文档
- **更新时间**: 2024-12-31

### 什么是RabbitMQ
RabbitMQ是一个开源的消息代理软件（Message Broker），实现了高级消息队列协议（AMQP 0-9-1）。它作为应用程序之间的中间件，实现异步消息传递、系统解耦和负载均衡。RabbitMQ使用Erlang语言编写，具有高可用性、可扩展性和可靠性。

### 核心价值
- **异步通信**: 生产者和消费者无需同时在线
- **系统解耦**: 降低系统间的耦合度
- **流量削峰**: 应对突发流量，保护下游系统
- **可靠传输**: 支持消息持久化和确认机制
- **灵活路由**: 多种交换机类型支持复杂路由场景

## 🎯 学习目标
- [ ] 理解AMQP 0-9-1协议模型和核心概念
- [ ] 掌握四种交换机类型及其应用场景
- [ ] 熟练使用消息确认机制保证可靠性
- [ ] 理解并实现死信队列和延迟队列
- [ ] 掌握RabbitMQ集群部署和高可用配置
- [ ] 能够解决消息丢失、重复消费等常见问题


## 📖 基础概念

### 1.1 AMQP 0-9-1模型
AMQP（Advanced Message Queuing Protocol）是一个消息传递协议，RabbitMQ实现了AMQP 0-9-1版本。

**核心组件**:
- **Producer（生产者）**: 发送消息的应用程序
- **Exchange（交换机）**: 接收生产者的消息并路由到队列
- **Queue（队列）**: 存储消息，等待消费者消费
- **Consumer（消费者）**: 接收并处理消息的应用程序
- **Binding（绑定）**: 交换机和队列之间的路由规则
- **Routing Key（路由键）**: 消息的路由标识
- **Virtual Host（虚拟主机）**: 逻辑隔离的环境

**消息流转过程**:
```
Producer → Exchange → Binding → Queue → Consumer
```

### 1.2 核心概念详解

#### Connection（连接）
- TCP长连接，应用程序与RabbitMQ服务器之间的网络连接
- 连接建立后需要进行身份认证
- 支持TLS加密传输
- 应用程序关闭时应优雅地关闭连接

#### Channel（信道）
- 在一个Connection内部建立的逻辑连接
- 多个Channel共享一个TCP连接，节省系统资源
- 每个Channel都有唯一的ID
- 不同Channel之间的通信完全隔离
- 建议每个线程使用独立的Channel

#### Virtual Host（虚拟主机）
- 类似于Web服务器的虚拟主机概念
- 提供完全隔离的环境（用户、交换机、队列等）
- 默认虚拟主机为 `/`
- 不同虚拟主机之间的资源完全隔离

### 1.3 应用场景
- **异步处理**: 用户注册后发送邮件、短信通知
- **应用解耦**: 订单系统与库存系统、物流系统解耦
- **流量削峰**: 秒杀活动中削峰填谷
- **日志收集**: 分布式系统的日志聚合
- **延迟任务**: 订单超时自动取消
- **消息广播**: 系统通知、配置更新推送


## 🔥 核心特性 (重点)

### 2.1 交换机类型 🔥

RabbitMQ提供四种交换机类型，每种类型有不同的路由策略。

#### Direct Exchange（直连交换机）
**特点**: 根据Routing Key精确匹配路由消息

**工作原理**:
- 消息的Routing Key与Binding的Routing Key完全匹配时，消息才会被路由到队列
- 适用于单播路由场景
- 默认交换机（空字符串）就是Direct类型

**应用场景**:
- 日志系统按级别路由（error、warn、info）
- 任务分发到指定的工作队列

**代码示例**:
```java
/**
 * Direct Exchange示例
 * @author erik.zhou
 */
public class DirectExchangeExample {
    private static final String EXCHANGE_NAME = "direct_logs";
    
    // 生产者
    public void sendMessage(String severity, String message) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            // 声明Direct类型交换机
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT, true);
            
            // 发送消息，指定routing key
            channel.basicPublish(EXCHANGE_NAME, severity, null, message.getBytes());
            System.out.println("发送消息: [" + severity + "] " + message);
        }
    }
    
    // 消费者
    public void receiveMessage(String severity) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT, true);
        String queueName = channel.queueDeclare().getQueue();
        
        // 绑定队列到交换机，指定routing key
        channel.queueBind(queueName, EXCHANGE_NAME, severity);
        
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println("接收消息: [" + severity + "] " + message);
        };
        
        channel.basicConsume(queueName, true, deliverCallback, consumerTag -> {});
    }
}
```

#### Fanout Exchange（扇出交换机）
**特点**: 广播消息到所有绑定的队列，忽略Routing Key

**工作原理**:
- 将消息路由到所有绑定的队列
- 不需要Routing Key
- 性能最好，因为不需要路由判断

**应用场景**:
- 系统广播通知
- 实时数据同步到多个系统
- 缓存更新通知

**代码示例**:
```java
/**
 * Fanout Exchange示例
 * @author erik.zhou
 */
public class FanoutExchangeExample {
    private static final String EXCHANGE_NAME = "logs";
    
    // 生产者
    public void sendMessage(String message) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            // 声明Fanout类型交换机
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.FANOUT, true);
            
            // 发送消息，routing key被忽略
            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
            System.out.println("广播消息: " + message);
        }
    }
}
```


#### Topic Exchange（主题交换机）
**特点**: 根据Routing Key的模式匹配路由消息

**工作原理**:
- Routing Key是由点号（.）分隔的单词列表
- 支持通配符：`*`（匹配一个单词）、`#`（匹配零个或多个单词）
- 适用于复杂的路由场景

**应用场景**:
- 日志系统按模块和级别路由（user.error、order.warn）
- 消息按地域和类型分发（cn.beijing.order、us.newyork.user）

**代码示例**:
```java
/**
 * Topic Exchange示例
 * @author erik.zhou
 */
public class TopicExchangeExample {
    private static final String EXCHANGE_NAME = "topic_logs";
    
    // 消费者绑定示例
    public void bindQueue() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC, true);
        String queueName = channel.queueDeclare().getQueue();
        
        // 绑定多个routing key模式
        channel.queueBind(queueName, EXCHANGE_NAME, "*.error");      // 匹配所有error级别
        channel.queueBind(queueName, EXCHANGE_NAME, "user.#");       // 匹配user模块所有消息
        channel.queueBind(queueName, EXCHANGE_NAME, "order.*.info"); // 匹配order子模块的info消息
    }
}
```

**匹配规则示例**:
- `user.error` 匹配 `*.error` 和 `user.#`
- `user.login.info` 匹配 `user.#`
- `order.payment.info` 匹配 `order.*.info` 和 `#.info`

#### Headers Exchange（头交换机）
**特点**: 根据消息头属性路由，忽略Routing Key

**工作原理**:
- 使用消息的headers属性进行匹配
- 支持多个header匹配
- `x-match`参数：`any`（任意一个匹配）或`all`（全部匹配）

**应用场景**:
- 需要根据多个属性路由的复杂场景
- Routing Key无法满足的路由需求

**代码示例**:
```java
/**
 * Headers Exchange示例
 * @author erik.zhou
 */
public class HeadersExchangeExample {
    private static final String EXCHANGE_NAME = "headers_exchange";
    
    // 绑定队列
    public void bindQueue() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.HEADERS, true);
        String queueName = channel.queueDeclare().getQueue();
        
        // 设置匹配规则
        Map<String, Object> headers = new HashMap<>();
        headers.put("x-match", "all");  // 所有header都要匹配
        headers.put("format", "pdf");
        headers.put("type", "report");
        
        channel.queueBind(queueName, EXCHANGE_NAME, "", headers);
    }
    
    // 发送消息
    public void sendMessage(String message) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.HEADERS, true);
            
            // 设置消息headers
            AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                .headers(Map.of("format", "pdf", "type", "report"))
                .build();
            
            channel.basicPublish(EXCHANGE_NAME, "", properties, message.getBytes());
        }
    }
}
```


### 2.2 消息确认机制 🔥 (⚠️ 难点)

消息确认机制是保证消息可靠性的核心特性，包括生产者确认和消费者确认。

#### 消费者确认（Consumer Acknowledgement）

**自动确认模式（autoAck=true）**:
- 消息一旦被投递给消费者，立即从队列中删除
- 优点：吞吐量高
- 缺点：消费者处理失败时消息会丢失

**手动确认模式（autoAck=false）**:
- 消费者处理完成后显式发送确认
- 优点：保证消息不丢失
- 缺点：需要手动管理确认逻辑

**确认方式**:
- `basicAck`: 确认单条或多条消息
- `basicNack`: 拒绝单条或多条消息
- `basicReject`: 拒绝单条消息

**代码示例**:
```java
/**
 * 消费者确认示例
 * @author erik.zhou
 */
public class ConsumerAckExample {
    
    public void consumeWithManualAck() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        String queueName = "task_queue";
        channel.queueDeclare(queueName, true, false, false, null);
        
        // 设置预取数量，防止消费者积压过多消息
        channel.basicQos(1);
        
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            
            try {
                System.out.println("处理消息: " + message);
                // 模拟业务处理
                doWork(message);
                
                // 手动确认消息（单条确认）
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
                System.out.println("消息确认成功");
                
            } catch (Exception e) {
                System.err.println("处理失败: " + e.getMessage());
                
                // 拒绝消息并重新入队
                // 参数：deliveryTag, multiple, requeue
                channel.basicNack(delivery.getEnvelope().getDeliveryTag(), false, true);
            }
        };
        
        // autoAck设置为false，启用手动确认
        channel.basicConsume(queueName, false, deliverCallback, consumerTag -> {});
    }
    
    private void doWork(String message) throws InterruptedException {
        // 模拟耗时操作
        Thread.sleep(1000);
    }
}
```

#### 生产者确认（Publisher Confirms）

**确认模式**:
1. **普通确认模式**: 同步等待每条消息的确认
2. **批量确认模式**: 批量发送后统一等待确认
3. **异步确认模式**: 异步回调处理确认结果

**代码示例**:
```java
/**
 * 生产者确认示例
 * @author erik.zhou
 */
public class PublisherConfirmExample {
    
    // 1. 普通确认模式（同步）
    public void publishWithSyncConfirm() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            String queueName = "confirm_queue";
            channel.queueDeclare(queueName, true, false, false, null);
            
            // 开启发布确认模式
            channel.confirmSelect();
            
            String message = "Hello RabbitMQ";
            channel.basicPublish("", queueName, null, message.getBytes());
            
            // 等待确认（阻塞）
            if (channel.waitForConfirms()) {
                System.out.println("消息发送成功");
            } else {
                System.out.println("消息发送失败");
            }
        }
    }
    
    // 2. 批量确认模式
    public void publishWithBatchConfirm() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            String queueName = "confirm_queue";
            channel.queueDeclare(queueName, true, false, false, null);
            
            channel.confirmSelect();
            
            int batchSize = 100;
            for (int i = 0; i < 1000; i++) {
                String message = "Message " + i;
                channel.basicPublish("", queueName, null, message.getBytes());
                
                // 每100条确认一次
                if ((i + 1) % batchSize == 0) {
                    channel.waitForConfirms();
                    System.out.println("批次 " + (i / batchSize + 1) + " 确认成功");
                }
            }
        }
    }
    
    // 3. 异步确认模式（推荐）
    public void publishWithAsyncConfirm() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            String queueName = "confirm_queue";
            channel.queueDeclare(queueName, true, false, false, null);
            
            channel.confirmSelect();
            
            // 添加确认监听器
            channel.addConfirmListener(new ConfirmListener() {
                @Override
                public void handleAck(long deliveryTag, boolean multiple) {
                    System.out.println("消息确认成功: " + deliveryTag + ", multiple: " + multiple);
                }
                
                @Override
                public void handleNack(long deliveryTag, boolean multiple) {
                    System.err.println("消息确认失败: " + deliveryTag + ", multiple: " + multiple);
                    // 这里可以实现重试逻辑
                }
            });
            
            // 发送消息
            for (int i = 0; i < 1000; i++) {
                String message = "Message " + i;
                channel.basicPublish("", queueName, null, message.getBytes());
            }
            
            // 等待所有确认完成
            channel.waitForConfirmsOrDie(5000);
        }
    }
}
```


### 2.3 死信队列（Dead Letter Queue）🔥 (⚠️ 难点)

死信队列用于处理无法被正常消费的消息，是保证消息可靠性的重要机制。

#### 消息成为死信的情况
1. **消息被拒绝**（basicReject/basicNack）且requeue=false
2. **消息过期**（TTL超时）
3. **队列达到最大长度**

#### 死信队列配置

**代码示例**:
```java
/**
 * 死信队列示例
 * @author erik.zhou
 */
public class DeadLetterQueueExample {
    
    private static final String NORMAL_EXCHANGE = "normal_exchange";
    private static final String NORMAL_QUEUE = "normal_queue";
    private static final String DEAD_EXCHANGE = "dead_exchange";
    private static final String DEAD_QUEUE = "dead_queue";
    
    public void setupDeadLetterQueue() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            // 1. 声明死信交换机和队列
            channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT, true);
            channel.queueDeclare(DEAD_QUEUE, true, false, false, null);
            channel.queueBind(DEAD_QUEUE, DEAD_EXCHANGE, "dead");
            
            // 2. 声明正常交换机
            channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT, true);
            
            // 3. 声明正常队列，配置死信交换机
            Map<String, Object> args = new HashMap<>();
            args.put("x-dead-letter-exchange", DEAD_EXCHANGE);      // 死信交换机
            args.put("x-dead-letter-routing-key", "dead");          // 死信routing key
            args.put("x-message-ttl", 10000);                       // 消息TTL 10秒
            args.put("x-max-length", 10);                           // 队列最大长度
            
            channel.queueDeclare(NORMAL_QUEUE, true, false, false, args);
            channel.queueBind(NORMAL_QUEUE, NORMAL_EXCHANGE, "normal");
            
            System.out.println("死信队列配置完成");
        }
    }
    
    // 发送消息到正常队列
    public void sendMessage(String message) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            // 可以为单条消息设置TTL
            AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                .expiration("5000")  // 5秒过期
                .build();
            
            channel.basicPublish(NORMAL_EXCHANGE, "normal", properties, message.getBytes());
            System.out.println("发送消息: " + message);
        }
    }
    
    // 消费正常队列（模拟消费失败）
    public void consumeNormalQueue() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println("接收消息: " + message);
            
            try {
                // 模拟处理失败
                if (message.contains("error")) {
                    throw new RuntimeException("处理失败");
                }
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            } catch (Exception e) {
                // 拒绝消息，不重新入队，消息会进入死信队列
                channel.basicNack(delivery.getEnvelope().getDeliveryTag(), false, false);
            }
        };
        
        channel.basicConsume(NORMAL_QUEUE, false, deliverCallback, consumerTag -> {});
    }
    
    // 消费死信队列
    public void consumeDeadLetterQueue() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println("死信队列接收消息: " + message);
            
            // 记录死信消息，进行人工处理或告警
            logDeadLetter(message, delivery);
            
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        
        channel.basicConsume(DEAD_QUEUE, false, deliverCallback, consumerTag -> {});
    }
    
    private void logDeadLetter(String message, Delivery delivery) {
        // 获取死信原因
        Map<String, Object> headers = delivery.getProperties().getHeaders();
        if (headers != null) {
            String reason = (String) headers.get("x-first-death-reason");
            System.out.println("死信原因: " + reason);
        }
    }
}
```

#### 延迟队列实现

利用死信队列的TTL特性可以实现延迟队列：

```java
/**
 * 延迟队列示例
 * @author erik.zhou
 */
public class DelayQueueExample {
    
    // 创建延迟队列
    public void setupDelayQueue(int delayMillis) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            String delayExchange = "delay_exchange";
            String delayQueue = "delay_queue_" + delayMillis;
            String targetExchange = "target_exchange";
            
            // 声明目标交换机
            channel.exchangeDeclare(targetExchange, BuiltinExchangeType.DIRECT, true);
            
            // 声明延迟队列，配置TTL和死信交换机
            Map<String, Object> args = new HashMap<>();
            args.put("x-dead-letter-exchange", targetExchange);
            args.put("x-dead-letter-routing-key", "target");
            args.put("x-message-ttl", delayMillis);
            
            channel.queueDeclare(delayQueue, true, false, false, args);
            channel.exchangeDeclare(delayExchange, BuiltinExchangeType.DIRECT, true);
            channel.queueBind(delayQueue, delayExchange, "delay");
        }
    }
    
    // 发送延迟消息
    public void sendDelayMessage(String message, int delayMillis) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            String delayExchange = "delay_exchange";
            channel.basicPublish(delayExchange, "delay", null, message.getBytes());
            System.out.println("发送延迟消息: " + message + ", 延迟: " + delayMillis + "ms");
        }
    }
}
```


### 2.4 消息持久化

消息持久化确保RabbitMQ重启后消息不丢失。

#### 持久化三要素
1. **交换机持久化**: `durable=true`
2. **队列持久化**: `durable=true`
3. **消息持久化**: `deliveryMode=2`

**代码示例**:
```java
/**
 * 消息持久化示例
 * @author erik.zhou
 */
public class MessagePersistenceExample {
    
    public void sendPersistentMessage() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            String exchangeName = "persistent_exchange";
            String queueName = "persistent_queue";
            
            // 1. 声明持久化交换机
            channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT, true);
            
            // 2. 声明持久化队列
            channel.queueDeclare(queueName, true, false, false, null);
            channel.queueBind(queueName, exchangeName, "key");
            
            // 3. 发送持久化消息
            AMQP.BasicProperties properties = MessageProperties.PERSISTENT_TEXT_PLAIN;
            // 或者手动设置
            // AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
            //     .deliveryMode(2)  // 2表示持久化
            //     .build();
            
            String message = "持久化消息";
            channel.basicPublish(exchangeName, "key", properties, message.getBytes());
            System.out.println("发送持久化消息: " + message);
        }
    }
}
```

### 2.5 集群与高可用 🔥 (⚠️ 难点)

RabbitMQ集群提供高可用性和负载均衡能力。

#### 集群模式

**普通集群模式**:
- 元数据（交换机、队列定义）在所有节点同步
- 消息只存储在声明队列的节点上
- 其他节点接收到消息请求时，会从存储节点拉取

**镜像队列模式**:
- 队列内容在多个节点镜像
- 提供高可用性
- 主节点故障时，从节点自动升级为主节点

#### 集群配置

**docker-compose配置示例**:
```yaml
version: '3.8'
services:
  rabbitmq1:
    image: rabbitmq:3.12-management
    hostname: rabbitmq1
    environment:
      RABBITMQ_ERLANG_COOKIE: 'secret_cookie'
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: admin123
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - ./rabbitmq1:/var/lib/rabbitmq
    networks:
      - rabbitmq-cluster

  rabbitmq2:
    image: rabbitmq:3.12-management
    hostname: rabbitmq2
    environment:
      RABBITMQ_ERLANG_COOKIE: 'secret_cookie'
    depends_on:
      - rabbitmq1
    volumes:
      - ./rabbitmq2:/var/lib/rabbitmq
    networks:
      - rabbitmq-cluster

  rabbitmq3:
    image: rabbitmq:3.12-management
    hostname: rabbitmq3
    environment:
      RABBITMQ_ERLANG_COOKIE: 'secret_cookie'
    depends_on:
      - rabbitmq1
    volumes:
      - ./rabbitmq3:/var/lib/rabbitmq
    networks:
      - rabbitmq-cluster

networks:
  rabbitmq-cluster:
    driver: bridge
```

**镜像队列策略配置**:
```bash
# 设置镜像队列策略（所有队列镜像到所有节点）
rabbitmqctl set_policy ha-all "^" '{"ha-mode":"all"}'

# 设置镜像队列策略（镜像到2个节点）
rabbitmqctl set_policy ha-two "^" '{"ha-mode":"exactly","ha-params":2}'

# 设置镜像队列策略（指定节点）
rabbitmqctl set_policy ha-nodes "^" '{"ha-mode":"nodes","ha-params":["rabbit@node1","rabbit@node2"]}'
```

**Java客户端连接集群**:
```java
/**
 * 集群连接示例
 * @author erik.zhou
 */
public class ClusterConnectionExample {
    
    public Connection createClusterConnection() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        
        // 配置多个节点地址
        Address[] addresses = new Address[]{
            new Address("192.168.1.101", 5672),
            new Address("192.168.1.102", 5672),
            new Address("192.168.1.103", 5672)
        };
        
        factory.setUsername("admin");
        factory.setPassword("admin123");
        
        // 自动重连配置
        factory.setAutomaticRecoveryEnabled(true);
        factory.setNetworkRecoveryInterval(5000);
        
        // 创建连接，客户端会自动选择可用节点
        Connection connection = factory.newConnection(addresses);
        
        return connection;
    }
}
```


## 💻 实战应用

### 3.1 环境搭建

#### Docker方式安装
```bash
# 拉取镜像（带管理界面）
docker pull rabbitmq:3.12-management

# 启动容器
docker run -d --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin123 \
  rabbitmq:3.12-management

# 访问管理界面
# http://localhost:15672
# 用户名: admin
# 密码: admin123
```

#### Maven依赖
```xml
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.20.0</version>
</dependency>
```

#### Spring Boot集成
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### 3.2 快速开始

#### 简单队列模式

**生产者**:
```java
/**
 * 简单队列生产者
 * @author erik.zhou
 */
public class SimpleProducer {
    
    private static final String QUEUE_NAME = "hello";
    
    public static void main(String[] args) throws Exception {
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin123");
        
        // 创建连接和信道
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            // 声明队列
            channel.queueDeclare(QUEUE_NAME, false, false, false, null);
            
            // 发送消息
            String message = "Hello RabbitMQ!";
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("发送消息: " + message);
        }
    }
}
```

**消费者**:
```java
/**
 * 简单队列消费者
 * @author erik.zhou
 */
public class SimpleConsumer {
    
    private static final String QUEUE_NAME = "hello";
    
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin123");
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        // 声明队列
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println("等待接收消息...");
        
        // 定义消费回调
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            System.out.println("接收消息: " + message);
        };
        
        // 开始消费
        channel.basicConsume(QUEUE_NAME, true, deliverCallback, consumerTag -> {});
    }
}
```

### 3.3 Spring Boot集成案例

#### 配置文件
```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: admin
    password: admin123
    virtual-host: /
    # 开启发布确认
    publisher-confirm-type: correlated
    # 开启发布返回
    publisher-returns: true
    # 消费者配置
    listener:
      simple:
        # 手动确认
        acknowledge-mode: manual
        # 并发消费者数量
        concurrency: 5
        max-concurrency: 10
        # 预取数量
        prefetch: 1
```

#### 配置类
```java
/**
 * RabbitMQ配置类
 * @author erik.zhou
 */
@Configuration
public class RabbitMQConfig {
    
    // 声明交换机
    @Bean
    public DirectExchange orderExchange() {
        return new DirectExchange("order.exchange", true, false);
    }
    
    // 声明队列
    @Bean
    public Queue orderQueue() {
        return QueueBuilder.durable("order.queue")
            .withArgument("x-message-ttl", 60000)  // 消息TTL
            .withArgument("x-max-length", 10000)   // 队列最大长度
            .build();
    }
    
    // 绑定
    @Bean
    public Binding orderBinding(Queue orderQueue, DirectExchange orderExchange) {
        return BindingBuilder.bind(orderQueue)
            .to(orderExchange)
            .with("order.create");
    }
    
    // 配置消息转换器（支持JSON）
    @Bean
    public MessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

#### 生产者
```java
/**
 * 订单消息生产者
 * @author erik.zhou
 */
@Service
@Slf4j
public class OrderProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    @PostConstruct
    public void init() {
        // 设置确认回调
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (ack) {
                log.info("消息发送成功: {}", correlationData);
            } else {
                log.error("消息发送失败: {}, 原因: {}", correlationData, cause);
            }
        });
        
        // 设置返回回调
        rabbitTemplate.setReturnsCallback(returned -> {
            log.error("消息被退回: {}", returned.getMessage());
        });
    }
    
    public void sendOrder(Order order) {
        try {
            // 设置消息ID用于确认回调
            CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
            
            rabbitTemplate.convertAndSend(
                "order.exchange",
                "order.create",
                order,
                correlationData
            );
            
            log.info("发送订单消息: {}", order);
        } catch (Exception e) {
            log.error("发送订单消息失败", e);
        }
    }
}
```

#### 消费者
```java
/**
 * 订单消息消费者
 * @author erik.zhou
 */
@Component
@Slf4j
public class OrderConsumer {
    
    @RabbitListener(queues = "order.queue")
    public void handleOrder(Order order, Message message, Channel channel) throws IOException {
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        
        try {
            log.info("接收订单消息: {}", order);
            
            // 处理订单业务逻辑
            processOrder(order);
            
            // 手动确认
            channel.basicAck(deliveryTag, false);
            log.info("订单处理成功: {}", order.getOrderNo());
            
        } catch (Exception e) {
            log.error("订单处理失败: {}", order.getOrderNo(), e);
            
            // 判断是否重试
            Integer retryCount = (Integer) message.getMessageProperties()
                .getHeaders().getOrDefault("retry-count", 0);
            
            if (retryCount < 3) {
                // 重新入队
                channel.basicNack(deliveryTag, false, true);
            } else {
                // 拒绝消息，进入死信队列
                channel.basicNack(deliveryTag, false, false);
                log.error("订单处理失败次数过多，进入死信队列: {}", order.getOrderNo());
            }
        }
    }
    
    private void processOrder(Order order) {
        // 订单处理逻辑
        log.info("处理订单: {}", order.getOrderNo());
    }
}
```


## ✨ 最佳实践

### 4.1 性能优化

#### 1. 连接和信道管理
```java
/**
 * 连接池管理
 * @author erik.zhou
 */
@Configuration
public class RabbitMQConnectionConfig {
    
    @Bean
    public CachingConnectionFactory connectionFactory() {
        CachingConnectionFactory factory = new CachingConnectionFactory();
        factory.setHost("localhost");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin123");
        
        // 连接池配置
        factory.setChannelCacheSize(50);  // 信道缓存数量
        factory.setChannelCheckoutTimeout(5000);  // 获取信道超时时间
        
        // 连接超时配置
        factory.setConnectionTimeout(30000);
        
        return factory;
    }
}
```

#### 2. 批量发送消息
```java
/**
 * 批量发送优化
 * @author erik.zhou
 */
public class BatchPublishExample {
    
    public void batchPublish(List<String> messages) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            String queueName = "batch_queue";
            channel.queueDeclare(queueName, true, false, false, null);
            
            // 批量发送
            for (String message : messages) {
                channel.basicPublish("", queueName, null, message.getBytes());
            }
            
            // 等待所有消息确认
            channel.waitForConfirmsOrDie(5000);
        }
    }
}
```

#### 3. 预取数量优化
```java
/**
 * 预取数量配置
 * @author erik.zhou
 */
public class PrefetchExample {
    
    public void consumeWithPrefetch() throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        Connection connection = factory.newConnection();
        Channel channel = connection.createChannel();
        
        String queueName = "task_queue";
        channel.queueDeclare(queueName, true, false, false, null);
        
        // 设置预取数量
        // prefetchCount: 每次从队列获取的消息数量
        // global: false表示应用于当前channel，true表示应用于整个connection
        channel.basicQos(1, false);  // 每次只取1条消息
        
        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), StandardCharsets.UTF_8);
            try {
                processMessage(message);
                channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
            } catch (Exception e) {
                channel.basicNack(delivery.getEnvelope().getDeliveryTag(), false, true);
            }
        };
        
        channel.basicConsume(queueName, false, deliverCallback, consumerTag -> {});
    }
    
    private void processMessage(String message) {
        // 处理消息
    }
}
```

### 4.2 常见陷阱

#### ⚠️ 陷阱1: 消息丢失

**原因**:
- 未开启消息持久化
- 未开启发布确认
- 消费者自动确认模式下处理失败

**解决方案**:
```java
/**
 * 防止消息丢失
 * @author erik.zhou
 */
public class PreventMessageLossExample {
    
    public void sendReliableMessage(String message) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost("localhost");
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            String exchangeName = "reliable_exchange";
            String queueName = "reliable_queue";
            
            // 1. 交换机持久化
            channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT, true);
            
            // 2. 队列持久化
            channel.queueDeclare(queueName, true, false, false, null);
            channel.queueBind(queueName, exchangeName, "key");
            
            // 3. 开启发布确认
            channel.confirmSelect();
            
            // 4. 消息持久化
            AMQP.BasicProperties properties = MessageProperties.PERSISTENT_TEXT_PLAIN;
            
            // 5. 发送消息
            channel.basicPublish(exchangeName, "key", properties, message.getBytes());
            
            // 6. 等待确认
            if (!channel.waitForConfirms(5000)) {
                throw new RuntimeException("消息发送失败");
            }
            
            System.out.println("消息发送成功: " + message);
        }
    }
}
```

#### ⚠️ 陷阱2: 消息重复消费

**原因**:
- 消费者处理完成但确认失败
- 网络抖动导致重复投递

**解决方案**:
```java
/**
 * 防止重复消费（幂等性）
 * @author erik.zhou
 */
@Service
public class IdempotentConsumer {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @RabbitListener(queues = "order.queue")
    public void handleOrder(Order order, Message message, Channel channel) throws IOException {
        long deliveryTag = message.getMessageProperties().getDeliveryTag();
        String messageId = message.getMessageProperties().getMessageId();
        
        try {
            // 使用Redis实现幂等性检查
            String key = "mq:consumed:" + messageId;
            Boolean success = redisTemplate.opsForValue()
                .setIfAbsent(key, "1", 24, TimeUnit.HOURS);
            
            if (Boolean.FALSE.equals(success)) {
                // 消息已被消费过
                log.warn("消息重复消费，跳过: {}", messageId);
                channel.basicAck(deliveryTag, false);
                return;
            }
            
            // 处理业务逻辑
            processOrder(order);
            
            // 确认消息
            channel.basicAck(deliveryTag, false);
            
        } catch (Exception e) {
            log.error("处理消息失败", e);
            channel.basicNack(deliveryTag, false, true);
        }
    }
    
    private void processOrder(Order order) {
        // 订单处理逻辑
    }
}
```

#### ⚠️ 陷阱3: 消息积压

**原因**:
- 消费者处理速度慢
- 消费者数量不足
- 消费者宕机

**解决方案**:
```java
/**
 * 处理消息积压
 * @author erik.zhou
 */
@Configuration
public class MessageBacklogSolution {
    
    // 1. 增加消费者并发数
    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(
            ConnectionFactory connectionFactory) {
        SimpleRabbitListenerContainerFactory factory = 
            new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        
        // 设置并发消费者数量
        factory.setConcurrentConsumers(10);
        factory.setMaxConcurrentConsumers(20);
        
        // 设置预取数量
        factory.setPrefetchCount(10);
        
        return factory;
    }
    
    // 2. 设置队列最大长度，防止无限积压
    @Bean
    public Queue limitedQueue() {
        return QueueBuilder.durable("limited.queue")
            .maxLength(100000)  // 最大长度
            .overflow(QueueBuilder.Overflow.rejectPublish)  // 超出后拒绝发布
            .build();
    }
}
```

#### ⚠️ 陷阱4: 内存溢出

**原因**:
- 消息体过大
- 队列积压过多消息
- 未设置队列长度限制

**解决方案**:
```java
/**
 * 防止内存溢出
 * @author erik.zhou
 */
@Configuration
public class MemoryOptimizationConfig {
    
    @Bean
    public Queue optimizedQueue() {
        return QueueBuilder.durable("optimized.queue")
            // 设置队列最大长度
            .maxLength(10000)
            // 设置队列最大字节数
            .maxLengthBytes(10485760)  // 10MB
            // 超出策略：删除最旧的消息
            .overflow(QueueBuilder.Overflow.dropHead)
            // 设置消息TTL
            .ttl(3600000)  // 1小时
            .build();
    }
}
```

### 4.3 监控与运维

#### 监控指标
```java
/**
 * RabbitMQ监控
 * @author erik.zhou
 */
@Component
@Slf4j
public class RabbitMQMonitor {
    
    @Autowired
    private RabbitAdmin rabbitAdmin;
    
    @Scheduled(fixedRate = 60000)  // 每分钟执行一次
    public void monitorQueues() {
        Properties queueProperties = rabbitAdmin.getQueueProperties("order.queue");
        
        if (queueProperties != null) {
            Integer messageCount = (Integer) queueProperties.get("QUEUE_MESSAGE_COUNT");
            Integer consumerCount = (Integer) queueProperties.get("QUEUE_CONSUMER_COUNT");
            
            log.info("队列监控 - 消息数量: {}, 消费者数量: {}", messageCount, consumerCount);
            
            // 告警：消息积压超过阈值
            if (messageCount != null && messageCount > 10000) {
                log.error("队列消息积压告警: {}", messageCount);
                // 发送告警通知
            }
            
            // 告警：无消费者
            if (consumerCount != null && consumerCount == 0) {
                log.error("队列无消费者告警");
                // 发送告警通知
            }
        }
    }
}
```


## ❓ 常见问题

### Q1: 如何保证消息不丢失？
**A**: 需要从三个方面保证：
1. **生产者端**: 开启发布确认（Publisher Confirms），确保消息成功到达Broker
2. **Broker端**: 
   - 交换机持久化（durable=true）
   - 队列持久化（durable=true）
   - 消息持久化（deliveryMode=2）
3. **消费者端**: 使用手动确认模式（autoAck=false），处理成功后再确认

### Q2: 如何保证消息顺序性？
**A**: RabbitMQ本身不保证全局顺序，但可以通过以下方式保证局部顺序：
1. **单队列单消费者**: 最简单但性能最差
2. **按业务分区**: 相同业务ID的消息发送到同一队列
3. **使用消息组**: 利用RabbitMQ的消息分组特性

```java
// 按订单ID分区发送
public void sendOrderMessage(Order order) {
    String routingKey = "order." + (order.getOrderId() % 10);
    rabbitTemplate.convertAndSend("order.exchange", routingKey, order);
}
```

### Q3: 如何实现延迟队列？
**A**: 有两种方式：
1. **死信队列+TTL**: 利用消息过期后进入死信队列的特性
2. **RabbitMQ延迟插件**: 安装rabbitmq_delayed_message_exchange插件

```bash
# 安装延迟插件
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

### Q4: 消息堆积如何处理？
**A**: 
1. **临时方案**: 
   - 增加消费者数量
   - 提高消费者处理速度
   - 临时扩容队列
2. **长期方案**:
   - 优化消费者业务逻辑
   - 设置合理的预取数量
   - 设置队列最大长度和TTL
   - 监控告警机制

### Q5: RabbitMQ和Kafka如何选择？
**A**: 
- **RabbitMQ适用场景**:
  - 需要复杂路由规则
  - 消息量不是特别大（万级/秒）
  - 需要消息优先级
  - 需要延迟队列
  - 对消息可靠性要求高

- **Kafka适用场景**:
  - 大数据量（十万级/秒以上）
  - 日志收集、流式处理
  - 需要消息回溯
  - 对吞吐量要求极高

### Q6: 如何避免消息重复消费？
**A**: 实现消费幂等性：
1. **数据库唯一约束**: 利用数据库唯一索引
2. **Redis去重**: 使用Redis的SETNX命令
3. **业务ID去重**: 在业务层面判断是否已处理

### Q7: 集群节点宕机如何处理？
**A**: 
1. **镜像队列**: 配置镜像队列策略，消息会同步到多个节点
2. **客户端重连**: 配置自动重连机制
3. **监控告警**: 及时发现节点故障

```java
// 配置自动重连
factory.setAutomaticRecoveryEnabled(true);
factory.setNetworkRecoveryInterval(5000);
```

## 🔗 相关资源

### 官方资源
- [RabbitMQ官方文档](https://www.rabbitmq.com/documentation.html)
- [AMQP 0-9-1协议规范](https://www.rabbitmq.com/tutorials/amqp-concepts)
- [RabbitMQ GitHub](https://github.com/rabbitmq/rabbitmq-server)
- [RabbitMQ管理界面文档](https://www.rabbitmq.com/management.html)

### 推荐文章
- [RabbitMQ消息可靠性投递](https://www.rabbitmq.com/reliability.html)
- [RabbitMQ性能优化指南](https://www.rabbitmq.com/performance.html)
- [RabbitMQ集群配置](https://www.rabbitmq.com/clustering.html)

### 视频教程
- [RabbitMQ官方教程](https://www.rabbitmq.com/getstarted.html)
- [尚硅谷RabbitMQ教程](https://www.bilibili.com/video/BV1cb4y1o7zz)

### 书籍推荐
- 《RabbitMQ实战指南》
- 《RabbitMQ in Action》

## 📝 学习检查清单

- [ ] 理解AMQP 0-9-1协议模型
- [ ] 掌握四种交换机类型及应用场景
- [ ] 理解Connection和Channel的区别
- [ ] 掌握消息确认机制（生产者和消费者）
- [ ] 理解并实现死信队列
- [ ] 掌握消息持久化配置
- [ ] 了解RabbitMQ集群架构
- [ ] 能够配置镜像队列
- [ ] 掌握Spring Boot集成RabbitMQ
- [ ] 能够解决消息丢失问题
- [ ] 能够实现消息幂等性
- [ ] 了解性能优化方法
- [ ] 掌握监控和运维技巧

---

**@author erik.zhou**  
**文档版本**: 1.0  
**最后更新**: 2024-12-31
