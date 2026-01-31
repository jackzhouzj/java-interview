# JVM 疑难杂症 - 完整解析

> 生产环境 JVM 问题的诊断和解决
> 
> @author erik.zhou

## 📚 概述

JVM 问题是生产环境最常见、影响最大的问题之一。本文档详细介绍：
- 常见 JVM 问题的现象
- 问题定位和分析方法
- 解决方案和预防措施
- 实战案例和经验总结

## 🎯 学习目标

- [ ] 掌握 JVM 诊断工具的使用
- [ ] 能够分析 OOM 问题
- [ ] 能够定位内存泄漏
- [ ] 能够分析 CPU 飙高问题
- [ ] 能够优化 GC 参数
- [ ] 能够分析线程死锁

---

## 1. 内存溢出（OOM）🔥

### 1.1 问题现象

**典型错误**：
```
java.lang.OutOfMemoryError: Java heap space
java.lang.OutOfMemoryError: GC overhead limit exceeded
java.lang.OutOfMemoryError: Metaspace
java.lang.OutOfMemoryError: unable to create new native thread
```

**系统表现**：
- 应用突然崩溃
- 接口响应超时
- 日志中出现 OOM 错误
- 监控显示内存持续增长

### 1.2 问题分析

**OOM 类型分析**：

**1. Java heap space（堆内存溢出）**

**原因**：
- 对象创建过多，无法被 GC 回收
- 内存泄漏
- 堆内存设置过小

**定位步骤**：

```bash
# 1. 配置 JVM 参数，OOM 时自动生成堆转储
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/heapdump.hprof

# 2. 使用 jmap 手动生成堆转储
jmap -dump:format=b,file=heap.dump <pid>

# 3. 使用 MAT 或 JProfiler 分析堆转储
# 查看占用内存最多的对象
# 查看对象的引用链
# 找出内存泄漏的根源
```

**案例 1：大对象导致 OOM**

```java
// 问题代码
@GetMapping("/export")
public void exportData(HttpServletResponse response) {
    // 一次性查询 100 万条数据到内存
    List<User> users = userRepository.findAll();  // ❌ 错误
    
    // 导出到 Excel
    ExcelWriter writer = new ExcelWriter(response.getOutputStream());
    writer.write(users);
}

// 解决方案：分批查询
@GetMapping("/export")
public void exportData(HttpServletResponse response) {
    int pageSize = 1000;
    int pageNum = 0;
    
    ExcelWriter writer = new ExcelWriter(response.getOutputStream());
    
    while (true) {
        // 分批查询
        Pageable pageable = PageRequest.of(pageNum, pageSize);
        Page<User> page = userRepository.findAll(pageable);
        
        if (page.isEmpty()) {
            break;
        }
        
        // 分批写入
        writer.write(page.getContent());
        pageNum++;
        
        // 清理已处理的数据
        page.getContent().clear();
    }
    
    writer.close();
}
```

**案例 2：缓存导致 OOM**

```java
// 问题代码
@Service
public class UserService {
    // 无限制的缓存
    private Map<Long, User> cache = new HashMap<>();  // ❌ 错误
    
    public User getUser(Long id) {
        User user = cache.get(id);
        if (user == null) {
            user = userRepository.findById(id);
            cache.put(id, user);  // 缓存会无限增长
        }
        return user;
    }
}

// 解决方案 1：使用 LRU 缓存
@Service
public class UserService {
    // 使用 Guava Cache，限制大小
    private LoadingCache<Long, User> cache = CacheBuilder.newBuilder()
        .maximumSize(10000)  // 最多缓存 10000 个
        .expireAfterWrite(1, TimeUnit.HOURS)  // 1 小时过期
        .build(new CacheLoader<Long, User>() {
            @Override
            public User load(Long id) {
                return userRepository.findById(id);
            }
        });
    
    public User getUser(Long id) {
        return cache.getUnchecked(id);
    }
}

// 解决方案 2：使用 Redis
@Service
public class UserService {
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    public User getUser(Long id) {
        String key = "user:" + id;
        String json = redisTemplate.opsForValue().get(key);
        
        if (json != null) {
            return JSON.parseObject(json, User.class);
        }
        
        User user = userRepository.findById(id);
        redisTemplate.opsForValue().set(key, JSON.toJSONString(user), 1, TimeUnit.HOURS);
        return user;
    }
}
```

