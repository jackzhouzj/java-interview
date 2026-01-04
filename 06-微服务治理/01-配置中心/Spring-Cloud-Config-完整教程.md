# Spring Cloud Config 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)

## 📚 技术概述
- **版本**: 4.1.0
- **官方文档**: https://spring.io/projects/spring-cloud-config
- **GitHub**: https://github.com/spring-cloud/spring-cloud-config
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: Spring Boot、Git基础、微服务架构基础
- **文档来源**: 官方文档 + 实战经验
- **更新时间**: 2024-12-31
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Spring Cloud Config的核心概念和架构设计
- [ ] 掌握Config Server的搭建和配置
- [ ] 掌握Config Client的集成和使用
- [ ] 理解配置的加密解密机制
- [ ] 掌握配置的动态刷新方法
- [ ] 了解Config Server的高可用部署
- [ ] 掌握多环境配置管理方案
- [ ] 了解Config与其他配置中心的对比

## 📖 基础概念

### 1.1 什么是Spring Cloud Config

Spring Cloud Config是Spring Cloud生态中的配置管理工具,为分布式系统中的外部化配置提供服务器端和客户端支持。

**核心功能**:
- **集中管理配置**: 将所有微服务的配置文件集中存储在Git仓库
- **环境隔离**: 支持多环境配置(dev、test、prod)
- **配置加密**: 支持敏感信息加密存储
- **动态刷新**: 支持配置的动态更新,无需重启服务
- **版本管理**: 基于Git的版本控制和回滚

### 1.2 核心组件

#### Config Server (配置服务器)
提供配置文件的HTTP API接口,从Git仓库读取配置并提供给客户端。

**主要职责**:
- 从Git仓库拉取配置文件
- 提供REST API供客户端访问
- 支持配置文件的加密解密
- 支持多种存储后端(Git、SVN、本地文件系统)

#### Config Client (配置客户端)
微服务应用,从Config Server获取配置信息。

**主要职责**:
- 启动时从Config Server获取配置
- 支持配置的动态刷新
- 支持配置的本地缓存

### 1.3 工作原理

```
┌─────────────┐
│   Git Repo  │  (配置文件存储)
└──────┬──────┘
       │
       │ pull
       ▼
┌─────────────┐
│Config Server│  (配置服务器)
└──────┬──────┘
       │
       │ HTTP API
       ▼
┌─────────────┐
│Config Client│  (微服务应用)
└─────────────┘
```

**配置获取流程**:
1. Config Server启动时从Git仓库克隆配置文件
2. Config Client启动时向Config Server请求配置
3. Config Server返回对应环境的配置信息
4. Config Client使用获取的配置启动应用

### 1.4 配置文件命名规则

**标准命名格式**:
```
{application}-{profile}.yml
{application}-{profile}.properties
```

**示例**:
```
application.yml          # 所有应用共享的配置
user-service.yml         # user-service的默认配置
user-service-dev.yml     # user-service的开发环境配置
user-service-test.yml    # user-service的测试环境配置
user-service-prod.yml    # user-service的生产环境配置
```

**配置优先级** (从高到低):
```
{application}-{profile}.yml
{application}.yml
application-{profile}.yml
application.yml
```

## 🔥 核心特性 (重点)

### 2.1 多种存储后端支持 🔥

#### 2.1.1 Git存储 (推荐)

**优势**:
- 版本控制,支持回滚
- 支持分支管理
- 团队协作方便
- 审计追踪完整

**配置示例**:
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo.git
          # 搜索路径,支持多个目录
          search-paths: '{application}'
          # 默认分支
          default-label: main
          # 克隆超时时间(秒)
          timeout: 10
          # 强制拉取
          force-pull: true
```

**私有仓库配置**:
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo.git
          username: your-username
          password: your-password
          # 或使用SSH密钥
          # uri: git@github.com:your-org/config-repo.git
          # ignore-local-ssh-settings: false
```

#### 2.1.2 本地文件系统

**适用场景**: 开发测试环境

**配置示例**:
```yaml
spring:
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/configs,file:/opt/configs
```

