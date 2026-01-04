# Redis 完整教程

## 📋 目录
- 技术概述
- 学习目标
- 基础概念
- 核心特性（重点）
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: Redis 7.x
- **官方文档**: https://redis.io/docs/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 数据结构基础、缓存概念
- **文档来源**: Context7 - Redis Documentation
- **更新时间**: 2024-12-31

## 🎯 学习目标
- [ ] 掌握Redis五大基本数据类型和应用场景
- [ ] 理解Redis持久化机制（RDB和AOF）
- [ ] 掌握Redis集群和高可用方案
- [ ] 解决缓存穿透、击穿、雪崩问题
- [ ] 实现分布式锁

## 📖 基础概念

### 1.1 什么是Redis
Redis（Remote Dictionary Server）是一个开源的内存数据存储系统，被数百万开发者用作：
- **缓存**：提高应用性能
- **向量数据库**：支持AI应用
- **文档数据库**：存储JSON文档
- **流引擎**：处理实时数据流
- **消息代理**：实现发布订阅

Redis支持复杂的数据类型（字符串、哈希、列表、集合、有序集合、JSON等），并提供原子操作。内置复制功能和多种持久化选项。

### 1.2 核心概念
- **内存存储**：所有数据存储在内存中，读写速度极快
- **持久化**：支持RDB快照和AOF日志两种持久化方式
- **数据结构**：提供丰富的数据类型，不仅仅是简单的key-value
- **单线程模型**：使用单线程处理命令，避免并发问题
- **主从复制**：支持数据复制，实现读写分离
- **集群模式**：支持分布式部署，提供高可用和水平扩展

### 1.3 应用场景
- **缓存系统**：缓存热点数据，减轻数据库压力
- **会话存储**：存储用户会话信息
- **排行榜**：使用有序集合实现实时排行
- **计数器**：网站访问量、点赞数等
- **分布式锁**：实现分布式系统的互斥访问
- **消息队列**：使用列表或Stream实现简单消息队列
- **实时分析**：使用HyperLogLog进行基数统计


## 🔥 核心特性 (重点)

### 2.1 数据类型 🔥

#### 2.1.1 String（字符串）
最基本的数据类型，可以存储字符串、整数、浮点数。

```redis
# 设置和获取
SET name "erik"
GET name

# 数值操作
SET counter 100
INCR counter        # 自增1，返回101
INCRBY counter 10   # 增加10，返回111
DECR counter        # 自减1，返回110

# 设置过期时间
SETEX session:token 3600 "abc123"  # 设置3600秒过期
```

**应用场景**：缓存、计数器、分布式锁、限流

#### 2.1.2 Hash（哈希）
键值对集合，适合存储对象。

```redis
# 设置和获取
HSET user:1001 name "erik" age 25 email "erik@example.com"
HGET user:1001 name
HGETALL user:1001

# 数值操作
HINCRBY user:1001 age 1  # 年龄增加1
```

**应用场景**：存储用户信息、商品信息等对象数据

#### 2.1.3 List（列表）
有序的字符串列表，支持双端操作。

```redis
# 左侧插入
LPUSH queue:tasks "task1" "task2" "task3"

# 右侧弹出
RPOP queue:tasks

# 获取范围
LRANGE queue:tasks 0 -1

# 阻塞弹出（用于消息队列）
BRPOP queue:tasks 30  # 阻塞30秒等待数据
```

**应用场景**：消息队列、最新列表、栈和队列

#### 2.1.4 Set（集合）
无序的字符串集合，元素唯一。

```redis
# 添加元素
SADD tags:article:1 "Java" "Redis" "MySQL"

# 获取所有元素
SMEMBERS tags:article:1

# 集合运算
SINTER tags:article:1 tags:article:2  # 交集
SUNION tags:article:1 tags:article:2  # 并集
SDIFF tags:article:1 tags:article:2   # 差集
```

**应用场景**：标签系统、共同好友、去重

#### 2.1.5 Sorted Set（有序集合）🔥
有序的集合，每个元素关联一个分数。

```redis
# 添加元素
ZADD leaderboard 100 "player1" 200 "player2" 150 "player3"

# 获取排名（从高到低）
ZREVRANGE leaderboard 0 9 WITHSCORES

# 获取分数
ZSCORE leaderboard "player1"

# 增加分数
ZINCRBY leaderboard 50 "player1"

# 按分数范围查询
ZRANGEBYSCORE leaderboard 100 200
```

