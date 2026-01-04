# Spring WebFlux 完整教程

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
- **官方文档**: https://docs.spring.io/spring-framework/reference/web/webflux.html
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Java 基础（Lambda、Stream API）
  - Spring MVC 基础
  - 响应式编程基本概念
  - HTTP 协议基础
- **文档来源**: Spring Framework 官方文档 (Context7)
- **更新时间**: 2024-12-31
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解响应式编程的核心思想和优势
- [ ] 掌握 Reactor 框架的 Mono 和 Flux 使用
- [ ] 理解背压（Backpressure）机制
- [ ] 掌握 WebFlux 的两种编程模型
- [ ] 能够构建响应式 Web 应用
- [ ] 理解 WebFlux 与 Spring MVC 的区别
- [ ] 掌握响应式错误处理和调试技巧

## 📖 基础概念

### 1.1 什么是 Spring WebFlux

Spring WebFlux 是 Spring Framework 5.0 引入的响应式 Web 框架，它提供了一种全新的非阻塞、事件驱动的编程模型。

**核心特点**：
- **非阻塞 I/O**: 基于 Netty、Undertow 等非阻塞服务器
- **响应式流**: 支持 Reactive Streams 规范
- **函数式编程**: 提供函数式端点编程模型
- **背压支持**: 自动处理生产者和消费者速度不匹配问题

### 1.2 核心概念

#### 响应式编程（Reactive Programming）
响应式编程是一种面向数据流和变化传播的编程范式。在响应式编程中，数据流是第一公民，程序通过订阅数据流来响应变化。

**关键特征**：
- **异步非阻塞**: 不会阻塞线程等待结果
- **数据流驱动**: 以数据流的方式处理数据
- **声明式**: 描述"做什么"而非"怎么做"
- **背压机制**: 消费者可以控制生产速度

#### Reactive Streams 规范
Reactive Streams 是一个提供非阻塞背压的异步流处理标准，包含四个核心接口：
- **Publisher**: 数据发布者
- **Subscriber**: 数据订阅者
- **Subscription**: 订阅关系
- **Processor**: 既是发布者又是订阅者

#### Reactor 框架
Reactor 是 Spring WebFlux 的核心响应式库，提供了两个核心类型：
- **Mono**: 表示 0 或 1 个元素的异步序列
- **Flux**: 表示 0 到 N 个元素的异步序列

### 1.3 应用场景

**适合使用 WebFlux 的场景**：
- 高并发场景（如实时消息推送、聊天应用）
- I/O 密集型应用（大量数据库、网络调用）
- 微服务架构中的服务间通信
- 流式数据处理（如日志处理、数据分析）
- 需要背压控制的场景

**不适合使用 WebFlux 的场景**：
- CPU 密集型计算
- 阻塞式数据库操作（如 JDBC）
- 团队对响应式编程不熟悉
- 简单的 CRUD 应用

### 1.4 WebFlux vs Spring MVC

| 特性 | Spring MVC | Spring WebFlux |
|------|-----------|----------------|
| 编程模型 | 同步阻塞 | 异步非阻塞 |
| 服务器 | Servlet 容器（Tomcat） | Netty、Undertow |
| 并发模型 | 每请求一线程 | 事件循环 |
| 数据库支持 | JDBC、JPA | R2DBC、MongoDB Reactive |
| 学习曲线 | 平缓 | 陡峭 |
| 调试难度 | 简单 | 困难 ⚠️ |
| 适用场景 | 通用 Web 应用 | 高并发、I/O 密集型 |

## 🔥 核心特性 (重点)

### 2.1 Reactor 核心类型 🔥

#### Mono - 0 或 1 个元素

```java
// 创建 Mono
Mono<String> mono1 = Mono.just("Hello");
Mono<String> mono2 = Mono.empty();
Mono<String> mono3 = Mono.error(new RuntimeException("Error"));

// 从 Callable 创建
Mono<String> mono4 = Mono.fromCallable(() -> {
    // 执行某些操作
    return "Result";
});

// 延迟创建
Mono<String> mono5 = Mono.defer(() -> Mono.just("Deferred"));

// 常用操作符
Mono<String> result = Mono.just("hello")
    .map(String::toUpperCase)           // 转换
    .filter(s -> s.length() > 3)        // 过滤
    .defaultIfEmpty("default")          // 默认值
    .flatMap(s -> Mono.just(s + "!"))  // 扁平化映射
    .doOnNext(System.out::println)      // 副作用
    .doOnError(e -> System.err.println("Error: " + e))
    .onErrorReturn("fallback");         // 错误处理
```

#### Flux - 0 到 N 个元素

```java
// 创建 Flux
Flux<String> flux1 = Flux.just("A", "B", "C");
Flux<Integer> flux2 = Flux.range(1, 10);
Flux<Long> flux3 = Flux.interval(Duration.ofSeconds(1));
Flux<String> flux4 = Flux.fromIterable(Arrays.asList("X", "Y", "Z"));

// 常用操作符
Flux<String> result = Flux.just("apple", "banana", "cherry")
    .map(String::toUpperCase)           // 转换每个元素
    .filter(s -> s.startsWith("A"))     // 过滤
    .flatMap(s -> Flux.just(s.split(""))) // 扁平化
    .distinct()                          // 去重
    .take(5)                            // 取前5个
    .skip(2)                            // 跳过前2个
    .collectList()                      // 收集为 List
    .flux();                            // 转回 Flux
```