#### 2.1.3 SVN存储

**配置示例**:
```yaml
spring:
  cloud:
    config:
      server:
        svn:
          uri: https://svn.example.com/config-repo
          username: your-username
          password: your-password
```

### 2.2 配置加密解密 🔥 (⚠️ 难点)

#### 2.2.1 对称加密

**安装JCE**:
从Oracle官网下载JCE无限制权限策略文件,替换JRE的security目录下的文件。

**配置加密密钥**:
```yaml
# bootstrap.yml
encrypt:
  key: my-secret-key-123456  # 对称加密密钥
```

**加密配置值**:
```bash
# 使用Config Server的加密端点
curl http://localhost:8888/encrypt -d "mysecretpassword"
# 返回: AQA3eHg7...加密后的字符串
```

**在配置文件中使用加密值**:
```yaml
spring:
  datasource:
    password: '{cipher}AQA3eHg7...加密后的字符串'
```

**注意事项**:
- 加密值必须以 `{cipher}` 前缀开头
- Config Server会自动解密后返回给客户端
- 密钥不要提交到Git仓库

#### 2.2.2 非对称加密 (推荐生产环境)

**生成密钥对**:
```bash
# 生成keystore
keytool -genkeypair -alias config-server-key \
  -keyalg RSA -keysize 2048 \
  -keystore config-server.jks \
  -storepass mypassword \
  -keypass mypassword
```

**配置密钥库**:
```yaml
encrypt:
  key-store:
    location: classpath:/config-server.jks
    password: mypassword
    alias: config-server-key
    secret: mypassword
```

**优势**:
- 更安全,私钥不需要在客户端存储
- 支持密钥轮换
- 符合安全合规要求

### 2.3 配置动态刷新 🔥

#### 2.3.1 手动刷新

**添加依赖**:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

**开启refresh端点**:
```yaml
management:
  endpoints:
    web:
      exposure:
        include: refresh
```

**使用@RefreshScope注解**:
```java
package com.example.userservice.controller;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * 配置刷新测试控制器
 * 
 * @author erik.zhou
 */
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

**触发刷新**:
```bash
curl -X POST http://localhost:8080/actuator/refresh
```

#### 2.3.2 自动刷新 (Spring Cloud Bus)

**架构图**:
```
Git Repo → Config Server → RabbitMQ/Kafka → Config Clients
                ↓              ↓
            Webhook      Bus Refresh
```

**添加依赖**:
```xml
<!-- Config Server端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>

<!-- Config Client端 -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

**配置RabbitMQ**:
```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

management:
  endpoints:
    web:
      exposure:
        include: bus-refresh
```

**配置Git Webhook**:
在Git仓库设置Webhook,当配置文件变更时自动触发:
```
POST http://config-server:8888/actuator/bus-refresh
```

**工作流程**:
1. 开发人员提交配置变更到Git
2. Git触发Webhook调用Config Server的/bus-refresh端点
3. Config Server通过消息总线通知所有客户端
4. 客户端收到消息后自动刷新配置

### 2.4 多环境配置管理 🔥

#### 2.4.1 使用Profile区分环境

**Git仓库结构**:
```
config-repo/
├── application.yml              # 公共配置
├── application-dev.yml          # 开发环境公共配置
├── application-test.yml         # 测试环境公共配置
├── application-prod.yml         # 生产环境公共配置
├── user-service.yml             # 用户服务默认配置
├── user-service-dev.yml         # 用户服务开发环境配置
├── user-service-test.yml        # 用户服务测试环境配置
└── user-service-prod.yml        # 用户服务生产环境配置
```

**客户端配置**:
```yaml
spring:
  application:
    name: user-service
  profiles:
    active: dev  # 指定环境
  cloud:
    config:
      uri: http://localhost:8888
      profile: ${spring.profiles.active}
```

#### 2.4.2 使用Label区分版本

**Git分支结构**:
```
main        → 生产环境配置
develop     → 开发环境配置
release/*   → 预发布环境配置
```

