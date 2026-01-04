# Spring Cache 完整教程

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
- **版本**: Spring Framework 6.1.x / Spring Boot 3.2.x
- **官方文档**: https://docs.spring.io/spring-framework/reference/integration/cache.html
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Spring Framework 基础
  - Spring Boot 基础
  - Java 注解编程
  - AOP 概念

## 🎯 学习目标
- [ ] 理解 Spring Cache 抽象层的设计理念
- [ ] 掌握核心缓存注解的使用方法
- [ ] 能够配置不同的缓存提供者
- [ ] 理解缓存键生成策略
- [ ] 掌握条件缓存和同步缓存
- [ ] 能够处理常见的缓存问题

## 📖 基础概念

### 1.1 什么是 Spring Cache

Spring Cache 是 Spring Framework 提供的缓存抽象层，它通过注解驱动的方式为应用程序提供透明的缓存支持。Spring Cache 本身不提供缓存实现，而是提供了一套统一的 API 和注解，允许开发者无缝切换不同的缓存提供者。

**核心设计理念**：
- **声明式缓存**：通过注解声明缓存行为，无需侵入业务代码
- **抽象统一**：提供统一的缓存接口，屏蔽底层实现差异
- **灵活配置**：支持多种缓存提供者和自定义配置
- **AOP 驱动**：基于 Spring AOP 实现缓存拦截

### 1.2 核心概念


- **Cache（缓存）**：存储键值对的容器，每个缓存有唯一的名称
- **CacheManager（缓存管理器）**：管理多个 Cache 实例的组件
- **KeyGenerator（键生成器）**：根据方法参数生成缓存键的策略
- **CacheResolver（缓存解析器）**：动态解析使用哪个缓存的策略

### 1.3 应用场景

- **数据库查询结果缓存**：减少数据库访问压力
- **计算密集型操作缓存**：避免重复计算
- **外部 API 调用缓存**：减少网络请求
- **配置信息缓存**：提高配置读取性能
- **用户会话数据缓存**：提升用户体验

## 🔥 核心特性 (重点)

### 2.1 核心缓存注解 🔥

Spring Cache 提供了四个核心注解来声明缓存行为：

#### @Cacheable - 缓存查询结果

`@Cacheable` 是最常用的缓存注解，用于标记方法的返回值应该被缓存。当方法被调用时，Spring 会先检查缓存中是否存在对应的值，如果存在则直接返回缓存值，否则执行方法并将结果存入缓存。

```java
@Service
public class BookService {
    
    /**
     * 根据 ISBN 查询图书
     * 首次调用会执行方法并缓存结果
     * 后续相同参数的调用直接返回缓存值
     */
    @Cacheable("books")
    public Book findBook(ISBN isbn) {
        // 模拟数据库查询
        return bookRepository.findByIsbn(isbn);
    }
    
    /**
     * 使用多个缓存名称
     */
    @Cacheable(cacheNames = {"books", "isbns"})
    public Book findBookMultiCache(ISBN isbn) {
        return bookRepository.findByIsbn(isbn);
    }
}
```

#### @CachePut - 更新缓存

`@CachePut` 注解强制方法执行并将结果更新到缓存中，常用于更新操作。与 `@Cacheable` 不同，它总是会执行方法。

```java
@Service
public class BookService {
    
    /**
     * 更新图书信息
     * 方法总是会执行，并将结果更新到缓存
     */
    @CachePut(cacheNames = "books", key = "#isbn")
    public Book updateBook(ISBN isbn, BookDescriptor descriptor) {
        Book book = bookRepository.findByIsbn(isbn);
        book.setDescription(descriptor.getDescription());
        return bookRepository.save(book);
    }
}
```


#### @CacheEvict - 清除缓存

`@CacheEvict` 注解用于从缓存中移除数据，常用于删除或批量更新操作。

```java
@Service
public class BookService {
    
    /**
     * 删除单个缓存条目
     */
    @CacheEvict(cacheNames = "books", key = "#isbn")
    public void deleteBook(ISBN isbn) {
        bookRepository.deleteByIsbn(isbn);
    }
    
    /**
     * 清空整个缓存
     * allEntries = true 会删除该缓存中的所有条目
     */
    @CacheEvict(cacheNames = "books", allEntries = true)
    public void loadBooks(InputStream batch) {
        // 批量导入图书
        bookRepository.batchImport(batch);
    }
    
    /**
     * 在方法执行前清除缓存
     * beforeInvocation = true 确保即使方法抛出异常也会清除缓存
     */
    @CacheEvict(cacheNames = "books", beforeInvocation = true)
    public void clearBooksBeforeOperation() {
        // 执行某些操作
    }
}
```

#### @Caching - 组合多个缓存操作

`@Caching` 注解允许在同一个方法上组合多个缓存注解，适用于复杂的缓存场景。