### 2.2 背压机制 (⚠️ 难点)

背压（Backpressure）是响应式流的核心机制，用于处理生产者和消费者速度不匹配的问题。

**背压策略**：

```java
// 1. BUFFER - 缓冲所有元素（可能导致内存溢出）
Flux.range(1, 1000)
    .onBackpressureBuffer(100)  // 缓冲最多100个元素
    .subscribe(System.out::println);

// 2. DROP - 丢弃无法处理的元素
Flux.range(1, 1000)
    .onBackpressureDrop(dropped -> 
        System.out.println("Dropped: " + dropped))
    .subscribe(System.out::println);

// 3. LATEST - 只保留最新的元素
Flux.range(1, 1000)
    .onBackpressureLatest()
    .subscribe(System.out::println);

// 4. ERROR - 抛出异常
Flux.range(1, 1000)
    .onBackpressureError()
    .subscribe(System.out::println);
```

**请求控制**：

```java
Flux.range(1, 100)
    .subscribe(new BaseSubscriber<Integer>() {
        @Override
        protected void hookOnSubscribe(Subscription subscription) {
            request(10);  // 初始请求10个元素
        }

        @Override
        protected void hookOnNext(Integer value) {
            System.out.println("Received: " + value);
            if (value % 10 == 0) {
                request(10);  // 每处理10个，再请求10个
            }
        }
    });
```

### 2.3 两种编程模型 🔥

#### 注解式控制器（Annotated Controllers）

与 Spring MVC 类似，但支持响应式类型：

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    private final UserRepository userRepository;

    public UserController(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    // 返回单个对象
    @GetMapping("/{id}")
    public Mono<User> getUser(@PathVariable String id) {
        return userRepository.findById(id);
    }

    // 返回集合
    @GetMapping
    public Flux<User> getAllUsers() {
        return userRepository.findAll();
    }

    // 接收响应式请求体
    @PostMapping
    public Mono<User> createUser(@RequestBody Mono<User> userMono) {
        return userMono.flatMap(userRepository::save);
    }

    // 流式响应
    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<User> streamUsers() {
        return userRepository.findAll()
            .delayElements(Duration.ofSeconds(1));
    }

    // 参数校验
    @PostMapping("/validate")
    public Mono<User> createUserWithValidation(
            @Valid @RequestBody Mono<User> userMono) {
        return userMono
            .flatMap(userRepository::save)
            .onErrorResume(ex -> {
                // 处理校验错误
                return Mono.error(new ValidationException(ex.getMessage()));
            });
    }
}
```

#### 函数式端点（Functional Endpoints）

更加轻量级的函数式编程模型：

```java
// Handler 类
@Component
public class UserHandler {

    private final UserRepository repository;

    public UserHandler(UserRepository repository) {
        this.repository = repository;
    }

    public Mono<ServerResponse> listUsers(ServerRequest request) {
        Flux<User> users = repository.findAll();
        return ServerResponse.ok()
            .contentType(MediaType.APPLICATION_JSON)
            .body(users, User.class);
    }

    public Mono<ServerResponse> getUser(ServerRequest request) {
        String userId = request.pathVariable("id");
        return repository.findById(userId)
            .flatMap(user -> ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(user))
            .switchIfEmpty(ServerResponse.notFound().build());
    }

    public Mono<ServerResponse> createUser(ServerRequest request) {
        Mono<User> userMono = request.bodyToMono(User.class);
        return userMono
            .flatMap(repository::save)
            .flatMap(user -> ServerResponse.ok()
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(user));
    }

    public Mono<ServerResponse> deleteUser(ServerRequest request) {
        String userId = request.pathVariable("id");
        return repository.deleteById(userId)
            .then(ServerResponse.noContent().build());
    }
}

// Router 配置
@Configuration
public class UserRouter {

    @Bean
    public RouterFunction<ServerResponse> userRoutes(UserHandler handler) {
        return RouterFunctions
            .route(GET("/api/users"), handler::listUsers)
            .andRoute(GET("/api/users/{id}"), handler::getUser)
            .andRoute(POST("/api/users"), handler::createUser)
            .andRoute(DELETE("/api/users/{id}"), handler::deleteUser);
    }
}
```

### 2.4 响应式数据访问

#### R2DBC（Reactive Relational Database Connectivity）

```java
// 依赖配置（pom.xml）
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-postgresql</artifactId>
</dependency>

// Repository 接口
public interface UserRepository extends ReactiveCrudRepository<User, String> {
    
    Flux<User> findByName(String name);
    
    Mono<User> findByEmail(String email);
    
    @Query("SELECT * FROM users WHERE age > :age")
    Flux<User> findByAgeGreaterThan(int age);
}