**客户端配置**:
```yaml
spring:
  cloud:
    config:
      uri: http://localhost:8888
      label: main  # 指定Git分支
```

**动态切换版本**:
```bash
# 获取main分支的配置
curl http://localhost:8888/user-service/dev/main

# 获取develop分支的配置
curl http://localhost:8888/user-service/dev/develop
```

## 💻 实战应用

### 3.1 搭建Config Server

#### 3.1.1 创建Config Server项目

**添加依赖**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
    </parent>

    <groupId>com.example</groupId>
    <artifactId>config-server</artifactId>
    <version>1.0.0</version>

    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2023.0.0</spring-cloud.version>
    </properties>

    <dependencies>
        <!-- Config Server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>

        <!-- Actuator -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- Security (可选) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

**启动类**:
```java
package com.example.configserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

/**
 * Config Server启动类
 * 
 * @author erik.zhou
 */
@SpringBootApplication
@EnableConfigServer  // 开启Config Server
public class ConfigServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

**配置文件 (application.yml)**:
```yaml
server:
  port: 8888

spring:
  application:
    name: config-server
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo.git
          # 搜索路径,支持占位符
          search-paths: '{application}'
          # 默认分支
          default-label: main
          # 克隆超时时间(秒)
          timeout: 10
          # 强制拉取
          force-pull: true
          # 私有仓库认证
          username: ${GIT_USERNAME}
          password: ${GIT_PASSWORD}

# 加密配置
encrypt:
  key: ${ENCRYPT_KEY}

# Actuator配置
management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always

# 日志配置
logging:
  level:
    org.springframework.cloud.config: DEBUG
```

#### 3.1.2 配置安全认证

**Security配置类**:
```java
package com.example.configserver.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.core.userdetails.User;
import org.springframework.security.core.userdetails.UserDetailsService;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;
import org.springframework.security.provisioning.InMemoryUserDetailsManager;
import org.springframework.security.web.SecurityFilterChain;

/**
 * Security配置
 * 
 * @author erik.zhou
 */
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().authenticated()
            )
            .httpBasic();
        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        InMemoryUserDetailsManager manager = new InMemoryUserDetailsManager();
        manager.createUser(User.withUsername("config-user")
            .password(passwordEncoder().encode("config-password"))
            .roles("USER")
            .build());
        return manager;
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 3.2 搭建Config Client

#### 3.2.1 创建Config Client项目

**添加依赖**:
```xml
<dependencies>
    <!-- Config Client -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>

    <!-- Bootstrap -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-bootstrap</artifactId>
    </dependency>

    <!-- Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Actuator -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

**bootstrap.yml配置**:
```yaml
spring:
  application:
    name: user-service
  profiles:
    active: dev
  cloud:
    config:
      # Config Server地址
      uri: http://localhost:8888
      # 环境
      profile: ${spring.profiles.active}
      # Git分支
      label: main
      # 认证信息
      username: config-user
      password: config-password
      # 快速失败
      fail-fast: true
      # 重试配置
      retry:
        initial-interval: 1000
        max-attempts: 6
        max-interval: 2000
        multiplier: 1.1
```

**application.yml配置**:
```yaml
server:
  port: 8080

management:
  endpoints:
    web:
      exposure:
        include: refresh,health,info

logging:
  level:
    root: INFO
    com.example: DEBUG
```

#### 3.2.2 启动类

```java
package com.example.userservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * 用户服务启动类
 * 
 * @author erik.zhou
 */
@SpringBootApplication
public class UserServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}
```

#### 3.2.3 配置读取示例

**使用@Value注解**:
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
@RefreshScope
public class ConfigController {

    @Value("${user.name:default}")
    private String userName;

    @Value("${user.age:0}")
    private Integer userAge;

    @Value("${spring.datasource.url:}")
    private String datasourceUrl;

    @GetMapping("/get")
    public String getConfig() {
        return String.format("userName: %s, userAge: %d, datasourceUrl: %s", 
            userName, userAge, datasourceUrl);
    }
}
```