**2. GC overhead limit exceeded**

**原因**：
- GC 时间占比超过 98%
- 回收的内存少于 2%
- 说明内存严重不足

**解决方案**：
```bash
# 1. 增加堆内存
-Xms4g -Xmx4g

# 2. 优化代码，减少对象创建

# 3. 如果确实需要，可以关闭这个限制（不推荐）
-XX:-UseGCOverheadLimit
```

**3. Metaspace（元空间溢出）**

**原因**：
- 加载的类太多
- 动态生成类（如 CGLIB、反射）
- 元空间设置过小

**定位步骤**：

```bash
# 1. 查看元空间使用情况
jstat -gc <pid> 1000

# 2. 查看加载的类数量
jstat -class <pid>

# 3. 使用 Arthas 查看类加载情况
classloader -t
```

**案例：动态代理导致 Metaspace OOM**

```java
// 问题代码
@Service
public class ProxyService {
    
    public void createProxy() {
        // 每次都创建新的代理类
        for (int i = 0; i < 100000; i++) {
            UserService proxy = (UserService) Proxy.newProxyInstance(
                UserService.class.getClassLoader(),
                new Class[]{UserService.class},
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) {
                        // 代理逻辑
                        return null;
                    }
                }
            );
        }
    }
}

// 解决方案：复用代理对象
@Service
public class ProxyService {
    private UserService proxy;
    
    @PostConstruct
    public void init() {
        // 只创建一次代理
        proxy = (UserService) Proxy.newProxyInstance(
            UserService.class.getClassLoader(),
            new Class[]{UserService.class},
            new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) {
                    // 代理逻辑
                    return null;
                }
            }
        );
    }
}
```

**4. unable to create new native thread**

**原因**：
- 创建的线程数超过系统限制
- 线程泄漏（线程未正确关闭）

**定位步骤**：

```bash
# 1. 查看进程的线程数
ps -eLf | grep <pid> | wc -l

# 2. 查看系统线程限制
ulimit -u

# 3. 查看线程快照
jstack <pid> > thread.dump

# 4. 统计线程状态
grep "java.lang.Thread.State" thread.dump | sort | uniq -c
```

**案例：线程池配置不当**

```java
// 问题代码
@Service
public class TaskService {
    
    public void executeTask() {
        // 每次都创建新的线程池
        ExecutorService executor = Executors.newCachedThreadPool();  // ❌ 错误
        
        for (int i = 0; i < 1000; i++) {
            executor.submit(() -> {
                // 任务逻辑
            });
        }
        
        // 没有关闭线程池，导致线程泄漏
    }
}

// 解决方案：复用线程池
@Configuration
public class ThreadPoolConfig {
    
    @Bean
    public ThreadPoolTaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setThreadNamePrefix("task-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
}

@Service
public class TaskService {
    
    @Autowired
    private ThreadPoolTaskExecutor taskExecutor;
    
    public void executeTask() {
        for (int i = 0; i < 1000; i++) {
            taskExecutor.submit(() -> {
                // 任务逻辑
            });
        }
    }
}
```

### 1.3 解决方案

**1. 调整 JVM 参数**

```bash
# 堆内存设置
-Xms4g                    # 初始堆大小
-Xmx4g                    # 最大堆大小（建议与 Xms 相同）

# 元空间设置
-XX:MetaspaceSize=256m    # 初始元空间大小
-XX:MaxMetaspaceSize=512m # 最大元空间大小

# OOM 时生成堆转储
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/heapdump.hprof

# GC 日志
-Xlog:gc*:file=/tmp/gc.log:time,uptime:filecount=10,filesize=100m
```