// 使用示例
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public Mono<User> createUser(User user) {
        return userRepository.save(user);
    }
    
    public Flux<User> findUsersByName(String name) {
        return userRepository.findByName(name);
    }
    
    public Mono<Void> deleteUser(String id) {
        return userRepository.deleteById(id);
    }
}
```

#### MongoDB Reactive

```java
// 依赖配置
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb-reactive</artifactId>
</dependency>

// Repository
public interface ProductRepository extends ReactiveMongoRepository<Product, String> {
    
    Flux<Product> findByCategory(String category);
    
    Flux<Product> findByPriceBetween(double minPrice, double maxPrice);
}

// 使用示例
@Service
public class ProductService {
    
    private final ProductRepository productRepository;
    
    public Flux<Product> searchProducts(String category, double minPrice, double maxPrice) {
        return productRepository.findByCategory(category)
            .filter(p -> p.getPrice() >= minPrice && p.getPrice() <= maxPrice);
    }
}
```

### 2.5 错误处理 (⚠️ 难点)

响应式编程中的错误处理与传统方式不同，需要使用特定的操作符：

```java
// 1. onErrorReturn - 返回默认值
Mono<String> result = Mono.error(new RuntimeException("Error"))
    .onErrorReturn("Default Value");

// 2. onErrorResume - 切换到备用流
Mono<User> user = userRepository.findById(id)
    .onErrorResume(ex -> {
        logger.error("Failed to find user", ex);
        return Mono.just(new User("default"));
    });

// 3. onErrorMap - 转换错误类型
Mono<User> user = userRepository.findById(id)
    .onErrorMap(ex -> new UserNotFoundException("User not found: " + id));

// 4. doOnError - 副作用处理（不改变流）
Mono<User> user = userRepository.findById(id)
    .doOnError(ex -> logger.error("Error occurred", ex));

// 5. retry - 重试
Mono<User> user = userRepository.findById(id)
    .retry(3)  // 重试3次
    .retryWhen(Retry.backoff(3, Duration.ofSeconds(1)));  // 指数退避重试

// 6. timeout - 超时处理
Mono<User> user = userRepository.findById(id)
    .timeout(Duration.ofSeconds(5))
    .onErrorReturn(new User("timeout"));
```

**全局异常处理**：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public Mono<ResponseEntity<ErrorResponse>> handleUserNotFound(
            UserNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage()
        );
        return Mono.just(ResponseEntity.status(HttpStatus.NOT_FOUND).body(error));
    }

    @ExceptionHandler(ValidationException.class)
    public Mono<ResponseEntity<ErrorResponse>> handleValidation(
            ValidationException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            ex.getMessage()
        );
        return Mono.just(ResponseEntity.badRequest().body(error));
    }

    @ExceptionHandler(Exception.class)
    public Mono<ResponseEntity<ErrorResponse>> handleGeneral(Exception ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "Internal server error"
        );
        return Mono.just(ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(error));
    }
}
```

### 2.6 常用操作符总结 🔥

#### 转换操作符
```java
// map - 一对一转换
Flux.just(1, 2, 3).map(i -> i * 2);  // 2, 4, 6

// flatMap - 一对多转换（扁平化）
Flux.just(1, 2, 3).flatMap(i -> Flux.just(i, i * 2));  // 1, 2, 2, 4, 3, 6

// flatMapSequential - 保持顺序的 flatMap
Flux.just(1, 2, 3).flatMapSequential(i -> Flux.just(i, i * 2));

// concatMap - 串行执行的 flatMap
Flux.just(1, 2, 3).concatMap(i -> Flux.just(i, i * 2));
```

#### 过滤操作符
```java
// filter - 过滤
Flux.range(1, 10).filter(i -> i % 2 == 0);  // 2, 4, 6, 8, 10

// take - 取前N个
Flux.range(1, 10).take(5);  // 1, 2, 3, 4, 5

// skip - 跳过前N个
Flux.range(1, 10).skip(5);  // 6, 7, 8, 9, 10

// distinct - 去重
Flux.just(1, 2, 2, 3, 3, 3).distinct();  // 1, 2, 3
```

#### 组合操作符
```java
// merge - 合并多个流（交错）
Flux<Integer> flux1 = Flux.just(1, 3, 5);
Flux<Integer> flux2 = Flux.just(2, 4, 6);
Flux.merge(flux1, flux2);  // 1, 2, 3, 4, 5, 6（顺序不确定）

// concat - 连接多个流（顺序）
Flux.concat(flux1, flux2);  // 1, 3, 5, 2, 4, 6

// zip - 配对组合
Flux.zip(flux1, flux2, (a, b) -> a + b);  // 3, 7, 11
```

#### 聚合操作符
```java
// collectList - 收集为 List
Flux.just(1, 2, 3).collectList();  // Mono<List<Integer>>

// collectMap - 收集为 Map
Flux.just("a", "bb", "ccc")
    .collectMap(String::length, s -> s);  // Mono<Map<Integer, String>>

// reduce - 归约
Flux.range(1, 5).reduce((a, b) -> a + b);  // Mono<15>

// count - 计数
Flux.range(1, 10).count();  // Mono<10>
```

