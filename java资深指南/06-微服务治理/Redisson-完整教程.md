# Redisson 完整教程

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
- **版本**: 3.x
- **官方文档**: https://github.com/redisson/redisson
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Java基础
  - Redis基础
  - 多线程与并发编程
  - 分布式系统基础概念
- **文档来源**: Context7 + 官方GitHub文档
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Redisson的核心概念和应用场景
- [ ] 掌握分布式锁的实现原理和使用方法
- [ ] 熟练使用Redisson的分布式集合和对象
- [ ] 理解看门狗（Watchdog）机制的工作原理
- [ ] 掌握Redisson的配置和连接模式
- [ ] 能够在实际项目中正确使用Redisson解决分布式问题

## 📖 基础概念

### 1.1 什么是Redisson

Redisson是一个在Redis基础上实现的Java驻内存数据网格（In-Memory Data Grid）。它不仅提供了一系列的分布式Java常用对象，还提供了许多分布式服务。

**核心特点**：
- 基于Netty框架的事件驱动通信层
- 提供丰富的分布式对象和服务
- 支持同步、异步、响应式和RxJava API
- 自动重连和失败重试机制
- 支持Redis单机、主从、哨兵、集群等多种部署模式

### 1.2 Redisson vs Jedis/Lettuce

| 特性 | Redisson | Jedis | Lettuce |
|------|----------|-------|---------|
| 定位 | 分布式框架 | Redis客户端 | Redis客户端 |
| API风格 | 面向对象 | 命令式 | 命令式 |
| 分布式锁 | 内置实现 | 需自行实现 | 需自行实现 |
| 异步支持 | 完整支持 | 不支持 | 支持 |
| 响应式编程 | 支持 | 不支持 | 支持 |
| 学习曲线 | 较陡 | 平缓 | 中等 |


### 1.3 应用场景

**典型应用场景**：
1. **分布式锁**: 解决分布式环境下的资源竞争问题
2. **分布式集合**: 跨JVM共享数据结构
3. **分布式对象**: 实现分布式缓存和数据共享
4. **分布式服务**: 远程服务调用、分布式任务调度等
5. **缓存管理**: 实现本地缓存与Redis的二级缓存
6. **限流降级**: 实现分布式限流和熔断

### 1.4 核心组件架构

```
┌─────────────────────────────────────────────────────────┐
│                    Redisson Client                       │
├─────────────────────────────────────────────────────────┤
│  分布式锁        │  分布式集合      │  分布式对象       │
│  - RLock         │  - RMap          │  - RBucket        │
│  - RReadWriteLock│  - RSet          │  - RAtomicLong    │
│  - RSemaphore    │  - RList         │  - RBitSet        │
│  - RCountDownLatch│ - RQueue        │  - RBloomFilter   │
├─────────────────────────────────────────────────────────┤
│  分布式服务      │  事务支持        │  脚本执行         │
│  - RExecutorService│ - RTransaction │  - RScript        │
│  - RScheduledExecutorService│       │                   │
├─────────────────────────────────────────────────────────┤
│              Netty 通信层 (异步/响应式)                  │
├─────────────────────────────────────────────────────────┤
│                    Redis Server                          │
│         (单机/主从/哨兵/集群)                            │
└─────────────────────────────────────────────────────────┘
```

## 🔥 核心特性 (重点)

### 2.1 分布式锁 🔥

分布式锁是Redisson最核心的功能之一，用于解决分布式环境下的资源竞争问题。

#### 2.1.1 可重入锁（RLock）

**基本使用**：
```java
import org.redisson.Redisson;
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;

public class RedissonLockExample {
    
    public static void main(String[] args) {
        // 创建Redisson客户端
        Config config = new Config();
        config.useSingleServer().setAddress("redis://127.0.0.1:6379");
        RedissonClient redisson = Redisson.create(config);
        
        // 获取锁对象
        RLock lock = redisson.getLock("myLock");
        
        try {
            // 加锁（自动续期）
            lock.lock();
            
            // 执行业务逻辑
            System.out.println("获取锁成功，执行业务逻辑");
            performCriticalOperation();
            
        } finally {
            // 释放锁
            lock.unlock();
        }
        
        redisson.shutdown();
    }
    
    private static void performCriticalOperation() {
        // 业务逻辑
    }
}
```

**带超时的锁**：
```java
// 方式1：指定锁的自动释放时间（10秒后自动释放）
lock.lock(10, TimeUnit.SECONDS);
try {
    performOperation();
} finally {
    lock.unlock();
}

// 方式2：尝试获取锁，等待最多100秒，锁定后10秒自动释放
boolean acquired = lock.tryLock(100, 10, TimeUnit.SECONDS);
if (acquired) {
    try {
        performOperation();
    } finally {
        lock.unlock();
    }
} else {
    System.out.println("无法在100秒内获取锁");
}
```

**检查锁状态**：
```java
// 检查锁是否被任意线程持有
boolean isLocked = lock.isLocked();

// 检查锁是否被当前线程持有
boolean isHeldByMe = lock.isHeldByCurrentThread();

// 获取锁的剩余有效时间（毫秒）
long ttl = lock.remainTimeToLive();
```


#### 2.1.2 看门狗机制 (⚠️ 难点)

**什么是看门狗（Watchdog）**：

看门狗是Redisson的核心机制之一，用于防止锁持有者崩溃导致锁永久无法释放。

**工作原理**：
1. 当使用`lock()`方法（不指定超时时间）获取锁时，Redisson会启动看门狗
2. 看门狗默认每10秒检查一次，如果锁仍被持有，则自动续期30秒
3. 默认锁的超时时间是30秒（可通过`Config.lockWatchdogTimeout`配置）
4. 当线程正常释放锁或线程崩溃时，看门狗会停止续期