**2. 代码优化**

```java
// 1. 及时释放资源
try (InputStream is = new FileInputStream("file.txt")) {
    // 使用资源
} // 自动关闭

// 2. 使用对象池
ObjectPool<Connection> pool = new GenericObjectPool<>(factory);

// 3. 避免创建大对象
// 使用流式处理代替一次性加载

// 4. 使用弱引用缓存
Map<String, WeakReference<Object>> cache = new WeakHashMap<>();
```

**3. 监控告警**

```java
// 配置内存告警
@Component
public class MemoryMonitor {
    
    @Scheduled(fixedDelay = 60000)
    public void checkMemory() {
        MemoryMXBean memoryMXBean = ManagementFactory.getMemoryMXBean();
        MemoryUsage heapUsage = memoryMXBean.getHeapMemoryUsage();
        
        long used = heapUsage.getUsed();
        long max = heapUsage.getMax();
        double usage = (double) used / max;
        
        if (usage > 0.9) {
            // 发送告警
            alertService.sendAlert("内存使用率超过 90%: " + usage);
        }
    }
}
```

---

## 2. 内存泄漏 🔥

### 2.1 问题现象

**系统表现**：
- 内存持续增长，不回落
- Full GC 频繁，但内存不释放
- 最终导致 OOM

**监控表现**：
- 堆内存使用率持续上升
- Old Gen 区域持续增长
- GC 后内存不下降

### 2.2 问题分析

**常见内存泄漏场景**：

**1. 静态集合持有对象引用**

```java
// 问题代码
public class CacheManager {
    // 静态集合，永远不会被 GC
    private static Map<String, Object> cache = new HashMap<>();  // ❌ 内存泄漏
    
    public static void put(String key, Object value) {
        cache.put(key, value);
    }
}

// 解决方案 1：使用弱引用
public class CacheManager {
    private static Map<String, WeakReference<Object>> cache = new WeakHashMap<>();
    
    public static void put(String key, Object value) {
        cache.put(key, new WeakReference<>(value));
    }
}

// 解决方案 2：设置过期时间
public class CacheManager {
    private static LoadingCache<String, Object> cache = CacheBuilder.newBuilder()
        .maximumSize(1000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .build(new CacheLoader<String, Object>() {
            @Override
            public Object load(String key) {
                return null;
            }
        });
}
```

**2. 监听器未注销**

```java
// 问题代码
public class EventManager {
    private List<EventListener> listeners = new ArrayList<>();
    
    public void addEventListener(EventListener listener) {
        listeners.add(listener);  // ❌ 监听器永远不会被移除
    }
}

// 解决方案：提供注销方法
public class EventManager {
    private List<EventListener> listeners = new ArrayList<>();
    
    public void addEventListener(EventListener listener) {
        listeners.add(listener);
    }
    
    public void removeEventListener(EventListener listener) {
        listeners.remove(listener);
    }
}

// 使用示例
public class MyComponent {
    private EventListener listener;
    
    @PostConstruct
    public void init() {
        listener = new EventListener() {
            @Override
            public void onEvent(Event event) {
                // 处理事件
            }
        };
        eventManager.addEventListener(listener);
    }
    
    @PreDestroy
    public void destroy() {
        // 注销监听器
        eventManager.removeEventListener(listener);
    }
}
```

**3. ThreadLocal 未清理**

```java
// 问题代码
public class UserContext {
    private static ThreadLocal<User> userThreadLocal = new ThreadLocal<>();  // ❌ 可能泄漏
    
    public static void setUser(User user) {
        userThreadLocal.set(user);
    }
    
    public static User getUser() {
        return userThreadLocal.get();
    }
}

// 解决方案：使用后及时清理
public class UserContext {
    private static ThreadLocal<User> userThreadLocal = new ThreadLocal<>();
    
    public static void setUser(User user) {
        userThreadLocal.set(user);
    }
    
    public static User getUser() {
        return userThreadLocal.get();
    }
    
    public static void clear() {
        userThreadLocal.remove();  // 清理 ThreadLocal
    }
}

// 使用示例（Filter 中）
@Component
public class UserContextFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
        try {
            // 设置用户信息
            User user = getUserFromRequest(request);
            UserContext.setUser(user);
            
            chain.doFilter(request, response);
        } finally {
            // 清理 ThreadLocal
            UserContext.clear();
        }
    }
}
```