```java
@Service
public class BookService {
    
    /**
     * 导入图书时需要清除多个缓存
     */
    @Caching(evict = {
        @CacheEvict(cacheNames = "primary"),
        @CacheEvict(cacheNames = "secondary", key = "#deposit")
    })
    public Book importBooks(String deposit, Date date) {
        // 导入逻辑
        return bookRepository.importFromDeposit(deposit, date);
    }
    
    /**
     * 同时进行缓存和清除操作
     */
    @Caching(
        cacheable = @Cacheable("books"),
        evict = @CacheEvict(cacheNames = "temp", allEntries = true)
    )
    public Book findAndCleanup(ISBN isbn) {
        return bookRepository.findByIsbn(isbn);
    }
}
```

### 2.2 启用缓存支持

在 Spring Boot 应用中启用缓存非常简单，只需在配置类上添加 `@EnableCaching` 注解：

```java
@Configuration
@EnableCaching
public class CacheConfiguration {
    // 缓存配置
}
```


### 2.3 缓存键生成策略 (⚠️ 难点)

缓存的本质是键值存储，因此如何生成缓存键至关重要。Spring Cache 提供了灵活的键生成策略。

#### 默认键生成规则

Spring Cache 使用 `SimpleKeyGenerator` 作为默认的键生成器：

- **无参数**：返回 `SimpleKey.EMPTY`
- **单个参数**：直接使用该参数作为键
- **多个参数**：返回包含所有参数的 `SimpleKey`

```java
@Service
public class UserService {
    
    // 键为 userId
    @Cacheable("users")
    public User findUser(Long userId) {
        return userRepository.findById(userId);
    }
    
    // 键为 SimpleKey[userId, includeDetails]
    @Cacheable("users")
    public User findUser(Long userId, boolean includeDetails) {
        return userRepository.findById(userId);
    }
}
```

#### 自定义键表达式（SpEL）

使用 Spring Expression Language (SpEL) 可以灵活定义缓存键：

```java
@Service
public class BookService {
    
    /**
     * 使用方法参数作为键
     */
    @Cacheable(cacheNames = "books", key = "#isbn")
    public Book findBook(ISBN isbn, boolean checkWarehouse, boolean includeUsed) {
        return bookRepository.findByIsbn(isbn);
    }
    
    /**
     * 使用参数的属性作为键
     */
    @Cacheable(cacheNames = "books", key = "#isbn.rawNumber")
    public Book findBookByRawNumber(ISBN isbn, boolean checkWarehouse) {
        return bookRepository.findByIsbn(isbn);
    }
    
    /**
     * 使用静态方法生成键
     */
    @Cacheable(cacheNames = "books", key = "T(com.example.util.CacheKeyUtil).hash(#isbn)")
    public Book findBookWithCustomKey(ISBN isbn) {
        return bookRepository.findByIsbn(isbn);
    }
    
    /**
     * 组合多个参数生成键
     */
    @Cacheable(cacheNames = "books", key = "#isbn + '-' + #language")
    public Book findBookByLanguage(ISBN isbn, String language) {
        return bookRepository.findByIsbnAndLanguage(isbn, language);
    }
}
```

#### 自定义 KeyGenerator

对于复杂的键生成逻辑，可以实现自定义的 `KeyGenerator`：

```java
@Component
public class CustomKeyGenerator implements KeyGenerator {
    
    @Override
    public Object generate(Object target, Method method, Object... params) {
        StringBuilder key = new StringBuilder();
        key.append(target.getClass().getSimpleName()).append(".");
        key.append(method.getName()).append(":");
        
        for (Object param : params) {
            key.append(param.toString()).append(",");
        }
        
        return key.toString();
    }
}

// 使用自定义 KeyGenerator
@Service
public class ProductService {
    
    @Cacheable(cacheNames = "products", keyGenerator = "customKeyGenerator")
    public Product findProduct(Long productId, String region) {
        return productRepository.findByIdAndRegion(productId, region);
    }
}
```


### 2.4 条件缓存 🔥

Spring Cache 支持基于条件的缓存控制，提供了 `condition` 和 `unless` 两个属性。

#### condition - 方法执行前判断

`condition` 属性在方法执行前评估，决定是否使用缓存：

```java
@Service
public class BookService {
    
    /**
     * 只缓存书名长度小于 32 的查询结果
     */
    @Cacheable(cacheNames = "books", condition = "#name.length() < 32")
    public Book findBook(String name) {
        return bookRepository.findByName(name);
    }
    
    /**
     * 只有当 useCache 为 true 时才使用缓存
     */
    @Cacheable(cacheNames = "books", condition = "#useCache == true")
    public Book findBook(ISBN isbn, boolean useCache) {
        return bookRepository.findByIsbn(isbn);
    }
}
```