**配置看门狗超时时间**：
```java
Config config = new Config();
config.setLockWatchdogTimeout(30000); // 设置为30秒（默认值）
config.useSingleServer().setAddress("redis://127.0.0.1:6379");
RedissonClient redisson = Redisson.create(config);
```

**看门狗的触发条件**：
```java
// 情况1：会启动看门狗（自动续期）
lock.lock();

// 情况2：不会启动看门狗（指定了超时时间）
lock.lock(10, TimeUnit.SECONDS);

// 情况3：会启动看门狗
boolean acquired = lock.tryLock();

// 情况4：不会启动看门狗（指定了leaseTime）
boolean acquired = lock.tryLock(100, 10, TimeUnit.SECONDS);
```

**⚠️ 注意事项**：
- 看门狗只在未指定`leaseTime`时生效
- 看门狗机制会增加Redis的负载（定期续期）
- 如果业务逻辑执行时间可预估，建议指定`leaseTime`避免看门狗开销
- 看门狗续期失败不会抛出异常，但锁可能会被其他线程获取

#### 2.1.3 读写锁（RReadWriteLock）

读写锁允许多个读操作并发执行，但写操作是互斥的。

```java
import org.redisson.api.RReadWriteLock;
import org.redisson.api.RLock;

public class ReadWriteLockExample {
    
    public void readData(RedissonClient redisson) {
        RReadWriteLock rwLock = redisson.getReadWriteLock("myRWLock");
        RLock readLock = rwLock.readLock();
        
        // 多个线程可以同时获取读锁
        readLock.lock();
        try {
            String data = performReadOperation();
            System.out.println("读取数据: " + data);
        } finally {
            readLock.unlock();
        }
    }
    
    public void writeData(RedissonClient redisson, String newData) {
        RReadWriteLock rwLock = redisson.getReadWriteLock("myRWLock");
        RLock writeLock = rwLock.writeLock();
        
        // 写锁是排他的，同一时间只有一个线程可以获取
        writeLock.lock();
        try {
            performWriteOperation(newData);
            System.out.println("写入数据: " + newData);
        } finally {
            writeLock.unlock();
        }
    }
    
    private String performReadOperation() {
        // 读取操作
        return "data";
    }
    
    private void performWriteOperation(String data) {
        // 写入操作
    }
}
```

**读写锁的特性**：
- 读锁可以被多个线程同时持有
- 写锁是排他的，持有写锁时不能有其他读锁或写锁
- 读锁和写锁之间互斥
- 适用于读多写少的场景

#### 2.1.4 公平锁（Fair Lock）

公平锁保证线程按照请求锁的顺序获取锁。

```java
RLock fairLock = redisson.getFairLock("myFairLock");
fairLock.lock();
try {
    // 业务逻辑
} finally {
    fairLock.unlock();
}
```

#### 2.1.5 联锁（MultiLock）

联锁允许同时锁定多个资源。

```java
RLock lock1 = redisson.getLock("lock1");
RLock lock2 = redisson.getLock("lock2");
RLock lock3 = redisson.getLock("lock3");

// 创建联锁
RLock multiLock = redisson.getMultiLock(lock1, lock2, lock3);

multiLock.lock();
try {
    // 同时持有三个锁
    performMultiResourceOperation();
} finally {
    multiLock.unlock();
}
```


### 2.2 分布式集合 🔥

Redisson提供了一系列分布式集合，这些集合实现了Java标准集合接口，可以在分布式环境中使用。

#### 2.2.1 分布式Map（RMap）

RMap实现了`java.util.concurrent.ConcurrentMap`接口。

```java
import org.redisson.api.RMap;

public class RMapExample {
    
    public void demonstrateRMap(RedissonClient redisson) {
        RMap<String, String> map = redisson.getMap("myMap");
        
        // 标准操作
        map.put("key1", "value1");
        String value = map.get("key1");
        map.putIfAbsent("key2", "value2");
        map.remove("key1");
        
        // 快速操作（不返回旧值，性能更好）
        map.fastPut("key3", "value3");
        map.fastPutIfAbsent("key4", "value4");
        map.fastRemove("key3");
        
        // 批量操作
        Map<String, String> entries = new HashMap<>();
        entries.put("key5", "value5");
        entries.put("key6", "value6");
        map.putAll(entries);
        
        // 异步操作
        RFuture<String> asyncValue = map.putAsync("key7", "value7");
        RFuture<Void> asyncFastPut = map.fastPutAsync("key8", "value8");
        
        // 遍历
        for (Map.Entry<String, String> entry : map.entrySet()) {
            System.out.println(entry.getKey() + " = " + entry.getValue());
        }
    }
}
```

**RMap的高级特性**：
```java
// 设置过期时间
map.put("tempKey", "tempValue", 10, TimeUnit.SECONDS);

// 获取并删除
String removedValue = map.remove("key1");

// 原子操作
map.replace("key1", "oldValue", "newValue");

// 计算操作
map.compute("counter", (key, value) -> {
    int count = value == null ? 0 : Integer.parseInt(value);
    return String.valueOf(count + 1);
});
```

#### 2.2.2 分布式Set（RSet）

RSet实现了`java.util.Set`接口。

```java
import org.redisson.api.RSet;

public class RSetExample {
    
    public void demonstrateRSet(RedissonClient redisson) {
        RSet<String> set = redisson.getSet("mySet");
        
        // 标准集合操作
        set.add("element1");
        set.add("element2");
        boolean contains = set.contains("element1");
        set.remove("element1");
        
        // 集合运算
        RSet<String> set2 = redisson.getSet("mySet2");
        set2.addAll(Arrays.asList("element2", "element3", "element4"));
        
        // 交集
        Set<String> intersection = set.readIntersection("mySet2");
        
        // 并集
        Set<String> union = set.readUnion("mySet2");
        
        // 差集
        Set<String> diff = set.readDiff("mySet2");
        
        // 批量操作
        set.addAll(Arrays.asList("element5", "element6", "element7"));
        
        // 遍历
        for (String element : set) {
            System.out.println(element);
        }
    }
}
```

