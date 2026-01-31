# Dubbo 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [相关资源](#相关资源)
- [学习检查清单](#学习检查清单)

## 📚 技术概述
- **版本**: Dubbo 3.3.x
- **官方文档**: https://dubbo.apache.org/
- **GitHub**: https://github.com/apache/dubbo
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Java基础
  - Spring/Spring Boot
  - 网络编程基础
  - 分布式系统基础概念
  - Zookeeper或Nacos（注册中心）

## 🎯 学习目标
- [ ] 理解Dubbo的核心架构和工作原理
- [ ] 掌握服务提供者和消费者的开发
- [ ] 熟练使用负载均衡和容错机制
- [ ] 理解SPI扩展机制
- [ ] 掌握协议选择和性能优化
- [ ] 能够进行服务治理和监控

## 📖 基础概念

### 1.1 什么是Dubbo

Apache Dubbo 是一款高性能、轻量级的开源Java RPC框架，提供了三大核心能力：
- **面向接口的远程方法调用**：像调用本地方法一样调用远程服务
- **智能容错和负载均衡**：提供多种负载均衡策略和容错机制
- **服务自动注册和发现**：支持多种注册中心，实现服务的动态上下线

Dubbo 3.x 是新一代云原生微服务框架，完全兼容gRPC协议，支持HTTP/2、Protocol Buffers等现代化特性。

### 1.2 核心概念

- **Provider（服务提供者）**: 暴露服务的服务提供方，向注册中心注册自己提供的服务
- **Consumer（服务消费者）**: 调用远程服务的服务消费方，从注册中心订阅所需服务
- **Registry（注册中心）**: 服务注册与发现的中心节点，支持Zookeeper、Nacos、Etcd等
- **Monitor（监控中心）**: 统计服务调用次数和调用时间的监控中心
- **Container（服务容器）**: 服务运行的容器，负责服务的启动、加载和运行

### 1.3 应用场景

- **微服务架构**: 作为微服务间通信的RPC框架
- **分布式系统**: 实现分布式服务的调用和治理
- **服务化改造**: 将单体应用拆分为服务化架构
- **跨语言通信**: 通过Triple协议实现与gRPC的互通
- **高并发场景**: 利用异步调用和连接池提升性能

## 🔥 核心特性

### 2.1 RPC协议支持 🔥

Dubbo 3.x 支持多种RPC协议：

**Triple协议（推荐）**:
- 完全兼容gRPC
- 基于HTTP/2，支持流式调用
- 支持Protocol Buffers序列化
- 云原生友好

```java
@DubboService(protocol = "tri")
public class UserServiceImpl implements UserService {
    @Override
    public User getUserById(Long id) {
        // 业务逻辑
        return new User(id, "张三");
    }
}
```

**Dubbo协议**:
- 基于TCP的二进制协议
- 高性能、低延迟
- 适合内网高并发场景

```yaml
dubbo:
  protocol:
    name: dubbo
    port: 20880
```

**REST协议**:
- 基于HTTP的RESTful风格
- 跨语言、跨平台
- 适合对外开放的API

### 2.2 服务暴露与引用 🔥

**服务提供者（Provider）**:


```java
package com.example.provider;

import org.apache.dubbo.config.annotation.DubboService;

/**
 * 服务提供者实现
 * @author erik.zhou
 */
@DubboService(version = "1.0.0", timeout = 3000)
public class UserServiceImpl implements UserService {
    
    @Override
    public User getUserById(Long id) {
        // 模拟数据库查询
        User user = new User();
        user.setId(id);
        user.setName("用户" + id);
        return user;
    }
}
```

**服务消费者（Consumer）**:

```java
package com.example.consumer;

import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.stereotype.Service;

/**
 * 服务消费者
 * @author erik.zhou
 */
@Service
public class UserController {
    
    @DubboReference(version = "1.0.0", timeout = 3000)
    private UserService userService;
    
    public User getUser(Long id) {
        // 像调用本地方法一样调用远程服务
        return userService.getUserById(id);
    }
}
```

### 2.3 负载均衡策略 🔥

Dubbo提供多种负载均衡算法：

**1. Random（随机）- 默认策略**:
```java
@DubboReference(loadbalance = "random")
private UserService userService;
```

**2. RoundRobin（轮询）**:
```java
@DubboReference(loadbalance = "roundrobin")
private UserService userService;
```

**3. LeastActive（最少活跃调用数）**:
```java
@DubboReference(loadbalance = "leastactive")
private UserService userService;
```

**4. ConsistentHash（一致性哈希）**:
```java
@DubboReference(loadbalance = "consistenthash")
private UserService userService;
```

### 2.4 容错机制 🔥

**Failover（失败自动切换）- 默认策略**:
```java
@DubboService(cluster = "failover", retries = 2)
public class UserServiceImpl implements UserService {
    // 失败后自动切换到其他服务器重试
}
```

**Failfast（快速失败）**:
```java
@DubboService(cluster = "failfast")
public class UserServiceImpl implements UserService {
    // 只发起一次调用，失败立即报错
}
```

**Failsafe（失败安全）**:
```java
@DubboService(cluster = "failsafe")
public class UserServiceImpl implements UserService {
    // 出现异常时忽略，返回空结果
}
```

**Failback（失败自动恢复）**:
```java
@DubboService(cluster = "failback")
public class UserServiceImpl implements UserService {
    // 失败后记录请求，定时重发
}
```

### 2.5 SPI扩展机制 ⚠️ 难点

Dubbo的SPI（Service Provider Interface）是其核心扩展机制，允许用户自定义实现。

**SPI机制原理**:
- Dubbo改进了JDK的SPI机制
- 支持按需加载扩展点
- 支持依赖注入和AOP
- 支持自适应扩展

**自定义负载均衡策略示例**:

```java
package com.example.loadbalance;

import org.apache.dubbo.common.URL;
import org.apache.dubbo.rpc.Invocation;
import org.apache.dubbo.rpc.Invoker;
import org.apache.dubbo.rpc.cluster.loadbalance.AbstractLoadBalance;

import java.util.List;

/**
 * 自定义负载均衡策略
 * @author erik.zhou
 */
public class CustomLoadBalance extends AbstractLoadBalance {
    
    @Override
    protected <T> Invoker<T> doSelect(List<Invoker<T>> invokers, 
                                      URL url, 
                                      Invocation invocation) {
        // 自定义负载均衡逻辑
        // 例如：根据业务参数选择服务器
        String userId = invocation.getArguments()[0].toString();
        int index = Math.abs(userId.hashCode()) % invokers.size();
        return invokers.get(index);
    }
}
```

**配置SPI扩展**:

在 `resources/META-INF/dubbo/org.apache.dubbo.rpc.cluster.LoadBalance` 文件中：
```
custom=com.example.loadbalance.CustomLoadBalance
```

使用自定义扩展：
```java
@DubboReference(loadbalance = "custom")
private UserService userService;
```

### 2.6 协议选择 ⚠️ 难点

**协议对比**:

| 协议 | 传输协议 | 序列化 | 适用场景 | 性能 |
|------|---------|--------|---------|------|
| Triple | HTTP/2 | Protobuf/JSON | 云原生、跨语言 | 高 |
| Dubbo | TCP | Hessian2 | 内网高并发 | 极高 |
| REST | HTTP | JSON | 对外开放API | 中 |
| gRPC | HTTP/2 | Protobuf | 跨语言通信 | 高 |

**协议选择建议**:
- **内网微服务**: 优先使用Dubbo协议（性能最优）
- **云原生场景**: 使用Triple协议（兼容gRPC）
- **跨语言调用**: 使用Triple或gRPC协议
- **对外API**: 使用REST协议

### 2.7 异步调用

Dubbo支持异步调用，提升系统吞吐量：

```java
/**
 * 异步服务接口
 * @author erik.zhou
 */
public interface AsyncUserService {
    CompletableFuture<User> getUserByIdAsync(Long id);
}

/**
 * 异步服务实现
 * @author erik.zhou
 */
@DubboService
public class AsyncUserServiceImpl implements AsyncUserService {
    
    @Override
    public CompletableFuture<User> getUserByIdAsync(Long id) {
        return CompletableFuture.supplyAsync(() -> {
            // 异步执行业务逻辑
            User user = new User();
            user.setId(id);
            user.setName("用户" + id);
            return user;
        });
    }
}

/**
 * 异步调用示例
 * @author erik.zhou
 */
@Service
public class UserController {
    
    @DubboReference
    private AsyncUserService asyncUserService;
    
    public void asyncGetUser(Long id) {
        asyncUserService.getUserByIdAsync(id)
            .thenAccept(user -> {
                System.out.println("获取到用户: " + user.getName());
            })
            .exceptionally(e -> {
                System.err.println("调用失败: " + e.getMessage());
                return null;
            });
    }
}
```

## 💻 实战应用

### 3.1 环境搭建

**1. 添加Maven依赖**:

```xml
<dependencies>
    <!-- Dubbo Spring Boot Starter -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-spring-boot-starter</artifactId>
        <version>3.3.0</version>
    </dependency>
    
    <!-- Nacos注册中心 -->
    <dependency>
        <groupId>com.alibaba.nacos</groupId>
        <artifactId>nacos-client</artifactId>
        <version>2.3.0</version>
    </dependency>
    
    <!-- Zookeeper注册中心（可选） -->
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo-dependencies-zookeeper</artifactId>
        <version>3.3.0</version>
        <type>pom</type>
    </dependency>
</dependencies>
```

**2. 启动注册中心**:

使用Nacos（推荐）:
```bash
# 下载Nacos
wget https://github.com/alibaba/nacos/releases/download/2.3.0/nacos-server-2.3.0.tar.gz

# 解压并启动
tar -xzf nacos-server-2.3.0.tar.gz
cd nacos/bin
sh startup.sh -m standalone
```

访问控制台: http://localhost:8848/nacos （用户名/密码: nacos/nacos）

### 3.2 快速开始

**步骤1: 定义服务接口**

```java
package com.example.api;

/**
 * 用户服务接口
 * @author erik.zhou
 */
public interface UserService {
    
    /**
     * 根据ID获取用户
     * @param id 用户ID
     * @return 用户信息
     */
    User getUserById(Long id);
    
    /**
     * 创建用户
     * @param user 用户信息
     * @return 是否成功
     */
    boolean createUser(User user);
}
```

**步骤2: 实现服务提供者**

```java
package com.example.provider;

import com.example.api.UserService;
import org.apache.dubbo.config.annotation.DubboService;

/**
 * 用户服务实现
 * @author erik.zhou
 */
@DubboService(
    version = "1.0.0",
    timeout = 3000,
    retries = 2,
    loadbalance = "random"
)
public class UserServiceImpl implements UserService {
    
    @Override
    public User getUserById(Long id) {
        // 模拟数据库查询
        User user = new User();
        user.setId(id);
        user.setName("张三");
        user.setAge(25);
        return user;
    }
    
    @Override
    public boolean createUser(User user) {
        // 模拟数据库插入
        System.out.println("创建用户: " + user.getName());
        return true;
    }
}
```

**配置文件（application.yml）**:

```yaml
spring:
  application:
    name: dubbo-provider

dubbo:
  application:
    name: dubbo-provider
    # 启用QoS（Quality of Service）
    qos-enable: true
    qos-port: 22222
  
  # 协议配置
  protocol:
    name: tri  # 使用Triple协议
    port: 20880
  
  # 注册中心配置
  registry:
    address: nacos://localhost:8848
    # 或使用Zookeeper
    # address: zookeeper://localhost:2181
  
  # 元数据中心配置
  metadata-report:
    address: nacos://localhost:8848
  
  # 配置中心
  config-center:
    address: nacos://localhost:8848
```

**启动类**:

```java
package com.example.provider;

import org.apache.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * 服务提供者启动类
 * @author erik.zhou
 */
@SpringBootApplication
@EnableDubbo
public class ProviderApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
        System.out.println("Dubbo Provider 启动成功！");
    }
}
```

**步骤3: 实现服务消费者**

```java
package com.example.consumer;

import com.example.api.UserService;
import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.web.bind.annotation.*;

/**
 * 用户控制器
 * @author erik.zhou
 */
@RestController
@RequestMapping("/user")
public class UserController {
    
    @DubboReference(
        version = "1.0.0",
        timeout = 3000,
        retries = 2,
        check = false  // 启动时不检查服务是否可用
    )
    private UserService userService;
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUserById(id);
    }
    
    @PostMapping
    public boolean createUser(@RequestBody User user) {
        return userService.createUser(user);
    }
}
```

**消费者配置（application.yml）**:

```yaml
spring:
  application:
    name: dubbo-consumer

server:
  port: 8080

dubbo:
  application:
    name: dubbo-consumer
  
  # 注册中心配置
  registry:
    address: nacos://localhost:8848
  
  # 消费者配置
  consumer:
    timeout: 3000
    check: false  # 启动时不检查服务
    retries: 2
```

**消费者启动类**:

```java
package com.example.consumer;

import org.apache.dubbo.config.spring.context.annotation.EnableDubbo;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * 服务消费者启动类
 * @author erik.zhou
 */
@SpringBootApplication
@EnableDubbo
public class ConsumerApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
        System.out.println("Dubbo Consumer 启动成功！");
    }
}
```

### 3.3 进阶案例

**案例1: 服务分组**

当同一个接口有多个实现时，可以使用分组区分：

```java
// 提供者端
@DubboService(group = "vip")
public class VipUserServiceImpl implements UserService {
    // VIP用户服务实现
}

@DubboService(group = "normal")
public class NormalUserServiceImpl implements UserService {
    // 普通用户服务实现
}

// 消费者端
@DubboReference(group = "vip")
private UserService vipUserService;

@DubboReference(group = "normal")
private UserService normalUserService;
```

**案例2: 服务降级**

```java
package com.example.consumer;

import com.example.api.UserService;
import org.apache.dubbo.config.annotation.DubboReference;
import org.springframework.stereotype.Service;

/**
 * 带降级的服务调用
 * @author erik.zhou
 */
@Service
public class UserServiceWrapper {
    
    @DubboReference(
        version = "1.0.0",
        timeout = 3000,
        mock = "com.example.consumer.UserServiceMock"
    )
    private UserService userService;
    
    public User getUserById(Long id) {
        try {
            return userService.getUserById(id);
        } catch (Exception e) {
            // 降级处理
            return getDefaultUser(id);
        }
    }
    
    private User getDefaultUser(Long id) {
        User user = new User();
        user.setId(id);
        user.setName("默认用户");
        return user;
    }
}

/**
 * Mock实现类
 * @author erik.zhou
 */
public class UserServiceMock implements UserService {
    
    @Override
    public User getUserById(Long id) {
        User user = new User();
        user.setId(id);
        user.setName("Mock用户");
        return user;
    }
    
    @Override
    public boolean createUser(User user) {
        return false;
    }
}
```

**案例3: 泛化调用**

不需要服务接口，直接通过GenericService调用：

```java
package com.example.consumer;

import org.apache.dubbo.config.ApplicationConfig;
import org.apache.dubbo.config.ReferenceConfig;
import org.apache.dubbo.config.RegistryConfig;
import org.apache.dubbo.rpc.service.GenericService;

/**
 * 泛化调用示例
 * @author erik.zhou
 */
public class GenericInvokeExample {
    
    public static void main(String[] args) {
        // 配置应用
        ApplicationConfig application = new ApplicationConfig();
        application.setName("generic-consumer");
        
        // 配置注册中心
        RegistryConfig registry = new RegistryConfig();
        registry.setAddress("nacos://localhost:8848");
        
        // 配置引用
        ReferenceConfig<GenericService> reference = new ReferenceConfig<>();
        reference.setApplication(application);
        reference.setRegistry(registry);
        reference.setInterface("com.example.api.UserService");
        reference.setVersion("1.0.0");
        reference.setGeneric("true");  // 声明为泛化接口
        
        // 获取泛化服务
        GenericService genericService = reference.get();
        
        // 调用方法
        Object result = genericService.$invoke(
            "getUserById",  // 方法名
            new String[]{"java.lang.Long"},  // 参数类型
            new Object[]{1L}  // 参数值
        );
        
        System.out.println("调用结果: " + result);
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

**1. 连接数优化**:

```yaml
dubbo:
  provider:
    # 服务提供者每个服务的最大连接数
    connections: 10
  
  consumer:
    # 服务消费者对每个提供者的最大连接数
    connections: 5
```

**2. 线程池优化**:

```yaml
dubbo:
  protocol:
    name: dubbo
    port: 20880
    # 线程池类型：fixed（固定大小）、cached（缓存线程池）、limited（弹性线程池）
    threadpool: fixed
    # 线程池核心线程数
    threads: 200
    # 线程池队列大小
    queues: 0
```

**3. 序列化优化**:

```yaml
dubbo:
  protocol:
    name: dubbo
    # 序列化方式：hessian2（默认）、fastjson、kryo、fst
    serialization: kryo
```

**4. 启用本地缓存**:

```java
@DubboReference(
    cache = "lru",  // 启用LRU缓存
    cacheSize = 1000  // 缓存大小
)
private UserService userService;
```

### 4.2 常见陷阱

**⚠️ 陷阱1: 服务超时设置不当**

```java
// 错误：超时时间过短
@DubboService(timeout = 100)  // 100ms可能不够
public class UserServiceImpl implements UserService {
    // ...
}

// 正确：根据业务设置合理的超时时间
@DubboService(timeout = 3000)  // 3秒
public class UserServiceImpl implements UserService {
    // ...
}
```

**⚠️ 陷阱2: 重试次数设置不当**

```java
// 错误：对非幂等操作设置重试
@DubboService(retries = 2)
public class OrderServiceImpl implements OrderService {
    @Override
    public void createOrder(Order order) {
        // 创建订单是非幂等操作，不应该重试
    }
}

// 正确：非幂等操作不设置重试
@DubboService(retries = 0)
public class OrderServiceImpl implements OrderService {
    @Override
    public void createOrder(Order order) {
        // 创建订单
    }
}

// 正确：幂等操作可以设置重试
@DubboService(retries = 2)
public class OrderServiceImpl implements OrderService {
    @Override
    public Order getOrderById(Long id) {
        // 查询订单是幂等操作，可以重试
        return null;
    }
}
```

**⚠️ 陷阱3: 忘记设置版本号**

```java
// 错误：不设置版本号，升级时可能出现兼容性问题
@DubboService
public class UserServiceImpl implements UserService {
    // ...
}

// 正确：始终设置版本号
@DubboService(version = "1.0.0")
public class UserServiceImpl implements UserService {
    // ...
}

// 升级时使用新版本号
@DubboService(version = "2.0.0")
public class UserServiceImplV2 implements UserService {
    // ...
}
```

**⚠️ 陷阱4: 服务接口返回值使用内部类**

```java
// 错误：返回值使用内部类，序列化可能失败
public class UserService {
    public class User {  // 内部类
        private Long id;
        private String name;
    }
    
    User getUserById(Long id);
}

// 正确：使用独立的类
public class User {  // 独立的类
    private Long id;
    private String name;
}

public interface UserService {
    User getUserById(Long id);
}
```

### 4.3 监控与运维

**1. 启用Dubbo Admin**:

```bash
# 下载Dubbo Admin
git clone https://github.com/apache/dubbo-admin.git
cd dubbo-admin

# 修改配置
vim dubbo-admin-server/src/main/resources/application.properties

# 配置注册中心地址
admin.registry.address=nacos://localhost:8848
admin.config-center=nacos://localhost:8848
admin.metadata-report.address=nacos://localhost:8848

# 启动
mvn clean package
java -jar dubbo-admin-server/target/dubbo-admin-server-0.6.0.jar
```

访问控制台: http://localhost:8080

**2. 集成Prometheus监控**:

```xml
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-metrics-prometheus</artifactId>
    <version>3.3.0</version>
</dependency>
```

```yaml
dubbo:
  metrics:
    protocol: prometheus
    port: 9090
```

**3. 日志配置**:

```yaml
logging:
  level:
    org.apache.dubbo: INFO
    # 开启调试日志
    # org.apache.dubbo: DEBUG
```

### 4.4 安全配置

**1. 启用Token验证**:

```java
// 提供者端
@DubboService(token = "true")
public class UserServiceImpl implements UserService {
    // ...
}

// 消费者端会自动携带token
```

**2. 启用访问控制**:

```java
// 提供者端
@DubboService(
    filter = "accesslog",  // 记录访问日志
    accesslog = "true"
)
public class UserServiceImpl implements UserService {
    // ...
}
```

**3. 启用SSL/TLS**:

```yaml
dubbo:
  protocol:
    name: tri
    port: 20880
    ssl-enabled: true
```

## ❓ 常见问题

### Q1: Dubbo和Spring Cloud有什么区别？

**A**: 
- **Dubbo**: 专注于RPC调用，性能更高，适合内网高并发场景
- **Spring Cloud**: 提供完整的微服务解决方案，包括配置中心、网关、熔断器等
- **选择建议**: 
  - 内网微服务通信：优先Dubbo
  - 需要完整微服务生态：选择Spring Cloud
  - 两者可以结合使用：Dubbo负责RPC，Spring Cloud提供其他组件

### Q2: 如何选择注册中心？

**A**:
- **Nacos**: 推荐，功能最全（注册中心+配置中心），云原生友好
- **Zookeeper**: 稳定可靠，但运维复杂
- **Consul**: 支持多数据中心，适合跨地域部署
- **Etcd**: 云原生场景，与Kubernetes集成好

### Q3: 服务调用失败如何排查？

**A**:
1. 检查注册中心是否正常
2. 检查服务是否已注册（通过Dubbo Admin查看）
3. 检查网络连通性（telnet provider_ip provider_port）
4. 检查超时配置是否合理
5. 查看日志（开启DEBUG级别）
6. 使用Dubbo QoS命令排查：`telnet localhost 22222`

### Q4: 如何实现服务的灰度发布？

**A**:
```java
// 使用标签路由实现灰度发布
// 提供者端打标签
@DubboService(parameters = {"tag", "gray"})
public class UserServiceImpl implements UserService {
    // 灰度版本
}

// 消费者端指定标签
@DubboReference(parameters = {"tag", "gray"})
private UserService userService;
```

### Q5: Dubbo 3.x相比2.x有哪些重大变化？

**A**:
- **Triple协议**: 完全兼容gRPC，支持HTTP/2
- **应用级服务发现**: 减少注册中心压力
- **云原生支持**: 更好地支持Kubernetes
- **性能提升**: 优化了序列化和网络传输
- **向后兼容**: 完全兼容Dubbo 2.x

## 🔗 相关资源

- **官方文档**: https://dubbo.apache.org/zh/
- **GitHub仓库**: https://github.com/apache/dubbo
- **Dubbo Admin**: https://github.com/apache/dubbo-admin
- **示例代码**: https://github.com/apache/dubbo-samples
- **社区论坛**: https://github.com/apache/dubbo/discussions
- **中文社区**: https://dubbo.apache.org/zh/community/

## 📝 学习检查清单

- [ ] 理解Dubbo的核心架构（Provider、Consumer、Registry、Monitor）
- [ ] 掌握服务提供者和消费者的开发
- [ ] 熟练使用@DubboService和@DubboReference注解
- [ ] 理解并能配置负载均衡策略
- [ ] 理解并能配置容错机制
- [ ] 掌握服务分组和版本管理
- [ ] 理解SPI扩展机制
- [ ] 能够选择合适的协议（Triple、Dubbo、REST）
- [ ] 掌握异步调用的使用
- [ ] 能够进行性能优化（连接数、线程池、序列化）
- [ ] 掌握服务降级和Mock的使用
- [ ] 能够使用Dubbo Admin进行服务治理
- [ ] 理解泛化调用的使用场景
- [ ] 掌握常见问题的排查方法

---

**文档版本**: v1.0.0  
**最后更新**: 2024-01-04  
**文档来源**: Apache Dubbo官方文档 + Context7  
**作者**: @author erik.zhou