**4. 连接未关闭**

```java
// 问题代码
public class DatabaseService {
    
    public List<User> getUsers() {
        Connection conn = null;
        Statement stmt = null;
        ResultSet rs = null;
        
        try {
            conn = dataSource.getConnection();
            stmt = conn.createStatement();
            rs = stmt.executeQuery("SELECT * FROM user");
            
            List<User> users = new ArrayList<>();
            while (rs.next()) {
                users.add(mapUser(rs));
            }
            return users;
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
        // ❌ 没有关闭连接，导致连接泄漏
    }
}

// 解决方案：使用 try-with-resources
public class DatabaseService {
    
    public List<User> getUsers() {
        String sql = "SELECT * FROM user";
        
        try (Connection conn = dataSource.getConnection();
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            
            List<User> users = new ArrayList<>();
            while (rs.next()) {
                users.add(mapUser(rs));
            }
            return users;
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
        // 自动关闭连接
    }
}
```

### 2.3 定位方法

**使用 MAT 分析堆转储**：

```bash
# 1. 生成堆转储
jmap -dump:format=b,file=heap1.dump <pid>

# 等待一段时间后再生成一次
jmap -dump:format=b,file=heap2.dump <pid>

# 2. 使用 MAT 打开堆转储
# 查看 Leak Suspects（泄漏嫌疑）
# 查看 Dominator Tree（支配树）
# 查看对象的引用链

# 3. 对比两次堆转储
# 找出持续增长的对象
```

**使用 Arthas 在线诊断**：

```bash
# 1. 启动 Arthas
java -jar arthas-boot.jar

# 2. 查看堆内存使用
dashboard

# 3. 查看对象实例数
memory

# 4. 查看某个类的实例
sc -d com.example.User
vmtool --action getInstances --className com.example.User --limit 10

# 5. 查看对象的引用链
vmtool --action getInstances --className com.example.User --limit 1 | grep -A 100 "instances"
```



---

## 3. CPU 飙高 🔥

### 3.1 问题现象

**系统表现**：
- CPU 使用率突然飙升到 100%
- 应用响应变慢
- 服务器负载高

**监控表现**：
- top 命令显示 Java 进程 CPU 占用高
- 接口响应时间增加
- 系统吞吐量下降

### 3.2 问题分析

**定位步骤**：

```bash
# 1. 找出 CPU 占用高的进程
top
# 记录 Java 进程的 PID

# 2. 找出该进程中 CPU 占用高的线程
top -Hp <pid>
# 记录线程 ID（十进制）

# 3. 将线程 ID 转换为十六进制
printf "%x\n" <thread_id>

# 4. 生成线程快照
jstack <pid> > thread.dump

# 5. 在线程快照中搜索十六进制线程 ID
grep <hex_thread_id> thread.dump -A 50
```

**案例 1：死循环导致 CPU 飙高**

```java
// 问题代码
@Service
public class DataProcessor {
    
    public void processData() {
        List<Data> dataList = dataRepository.findAll();
        
        // 死循环
        while (true) {  // ❌ 错误
            for (Data data : dataList) {
                // 处理数据
                process(data);
            }
        }
    }
}

// 解决方案：添加退出条件
@Service
public class DataProcessor {
    
    private volatile boolean running = true;
    
    public void processData() {
        while (running) {
            List<Data> dataList = dataRepository.findPending();
            
            if (dataList.isEmpty()) {
                // 没有数据时休眠
                Thread.sleep(1000);
                continue;
            }
            
            for (Data data : dataList) {
                process(data);
            }
        }
    }
    
    public void stop() {
        running = false;
    }
}
```