#### 2.2.3 分布式List（RList）

RList实现了`java.util.List`接口，保持元素的插入顺序。

```java
import org.redisson.api.RList;

public class RListExample {
    
    public void demonstrateRList(RedissonClient redisson) {
        RList<String> list = redisson.getList("myList");
        
        // 标准列表操作
        list.add("item1");
        list.add("item2");
        list.add(0, "item0"); // 在索引0处插入
        
        String first = list.get(0);
        list.remove("item1");
        list.remove(0); // 按索引删除
        
        // 批量操作
        list.addAll(Arrays.asList("item3", "item4", "item5"));
        List<String> subList = list.subList(0, 3);
        
        // 排序
        list.sort(Comparator.naturalOrder());
        
        // 异步操作
        RFuture<Boolean> addFuture = list.addAsync("item6");
        
        // 遍历
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
    }
}
```

#### 2.2.4 分布式Queue（RQueue）

RQueue实现了`java.util.Queue`接口。

```java
import org.redisson.api.RQueue;

public class RQueueExample {
    
    public void demonstrateRQueue(RedissonClient redisson) {
        RQueue<String> queue = redisson.getQueue("myQueue");
        
        // 入队
        queue.offer("task1");
        queue.offer("task2");
        queue.offer("task3");
        
        // 出队
        String task = queue.poll();
        System.out.println("处理任务: " + task);
        
        // 查看队首元素（不移除）
        String peek = queue.peek();
        
        // 批量操作
        queue.addAll(Arrays.asList("task4", "task5", "task6"));
        
        // 获取队列大小
        int size = queue.size();
    }
}
```


### 2.3 分布式对象

#### 2.3.1 对象桶（RBucket）

RBucket用于存储任意类型的对象。

```java
import org.redisson.api.RBucket;

public class RBucketExample {
    
    public void demonstrateRBucket(RedissonClient redisson) {
        RBucket<User> bucket = redisson.getBucket("user:1001");
        
        // 存储对象
        User user = new User("张三", 25);
        bucket.set(user);
        
        // 设置过期时间
        bucket.set(user, 10, TimeUnit.MINUTES);
        
        // 获取对象
        User retrievedUser = bucket.get();
        
        // 如果不存在则设置
        boolean wasSet = bucket.setIfAbsent(user);
        
        // 获取并删除
        User deletedUser = bucket.getAndDelete();
        
        // 比较并设置
        bucket.compareAndSet(user, new User("李四", 30));
    }
}

class User {
    private String name;
    private int age;
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // getters and setters
}
```

#### 2.3.2 原子长整型（RAtomicLong）

RAtomicLong提供分布式的原子操作。

```java
import org.redisson.api.RAtomicLong;

public class RAtomicLongExample {
    
    public void demonstrateRAtomicLong(RedissonClient redisson) {
        RAtomicLong atomicLong = redisson.getAtomicLong("myCounter");
        
        // 设置初始值
        atomicLong.set(100);
        
        // 自增
        long newValue = atomicLong.incrementAndGet();
        
        // 自减
        long decremented = atomicLong.decrementAndGet();
        
        // 增加指定值
        long added = atomicLong.addAndGet(10);
        
        // 比较并设置
        boolean updated = atomicLong.compareAndSet(110, 200);
        
        // 获取当前值
        long currentValue = atomicLong.get();
    }
}
```

#### 2.3.3 布隆过滤器（RBloomFilter）

布隆过滤器用于快速判断元素是否存在，适用于海量数据去重。

```java
import org.redisson.api.RBloomFilter;

public class RBloomFilterExample {
    
    public void demonstrateRBloomFilter(RedissonClient redisson) {
        RBloomFilter<String> bloomFilter = redisson.getBloomFilter("myBloomFilter");
        
        // 初始化布隆过滤器
        // 预期元素数量：100000，误判率：0.01
        bloomFilter.tryInit(100000L, 0.01);
        
        // 添加元素
        bloomFilter.add("user:1001");
        bloomFilter.add("user:1002");
        bloomFilter.add("user:1003");
        
        // 判断元素是否存在
        boolean exists = bloomFilter.contains("user:1001"); // true
        boolean notExists = bloomFilter.contains("user:9999"); // 可能为true（误判）
        
        // 获取已添加的元素数量（估算值）
        long count = bloomFilter.count();
    }
}
```

### 2.4 配置与连接模式 (⚠️ 难点)

#### 2.4.1 单机模式

```java
import org.redisson.Redisson;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;

public class SingleServerConfig {
    
    public RedissonClient createClient() {
        // 默认连接到 localhost:6379
        RedissonClient redisson = Redisson.create();
        
        // 自定义配置
        Config config = new Config();
        config.useSingleServer()
            .setAddress("redis://127.0.0.1:6379")
            .setPassword("myPassword")
            .setDatabase(0)
            .setConnectionPoolSize(64)          // 连接池大小
            .setConnectionMinimumIdleSize(10)   // 最小空闲连接数
            .setIdleConnectionTimeout(10000)    // 空闲连接超时时间
            .setConnectTimeout(10000)           // 连接超时时间
            .setTimeout(3000)                   // 命令执行超时时间
            .setRetryAttempts(3)                // 重试次数
            .setRetryInterval(1500);            // 重试间隔
        
        return Redisson.create(config);
    }
}
```

#### 2.4.2 集群模式