#### unless - 方法执行后判断

`unless` 属性在方法执行后评估，决定是否将结果放入缓存：

```java
@Service
public class BookService {
    
    /**
     * 不缓存精装书（hardback）
     */
    @Cacheable(cacheNames = "books", unless = "#result.hardback")
    public Book findBook(String name) {
        return bookRepository.findByName(name);
    }
    
    /**
     * 不缓存 null 结果
     */
    @Cacheable(cacheNames = "books", unless = "#result == null")
    public Book findBook(ISBN isbn) {
        return bookRepository.findByIsbn(isbn);
    }
    
    /**
     * 组合使用 condition 和 unless
     * 只缓存书名长度小于 32 且不是精装书的结果
     */
    @Cacheable(
        cacheNames = "books",
        condition = "#name.length() < 32",
        unless = "#result.hardback"
    )
    public Book findBookWithConditions(String name) {
        return bookRepository.findByName(name);
    }
}
```

### 2.5 同步缓存 (⚠️ 难点)

在高并发场景下，多个线程可能同时请求同一个未缓存的数据，导致缓存击穿问题。Spring Cache 提供了同步缓存机制来解决这个问题。

```java
@Service
public class ExpensiveService {
    
    /**
     * 使用 sync = true 启用同步缓存
     * 在多线程环境下，只有一个线程会执行方法，其他线程等待结果
     * 注意：不是所有缓存提供者都支持同步模式
     */
    @Cacheable(cacheNames = "foos", sync = true)
    public Foo executeExpensiveOperation(String id) {
        // 执行耗时操作
        return performExpensiveComputation(id);
    }
    
    /**
     * 支持 CompletableFuture 的异步缓存
     */
    @Cacheable(cacheNames = "asyncFoos", sync = true)
    public CompletableFuture<Foo> executeAsyncOperation(String id) {
        return CompletableFuture.supplyAsync(() -> 
            performExpensiveComputation(id)
        );
    }
    
    private Foo performExpensiveComputation(String id) {
        // 模拟耗时操作
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return new Foo(id);
    }
}
```

**注意事项**：
- `sync = true` 不能与 `unless` 一起使用
- 不是所有缓存提供者都支持同步模式（如 ConcurrentMapCacheManager 支持，但某些分布式缓存可能不支持）
- 同步缓存会影响性能，应根据实际场景权衡使用


## 💻 实战应用

### 3.1 配置不同的缓存提供者

Spring Cache 支持多种缓存提供者，可以根据应用需求选择合适的实现。

#### 3.1.1 ConcurrentMapCacheManager（简单场景）

基于 JDK `ConcurrentHashMap` 的缓存实现，适合测试和简单应用：

```java
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.concurrent.ConcurrentMapCache;
import org.springframework.cache.support.SimpleCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Arrays;

@Configuration
@EnableCaching
public class ConcurrentMapCacheConfig {

    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        cacheManager.setCaches(Arrays.asList(
            new ConcurrentMapCache("default"),
            new ConcurrentMapCache("books"),
            new ConcurrentMapCache("users")
        ));
        return cacheManager;
    }
}
```

**特点**：
- ✅ 配置简单，无需外部依赖
- ✅ 适合单机应用和测试环境
- ❌ 不支持过期时间
- ❌ 不支持持久化
- ❌ 不支持分布式

#### 3.1.2 Caffeine（高性能本地缓存）

Caffeine 是 Java 8+ 的高性能缓存库，推荐用于生产环境的本地缓存：

**添加依赖**：
```xml
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>
```

**按需创建缓存**：
```java
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableCaching
public class CaffeineCacheOnDemandConfig {

    @Bean
    public CacheManager cacheManager() {
        // 缓存会在首次使用时自动创建
        return new CaffeineCacheManager();
    }
}
```

**显式定义缓存并配置属性**：
```java
import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.caffeine.CaffeineCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Arrays;
import java.util.concurrent.TimeUnit;

@Configuration
@EnableCaching
public class CustomCaffeineCacheConfig {

    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        
        // 配置 Caffeine 属性
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .initialCapacity(100)           // 初始容量
            .maximumSize(500)               // 最大条目数
            .expireAfterAccess(10, TimeUnit.MINUTES)  // 访问后过期时间
            .recordStats());                // 启用统计
        
        // 显式定义缓存名称
        cacheManager.setCacheNames(Arrays.asList("users", "products", "orders"));
        
        return cacheManager;
    }
}
```