## 💻 实战应用

### 3.1 环境搭建

#### Maven 依赖配置

```xml
<dependencies>
    <!-- Spring Boot WebFlux Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    
    <!-- R2DBC 支持 -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-r2dbc</artifactId>
    </dependency>
    
    <!-- PostgreSQL R2DBC 驱动 -->
    <dependency>
        <groupId>io.r2dbc</groupId>
        <artifactId>r2dbc-postgresql</artifactId>
    </dependency>
    
    <!-- Reactor 测试支持 -->
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

#### application.yml 配置

```yaml
spring:
  r2dbc:
    url: r2dbc:postgresql://localhost:5432/testdb
    username: postgres
    password: password
  
  webflux:
    # 基础路径
    base-path: /api
    
server:
  port: 8080
  
logging:
  level:
    org.springframework.r2dbc: DEBUG
    io.r2dbc: DEBUG
```

### 3.2 快速开始 - RESTful API

#### 实体类

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Table("users")
public class User {
    
    @Id
    private String id;
    
    private String name;
    
    private String email;
    
    private Integer age;
    
    @CreatedDate
    private LocalDateTime createdAt;
    
    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

#### Repository

```java
public interface UserRepository extends ReactiveCrudRepository<User, String> {
    
    Flux<User> findByName(String name);
    
    Mono<User> findByEmail(String email);
    
    Flux<User> findByAgeGreaterThan(Integer age);
}
```

#### Service

```java
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public Flux<User> getAllUsers() {
        return userRepository.findAll();
    }
    
    public Mono<User> getUserById(String id) {
        return userRepository.findById(id)
            .switchIfEmpty(Mono.error(
                new UserNotFoundException("User not found: " + id)));
    }
    
    public Mono<User> createUser(User user) {
        user.setId(UUID.randomUUID().toString());
        return userRepository.save(user);
    }
    
    public Mono<User> updateUser(String id, User user) {
        return userRepository.findById(id)
            .flatMap(existingUser -> {
                existingUser.setName(user.getName());
                existingUser.setEmail(user.getEmail());
                existingUser.setAge(user.getAge());
                return userRepository.save(existingUser);
            })
            .switchIfEmpty(Mono.error(
                new UserNotFoundException("User not found: " + id)));
    }
    
    public Mono<Void> deleteUser(String id) {
        return userRepository.deleteById(id);
    }
}
```

#### Controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public Flux<User> getAllUsers() {
        return userService.getAllUsers();
    }
    
    @GetMapping("/{id}")
    public Mono<User> getUserById(@PathVariable String id) {
        return userService.getUserById(id);
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<User> createUser(@Valid @RequestBody User user) {
        return userService.createUser(user);
    }
    
    @PutMapping("/{id}")
    public Mono<User> updateUser(
            @PathVariable String id,
            @Valid @RequestBody User user) {
        return userService.updateUser(id, user);
    }
    
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public Mono<Void> deleteUser(@PathVariable String id) {
        return userService.deleteUser(id);
    }
}
```

### 3.3 进阶案例

#### 案例1：服务器推送事件（SSE）

```java
@RestController
@RequestMapping("/api/events")
public class EventController {
    
    // 实时推送服务器时间
    @GetMapping(value = "/time", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<String> streamTime() {
        return Flux.interval(Duration.ofSeconds(1))
            .map(seq -> "Current time: " + LocalDateTime.now());
    }
    
    // 实时推送用户创建事件
    @GetMapping(value = "/users", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<User> streamUsers() {
        return userRepository.findAll()
            .delayElements(Duration.ofMillis(500));
    }
}
```

#### 案例2：WebClient 响应式 HTTP 客户端

```java
@Service
public class ExternalApiService {
    
    private final WebClient webClient;
    
    public ExternalApiService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder
            .baseUrl("https://api.example.com")
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();
    }
    
    // GET 请求
    public Mono<User> getUser(String id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .bodyToMono(User.class)
            .timeout(Duration.ofSeconds(5))
            .retry(3);
    }
    
    // POST 请求
    public Mono<User> createUser(User user) {
        return webClient.post()
            .uri("/users")
            .bodyValue(user)
            .retrieve()
            .bodyToMono(User.class);
    }
    
    // 并行调用多个 API
    public Mono<UserProfile> getUserProfile(String userId) {
        Mono<User> userMono = getUser(userId);
        Mono<List<Order>> ordersMono = getUserOrders(userId);
        Mono<Address> addressMono = getUserAddress(userId);
        
        return Mono.zip(userMono, ordersMono, addressMono)
            .map(tuple -> new UserProfile(
                tuple.getT1(),
                tuple.getT2(),
                tuple.getT3()
            ));
    }
    
    // 错误处理
    public Mono<User> getUserWithFallback(String id) {
        return webClient.get()
            .uri("/users/{id}", id)
            .retrieve()
            .onStatus(HttpStatusCode::is4xxClientError, 
                response -> Mono.error(new UserNotFoundException("User not found")))
            .onStatus(HttpStatusCode::is5xxServerError,
                response -> Mono.error(new ServiceException("Service unavailable")))
            .bodyToMono(User.class)
            .onErrorResume(ex -> {
                logger.error("Failed to get user", ex);
                return Mono.just(getDefaultUser());
            });
    }
}
```