```java
public class ClusterConfig {
    
    public RedissonClient createClusterClient() {
        Config config = new Config();
        config.useClusterServers()
            .addNodeAddress("redis://127.0.0.1:7000")
            .addNodeAddress("redis://127.0.0.1:7001")
            .addNodeAddress("redis://127.0.0.1:7002")
            .setPassword("myPassword")
            .setMasterConnectionPoolSize(64)
            .setSlaveConnectionPoolSize(64)
            .setIdleConnectionTimeout(10000)
            .setConnectTimeout(10000)
            .setTimeout(3000)
            .setRetryAttempts(3)
            .setRetryInterval(1500);
        
        return Redisson.create(config);
    }
}
```


#### 2.4.3 哨兵模式

```java
public class SentinelConfig {
    
    public RedissonClient createSentinelClient() {
        Config config = new Config();
        config.useSentinelServers()
            .setMasterName("mymaster")
            .addSentinelAddress("redis://127.0.0.1:26379")
            .addSentinelAddress("redis://127.0.0.1:26380")
            .addSentinelAddress("redis://127.0.0.1:26381")
            .setPassword("myPassword")
            .setMasterConnectionPoolSize(64)
            .setSlaveConnectionPoolSize(64)
            .setIdleConnectionTimeout(10000)
            .setConnectTimeout(10000)
            .setTimeout(3000);
        
        return Redisson.create(config);
    }
}
```

#### 2.4.4 高级配置

```java
public class AdvancedConfig {
    
    public RedissonClient createAdvancedClient() {
        Config config = new Config();
        
        // 传输模式配置
        config.setTransportMode(TransportMode.EPOLL); // EPOLL, NIO, KQUEUE
        
        // 线程池配置
        config.setThreads(16);          // 执行器线程数
        config.setNettyThreads(32);     // Netty线程数
        
        // 编解码器配置
        config.setCodec(new org.redisson.codec.Kryo5Codec()); // 序列化方式
        
        // 看门狗配置
        config.setLockWatchdogTimeout(30000); // 看门狗超时时间（30秒）
        
        // 从节点同步检查
        config.setCheckLockSyncedSlaves(true);
        
        // 单机服务器配置
        config.useSingleServer()
            .setAddress("redis://127.0.0.1:6379")
            .setRetryAttempts(3)
            .setRetryInterval(1500)
            .setTimeout(3000)
            .setConnectTimeout(10000);
        
        return Redisson.create(config);
    }
}
```

### 2.5 异步与响应式编程

#### 2.5.1 异步API

```java
import org.redisson.api.RFuture;

public class AsyncExample {
    
    public void demonstrateAsync(RedissonClient redisson) {
        RMap<String, String> map = redisson.getMap("myMap");
        
        // 异步put操作
        RFuture<String> putFuture = map.putAsync("key1", "value1");
        
        // 处理异步结果
        putFuture.whenComplete((result, exception) -> {
            if (exception == null) {
                System.out.println("Put成功，旧值: " + result);
            } else {
                System.err.println("Put失败: " + exception.getMessage());
            }
        });
        
        // 异步get操作
        RFuture<String> getFuture = map.getAsync("key1");
        getFuture.thenAccept(value -> {
            System.out.println("获取到的值: " + value);
        });
        
        // 链式异步操作
        map.putAsync("key2", "value2")
            .thenCompose(v -> map.getAsync("key2"))
            .thenAccept(value -> System.out.println("最终值: " + value));
    }
}
```

#### 2.5.2 响应式API（Reactive）

```java
import org.redisson.api.RedissonReactiveClient;
import org.redisson.api.RMapReactive;
import reactor.core.publisher.Mono;

public class ReactiveExample {
    
    public void demonstrateReactive(RedissonClient redisson) {
        RedissonReactiveClient reactiveClient = redisson.reactive();
        RMapReactive<String, String> map = reactiveClient.getMap("myMap");
        
        // 响应式put操作
        Mono<String> putMono = map.put("key1", "value1");
        
        putMono.doOnSuccess(oldValue -> {
            System.out.println("Put成功，旧值: " + oldValue);
        }).subscribe();
        
        // 响应式get操作
        Mono<String> getMono = map.get("key1");
        getMono.subscribe(value -> {
            System.out.println("获取到的值: " + value);
        });
        
        // 链式响应式操作
        map.put("key2", "value2")
            .then(map.get("key2"))
            .subscribe(value -> System.out.println("最终值: " + value));
    }
}
```

## 💻 实战应用

### 3.1 环境搭建

#### 3.1.1 Maven依赖

```xml
<dependencies>
    <!-- Redisson核心依赖 -->
    <dependency>
        <groupId>org.redisson</groupId>
        <artifactId>redisson</artifactId>
        <version>3.25.0</version>
    </dependency>
    
    <!-- Spring Boot集成（可选） -->
    <dependency>
        <groupId>org.redisson</groupId>
        <artifactId>redisson-spring-boot-starter</artifactId>
        <version>3.25.0</version>
    </dependency>
</dependencies>
```

#### 3.1.2 Spring Boot配置

**application.yml**:
```yaml
spring:
  redis:
    redisson:
      config: classpath:redisson.yaml
```

**redisson.yaml**:
```yaml
singleServerConfig:
  address: "redis://127.0.0.1:6379"
  password: null
  database: 0
  connectionPoolSize: 64
  connectionMinimumIdleSize: 10
  idleConnectionTimeout: 10000
  connectTimeout: 10000
  timeout: 3000
  retryAttempts: 3
  retryInterval: 1500
  
lockWatchdogTimeout: 30000
threads: 16
nettyThreads: 32
```


### 3.2 实战案例1：分布式锁防止重复下单