**应用场景**：排行榜、延迟队列、优先级队列


### 2.2 持久化机制 🔥

Redis提供两种持久化方式，可以单独使用或组合使用。

#### 2.2.1 RDB（Redis Database）
RDB持久化在指定时间间隔内对数据集进行快照。

**配置**
```conf
# 900秒内至少1个key被修改，则触发快照
save 900 1
# 300秒内至少10个key被修改
save 300 10
# 60秒内至少10000个key被修改
save 60 10000

# RDB文件名
dbfilename dump.rdb

# 保存目录
dir /var/lib/redis
```

**手动触发**
```redis
SAVE      # 同步保存，阻塞Redis服务
BGSAVE    # 后台异步保存，不阻塞服务
```

**优点**
- 文件紧凑，适合备份和灾难恢复
- 恢复速度快
- 对性能影响小（fork子进程执行）

**缺点**
- 可能丢失最后一次快照之后的数据
- fork子进程时，数据集较大会导致短暂停顿

#### 2.2.2 AOF（Append Only File）
AOF持久化记录每个写操作，服务器重启时重新执行这些命令恢复数据。

**配置**
```conf
# 开启AOF
appendonly yes

# AOF文件名
appendfilename "appendonly.aof"

# 同步策略
appendfsync always    # 每个命令都同步，最安全但最慢
appendfsync everysec  # 每秒同步一次（推荐）
appendfsync no        # 由操作系统决定，最快但不安全
```

**AOF重写**
```redis
# 手动触发重写
BGREWRITEAOF

# 自动重写配置
auto-aof-rewrite-percentage 100  # AOF文件大小增长100%时重写
auto-aof-rewrite-min-size 64mb   # AOF文件最小64MB才重写
```

**优点**
- 数据更安全，最多丢失1秒数据
- AOF文件是追加日志，即使写入一半崩溃也能修复
- AOF文件过大时会自动重写

**缺点**
- AOF文件通常比RDB文件大
- 恢复速度比RDB慢
- 某些fsync策略下性能较低

#### 2.2.3 混合持久化（推荐）
Redis 4.0+支持RDB和AOF混合使用：

```conf
aof-use-rdb-preamble yes
```

AOF重写时，将当前数据以RDB格式写入AOF文件开头，后续增量以AOF格式追加。结合了两者的优点。


### 2.3 集群与高可用 🔥

#### 2.3.1 主从复制
实现数据备份和读写分离。

**配置从节点**
```conf
# 在从节点配置文件中指定主节点
replicaof 192.168.1.100 6379
masterauth <master-password>
```

**特点**
- 主节点负责写操作，从节点负责读操作
- 数据从主节点异步复制到从节点
- 一个主节点可以有多个从节点
- 从节点也可以有自己的从节点（级联复制）

#### 2.3.2 哨兵模式（Sentinel）
实现自动故障转移。

**哨兵配置**
```conf
# sentinel.conf
sentinel monitor mymaster 192.168.1.100 6379 2
sentinel auth-pass mymaster <password>
sentinel down-after-milliseconds mymaster 5000
sentinel parallel-syncs mymaster 1
sentinel failover-timeout mymaster 180000
```

**启动哨兵**
```bash
redis-sentinel /path/to/sentinel.conf
```

**特点**
- 监控主从节点是否正常运行
- 主节点故障时自动进行故障转移
- 通知客户端新的主节点地址

#### 2.3.3 Redis Cluster
实现数据分片和高可用。

**创建集群**
```bash
# 创建6个节点（3主3从）
redis-cli --cluster create \
  192.168.1.101:7000 192.168.1.102:7000 192.168.1.103:7000 \
  192.168.1.101:7001 192.168.1.102:7001 192.168.1.103:7001 \
  --cluster-replicas 1
```

**连接集群**
```bash
redis-cli -c -p 7000
```

**特点**
- 数据自动分片到16384个槽位
- 每个主节点负责一部分槽位
- 支持自动故障转移
- 客户端请求会自动重定向到正确的节点

**槽位分配示例**
```redis
# 设置key时，Redis会计算key的槽位
SET foo bar
# -> Redirected to slot [12182] located at 127.0.0.1:7002

SET hello world
# -> Redirected to slot [866] located at 127.0.0.1:7000
```