#### 案例3：响应式缓存

```java
@Service
public class CachedUserService {
    
    private final UserRepository userRepository;
    private final Map<String, User> cache = new ConcurrentHashMap<>();
    
    public Mono<User> getUserWithCache(String id) {
        // 先查缓存
        User cachedUser = cache.get(id);
        if (cachedUser != null) {
            return Mono.just(cachedUser);
        }
        
        // 缓存未命中，查数据库
        return userRepository.findById(id)
            .doOnNext(user -> cache.put(id, user))  // 更新缓存
            .switchIfEmpty(Mono.error(
                new UserNotFoundException("User not found: " + id)));
    }
    
    public Mono<User> updateUserAndInvalidateCache(String id, User user) {
        return userRepository.findById(id)
            .flatMap(existingUser -> {
                existingUser.setName(user.getName());
                existingUser.setEmail(user.getEmail());
                return userRepository.save(existingUser);
            })
            .doOnNext(updatedUser -> cache.remove(id));  // 清除缓存
    }
}
```

#### 案例4：响应式事务

```java
@Service
public class TransactionalService {
    
    private final UserRepository userRepository;
    private final OrderRepository orderRepository;
    private final ReactiveTransactionManager transactionManager;
    
    @Transactional
    public Mono<Order> createOrderForUser(String userId, Order order) {
        return userRepository.findById(userId)
            .switchIfEmpty(Mono.error(
                new UserNotFoundException("User not found")))
            .flatMap(user -> {
                order.setUserId(userId);
                return orderRepository.save(order);
            });
    }
    
    // 手动事务控制
    public Mono<Void> transferBalance(String fromUserId, String toUserId, 
                                      BigDecimal amount) {
        TransactionalOperator operator = TransactionalOperator.create(
            transactionManager);
        
        return userRepository.findById(fromUserId)
            .zipWith(userRepository.findById(toUserId))
            .flatMap(tuple -> {
                User fromUser = tuple.getT1();
                User toUser = tuple.getT2();
                
                if (fromUser.getBalance().compareTo(amount) < 0) {
                    return Mono.error(new InsufficientBalanceException());
                }
                
                fromUser.setBalance(fromUser.getBalance().subtract(amount));
                toUser.setBalance(toUser.getBalance().add(amount));
                
                return userRepository.save(fromUser)
                    .then(userRepository.save(toUser))
                    .then();
            })
            .as(operator::transactional);
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

#### 1. 合理使用操作符

```java
// ❌ 不推荐：多次订阅
Mono<User> userMono = userRepository.findById(id);
userMono.subscribe(user -> doSomething1(user));
userMono.subscribe(user -> doSomething2(user));  // 会触发两次数据库查询

// ✅ 推荐：使用 cache()
Mono<User> userMono = userRepository.findById(id).cache();
userMono.subscribe(user -> doSomething1(user));
userMono.subscribe(user -> doSomething2(user));  // 只查询一次
```

#### 2. 避免阻塞操作

```java
// ❌ 不推荐：在响应式流中使用阻塞操作
Mono<User> user = Mono.fromCallable(() -> {
    Thread.sleep(1000);  // 阻塞操作
    return new User();
});

// ✅ 推荐：使用 subscribeOn 指定调度器
Mono<User> user = Mono.fromCallable(() -> {
    Thread.sleep(1000);
    return new User();
}).subscribeOn(Schedulers.boundedElastic());  // 在弹性线程池执行
```

#### 3. 批量操作优化

```java
// ❌ 不推荐：逐个保存
public Flux<User> saveUsers(List<User> users) {
    return Flux.fromIterable(users)
        .flatMap(userRepository::save);  // N次数据库操作
}

// ✅ 推荐：批量保存
public Flux<User> saveUsers(List<User> users) {
    return userRepository.saveAll(users);  // 1次批量操作
}
```

#### 4. 控制并发度

```java
// 限制并发请求数
Flux.range(1, 1000)
    .flatMap(i -> callExternalApi(i), 10)  // 最多10个并发请求
    .subscribe();
```

### 4.2 调试技巧 (⚠️ 难点)

响应式编程的调试比传统编程困难，需要掌握特殊技巧：

#### 1. 使用 log() 操作符

```java
Flux.range(1, 5)
    .map(i -> i * 2)
    .log()  // 打印所有信号
    .subscribe();
```

#### 2. 使用 checkpoint()

```java
Flux.range(1, 5)
    .map(i -> i * 2)
    .checkpoint("After map")  // 添加检查点
    .filter(i -> i > 5)
    .checkpoint("After filter")
    .subscribe();