```java
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 订单服务
 * @author erik.zhou
 */
@Service
public class OrderService {
    
    @Autowired
    private RedissonClient redissonClient;
    
    /**
     * 创建订单（防止重复提交）
     */
    public String createOrder(Long userId, Long productId) {
        // 使用用户ID和商品ID作为锁的key
        String lockKey = "order:lock:" + userId + ":" + productId;
        RLock lock = redissonClient.getLock(lockKey);
        
        try {
            // 尝试获取锁，最多等待10秒，锁定后30秒自动释放
            boolean acquired = lock.tryLock(10, 30, TimeUnit.SECONDS);
            
            if (!acquired) {
                throw new RuntimeException("系统繁忙，请稍后重试");
            }
            
            // 检查是否已经下过单
            if (checkOrderExists(userId, productId)) {
                throw new RuntimeException("订单已存在，请勿重复提交");
            }
            
            // 创建订单
            String orderId = generateOrderId();
            saveOrder(orderId, userId, productId);
            
            return orderId;
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("获取锁被中断", e);
        } finally {
            // 释放锁（只有锁的持有者才能释放）
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
            }
        }
    }
    
    private boolean checkOrderExists(Long userId, Long productId) {
        // 检查订单是否存在
        return false;
    }
    
    private String generateOrderId() {
        // 生成订单ID
        return "ORDER" + System.currentTimeMillis();
    }
    
    private void saveOrder(String orderId, Long userId, Long productId) {
        // 保存订单到数据库
    }
}
```

### 3.3 实战案例2：分布式限流

```java
import org.redisson.api.RRateLimiter;
import org.redisson.api.RateIntervalUnit;
import org.redisson.api.RateType;
import org.redisson.api.RedissonClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * 限流器
 * @author erik.zhou
 */
@Component
public class RateLimiterService {
    
    @Autowired
    private RedissonClient redissonClient;
    
    /**
     * API限流：每秒最多10个请求
     */
    public boolean tryAcquire(String apiName) {
        String limiterKey = "rate:limiter:" + apiName;
        RRateLimiter rateLimiter = redissonClient.getRateLimiter(limiterKey);
        
        // 初始化限流器：每秒最多10个令牌
        rateLimiter.trySetRate(RateType.OVERALL, 10, 1, RateIntervalUnit.SECONDS);
        
        // 尝试获取1个令牌
        return rateLimiter.tryAcquire(1);
    }
    
    /**
     * 用户级别限流：每分钟最多100个请求
     */
    public boolean tryAcquireForUser(Long userId) {
        String limiterKey = "rate:limiter:user:" + userId;
        RRateLimiter rateLimiter = redissonClient.getRateLimiter(limiterKey);
        
        // 初始化限流器：每分钟最多100个令牌
        rateLimiter.trySetRate(RateType.OVERALL, 100, 1, RateIntervalUnit.MINUTES);
        
        return rateLimiter.tryAcquire(1);
    }
}

/**
 * API控制器
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api")
public class ApiController {
    
    @Autowired
    private RateLimiterService rateLimiterService;
    
    @GetMapping("/data")
    public Result getData() {
        // 检查限流
        if (!rateLimiterService.tryAcquire("getData")) {
            return Result.error("请求过于频繁，请稍后重试");
        }
        
        // 处理业务逻辑
        return Result.success(fetchData());
    }
    
    private Object fetchData() {
        // 获取数据
        return new Object();
    }
}
```

### 3.4 实战案例3：分布式缓存

```java
import org.redisson.api.RBucket;
import org.redisson.api.RedissonClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 用户缓存服务
 * @author erik.zhou
 */
@Service
public class UserCacheService {
    
    @Autowired
    private RedissonClient redissonClient;
    
    @Autowired
    private UserMapper userMapper;
    
    /**
     * 获取用户信息（带缓存）
     */
    public User getUserById(Long userId) {
        String cacheKey = "user:cache:" + userId;
        RBucket<User> bucket = redissonClient.getBucket(cacheKey);
        
        // 先从缓存获取
        User user = bucket.get();
        if (user != null) {
            return user;
        }
        
        // 缓存未命中，从数据库查询
        user = userMapper.selectById(userId);
        if (user != null) {
            // 写入缓存，设置过期时间为10分钟
            bucket.set(user, 10, TimeUnit.MINUTES);
        }
        
        return user;
    }
    
    /**
     * 更新用户信息（同时更新缓存）
     */
    public void updateUser(User user) {
        // 更新数据库
        userMapper.updateById(user);
        
        // 更新缓存
        String cacheKey = "user:cache:" + user.getId();
        RBucket<User> bucket = redissonClient.getBucket(cacheKey);
        bucket.set(user, 10, TimeUnit.MINUTES);
    }
    
    /**
     * 删除用户（同时删除缓存）
     */
    public void deleteUser(Long userId) {
        // 删除数据库记录
        userMapper.deleteById(userId);
        
        // 删除缓存
        String cacheKey = "user:cache:" + userId;
        redissonClient.getBucket(cacheKey).delete();
    }
}
```


### 3.5 实战案例4：分布式信号量（限制并发数）

```java
import org.redisson.api.RSemaphore;
import org.redisson.api.RedissonClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 资源池服务（限制并发访问数量）
 * @author erik.zhou
 */
@Service
public class ResourcePoolService {
    
    @Autowired
    private RedissonClient redissonClient;
    
    /**
     * 限制同时处理的任务数量（例如：最多10个并发）
     */
    public void processTask(String taskId) {
        String semaphoreKey = "semaphore:task:pool";
        RSemaphore semaphore = redissonClient.getSemaphore(semaphoreKey);
        
        try {
            // 设置信号量许可数为10
            semaphore.trySetPermits(10);
            
            // 尝试获取许可，最多等待5秒
            boolean acquired = semaphore.tryAcquire(5, TimeUnit.SECONDS);
            
            if (!acquired) {
                throw new RuntimeException("系统繁忙，请稍后重试");
            }
            
            // 执行任务
            System.out.println("开始处理任务: " + taskId);
            performTask(taskId);
            System.out.println("任务处理完成: " + taskId);
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("获取许可被中断", e);
        } finally {
            // 释放许可
            semaphore.release();
        }
    }
    
    private void performTask(String taskId) {
        // 模拟任务处理
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

### 3.6 实战案例5：分布式CountDownLatch

```java
import org.redisson.api.RCountDownLatch;
import org.redisson.api.RedissonClient;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 批量任务协调服务
 * @author erik.zhou
 */
