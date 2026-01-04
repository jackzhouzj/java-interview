# Spring Framework 完整教程

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
- **版本**: Spring Framework 6.1.x
- **官方文档**: https://spring.io/projects/spring-framework
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Java基础（JDK 17+）
  - 面向对象编程
  - Java注解
  - Maven/Gradle构建工具
- **文档来源**: Context7 + Spring官方文档
- **更新时间**: 2024-12-31
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解IoC（控制反转）和DI（依赖注入）的核心概念
- [ ] 掌握Spring Bean的生命周期管理
- [ ] 理解并应用AOP（面向切面编程）
- [ ] 掌握Spring事务管理机制
- [ ] 理解循环依赖的解决方案
- [ ] 能够使用Spring构建企业级应用

## 📖 基础概念

### 1.1 什么是Spring Framework

Spring Framework是一个全面的企业级Java应用开发框架，为开发Java应用提供基础设施支持。它专注于依赖注入、面向切面编程和声明式事务管理，通过IoC容器促进松耦合设计，并提供与各种企业技术（包括数据访问、消息传递、Web服务等）的集成。

Spring Framework是所有Spring项目的基础，旨在通过其控制反转（IoC）容器使Java企业开发更加简单。

### 1.2 核心概念

- **IoC（Inversion of Control）**: 控制反转，将对象的创建和依赖关系的管理交给Spring容器
- **DI（Dependency Injection）**: 依赖注入，IoC的一种实现方式，通过构造器、工厂方法或属性注入依赖
- **Bean**: Spring容器管理的对象
- **ApplicationContext**: Spring的IoC容器，负责实例化、配置和组装Bean
- **AOP（Aspect-Oriented Programming）**: 面向切面编程，用于分离横切关注点
- **Proxy**: 代理对象，Spring AOP通过JDK动态代理或CGLIB代理实现

### 1.3 应用场景

- 企业级Web应用开发
- 微服务架构的基础框架
- 数据访问层的事务管理
- 系统日志、安全、缓存等横切关注点的统一处理
- 复杂业务逻辑的解耦和组织

## 🔥 核心特性

### 2.1 IoC容器与依赖注入 🔥

#### 2.1.1 IoC容器基础

Spring的IoC容器是框架的核心技术。它提供了依赖管理和Bean生命周期管理的完整基础。依赖注入（DI）是IoC的一种特殊形式，对象仅通过构造器参数、工厂方法参数或在对象实例构造或从工厂方法返回后设置的属性来定义它们的依赖关系。

**基本配置示例**：

```java
@Configuration
public class AppConfig {
    
    @Bean
    public UserService userService() {
        return new UserServiceImpl(userRepository());
    }
    
    @Bean
    public UserRepository userRepository() {
        return new UserRepositoryImpl();
    }
}
```

#### 2.1.2 依赖注入方式

**构造器注入（推荐）**：

```java
@Service
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final InventoryService inventoryService;
    
    // 构造器注入 - 推荐方式
    public OrderService(OrderRepository orderRepository, 
                       InventoryService inventoryService) {
        this.orderRepository = orderRepository;
        this.inventoryService = inventoryService;
    }
}
```

**Setter注入**：

```java
@Service
public class UserService {
    
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

**字段注入（不推荐）**：

```java
@Service
public class ProductService {
    
    @Autowired
    private ProductRepository productRepository;  // 不推荐：难以测试
}
```

### 2.2 Bean生命周期 🔥 (⚠️ 难点)

#### 2.2.1 完整生命周期阶段

Spring Bean的生命周期包含以下关键阶段：

1. **实例化（Instantiation）**: 创建Bean实例
2. **属性赋值（Populate Properties）**: 注入依赖
3. **初始化前处理（BeanPostProcessor.postProcessBeforeInitialization）**
4. **初始化（Initialization）**: 执行初始化回调
5. **初始化后处理（BeanPostProcessor.postProcessAfterInitialization）**
6. **使用（In Use）**: Bean可以被使用
7. **销毁（Destruction）**: 容器关闭时执行销毁回调

#### 2.2.2 初始化和销毁回调

**使用JSR-250注解（推荐）**：

```java
public class CachingMovieLister {

    @PostConstruct
    public void populateMovieCache() {
        // 在Bean初始化后填充电影缓存
        System.out.println("初始化缓存...");
    }

