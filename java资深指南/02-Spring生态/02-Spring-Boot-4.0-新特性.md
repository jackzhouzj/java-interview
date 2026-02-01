# Spring Boot 4.0 新特性与升级指南

> 📚 **版本**: Spring Boot 4.0.0  
> 🎯 **学习难度**: ⭐⭐⭐⭐  
> 🔥 **重要程度**: ⭐⭐⭐⭐⭐  
> ⏱️ **预计学习时长**: 20-25小时  
> 📅 **最后更新**: 2025-02-01  
> 👤 **作者**: erik.zhou

---

## 📖 目录

- [技术概述](#技术概述)
- [重大变更](#重大变更)
- [核心新特性](#核心新特性)
- [虚拟线程支持](#虚拟线程支持)
- [Docker Compose集成](#docker-compose集成)
- [Testcontainers集成](#testcontainers集成)
- [可观测性增强](#可观测性增强)
- [升级指南](#升级指南)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [相关资源](#相关资源)

---

## 📚 技术概述

### 什么是 Spring Boot 4.0？

Spring Boot 4.0 是Spring Boot框架的重大版本更新，于2025年发布。它基于Spring Framework 7.0.1构建，要求JDK 24作为最低版本，带来了虚拟线程原生支持、Docker Compose深度集成、Testcontainers自动配置等重要特性。

### 版本信息

- **发布日期**: 2025年
- **Spring Framework版本**: 7.0.1
- **最低JDK版本**: JDK 24
- **前置版本**: Spring Boot 3.x
- **支持周期**: 长期支持(LTS)

### 主要特性概览

| 特性 | 重要性 | 状态 |
|------|--------|------|
| 虚拟线程原生支持 | ⭐⭐⭐⭐⭐ | 稳定 |
| Docker Compose集成 | ⭐⭐⭐⭐⭐ | 稳定 |
| Testcontainers自动配置 | ⭐⭐⭐⭐⭐ | 稳定 |
| 增强的可观测性 | ⭐⭐⭐⭐ | 稳定 |
| GraalVM Native Image优化 | ⭐⭐⭐⭐ | 稳定 |
| 改进的开发者体验 | ⭐⭐⭐⭐ | 稳定 |

---

## 🔥 重大变更

### 1. JDK版本要求

```xml
<!-- Maven配置 -->
<properties>
    <java.version>24</java.version>
    <maven.compiler.source>24</maven.compiler.source>
    <maven.compiler.target>24</maven.compiler.target>
</properties>

<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.0.0</version>
</parent>
```

```groovy
// Gradle配置
plugins {
    id 'java'
    id 'org.springframework.boot' version '4.0.0'
    id 'io.spring.dependency-management' version '1.1.0'
}

java {
    sourceCompatibility = '24'
    targetCompatibility = '24'
}
```

### 2. 依赖版本更新

| 依赖 | Spring Boot 3.x | Spring Boot 4.0 |
|------|----------------|-----------------|
| Spring Framework | 6.x | 7.0.1 |
| Jakarta EE | 10 | 11 |
| Hibernate | 6.2.x | 6.4.x |
| Tomcat | 10.1.x | 11.0.x |
| Jetty | 12.0.x | 12.1.x |

---

## 🔥 核心新特性

### 1. 虚拟线程原生支持

🔥 **重点**: Spring Boot 4.0原生支持Java虚拟线程，大幅提升并发性能

#### 1.1 启用虚拟线程

```yaml
# application.yml
spring:
  threads:
    virtual:
      enabled: true  # 启用虚拟线程
```

```java
@SpringBootApplication
public class Application {
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    
    // 自动使用虚拟线程的TaskExecutor
    @Bean
    public TaskExecutor taskExecutor() {
        return new SimpleAsyncTaskExecutor();  // 自动使用虚拟线程
    }
}
```

#### 1.2 Web请求处理

```java
@RestController
@RequestMapping("/api")
public class VirtualThreadController {
    
    @Autowired
    private UserService userService;
    
    // 每个请求自动在虚拟线程中处理
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable Long id) {
        // 阻塞调用不会阻塞平台线程
        return userService.findById(id);
    }
    
    // 异步方法自动使用虚拟线程
    @Async
    @GetMapping("/users/{id}/async")
    public CompletableFuture<User> getUserAsync(@PathVariable Long id) {
        return CompletableFuture.completedFuture(
            userService.findById(id)
        );
    }
}
```

#### 1.3 数据库连接池配置

```yaml
# application.yml
spring:
  datasource:
    hikari:
      # 虚拟线程下可以大幅增加连接池大小
      maximum-pool-size: 1000  # 传统线程通常只能设置10-50
      minimum-idle: 100
```

```java
@Configuration
public class DatabaseConfig {
    
    @Bean
    public HikariDataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        config.setUsername("root");
        config.setPassword("password");
        
        // 虚拟线程环境下的优化配置
        config.setMaximumPoolSize(1000);
        config.setMinimumIdle(100);
        config.setConnectionTimeout(30000);
        
        return new HikariDataSource(config);
    }
}
```

#### 1.4 性能对比

```java
@RestController
public class PerformanceTestController {
    
    @GetMapping("/test/platform-threads")
    public String testPlatformThreads() throws InterruptedException {
        long start = System.currentTimeMillis();
        
        // 传统平台线程 - 受限于线程池大小
        ExecutorService executor = Executors.newFixedThreadPool(100);
        for (int i = 0; i < 10000; i++) {
            executor.submit(() -> {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            });
        }
        executor.shutdown();
        executor.awaitTermination(1, TimeUnit.HOURS);
        
        return "Platform threads: " + (System.currentTimeMillis() - start) + "ms";
    }
    
    @GetMapping("/test/virtual-threads")
    public String testVirtualThreads() throws InterruptedException {
        long start = System.currentTimeMillis();
        
        // 虚拟线程 - 可以轻松创建数百万个
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < 10000; i++) {
                executor.submit(() -> {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
            }
        }
        
        return "Virtual threads: " + (System.currentTimeMillis() - start) + "ms";
    }
}
```

---

### 2. Docker Compose集成

🔥 **重点**: 开发环境自动管理Docker服务生命周期

#### 2.1 基本配置

```yaml
# compose.yaml (项目根目录)
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  kafka:
    image: confluentinc/cp-kafka:7.5.0
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
    ports:
      - "9092:9092"
```

```yaml
# application.yml
spring:
  docker:
    compose:
      enabled: true  # 启用Docker Compose支持
      file: compose.yaml  # 指定compose文件
      lifecycle-management: start-and-stop  # 生命周期管理
      start:
        command: up  # 启动命令
      stop:
        command: down  # 停止命令
        timeout: 10s
```

#### 2.2 自动配置数据源

```java
@SpringBootApplication
public class Application {
    
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
        // Spring Boot自动:
        // 1. 启动compose.yaml中的服务
        // 2. 等待服务就绪
        // 3. 配置数据源连接
        // 4. 应用关闭时停止服务
    }
}
```

```yaml
# application.yml - 无需手动配置连接信息
spring:
  datasource:
    # Spring Boot自动从Docker Compose服务中获取连接信息
    # url: jdbc:postgresql://localhost:5432/mydb  # 自动配置
    # username: user  # 自动配置
    # password: password  # 自动配置
  
  data:
    redis:
      # host: localhost  # 自动配置
      # port: 6379  # 自动配置
```

#### 2.3 服务依赖管理

```yaml
# compose.yaml
services:
  app:
    build: .
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    environment:
      SPRING_PROFILES_ACTIVE: dev
  
  postgres:
    image: postgres:16
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---

### 3. Testcontainers自动配置

🔥 **重点**: 测试环境自动管理容器化依赖

#### 3.1 基本使用

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

```java
@SpringBootTest
@Testcontainers  // 启用Testcontainers支持
class UserServiceTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");
    
    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        // Spring Boot 4.0自动配置，无需手动设置
        // registry.add("spring.datasource.url", postgres::getJdbcUrl);
        // registry.add("spring.datasource.username", postgres::getUsername);
        // registry.add("spring.datasource.password", postgres::getPassword);
    }
    
    @Autowired
    private UserService userService;
    
    @Test
    void testCreateUser() {
        User user = new User("test@example.com", "Test User");
        User saved = userService.save(user);
        
        assertThat(saved.getId()).isNotNull();
        assertThat(saved.getEmail()).isEqualTo("test@example.com");
    }
}
```

#### 3.2 多容器测试

```java
@SpringBootTest
@Testcontainers
class IntegrationTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:16");
    
    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379);
    
    @Container
    static KafkaContainer kafka = new KafkaContainer(
        DockerImageName.parse("confluentinc/cp-kafka:7.5.0")
    );
    
    @Test
    void testFullIntegration() {
        // 所有容器自动启动和配置
        // 测试完成后自动清理
    }
}
```

#### 3.3 测试配置类

```java
@TestConfiguration(proxyBeanMethods = false)
public class TestContainersConfiguration {
    
    @Bean
    @ServiceConnection  // Spring Boot 4.0新注解
    PostgreSQLContainer<?> postgresContainer() {
        return new PostgreSQLContainer<>("postgres:16");
    }
    
    @Bean
    @ServiceConnection
    GenericContainer<?> redisContainer() {
        return new GenericContainer<>("redis:7-alpine")
            .withExposedPorts(6379);
    }
    
    @Bean
    @ServiceConnection
    KafkaContainer kafkaContainer() {
        return new KafkaContainer(
            DockerImageName.parse("confluentinc/cp-kafka:7.5.0")
        );
    }
}
```

---

### 4. 可观测性增强

#### 4.1 改进的Actuator端点

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: "*"  # 暴露所有端点
  endpoint:
    health:
      show-details: always
      probes:
        enabled: true  # Kubernetes探针支持
  
  # 新增的虚拟线程监控
  metrics:
    tags:
      application: ${spring.application.name}
    export:
      prometheus:
        enabled: true
  
  # 分布式追踪
  tracing:
    sampling:
      probability: 1.0  # 100%采样
```

#### 4.2 虚拟线程监控

```java
@RestController
@RequestMapping("/actuator")
public class CustomActuatorEndpoint {
    
    @GetMapping("/virtual-threads")
    public Map<String, Object> virtualThreadsInfo() {
        return Map.of(
            "isVirtualThreadsEnabled", 
                Thread.currentThread().isVirtual(),
            "activeVirtualThreads", 
                getActiveVirtualThreadCount(),
            "platformThreads", 
                Thread.getAllStackTraces().size()
        );
    }
    
    private long getActiveVirtualThreadCount() {
        // 获取活跃虚拟线程数量
        return Thread.getAllStackTraces().keySet().stream()
            .filter(Thread::isVirtual)
            .count();
    }
}
```

---

## 🔄 升级指南

### 1. 从Spring Boot 3.x升级

#### 步骤1: 更新依赖版本

```xml
<!-- pom.xml -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>4.0.0</version>  <!-- 从3.x升级到4.0 -->
</parent>

<properties>
    <java.version>24</java.version>  <!-- 从17/21升级到24 -->
</properties>
```

#### 步骤2: 更新JDK

```bash
# 下载并安装JDK 24
# 验证版本
java -version
# 输出: java version "24" ...
```

#### 步骤3: 处理废弃API

```java
// 旧代码 (Spring Boot 3.x)
@Configuration
public class OldConfig {
    
    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**");
            }
        };
    }
}

// 新代码 (Spring Boot 4.0)
@Configuration
public class NewConfig {
    
    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(List.of("*"));
        configuration.setAllowedMethods(List.of("*"));
        
        UrlBasedCorsConfigurationSource source = 
            new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

#### 步骤4: 启用虚拟线程

```yaml
# application.yml
spring:
  threads:
    virtual:
      enabled: true  # 新增配置
```

#### 步骤5: 迁移测试

```java
// 旧代码
@SpringBootTest
class OldTest {
    
    @Test
    void test() {
        // 手动管理测试数据库
    }
}

// 新代码 - 使用Testcontainers
@SpringBootTest
@Testcontainers
class NewTest {
    
    @Container
    static PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:16");
    
    @Test
    void test() {
        // 自动管理测试数据库
    }
}
```

---

## ✨ 最佳实践

### 1. 虚拟线程最佳实践

```java
@Configuration
public class VirtualThreadConfig {
    
    // ✅ 推荐: 为I/O密集型任务启用虚拟线程
    @Bean
    public TaskExecutor ioTaskExecutor() {
        SimpleAsyncTaskExecutor executor = new SimpleAsyncTaskExecutor();
        executor.setVirtualThreads(true);
        return executor;
    }
    
    // ✅ 推荐: 为CPU密集型任务使用平台线程
    @Bean
    public TaskExecutor cpuTaskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(Runtime.getRuntime().availableProcessors());
        executor.setMaxPoolSize(Runtime.getRuntime().availableProcessors() * 2);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("cpu-");
        executor.initialize();
        return executor;
    }
}
```

### 2. Docker Compose最佳实践

```yaml
# compose.yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: ${DB_NAME:-mydb}
      POSTGRES_USER: ${DB_USER:-user}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-password}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
  
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis-data:/data

volumes:
  postgres-data:
  redis-data:
```

### 3. 测试最佳实践

```java
@SpringBootTest
@Testcontainers
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class BestPracticeTest {
    
    // 使用静态容器在所有测试间共享
    @Container
    static PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:16")
            .withReuse(true);  // 容器重用
    
    @Test
    @Order(1)
    void testCreate() {
        // 测试创建
    }
    
    @Test
    @Order(2)
    void testRead() {
        // 测试读取
    }
    
    @AfterAll
    static void cleanup() {
        // 清理资源
    }
}
```

---

## ❓ 常见问题

### Q1: 必须升级到JDK 24吗？
A: 是的。Spring Boot 4.0要求最低JDK 24。如果无法升级JDK，请继续使用Spring Boot 3.x。

### Q2: 虚拟线程会自动应用到所有地方吗？
A: 不会。需要显式启用`spring.threads.virtual.enabled=true`，并且只有支持的组件才会使用虚拟线程。

### Q3: Docker Compose集成在生产环境可用吗？
A: 不推荐。Docker Compose集成主要用于开发和测试环境。生产环境应使用Kubernetes等容器编排平台。

### Q4: 如何判断是否应该升级？
A: 考虑以下因素：
- 应用是否I/O密集型（虚拟线程优势明显）
- 团队是否准备好使用JDK 24
- 是否需要最新的Spring特性
- 现有代码的兼容性

---

## 🔗 相关资源

### 官方文档
- [Spring Boot 4.0 Reference](https://docs.spring.io/spring-boot/docs/4.0.0/reference/)
- [Spring Boot 4.0 Release Notes](https://github.com/spring-projects/spring-boot/releases/tag/v4.0.0)
- [Migration Guide](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-4.0-Migration-Guide)

### 推荐阅读
- [Virtual Threads in Spring Boot](https://spring.io/blog/virtual-threads)
- [Docker Compose Support](https://spring.io/blog/docker-compose)
- [Testcontainers Integration](https://spring.io/blog/testcontainers)

---

**恭喜你完成了Spring Boot 4.0新特性的学习！** 🎉

---

> 👤 **作者**: erik.zhou  
> 📅 **最后更新**: 2025-02-01