```

#### 3. 使用 Hooks 全局调试

```java
@SpringBootApplication
public class Application {
    
    public static void main(String[] args) {
        // 启用调试模式（生产环境禁用）
        Hooks.onOperatorDebug();
        
        SpringApplication.run(Application.class, args);
    }
}
```

#### 4. 使用 StepVerifier 测试

```java
@Test
public void testUserService() {
    User user = new User("1", "John", "john@example.com", 30);
    
    when(userRepository.findById("1")).thenReturn(Mono.just(user));
    
    StepVerifier.create(userService.getUserById("1"))
        .expectNext(user)
        .verifyComplete();
}

@Test
public void testErrorHandling() {
    when(userRepository.findById("999"))
        .thenReturn(Mono.empty());
    
    StepVerifier.create(userService.getUserById("999"))
        .expectError(UserNotFoundException.class)
        .verify();
}
```

### 4.3 常见陷阱 ⚠️

#### 陷阱1：忘记订阅

```java
// ❌ 错误：没有订阅，代码不会执行
userRepository.save(user);

// ✅ 正确：必须订阅才会执行
userRepository.save(user).subscribe();

// ✅ 更好：在 Controller 中返回，由框架自动订阅
@PostMapping
public Mono<User> createUser(@RequestBody User user) {
    return userRepository.save(user);  // 框架会自动订阅
}
```

#### 陷阱2：阻塞操作

```java
// ❌ 错误：在响应式流中使用 block()
@GetMapping("/{id}")
public User getUser(@PathVariable String id) {
    return userRepository.findById(id).block();  // 阻塞！
}

// ✅ 正确：返回响应式类型
@GetMapping("/{id}")
public Mono<User> getUser(@PathVariable String id) {
    return userRepository.findById(id);
}
```

#### 陷阱3：错误的错误处理

```java
// ❌ 错误：try-catch 无法捕获响应式流中的异常
try {
    userRepository.findById(id).subscribe();
} catch (Exception e) {
    // 捕获不到异常
}

// ✅ 正确：使用响应式错误处理操作符
userRepository.findById(id)
    .onErrorResume(ex -> {
        logger.error("Error", ex);
        return Mono.empty();
    })
    .subscribe();
```

#### 陷阱4：不当的 flatMap 使用

```java
// ❌ 错误：嵌套 flatMap 导致回调地狱
userRepository.findById(userId)
    .flatMap(user -> 
        orderRepository.findByUserId(userId)
            .flatMap(orders -> 
                addressRepository.findByUserId(userId)
                    .map(address -> new UserProfile(user, orders, address))
            )
    );

// ✅ 正确：使用 zip 组合
Mono.zip(
    userRepository.findById(userId),
    orderRepository.findByUserId(userId).collectList(),
    addressRepository.findByUserId(userId)
).map(tuple -> new UserProfile(
    tuple.getT1(),
    tuple.getT2(),
    tuple.getT3()
));
```

#### 陷阱5：内存泄漏

```java
// ❌ 错误：无限流没有取消订阅
Flux.interval(Duration.ofSeconds(1))
    .subscribe(i -> System.out.println(i));  // 永远不会停止

// ✅ 正确：保存 Disposable 并在适当时候取消
Disposable subscription = Flux.interval(Duration.ofSeconds(1))
    .subscribe(i -> System.out.println(i));

// 在需要时取消订阅
subscription.dispose();
```

### 4.4 编码规范

#### 1. 命名规范

```java
// 响应式方法返回 Mono/Flux，建议不加特殊前缀
public Mono<User> getUser(String id) { ... }        // ✅ 推荐
public Mono<User> getUserMono(String id) { ... }    // ❌ 不推荐

// 变量命名清晰表达意图
Mono<User> userMono = ...;      // ✅ 推荐
Flux<Order> orders = ...;       // ✅ 推荐
```

#### 2. 链式调用格式

```java
// ✅ 推荐：每个操作符单独一行
return userRepository.findById(id)
    .map(User::getName)
    .filter(name -> name.length() > 3)
    .defaultIfEmpty("Unknown")
    .doOnNext(logger::info);

// ❌ 不推荐：所有操作符在一行
return userRepository.findById(id).map(User::getName).filter(name -> name.length() > 3).defaultIfEmpty("Unknown");
```

#### 3. 错误处理位置

```java
// ✅ 推荐：在合适的位置处理错误
return userRepository.findById(id)
    .switchIfEmpty(Mono.error(new UserNotFoundException()))
    .flatMap(user -> orderRepository.findByUserId(user.getId()))
    .onErrorResume(ex -> {
        logger.error("Failed to get orders", ex);
        return Flux.empty();
    });