### 2.4 缓存问题与解决方案 (⚠️ 难点)

#### 2.4.1 缓存穿透
**问题**：查询不存在的数据，缓存和数据库都没有，导致每次请求都打到数据库。

**解决方案1：布隆过滤器**
```java
// 使用Redisson的布隆过滤器
RBloomFilter<String> bloomFilter = redisson.getBloomFilter("user:bloom");
bloomFilter.tryInit(100000L, 0.01);  // 预计元素数量和误判率

// 添加元素
bloomFilter.add("user:1001");

// 查询前先判断
if (!bloomFilter.contains("user:9999")) {
    return null;  // 一定不存在
}
// 可能存在，继续查询缓存和数据库
```

**解决方案2：缓存空值**
```java
String value = redis.get(key);
if (value == null) {
    value = db.query(key);
    if (value == null) {
        // 缓存空值，设置较短过期时间
        redis.setex(key, 60, "NULL");
        return null;
    }
    redis.setex(key, 3600, value);
}
return "NULL".equals(value) ? null : value;
```

#### 2.4.2 缓存击穿
**问题**：热点key过期瞬间，大量请求同时打到数据库。

**解决方案1：互斥锁**
```java
public String getData(String key) {
    String value = redis.get(key);
    if (value == null) {
        // 尝试获取分布式锁
        String lockKey = "lock:" + key;
        if (redis.setnx(lockKey, "1", 10)) {  // 10秒过期
            try {
                // 再次检查缓存
                value = redis.get(key);
                if (value == null) {
                    // 查询数据库
                    value = db.query(key);
                    redis.setex(key, 3600, value);
                }
            } finally {
                redis.del(lockKey);
            }
        } else {
            // 等待一段时间后重试
            Thread.sleep(50);
            return getData(key);
        }
    }
    return value;
}
```

**解决方案2：热点数据永不过期**
```java
// 逻辑过期：在value中存储过期时间
public String getData(String key) {
    String json = redis.get(key);
    if (json == null) {
        return loadAndCache(key);
    }
    
    CacheData data = JSON.parseObject(json, CacheData.class);
    if (data.getExpireTime() < System.currentTimeMillis()) {
        // 异步更新缓存
        threadPool.execute(() -> loadAndCache(key));
    }
    return data.getValue();
}
```

#### 2.4.3 缓存雪崩 (⚠️ 难点)
**问题**：大量key同时过期，或Redis宕机，导致请求全部打到数据库。

**解决方案1：过期时间加随机值**
```java
// 避免同时过期
int expireTime = 3600 + new Random().nextInt(300);  // 3600-3900秒
redis.setex(key, expireTime, value);
```

**解决方案2：使用集群和哨兵**
```java
// 配置Redis集群或哨兵，提高可用性
// 即使部分节点宕机，服务仍可用
```

**解决方案3：多级缓存**
```java
// 本地缓存 + Redis缓存 + 数据库
public String getData(String key) {
    // 1. 查询本地缓存
    String value = localCache.get(key);
    if (value != null) return value;
    
    // 2. 查询Redis
    value = redis.get(key);
    if (value != null) {
        localCache.put(key, value, 60);  // 本地缓存60秒
        return value;
    }
    
    // 3. 查询数据库
    value = db.query(key);
    redis.setex(key, 3600, value);
    localCache.put(key, value, 60);
    return value;
}
```

**解决方案4：限流降级**
```java
// 使用Sentinel或Hystrix进行限流
@SentinelResource(value = "getData", 
    blockHandler = "handleBlock",
    fallback = "handleFallback")
public String getData(String key) {
    // 正常业务逻辑
}

public String handleBlock(String key, BlockException ex) {
    return "系统繁忙，请稍后重试";
}
```


### 2.5 分布式锁 (⚠️ 难点)

