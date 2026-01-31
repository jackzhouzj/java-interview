# Nacos 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)

## 📚 技术概述
- **版本**: 2.3.0
- **官方文档**: https://nacos.io/zh-cn/docs/what-is-nacos.html
- **GitHub**: https://github.com/alibaba/nacos
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: Spring Boot、Spring Cloud、微服务架构基础
- **文档来源**: Context7 + 官方文档
- **更新时间**: 2024-12-31
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Nacos的核心概念和架构设计
- [ ] 掌握Nacos作为配置中心的使用方法
- [ ] 掌握Nacos作为服务注册中心的使用方法
- [ ] 掌握Nacos集群部署和高可用配置
- [ ] 理解命名空间、分组、DataId的设计理念
- [ ] 掌握Nacos在Spring Cloud中的集成方式
- [ ] 了解Nacos的最佳实践和性能优化

## 📖 基础概念

### 1.1 什么是Nacos

Nacos（Dynamic Naming and Configuration Service）是阿里巴巴开源的一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台。

**核心功能**：
- **服务发现和服务健康监测**：支持基于DNS和基于RPC的服务发现
- **动态配置服务**：以中心化、外部化和动态化的方式管理所有环境的应用配置
- **动态DNS服务**：支持权重路由，更容易实现中间层负载均衡
- **服务及其元数据管理**：管理数据中心的所有服务及元数据

### 1.2 核心概念

#### 命名空间（Namespace）
用于进行租户粒度的配置隔离。不同的命名空间下，可以存在相同的Group或DataId的配置。

**使用场景**：
- 开发环境、测试环境、生产环境的配置隔离
- 不同租户之间的配置隔离

#### 配置分组（Group）
Nacos中的一组配置集，是组织配置的维度之一。通过一个有意义的字符串对配置集进行分组。

**默认分组**：DEFAULT_GROUP

**使用场景**：
- 按照业务模块分组：订单服务组、用户服务组
- 按照功能分组：数据库配置组、缓存配置组

#### 配置集ID（Data ID）
Nacos中的某个配置集的ID。配置集ID是组织划分配置的维度之一。

**命名规则**：通常采用类似 `${prefix}-${spring.profiles.active}.${file-extension}` 的格式
- prefix：默认为 spring.application.name 的值
- spring.profiles.active：当前环境对应的profile
- file-extension：配置内容的数据格式，如properties、yaml等

**示例**：
- `user-service-dev.yaml`
- `order-service-prod.properties`

#### 服务（Service）
通过预定义接口网络访问的提供给客户端的软件功能。

#### 实例（Instance）
提供一个或多个服务的具有可访问网络地址（IP:Port）的进程。

### 1.3 应用场景

#### 配置中心场景
- 微服务配置集中管理
- 配置动态刷新
- 灰度发布配置
- 配置版本管理和回滚

#### 服务注册发现场景
- 微服务注册与发现
- 服务健康检查
- 负载均衡
- 服务路由

#### 服务管理场景
- 服务元数据管理
- 服务流量管理
- 服务降级和熔断

## 🔥 核心特性 (重点)

### 2.1 配置管理 🔥

#### 2.1.1 配置的CRUD操作

**发布配置**：
```bash
# 使用API发布配置
curl -X POST "http://127.0.0.1:8848/nacos/v1/cs/configs" \
  -d "dataId=user-service.yaml" \
  -d "group=DEFAULT_GROUP" \
  -d "content=server.port=8080"
```

**获取配置**：
```bash
curl -X GET "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=user-service.yaml&group=DEFAULT_GROUP"
```

**删除配置**：
```bash
curl -X DELETE "http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=user-service.yaml&group=DEFAULT_GROUP"
```

#### 2.1.2 配置监听机制

Nacos客户端会对配置进行长轮询，一旦配置发生变化，客户端会立即收到通知。

**长轮询原理**：
1. 客户端发起长轮询请求，超时时间30秒
2. 服务端hold住请求，如果配置变更则立即返回
3. 如果30秒内无变更，返回空结果
4. 客户端收到响应后立即发起下一次长轮询

#### 2.1.3 配置动态刷新 (⚠️ 难点)

**@RefreshScope注解**：
```java
@RestController
@RefreshScope  // 支持配置动态刷新
public class ConfigController {
    
    @Value("${user.name:default}")
    private String userName;
    
    @Value("${user.age:0}")
    private Integer userAge;
    
    @GetMapping("/config")
    public String getConfig() {
        return "userName: " + userName + ", userAge: " + userAge;
    }
}
```

**注意事项**：
- @RefreshScope会创建代理对象，每次访问时重新获取配置值
- 不要在@RefreshScope的Bean中使用@Scheduled等定时任务
- 静态变量和final变量无法动态刷新

