# Spring Boot 完整教程

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
- **版本**: Spring Boot 3.2.x
- **官方文档**: https://spring.io/projects/spring-boot
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Spring Framework核心知识
  - Java基础（JDK 17+）
  - Maven/Gradle构建工具
  - 基本的Web开发知识
- **文档来源**: Context7 + Spring Boot官方文档
- **更新时间**: 2024-12-31
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Spring Boot的设计理念和核心价值
- [ ] 掌握Spring Boot自动配置原理
- [ ] 理解Starter机制和依赖管理
- [ ] 掌握Spring Boot Actuator监控
- [ ] 理解条件注解的使用
- [ ] 能够创建自定义Starter
- [ ] 掌握Spring Boot配置管理

## 📖 基础概念

### 1.1 什么是Spring Boot

Spring Boot是一个基于Spring Framework的快速应用开发框架，旨在简化Spring应用的初始搭建和开发过程。它通过"约定优于配置"的理念，提供了开箱即用的功能，让开发者能够快速构建独立的、生产级别的Spring应用。

**核心特性**：
- 创建独立的Spring应用
- 直接嵌入Tomcat、Jetty或Undertow（无需部署WAR文件）
- 提供"starter"依赖，简化构建配置
- 自动配置Spring和第三方库
- 提供生产就绪功能，如指标、健康检查和外部化配置
- 完全无代码生成，无需XML配置

### 1.2 核心概念

- **自动配置（Auto-Configuration）**: 根据类路径和配置自动配置Spring应用
- **Starter**: 一组方便的依赖描述符，简化依赖管理
- **条件注解（Conditional Annotations）**: 控制配置类何时生效
- **Actuator**: 提供生产就绪的监控和管理功能
- **外部化配置**: 支持多种配置源（properties、YAML、环境变量等）
- **嵌入式服务器**: 内置Tomcat、Jetty或Undertow

### 1.3 应用场景

- 快速构建微服务应用
- 开发RESTful API
- 构建企业级Web应用
- 开发批处理应用
- 构建云原生应用
- 快速原型开发


## 🔥 核心特性

### 2.1 自动配置机制 🔥 (⚠️ 难点)

#### 2.1.1 自动配置原理

Spring Boot的自动配置是其最核心的特性。它根据类路径中的内容和应用程序属性自动配置应用程序Bean，大大减少了样板配置代码。

**自动配置的工作流程**：

1. Spring Boot启动时扫描`META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件
2. 加载所有自动配置类
3. 根据条件注解判断是否应用该配置
4. 创建并注册相应的Bean

**自动配置类示例**：

```java
@AutoConfiguration
@ConditionalOnClass(SomeClass.class)
@ConditionalOnMissingBean
public class MyAutoConfiguration {
    
    @Bean
    public MyService myService() {
        return new MyService();
    }
}
```

#### 2.1.2 条件注解详解 (⚠️ 难点)

条件注解是自动配置的核心机制，用于控制配置类何时生效。

**常用条件注解**：

| 注解 | 说明 | 示例 |
|------|------|------|
| @ConditionalOnClass | 类路径存在指定类时生效 | @ConditionalOnClass(RedisTemplate.class) |
| @ConditionalOnMissingClass | 类路径不存在指定类时生效 | @ConditionalOnMissingClass("com.example.SomeClass") |
| @ConditionalOnBean | 容器中存在指定Bean时生效 | @ConditionalOnBean(DataSource.class) |
| @ConditionalOnMissingBean | 容器中不存在指定Bean时生效 | @ConditionalOnMissingBean |
| @ConditionalOnProperty | 配置属性满足条件时生效 | @ConditionalOnProperty(name="app.enabled", havingValue="true") |
| @ConditionalOnResource | 类路径存在指定资源时生效 | @ConditionalOnResource(resources="classpath:config.xml") |
| @ConditionalOnWebApplication | Web应用环境时生效 | @ConditionalOnWebApplication |
| @ConditionalOnExpression | SpEL表达式为true时生效 | @ConditionalOnExpression("${app.enabled:true}") |

**完整的自动配置示例**：

```java
@Configuration
@ConditionalOnClass(RedisTemplate.class)
@EnableConfigurationProperties(CacheProperties.class)
public class CacheAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnProperty(prefix = "app.cache", name = "enabled", havingValue = "true")
    public CacheManager cacheManager(CacheProperties properties) {
        RedisCacheManager manager = new RedisCacheManager();
        manager.setHost(properties.getHost());
        manager.setPort(properties.getPort());
        manager.setTtl(properties.getTtl());
        return manager;
    }

    @Bean
    @ConditionalOnMissingBean
    public CacheService cacheService(CacheManager cacheManager) {
        return new CacheService(cacheManager);
    }
}