#### 2.5.1 基础实现
```java
public class RedisLock {
    private Jedis jedis;
    
    /**
     * 获取锁
     * @param lockKey 锁的key
     * @param requestId 请求标识（UUID）
     * @param expireTime 过期时间（秒）
     */
    public boolean tryLock(String lockKey, String requestId, int expireTime) {
        // SET key value NX EX seconds
        String result = jedis.set(lockKey, requestId, "NX", "EX", expireTime);
        return "OK".equals(result);
    }
    
    /**
     * 释放锁（使用Lua脚本保证原子性）
     */
    public boolean unlock(String lockKey, String requestId) {
        String script = 
            "if redis.call('get', KEYS[1]) == ARGV[1] then " +
            "    return redis.call('del', KEYS[1]) " +
            "else " +
            "    return 0 " +
            "end";
        
        Object result = jedis.eval(script, 
            Collections.singletonList(lockKey),
            Collections.singletonList(requestId));
        
        return Long.valueOf(1).equals(result);
    }
}
```

**使用示例**
```java
String lockKey = "lock:order:1001";
String requestId = UUID.randomUUID().toString();

try {
    // 尝试获取锁，10秒过期
    if (redisLock.tryLock(lockKey, requestId, 10)) {
        // 执行业务逻辑
        processOrder();
    } else {
        return "系统繁忙，请稍后重试";
    }
} finally {
    // 释放锁
    redisLock.unlock(lockKey, requestId);
}
```

#### 2.5.2 Redisson实现（推荐）
```java
// 使用Redisson的分布式锁
RLock lock = redisson.getLock("lock:order:1001");

try {
    // 尝试加锁，最多等待10秒，锁30秒后自动释放
    if (lock.tryLock(10, 30, TimeUnit.SECONDS)) {
        // 执行业务逻辑
        processOrder();
    }
} catch (InterruptedException e) {
    Thread.currentThread().interrupt();
} finally {
    // 释放锁（只有持有锁的线程才能释放）
    if (lock.isHeldByCurrentThread()) {
        lock.unlock();
    }
}
```

**Redisson的优势**
- 自动续期（看门狗机制）
- 可重入锁
- 公平锁支持
- 读写锁支持
- 联锁和红锁支持

#### 2.5.3 RedLock算法
在多个独立的Redis实例上获取锁，提高可靠性。

```java
// 使用Redisson的RedLock
RLock lock1 = redisson1.getLock("lock:order");
RLock lock2 = redisson2.getLock("lock:order");
RLock lock3 = redisson3.getLock("lock:order");

RedissonRedLock redLock = new RedissonRedLock(lock1, lock2, lock3);

try {
    // 在大多数实例上获取锁才算成功
    if (redLock.tryLock(10, 30, TimeUnit.SECONDS)) {
        // 执行业务逻辑
        processOrder();
    }
} finally {
    redLock.unlock();
}
```


## 💻 实战应用

### 3.1 环境搭建

**Docker安装Redis 7.x**
```bash
# 拉取Redis镜像
docker pull redis:7.2

# 启动Redis容器
docker run -d \
  --name redis7 \
  -p 6379:6379 \
  -v /data/redis:/data \
  redis:7.2 \
  redis-server --appendonly yes

# 连接Redis
docker exec -it redis7 redis-cli
```

**配置文件优化(redis.conf)**
```conf
# 绑定地址
bind 0.0.0.0

# 端口
port 6379

# 密码
requirepass your_password

# 最大内存
maxmemory 2gb

# 内存淘汰策略
maxmemory-policy allkeys-lru

# 持久化
appendonly yes
appendfsync everysec

# 慢查询日志
slowlog-log-slower-than 10000  # 10毫秒
slowlog-max-len 128
```

### 3.2 Spring Boot集成

**添加依赖**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>
```

**配置文件**
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
    timeout: 3000ms
```

**RedisTemplate配置**
```java
@Configuration
public class RedisConfig {
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory connectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);
        
        // 使用Jackson序列化
        Jackson2JsonRedisSerializer<Object> serializer = 
            new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper mapper = new ObjectMapper();
        mapper.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        mapper.activateDefaultTyping(
            LaissezFaireSubTypeValidator.instance,
            ObjectMapper.DefaultTyping.NON_FINAL);
        serializer.setObjectMapper(mapper);
        
        // 设置key和value的序列化规则
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(serializer);
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(serializer);
        
        template.afterPropertiesSet();
        return template;
    }
}
```


### 3.3 进阶案例