**不同缓存使用不同配置**：
```java
import com.github.benmanes.caffeine.cache.Caffeine;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.cache.caffeine.CaffeineCache;
import org.springframework.cache.support.SimpleCacheManager;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Arrays;
import java.util.concurrent.TimeUnit;

@Configuration
@EnableCaching
public class MultiCaffeineCacheConfig {

    @Bean
    public CacheManager cacheManager() {
        SimpleCacheManager cacheManager = new SimpleCacheManager();
        
        cacheManager.setCaches(Arrays.asList(
            // 用户缓存：5分钟过期，最多1000条
            buildCache("users", 5, 1000),
            
            // 产品缓存：30分钟过期，最多5000条
            buildCache("products", 30, 5000),
            
            // 订单缓存：10分钟过期，最多2000条
            buildCache("orders", 10, 2000)
        ));
        
        return cacheManager;
    }
    
    private CaffeineCache buildCache(String name, int minutesToExpire, int maxSize) {
        return new CaffeineCache(name, Caffeine.newBuilder()
            .expireAfterWrite(minutesToExpire, TimeUnit.MINUTES)
            .maximumSize(maxSize)
            .build());
    }
}
```

**特点**：
- ✅ 高性能，优于 Guava Cache
- ✅ 支持多种过期策略（写入后过期、访问后过期）
- ✅ 支持自动加载和刷新
- ✅ 支持统计信息
- ❌ 仅支持本地缓存，不支持分布式

#### 3.1.3 Redis（分布式缓存）

Redis 是最常用的分布式缓存解决方案：

**添加依赖**：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**配置 Redis 缓存**：
```java
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@Configuration
@EnableCaching
public class RedisCacheConfig {

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        // 默认缓存配置
        RedisCacheConfiguration defaultConfig = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30))  // 默认过期时间 30 分钟
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new StringRedisSerializer()))
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()))
            .disableCachingNullValues();  // 不缓存 null 值
        
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(defaultConfig)
            .build();
    }
}
```


**不同缓存使用不同 TTL**：
```java
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.serializer.GenericJackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;
import java.util.HashMap;
import java.util.Map;

@Configuration
@EnableCaching
public class MultiRedisCacheConfig {

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        // 默认配置
        RedisCacheConfiguration defaultConfig = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30))
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new StringRedisSerializer()))
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()))
            .disableCachingNullValues();
        
        // 为不同缓存配置不同的 TTL
        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();
        
        // 用户缓存：5 分钟
        cacheConfigurations.put("users", 
            defaultConfig.entryTtl(Duration.ofMinutes(5)));
        
        // 产品缓存：1 小时
        cacheConfigurations.put("products", 
            defaultConfig.entryTtl(Duration.ofHours(1)));
        
        // 配置缓存：24 小时
        cacheConfigurations.put("configs", 
            defaultConfig.entryTtl(Duration.ofHours(24)));
        
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(defaultConfig)
            .withInitialCacheConfigurations(cacheConfigurations)
            .build();
    }
}
```

**application.yml 配置**：
```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: your_password
    database: 0
    lettuce:
      pool:
        max-active: 8
        max-idle: 8
        min-idle: 0
        max-wait: -1ms
  cache:
    type: redis
    redis:
      time-to-live: 1800000  # 默认过期时间（毫秒）
      cache-null-values: false  # 不缓存 null 值
```

**特点**：
- ✅ 支持分布式部署
- ✅ 支持数据持久化
- ✅ 支持丰富的数据结构
- ✅ 高性能
- ❌ 需要额外的 Redis 服务器
- ❌ 网络延迟相对本地缓存更高

### 3.2 完整示例：用户服务缓存

下面是一个完整的用户服务示例，展示了 Spring Cache 的各种用法：

```java
import lombok.extern.slf4j.Slf4j;
import org.springframework.cache.annotation.*;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * 用户服务
 * 
 * @author erik.zhou
 */
@Slf4j
@Service
@CacheConfig(cacheNames = "users")  // 类级别的缓存配置
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    /**
     * 根据 ID 查询用户
     * 使用默认的键生成策略（userId）
     */
    @Cacheable
    public User findById(Long userId) {
        log.info("从数据库查询用户: {}", userId);
        return userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException(userId));
    }
    
    /**
     * 根据用户名查询用户
     * 自定义缓存键
     */
    @Cacheable(key = "'username:' + #username")
    public User findByUsername(String username) {
        log.info("从数据库查询用户: {}", username);
        return userRepository.findByUsername(username)
            .orElseThrow(() -> new UserNotFoundException(username));
    }
    
    /**
     * 查询所有用户
     * 只缓存用户数量小于 100 的结果
     */
    @Cacheable(key = "'all'", unless = "#result.size() > 100")
    public List<User> findAll() {
        log.info("从数据库查询所有用户");
        return userRepository.findAll();
    }
    
    /**
     * 创建用户
     * 创建后清除 "all" 缓存
     */
    @CacheEvict(key = "'all'")
    public User create(User user) {
        log.info("创建用户: {}", user.getUsername());
        return userRepository.save(user);
    }
    
    /**
     * 更新用户
     * 更新缓存中的用户信息
     */
    @CachePut(key = "#user.id")
    public User update(User user) {
        log.info("更新用户: {}", user.getId());
        return userRepository.save(user);
    }
    
    /**
     * 删除用户
     * 同时清除多个相关缓存
     */
    @Caching(evict = {
        @CacheEvict(key = "#userId"),
        @CacheEvict(key = "'username:' + #username"),
        @CacheEvict(key = "'all'")
    })
    public void delete(Long userId, String username) {
        log.info("删除用户: {}", userId);
        userRepository.deleteById(userId);
    }
    
    /**
     * 批量导入用户
     * 清空整个用户缓存
     */
    @CacheEvict(allEntries = true)
    public void batchImport(List<User> users) {
        log.info("批量导入 {} 个用户", users.size());
        userRepository.saveAll(users);
    }
    
    /**
     * 查询活跃用户
     * 只有当 useCache 为 true 时才使用缓存
     */
    @Cacheable(key = "'active'", condition = "#useCache == true")
    public List<User> findActiveUsers(boolean useCache) {
        log.info("从数据库查询活跃用户");
        return userRepository.findByStatus("ACTIVE");
    }
}
```