**案例 2：正则表达式回溯导致 CPU 飙高**

```java
// 问题代码
public class Validator {
    
    // 复杂的正则表达式
    private static final Pattern PATTERN = 
        Pattern.compile("(a+)+b");  // ❌ 可能导致回溯
    
    public boolean validate(String input) {
        return PATTERN.matcher(input).matches();
    }
}

// 当输入 "aaaaaaaaaaaaaaaaaaaaaaaac" 时，会导致大量回溯，CPU 飙高

// 解决方案 1：优化正则表达式
public class Validator {
    // 使用原子组，避免回溯
    private static final Pattern PATTERN = 
        Pattern.compile("(?>a+)+b");
    
    public boolean validate(String input) {
        return PATTERN.matcher(input).matches();
    }
}

// 解决方案 2：添加超时限制
public class Validator {
    private static final Pattern PATTERN = Pattern.compile("(a+)+b");
    
    public boolean validate(String input) {
        Future<Boolean> future = executor.submit(() -> {
            return PATTERN.matcher(input).matches();
        });
        
        try {
            // 超时 1 秒
            return future.get(1, TimeUnit.SECONDS);
        } catch (TimeoutException e) {
            future.cancel(true);
            return false;
        }
    }
}
```

**案例 3：频繁 GC 导致 CPU 飙高**

```bash
# 查看 GC 情况
jstat -gc <pid> 1000

# 如果看到 Full GC 频繁，说明内存不足
# 解决方案：
# 1. 增加堆内存
# 2. 优化代码，减少对象创建
# 3. 调整 GC 参数
```

### 3.3 解决方案

**1. 使用 Arthas 快速定位**

```bash
# 1. 启动 Arthas
java -jar arthas-boot.jar

# 2. 查看 CPU 占用高的线程
thread -n 3

# 3. 查看某个线程的堆栈
thread <thread_id>

# 4. 监控方法执行时间
trace com.example.Service method

# 5. 查看方法调用次数和耗时
monitor -c 5 com.example.Service method
```

**2. 代码优化**

```java
// 1. 避免死循环
while (condition) {
    // 添加退出条件
    // 添加休眠
}

// 2. 优化算法
// 使用更高效的算法和数据结构

// 3. 使用缓存
// 避免重复计算

// 4. 异步处理
@Async
public void processData() {
    // 耗时操作
}
```

---

## 4. Full GC 频繁 🔥

### 4.1 问题现象

**系统表现**：
- 应用频繁卡顿
- 接口响应时间长
- 吞吐量下降

**GC 日志表现**：
- Full GC 频率高（如每分钟多次）
- Full GC 时间长（如超过 1 秒）
- Old Gen 区域持续增长

### 4.2 问题分析

**查看 GC 情况**：

```bash
# 1. 实时查看 GC 统计
jstat -gc <pid> 1000

# 输出说明：
# S0C: Survivor 0 容量
# S1C: Survivor 1 容量
# S0U: Survivor 0 使用量
# S1U: Survivor 1 使用量
# EC: Eden 容量
# EU: Eden 使用量
# OC: Old Gen 容量
# OU: Old Gen 使用量
# MC: Metaspace 容量
# MU: Metaspace 使用量
# YGC: Young GC 次数
# YGCT: Young GC 总时间
# FGC: Full GC 次数
# FGCT: Full GC 总时间

# 2. 查看 GC 原因
jstat -gccause <pid> 1000

# 3. 分析 GC 日志
# 配置 GC 日志参数
-Xlog:gc*:file=/tmp/gc.log:time,uptime:filecount=10,filesize=100m

# 使用 GCEasy 或 GCViewer 分析日志
```

**常见原因**：

**1. 内存设置不合理**

```bash
# 问题：堆内存太小
-Xms512m -Xmx512m  # ❌ 太小

# 解决方案：增加堆内存
-Xms4g -Xmx4g

# 问题：新生代太小
-XX:NewRatio=4  # Old:Young = 4:1，新生代太小

# 解决方案：增加新生代
-XX:NewRatio=2  # Old:Young = 2:1
# 或直接设置新生代大小
-Xmn2g
```