@Service
public class BatchTaskService {
    
    @Autowired
    private RedissonClient redissonClient;
    
    /**
     * 主线程：等待所有子任务完成
     */
    public void waitForAllTasks(String batchId, int taskCount) {
        String latchKey = "latch:batch:" + batchId;
        RCountDownLatch latch = redissonClient.getCountDownLatch(latchKey);
        
        try {
            // 设置计数器
            latch.trySetCount(taskCount);
            
            // 等待所有任务完成（最多等待60秒）
            boolean completed = latch.await(60, TimeUnit.SECONDS);
            
            if (completed) {
                System.out.println("所有任务已完成");
                processBatchResult(batchId);
            } else {
                System.out.println("部分任务超时未完成");
            }
            
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            throw new RuntimeException("等待被中断", e);
        }
    }
    
    /**
     * 子任务：完成后递减计数器
     */
    public void completeTask(String batchId, String taskId) {
        // 执行任务
        performTask(taskId);
        
        // 递减计数器
        String latchKey = "latch:batch:" + batchId;
        RCountDownLatch latch = redissonClient.getCountDownLatch(latchKey);
        latch.countDown();
        
        System.out.println("任务完成: " + taskId);
    }
    
    private void performTask(String taskId) {
        // 执行任务逻辑
    }
    
    private void processBatchResult(String batchId) {
        // 处理批量任务结果
    }
}
```

## ✨ 最佳实践

### 4.1 分布式锁使用最佳实践

#### 4.1.1 锁的粒度控制

```java
// ❌ 错误：锁粒度过大
public void badExample(RedissonClient redisson) {
    RLock lock = redisson.getLock("global:lock");
    lock.lock();
    try {
        processOrder();
        processPayment();
        processShipment();
    } finally {
        lock.unlock();
    }
}

// ✅ 正确：锁粒度细化
public void goodExample(RedissonClient redisson, String orderId) {
    RLock lock = redisson.getLock("order:lock:" + orderId);
    lock.lock();
    try {
        processOrder(orderId);
    } finally {
        lock.unlock();
    }
}
```

#### 4.1.2 避免死锁

```java
// ❌ 错误：可能导致死锁
public void badExample(RedissonClient redisson) {
    RLock lock1 = redisson.getLock("lock1");
    RLock lock2 = redisson.getLock("lock2");
    
    lock1.lock();
    // 如果这里获取lock2失败，lock1永远不会释放
    lock2.lock();
    try {
        // 业务逻辑
    } finally {
        lock2.unlock();
        lock1.unlock();
    }
}

// ✅ 正确：使用tryLock避免死锁
public void goodExample(RedissonClient redisson) {
    RLock lock1 = redisson.getLock("lock1");
    RLock lock2 = redisson.getLock("lock2");
    
    try {
        boolean acquired1 = lock1.tryLock(10, TimeUnit.SECONDS);
        if (!acquired1) {
            throw new RuntimeException("获取lock1失败");
        }
        
        boolean acquired2 = lock2.tryLock(10, TimeUnit.SECONDS);
        if (!acquired2) {
            lock1.unlock();
            throw new RuntimeException("获取lock2失败");
        }
        
        // 业务逻辑
        
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        if (lock2.isHeldByCurrentThread()) {
            lock2.unlock();
        }
        if (lock1.isHeldByCurrentThread()) {
            lock1.unlock();
        }
    }
}

// ✅ 更好：使用MultiLock
public void betterExample(RedissonClient redisson) {
    RLock lock1 = redisson.getLock("lock1");
    RLock lock2 = redisson.getLock("lock2");
    RLock multiLock = redisson.getMultiLock(lock1, lock2);
    
    try {
        boolean acquired = multiLock.tryLock(10, 30, TimeUnit.SECONDS);
        if (!acquired) {
            throw new RuntimeException("获取锁失败");
        }
        
        // 业务逻辑
        
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        multiLock.unlock();
    }
}
```


#### 4.1.3 合理设置超时时间

```java
// ❌ 错误：不设置超时时间，可能永久阻塞
public void badExample(RedissonClient redisson) {
    RLock lock = redisson.getLock("myLock");
    lock.lock(); // 可能永久等待
    try {
        // 业务逻辑
    } finally {
        lock.unlock();
    }
}

// ✅ 正确：设置合理的超时时间
public void goodExample(RedissonClient redisson) {
    RLock lock = redisson.getLock("myLock");
    try {
        // 等待最多10秒，锁定后30秒自动释放
        boolean acquired = lock.tryLock(10, 30, TimeUnit.SECONDS);
        if (!acquired) {
            throw new RuntimeException("获取锁超时");
        }
        
        // 业务逻辑
        
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        if (lock.isHeldByCurrentThread()) {
            lock.unlock();
        }
    }
}
```

#### 4.1.4 正确释放锁

```java
// ❌ 错误：未检查锁持有者就释放
public void badExample(RedissonClient redisson) {
    RLock lock = redisson.getLock("myLock");
    lock.lock();
    try {
        // 业务逻辑
    } finally {
        lock.unlock(); // 可能抛出IllegalMonitorStateException
    }
}

// ✅ 正确：检查锁持有者再释放
public void goodExample(RedissonClient redisson) {
    RLock lock = redisson.getLock("myLock");
    lock.lock();
    try {
        // 业务逻辑
    } finally {
        if (lock.isHeldByCurrentThread()) {
            lock.unlock();
        }
    }
}
```

### 4.2 性能优化

#### 4.2.1 使用fast操作

```java
// 普通操作会返回旧值，性能较低
String oldValue = map.put("key", "value");