@ConfigurationProperties(prefix = "app.cache")
class CacheProperties {
    private boolean enabled = true;
    private String host = "localhost";
    private int port = 6379;
    private int ttl = 3600;

    // Getter和Setter方法
    public boolean isEnabled() { 
        return enabled; 
    }
    
    public void setEnabled(boolean enabled) { 
        this.enabled = enabled; 
    }
    
    public String getHost() { 
        return host; 
    }
    
    public void setHost(String host) { 
        this.host = host; 
    }
    
    public int getPort() { 
        return port; 
    }
    
    public void setPort(int port) { 
        this.port = port; 
    }
    
    public int getTtl() { 
        return ttl; 
    }
    
    public void setTtl(int ttl) { 
        this.ttl = ttl; 
    }
}
```

**配置文件（application.yml）**：

```yaml
app:
  cache:
    enabled: true
    host: localhost
    port: 6379
    ttl: 3600
```

### 2.2 Starter机制 🔥

#### 2.2.1 什么是Starter

Starter是一组方便的依赖描述符，可以包含在应用程序中。它提供了一站式的依赖管理，无需搜索示例代码和复制粘贴大量的依赖描述符。

**Starter的优势**：
- 简化依赖管理
- 提供默认配置
- 版本兼容性保证
- 开箱即用

**常用官方Starter**：

| Starter | 功能 |
|---------|------|
| spring-boot-starter-web | Web应用开发（包含Spring MVC、Tomcat） |
| spring-boot-starter-data-jpa | JPA数据访问 |
| spring-boot-starter-data-redis | Redis数据访问 |
| spring-boot-starter-security | Spring Security安全框架 |
| spring-boot-starter-test | 测试框架（JUnit、Mockito等） |
| spring-boot-starter-actuator | 生产就绪功能 |
| spring-boot-starter-validation | Bean验证 |
| spring-boot-starter-aop | AOP支持 |

**使用Starter示例**：

```xml
<dependencies>
    <!-- Web Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- JPA Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- MySQL驱动 -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

#### 2.2.2 创建自定义Starter (⚠️ 难点)

创建自定义Starter可以将通用功能封装成可复用的组件。

**Starter项目结构**：

```
my-spring-boot-starter/
├── pom.xml
└── src/main/
    ├── java/
    │   └── com/example/starter/
    │       ├── MyAutoConfiguration.java
    │       ├── MyProperties.java
    │       └── MyService.java
    └── resources/
        └── META-INF/
            └── spring/
                └── org.springframework.boot.autoconfigure.AutoConfiguration.imports
```

**步骤1：创建自动配置类**：

```java
package com.example.starter;

import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConditionalOnClass(MyService.class)
@EnableConfigurationProperties(MyProperties.class)
public class MyAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnProperty(prefix = "my.service", name = "enabled", havingValue = "true", matchIfMissing = true)
    public MyService myService(MyProperties properties) {
        MyService service = new MyService();
        service.setPrefix(properties.getPrefix());
        service.setSuffix(properties.getSuffix());
        return service;
    }
}
```

**步骤2：创建配置属性类**：

```java
package com.example.starter;

import org.springframework.boot.context.properties.ConfigurationProperties;

/**
 * 自定义Starter配置属性
 * 
 * @author erik.zhou
 */
@ConfigurationProperties(prefix = "my.service")
public class MyProperties {
    
    /**
     * Whether to enable the service
     */
    private boolean enabled = true;
    
    /**
     * Prefix to add to messages
     */
    private String prefix = "Hello";
    
    /**
     * Suffix to add to messages
     */
    private String suffix = "!";

    // Getter和Setter方法
    public boolean isEnabled() {
        return enabled;
    }

    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
    }

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

**步骤3：创建服务类**：

```java
package com.example.starter;

public class MyService {
    
    private String prefix;
    private String suffix;