### 2.2 服务注册与发现 🔥

#### 2.2.1 服务注册

**自动注册**：
```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        namespace: dev
        group: DEFAULT_GROUP
        # 服务实例权重，范围1-100，值越大权重越大
        weight: 1
        # 集群名称
        cluster-name: DEFAULT
        # 元数据
        metadata:
          version: 1.0
          region: cn-hangzhou
```

**手动注册**：
```java
@Configuration
public class NacosConfig {
    
    @Bean
    public NamingService namingService() throws NacosException {
        Properties properties = new Properties();
        properties.put("serverAddr", "127.0.0.1:8848");
        properties.put("namespace", "dev");
        return NamingFactory.createNamingService(properties);
    }
}

@Service
public class ServiceRegistry {
    
    @Autowired
    private NamingService namingService;
    
    public void registerInstance() throws NacosException {
        Instance instance = new Instance();
        instance.setIp("192.168.1.100");
        instance.setPort(8080);
        instance.setWeight(1.0);
        instance.setHealthy(true);
        
        Map<String, String> metadata = new HashMap<>();
        metadata.put("version", "1.0");
        instance.setMetadata(metadata);
        
        namingService.registerInstance("user-service", instance);
    }
}
```

#### 2.2.2 服务发现

**使用RestTemplate**：
```java
@Configuration
public class RestTemplateConfig {
    
    @Bean
    @LoadBalanced  // 开启负载均衡
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

@Service
public class UserService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    public String getUser(Long userId) {
        // 使用服务名调用
        String url = "http://user-service/user/" + userId;
        return restTemplate.getForObject(url, String.class);
    }
}
```

**使用OpenFeign**：
```java
@FeignClient(name = "user-service")
public interface UserFeignClient {
    
    @GetMapping("/user/{userId}")
    User getUser(@PathVariable("userId") Long userId);
}

@Service
public class OrderService {
    
    @Autowired
    private UserFeignClient userFeignClient;
    
    public void createOrder(Long userId) {
        User user = userFeignClient.getUser(userId);
        // 创建订单逻辑
    }
}
```

#### 2.2.3 健康检查机制 (⚠️ 难点)

Nacos支持两种健康检查模式：

**1. 客户端心跳模式（临时实例）**：
- 客户端每5秒向服务端发送一次心跳
- 服务端15秒未收到心跳，标记实例为不健康
- 服务端30秒未收到心跳，删除实例

**2. 服务端探测模式（持久化实例）**：
- 服务端主动探测实例健康状态
- 支持TCP、HTTP、MySQL等多种探测方式
- 实例下线后不会被删除，只标记为不健康

**配置持久化实例**：
```yaml
spring:
  cloud:
    nacos:
      discovery:
        ephemeral: false  # 设置为持久化实例
```

**健康检查配置**：
```java
@Configuration
public class NacosHealthCheckConfig {
    
    @Bean
    public Instance createInstance() {
        Instance instance = new Instance();
        instance.setIp("192.168.1.100");
        instance.setPort(8080);
        instance.setEphemeral(false);  // 持久化实例
        
        // 配置健康检查
        instance.setHealthy(true);
        Map<String, String> metadata = new HashMap<>();
        metadata.put("preserved.heart.beat.timeout", "15000");  // 心跳超时时间
        metadata.put("preserved.ip.delete.timeout", "30000");   // 实例删除超时时间
        instance.setMetadata(metadata);
        
        return instance;
    }
}
```

### 2.3 命名空间与多环境管理 🔥

#### 2.3.1 命名空间设计

**创建命名空间**：
1. 登录Nacos控制台
2. 进入"命名空间"菜单
3. 点击"新建命名空间"
4. 填写命名空间ID和名称

**命名空间配置**：
```yaml
spring:
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        namespace: dev-namespace-id  # 开发环境命名空间ID
        group: DEFAULT_GROUP
      discovery:
        server-addr: 127.0.0.1:8848
        namespace: dev-namespace-id
```

#### 2.3.2 多环境配置方案

**方案一：使用命名空间隔离**
```
dev-namespace    -> 开发环境所有服务
test-namespace   -> 测试环境所有服务
prod-namespace   -> 生产环境所有服务
```

**方案二：使用Group隔离**
```
DEFAULT_GROUP    -> 生产环境
DEV_GROUP        -> 开发环境
TEST_GROUP       -> 测试环境
```

**方案三：使用DataId隔离**
```
user-service-dev.yaml
user-service-test.yaml
user-service-prod.yaml
```