// fast操作不返回旧值，性能更好
map.fastPut("key", "value");
```

#### 4.2.2 批量操作

```java
// ❌ 错误：循环单次操作
for (String key : keys) {
    map.put(key, getValue(key));
}

// ✅ 正确：批量操作
Map<String, String> batch = new HashMap<>();
for (String key : keys) {
    batch.put(key, getValue(key));
}
map.putAll(batch);
```

#### 4.2.3 使用异步操作

```java
// 同步操作（阻塞）
for (int i = 0; i < 100; i++) {
    map.put("key" + i, "value" + i);
}

// 异步操作（非阻塞）
List<RFuture<String>> futures = new ArrayList<>();
for (int i = 0; i < 100; i++) {
    RFuture<String> future = map.putAsync("key" + i, "value" + i);
    futures.add(future);
}

// 等待所有操作完成
for (RFuture<String> future : futures) {
    future.get();
}
```

### 4.3 连接池配置优化

```java
Config config = new Config();
config.useSingleServer()
    .setAddress("redis://127.0.0.1:6379")
    // 连接池大小（根据并发量调整）
    .setConnectionPoolSize(64)
    // 最小空闲连接数
    .setConnectionMinimumIdleSize(10)
    // 空闲连接超时时间（毫秒）
    .setIdleConnectionTimeout(10000)
    // 连接超时时间（毫秒）
    .setConnectTimeout(10000)
    // 命令执行超时时间（毫秒）
    .setTimeout(3000)
    // 重试次数
    .setRetryAttempts(3)
    // 重试间隔（毫秒）
    .setRetryInterval(1500);
```

### 4.4 序列化优化

```java
// 使用高性能序列化方式
Config config = new Config();
config.setCodec(new org.redisson.codec.Kryo5Codec()); // Kryo序列化
// 或者
config.setCodec(new org.redisson.codec.FstCodec()); // FST序列化
// 或者
config.setCodec(new org.redisson.codec.SnappyCodecV2()); // Snappy压缩
```

## ⚠️ 常见陷阱

### 4.5.1 看门狗失效

```java
// ⚠️ 陷阱：指定了leaseTime，看门狗不会生效
RLock lock = redisson.getLock("myLock");
lock.lock(10, TimeUnit.SECONDS); // 看门狗不会续期
try {
    // 如果业务逻辑执行超过10秒，锁会自动释放
    longRunningTask(); // 可能执行20秒
} finally {
    lock.unlock();
}

// ✅ 解决方案1：不指定leaseTime，让看门狗自动续期
lock.lock(); // 看门狗会自动续期
try {
    longRunningTask();
} finally {
    lock.unlock();
}

// ✅ 解决方案2：指定足够长的leaseTime
lock.lock(60, TimeUnit.SECONDS); // 确保业务逻辑能在60秒内完成
try {
    longRunningTask();
} finally {
    lock.unlock();
}
```

### 4.5.2 锁重入问题

```java
// ⚠️ 陷阱：同一线程多次获取锁，需要相同次数的unlock
RLock lock = redisson.getLock("myLock");
lock.lock(); // 第1次加锁
lock.lock(); // 第2次加锁（重入）
try {
    // 业务逻辑
} finally {
    lock.unlock(); // 第1次解锁
    lock.unlock(); // 第2次解锁（必须）
}
```

### 4.5.3 集合操作的原子性

```java
// ⚠️ 陷阱：多个操作不是原子的
RMap<String, Integer> map = redisson.getMap("myMap");
Integer value = map.get("counter");
if (value == null) {
    value = 0;
}
map.put("counter", value + 1); // 非原子操作，可能导致并发问题

// ✅ 解决方案：使用原子操作
map.compute("counter", (key, oldValue) -> {
    return oldValue == null ? 1 : oldValue + 1;
});
```


## ❓ 常见问题

### Q1: Redisson和Jedis/Lettuce有什么区别？

**A**: 
- **定位不同**: Jedis/Lettuce是Redis客户端，提供基础的Redis命令操作；Redisson是分布式框架，提供高级的分布式对象和服务
- **API风格**: Jedis/Lettuce是命令式API；Redisson是面向对象的API
- **功能丰富度**: Redisson内置了分布式锁、分布式集合、分布式对象等高级功能
- **学习成本**: Jedis最简单，Lettuce中等，Redisson较复杂
- **使用场景**: 简单的Redis操作用Jedis/Lettuce；复杂的分布式场景用Redisson

### Q2: 看门狗机制是如何工作的？

**A**: 
1. 当使用`lock()`方法（不指定leaseTime）获取锁时，Redisson会启动看门狗
2. 看门狗默认每10秒检查一次（`lockWatchdogTimeout / 3`）
3. 如果锁仍被当前线程持有，看门狗会将锁的过期时间续期到30秒（默认值）
4. 当线程正常释放锁或线程崩溃时，看门狗会停止续期
5. 看门狗只在未指定`leaseTime`时生效

### Q3: 如何选择合适的锁类型？

**A**:
- **RLock（可重入锁）**: 最常用，适用于大多数场景
- **RReadWriteLock（读写锁）**: 读多写少的场景，允许多个读操作并发
- **RFairLock（公平锁）**: 需要保证线程获取锁的顺序时使用
- **RMultiLock（联锁）**: 需要同时锁定多个资源时使用
- **RSemaphore（信号量）**: 限制并发访问数量时使用

### Q4: Redisson的性能如何？

**A**:
- Redisson基于Netty，性能优秀
- 支持异步和响应式编程，可以充分利用系统资源
- 使用连接池管理连接，减少连接开销
- 支持批量操作，减少网络往返次数
- 可以通过配置优化（连接池大小、序列化方式等）进一步提升性能

### Q5: 分布式锁会有什么问题？

**A**:
1. **锁超时问题**: 业务逻辑执行时间超过锁的超时时间，导致锁被自动释放
   - 解决方案：使用看门狗机制或设置足够长的超时时间
2. **死锁问题**: 多个锁之间相互等待
   - 解决方案：使用tryLock设置超时时间，或使用MultiLock
3. **锁误删问题**: 线程A的锁被线程B误删
   - 解决方案：Redisson已经通过线程ID解决了这个问题
4. **主从切换问题**: Redis主节点宕机，从节点还未同步锁数据
   - 解决方案：使用RedLock算法（Redisson支持）

### Q6: 如何处理Redis连接失败？

**A**:
```java
Config config = new Config();
config.useSingleServer()
    .setAddress("redis://127.0.0.1:6379")
    .setRetryAttempts(3)        // 重试3次
    .setRetryInterval(1500)     // 重试间隔1.5秒
    .setConnectTimeout(10000);  // 连接超时10秒