**使用@ConfigurationProperties**:
```java
package com.example.userservice.config;

import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.cloud.context.config.annotation.RefreshScope;
import org.springframework.stereotype.Component;

/**
 * 用户配置属性
 * 
 * @author erik.zhou
 */
@Component
@ConfigurationProperties(prefix = "user")
@RefreshScope
public class UserProperties {

    private String name;
    private Integer age;
    private String email;

    // Getter和Setter方法
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}
```

**使用配置属性**:
```java
package com.example.userservice.service;

import com.example.userservice.config.UserProperties;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 用户服务
 * 
 * @author erik.zhou
 */
@Service
public class UserService {

    @Autowired
    private UserProperties userProperties;

    public String getUserInfo() {
        return String.format("Name: %s, Age: %d, Email: %s",
            userProperties.getName(),
            userProperties.getAge(),
            userProperties.getEmail());
    }
}
```

### 3.3 准备配置仓库

#### 3.3.1 创建Git仓库

**仓库结构**:
```
config-repo/
├── application.yml
├── application-dev.yml
├── application-test.yml
├── application-prod.yml
├── user-service.yml
├── user-service-dev.yml
├── user-service-test.yml
├── user-service-prod.yml
├── order-service.yml
├── order-service-dev.yml
├── order-service-test.yml
└── order-service-prod.yml
```

#### 3.3.2 配置文件示例

**application.yml (公共配置)**:
```yaml
# 公共配置
management:
  endpoints:
    web:
      exposure:
        include: '*'

logging:
  level:
    root: INFO

spring:
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: GMT+8
```

**application-dev.yml (开发环境公共配置)**:
```yaml
logging:
  level:
    root: DEBUG
    com.example: DEBUG
```

**user-service.yml (用户服务默认配置)**:
```yaml
user:
  name: default-user
  age: 0
  email: default@example.com

server:
  port: 8080
```

**user-service-dev.yml (用户服务开发环境配置)**:
```yaml
user:
  name: dev-user
  age: 18
  email: dev@example.com

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/user_dev?useUnicode=true&characterEncoding=utf8
    username: root
    password: '{cipher}AQA3eHg7...加密后的密码'
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5

  redis:
    host: localhost
    port: 6379
    database: 0
```

**user-service-prod.yml (用户服务生产环境配置)**:
```yaml
user:
  name: prod-user
  age: 25
  email: prod@example.com

spring:
  datasource:
    url: jdbc:mysql://prod-db:3306/user_prod?useUnicode=true&characterEncoding=utf8
    username: prod_user
    password: '{cipher}AQBXX...生产环境加密密码'
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 50
      minimum-idle: 10

  redis:
    host: prod-redis
    port: 6379
    database: 0
    password: '{cipher}AQCYY...Redis密码'

server:
  port: 8080
```

### 3.4 Config Server API使用

#### 3.4.1 HTTP端点说明

**获取配置的URL格式**:
```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

**示例**:
```bash
# 获取user-service的dev环境配置(main分支)
curl http://localhost:8888/user-service/dev/main

# 获取user-service的dev环境配置(YAML格式)
curl http://localhost:8888/user-service-dev.yml

# 获取user-service的prod环境配置(Properties格式)
curl http://localhost:8888/user-service-prod.properties
```

#### 3.4.2 加密解密端点

**加密**:
```bash
curl http://localhost:8888/encrypt -d "mysecretpassword"
```

**解密**:
```bash
curl http://localhost:8888/decrypt -d "AQA3eHg7...加密后的字符串"
```

### 3.5 配置刷新实战

#### 3.5.1 手动刷新单个服务

**步骤**:
1. 修改Git仓库中的配置文件
2. 提交并推送到远程仓库
3. 调用服务的refresh端点

```bash
# 刷新配置
curl -X POST http://localhost:8080/actuator/refresh
```

#### 3.5.2 使用Spring Cloud Bus批量刷新

**Config Server配置**:
```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest

management:
  endpoints:
    web:
      exposure:
        include: bus-refresh
```

**触发批量刷新**:
```bash
# 刷新所有服务
curl -X POST http://localhost:8888/actuator/bus-refresh