    public String wrap(String message) {
        return prefix + " " + message + " " + suffix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

**步骤4：注册自动配置类**：

在`src/main/resources/META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports`文件中：

```text
com.example.starter.MyAutoConfiguration
```

**步骤5：配置pom.xml**：

```xml
<project>
    <groupId>com.example</groupId>
    <artifactId>my-spring-boot-starter</artifactId>
    <version>1.0.0</version>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

**使用自定义Starter**：

```xml
<!-- 在其他项目中引入 -->
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-spring-boot-starter</artifactId>
    <version>1.0.0</version>
</dependency>
```

```java
@RestController
public class DemoController {
    
    @Autowired
    private MyService myService;
    
    @GetMapping("/hello")
    public String hello() {
        return myService.wrap("World");  // 输出: Hello World !
    }
}
```

**配置文件**：

```yaml
my:
  service:
    enabled: true
    prefix: "你好"
    suffix: "！"
```


### 2.3 Spring Boot Actuator 🔥

#### 2.3.1 什么是Actuator

Spring Boot Actuator提供了生产就绪的功能，帮助监控和管理应用程序。它通过HTTP端点或JMX暴露应用程序的运行时信息。

**添加Actuator依赖**：

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```

#### 2.3.2 常用端点

| 端点 | 说明 | 默认启用 |
|------|------|---------|
| /actuator/health | 应用健康状态 | 是 |
| /actuator/info | 应用信息 | 是 |
| /actuator/metrics | 应用指标 | 是 |
| /actuator/env | 环境属性 | 是 |
| /actuator/beans | Spring Bean列表 | 是 |
| /actuator/mappings | 请求映射 | 是 |
| /actuator/loggers | 日志配置 | 是 |
| /actuator/threaddump | 线程转储 | 是 |
| /actuator/heapdump | 堆转储 | 是 |
| /actuator/shutdown | 关闭应用（需启用） | 否 |

**配置Actuator**：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,env  # 暴露的端点
      base-path: /actuator  # 基础路径
  endpoint:
    health:
      show-details: always  # 显示详细健康信息
    shutdown:
      enabled: true  # 启用shutdown端点
  metrics:
    tags:
      application: ${spring.application.name}  # 添加标签
```

#### 2.3.3 自定义健康检查

```java
@Component
public class CustomHealthIndicator implements HealthIndicator {

    @Override
    public Health health() {
        // 执行健康检查逻辑
        boolean isHealthy = checkExternalService();
        
        if (isHealthy) {
            return Health.up()
                .withDetail("service", "available")
                .withDetail("timestamp", System.currentTimeMillis())
                .build();
        } else {
            return Health.down()
                .withDetail("service", "unavailable")
                .withDetail("error", "Connection timeout")
                .build();
        }
    }
    
    private boolean checkExternalService() {
        // 实际的健康检查逻辑
        return true;
    }
}
```

#### 2.3.4 自定义指标

```java
@Service
public class OrderService {
    
    private final MeterRegistry meterRegistry;
    private final Counter orderCounter;
    
    public OrderService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        // 创建计数器
        this.orderCounter = Counter.builder("orders.created")
            .description("Total number of orders created")
            .tag("type", "online")
            .register(meterRegistry);
    }
    
    public void createOrder(Order order) {
        // 业务逻辑
        orderCounter.increment();
        
        // 记录订单金额
        meterRegistry.gauge("orders.amount", order.getAmount());
    }
}
```

### 2.4 配置管理

#### 2.4.1 配置文件

Spring Boot支持多种配置文件格式：

**application.properties**：

```properties
# 服务器配置
server.port=8080
server.servlet.context-path=/api

# 数据源配置
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA配置
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
```

**application.yml**（推荐）：

```yaml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

#### 2.4.2 多环境配置

**配置文件命名规则**：
- `application.yml` - 默认配置
- `application-dev.yml` - 开发环境
- `application-test.yml` - 测试环境
- `application-prod.yml` - 生产环境

**激活配置文件**：

```yaml
# application.yml
spring:
  profiles:
    active: dev  # 激活dev配置
```

或通过命令行：

```bash
java -jar myapp.jar --spring.profiles.active=prod
```

**示例配置**：

```yaml
# application-dev.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/dev_db
logging:
  level:
    root: DEBUG

# application-prod.yml
spring:
  datasource:
    url: jdbc:mysql://prod-server:3306/prod_db
logging:
  level:
    root: WARN
```

#### 2.4.3 配置属性绑定

```java
@Component
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    
    private String name;
    private String version;
    private Security security = new Security();
    
    public static class Security {
        private boolean enabled;
        private List<String> allowedOrigins;
        
        // Getter和Setter
        public boolean isEnabled() {
            return enabled;
        }
        
        public void setEnabled(boolean enabled) {
            this.enabled = enabled;
        }
        
        public List<String> getAllowedOrigins() {
            return allowedOrigins;
        }
        
        public void setAllowedOrigins(List<String> allowedOrigins) {
            this.allowedOrigins = allowedOrigins;
        }
    }
    
    // Getter和Setter
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public String getVersion() {
        return version;
    }
    
    public void setVersion(String version) {
        this.version = version;
    }
    
    public Security getSecurity() {
        return security;
    }
    
    public void setSecurity(Security security) {
        this.security = security;
    }
}
```

**配置文件**：

```yaml
app:
  name: MyApplication
  version: 1.0.0
  security:
    enabled: true
    allowed-origins:
      - http://localhost:3000
      - https://example.com
```

## 💻 实战应用

### 3.1 环境搭建

**使用Spring Initializr创建项目**：

访问 https://start.spring.io/ 或使用IDE插件：

1. 选择项目类型：Maven/Gradle
2. 选择语言：Java
3. 选择Spring Boot版本：3.2.x
4. 填写项目信息
5. 添加依赖：Web、JPA、MySQL等
6. 生成并下载项目

**Maven依赖示例**：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.1</version>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

### 3.2 快速开始

**创建启动类**：

```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

**创建REST Controller**：

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    @PostMapping
    public User createUser(@RequestBody @Valid User user) {
        return userService.save(user);
    }
    
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody @Valid User user) {
        return userService.update(id, user);
    }
    
    @DeleteMapping("/{id}")
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
}
```

**运行应用**：

```bash
mvn spring-boot:run
```

或

```bash
./mvnw spring-boot:run
```

访问：http://localhost:8080/api/users

### 3.3 进阶案例：构建完整的RESTful API

**实体类**：

```java
@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 50)
    @NotBlank(message = "用户名不能为空")
    private String username;
    
    @Column(nullable = false, length = 100)
    @Email(message = "邮箱格式不正确")
    private String email;
    
    @Column(nullable = false)
    @Min(value = 0, message = "年龄不能小于0")
    @Max(value = 150, message = "年龄不能大于150")
    private Integer age;
    
    @CreationTimestamp
    @Column(name = "created_at", updatable = false)
    private LocalDateTime createdAt;
    
    @UpdateTimestamp
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
    
    // 构造器、Getter、Setter省略
}
```

**Repository层**：

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    
    Optional<User> findByUsername(String username);
    