**推荐方案**：命名空间 + DataId
- 命名空间：区分大环境（开发、测试、生产）
- Group：区分业务模块或团队
- DataId：区分具体服务和配置文件

### 2.4 集群部署 (⚠️ 难点)

#### 2.4.1 集群架构

Nacos集群采用AP架构（可用性优先），支持水平扩展。

**集群模式架构**：
```
                    ┌─────────────┐
                    │   Nginx/SLB  │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │ Nacos-1 │       │ Nacos-2 │       │ Nacos-3 │
   └────┬────┘       └────┬────┘       └────┬────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    ┌──────▼──────┐
                    │    MySQL    │
                    └─────────────┘
```

**最小集群要求**：
- 节点数量：≥3（保证高可用）
- 数据库：MySQL 5.6.5+（生产环境必须使用外部数据库）
- 内存：每个节点≥2GB
- 网络：节点间网络互通

#### 2.4.2 集群配置步骤

**1. 配置数据库**

创建数据库并执行初始化脚本：
```sql
-- 创建数据库
CREATE DATABASE nacos_config CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 执行 nacos/conf/nacos-mysql.sql 脚本
```

**2. 修改application.properties**

```properties
# 数据库配置
spring.datasource.platform=mysql
db.num=1
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=nacos
db.password.0=nacos

# 集群配置
nacos.core.auth.enabled=true
nacos.core.auth.server.identity.key=nacos
nacos.core.auth.server.identity.value=nacos
```

**3. 配置cluster.conf**

在 `nacos/conf/cluster.conf` 中配置集群节点：
```
192.168.1.101:8848
192.168.1.102:8848
192.168.1.103:8848
```

**4. 启动集群**

在每个节点上执行：
```bash
sh startup.sh
```

**5. 配置负载均衡**

使用Nginx配置负载均衡：
```nginx
upstream nacos-cluster {
    server 192.168.1.101:8848;
    server 192.168.1.102:8848;
    server 192.168.1.103:8848;
}

server {
    listen 80;
    server_name nacos.example.com;
    
    location / {
        proxy_pass http://nacos-cluster;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

#### 2.4.3 集群数据一致性

Nacos使用Raft协议保证集群数据一致性：

**Raft协议特点**：
- Leader选举：集群中只有一个Leader节点
- 日志复制：Leader将变更同步到Follower
- 安全性：保证已提交的日志不会丢失

**数据同步流程**：
1. 客户端向Leader写入数据
2. Leader将数据写入本地日志
3. Leader将日志复制到Follower
4. 超过半数Follower确认后，Leader提交日志
5. Leader通知Follower提交日志

## 💻 实战应用

### 3.1 环境搭建

#### 3.1.1 单机模式安装

**下载Nacos**：
```bash
# 下载最新版本
wget https://github.com/alibaba/nacos/releases/download/2.3.0/nacos-server-2.3.0.tar.gz

# 解压
tar -xvf nacos-server-2.3.0.tar.gz
cd nacos/bin
```

**启动Nacos**：
```bash
# Linux/Mac
sh startup.sh -m standalone

# Windows
startup.cmd -m standalone
```

**访问控制台**：
- URL: http://localhost:8848/nacos
- 默认用户名/密码: nacos/nacos

#### 3.1.2 Docker部署

**单机模式**：
```bash
docker run -d \
  --name nacos-standalone \
  -e MODE=standalone \
  -p 8848:8848 \
  -p 9848:9848 \
  -p 9849:9849 \
  nacos/nacos-server:v2.3.0
```

**集群模式**：
```yaml
version: '3'
services:
  nacos1:
    image: nacos/nacos-server:v2.3.0
    container_name: nacos1
    environment:
      - PREFER_HOST_MODE=hostname
      - MODE=cluster
      - NACOS_SERVERS=nacos1:8848 nacos2:8848 nacos3:8848
      - MYSQL_SERVICE_HOST=mysql
      - MYSQL_SERVICE_DB_NAME=nacos_config
      - MYSQL_SERVICE_USER=nacos
      - MYSQL_SERVICE_PASSWORD=nacos
    ports:
      - "8848:8848"
      - "9848:9848"
    depends_on:
      - mysql
  
  nacos2:
    image: nacos/nacos-server:v2.3.0
    container_name: nacos2
    environment:
      - PREFER_HOST_MODE=hostname
      - MODE=cluster
      - NACOS_SERVERS=nacos1:8848 nacos2:8848 nacos3:8848
      - MYSQL_SERVICE_HOST=mysql
      - MYSQL_SERVICE_DB_NAME=nacos_config
      - MYSQL_SERVICE_USER=nacos
      - MYSQL_SERVICE_PASSWORD=nacos
    ports:
      - "8849:8848"
      - "9849:9848"
    depends_on:
      - mysql
  
  nacos3:
    image: nacos/nacos-server:v2.3.0
    container_name: nacos3
    environment:
      - PREFER_HOST_MODE=hostname
      - MODE=cluster
      - NACOS_SERVERS=nacos1:8848 nacos2:8848 nacos3:8848
      - MYSQL_SERVICE_HOST=mysql
      - MYSQL_SERVICE_DB_NAME=nacos_config
      - MYSQL_SERVICE_USER=nacos
      - MYSQL_SERVICE_PASSWORD=nacos
    ports:
      - "8850:8848"
      - "9850:9848"
    depends_on:
      - mysql
  
  mysql:
    image: mysql:8.0
    container_name: mysql
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=nacos_config
      - MYSQL_USER=nacos
      - MYSQL_PASSWORD=nacos
    volumes:
      - ./mysql:/var/lib/mysql
      - ./nacos-mysql.sql:/docker-entrypoint-initdb.d/nacos-mysql.sql
    ports:
      - "3306:3306"