# 刷新指定服务
curl -X POST http://localhost:8888/actuator/bus-refresh/user-service:8080
```

**配置Git Webhook**:
在GitHub/GitLab中配置Webhook:
- URL: `http://config-server:8888/actuator/bus-refresh`
- Content type: `application/json`
- Events: `Push events`

## ✨ 最佳实践

### 4.1 配置管理最佳实践

#### 4.1.1 配置分层设计

**三层配置结构**:
```
1. 公共配置 (application.yml)
   - 所有服务共享的配置
   - 日志、监控、通用组件配置

2. 应用配置 ({application}.yml)
   - 特定服务的默认配置
   - 服务端口、应用名称

3. 环境配置 ({application}-{profile}.yml)
   - 特定环境的配置
   - 数据库、Redis、MQ等环境相关配置
```

**配置优先级** (从高到低):
```
{application}-{profile}.yml
{application}.yml
application-{profile}.yml
application.yml
```

#### 4.1.2 敏感信息管理

**必须加密的配置**:
- 数据库密码
- Redis密码
- 消息队列密码
- 第三方API密钥
- JWT密钥

**加密流程**:
```bash
# 1. 加密敏感信息
curl http://localhost:8888/encrypt -d "prod_db_password"
# 返回: AQA3eHg7...

# 2. 在配置文件中使用
spring:
  datasource:
    password: '{cipher}AQA3eHg7...'

# 3. Config Server自动解密后返回给客户端
```

**环境变量管理密钥**:
```bash
# 不要在配置文件中硬编码密钥
export ENCRYPT_KEY=my-secret-key-123456

# 在application.yml中引用
encrypt:
  key: ${ENCRYPT_KEY}
```

#### 4.1.3 配置文件命名规范

**命名规则**:
```
{application}-{profile}.{extension}

示例:
- user-service-dev.yml
- order-service-prod.properties
- payment-service-test.yml
```

**目录结构规范**:
```
config-repo/
├── common/                    # 公共配置
│   ├── application.yml
│   ├── application-dev.yml
│   └── application-prod.yml
├── user-service/              # 用户服务配置
│   ├── user-service.yml
│   ├── user-service-dev.yml
│   └── user-service-prod.yml
└── order-service/             # 订单服务配置
    ├── order-service.yml
    ├── order-service-dev.yml
    └── order-service-prod.yml
```

**Config Server配置多路径**:
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo.git
          search-paths: common,{application}
```

### 4.2 高可用部署

#### 4.2.1 Config Server集群部署

**架构图**:
```
                    ┌─────────────┐
                    │   Nginx/SLB  │
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │Config-1 │       │Config-2 │       │Config-3 │
   └────┬────┘       └────┬────┘       └────┬────┘
        │                  │                  │
        └──────────────────┼──────────────────┘
                           │
                    ┌──────▼──────┐
                    │   Git Repo   │
                    └─────────────┘
```

**Nginx配置**:
```nginx
upstream config-server {
    server 192.168.1.101:8888;
    server 192.168.1.102:8888;
    server 192.168.1.103:8888;
}

