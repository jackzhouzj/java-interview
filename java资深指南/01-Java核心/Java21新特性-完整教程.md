# Java 21 新特性-完整教程

> @author erik.zhou  
> 难度: ⭐⭐⭐⭐  
> 版本: Java 21 LTS (2023.09)

## 📋 目录

- [Java 21概述](#java-21概述)
- [虚拟线程](#虚拟线程)
- [结构化并发](#结构化并发)
- [模式匹配增强](#模式匹配增强)
- [其他新特性](#其他新特性)

---

## 🎯 Java 21概述

### 为什么选择Java 21

```
Java 21是继Java 17之后的新LTS版本：

1. 长期支持（LTS）
   - 支持到2031年
   - 生产环境推荐
   
2. 革命性特性
   - Virtual Threads（虚拟线程）
   - Structured Concurrency（结构化并发）
   - Pattern Matching增强
   
3. 性能提升
   - 启动速度更快
   - 内存占用更少
   - GC性能优化
   
4. 开发体验
   - 代码更简洁
   - 类型推断增强
   - API改进
```

### Java版本演进

```
Java 8 (2014) → Java 11 (2018) → Java 17 (2021) → Java 21 (2023)
   LTS              LTS              LTS              LTS

重要版本：
- Java 8: Lambda、Stream API
- Java 11: 模块化、var关键字
- Java 17: Sealed Classes、Pattern Matching
- Java 21: Virtual Threads、Structured Concurrency
```

---

## 🚀 虚拟线程（Virtual Threads）

### 什么是虚拟线程

```
虚拟线程是轻量级线程，由JVM管理而不是操作系统：

传统线程（Platform Thread）：
- 1:1映射到OS线程
- 创建成本高（~1MB栈空间）
- 数量受限（几千个）
- 上下文切换开销大

虚拟线程（Virtual Thread）：
- M:N映射到OS线程
- 创建成本低（~1KB栈空间）
- 数量几乎无限（百万级）
- 上下文切换开销小
```

### 创建虚拟线程

```java
/**
 * 虚拟线程创建方式
 * @author erik.zhou
 */
public class VirtualThreadExample {
    
    /**
     * 方式1：Thread.ofVirtual()
     */
    public void createVirtualThread1() {
        Thread thread = Thread.ofVirtual().start(() -> {
            System.out.println("Hello from virtual thread!");
        });
    }
    
    /**
     * 方式2：Thread.startVirtualThread()
     */
    public void createVirtualThread2() {
        Thread.startVirtualThread(() -> {
            System.out.println("Hello from virtual thread!");
        });
    }
    
    /**
     * 方式3：Executors.newVirtualThreadPerTaskExecutor()
     */
    public void createVirtualThread3() {
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            executor.submit(() -> {
                System.out.println("Hello from virtual thread!");
            });
        }
    }
    
    /**
     * 对比：传统线程池
     */
    public void traditionalThreadPool() {
        ExecutorService executor = Executors.newFixedThreadPool(100);
        // 最多100个线程
    }
}
```

### 虚拟线程实战

```java
/**
 * 虚拟线程实战案例
 * @author erik.zhou
 */
@Service
public class OrderService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    /**
     * 传统方式：处理大量订单（受线程数限制）
     */
    public void processOrdersTraditional(List<Order> orders) {
        ExecutorService executor = Executors.newFixedThreadPool(100);
        
        for (Order order : orders) {
            executor.submit(() -> {
                // 调用外部服务（IO密集）
                String result = restTemplate.getForObject(
                    "http://payment-service/pay/" + order.getId(), 
                    String.class
                );
                processResult(result);
            });
        }
        
        executor.shutdown();
    }
    
    /**
     * 虚拟线程方式：处理百万级订单
     */
    public void processOrdersVirtual(List<Order> orders) {
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (Order order : orders) {
                executor.submit(() -> {
                    // 调用外部服务（IO密集）
                    String result = restTemplate.getForObject(
                        "http://payment-service/pay/" + order.getId(), 
                        String.class
                    );
                    processResult(result);
                });
            }
        }
        // 可以轻松处理百万级任务
    }
    
    /**
     * Spring Boot 3.2+ 自动支持虚拟线程
     */
    @GetMapping("/orders/{id}")
    public Order getOrder(@PathVariable Long id) {
        // 每个请求自动使用虚拟线程处理
        return orderRepository.findById(id).orElseThrow();
    }
}
```

### 虚拟线程最佳实践

```java
/**
 * 虚拟线程最佳实践
 * @author erik.zhou
 */
public class VirtualThreadBestPractices {
    
    /**
     * ✅ 适合使用虚拟线程的场景
     */
    
    // 1. IO密集型任务
    public void ioIntensiveTask() {
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            executor.submit(() -> {
                // 网络请求
                String response = httpClient.get("https://api.example.com");
                
                // 数据库查询
                List<User> users = userRepository.findAll();
                
                // 文件读写
                Files.readAllLines(Path.of("data.txt"));
            });
        }
    }
    
    // 2. 高并发场景
    public void highConcurrency() {
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            // 处理100万个请求
            for (int i = 0; i < 1_000_000; i++) {
                executor.submit(() -> {
                    handleRequest();
                });
            }
        }
    }
    
    /**
     * ❌ 不适合使用虚拟线程的场景
     */
    
    // 1. CPU密集型任务
    public void cpuIntensiveTask() {
        // 使用传统线程池（线程数=CPU核心数）
        ExecutorService executor = Executors.newFixedThreadPool(
            Runtime.getRuntime().availableProcessors()
        );
        
        executor.submit(() -> {
            // 复杂计算
            calculatePrimeNumbers(1_000_000);
        });
    }
    
    // 2. 使用synchronized的代码
    public void synchronizedCode() {
        // ⚠️ 虚拟线程在synchronized块中会固定到平台线程
        synchronized (this) {
            // 这里会失去虚拟线程的优势
            doSomething();
        }
        
        // ✅ 使用ReentrantLock代替
        ReentrantLock lock = new ReentrantLock();
        lock.lock();
        try {
            doSomething();
        } finally {
            lock.unlock();
        }
    }
    
    /**
     * 虚拟线程监控
     */
    public void monitoring() {
        // 获取虚拟线程信息
        Thread thread = Thread.currentThread();
        
        System.out.println("Is virtual: " + thread.isVirtual());
        System.out.println("Thread ID: " + thread.threadId());
        System.out.println("Thread name: " + thread.getName());
    }
}
```

### 性能对比

```java
/**
 * 虚拟线程性能测试
 * @author erik.zhou
 */
public class VirtualThreadPerformanceTest {
    
    /**
     * 测试：创建100万个线程
     */
    @Test
    public void testCreateMillionThreads() {
        // 传统线程：OOM（内存不足）
        // ExecutorService executor = Executors.newCachedThreadPool();
        
        // 虚拟线程：轻松完成
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            long start = System.currentTimeMillis();
            
            for (int i = 0; i < 1_000_000; i++) {
                executor.submit(() -> {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        Thread.currentThread().interrupt();
                    }
                });
            }
            
            long end = System.currentTimeMillis();
            System.out.println("Time: " + (end - start) + "ms");
            // 结果：约2-3秒完成
        }
    }
    
    /**
     * 测试：IO密集型任务
     */
    @Test
    public void testIOIntensive() {
        int taskCount = 10_000;
        
        // 传统线程池
        long time1 = testWithThreadPool(taskCount);
        System.out.println("Thread Pool: " + time1 + "ms");
        
        // 虚拟线程
        long time2 = testWithVirtualThreads(taskCount);
        System.out.println("Virtual Threads: " + time2 + "ms");
        
        // 结果：虚拟线程快5-10倍
    }
    
    private long testWithThreadPool(int taskCount) {
        ExecutorService executor = Executors.newFixedThreadPool(100);
        long start = System.currentTimeMillis();
        
        for (int i = 0; i < taskCount; i++) {
            executor.submit(() -> simulateIOTask());
        }
        
        executor.shutdown();
        try {
            executor.awaitTermination(1, TimeUnit.HOURS);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        return System.currentTimeMillis() - start;
    }
    
    private long testWithVirtualThreads(int taskCount) {
        try (ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor()) {
            long start = System.currentTimeMillis();
            
            for (int i = 0; i < taskCount; i++) {
                executor.submit(() -> simulateIOTask());
            }
            
            return System.currentTimeMillis() - start;
        }
    }
    
    private void simulateIOTask() {
        try {
            Thread.sleep(100);  // 模拟IO等待
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

---

## 🔗 结构化并发（Structured Concurrency）

### 什么是结构化并发

```
结构化并发让并发代码更易理解和维护：

传统并发问题：
- 子任务生命周期难以管理
- 异常处理复杂
- 资源泄漏风险

结构化并发优势：
- 子任务生命周期与父任务绑定
- 统一的异常处理
- 自动资源清理
```

### 使用结构化并发

```java
/**
 * 结构化并发示例
 * @author erik.zhou
 */
public class StructuredConcurrencyExample {
    
    /**
     * 传统方式：手动管理子任务
     */
    public Order getOrderTraditional(Long orderId) throws Exception {
        ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
        
        try {
            // 并发查询用户和商品信息
            Future<User> userFuture = executor.submit(() -> getUserInfo(orderId));
            Future<Product> productFuture = executor.submit(() -> getProductInfo(orderId));
            
            // 等待结果
            User user = userFuture.get();
            Product product = productFuture.get();
            
            return new Order(user, product);
            
        } finally {
            executor.shutdown();
        }
    }
    
    /**
     * 结构化并发方式：自动管理
     */
    public Order getOrderStructured(Long orderId) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            
            // 启动子任务
            Supplier<User> userTask = scope.fork(() -> getUserInfo(orderId));
            Supplier<Product> productTask = scope.fork(() -> getProductInfo(orderId));
            
            // 等待所有任务完成或失败
            scope.join();
            scope.throwIfFailed();
            
            // 获取结果
            User user = userTask.get();
            Product product = productTask.get();
            
            return new Order(user, product);
        }
        // 自动清理资源
    }
    
    /**
     * 高级用例：超时控制
     */
    public Order getOrderWithTimeout(Long orderId) throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
            
            Supplier<User> userTask = scope.fork(() -> getUserInfo(orderId));
            Supplier<Product> productTask = scope.fork(() -> getProductInfo(orderId));
            
            // 等待最多2秒
            scope.join().throwIfFailed(e -> new TimeoutException("查询超时"));
            
            return new Order(userTask.get(), productTask.get());
        }
    }
    
    /**
     * 高级用例：获取最快的结果
     */
    public String getFromMultipleSources() throws Exception {
        try (var scope = new StructuredTaskScope.ShutdownOnSuccess<String>()) {
            
            // 从多个数据源查询
            scope.fork(() -> queryFromSource1());
            scope.fork(() -> queryFromSource2());
            scope.fork(() -> queryFromSource3());
            
            // 返回最快完成的结果
            scope.join();
            
            return scope.result();
        }
    }
    
    private User getUserInfo(Long orderId) {
        // 模拟查询
        return new User();
    }
    
    private Product getProductInfo(Long orderId) {
        // 模拟查询
        return new Product();
    }
}
```

---

## 🎨 模式匹配增强

### Record Patterns

```java
/**
 * Record模式匹配
 * @author erik.zhou
 */
public class RecordPatternExample {
    
    record Point(int x, int y) {}
    record Circle(Point center, int radius) {}
    record Rectangle(Point topLeft, Point bottomRight) {}
    
    /**
     * 传统方式
     */
    public void printShapeTraditional(Object shape) {
        if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            Point center = circle.center();
            System.out.println("Circle at (" + center.x() + ", " + center.y() + ")");
        } else if (shape instanceof Rectangle) {
            Rectangle rect = (Rectangle) shape;
            Point topLeft = rect.topLeft();
            System.out.println("Rectangle at (" + topLeft.x() + ", " + topLeft.y() + ")");
        }
    }
    
    /**
     * Java 21方式：Record模式匹配
     */
    public void printShape(Object shape) {
        switch (shape) {
            case Circle(Point(int x, int y), int radius) ->
                System.out.println("Circle at (" + x + ", " + y + ") with radius " + radius);
            
            case Rectangle(Point(int x1, int y1), Point(int x2, int y2)) ->
                System.out.println("Rectangle from (" + x1 + ", " + y1 + ") to (" + x2 + ", " + y2 + ")");
            
            default ->
                System.out.println("Unknown shape");
        }
    }
}
```

### Switch表达式增强

```java
/**
 * Switch表达式增强
 * @author erik.zhou
 */