```

### 3.2 Spring Cloud集成

#### 3.2.1 添加依赖

```xml
<dependencies>
    <!-- Spring Cloud Alibaba Nacos Config -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    </dependency>
    
    <!-- Spring Cloud Alibaba Nacos Discovery -->
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
    
    <!-- Spring Cloud LoadBalancer -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2022.0.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

#### 3.2.2 配置文件

**bootstrap.yml**（配置中心配置）：
```yaml
spring:
  application:
    name: user-service
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        namespace: dev
        group: DEFAULT_GROUP
        file-extension: yaml
        # 共享配置
        shared-configs:
          - data-id: common-mysql.yaml
            group: COMMON_GROUP
            refresh: true
          - data-id: common-redis.yaml
            group: COMMON_GROUP
            refresh: true
        # 扩展配置
        extension-configs:
          - data-id: user-service-ext.yaml
            group: DEFAULT_GROUP
            refresh: true
```

**application.yml**（服务发现配置）：
```yaml
spring:
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        namespace: dev
        group: DEFAULT_GROUP
        weight: 1
        cluster-name: DEFAULT
        metadata:
          version: 1.0.0
          region: cn-hangzhou

server:
  port: 8080

management:
  endpoints:
    web:
      exposure:
        include: '*'
```

#### 3.2.3 启动类配置

```java
package com.example.userservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * 用户服务启动类
 * 
 * @author erik.zhou
 */
@SpringBootApplication
@EnableDiscoveryClient  // 开启服务发现
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

### 3.3 配置管理实战

#### 3.3.1 配置发布

**在Nacos控制台发布配置**：
1. 登录Nacos控制台
2. 进入"配置管理" -> "配置列表"
3. 点击"+"新建配置
4. 填写配置信息：
   - Data ID: user-service-dev.yaml
   - Group: DEFAULT_GROUP
   - 配置格式: YAML
   - 配置内容:
```yaml
server:
  port: 8080

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/user_db
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver

user:
  name: zhangsan
  age: 18
```

#### 3.3.2 配置读取

```java
package com.example.userservice.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 配置测试控制器
 * 
 * @author erik.zhou
 */
@RestController
@RequestMapping("/config")
@RefreshScope  // 支持配置动态刷新
public class ConfigController {

    @Value("${user.name:default}")
    private String userName;

    @Value("${user.age:0}")
    private Integer userAge;

    @GetMapping("/get")
    public String getConfig() {
        return "userName: " + userName + ", userAge: " + userAge;
    }
}
```

#### 3.3.3 配置监听

```java
package com.example.userservice.config;

import com.alibaba.nacos.api.NacosFactory;
import com.alibaba.nacos.api.config.ConfigService;
import com.alibaba.nacos.api.config.listener.Listener;
import com.alibaba.nacos.api.exception.NacosException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;
import java.util.Properties;
import java.util.concurrent.Executor;

/**
 * Nacos配置监听器
 * 
 * @author erik.zhou
 */
@Configuration
public class NacosConfigListener {

    private static final Logger logger = LoggerFactory.getLogger(NacosConfigListener.class);

    @Value("${spring.cloud.nacos.config.server-addr}")
    private String serverAddr;

    @Value("${spring.cloud.nacos.config.namespace}")
    private String namespace;