server {
    listen 80;
    server_name config.example.com;
    
    location / {
        proxy_pass http://config-server;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

**客户端配置**:
```yaml
spring:
  cloud:
    config:
      # 配置多个Config Server地址
      uri: http://config1:8888,http://config2:8888,http://config3:8888
      fail-fast: true
      retry:
        max-attempts: 6
```

#### 4.2.2 配置本地缓存

**开启本地缓存**:
```yaml
spring:
  cloud:
    config:
      # 允许使用本地缓存
      allow-override: true
      override-none: true
      override-system-properties: false
```

**本地缓存位置**:
```
~/.config-repo/{application}-{profile}.properties
```

**缓存策略**:
- Config Server不可用时使用本地缓存
- 定期从Config Server更新缓存
- 缓存文件加密存储

### 4.3 性能优化

#### 4.3.1 Git仓库优化

**使用浅克隆**:
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo.git
          # 浅克隆,只克隆最新版本
          clone-on-start: true
          # 删除未跟踪的分支
          delete-untracked-branches: true
```

**配置本地缓存**:
```yaml
spring:
  cloud:
    config:
      server:
        git:
          # 本地缓存目录
          basedir: /tmp/config-repo
          # 强制拉取
          force-pull: true
```

#### 4.3.2 客户端启动优化

**快速失败配置**:
```yaml
spring:
  cloud:
    config:
      fail-fast: true  # 快速失败
      retry:
        initial-interval: 1000
        max-attempts: 3  # 减少重试次数
        max-interval: 2000
```

**异步加载配置**:
```java
package com.example.userservice.config;

import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.cloud.context.refresh.ContextRefresher;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

/**
 * 异步配置加载
 * 
 * @author erik.zhou
 */
@Component
public class AsyncConfigLoader {

    private final ContextRefresher contextRefresher;

    public AsyncConfigLoader(ContextRefresher contextRefresher) {
        this.contextRefresher = contextRefresher;
    }

    @EventListener(ApplicationReadyEvent.class)
    public void onApplicationReady() {
        // 应用启动后异步刷新配置
        new Thread(() -> {
            try {
                Thread.sleep(5000);
                contextRefresher.refresh();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();
    }
}
```

### 4.4 安全加固

#### 4.4.1 Config Server安全配置

**启用HTTPS**:
```yaml
server:
  port: 8443
  ssl:
    enabled: true
    key-store: classpath:keystore.jks
    key-store-password: changeit
    key-store-type: JKS
    key-alias: config-server
```

**IP白名单**:
```java
package com.example.configserver.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.web.SecurityFilterChain;

/**
 * IP白名单配置
 * 
 * @author erik.zhou
 */
@Configuration
public class IpWhitelistConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health").permitAll()
                .anyRequest().access((authentication, context) -> {
                    String remoteAddr = context.getRequest().getRemoteAddr();
                    // 只允许内网IP访问
                    return remoteAddr.startsWith("192.168.") || 
                           remoteAddr.startsWith("10.") ||
                           remoteAddr.equals("127.0.0.1");
                })
            );
        return http.build();
    }
}
```

#### 4.4.2 Git仓库权限控制

**使用私有仓库**:
- 配置文件必须存储在私有Git仓库
- 使用SSH密钥或Personal Access Token认证
- 定期轮换访问凭证

**分支保护**:
- 主分支(main/master)设置保护
- 配置变更需要Code Review
- 启用提交签名验证

### 4.5 监控告警

#### 4.5.1 健康检查

**自定义健康检查**:
```java
package com.example.configserver.health;

import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

/**
 * Git仓库健康检查
 * 
 * @author erik.zhou
 */
@Component
public class GitHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        try {
            // 检查Git仓库连接
            // 检查最后更新时间
            return Health.up()
                .withDetail("repository", "connected")
                .withDetail("lastUpdate", System.currentTimeMillis())
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```

#### 4.5.2 配置变更审计

**记录配置访问日志**:
```java
package com.example.configserver.audit;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.actuate.audit.AuditEvent;
import org.springframework.boot.actuate.audit.listener.AuditApplicationEvent;
import org.springframework.context.event.EventListener;
import org.springframework.stereotype.Component;

/**
 * 配置访问审计
 * 
 * @author erik.zhou
 */
@Component
public class ConfigAuditListener {

    private static final Logger logger = LoggerFactory.getLogger(ConfigAuditListener.class);

    @EventListener
    public void onAuditEvent(AuditApplicationEvent event) {
        AuditEvent auditEvent = event.getAuditEvent();
        logger.info("配置访问审计: principal={}, type={}, data={}",
            auditEvent.getPrincipal(),
            auditEvent.getType(),
            auditEvent.getData());
    }
}
```

### 4.6 常见陷阱

#### ⚠️ 陷阱1: bootstrap.yml未生效

**原因**:
- Spring Boot 2.4+默认不加载bootstrap.yml
- 缺少spring-cloud-starter-bootstrap依赖

**解决方案**:
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bootstrap</artifactId>
</dependency>
```

或使用spring.config.import:
```yaml
# application.yml
spring:
  config:
    import: optional:configserver:http://localhost:8888
```

#### ⚠️ 陷阱2: 配置刷新不生效

**原因**:
- 未添加@RefreshScope注解
- 使用了构造器注入
- 配置类是单例且未标记@RefreshScope

**解决方案**:
```java
// 错误示例
@Service
public class UserService {
    private final String userName;
    
    // 构造器注入无法刷新
    public UserService(@Value("${user.name}") String userName) {
        this.userName = userName;
    }
}

// 正确示例
@Service
@RefreshScope
public class UserService {
    @Value("${user.name}")
    private String userName;
    
    // 使用字段注入或Setter注入
}
```

#### ⚠️ 陷阱3: 加密配置无法解密

**原因**:
- 未安装JCE无限制权限策略文件
- 加密密钥配置错误
- 加密值格式错误(缺少{cipher}前缀)

**解决方案**:
```yaml
# 确保加密值格式正确
spring:
  datasource:
    password: '{cipher}AQA3eHg7...'  # 必须有单引号和{cipher}前缀
```

#### ⚠️ 陷阱4: Git仓库拉取失败

**原因**:
- 网络不通
- 认证失败
- 分支不存在
- 仓库地址错误

**解决方案**:
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/your-org/config-repo.git
          username: ${GIT_USERNAME}
          password: ${GIT_PASSWORD}
          timeout: 10
          # 启用详细日志
          
logging:
  level:
    org.springframework.cloud.config.server: DEBUG
```

#### ⚠️ 陷阱5: 配置优先级混乱

**配置加载顺序** (从高到低):
```
1. 命令行参数
2. 系统环境变量
3. Config Server配置
4. application.yml本地配置
5. bootstrap.yml配置
```

**最佳实践**:
- 环境相关配置放在Config Server
- 应用基础配置放在本地application.yml
- 启动配置放在bootstrap.yml

## ❓ 常见问题

### Q1: Spring Cloud Config和Nacos/Apollo有什么区别?

**A**: 主要区别:

| 特性 | Spring Cloud Config | Nacos | Apollo |
|------|---------------------|-------|--------|
| 存储方式 | Git/SVN/本地文件 | 数据库 | 数据库 |
| 配置界面 | 无(依赖Git) | 有 | 有 |
| 动态刷新 | 需要手动/Bus | 自动 | 自动 |
| 版本管理 | Git原生支持 | 支持 | 支持 |
| 权限控制 | 依赖Git | 支持 | 支持 |
| 灰度发布 | 不支持 | 支持 | 支持 |
| 学习成本 | 低 | 中 | 中 |
| 社区活跃度 | 高 | 高 | 高 |

**选择建议**:
- 小型项目、已有Git基础设施: Spring Cloud Config
- 需要配置界面、灰度发布: Nacos或Apollo
- 阿里云生态: Nacos
- 携程生态: Apollo

### Q2: 如何实现配置的灰度发布?

**A**: Spring Cloud Config本身不支持灰度发布,可以通过以下方案实现:

**方案1: 使用Git分支**
```
main分支 → 正式环境
gray分支 → 灰度环境

# 灰度实例配置
spring:
  cloud:
    config:
      label: gray
```

**方案2: 使用不同的配置文件**
```
user-service-prod.yml      # 正式配置
user-service-prod-gray.yml # 灰度配置

# 灰度实例配置
spring:
  profiles:
    active: prod-gray
```

**方案3: 结合服务网格**
使用Istio等服务网格实现流量灰度,配置保持一致。

### Q3: Config Server如何保证高可用?

**A**: 高可用方案:

**1. 多实例部署**
```
部署3个以上Config Server实例
使用Nginx/SLB做负载均衡
```

**2. 客户端配置多地址**
```yaml
spring:
  cloud:
    config:
      uri: http://config1:8888,http://config2:8888,http://config3:8888
      fail-fast: false
      retry:
        max-attempts: 6
```

**3. 本地缓存**
```yaml
spring:
  cloud:
    config:
      allow-override: true
```

**4. 健康检查**
```yaml
management:
  endpoint:
    health:
      show-details: always
```

### Q4: 如何保证配置的安全性?

**A**: 安全措施:

**1. 敏感信息加密**
```yaml
spring:
  datasource:
    password: '{cipher}AQA3eHg7...'
```

**2. 使用HTTPS**
```yaml
server:
  ssl:
    enabled: true
```

**3. 启用认证**
```yaml
spring:
  security:
    user:
      name: config-user
      password: config-password
```

**4. Git仓库权限控制**
- 使用私有仓库
- SSH密钥认证
- 分支保护

**5. 网络隔离**
- Config Server部署在内网
- 配置IP白名单
- 使用VPN访问

### Q5: 配置刷新会影响正在处理的请求吗?

**A**: 不会影响正在处理的请求。

**原理**:
- @RefreshScope会创建代理对象
- 配置刷新时创建新的Bean实例
- 正在处理的请求继续使用旧实例
- 新请求使用新实例

**注意事项**:
- 避免在@RefreshScope的Bean中保存状态
- 数据库连接池等资源类不要使用@RefreshScope
- 配置刷新可能导致短暂的性能下降

### Q6: 如何回滚配置?

**A**: 回滚方案:

**方案1: Git回滚**
```bash
# 回滚到上一个版本
git revert HEAD
git push

# 刷新配置
curl -X POST http://localhost:8888/actuator/bus-refresh
```

**方案2: 使用Git标签**
```bash
# 创建标签
git tag v1.0.0
git push origin v1.0.0

# 回滚到标签
spring:
  cloud:
    config:
      label: v1.0.0
```

**方案3: 使用分支**
```bash
# 切换到稳定分支
spring:
  cloud:
    config:
      label: stable
```

### Q7: Config Server启动慢怎么优化?

**A**: 优化方案:

**1. 使用浅克隆**
```yaml
spring:
  cloud:
    config:
      server:
        git:
          clone-on-start: true
```

**2. 配置本地缓存**
```yaml
spring:
  cloud:
    config:
      server:
        git:
          basedir: /tmp/config-repo
```

**3. 减少搜索路径**
```yaml
spring:
  cloud:
    config:
      server:
        git:
          search-paths: '{application}'  # 只搜索应用目录
```

**4. 使用本地Git仓库**
```yaml
spring:
  cloud:
    config:
      server:
        git:
          uri: file:///opt/config-repo
```

## 📚 参考资料

### 官方文档
- [Spring Cloud Config官方文档](https://spring.io/projects/spring-cloud-config)
- [Spring Cloud Config参考指南](https://docs.spring.io/spring-cloud-config/docs/current/reference/html/)

### 推荐阅读
- 《Spring Cloud微服务实战》
- 《Spring Cloud Alibaba微服务原理与实战》
- [Spring Cloud Config最佳实践](https://spring.io/guides/gs/centralized-configuration/)

### 相关技术
- [Spring Cloud Bus](https://spring.io/projects/spring-cloud-bus)
- [Spring Cloud Netflix](https://spring.io/projects/spring-cloud-netflix)
- [Nacos配置中心](./Nacos-完整教程.md)
- [Apollo配置中心](./Apollo-完整教程.md)

---

**学习建议**:
1. 先掌握Git基础操作
2. 理解Spring Boot配置加载机制
3. 从单机模式开始实践
4. 逐步学习加密、刷新等高级特性
5. 最后学习集群部署和高可用方案

**实战项目推荐**:
- 搭建一个完整的Config Server + Client环境
- 实现配置的加密解密
- 集成Spring Cloud Bus实现自动刷新
- 部署Config Server集群

**下一步学习**:
- [Nacos配置中心](./Nacos-完整教程.md) - 更强大的配置管理
- [Apollo配置中心](./Apollo-完整教程.md) - 携程开源的配置中心
- [Spring Cloud Gateway](../Spring-Cloud-Gateway-完整教程.md) - API网关
- [Spring Cloud Alibaba](../../02-Spring生态/07-Spring-Cloud-Alibaba-完整教程.md) - 微服务全家桶