### 3.3 Spring Boot 自动配置

Spring Boot 提供了开箱即用的缓存自动配置，只需添加依赖和 `@EnableCaching` 注解即可。

**application.yml 配置**：
```yaml
spring:
  cache:
    # 缓存类型：simple, caffeine, redis, ehcache 等
    type: caffeine
    
    # Caffeine 配置
    caffeine:
      spec: maximumSize=500,expireAfterAccess=600s
    
    # 缓存名称
    cache-names:
      - users
      - products
      - orders
```

**使用 Spring Boot Starter**：
```xml
<!-- Caffeine -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
</dependency>

<!-- Redis -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

## ✨ 最佳实践

### 4.1 缓存设计原则

#### 1. 合理选择缓存粒度

```java
// ❌ 不推荐：缓存整个列表
@Cacheable("users")
public List<User> findAll() {
    return userRepository.findAll();
}

// ✅ 推荐：缓存单个对象
@Cacheable(cacheNames = "users", key = "#userId")
public User findById(Long userId) {
    return userRepository.findById(userId).orElse(null);
}
```

#### 2. 设置合理的过期时间

```java
// 根据数据特性设置不同的过期时间
// 用户信息：5-10 分钟
// 配置信息：1-24 小时
// 统计数据：1-5 分钟
// 热点数据：可以更长
```

#### 3. 避免缓存穿透

```java
@Service
public class ProductService {
    
    /**
     * 缓存空结果，避免缓存穿透
     * 但要设置较短的过期时间
     */
    @Cacheable(cacheNames = "products", key = "#productId")
    public Product findById(Long productId) {
        Product product = productRepository.findById(productId).orElse(null);
        // 即使是 null 也会被缓存（如果配置允许）
        return product;
    }
}
```

#### 4. 处理缓存雪崩

```java
// 为不同的缓存设置不同的过期时间，避免同时失效
@Configuration
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        Map<String, RedisCacheConfiguration> configs = new HashMap<>();
        
        // 添加随机偏移量，避免同时过期
        configs.put("users", createConfig(Duration.ofMinutes(5 + random(2))));
        configs.put("products", createConfig(Duration.ofMinutes(10 + random(3))));
        configs.put("orders", createConfig(Duration.ofMinutes(15 + random(5))));
        
        return RedisCacheManager.builder(factory)
            .withInitialCacheConfigurations(configs)
            .build();
    }
    
    private int random(int bound) {
        return ThreadLocalRandom.current().nextInt(bound);
    }
}
```


### 4.2 性能优化

#### 1. 使用 @CacheConfig 减少重复配置

```java
// ❌ 不推荐：每个方法都重复配置
@Service
public class UserService {
    
    @Cacheable(cacheNames = "users", keyGenerator = "customKeyGenerator")
    public User findById(Long id) { ... }
    
    @CachePut(cacheNames = "users", keyGenerator = "customKeyGenerator")
    public User update(User user) { ... }
}

// ✅ 推荐：使用类级别配置
@Service
@CacheConfig(cacheNames = "users", keyGenerator = "customKeyGenerator")
public class UserService {
    
    @Cacheable
    public User findById(Long id) { ... }
    
    @CachePut(key = "#user.id")
    public User update(User user) { ... }
}
```

#### 2. 避免缓存大对象

```java
// ❌ 不推荐：缓存包含大量数据的对象
@Cacheable("users")
public User findUserWithAllData(Long userId) {
    User user = userRepository.findById(userId);
    user.setOrders(orderRepository.findByUserId(userId));  // 可能有大量订单
    user.setComments(commentRepository.findByUserId(userId));  // 可能有大量评论
    return user;
}