    List<User> findByAgeGreaterThan(Integer age);
    
    @Query("SELECT u FROM User u WHERE u.email LIKE %:domain")
    List<User> findByEmailDomain(@Param("domain") String domain);
}
```

**Service层**：

```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public List<User> findAll() {
        return userRepository.findAll();
    }
    
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("用户不存在: " + id));
    }
    
    @Transactional
    public User save(User user) {
        // 检查用户名是否已存在
        if (userRepository.findByUsername(user.getUsername()).isPresent()) {
            throw new DuplicateResourceException("用户名已存在: " + user.getUsername());
        }
        return userRepository.save(user);
    }
    
    @Transactional
    public User update(Long id, User user) {
        User existingUser = findById(id);
        existingUser.setUsername(user.getUsername());
        existingUser.setEmail(user.getEmail());
        existingUser.setAge(user.getAge());
        return userRepository.save(existingUser);
    }
    
    @Transactional
    public void delete(Long id) {
        User user = findById(id);
        userRepository.delete(user);
    }
}
```

**全局异常处理**：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleResourceNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return new ResponseEntity<>(error, HttpStatus.NOT_FOUND);
    }
    
    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ErrorResponse> handleDuplicateResource(DuplicateResourceException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return new ResponseEntity<>(error, HttpStatus.CONFLICT);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errors.put(error.getField(), error.getDefaultMessage())
        );
        
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "验证失败",
            LocalDateTime.now(),
            errors
        );
        return new ResponseEntity<>(error, HttpStatus.BAD_REQUEST);
    }
}
```