    @PreDestroy
    public void clearMovieCache() {
        // 在Bean销毁前清理缓存
        System.out.println("清理缓存...");
    }
}
```

**使用@Bean注解的initMethod和destroyMethod**：

```java
public class BeanOne {
    
    public void init() {
        // 初始化逻辑
        System.out.println("BeanOne初始化");
    }
}

public class BeanTwo {
    
    public void cleanup() {
        // 销毁逻辑
        System.out.println("BeanTwo清理资源");
    }
}

@Configuration
public class AppConfig {

    @Bean(initMethod = "init")
    public BeanOne beanOne() {
        return new BeanOne();
    }

    @Bean(destroyMethod = "cleanup")
    public BeanTwo beanTwo() {
        return new BeanTwo();
    }
}
```

**实现InitializingBean和DisposableBean接口**：

```java
@Component
public class DatabaseConnection implements InitializingBean, DisposableBean {
    
    private Connection connection;
    
    @Override
    public void afterPropertiesSet() throws Exception {
        // Bean属性设置后执行
        connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/db");
        System.out.println("数据库连接已建立");
    }
    
    @Override
    public void destroy() throws Exception {
        // Bean销毁前执行
        if (connection != null && !connection.isClosed()) {
            connection.close();
            System.out.println("数据库连接已关闭");
        }
    }
}
```

### 2.3 循环依赖解决 (⚠️ 难点)

#### 2.3.1 什么是循环依赖

循环依赖是指两个或多个Bean相互依赖，形成一个闭环。例如：BeanA依赖BeanB，BeanB又依赖BeanA。

#### 2.3.2 Spring如何解决循环依赖

Spring通过**三级缓存**机制解决单例Bean的循环依赖问题：

1. **一级缓存（singletonObjects）**: 存放完全初始化好的单例Bean
2. **二级缓存（earlySingletonObjects）**: 存放早期暴露的单例Bean（已实例化但未完全初始化）
3. **三级缓存（singletonFactories）**: 存放单例Bean的工厂对象

**解决流程**：

```java
// 示例：A依赖B，B依赖A
@Service
public class ServiceA {
    @Autowired
    private ServiceB serviceB;
}

@Service
public class ServiceB {
    @Autowired
    private ServiceA serviceA;
}

// Spring解决步骤：
// 1. 创建A的实例，放入三级缓存
// 2. 为A注入依赖，发现需要B
// 3. 创建B的实例，放入三级缓存
// 4. 为B注入依赖，发现需要A
// 5. 从三级缓存获取A的早期引用，注入到B
// 6. B初始化完成，放入一级缓存
// 7. 将B注入到A
// 8. A初始化完成，放入一级缓存
```

**⚠️ 注意事项**：

- 构造器注入的循环依赖无法解决（会抛出BeanCurrentlyInCreationException）
- 原型（Prototype）作用域的循环依赖无法解决
- 建议通过重构代码避免循环依赖

### 2.4 AOP（面向切面编程）🔥

#### 2.4.1 AOP核心概念

- **Aspect（切面）**: 横切关注点的模块化，如日志、事务
- **Join Point（连接点）**: 程序执行的某个点，如方法调用
- **Pointcut（切点）**: 匹配连接点的表达式
- **Advice（通知）**: 在切点执行的动作
  - Before: 前置通知
  - After: 后置通知
  - AfterReturning: 返回后通知
  - AfterThrowing: 异常通知
  - Around: 环绕通知
- **Target Object（目标对象）**: 被代理的对象
- **AOP Proxy（AOP代理）**: 由AOP框架创建的对象，用于实现切面契约

#### 2.4.2 定义切面

```java
@Aspect
@Component
public class LoggingAspect {
    