public class SwitchEnhancementExample {
    
    /**
     * 传统switch
     */
    public String getDayTypeTraditional(String day) {
        String result;
        switch (day) {
            case "MONDAY":
            case "TUESDAY":
            case "WEDNESDAY":
            case "THURSDAY":
            case "FRIDAY":
                result = "工作日";
                break;
            case "SATURDAY":
            case "SUNDAY":
                result = "周末";
                break;
            default:
                result = "未知";
        }
        return result;
    }
    
    /**
     * Java 21 switch表达式
     */
    public String getDayType(String day) {
        return switch (day) {
            case "MONDAY", "TUESDAY", "WEDNESDAY", "THURSDAY", "FRIDAY" -> "工作日";
            case "SATURDAY", "SUNDAY" -> "周末";
            default -> "未知";
        };
    }
    
    /**
     * 模式匹配 + null处理
     */
    public String formatValue(Object obj) {
        return switch (obj) {
            case null -> "null";
            case Integer i -> "整数: " + i;
            case String s -> "字符串: " + s;
            case Long l -> "长整数: " + l;
            default -> "其他类型";
        };
    }
    
    /**
     * 守卫条件（when）
     */
    public String classifyNumber(int num) {
        return switch (num) {
            case int n when n < 0 -> "负数";
            case int n when n == 0 -> "零";
            case int n when n > 0 && n <= 10 -> "小正数";
            case int n when n > 10 -> "大正数";
            default -> "未知";
        };
    }
}
```

---

## 🆕 其他新特性

### 1. Sequenced Collections

```java
/**
 * 有序集合接口
 * @author erik.zhou
 */