**配置文件**：

```yaml
spring:
  application:
    name: user-service
  datasource:
    url: jdbc:mysql://localhost:3306/userdb?useSSL=false&serverTimezone=UTC
    username: root
    password: password
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.MySQL8Dialect
  jackson:
    time-zone: GMT+8
    date-format: yyyy-MM-dd HH:mm:ss

server:
  port: 8080
  servlet:
    context-path: /api

logging:
  level:
    com.example: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```


## ✨ 最佳实践

### 4.1 项目结构规范

```
src/main/java/com/example/demo/
├── DemoApplication.java          # 启动类
├── config/                       # 配置类
│   ├── WebConfig.java
│   ├── SecurityConfig.java
│   └── DatabaseConfig.java
├── controller/                   # 控制器层
│   ├── UserController.java
│   └── OrderController.java
├── service/                      # 服务层
│   ├── UserService.java
│   └── OrderService.java
├── repository/                   # 数据访问层
│   ├── UserRepository.java
│   └── OrderRepository.java
├── entity/                       # 实体类
│   ├── User.java
│   └── Order.java
├── dto/                          # 数据传输对象
│   ├── UserDTO.java
│   └── OrderDTO.java
├── exception/                    # 异常类
│   ├── ResourceNotFoundException.java
│   └── GlobalExceptionHandler.java
└── util/                         # 工具类
    └── DateUtil.java
```

### 4.2 配置管理最佳实践

1. **使用YAML格式**（更清晰、层次分明）

```yaml
# 推荐
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db
    username: root
    password: password
```

2. **敏感信息外部化**

```yaml
# 使用环境变量
spring:
  datasource:
    password: ${DB_PASSWORD}
```

3. **使用配置类而非@Value**

```java
// 不推荐
@Value("${app.name}")
private String appName;

// 推荐
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    // getter/setter
}
```

### 4.3 依赖管理最佳实践

1. **使用spring-boot-starter-parent**

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.1</version>
</parent>
```

2. **优先使用官方Starter**

```xml
<!-- 推荐：使用官方Starter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- 不推荐：手动管理依赖 -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>6.1.0</version>
</dependency>
```

3. **避免版本冲突**

```xml
<!-- 使用dependencyManagement统一管理版本 -->
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>2.0.32</version>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### 4.4 日志管理最佳实践

1. **使用SLF4J门面**

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class UserService {
    
    private static final Logger logger = LoggerFactory.getLogger(UserService.class);
    
    public User createUser(User user) {
        logger.info("创建用户: {}", user.getUsername());
        try {
            // 业务逻辑
            return userRepository.save(user);
        } catch (Exception e) {
            logger.error("创建用户失败", e);
            throw e;
        }
    }
}
```

2. **配置日志级别**

```yaml
logging:
  level:
    root: INFO
    com.example: DEBUG
    org.springframework.web: DEBUG
    org.hibernate.SQL: DEBUG
  file:
    name: logs/application.log
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
```

### 4.5 异常处理最佳实践

1. **使用@RestControllerAdvice统一处理**

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception ex) {
        logger.error("系统异常", ex);
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "系统繁忙，请稍后重试",
            LocalDateTime.now()
        );
        return new ResponseEntity<>(error, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```

2. **自定义业务异常**

```java
public class BusinessException extends RuntimeException {
    private final int code;
    
    public BusinessException(int code, String message) {
        super(message);
        this.code = code;
    }
    
    public int getCode() {
        return code;
    }
}
```

### 4.6 性能优化建议

1. **使用连接池**

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
```

2. **启用HTTP缓存**

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
            .addResourceLocations("classpath:/static/")
            .setCacheControl(CacheControl.maxAge(365, TimeUnit.DAYS));
    }
}
```

3. **使用异步处理**

```java
@Configuration
@EnableAsync
public class AsyncConfig {
    
    @Bean
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.initialize();
        return executor;
    }
}

@Service
public class EmailService {
    
    @Async
    public void sendEmail(String to, String subject, String content) {
        // 异步发送邮件
    }
}
```

## ❓ 常见问题

### Q1: Spring Boot和Spring Framework的区别？