    // 定义切点：匹配service包下所有类的所有方法
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}
    
    // 前置通知
    @Before("serviceLayer()")
    public void logBefore(JoinPoint joinPoint) {
        System.out.println("执行方法: " + joinPoint.getSignature().getName());
    }
    
    // 返回后通知
    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void logAfterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("方法返回: " + result);
    }
    
    // 异常通知
    @AfterThrowing(pointcut = "serviceLayer()", throwing = "error")
    public void logAfterThrowing(JoinPoint joinPoint, Throwable error) {
        System.out.println("方法异常: " + error.getMessage());
    }
    
    // 环绕通知
    @Around("serviceLayer()")
    public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        
        System.out.println("方法开始: " + joinPoint.getSignature().getName());
        Object result = joinPoint.proceed();  // 执行目标方法
        
        long endTime = System.currentTimeMillis();
        System.out.println("方法结束，耗时: " + (endTime - startTime) + "ms");
        
        return result;
    }
}
```

#### 2.4.3 代理机制 (⚠️ 难点)

Spring AOP使用JDK动态代理或CGLIB代理来创建目标对象的代理：

**JDK动态代理**：
- 目标对象实现了至少一个接口时使用
- 代理所有目标类型实现的接口
- 基于接口的代理

**CGLIB代理**：
- 目标对象没有实现任何接口时使用
- 在运行时生成目标类型的子类
- 基于继承的代理

```java
// JDK动态代理示例
public interface UserService {
    void addUser(User user);
}

@Service
public class UserServiceImpl implements UserService {
    @Override
    public void addUser(User user) {
        // 实现逻辑
    }
}
// Spring会使用JDK动态代理

// CGLIB代理示例
@Service
public class ProductService {  // 没有实现接口
    public void addProduct(Product product) {
        // 实现逻辑
    }
}
// Spring会使用CGLIB代理
```

**性能对比**：
- JDK动态代理和CGLIB代理的性能差异很小
- CGLIB已集成在spring-core中，无需额外依赖
- 选择代理策略不应以性能为主要考虑因素

### 2.5 事务管理 🔥

#### 2.5.1 启用事务管理

```java
@Configuration
@EnableTransactionManagement
public class TransactionConfig {
    
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }
}
```

#### 2.5.2 声明式事务

```java
@Service
public class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private InventoryRepository inventoryRepository;
    
    // 基本事务方法：成功自动提交，异常自动回滚
    @Transactional
    public Order createOrder(Long customerId, List<OrderItem> items) {
        Order order = new Order(customerId);

        for (OrderItem item : items) {
            // 减少库存
            inventoryRepository.decreaseStock(item.getProductId(), item.getQuantity());
            order.addItem(item);
        }

        orderRepository.save(order);
        return order;
    }
    
    // 只读事务：优化性能
    @Transactional(readOnly = true)
    public Order getOrder(Long orderId) {
        return orderRepository.findById(orderId);
    }
    
    // 指定回滚异常
    @Transactional(rollbackFor = Exception.class)
    public void processPayment(Long orderId) {
        // 处理支付逻辑
    }
    
    // 事务传播行为
    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void createAuditLog(String action) {
        // 总是在新事务中执行，不受外部事务影响
    }
}
```

#### 2.5.3 事务传播行为

| 传播行为 | 说明 |
|---------|------|
| REQUIRED（默认） | 如果当前存在事务，则加入该事务；如果不存在，则创建新事务 |
| REQUIRES_NEW | 创建新事务，如果当前存在事务，则挂起当前事务 |
| SUPPORTS | 如果当前存在事务，则加入该事务；如果不存在，则以非事务方式执行 |
| NOT_SUPPORTED | 以非事务方式执行，如果当前存在事务，则挂起当前事务 |
| MANDATORY | 如果当前存在事务，则加入该事务；如果不存在，则抛出异常 |
| NEVER | 以非事务方式执行，如果当前存在事务，则抛出异常 |
| NESTED | 如果当前存在事务，则在嵌套事务内执行；如果不存在，则创建新事务 |

## 💻 实战应用

### 3.1 环境搭建

**Maven依赖**：

```xml
<dependencies>
    <!-- Spring Context -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <version>6.1.3</version>
    </dependency>
    
    <!-- Spring AOP -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-aop</artifactId>
        <version>6.1.3</version>
    </dependency>
    
    <!-- AspectJ -->
    <dependency>
        <groupId>org.aspectj</groupId>
        <artifactId>aspectjweaver</artifactId>
        <version>1.9.21</version>
    </dependency>
    
    <!-- Spring JDBC（用于事务管理） -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-jdbc</artifactId>
        <version>6.1.3</version>
    </dependency>