// 使用try-catch捕获异常
try {
    RLock lock = redisson.getLock("myLock");
    lock.lock();
    // 业务逻辑
} catch (RedisException e) {
    // 处理Redis异常
    logger.error("Redis操作失败", e);
    // 降级处理
} finally {
    // 释放资源
}
```

### Q7: Redisson支持哪些Redis部署模式？

**A**:
- **单机模式**: `config.useSingleServer()`
- **主从模式**: `config.useMasterSlaveServers()`
- **哨兵模式**: `config.useSentinelServers()`
- **集群模式**: `config.useClusterServers()`
- **云服务模式**: 支持AWS、Azure、Google Cloud等

### Q8: 如何监控Redisson的运行状态？

**A**:
```java
// 获取Redisson客户端信息
NodesGroup nodesGroup = redisson.getNodesGroup();
Collection<Node> nodes = nodesGroup.getNodes();

for (Node node : nodes) {
    // 获取节点信息
    Map<String, String> info = node.info(InfoSection.ALL);
    
    // 获取节点统计信息
    long usedMemory = node.getMemoryStatistics().getUsedMemory();
    
    // 执行ping命令
    boolean isAlive = node.ping();
}

// 获取连接池信息
Config config = redisson.getConfig();
int poolSize = config.useSingleServer().getConnectionPoolSize();
```

### Q9: Redisson的序列化方式有哪些？

**A**:
- **JsonJacksonCodec**: JSON序列化（默认）
- **Kryo5Codec**: Kryo序列化（性能好）
- **FstCodec**: FST序列化（性能好）
- **SnappyCodecV2**: Snappy压缩
- **MsgPackJacksonCodec**: MessagePack序列化
- **SerializationCodec**: Java原生序列化（不推荐）

### Q10: 如何优雅地关闭Redisson客户端？

**A**:
```java
// 方式1：同步关闭
redisson.shutdown();

// 方式2：异步关闭
redisson.shutdown(2, 5, TimeUnit.SECONDS);
// 参数说明：
// - quietPeriod: 2秒，在此期间不接受新任务
// - timeout: 5秒，最多等待5秒后强制关闭

// Spring Boot中使用@PreDestroy
@Component
public class RedissonManager {
    
    @Autowired
    private RedissonClient redisson;
    
    @PreDestroy
    public void destroy() {
        if (redisson != null && !redisson.isShutdown()) {
            redisson.shutdown();
        }
    }
}
```

## 🔗 相关资源

### 官方资源
- **官方GitHub**: https://github.com/redisson/redisson
- **官方文档**: https://github.com/redisson/redisson/wiki
- **API文档**: https://www.javadoc.io/doc/org.redisson/redisson/latest/index.html
- **配置说明**: https://github.com/redisson/redisson/wiki/2.-Configuration

### 推荐文章
- Redisson分布式锁实现原理
- Redisson看门狗机制详解
- Redisson性能优化实践
- Redis分布式锁的正确姿势

### 视频教程
- Redisson入门到精通
- 分布式锁实战教程
- Redisson源码解析

### 相关技术
- Redis基础教程
- 分布式系统设计
- Java并发编程
- Spring Boot集成

## 📝 学习检查清单

### 基础知识
- [ ] 理解Redisson的核心概念和定位
- [ ] 掌握Redisson与Jedis/Lettuce的区别
- [ ] 了解Redisson的应用场景

### 分布式锁
- [ ] 掌握RLock的基本使用
- [ ] 理解看门狗机制的工作原理
- [ ] 掌握读写锁的使用场景
- [ ] 了解公平锁和联锁的使用

### 分布式集合
- [ ] 掌握RMap的使用和高级特性
- [ ] 掌握RSet的集合运算
- [ ] 掌握RList的列表操作
- [ ] 了解RQueue的队列操作

### 分布式对象
- [ ] 掌握RBucket的对象存储
- [ ] 掌握RAtomicLong的原子操作
- [ ] 了解RBloomFilter的使用场景

### 配置与优化
- [ ] 掌握单机、集群、哨兵模式的配置
- [ ] 理解连接池配置参数
- [ ] 掌握性能优化技巧
- [ ] 了解序列化方式的选择

### 实战应用
- [ ] 能够使用分布式锁防止重复提交
- [ ] 能够实现分布式限流
- [ ] 能够实现分布式缓存
- [ ] 能够使用信号量限制并发数

### 最佳实践
- [ ] 掌握锁的粒度控制
- [ ] 掌握避免死锁的方法
- [ ] 掌握正确释放锁的方式
- [ ] 了解常见陷阱和解决方案

---

**学习建议**：
1. 先掌握基础的分布式锁使用
2. 深入理解看门狗机制
3. 实践各种分布式集合的使用
4. 在实际项目中应用Redisson解决分布式问题
5. 关注性能优化和最佳实践

**下一步学习**：
- 深入学习Redis原理
- 学习分布式系统设计
- 学习Spring Cloud微服务架构
- 学习分布式事务处理