// ✅ 推荐：只缓存必要的数据
@Cacheable("users")
public User findUserBasicInfo(Long userId) {
    return userRepository.findById(userId);
}

@Cacheable("userOrders")
public List<Order> findUserOrders(Long userId) {
    return orderRepository.findByUserId(userId);
}
```

#### 3. 合理使用 sync 属性

```java
@Service
public class HotDataService {
    
    /**
     * 对于热点数据，使用 sync = true 避免缓存击穿
     */
    @Cacheable(cacheNames = "hotProducts", key = "#productId", sync = true)
    public Product getHotProduct(Long productId) {
        return productRepository.findById(productId);
    }
    
    /**
     * 对于普通数据，不需要使用 sync
     */
    @Cacheable(cacheNames = "normalProducts", key = "#productId")
    public Product getNormalProduct(Long productId) {
        return productRepository.findById(productId);
    }
}
```

### 4.3 常见陷阱

#### ⚠️ 陷阱 1：同一个类内部调用不生效

```java
// ❌ 错误：内部调用不会触发缓存
@Service
public class UserService {
    
    public User getUserInfo(Long userId) {
        // 直接调用本类方法，缓存不生效
        return this.findById(userId);
    }
    
    @Cacheable("users")
    public User findById(Long userId) {
        return userRepository.findById(userId);
    }
}

// ✅ 解决方案 1：通过注入自己来调用
@Service
public class UserService {
    
    @Autowired
    private UserService self;
    
    public User getUserInfo(Long userId) {
        // 通过代理对象调用，缓存生效
        return self.findById(userId);
    }
    
    @Cacheable("users")
    public User findById(Long userId) {
        return userRepository.findById(userId);
    }
}

// ✅ 解决方案 2：拆分到不同的类
@Service
public class UserQueryService {
    
    @Cacheable("users")
    public User findById(Long userId) {
        return userRepository.findById(userId);
    }
}

@Service
public class UserService {
    
    @Autowired
    private UserQueryService userQueryService;
    
    public User getUserInfo(Long userId) {
        return userQueryService.findById(userId);
    }
}
```


#### ⚠️ 陷阱 2：@CachePut 和 @Cacheable 同时使用

```java
// ❌ 错误：不要在同一个方法上同时使用
@Cacheable("users")
@CachePut("users")
public User findById(Long userId) {
    return userRepository.findById(userId);
}

// ✅ 正确：分开使用
@Cacheable("users")
public User findById(Long userId) {
    return userRepository.findById(userId);
}

@CachePut(cacheNames = "users", key = "#user.id")
public User update(User user) {
    return userRepository.save(user);
}
```

#### ⚠️ 陷阱 3：缓存键冲突

```java
// ❌ 错误：不同方法使用相同的键可能导致类型不匹配
@Service
public class DataService {
    
    @Cacheable(cacheNames = "data", key = "#id")
    public User findUser(Long id) {
        return userRepository.findById(id);
    }
    
    @Cacheable(cacheNames = "data", key = "#id")
    public Product findProduct(Long id) {
        return productRepository.findById(id);  // 可能返回 User 对象！
    }
}

// ✅ 正确：使用不同的缓存名称或添加前缀
@Service
public class DataService {
    
    @Cacheable(cacheNames = "users", key = "#id")
    public User findUser(Long id) {
        return userRepository.findById(id);
    }
    
    @Cacheable(cacheNames = "products", key = "#id")
    public Product findProduct(Long id) {
        return productRepository.findById(id);
    }
}
```

#### ⚠️ 陷阱 4：忘记清除缓存

```java
// ❌ 错误：更新数据后忘记清除缓存
@Service
public class UserService {
    
    @Cacheable("users")
    public User findById(Long userId) {
        return userRepository.findById(userId);
    }
    
    // 更新后没有清除缓存，导致读取到旧数据
    public User updateStatus(Long userId, String status) {
        User user = userRepository.findById(userId);
        user.setStatus(status);
        return userRepository.save(user);
    }
}

// ✅ 正确：更新后清除或更新缓存
@Service
public class UserService {
    
    @Cacheable("users")
    public User findById(Long userId) {
        return userRepository.findById(userId);
    }
    
    @CachePut(cacheNames = "users", key = "#userId")
    public User updateStatus(Long userId, String status) {
        User user = userRepository.findById(userId);
        user.setStatus(status);
        return userRepository.save(user);
    }
}
```

### 4.4 监控与调试

#### 1. 启用缓存统计（Caffeine）

```java
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager cacheManager = new CaffeineCacheManager();
        cacheManager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .recordStats());  // 启用统计
        return cacheManager;
    }
    
    /**
     * 获取缓存统计信息
     */
    @Bean
    public CacheStatsService cacheStatsService(CacheManager cacheManager) {
        return new CacheStatsService(cacheManager);
    }
}