public class SequencedCollectionsExample {
    
    public void demonstrateSequencedList() {
        List<String> list = new ArrayList<>();
        list.add("first");
        list.add("second");
        list.add("third");
        
        // 获取第一个和最后一个元素
        String first = list.getFirst();  // "first"
        String last = list.getLast();    // "third"
        
        // 反转视图
        List<String> reversed = list.reversed();
        System.out.println(reversed);  // [third, second, first]
    }
    
    public void demonstrateSequencedSet() {
        SequencedSet<String> set = new LinkedHashSet<>();
        set.add("a");
        set.add("b");
        set.add("c");
        
        String first = set.getFirst();  // "a"
        String last = set.getLast();    // "c"
        
        SequencedSet<String> reversed = set.reversed();
    }
    
    public void demonstrateSequencedMap() {
        SequencedMap<String, Integer> map = new LinkedHashMap<>();
        map.put("one", 1);
        map.put("two", 2);
        map.put("three", 3);
        
        Map.Entry<String, Integer> first = map.firstEntry();  // one=1
        Map.Entry<String, Integer> last = map.lastEntry();    // three=3
        
        SequencedMap<String, Integer> reversed = map.reversed();
    }
}
```

### 2. String Templates（预览）

```java
/**
 * 字符串模板
 * @author erik.zhou
 */