**2. 对象晋升过快**

```java
// 问题代码：大对象直接进入老年代
public class DataProcessor {
    
    public void process() {
        // 创建大对象（超过 -XX:PretenureSizeThreshold）
        byte[] data = new byte[10 * 1024 * 1024];  // 10MB
        // 处理数据
    }
}

// 解决方案：分批处理
public class DataProcessor {
    
    public void process() {
        int batchSize = 1024 * 1024;  // 1MB
        for (int i = 0; i < 10; i++) {
            byte[] data = new byte[batchSize];
            // 处理数据
            // 处理完后，data 可以被 Young GC 回收
        }
    }
}
```

**3. 内存泄漏**

```java
// 参考前面的内存泄漏章节
```

### 4.3 解决方案

**1. 调整 GC 参数**

```bash
# 使用 G1 GC（推荐）
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200  # 最大暂停时间 200ms
-XX:G1HeapRegionSize=16m  # Region 大小
-XX:InitiatingHeapOccupancyPercent=45  # 触发并发标记的堆占用阈值

# 使用 ZGC（JDK 11+，低延迟）
-XX:+UseZGC
-XX:ZCollectionInterval=120  # GC 间隔

# 使用 CMS（已废弃，不推荐）
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=70
```

**2. 代码优化**

```java
// 1. 对象复用
// 使用对象池
ObjectPool<Connection> pool = new GenericObjectPool<>(factory);

// 2. 减少对象创建
// 使用 StringBuilder 代替 String 拼接
StringBuilder sb = new StringBuilder();
for (String s : list) {
    sb.append(s);
}

// 3. 及时释放引用
list.clear();
map.clear();

// 4. 使用基本类型
int[] array = new int[1000];  // 而不是 Integer[]
```

**3. 监控和告警**

```java
@Component
public class GCMonitor {
    
    @Scheduled(fixedDelay = 60000)
    public void checkGC() {
        List<GarbageCollectorMXBean> gcBeans = 
            ManagementFactory.getGarbageCollectorMXBeans();
        
        for (GarbageCollectorMXBean gcBean : gcBeans) {
            long count = gcBean.getCollectionCount();
            long time = gcBean.getCollectionTime();
            
            if (gcBean.getName().contains("Old") || gcBean.getName().contains("MarkSweep")) {
                // Full GC 监控
                if (count > lastFullGCCount + 10) {  // 1 分钟内 Full GC 超过 10 次
                    alertService.sendAlert("Full GC 频繁: " + count);
                }
                lastFullGCCount = count;
            }
        }
    }
}
```

---

## 5. 线程死锁 🔥

### 5.1 问题现象

**系统表现**：
- 应用卡死，无响应
- 部分接口超时
- CPU 使用率不高，但应用不工作

### 5.2 问题分析

**定位步骤**：

```bash
# 1. 生成线程快照
jstack <pid> > thread.dump

# 2. 查找死锁信息
grep "Found one Java-level deadlock" thread.dump -A 50

# 3. 使用 Arthas 检测死锁
thread -b
```

**死锁示例**：

```java
// 问题代码
public class DeadlockExample {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        synchronized (lock1) {
            System.out.println("Thread 1: Holding lock 1...");
            
            try { Thread.sleep(10); } catch (InterruptedException e) {}
            
            System.out.println("Thread 1: Waiting for lock 2...");
            synchronized (lock2) {  // ❌ 死锁
                System.out.println("Thread 1: Holding lock 1 & 2...");
            }
        }
    }
    
    public void method2() {
        synchronized (lock2) {
            System.out.println("Thread 2: Holding lock 2...");
            
            try { Thread.sleep(10); } catch (InterruptedException e) {}
            
            System.out.println("Thread 2: Waiting for lock 1...");
            synchronized (lock1) {  // ❌ 死锁
                System.out.println("Thread 2: Holding lock 1 & 2...");
            }
        }
    }
}
```