</dependencies>
```

### 3.2 快速开始

**创建配置类**：

```java
@Configuration
@ComponentScan(basePackages = "com.example")
@EnableAspectJAutoProxy
public class AppConfig {
    
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/mydb");
        dataSource.setUsername("root");
        dataSource.setPassword("password");
        return dataSource;
    }
}
```

**启动应用**：

```java
public class Application {
    
    public static void main(String[] args) {
        // 创建Spring容器
        ApplicationContext context = 
            new AnnotationConfigApplicationContext(AppConfig.class);
        
        // 获取Bean
        UserService userService = context.getBean(UserService.class);
        
        // 使用Bean
        userService.addUser(new User("张三", "zhangsan@example.com"));
        
        // 关闭容器
        ((ConfigurableApplicationContext) context).close();
    }
}
```

### 3.3 进阶案例：构建完整的服务层

```java
// 实体类
@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 50)
    private String username;
    
    @Column(nullable = false, length = 100)
    private String email;
    
    // 构造器、getter、setter省略
}

// Repository层
@Repository
public class UserRepository {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public void save(User user) {
        String sql = "INSERT INTO users (username, email) VALUES (?, ?)";
        jdbcTemplate.update(sql, user.getUsername(), user.getEmail());
    }
    
    public User findById(Long id) {
        String sql = "SELECT * FROM users WHERE id = ?";
        return jdbcTemplate.queryForObject(sql, 
            (rs, rowNum) -> new User(
                rs.getLong("id"),
                rs.getString("username"),
                rs.getString("email")
            ), 
            id);
    }
}

// Service层
@Service
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Transactional
    public void registerUser(User user) {
        // 业务逻辑：注册用户
        userRepository.save(user);
        // 其他业务逻辑...
    }
    
    @Transactional(readOnly = true)
    public User getUserById(Long id) {
        return userRepository.findById(id);
    }
}

// 切面：记录方法执行时间
@Aspect
@Component
public class PerformanceAspect {
    
    private static final Logger logger = LoggerFactory.getLogger(PerformanceAspect.class);
    
    @Around("execution(* com.example.service.*.*(..))")
    public Object logExecutionTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long startTime = System.currentTimeMillis();
        
        Object result = joinPoint.proceed();
        
        long executionTime = System.currentTimeMillis() - startTime;
        logger.info("{} 执行耗时: {}ms", 
            joinPoint.getSignature().toShortString(), executionTime);
        
        return result;
    }
}
```

## ✨ 最佳实践

### 4.1 依赖注入最佳实践

1. **优先使用构造器注入**
   - 保证依赖不可变
   - 便于单元测试
   - 避免NullPointerException

```java
@Service
public class OrderService {
    
    private final OrderRepository orderRepository;
    private final PaymentService paymentService;
    
    // 推荐：构造器注入
    public OrderService(OrderRepository orderRepository, 
                       PaymentService paymentService) {
        this.orderRepository = orderRepository;
        this.paymentService = paymentService;
    }
}
```

2. **避免字段注入**
   - 难以进行单元测试
   - 隐藏了类的依赖关系
   - 可能导致循环依赖

3. **使用@Qualifier解决多个实现**

```java
@Service
public class NotificationService {
    
    private final MessageSender messageSender;
    
    public NotificationService(@Qualifier("emailSender") MessageSender messageSender) {
        this.messageSender = messageSender;
    }
}
```

### 4.2 Bean作用域选择

```java
// 单例（默认）：适用于无状态Bean
@Service
@Scope("singleton")
public class UserService { }

// 原型：每次请求创建新实例
@Component
@Scope("prototype")
public class ShoppingCart { }

// 请求作用域：Web应用中每个HTTP请求一个实例
@Component
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public class LoginAction { }
```

### 4.3 事务管理最佳实践

1. **事务方法应该尽可能小**
   - 减少锁定时间
   - 提高并发性能

2. **只读事务优化查询**

```java
@Transactional(readOnly = true)
public List<User> getAllUsers() {
    return userRepository.findAll();
}
```

3. **合理设置事务超时**

```java
@Transactional(timeout = 30)  // 30秒超时
public void longRunningOperation() {
    // 长时间运行的操作
}
```

4. **明确指定回滚异常**

```java
@Transactional(rollbackFor = Exception.class)
public void criticalOperation() {
    // 所有异常都回滚
}
```

### 4.4 AOP使用建议

1. **切点表达式要精确**

```java
// 不推荐：过于宽泛
@Pointcut("execution(* *.*(..))")