@Service
public class CacheStatsService {
    
    private final CacheManager cacheManager;
    
    public CacheStatsService(CacheManager cacheManager) {
        this.cacheManager = cacheManager;
    }
    
    public void printStats() {
        cacheManager.getCacheNames().forEach(cacheName -> {
            Cache cache = cacheManager.getCache(cacheName);
            if (cache instanceof CaffeineCache) {
                com.github.benmanes.caffeine.cache.Cache<Object, Object> nativeCache = 
                    ((CaffeineCache) cache).getNativeCache();
                CacheStats stats = nativeCache.stats();
                
                System.out.println("缓存名称: " + cacheName);
                System.out.println("命中率: " + stats.hitRate());
                System.out.println("命中次数: " + stats.hitCount());
                System.out.println("未命中次数: " + stats.missCount());
                System.out.println("加载成功次数: " + stats.loadSuccessCount());
                System.out.println("驱逐次数: " + stats.evictionCount());
            }
        });
    }
}
```


#### 2. 日志调试

```yaml
# application.yml
logging:
  level:
    org.springframework.cache: DEBUG
    org.springframework.data.redis: DEBUG
```

```java
import lombok.extern.slf4j.Slf4j;

@Slf4j
@Service
public class UserService {
    
    @Cacheable("users")
    public User findById(Long userId) {
        log.debug("缓存未命中，从数据库查询用户: {}", userId);
        return userRepository.findById(userId);
    }
}
```

## ❓ 常见问题

### Q1: 如何选择合适的缓存提供者？

**A**: 根据应用场景选择：

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| 单机应用 | Caffeine | 高性能，配置简单 |
| 分布式应用 | Redis | 支持多实例共享缓存 |
| 测试环境 | ConcurrentMap | 无需外部依赖 |
| 大数据量 | Redis + Caffeine | 两级缓存，兼顾性能和容量 |

### Q2: 缓存什么时候会失效？

**A**: 缓存失效的几种情况：

1. **主动清除**：使用 `@CacheEvict` 注解
2. **过期时间到**：达到配置的 TTL
3. **容量限制**：达到最大容量后，根据淘汰策略移除
4. **应用重启**：本地缓存会丢失（Redis 等持久化缓存不会）

### Q3: 如何处理缓存和数据库的一致性问题？

**A**: 常见策略：

```java
@Service
public class UserService {
    
    /**
     * 策略 1：先更新数据库，再删除缓存（推荐）
     */
    @Transactional
    @CacheEvict(cacheNames = "users", key = "#user.id")
    public User update(User user) {
        return userRepository.save(user);
    }
    
    /**
     * 策略 2：先删除缓存，再更新数据库
     * 可能存在短暂的不一致
     */
    @Transactional
    public User updateWithPreEvict(User user) {
        cacheManager.getCache("users").evict(user.getId());
        return userRepository.save(user);
    }
    
    /**
     * 策略 3：使用 @CachePut 更新缓存
     * 适合更新操作返回完整对象的场景
     */
    @CachePut(cacheNames = "users", key = "#user.id")
    public User updateAndRefresh(User user) {
        return userRepository.save(user);
    }
}
```

### Q4: 如何实现两级缓存（本地 + Redis）？

**A**: 自定义 CacheManager 实现两级缓存：

```java
@Configuration
@EnableCaching
public class TwoLevelCacheConfig {
    
    @Bean
    public CacheManager cacheManager(
            RedisConnectionFactory redisConnectionFactory) {
        
        // 一级缓存：Caffeine（本地）
        CaffeineCacheManager caffeineCacheManager = new CaffeineCacheManager();
        caffeineCacheManager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(1000)
            .expireAfterWrite(5, TimeUnit.MINUTES));
        
        // 二级缓存：Redis（分布式）
        RedisCacheManager redisCacheManager = RedisCacheManager
            .builder(redisConnectionFactory)
            .cacheDefaults(RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofMinutes(30)))
            .build();
        
        // 组合两级缓存
        return new CompositeCacheManager(
            caffeineCacheManager,
            redisCacheManager
        );
    }
}
```

### Q5: 如何在运行时动态清除缓存？

**A**: 注入 CacheManager 手动操作缓存：

```java
@Service
public class CacheManagementService {
    
    private final CacheManager cacheManager;
    
    public CacheManagementService(CacheManager cacheManager) {
        this.cacheManager = cacheManager;
    }
    
    /**
     * 清除指定缓存的指定键
     */
    public void evict(String cacheName, Object key) {
        Cache cache = cacheManager.getCache(cacheName);
        if (cache != null) {
            cache.evict(key);
        }
    }
    
    /**
     * 清空指定缓存的所有数据
     */
    public void clear(String cacheName) {
        Cache cache = cacheManager.getCache(cacheName);
        if (cache != null) {
            cache.clear();
        }
    }
    