**线程快照示例**：

```
Found one Java-level deadlock:
=============================
"Thread-1":
  waiting to lock monitor 0x00007f8a1c004e00 (object 0x00000000d5f78a20, a java.lang.Object),
  which is held by "Thread-0"
"Thread-0":
  waiting to lock monitor 0x00007f8a1c007360 (object 0x00000000d5f78a30, a java.lang.Object),
  which is held by "Thread-1"
```

### 5.3 解决方案

**1. 避免嵌套锁**

```java
// 解决方案 1：按顺序获取锁
public class DeadlockSolution {
    private final Object lock1 = new Object();
    private final Object lock2 = new Object();
    
    public void method1() {
        // 总是先获取 lock1，再获取 lock2
        synchronized (lock1) {
            synchronized (lock2) {
                // 业务逻辑
            }
        }
    }
    
    public void method2() {
        // 同样先获取 lock1，再获取 lock2
        synchronized (lock1) {
            synchronized (lock2) {
                // 业务逻辑
            }
        }
    }
}
```

**2. 使用 tryLock**

```java
// 解决方案 2：使用 ReentrantLock 的 tryLock
public class DeadlockSolution {
    private final Lock lock1 = new ReentrantLock();
    private final Lock lock2 = new ReentrantLock();
    
    public void method1() {
        while (true) {
            if (lock1.tryLock()) {
                try {
                    if (lock2.tryLock()) {
                        try {
                            // 业务逻辑
                            break;
                        } finally {
                            lock2.unlock();
                        }
                    }
                } finally {
                    lock1.unlock();
                }
            }
            // 获取锁失败，休眠后重试
            try { Thread.sleep(10); } catch (InterruptedException e) {}
        }
    }
}
```

**3. 使用超时机制**

```java
// 解决方案 3：设置超时时间
public class DeadlockSolution {
    private final Lock lock1 = new ReentrantLock();
    private final Lock lock2 = new ReentrantLock();
    
    public void method1() throws InterruptedException {
        if (lock1.tryLock(1, TimeUnit.SECONDS)) {
            try {
                if (lock2.tryLock(1, TimeUnit.SECONDS)) {
                    try {
                        // 业务逻辑
                    } finally {
                        lock2.unlock();
                    }
                } else {
                    throw new RuntimeException("获取 lock2 超时");
                }
            } finally {
                lock1.unlock();
            }
        } else {
            throw new RuntimeException("获取 lock1 超时");
        }
    }
}
```

---

## 📝 学习检查清单

### OOM 问题
- [ ] 能够分析不同类型的 OOM
- [ ] 能够使用 jmap 生成堆转储
- [ ] 能够使用 MAT 分析堆转储
- [ ] 能够优化代码避免 OOM

### 内存泄漏
- [ ] 能够识别常见的内存泄漏场景
- [ ] 能够使用工具定位内存泄漏
- [ ] 能够修复内存泄漏问题

### CPU 飙高
- [ ] 能够定位 CPU 占用高的线程
- [ ] 能够分析线程快照
- [ ] 能够优化代码降低 CPU 使用

### Full GC 频繁
- [ ] 能够分析 GC 日志
- [ ] 能够调整 GC 参数
- [ ] 能够优化代码减少 GC

### 线程死锁
- [ ] 能够检测死锁
- [ ] 能够分析死锁原因
- [ ] 能够避免死锁

---

## 🛠️ 常用诊断命令

```bash
# 查看 Java 进程
jps -l

# 查看 GC 统计
jstat -gc <pid> 1000

# 生成堆转储
jmap -dump:format=b,file=heap.dump <pid>

# 查看堆内存
jmap -heap <pid>

# 生成线程快照
jstack <pid> > thread.dump

# 查看 JVM 参数
jinfo -flags <pid>

# 使用 Arthas
java -jar arthas-boot.jar
```

---

**@author erik.zhou**