```

## ❓ 常见问题

### Q1: 什么时候应该使用 WebFlux 而不是 Spring MVC？

**A**: 考虑以下因素：
- **高并发场景**：需要处理大量并发连接（如实时消息、聊天应用）
- **I/O 密集型**：大量数据库、网络调用
- **流式数据**：需要处理流式数据或服务器推送
- **微服务通信**：服务间需要响应式通信

**不适合使用 WebFlux**：
- 简单的 CRUD 应用
- 团队不熟悉响应式编程
- 需要使用阻塞式数据库（JDBC）
- CPU 密集型计算

### Q2: Mono 和 Flux 有什么区别？

**A**: 
- **Mono**: 表示 0 或 1 个元素的异步序列，类似于 `Optional` 或 `Future`
- **Flux**: 表示 0 到 N 个元素的异步序列，类似于 `Stream` 或 `List`

使用场景：
- 单个对象查询 → Mono
- 集合查询 → Flux
- 删除操作 → Mono<Void>
- 流式数据 → Flux

### Q3: 如何在 WebFlux 中使用传统的 JDBC 数据库？

**A**: 不推荐在 WebFlux 中使用 JDBC，因为 JDBC 是阻塞式的。建议：
1. 使用 R2DBC（响应式关系数据库连接）
2. 使用响应式 NoSQL 数据库（MongoDB Reactive）
3. 如果必须使用 JDBC，在单独的线程池中执行：

```java
Mono<User> user = Mono.fromCallable(() -> {
    // JDBC 阻塞操作
    return jdbcTemplate.queryForObject(...);
}).subscribeOn(Schedulers.boundedElastic());
```

### Q4: 如何调试响应式代码？ (⚠️ 难点)

**A**: 响应式代码调试确实困难，建议：
1. 使用 `log()` 操作符查看信号流
2. 使用 `checkpoint()` 添加检查点
3. 启用 `Hooks.onOperatorDebug()`（仅开发环境）
4. 使用 `StepVerifier` 编写测试
5. 使用 IDE 的响应式调试插件

### Q5: WebFlux 的性能真的比 Spring MVC 好吗？

**A**: 不一定。性能取决于具体场景：
- **高并发 I/O 密集型**：WebFlux 更好（更少的线程，更高的吞吐量）
- **CPU 密集型**：差异不大，甚至可能更差
- **低并发场景**：Spring MVC 可能更简单高效

性能测试建议：
- 在实际场景下进行压测
- 关注吞吐量、延迟、资源占用
- 考虑团队学习成本

### Q6: 如何处理响应式流中的异常？

**A**: 使用响应式错误处理操作符：
- `onErrorReturn()`: 返回默认值
- `onErrorResume()`: 切换到备用流
- `onErrorMap()`: 转换异常类型
- `retry()`: 重试
- `timeout()`: 超时处理

避免使用 try-catch，它无法捕获响应式流中的异常。

### Q7: 背压（Backpressure）是什么？为什么重要？ (⚠️ 难点)

**A**: 背压是响应式流的核心机制，用于处理生产者和消费者速度不匹配的问题。

**场景示例**：
- 生产者每秒产生 1000 条数据
- 消费者每秒只能处理 100 条数据
- 没有背压：内存溢出
- 有背压：消费者告诉生产者"慢点发"

**背压策略**：
- `BUFFER`: 缓冲数据（可能内存溢出）
- `DROP`: 丢弃数据
- `LATEST`: 只保留最新数据
- `ERROR`: 抛出异常

### Q8: 如何在 WebFlux 中实现文件上传和下载？

**A**: 

**文件上传**：
```java
@PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public Mono<String> uploadFile(@RequestPart("file") FilePart filePart) {
    String filename = filePart.filename();
    Path path = Paths.get("uploads", filename);
    
    return filePart.transferTo(path)
        .then(Mono.just("File uploaded: " + filename));
}
```

**文件下载**：
```java
@GetMapping("/download/{filename}")
public Mono<ResponseEntity<Resource>> downloadFile(@PathVariable String filename) {
    Path path = Paths.get("uploads", filename);
    Resource resource = new FileSystemResource(path);
    
    return Mono.just(ResponseEntity.ok()
        .header(HttpHeaders.CONTENT_DISPOSITION, 
            "attachment; filename=\"" + filename + "\"")
        .body(resource));
}
```

### Q9: WebFlux 支持 WebSocket 吗？

**A**: 支持。WebFlux 提供了完整的 WebSocket 支持：

```java
@Configuration
public class WebSocketConfig {
    
    @Bean
    public HandlerMapping webSocketMapping(WebSocketHandler handler) {
        Map<String, WebSocketHandler> map = new HashMap<>();
        map.put("/ws/chat", handler);
        
        SimpleUrlHandlerMapping mapping = new SimpleUrlHandlerMapping();
        mapping.setUrlMap(map);
        mapping.setOrder(1);
        return mapping;
    }
    
    @Bean
    public WebSocketHandler chatHandler() {
        return session -> {
            Flux<WebSocketMessage> output = session.receive()
                .map(msg -> "Echo: " + msg.getPayloadAsText())
                .map(session::textMessage);
            
            return session.send(output);
        };
    }
}
```

### Q10: 如何测试响应式代码？

**A**: 使用 `StepVerifier` 进行测试：

```java
@Test
public void testFlux() {
    Flux<Integer> flux = Flux.just(1, 2, 3, 4, 5);
    
    StepVerifier.create(flux)
        .expectNext(1)
        .expectNext(2)
        .expectNext(3)
        .expectNext(4)
        .expectNext(5)
        .verifyComplete();
}