    /**
     * 清空所有缓存
     */
    public void clearAll() {
        cacheManager.getCacheNames().forEach(cacheName -> {
            Cache cache = cacheManager.getCache(cacheName);
            if (cache != null) {
                cache.clear();
            }
        });
    }
    
    /**
     * 获取缓存值
     */
    public Object get(String cacheName, Object key) {
        Cache cache = cacheManager.getCache(cacheName);
        if (cache != null) {
            Cache.ValueWrapper wrapper = cache.get(key);
            return wrapper != null ? wrapper.get() : null;
        }
        return null;
    }
}
```


### Q6: SpEL 表达式中可以使用哪些变量？

**A**: Spring Cache 支持以下 SpEL 变量：

| 变量名 | 描述 | 示例 |
|--------|------|------|
| #root.method | 当前方法 | #root.method.name |
| #root.target | 目标对象 | #root.target.class.simpleName |
| #root.caches | 当前方法使用的缓存 | #root.caches[0].name |
| #root.methodName | 方法名 | #root.methodName |
| #root.targetClass | 目标类 | #root.targetClass |
| #root.args | 方法参数数组 | #root.args[0] |
| #参数名 | 方法参数 | #userId, #user.name |
| #result | 方法返回值（仅 unless） | #result.id |
| #p0, #p1... | 参数索引 | #p0, #p1 |
| #a0, #a1... | 参数索引（别名） | #a0, #a1 |

```java
@Service
public class ExampleService {
    
    @Cacheable(
        cacheNames = "examples",
        key = "#root.targetClass.simpleName + ':' + #root.methodName + ':' + #id"
    )
    public Example findById(Long id) {
        return repository.findById(id);
    }
    
    @Cacheable(
        cacheNames = "examples",
        key = "#user.id + '-' + #user.type",
        unless = "#result == null || #result.isEmpty()"
    )
    public List<Example> findByUser(User user) {
        return repository.findByUser(user);
    }
}
```

## 🔗 相关资源

### 官方文档
- [Spring Framework Cache Abstraction](https://docs.spring.io/spring-framework/reference/integration/cache.html)
- [Spring Boot Caching](https://docs.spring.io/spring-boot/docs/current/reference/html/io.html#io.caching)
- [Caffeine GitHub](https://github.com/ben-manes/caffeine)
- [Spring Data Redis](https://spring.io/projects/spring-data-redis)

### 推荐文章
- [Spring Cache 实战指南](https://spring.io/guides/gs/caching/)
- [缓存设计模式与最佳实践](https://martinfowler.com/bliki/TwoHardThings.html)
- [分布式缓存一致性问题](https://redis.io/docs/manual/patterns/distributed-locks/)

### 相关技术
- [Redis 完整教程](../04-数据库/02-Redis-完整教程.md)
- [Spring Boot 完整教程](./02-Spring-Boot-完整教程.md)
- [Spring AOP 完整教程](./01-Spring-Framework-完整教程.md#aop)

## 📝 学习检查清单

### 基础知识
- [ ] 理解 Spring Cache 抽象层的设计理念
- [ ] 掌握 Cache 和 CacheManager 的概念
- [ ] 了解缓存的应用场景

### 核心注解
- [ ] 掌握 @Cacheable 的使用方法
- [ ] 掌握 @CachePut 的使用方法
- [ ] 掌握 @CacheEvict 的使用方法
- [ ] 掌握 @Caching 的使用方法
- [ ] 理解 @EnableCaching 的作用

### 高级特性
- [ ] 掌握缓存键生成策略（默认、SpEL、自定义）
- [ ] 理解条件缓存（condition 和 unless）
- [ ] 掌握同步缓存（sync）的使用场景
- [ ] 了解不同缓存提供者的配置方法

### 实战应用
- [ ] 能够配置 ConcurrentMap 缓存
- [ ] 能够配置 Caffeine 缓存
- [ ] 能够配置 Redis 缓存
- [ ] 能够为不同缓存设置不同的过期时间
- [ ] 能够实现完整的 CRUD 缓存操作

### 最佳实践
- [ ] 理解缓存设计原则
- [ ] 掌握性能优化技巧
- [ ] 了解常见陷阱及解决方案
- [ ] 能够进行缓存监控和调试
- [ ] 理解缓存一致性问题及解决方案

### 进阶内容
- [ ] 了解两级缓存的实现方式
- [ ] 掌握动态缓存管理
- [ ] 理解缓存穿透、击穿、雪崩问题
- [ ] 能够根据业务场景选择合适的缓存方案

---

**文档信息**：
- **版本**: 1.0
- **最后更新**: 2024-12-31
- **文档来源**: Spring Framework 官方文档 (Context7)
- **作者**: @author erik.zhou