    @PostConstruct
    public void init() throws NacosException {
        Properties properties = new Properties();
        properties.put("serverAddr", serverAddr);
        properties.put("namespace", namespace);

        ConfigService configService = NacosFactory.createConfigService(properties);

        String dataId = "user-service-dev.yaml";
        String group = "DEFAULT_GROUP";

        // 添加监听器
        configService.addListener(dataId, group, new Listener() {
            @Override
            public Executor getExecutor() {
                return null;
            }

            @Override
            public void receiveConfigInfo(String configInfo) {
                logger.info("配置发生变更，新配置内容：{}", configInfo);
                // 处理配置变更逻辑
            }
        });
    }
}
```

### 3.4 服务注册发现实战

#### 3.4.1 服务提供者

```java
package com.example.userservice.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 用户服务提供者
 * 
 * @author erik.zhou
 */
@RestController
@RequestMapping("/user")
public class UserController {

    @GetMapping("/{userId}")
    public String getUser(@PathVariable Long userId) {
        return "User ID: " + userId + ", Name: zhangsan";
    }

    @GetMapping("/info")
    public String getUserInfo() {
        return "User Service - Port: 8080";
    }
}
```

#### 3.4.2 服务消费者（RestTemplate）

```java
package com.example.orderservice.config;

import org.springframework.cloud.client.loadbalancer.LoadBalanced;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.client.RestTemplate;

/**
 * RestTemplate配置
 * 
 * @author erik.zhou
 */
@Configuration
public class RestTemplateConfig {