#### 案例1：缓存用户信息
```java
@Service
public class UserService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private UserMapper userMapper;
    
    private static final String USER_CACHE_KEY = "user:info:";
    private static final long CACHE_EXPIRE = 3600;  // 1小时
    
    public User getUserById(Long userId) {
        String key = USER_CACHE_KEY + userId;
        
        // 1. 查询缓存
        User user = (User) redisTemplate.opsForValue().get(key);
        if (user != null) {
            return user;
        }
        
        // 2. 查询数据库
        user = userMapper.selectById(userId);
        if (user == null) {
            // 缓存空值，防止缓存穿透
            redisTemplate.opsForValue().set(key, new User(), 60, TimeUnit.SECONDS);
            return null;
        }
        
        // 3. 写入缓存，添加随机过期时间防止雪崩
        int randomExpire = CACHE_EXPIRE + new Random().nextInt(300);
        redisTemplate.opsForValue().set(key, user, randomExpire, TimeUnit.SECONDS);
        
        return user;
    }
    
    public void updateUser(User user) {
        // 1. 更新数据库
        userMapper.updateById(user);
        
        // 2. 删除缓存（延迟双删）
        String key = USER_CACHE_KEY + user.getId();
        redisTemplate.delete(key);
        
        // 延迟500ms再删除一次，防止脏数据
        new Thread(() -> {
            try {
                Thread.sleep(500);
                redisTemplate.delete(key);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }).start();
    }
}
```

#### 案例2：实现排行榜
```java
@Service
public class LeaderboardService {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    private static final String LEADERBOARD_KEY = "leaderboard:score";
    
    /**
     * 更新用户分数
     */
    public void updateScore(Long userId, double score) {
        redisTemplate.opsForZSet().add(LEADERBOARD_KEY, 
            "user:" + userId, score);
    }
    
    /**
     * 增加用户分数
     */
    public void incrementScore(Long userId, double delta) {
        redisTemplate.opsForZSet().incrementScore(LEADERBOARD_KEY, 
            "user:" + userId, delta);
    }
    
    /**
     * 获取排行榜（前N名）
     */
    public List<LeaderboardEntry> getTopN(int n) {
        Set<ZSetOperations.TypedTuple<Object>> result = 
            redisTemplate.opsForZSet().reverseRangeWithScores(
                LEADERBOARD_KEY, 0, n - 1);
        
        List<LeaderboardEntry> leaderboard = new ArrayList<>();
        int rank = 1;
        for (ZSetOperations.TypedTuple<Object> tuple : result) {
            LeaderboardEntry entry = new LeaderboardEntry();
            entry.setRank(rank++);
            entry.setUserId(Long.parseLong(
                tuple.getValue().toString().replace("user:", "")));
            entry.setScore(tuple.getScore());
            leaderboard.add(entry);
        }
        return leaderboard;
    }
    
    /**
     * 获取用户排名
     */
    public Long getUserRank(Long userId) {
        Long rank = redisTemplate.opsForZSet().reverseRank(
            LEADERBOARD_KEY, "user:" + userId);
        return rank != null ? rank + 1 : null;
    }
    
    /**
     * 获取用户分数
     */
    public Double getUserScore(Long userId) {
        return redisTemplate.opsForZSet().score(
            LEADERBOARD_KEY, "user:" + userId);
    }
}
```

#### 案例3：限流器
```java
@Component
public class RateLimiter {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 滑动窗口限流
     * @param key 限流key
     * @param limit 限制次数
     * @param window 时间窗口（秒）
     */
    public boolean tryAcquire(String key, int limit, int window) {
        long now = System.currentTimeMillis();
        long windowStart = now - window * 1000;
        
        // 移除窗口外的记录
        redisTemplate.opsForZSet().removeRangeByScore(key, 0, windowStart);
        
        // 统计窗口内的请求数
        Long count = redisTemplate.opsForZSet().count(key, windowStart, now);
        
        if (count != null && count < limit) {
            // 添加当前请求
            redisTemplate.opsForZSet().add(key, String.valueOf(now), now);
            // 设置过期时间
            redisTemplate.expire(key, window, TimeUnit.SECONDS);
            return true;
        }
        
        return false;
    }
}

// 使用示例
@RestController
public class ApiController {
    
    @Autowired
    private RateLimiter rateLimiter;
    
    @GetMapping("/api/data")
    public Result getData(HttpServletRequest request) {
        String ip = request.getRemoteAddr();
        String key = "rate:limit:" + ip;
        
        // 每个IP每分钟最多100次请求
        if (!rateLimiter.tryAcquire(key, 100, 60)) {
            return Result.error("请求过于频繁，请稍后重试");
        }
        
        // 正常业务逻辑
        return Result.success(getData());
    }
}
```


