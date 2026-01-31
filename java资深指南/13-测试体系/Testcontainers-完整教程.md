# Testcontainers 完整教程

> **作者**: erik.zhou  
> **创建时间**: 2025-01-31  
> **技术栈**: Testcontainers 1.19+, Spring Boot 3.x, JUnit 5, Docker

## 📋 目录

- [1. Testcontainers简介](#1-testcontainers简介)
- [2. 快速开始](#2-快速开始)
- [3. 数据库测试](#3-数据库测试)
- [4. 消息队列测试](#4-消息队列测试)
- [5. 微服务集成测试](#5-微服务集成测试)
- [6. 高级特性](#6-高级特性)
- [7. 最佳实践](#7-最佳实践)

---

## 1. Testcontainers简介

### 1.1 什么是Testcontainers

Testcontainers是一个Java库，提供轻量级、一次性的Docker容器实例，用于集成测试。

**核心特性**:
- 🐳 真实环境测试
- 🔄 自动容器管理
- 🚀 快速启动
- 🧪 隔离性好
- 📦 丰富的预配置模块

### 1.2 为什么使用Testcontainers

| 特性 | Testcontainers | 传统Mock | 内嵌数据库 |
|------|----------------|---------|-----------|
| 真实性 | ✅ 真实环境 | ❌ 模拟 | ⚠️ 部分真实 |
| 隔离性 | ✅ 完全隔离 | ✅ 隔离 | ⚠️ 共享 |
| 版本一致 | ✅ 与生产一致 | N/A | ❌ 可能不同 |
| 复杂场景 | ✅ 支持 | ❌ 困难 | ❌ 有限 |
| 学习成本 | ⚠️ 中等 | ✅ 低 | ✅ 低 |

### 1.3 应用场景

- **数据库测试**: MySQL、PostgreSQL、MongoDB
- **消息队列测试**: Kafka、RabbitMQ、Redis
- **搜索引擎测试**: Elasticsearch
- **微服务测试**: 多容器编排
- **端到端测试**: 完整应用栈

---

## 2. 快速开始

### 2.1 Maven依赖

```xml
<properties>
    <testcontainers.version>1.19.0</testcontainers.version>
</properties>

<dependencies>
    <!-- Testcontainers核心 -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>testcontainers</artifactId>
        <version>${testcontainers.version}</version>
        <scope>test</scope>
    </dependency>
    
    <!-- JUnit 5集成 -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>junit-jupiter</artifactId>
        <version>${testcontainers.version}</version>
        <scope>test</scope>
    </dependency>
    
    <!-- MySQL模块 -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>mysql</artifactId>
        <version>${testcontainers.version}</version>
        <scope>test</scope>
    </dependency>
    
    <!-- PostgreSQL模块 -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>postgresql</artifactId>
        <version>${testcontainers.version}</version>
        <scope>test</scope>
    </dependency>

    
    <!-- Kafka模块 -->
    <dependency>
        <groupId>org.testcontainers</groupId>
        <artifactId>kafka</artifactId>
        <version>${testcontainers.version}</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 2.2 第一个测试

```java
package com.example.testcontainers;

import org.junit.jupiter.api.Test;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import org.testcontainers.utility.DockerImageName;

import static org.assertj.core.api.Assertions.assertThat;

/**
 * Testcontainers快速入门
 * 
 * @author erik.zhou
 */
@Testcontainers
class QuickStartTest {
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>(
        DockerImageName.parse("redis:7-alpine")
    ).withExposedPorts(6379);
    
    @Test
    void testRedisConnection() {
        String host = redis.getHost();
        Integer port = redis.getFirstMappedPort();
        
        assertThat(host).isNotNull();
        assertThat(port).isGreaterThan(0);
        
        System.out.println("Redis运行在: " + host + ":" + port);
    }
}
```

---

## 3. 数据库测试

### 3.1 MySQL测试

```java
package com.example.testcontainers.database;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.MySQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import static org.assertj.core.api.Assertions.assertThat;

/**
 * MySQL集成测试
 * 
 * @author erik.zhou
 */
@SpringBootTest
@Testcontainers
class MySQLIntegrationTest {
    
    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test")
        .withInitScript("schema.sql");  // 初始化脚本
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mysql::getJdbcUrl);
        registry.add("spring.datasource.username", mysql::getUsername);
        registry.add("spring.datasource.password", mysql::getPassword);
    }
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    @Test
    void testDatabaseConnection() {
        Integer result = jdbcTemplate.queryForObject(
            "SELECT 1",
            Integer.class
        );
        assertThat(result).isEqualTo(1);
    }
    
    @Test
    void testUserRepository() {
        // 插入数据
        jdbcTemplate.update(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            "张三", "zhangsan@example.com"
        );
        
        // 查询数据
        Integer count = jdbcTemplate.queryForObject(
            "SELECT COUNT(*) FROM users WHERE name = ?",
            Integer.class,
            "张三"
        );
        
        assertThat(count).isEqualTo(1);
    }
}
```

### 3.2 PostgreSQL测试

```java
package com.example.testcontainers.database;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.Statement;

import static org.assertj.core.api.Assertions.assertThat;

/**
 * PostgreSQL集成测试
 * 
 * @author erik.zhou
 */
@SpringBootTest
@Testcontainers
class PostgreSQLIntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private DataSource dataSource;
    
    @Test
    void testPostgreSQLFeatures() throws Exception {
        try (Connection conn = dataSource.getConnection();
             Statement stmt = conn.createStatement()) {
            
            // 测试PostgreSQL特有功能
            stmt.execute("CREATE TABLE IF NOT EXISTS products (id SERIAL PRIMARY KEY, name VARCHAR(100))");
            stmt.execute("INSERT INTO products (name) VALUES ('Product 1')");
            
            ResultSet rs = stmt.executeQuery("SELECT * FROM products");
            assertThat(rs.next()).isTrue();
            assertThat(rs.getString("name")).isEqualTo("Product 1");
        }
    }
}
```

### 3.3 MongoDB测试

```java
package com.example.testcontainers.database;

import com.mongodb.client.MongoClient;
import com.mongodb.client.MongoClients;
import com.mongodb.client.MongoCollection;
import com.mongodb.client.MongoDatabase;
import org.bson.Document;
import org.junit.jupiter.api.Test;
import org.testcontainers.containers.MongoDBContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import static org.assertj.core.api.Assertions.assertThat;

/**
 * MongoDB集成测试
 * 
 * @author erik.zhou
 */
@Testcontainers
class MongoDBIntegrationTest {
    
    @Container
    static MongoDBContainer mongodb = new MongoDBContainer("mongo:7.0");
    
    @Test
    void testMongoDBOperations() {
        // 连接MongoDB
        try (MongoClient client = MongoClients.create(mongodb.getReplicaSetUrl())) {
            MongoDatabase database = client.getDatabase("testdb");
            MongoCollection<Document> collection = database.getCollection("users");
            
            // 插入文档
            Document user = new Document("name", "张三")
                .append("age", 25)
                .append("email", "zhangsan@example.com");
            collection.insertOne(user);
            
            // 查询文档
            Document found = collection.find(new Document("name", "张三")).first();
            assertThat(found).isNotNull();
            assertThat(found.getInteger("age")).isEqualTo(25);
        }
    }
}
```

---

## 4. 消息队列测试

### 4.1 Kafka测试

```java
package com.example.testcontainers.messaging;

import org.apache.kafka.clients.consumer.ConsumerConfig;
import org.apache.kafka.clients.consumer.ConsumerRecord;
import org.apache.kafka.clients.consumer.KafkaConsumer;
import org.apache.kafka.clients.producer.KafkaProducer;
import org.apache.kafka.clients.producer.ProducerConfig;
import org.apache.kafka.clients.producer.ProducerRecord;
import org.apache.kafka.common.serialization.StringDeserializer;
import org.apache.kafka.common.serialization.StringSerializer;
import org.junit.jupiter.api.Test;
import org.testcontainers.containers.KafkaContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import org.testcontainers.utility.DockerImageName;

import java.time.Duration;
import java.util.Collections;
import java.util.Properties;

import static org.assertj.core.api.Assertions.assertThat;

/**
 * Kafka集成测试
 * 
 * @author erik.zhou
 */
@Testcontainers
class KafkaIntegrationTest {
    
    @Container
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.5.0")
    );
    
    @Test
    void testKafkaProducerConsumer() {
        String topic = "test-topic";
        String message = "Hello Kafka!";
        
        // 生产者配置
        Properties producerProps = new Properties();
        producerProps.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, kafka.getBootstrapServers());
        producerProps.put(ProducerConfig.KEY_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        producerProps.put(ProducerConfig.VALUE_SERIALIZER_CLASS_CONFIG, StringSerializer.class);
        
        // 发送消息
        try (KafkaProducer<String, String> producer = new KafkaProducer<>(producerProps)) {
            producer.send(new ProducerRecord<>(topic, "key", message));
            producer.flush();
        }
        
        // 消费者配置
        Properties consumerProps = new Properties();
        consumerProps.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, kafka.getBootstrapServers());
        consumerProps.put(ConsumerConfig.GROUP_ID_CONFIG, "test-group");
        consumerProps.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
        consumerProps.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        consumerProps.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
        
        // 接收消息
        try (KafkaConsumer<String, String> consumer = new KafkaConsumer<>(consumerProps)) {
            consumer.subscribe(Collections.singletonList(topic));
            
            ConsumerRecord<String, String> record = consumer.poll(Duration.ofSeconds(10))
                .iterator()
                .next();
            
            assertThat(record.value()).isEqualTo(message);
        }
    }
}
```

### 4.2 RabbitMQ测试

```java
package com.example.testcontainers.messaging;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.GetResponse;
import org.junit.jupiter.api.Test;
import org.testcontainers.containers.RabbitMQContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import static org.assertj.core.api.Assertions.assertThat;

/**
 * RabbitMQ集成测试
 * 
 * @author erik.zhou
 */
@Testcontainers
class RabbitMQIntegrationTest {
    
    @Container
    static RabbitMQContainer rabbitmq = new RabbitMQContainer("rabbitmq:3.12-management-alpine");
    
    @Test
    void testRabbitMQMessaging() throws Exception {
        String queueName = "test-queue";
        String message = "Hello RabbitMQ!";
        
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(rabbitmq.getHost());
        factory.setPort(rabbitmq.getAmqpPort());
        factory.setUsername(rabbitmq.getAdminUsername());
        factory.setPassword(rabbitmq.getAdminPassword());
        
        try (Connection connection = factory.newConnection();
             Channel channel = connection.createChannel()) {
            
            // 声明队列
            channel.queueDeclare(queueName, false, false, false, null);
            
            // 发送消息
            channel.basicPublish("", queueName, null, message.getBytes());
            
            // 接收消息
            GetResponse response = channel.basicGet(queueName, true);
            String receivedMessage = new String(response.getBody());
            
            assertThat(receivedMessage).isEqualTo(message);
        }
    }
}
```

### 4.3 Redis测试

```java
package com.example.testcontainers.cache;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;
import org.testcontainers.utility.DockerImageName;

import static org.assertj.core.api.Assertions.assertThat;

/**
 * Redis集成测试
 * 
 * @author erik.zhou
 */
@SpringBootTest
@Testcontainers
class RedisIntegrationTest {
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>(
        DockerImageName.parse("redis:7-alpine")
    ).withExposedPorts(6379);
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", redis::getFirstMappedPort);
    }
    
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    @Test
    void testRedisOperations() {
        String key = "test:key";
        String value = "test-value";
        
        // 设置值
        redisTemplate.opsForValue().set(key, value);
        
        // 获取值
        String retrieved = redisTemplate.opsForValue().get(key);
        assertThat(retrieved).isEqualTo(value);
    }
}
```

---

## 5. 微服务集成测试

### 5.1 多容器编排

```java
package com.example.testcontainers.microservice;

import org.junit.jupiter.api.Test;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.containers.Network;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import static org.assertj.core.api.Assertions.assertThat;

/**
 * 多容器编排测试
 * 
 * @author erik.zhou
 */
@Testcontainers
class MultiContainerTest {
    
    // 创建网络
    static Network network = Network.newNetwork();
    
    // 数据库容器
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withNetwork(network)
        .withNetworkAliases("postgres");
    
    // 应用容器
    @Container
    static GenericContainer<?> app = new GenericContainer<>("myapp:latest")
        .withNetwork(network)
        .withEnv("DB_HOST", "postgres")
        .withEnv("DB_PORT", "5432")
        .withExposedPorts(8080)
        .dependsOn(postgres);
    
    @Test
    void testMultiContainerSetup() {
        String appUrl = "http://" + app.getHost() + ":" + app.getFirstMappedPort();
        assertThat(appUrl).isNotNull();
    }
}
```

### 5.2 Docker Compose集成

```java
package com.example.testcontainers.compose;

import org.junit.jupiter.api.Test;
import org.testcontainers.containers.ComposeContainer;
import org.testcontainers.containers.wait.strategy.Wait;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import java.io.File;

/**
 * Docker Compose集成测试
 * 
 * @author erik.zhou
 */
@Testcontainers
class DockerComposeTest {
    
    @Container
    static ComposeContainer environment = new ComposeContainer(
        new File("src/test/resources/docker-compose-test.yml")
    )
        .withExposedService("app", 8080, Wait.forHttp("/health"))
        .withExposedService("db", 5432, Wait.forListeningPort());
    
    @Test
    void testDockerComposeSetup() {
        String appHost = environment.getServiceHost("app", 8080);
        Integer appPort = environment.getServicePort("app", 8080);
        
        System.out.println("App运行在: " + appHost + ":" + appPort);
    }
}
```

---

## 6. 高级特性

### 6.1 自定义容器

```java
package com.example.testcontainers.custom;

import org.testcontainers.containers.GenericContainer;
import org.testcontainers.containers.wait.strategy.Wait;
import org.testcontainers.utility.DockerImageName;

/**
 * 自定义容器
 * 
 * @author erik.zhou
 */
public class CustomAppContainer extends GenericContainer<CustomAppContainer> {
    
    private static final int DEFAULT_PORT = 8080;
    
    public CustomAppContainer() {
        this(DockerImageName.parse("myapp:latest"));
    }
    
    public CustomAppContainer(DockerImageName dockerImageName) {
        super(dockerImageName);
        
        // 配置容器
        withExposedPorts(DEFAULT_PORT);
        waitingFor(Wait.forHttp("/health").forStatusCode(200));
        withEnv("SPRING_PROFILES_ACTIVE", "test");
    }
    
    public String getAppUrl() {
        return "http://" + getHost() + ":" + getMappedPort(DEFAULT_PORT);
    }
}
```

### 6.2 容器复用

```java
package com.example.testcontainers.reuse;

import org.junit.jupiter.api.Test;
import org.testcontainers.containers.PostgreSQLContainer;

/**
 * 容器复用示例
 * 
 * @author erik.zhou
 */
class ContainerReuseTest {
    
    // 启用容器复用
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withReuse(true);  // 关键配置
    
    @Test
    void test1() {
        postgres.start();
        // 测试逻辑
    }
    
    @Test
    void test2() {
        postgres.start();  // 复用已启动的容器
        // 测试逻辑
    }
}
```

### 6.3 日志收集

```java
package com.example.testcontainers.logging;

import org.junit.jupiter.api.Test;
import org.testcontainers.containers.GenericContainer;
import org.testcontainers.containers.output.Slf4jLogConsumer;
import org.testcontainers.junit.jupiter.Container;
import org.testcontainers.junit.jupiter.Testcontainers;

import lombok.extern.slf4j.Slf4j;

/**
 * 容器日志收集
 * 
 * @author erik.zhou
 */
@Slf4j
@Testcontainers
class LoggingTest {
    
    @Container
    static GenericContainer<?> app = new GenericContainer<>("myapp:latest")
        .withExposedPorts(8080)
        .withLogConsumer(new Slf4jLogConsumer(log));
    
    @Test
    void testWithLogging() {
        // 容器日志会自动输出到测试日志
        log.info("测试开始");
    }
}
```

---

## 7. 最佳实践

### 7.1 基础测试类

```java
package com.example.testcontainers.base;

import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.DynamicPropertyRegistry;
import org.springframework.test.context.DynamicPropertySource;
import org.testcontainers.containers.PostgreSQLContainer;
import org.testcontainers.junit.jupiter.Testcontainers;

/**
 * 基础测试类 - 所有集成测试继承此类
 * 
 * @author erik.zhou
 */
@SpringBootTest
@Testcontainers
public abstract class BaseIntegrationTest {
    
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withReuse(true);
    
    static {
        postgres.start();
    }
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }
}
```

### 7.2 性能优化

```java
/**
 * Testcontainers性能优化建议
 * 
 * @author erik.zhou
 */
public class PerformanceOptimization {
    
    /*
     * 1. 使用容器复用
     *    - withReuse(true)
     *    - 减少容器启动次数
     * 
     * 2. 使用轻量级镜像
     *    - alpine版本
     *    - 减少镜像大小
     * 
     * 3. 并行测试
     *    - JUnit 5并行执行
     *    - 注意资源隔离
     * 
     * 4. 使用静态容器
     *    - static Container
     *    - 测试类间共享
     * 
     * 5. 预热容器
     *    - 提前拉取镜像
     *    - docker pull命令
     */
}
```

---

## 8. 总结

Testcontainers是现代Java集成测试的最佳实践:

### 核心优势
- ✅ 真实环境测试
- ✅ 自动化容器管理
- ✅ 版本一致性
- ✅ 隔离性好
- ✅ 易于使用

### 使用建议
1. 优先使用预配置模块
2. 合理使用容器复用
3. 注意资源清理
4. 关注测试性能
5. 结合CI/CD流程

---

**作者**: erik.zhou  
**最后更新**: 2025-01-31  
**版本**: 1.0.0