// 推荐：精确匹配
@Pointcut("execution(* com.example.service.*Service.*(..))")
```

2. **避免在切面中执行耗时操作**
3. **合理使用通知类型**
   - 简单日志：使用@Before或@After
   - 性能监控：使用@Around
   - 异常处理：使用@AfterThrowing

## ❓ 常见问题

### Q1: @Autowired和@Resource的区别？

**A**: 
- `@Autowired`是Spring提供的注解，默认按类型（byType）装配
- `@Resource`是JSR-250提供的注解，默认按名称（byName）装配
- 推荐使用`@Autowired`，因为它是Spring原生注解，功能更强大

### Q2: 如何避免循环依赖？

**A**: 
1. 重构代码，消除循环依赖（最佳方案）
2. 使用`@Lazy`注解延迟加载
3. 使用Setter注入代替构造器注入（不推荐）
4. 将共同依赖提取到第三个类

### Q3: 事务不生效的常见原因？

**A**: 
1. 方法不是public的
2. 同一个类内部方法调用（绕过了代理）
3. 异常被catch了没有抛出
4. 数据库引擎不支持事务（如MyISAM）
5. 没有配置事务管理器

### Q4: Spring AOP和AspectJ的区别？

**A**: 
- Spring AOP：基于代理，只支持方法级别的拦截，运行时织入
- AspectJ：基于字节码操作，支持字段、构造器等拦截，编译时或类加载时织入
- Spring AOP更简单，AspectJ功能更强大

### Q5: 如何选择JDK动态代理还是CGLIB代理？

**A**: 
- Spring会自动选择：有接口用JDK动态代理，无接口用CGLIB
- 可以通过`@EnableAspectJAutoProxy(proxyTargetClass = true)`强制使用CGLIB
- 两者性能差异不大，不应作为主要考虑因素

## 🔗 相关资源

### 官方文档
- [Spring Framework官方文档](https://docs.spring.io/spring-framework/reference/)
- [Spring Framework GitHub](https://github.com/spring-projects/spring-framework)
- [Spring Framework API文档](https://docs.spring.io/spring-framework/docs/current/javadoc-api/)

### 推荐文章
- [Spring IoC容器深入理解](https://spring.io/guides/gs/spring-boot/)
- [Spring AOP实战指南](https://www.baeldung.com/spring-aop)
- [Spring事务管理详解](https://www.baeldung.com/transaction-configuration-with-jpa-and-spring)

### 视频教程
- [Spring Framework核心技术](https://www.youtube.com/springdevelopers)
- [深入理解Spring IoC](https://www.bilibili.com/video/BV1...)

### 推荐书籍
- 《Spring实战（第6版）》
- 《Spring源码深度解析》
- 《精通Spring 4.x企业应用开发实战》

## 📝 学习检查清单

### 基础知识
- [ ] 理解IoC和DI的概念和区别
- [ ] 掌握Bean的配置方式（XML、注解、Java配置）
- [ ] 理解ApplicationContext和BeanFactory的区别
- [ ] 掌握依赖注入的三种方式

### 核心特性
- [ ] 理解Bean的完整生命周期
- [ ] 掌握Bean的作用域（singleton、prototype等）
- [ ] 理解循环依赖的产生和解决机制
- [ ] 掌握AOP的核心概念和使用
- [ ] 理解JDK动态代理和CGLIB代理的区别
- [ ] 掌握声明式事务管理
- [ ] 理解事务传播行为

### 实战能力
- [ ] 能够搭建Spring项目
- [ ] 能够使用Spring构建分层架构
- [ ] 能够编写自定义切面
- [ ] 能够处理事务问题
- [ ] 能够解决常见的Spring问题

### 最佳实践
- [ ] 掌握依赖注入的最佳实践
- [ ] 理解何时使用不同的Bean作用域
- [ ] 掌握事务管理的最佳实践
- [ ] 理解AOP的适用场景和限制

---

**学习建议**：
1. 先理解IoC和DI的核心思想
2. 通过实践掌握Bean的配置和生命周期
3. 深入学习AOP和事务管理
4. 阅读Spring源码加深理解
5. 在实际项目中应用所学知识

**预计学习时长**: 30-40小时（基础学习）+ 80-100小时（进阶学习）