## ✨ 最佳实践

### 4.1 Key设计规范

**1. 命名规范**
```
业务模块:功能:唯一标识
例如：
user:info:1001
order:detail:20240101001
cache:product:list:page:1
```

**2. 避免过长的key**
```redis
# ❌ 不推荐
SET this_is_a_very_long_key_name_that_wastes_memory "value"

# ✅ 推荐
SET user:1001:name "value"
```

**3. 设置合理的过期时间**
```java
// 根据业务场景设置不同的过期时间
redisTemplate.opsForValue().set("session:token", token, 30, TimeUnit.MINUTES);
redisTemplate.opsForValue().set("cache:hot:data", data, 1, TimeUnit.HOURS);
redisTemplate.opsForValue().set("cache:cold:data", data, 1, TimeUnit.DAYS);
```

### 4.2 性能优化

**1. 使用Pipeline批量操作**
```java
// ❌ 逐条执行（网络开销大）
for (int i = 0; i < 1000; i++) {
    redisTemplate.opsForValue().set("key:" + i, "value" + i);
}

// ✅ 使用Pipeline
redisTemplate.executePipelined(new RedisCallback<Object>() {
    @Override
    public Object doInRedis(RedisConnection connection) {
        for (int i = 0; i < 1000; i++) {
            connection.set(("key:" + i).getBytes(), 
                          ("value" + i).getBytes());
        }
        return null;
    }
});
```

**2. 避免大key**
```java
// ❌ 单个key存储大量数据
redisTemplate.opsForList().rightPushAll("big:list", largeDataList);

// ✅ 拆分成多个小key
int batchSize = 1000;
for (int i = 0; i < largeDataList.size(); i += batchSize) {
    List<String> batch = largeDataList.subList(i, 
        Math.min(i + batchSize, largeDataList.size()));
    redisTemplate.opsForList().rightPushAll("list:" + (i / batchSize), batch);
}
```

**3. 选择合适的数据类型**
```java
// 存储对象信息
// ❌ 使用String存储JSON
redisTemplate.opsForValue().set("user:1001", JSON.toJSONString(user));

// ✅ 使用Hash存储
redisTemplate.opsForHash().put("user:1001", "name", user.getName());
redisTemplate.opsForHash().put("user:1001", "age", user.getAge());
// Hash可以单独更新某个字段，更高效
```

### 4.3 安全规范

**1. 设置密码**
```conf
requirepass your_strong_password
```

**2. 禁用危险命令**
```conf
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command KEYS ""
rename-command CONFIG "CONFIG_ADMIN_ONLY"
```

**3. 限制访问IP**
```conf
bind 127.0.0.1 192.168.1.100
```

**4. 使用SSL/TLS加密**
```conf
tls-port 6380
tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt
```

### 4.4 常见陷阱

**⚠️ 陷阱1：使用KEYS命令**
```redis
# ❌ KEYS会阻塞Redis，生产环境禁用
KEYS user:*

# ✅ 使用SCAN命令
SCAN 0 MATCH user:* COUNT 100
```

**⚠️ 陷阱2：不设置过期时间**
```java
// ❌ 可能导致内存泄漏
redisTemplate.opsForValue().set(key, value);

// ✅ 设置合理的过期时间
redisTemplate.opsForValue().set(key, value, 3600, TimeUnit.SECONDS);
```

**⚠️ 陷阱3：缓存与数据库不一致**
```java
// ❌ 先更新缓存，再更新数据库（可能导致不一致）
redisTemplate.opsForValue().set(key, newValue);
userMapper.updateById(user);

// ✅ 先更新数据库，再删除缓存
userMapper.updateById(user);
redisTemplate.delete(key);
```

**⚠️ 陷阱4：热key问题**
```java
// 问题：某个key访问量特别大，单个Redis节点压力大

// 解决方案1：本地缓存
@Cacheable(value = "hotData", key = "#id")
public Data getHotData(Long id) {
    return redisTemplate.opsForValue().get("hot:data:" + id);
}

// 解决方案2：key分散
String key = "hot:data:" + id + ":" + (id % 10);  // 分散到10个key
```


## ❓ 常见问题