public class StringTemplateExample {
    
    public void traditionalWay() {
        String name = "张三";
        int age = 25;
        
        // 传统方式
        String message = "姓名: " + name + ", 年龄: " + age;
        
        // String.format
        String message2 = String.format("姓名: %s, 年龄: %d", name, age);
    }
    
    public void stringTemplate() {
        String name = "张三";
        int age = 25;
        
        // 字符串模板（预览特性）
        // String message = STR."姓名: \{name}, 年龄: \{age}";
        
        // 支持表达式
        // String message2 = STR."明年年龄: \{age + 1}";
    }
}
```

### 3. 性能改进

```java
/**
 * Java 21性能改进
 * @author erik.zhou
 */
public class PerformanceImprovements {
    
    /**
     * 1. G1 GC改进
     * - 更快的Full GC
     * - 更好的内存管理
     */
    
    /**
     * 2. ZGC改进
     * - 支持分代GC
     * - 更低的延迟
     */
    
    /**
     * 3. 启动性能
     * - CDS（Class Data Sharing）改进
     * - 更快的类加载
     */
    
    /**
     * 4. JIT编译优化
     * - 更好的内联
     * - 更好的向量化
     */
}
```

---

## 🎓 迁移指南

### 从Java 17迁移到Java 21

```xml
<!-- 1. 更新Maven配置 -->
<properties>
    <java.version>21</java.version>
    <maven.compiler.source>21</maven.compiler.source>
    <maven.compiler.target>21</maven.compiler.target>
</properties>

<!-- 2. 更新Spring Boot版本 -->
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.0</version>  <!-- 支持Java 21 -->
</parent>
```

```yaml
# 3. 启用虚拟线程
spring:
  threads:
    virtual:
      enabled: true
```

```java
/**
 * 4. 代码迁移
 * @author erik.zhou
 */
public class MigrationGuide {
    
    // 替换传统线程池
    // ExecutorService executor = Executors.newFixedThreadPool(100);
    ExecutorService executor = Executors.newVirtualThreadPerTaskExecutor();
    
    // 替换synchronized为ReentrantLock
    // synchronized (this) { ... }
    ReentrantLock lock = new ReentrantLock();
    lock.lock();
    try {
        // ...
    } finally {
        lock.unlock();
    }
    
    // 使用新的集合API
    List<String> list = new ArrayList<>();
    String first = list.getFirst();  // 代替 list.get(0)
    String last = list.getLast();    // 代替 list.get(list.size() - 1)
}
```

---

## 📝 总结

### 核心特性

1. **虚拟线程** - 革命性特性，轻松处理百万级并发
2. **结构化并发** - 更好的并发代码管理
3. **模式匹配** - 更简洁的代码
4. **性能提升** - 更快的启动和运行速度

### 使用建议

1. **IO密集型应用** - 立即使用虚拟线程
2. **高并发场景** - 虚拟线程是最佳选择
3. **CPU密集型** - 继续使用传统线程池
4. **避免synchronized** - 在虚拟线程中使用ReentrantLock

### 学习路径

1. 理解虚拟线程原理
2. 实践虚拟线程应用
3. 学习结构化并发
4. 掌握模式匹配
5. 性能测试和优化

---

**作者**: erik.zhou  
**最后更新**: 2024-01-04