@Test
public void testError() {
    Flux<Integer> flux = Flux.just(1, 2, 3)
        .concatWith(Flux.error(new RuntimeException("Error")));
    
    StepVerifier.create(flux)
        .expectNext(1, 2, 3)
        .expectError(RuntimeException.class)
        .verify();
}

@Test
public void testWithVirtualTime() {
    StepVerifier.withVirtualTime(() -> 
        Flux.interval(Duration.ofSeconds(1)).take(3))
        .expectSubscription()
        .thenAwait(Duration.ofSeconds(3))
        .expectNext(0L, 1L, 2L)
        .verifyComplete();
}
```

## 🔗 相关资源

### 官方文档
- [Spring WebFlux 官方文档](https://docs.spring.io/spring-framework/reference/web/webflux.html)
- [Project Reactor 官方文档](https://projectreactor.io/docs)
- [R2DBC 官方文档](https://r2dbc.io/)
- [Reactive Streams 规范](https://www.reactive-streams.org/)

### 推荐书籍
- 《Spring 实战（第6版）》- WebFlux 章节
- 《Reactive Spring》- Josh Long
- 《Hands-On Reactive Programming in Spring 5》

### 推荐文章
- [Understanding Reactive Programming](https://spring.io/reactive)
- [Reactor 3 Reference Guide](https://projectreactor.io/docs/core/release/reference/)
- [Spring WebFlux vs Spring MVC](https://www.baeldung.com/spring-webflux-concurrency)

### 视频教程
- Spring 官方 YouTube 频道 - WebFlux 系列
- Baeldung WebFlux 教程系列

### 实战项目
- [Spring Petclinic Reactive](https://github.com/spring-petclinic/spring-petclinic-reactive)
- [Spring WebFlux Examples](https://github.com/spring-projects/spring-framework/tree/main/spring-webflux)

## 📝 学习检查清单

### 基础概念
- [ ] 理解响应式编程的核心思想
- [ ] 掌握 Reactive Streams 规范
- [ ] 理解 Mono 和 Flux 的区别
- [ ] 理解背压机制的原理和重要性
- [ ] 了解 WebFlux 与 Spring MVC 的区别

### 核心操作符
- [ ] 掌握转换操作符（map、flatMap、flatMapSequential）
- [ ] 掌握过滤操作符（filter、take、skip、distinct）
- [ ] 掌握组合操作符（merge、concat、zip）
- [ ] 掌握错误处理操作符（onErrorReturn、onErrorResume、retry）
- [ ] 掌握聚合操作符（collectList、reduce、count）

### 实战应用
- [ ] 能够创建响应式 RESTful API
- [ ] 掌握注解式控制器的使用
- [ ] 掌握函数式端点的使用
- [ ] 能够使用 R2DBC 进行数据库操作
- [ ] 能够使用 WebClient 进行 HTTP 调用
- [ ] 能够实现服务器推送事件（SSE）
- [ ] 能够处理文件上传和下载

### 最佳实践
- [ ] 掌握响应式代码的调试技巧
- [ ] 了解常见陷阱并能够避免
- [ ] 能够编写响应式单元测试
- [ ] 掌握性能优化技巧
- [ ] 遵循响应式编码规范

### 进阶主题
- [ ] 理解响应式事务处理
- [ ] 掌握 WebSocket 的使用
- [ ] 了解响应式安全配置
- [ ] 掌握响应式缓存策略
- [ ] 能够进行性能调优

---

## 🎓 学习建议

1. **循序渐进**：先掌握 Reactor 基础，再学习 WebFlux
2. **多写代码**：响应式编程需要大量实践才能掌握
3. **对比学习**：将 Spring MVC 代码改写为 WebFlux，加深理解
4. **阅读源码**：阅读 Reactor 和 WebFlux 源码，理解底层原理
5. **参与社区**：在 Stack Overflow、GitHub 上学习和交流
6. **性能测试**：在实际场景下进行性能测试，验证效果
7. **保持耐心**：响应式编程学习曲线陡峭，需要时间适应思维转变

## ⚠️ 重难点总结

### 🔥 重点内容
1. **Reactor 核心类型**：Mono 和 Flux 的使用
2. **两种编程模型**：注解式控制器和函数式端点
3. **常用操作符**：map、flatMap、filter、zip 等
4. **响应式数据访问**：R2DBC 和 MongoDB Reactive
5. **WebClient**：响应式 HTTP 客户端

### ⚠️ 难点内容
1. **响应式编程思维**：从命令式到声明式的思维转变
2. **背压机制**：理解和正确使用背压策略
3. **错误处理**：响应式流中的异常处理方式
4. **调试困难**：响应式代码的调试技巧
5. **性能调优**：合理使用操作符和调度器

---

**@author erik.zhou**  
**最后更新**: 2024-12-31  
**文档版本**: 1.0