### Q1: Redis为什么这么快？
A: 
1. **纯内存操作**：数据存储在内存中，读写速度极快
2. **单线程模型**：避免了线程切换和锁竞争的开销
3. **IO多路复用**：使用epoll等机制高效处理并发连接
4. **高效的数据结构**：针对不同场景优化的数据结构
5. **简单的协议**：RESP协议简单高效

### Q2: Redis单线程为什么还能支持高并发？
A: 
- Redis的瓶颈不在CPU，而在内存和网络IO
- 单线程避免了上下文切换和锁竞争
- 使用IO多路复用技术处理并发连接
- Redis 6.0+引入了多线程IO，进一步提升性能

### Q3: 如何选择RDB还是AOF？
A: 
- **RDB**：适合备份和灾难恢复，恢复速度快，但可能丢失数据
- **AOF**：数据更安全，但文件更大，恢复更慢
- **推荐**：使用混合持久化（RDB+AOF），兼顾性能和安全

### Q4: 缓存更新策略如何选择？
A: 
1. **Cache Aside（旁路缓存）**：最常用
   - 读：先查缓存，miss则查数据库并写入缓存
   - 写：先更新数据库，再删除缓存
2. **Read/Write Through**：应用程序只操作缓存，由缓存层负责数据库同步
3. **Write Behind（异步写入）**：先写缓存，异步批量写入数据库

### Q5: 如何保证缓存与数据库的一致性？
A: 
1. **延迟双删**：更新数据库后删除缓存，延迟后再删除一次
2. **设置较短的过期时间**：即使不一致，也会很快过期
3. **使用消息队列**：通过MQ异步更新缓存
4. **Canal监听binlog**：监听MySQL binlog，实时更新缓存

### Q6: Redis内存满了怎么办？
A: 
1. **设置maxmemory**：限制最大内存
2. **配置淘汰策略**：
   - noeviction：不淘汰，写入报错
   - allkeys-lru：淘汰最少使用的key（推荐）
   - allkeys-random：随机淘汰
   - volatile-lru：淘汰设置了过期时间的最少使用key
   - volatile-ttl：淘汰即将过期的key
3. **增加内存或扩展集群**

### Q7: 如何监控Redis性能？
A: 
```redis
# 查看实时命令
MONITOR

# 查看慢查询
SLOWLOG GET 10

# 查看内存使用
INFO memory

# 查看客户端连接
CLIENT LIST

# 查看统计信息
INFO stats
```

### Q8: Redis集群如何扩容？
A: 
```bash
# 添加新节点
redis-cli --cluster add-node new_host:new_port existing_host:existing_port

# 重新分配槽位
redis-cli --cluster reshard existing_host:existing_port

# 添加从节点
redis-cli --cluster add-node new_host:new_port existing_host:existing_port --cluster-slave
```


## 🔗 相关资源

### 官方文档
- Redis官方文档：https://redis.io/docs/
- Redis命令参考：https://redis.io/commands/
- Redis最佳实践：https://redis.io/docs/manual/patterns/

### 推荐书籍
- 《Redis设计与实现》
- 《Redis实战》
- 《Redis开发与运维》

### 推荐文章
- Redis持久化机制详解
- Redis集群原理与实践
- 缓存穿透、击穿、雪崩解决方案

### 工具推荐
- **Redis Desktop Manager**：图形化管理工具
- **RedisInsight**：官方可视化工具
- **redis-cli**：命令行工具
- **Redisson**：Java Redis客户端框架

### 相关技术
- **Redisson**：功能强大的Redis Java客户端
- **Lettuce**：Spring Boot默认的Redis客户端
- **Jedis**：传统的Redis Java客户端

## 📝 学习检查清单

- [ ] 理解Redis的基本概念和应用场景
- [ ] 掌握五大基本数据类型的使用
- [ ] 理解RDB和AOF持久化机制
- [ ] 掌握主从复制、哨兵、集群的配置
- [ ] 能够解决缓存穿透、击穿、雪崩问题
- [ ] 掌握分布式锁的实现
- [ ] 了解Redis性能优化方法
- [ ] 掌握Spring Boot集成Redis
- [ ] 完成至少3个实战案例
- [ ] 了解Redis监控和运维

---

**@author erik.zhou**  
**文档版本**: v1.0  
**最后更新**: 2024-12-31