    @Bean
    @LoadBalanced  // 开启负载均衡
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

```java
package com.example.orderservice.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

/**
 * 订单服务
 * 
 * @author erik.zhou
 */
@Service
public class OrderService {

    @Autowired
    private RestTemplate restTemplate;

    public String createOrder(Long userId) {
        // 使用服务名调用
        String url = "http://user-service/user/" + userId;
        String userInfo = restTemplate.getForObject(url, String.class);
        return "Order created for: " + userInfo;
    }
}
```

#### 3.4.3 服务消费者（OpenFeign）

**添加依赖**：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

**启用Feign**：
```java
package com.example.orderservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;

/**
 * 订单服务启动类
 * 
 * @author erik.zhou
 */
@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients  // 开启Feign客户端
public class OrderServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}
```

**定义Feign客户端**：
```java
package com.example.orderservice.feign;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

/**
 * 用户服务Feign客户端
 * 
 * @author erik.zhou
 */
@FeignClient(name = "user-service")
public interface UserFeignClient {

    @GetMapping("/user/{userId}")
    String getUser(@PathVariable("userId") Long userId);

    @GetMapping("/user/info")
    String getUserInfo();
}
```

**使用Feign客户端**：
```java
package com.example.orderservice.service;

import com.example.orderservice.feign.UserFeignClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 订单服务（使用Feign）
 * 
 * @author erik.zhou
 */
@Service
public class OrderFeignService {

    @Autowired
    private UserFeignClient userFeignClient;

    public String createOrder(Long userId) {
        String userInfo = userFeignClient.getUser(userId);
        return "Order created for: " + userInfo;
    }
}
```

### 3.5 灰度发布实战

#### 3.5.1 基于元数据的灰度发布

**配置灰度实例**：
```yaml
spring:
  cloud:
    nacos:
      discovery:
        metadata:
          version: v2  # 灰度版本
          gray: true   # 灰度标记
```

**自定义负载均衡规则**：
```java
package com.example.orderservice.config;

import com.alibaba.cloud.nacos.ribbon.NacosServer;
import com.netflix.client.config.IClientConfig;
import com.netflix.loadbalancer.AbstractLoadBalancerRule;
import com.netflix.loadbalancer.Server;
import org.springframework.util.CollectionUtils;

import java.util.List;
import java.util.stream.Collectors;

/**
 * 灰度发布负载均衡规则
 * 
 * @author erik.zhou
 */
public class GrayLoadBalancerRule extends AbstractLoadBalancerRule {

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
        // 初始化配置
    }

    @Override
    public Server choose(Object key) {
        List<Server> reachableServers = getLoadBalancer().getReachableServers();
        
        // 获取请求头中的灰度标记
        String grayTag = getGrayTagFromRequest();
        
        if ("true".equals(grayTag)) {
            // 灰度流量，路由到灰度实例
            List<Server> grayServers = reachableServers.stream()
                .filter(server -> {
                    if (server instanceof NacosServer) {
                        NacosServer nacosServer = (NacosServer) server;
                        return "true".equals(nacosServer.getMetadata().get("gray"));
                    }
                    return false;
                })
                .collect(Collectors.toList());
            
            if (!CollectionUtils.isEmpty(grayServers)) {
                int index = (int) (Math.random() * grayServers.size());
                return grayServers.get(index);
            }
        }
        
        // 正常流量，路由到正式实例
        List<Server> normalServers = reachableServers.stream()
            .filter(server -> {
                if (server instanceof NacosServer) {
                    NacosServer nacosServer = (NacosServer) server;
                    return !"true".equals(nacosServer.getMetadata().get("gray"));
                }
                return true;
            })
            .collect(Collectors.toList());
        
        if (!CollectionUtils.isEmpty(normalServers)) {
            int index = (int) (Math.random() * normalServers.size());
            return normalServers.get(index);
        }
        
        return null;
    }
    
    private String getGrayTagFromRequest() {
        // 从请求头或其他地方获取灰度标记
        // 这里简化处理，实际应该从ThreadLocal或请求上下文获取
        return "false";
    }
}
```

## ✨ 最佳实践

### 4.1 配置管理最佳实践

#### 4.1.1 配置分层设计

**三层配置结构**：
```
1. 公共配置（COMMON_GROUP）
   - common-mysql.yaml      # 数据库配置
   - common-redis.yaml      # Redis配置
   - common-mq.yaml         # 消息队列配置

2. 应用配置（DEFAULT_GROUP）
   - user-service.yaml      # 用户服务基础配置
   - order-service.yaml     # 订单服务基础配置

3. 环境配置（按环境区分）
   - user-service-dev.yaml  # 开发环境
   - user-service-test.yaml # 测试环境
   - user-service-prod.yaml # 生产环境
```

**配置优先级**：
```
环境配置 > 应用配置 > 公共配置
```

#### 4.1.2 配置命名规范

**DataId命名规范**：
```
${prefix}-${spring.profiles.active}.${file-extension}

示例：
- user-service-dev.yaml
- order-service-prod.properties
- payment-service-test.yml
```

**Group命名规范**：
```
- DEFAULT_GROUP：默认分组
- COMMON_GROUP：公共配置分组
- ${业务模块}_GROUP：业务模块分组（如 USER_GROUP、ORDER_GROUP）
```

#### 4.1.3 敏感信息加密

**使用Jasypt加密**：

**添加依赖**：
```xml
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>3.0.5</version>
</dependency>
```

**配置加密密钥**：
```yaml
jasypt:
  encryptor:
    password: ${JASYPT_PASSWORD}  # 从环境变量获取
    algorithm: PBEWithMD5AndDES
```

**加密配置值**：
```java
import org.jasypt.encryption.StringEncryptor;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.CommandLineRunner;
import org.springframework.stereotype.Component;

/**
 * 配置加密工具
 * 
 * @author erik.zhou
 */
@Component
public class ConfigEncryptor implements CommandLineRunner {

    @Autowired
    private StringEncryptor stringEncryptor;

    @Override
    public void run(String... args) {
        String password = "root123456";
        String encryptedPassword = stringEncryptor.encrypt(password);
        System.out.println("加密后的密码：" + encryptedPassword);
    }
}
```

**使用加密配置**：
```yaml
spring:
  datasource:
    password: ENC(加密后的密码)
```

### 4.2 服务注册发现最佳实践

#### 4.2.1 健康检查配置

```yaml
spring:
  cloud:
    nacos:
      discovery:
        # 心跳间隔（毫秒）
        heart-beat-interval: 5000
        # 心跳超时时间（毫秒）
        heart-beat-timeout: 15000
        # IP删除超时时间（毫秒）
        ip-delete-timeout: 30000
```

#### 4.2.2 服务权重配置

**动态调整权重**：
```java
package com.example.userservice.controller;

import com.alibaba.cloud.nacos.NacosDiscoveryProperties;
import com.alibaba.nacos.api.naming.NamingService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

/**
 * 服务权重管理
 * 
 * @author erik.zhou
 */
@RestController
public class WeightController {

    @Autowired
    private NacosDiscoveryProperties nacosDiscoveryProperties;

    @Autowired
    private NamingService namingService;

    @PostMapping("/weight/update")
    public String updateWeight(@RequestParam Double weight) throws Exception {
        String serviceName = nacosDiscoveryProperties.getService();
        String ip = nacosDiscoveryProperties.getIp();
        int port = nacosDiscoveryProperties.getPort();
        
        namingService.registerInstance(serviceName, ip, port, weight);
        return "权重更新成功：" + weight;
    }
}
```

#### 4.2.3 服务分组隔离

```yaml
spring:
  cloud:
    nacos:
      discovery:
        group: ${spring.profiles.active}_GROUP  # 按环境分组
        cluster-name: ${region}  # 按地域分组
```

### 4.3 性能优化

#### 4.3.1 客户端缓存

Nacos客户端会缓存服务列表，减少对服务端的请求：

```yaml
spring:
  cloud:
    nacos:
      discovery:
        # 缓存加载超时时间（毫秒）
        cache-load-timeout: 10000
```

#### 4.3.2 批量注册

```java
package com.example.config;

import com.alibaba.nacos.api.naming.NamingService;
import com.alibaba.nacos.api.naming.pojo.Instance;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.ArrayList;
import java.util.List;

/**
 * 批量服务注册
 * 
 * @author erik.zhou
 */
@Component
public class BatchServiceRegistry {

    @Autowired
    private NamingService namingService;

    public void batchRegister() throws Exception {
        List<Instance> instances = new ArrayList<>();
        
        for (int i = 0; i < 10; i++) {
            Instance instance = new Instance();
            instance.setIp("192.168.1." + (100 + i));
            instance.setPort(8080 + i);
            instance.setWeight(1.0);
            instances.add(instance);
        }
        
        // 批量注册（Nacos 2.x支持）
        namingService.batchRegisterInstance("user-service", "DEFAULT_GROUP", instances);
    }
}
```

#### 4.3.3 连接池优化

```yaml
spring:
  cloud:
    nacos:
      config:
        # 长轮询超时时间（毫秒）
        timeout: 30000
        # 最大重试次数
        max-retry: 3
      discovery:
        # 命名服务线程池大小
        naming-load-cache-at-start: true
```

### 4.4 安全加固

#### 4.4.1 开启鉴权

**修改application.properties**：
```properties
# 开启鉴权
nacos.core.auth.enabled=true
nacos.core.auth.system.type=nacos

# 默认token密钥（生产环境必须修改）
nacos.core.auth.plugin.nacos.token.secret.key=SecretKey012345678901234567890123456789012345678901234567890123456789

# 默认token过期时间（秒）
nacos.core.auth.plugin.nacos.token.expire.seconds=18000

# 服务端身份识别key和value（集群间通信使用）
nacos.core.auth.server.identity.key=serverIdentity
nacos.core.auth.server.identity.value=security
```

**客户端配置**：
```yaml
spring:
  cloud:
    nacos:
      config:
        server-addr: 127.0.0.1:8848
        username: nacos
        password: nacos
      discovery:
        server-addr: 127.0.0.1:8848
        username: nacos
        password: nacos
```

#### 4.4.2 配置访问控制

在Nacos控制台配置权限：
1. 创建用户
2. 创建角色
3. 分配权限（读、写、删除）
4. 绑定用户和角色

### 4.5 常见陷阱

#### ⚠️ 陷阱1：配置不生效

**原因**：
- bootstrap.yml配置错误
- DataId或Group配置错误
- 命名空间配置错误
- 未添加@RefreshScope注解

**解决方案**：
1. 检查bootstrap.yml是否正确加载
2. 确认DataId、Group、Namespace配置一致
3. 在需要动态刷新的Bean上添加@RefreshScope

#### ⚠️ 陷阱2：服务注册失败

**原因**：
- Nacos服务端未启动
- 网络不通
- 命名空间不存在
- 鉴权失败

**解决方案**：
1. 检查Nacos服务端状态
2. 测试网络连通性
3. 确认命名空间已创建
4. 检查用户名密码是否正确

#### ⚠️ 陷阱3：配置监听失效

**原因**：
- 长轮询超时
- 网络抖动
- 客户端缓存未更新

**解决方案**：
1. 增加长轮询超时时间
2. 配置重试机制
3. 手动刷新配置：POST /actuator/refresh

#### ⚠️ 陷阱4：集群脑裂

**原因**：
- 网络分区
- 节点时钟不同步
- 数据库连接异常

**解决方案**：
1. 使用奇数个节点（3、5、7）
2. 配置NTP时间同步
3. 使用高可用数据库
4. 配置合理的选举超时时间

## ❓ 常见问题

### Q1: Nacos和Eureka有什么区别？

**A**: 主要区别：

| 特性 | Nacos | Eureka |
|------|-------|--------|
| CAP理论 | AP + CP（可切换） | AP |
| 健康检查 | TCP/HTTP/MySQL | 心跳 |
| 负载均衡 | 权重/元数据 | Ribbon |
| 配置中心 | 支持 | 不支持 |
| 控制台 | 功能丰富 | 基础功能 |
| 社区活跃度 | 活跃 | 已停止维护 |

### Q2: 如何实现配置的灰度发布？

**A**: 三种方案：

1. **基于命名空间**：创建灰度命名空间，部分实例使用灰度配置
2. **基于Group**：创建灰度Group，通过配置切换
3. **基于DataId**：创建灰度DataId，通过配置文件切换

推荐使用方案1，隔离性最好。

### Q3: Nacos集群最少需要几个节点？

**A**: 最少3个节点。

**原因**：
- Nacos使用Raft协议保证数据一致性
- Raft要求超过半数节点存活才能正常工作
- 3个节点可以容忍1个节点故障
- 5个节点可以容忍2个节点故障

**推荐配置**：
- 开发/测试环境：1个节点（单机模式）
- 生产环境：3个或5个节点

### Q4: 如何监控Nacos的运行状态？

**A**: 多种监控方式：

**1. 控制台监控**：
- 访问 http://nacos-server:8848/nacos
- 查看服务列表、配置列表、集群状态

**2. Metrics接口**：
```bash
# 查看Nacos指标
curl http://nacos-server:8848/nacos/actuator/prometheus
```

**3. 集成Prometheus + Grafana**：
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'nacos'
    static_configs:
      - targets: ['nacos-server:8848']
    metrics_path: '/nacos/actuator/prometheus'
```

**4. 日志监控**：
- 查看 logs/nacos.log
- 关注ERROR和WARN级别日志

### Q5: 配置中心的配置如何备份和恢复？

**A**: 三种备份方式：

**1. 数据库备份**（推荐）：
```bash
# 备份MySQL数据库
mysqldump -u nacos -p nacos_config > nacos_backup.sql

# 恢复
mysql -u nacos -p nacos_config < nacos_backup.sql
```

**2. 使用Nacos API导出**：
```bash
# 导出所有配置
curl -X GET "http://nacos-server:8848/nacos/v1/cs/configs?export=true&group=*&tenant=*" > configs.zip
```

**3. 使用Nacos控制台**：
- 进入"配置管理" -> "配置列表"
- 点击"导出配置"
- 选择需要导出的配置

### Q6: 如何解决Nacos内存占用过高的问题？

**A**: 优化方案：

**1. 调整JVM参数**：
```bash
# 修改 bin/startup.sh
JAVA_OPT="${JAVA_OPT} -Xms512m -Xmx512m -Xmn256m"
JAVA_OPT="${JAVA_OPT} -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
```

**2. 清理历史配置**：
- 删除不再使用的配置
- 清理配置历史记录

**3. 优化客户端缓存**：
```yaml
spring:
  cloud:
    nacos:
      discovery:
        cache-load-timeout: 5000  # 减少缓存加载时间
```

**4. 使用持久化实例**：
```yaml
spring:
  cloud:
    nacos:
      discovery:
        ephemeral: false  # 减少心跳开销
```

### Q7: Nacos 1.x和2.x有什么区别？

**A**: 主要变化：

**架构升级**：
- 1.x：HTTP短连接
- 2.x：gRPC长连接（性能提升10倍）

**新增功能**：
- 批量注册API
- 配置加密传输
- 更强的鉴权能力

**兼容性**：
- 2.x完全兼容1.x客户端
- 建议升级到2.x以获得更好性能

## 🔗 相关资源

### 官方资源
- **官方网站**: https://nacos.io/
- **GitHub仓库**: https://github.com/alibaba/nacos
- **官方文档**: https://nacos.io/zh-cn/docs/what-is-nacos.html
- **版本发布**: https://github.com/alibaba/nacos/releases

### 学习资源
- **Spring Cloud Alibaba文档**: https://spring-cloud-alibaba-group.github.io/
- **Nacos架构设计**: https://nacos.io/zh-cn/docs/architecture.html
- **Nacos最佳实践**: https://nacos.io/zh-cn/docs/best-practice.html

### 社区资源
- **Nacos社区**: https://nacos.io/zh-cn/community/
- **问题反馈**: https://github.com/alibaba/nacos/issues
- **钉钉交流群**: 23146118

## 📝 学习检查清单

- [ ] 理解Nacos的核心概念（命名空间、分组、DataId）
- [ ] 掌握Nacos作为配置中心的使用
- [ ] 掌握Nacos作为服务注册中心的使用
- [ ] 理解配置动态刷新机制
- [ ] 理解服务健康检查机制
- [ ] 掌握Nacos集群部署方案
- [ ] 掌握Nacos在Spring Cloud中的集成
- [ ] 了解Nacos的性能优化方法
- [ ] 了解Nacos的安全加固措施
- [ ] 能够排查Nacos常见问题

---

**学习建议**：
1. 先在单机模式下熟悉Nacos的基本功能
2. 实践配置管理和服务注册发现
3. 搭建Nacos集群，理解高可用架构
4. 在实际项目中应用Nacos
5. 关注Nacos社区，了解最新特性

**下一步学习**：
- [Apollo配置中心](Apollo-完整教程.md)
- [Spring Cloud Config](Spring-Cloud-Config-完整教程.md)
- [Spring Cloud Gateway](../02-服务网关/Spring-Cloud-Gateway-完整教程.md)