**A**: 
- Spring Framework是基础框架，提供IoC、AOP等核心功能
- Spring Boot是基于Spring Framework的快速开发框架，提供自动配置、Starter等便利功能
- Spring Boot简化了Spring应用的配置和部署，但底层仍然是Spring Framework

### Q2: 如何排查自动配置问题？

**A**: 
1. 启用debug日志：`--debug`或`logging.level.org.springframework.boot.autoconfigure=DEBUG`
2. 查看自动配置报告：启动时会打印哪些配置生效、哪些未生效
3. 使用Actuator的`/actuator/conditions`端点查看条件评估结果
4. 检查类路径中是否存在必需的依赖

### Q3: 如何禁用特定的自动配置？

**A**: 
```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})
public class Application {
    // ...
}
```

或在配置文件中：

```yaml
spring:
  autoconfigure:
    exclude:
      - org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration
```

### Q4: 如何实现优雅停机？

**A**: 
```yaml
server:
  shutdown: graceful  # 启用优雅停机

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s  # 停机超时时间
```

### Q5: 如何处理循环依赖？

**A**: 
1. 重构代码，消除循环依赖（最佳方案）
2. 使用`@Lazy`注解延迟加载
3. 使用Setter注入代替构造器注入
4. 将共同依赖提取到第三个类

### Q6: 如何自定义Banner？

**A**: 
在`src/main/resources`下创建`banner.txt`文件：

```text
  __  __                            
 |  \/  |_   _     /\   _ __  _ __  
 | |\/| | | | |   /  \ | '_ \| '_ \ 
 | |  | | |_| |  / /\ \| |_) | |_) |
 |_|  |_|\__, | /_/  \_\ .__/| .__/ 
         |___/         |_|   |_|    
         
Spring Boot Version: ${spring-boot.version}
```

## 🔗 相关资源

### 官方文档
- [Spring Boot官网](https://spring.io/projects/spring-boot)
- [Spring Boot参考文档](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Boot API文档](https://docs.spring.io/spring-boot/docs/current/api/)
- [Spring Initializr](https://start.spring.io/)

### 推荐文章
- [Spring Boot自动配置原理](https://www.baeldung.com/spring-boot-custom-auto-configuration)
- [创建自定义Starter](https://www.baeldung.com/spring-boot-custom-starter)
- [Spring Boot Actuator指南](https://www.baeldung.com/spring-boot-actuators)

### 视频教程
- [Spring Boot官方教程](https://spring.io/guides)
- [Spring Boot微服务实战](https://www.youtube.com/springdevelopers)

### 推荐书籍
- 《Spring Boot实战》
- 《Spring Boot编程思想》
- 《Spring Boot微服务实战》

## 📝 学习检查清单

### 基础知识
- [ ] 理解Spring Boot的设计理念
- [ ] 掌握Spring Boot项目的创建和启动
- [ ] 理解@SpringBootApplication注解的作用
- [ ] 掌握配置文件的使用（properties和YAML）
- [ ] 理解多环境配置

### 核心特性
- [ ] 理解自动配置的原理
- [ ] 掌握条件注解的使用
- [ ] 理解Starter的作用和使用
- [ ] 能够创建自定义Starter
- [ ] 掌握Actuator的使用
- [ ] 理解配置属性绑定

### 实战能力
- [ ] 能够快速搭建Spring Boot项目
- [ ] 能够构建RESTful API
- [ ] 能够集成数据库（JPA/MyBatis）
- [ ] 能够实现全局异常处理
- [ ] 能够配置日志系统
- [ ] 能够使用Actuator监控应用

### 最佳实践
- [ ] 掌握项目结构规范
- [ ] 理解配置管理最佳实践
- [ ] 掌握依赖管理技巧
- [ ] 理解性能优化方法
- [ ] 掌握异常处理模式

---

**学习建议**：
1. 先掌握Spring Framework基础
2. 通过Spring Initializr快速创建项目
3. 理解自动配置原理（重点）
4. 实践创建自定义Starter
5. 学习Actuator监控应用
6. 在实际项目中应用所学知识

**预计学习时长**: 25-35小时（基础学习）+ 50-80小时（进阶学习）

**下一步学习**：
- Spring MVC：深入学习Web开发
- Spring Data JPA：数据访问层
- Spring Security：安全框架
- Spring Cloud：微服务架构
