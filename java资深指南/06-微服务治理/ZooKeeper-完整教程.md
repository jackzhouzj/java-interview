# ZooKeeper 完整教程

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
- **版本**: 3.8.x
- **官方文档**: https://zookeeper.apache.org/
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Java基础
  - 分布式系统基础概念
  - 网络编程基础
  - Linux基础操作

## 🎯 学习目标
- [ ] 理解ZooKeeper的核心概念和应用场景
- [ ] 掌握ZooKeeper的数据模型和节点类型
- [ ] 熟练使用Watch机制实现分布式协调
- [ ] 理解ZAB协议和集群选举机制
- [ ] 能够搭建和管理ZooKeeper集群
- [ ] 掌握ZooKeeper在分布式系统中的典型应用
- [ ] 了解ZooKeeper的性能优化和运维要点

## 📖 基础概念

### 1.1 什么是ZooKeeper

ZooKeeper是一个开源的分布式协调服务，为分布式应用提供一致性服务。它是一个高性能的协调服务，用于维护配置信息、命名服务、提供分布式同步和提供组服务。

**核心定位**:
- 分布式应用的协调服务
- 提供简单的原语集合
- 保证数据的强一致性
- 高可用和高性能

**@author erik.zhou**

### 1.2 核心概念

#### 1.2.1 ZNode（数据节点）
- **定义**: ZooKeeper中的数据存储单元，类似文件系统的文件和目录
- **特点**: 每个ZNode都可以存储数据，同时可以有子节点
- **路径**: 使用斜杠分隔的路径标识，如 `/app/config`

#### 1.2.2 Session（会话）
- **定义**: 客户端与ZooKeeper服务器之间的连接
- **特点**: 
  - 有超时时间
  - 支持心跳保活
  - 会话失效后临时节点会被删除

#### 1.2.3 Watcher（监听器）
- **定义**: 客户端注册在ZNode上的监听机制
- **特点**: 
  - 一次性触发
  - 异步通知
  - 轻量级

#### 1.2.4 ACL（访问控制列表）
- **定义**: ZooKeeper的权限控制机制
- **权限类型**: CREATE、READ、WRITE、DELETE、ADMIN

### 1.3 应用场景

#### 1.3.1 配置管理
- 集中式配置存储
- 配置动态更新
- 配置版本管理

#### 1.3.2 命名服务
- 分布式ID生成
- 服务注册与发现
- 资源命名

#### 1.3.3 分布式锁
- 排他锁
- 共享锁
- 读写锁

#### 1.3.4 集群管理
- 节点状态监控
- Master选举
- 负载均衡

#### 1.3.5 分布式队列
- FIFO队列
- 优先级队列
- 屏障（Barrier）

## 🔥 核心特性

### 2.1 数据模型 🔥

ZooKeeper的数据模型是一个树形的层次化命名空间，类似于文件系统。

#### 2.1.1 ZNode类型

**持久节点（PERSISTENT）**:
```java
// 创建持久节点
String path = zooKeeper.create(
    "/app/config",           // 路径
    "data".getBytes(),       // 数据
    ZooDefs.Ids.OPEN_ACL_UNSAFE,  // ACL
    CreateMode.PERSISTENT    // 节点类型
);
```

**持久顺序节点（PERSISTENT_SEQUENTIAL）**:
```java
// 创建持久顺序节点，ZooKeeper会自动添加序号
String path = zooKeeper.create(
    "/app/task-",
    "data".getBytes(),
    ZooDefs.Ids.OPEN_ACL_UNSAFE,
    CreateMode.PERSISTENT_SEQUENTIAL
);
// 返回: /app/task-0000000001
```

**临时节点（EPHEMERAL）**:
```java
// 创建临时节点，会话结束后自动删除
String path = zooKeeper.create(
    "/app/lock",
    "data".getBytes(),
    ZooDefs.Ids.OPEN_ACL_UNSAFE,
    CreateMode.EPHEMERAL
);
```

**临时顺序节点（EPHEMERAL_SEQUENTIAL）**:
```java
// 创建临时顺序节点
String path = zooKeeper.create(
    "/app/lock-",
    "data".getBytes(),
    ZooDefs.Ids.OPEN_ACL_UNSAFE,
    CreateMode.EPHEMERAL_SEQUENTIAL
);
```

#### 2.1.2 ZNode结构

每个ZNode包含以下信息：
- **data**: 存储的数据（最大1MB）
- **children**: 子节点列表
- **stat**: 状态信息（版本号、时间戳等）

```java
// 获取ZNode数据和状态
Stat stat = new Stat();
byte[] data = zooKeeper.getData("/app/config", false, stat);

System.out.println("数据: " + new String(data));
System.out.println("版本: " + stat.getVersion());
System.out.println("创建时间: " + stat.getCtime());
System.out.println("修改时间: " + stat.getMtime());
```

### 2.2 Watch机制 🔥

Watch是ZooKeeper实现分布式协调的核心机制，允许客户端监听ZNode的变化。

#### 2.2.1 Watch特性

**一次性触发**:
```java
// 注册Watch，只会触发一次
zooKeeper.exists("/app/config", new Watcher() {
    @Override
    public void process(WatchedEvent event) {
        System.out.println("节点变化: " + event.getType());
        // 如需继续监听，需要重新注册
        try {
            zooKeeper.exists("/app/config", this);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
});
```

**异步通知**:
```java
// Watch回调在单独的线程中执行
zooKeeper.getData("/app/config", new Watcher() {
    @Override
    public void process(WatchedEvent event) {
        // 异步回调
        System.out.println("数据变化通知");
    }
}, null);
```

#### 2.2.2 Watch类型

**getData Watch**:
```java
// 监听节点数据变化
zooKeeper.getData("/app/config", event -> {
    if (event.getType() == Watcher.Event.EventType.NodeDataChanged) {
        System.out.println("数据已更新");
    }
}, null);
```

**exists Watch**:
```java
// 监听节点创建、删除、数据变化
zooKeeper.exists("/app/config", event -> {
    switch (event.getType()) {
        case NodeCreated:
            System.out.println("节点已创建");
            break;
        case NodeDeleted:
            System.out.println("节点已删除");
            break;
        case NodeDataChanged:
            System.out.println("数据已变化");
            break;
    }
}, null);
```

**getChildren Watch**:
```java
// 监听子节点变化
zooKeeper.getChildren("/app", event -> {
    if (event.getType() == Watcher.Event.EventType.NodeChildrenChanged) {
        System.out.println("子节点列表已变化");
    }
}, null);
```

### 2.3 ZAB协议 (⚠️ 难点)

ZAB（ZooKeeper Atomic Broadcast）是ZooKeeper的核心一致性协议，保证分布式数据的一致性。

#### 2.3.1 ZAB协议核心概念

**Leader选举**:
- 集群启动时选举Leader
- Leader崩溃时重新选举
- 基于ZXID（事务ID）和myid（服务器ID）

**原子广播**:
- Leader接收写请求
- 广播给所有Follower
- 超过半数确认后提交

#### 2.3.2 ZAB协议流程

```
1. 选举阶段（Election）
   - 每个服务器投票给ZXID最大的服务器
   - 如果ZXID相同，选择myid最大的
   - 超过半数投票的服务器成为Leader

2. 发现阶段（Discovery）
   - Follower连接到Leader
   - 同步最新的事务日志

3. 同步阶段（Synchronization）
   - Leader将最新数据同步给Follower
   - 确保所有节点数据一致

4. 广播阶段（Broadcast）
   - Leader处理写请求
   - 通过两阶段提交广播给Follower
```

#### 2.3.3 ZXID（事务ID）

ZXID是一个64位的数字：
- **高32位**: epoch（选举周期）
- **低32位**: counter（事务计数器）

```java
// ZXID示例
// 0x100000001 表示第1个epoch的第1个事务
// 0x100000002 表示第1个epoch的第2个事务
// 0x200000001 表示第2个epoch的第1个事务
```

### 2.4 集群架构 🔥

#### 2.4.1 集群角色

**Leader（领导者）**:
- 处理所有写请求
- 负责事务的提交
- 协调Follower和Observer

**Follower（跟随者）**:
- 处理读请求
- 参与Leader选举
- 参与写请求的投票

**Observer（观察者）**:
- 处理读请求
- 不参与选举和投票
- 用于提升读性能

#### 2.4.2 集群配置

```properties
# zoo.cfg
# 数据目录
dataDir=/var/lib/zookeeper
# 客户端连接端口
clientPort=2181
# 心跳间隔（毫秒）
tickTime=2000
# Follower初始化连接Leader的超时时间（tickTime倍数）
initLimit=10
# Follower与Leader同步的超时时间（tickTime倍数）
syncLimit=5

# 集群配置
# server.id=host:port1:port2
# port1: Follower连接Leader的端口
# port2: Leader选举端口
server.1=192.168.1.101:2888:3888
server.2=192.168.1.102:2888:3888
server.3=192.168.1.103:2888:3888
```

#### 2.4.3 集群特性

**过半机制**:
- 写操作需要超过半数节点确认
- 集群可用需要超过半数节点存活
- 推荐奇数个节点（3、5、7）

**数据一致性**:
- 顺序一致性：客户端的更新按发送顺序应用
- 原子性：更新要么成功要么失败
- 单一视图：客户端看到的数据视图一致
- 可靠性：更新一旦成功就会持久化
- 实时性：客户端在一定时间内能看到最新数据

### 2.5 脑裂问题 (⚠️ 难点)

#### 2.5.1 什么是脑裂

脑裂（Split-Brain）是指集群因网络分区导致出现多个Leader的情况。

**场景示例**:
```
原始集群: [Leader] - [Follower1] - [Follower2]
网络分区后:
  分区1: [Leader]
  分区2: [Follower1] - [Follower2] -> 选举新Leader
结果: 出现两个Leader
```

#### 2.5.2 ZooKeeper如何避免脑裂

**过半机制**:
```
集群5个节点，网络分区:
  分区1: 3个节点 -> 可以选举Leader（超过半数）
  分区2: 2个节点 -> 无法选举Leader（未超过半数）
  
结果: 只有一个Leader，避免脑裂
```

**Epoch机制**:
- 每次选举epoch递增
- 旧epoch的Leader无法提交事务
- 保证只有最新的Leader有效

#### 2.5.3 脑裂预防最佳实践

```java
// 1. 使用奇数个节点
// 推荐: 3、5、7个节点

// 2. 合理设置超时时间
tickTime=2000
initLimit=10
syncLimit=5

// 3. 网络隔离检测
// 监控网络分区，及时告警

// 4. 使用Observer节点
// Observer不参与投票，不影响过半机制
server.4=192.168.1.104:2888:3888:observer
```

## 💻 实战应用

### 3.1 环境搭建

#### 3.1.1 单机模式

**下载安装**:
```bash
# 下载ZooKeeper
wget https://downloads.apache.org/zookeeper/zookeeper-3.8.3/apache-zookeeper-3.8.3-bin.tar.gz

# 解压
tar -zxvf apache-zookeeper-3.8.3-bin.tar.gz
cd apache-zookeeper-3.8.3-bin

# 配置
cp conf/zoo_sample.cfg conf/zoo.cfg
```

**启动服务**:
```bash
# 启动
bin/zkServer.sh start

# 查看状态
bin/zkServer.sh status

# 停止
bin/zkServer.sh stop
```

**客户端连接**:
```bash
# 连接本地ZooKeeper
bin/zkCli.sh -server localhost:2181

# 基本命令
ls /                    # 列出根节点
create /test "data"     # 创建节点
get /test              # 获取节点数据
set /test "newdata"    # 更新节点数据
delete /test           # 删除节点
```

#### 3.1.2 集群模式

**配置集群**:
```bash
# 节点1配置 (192.168.1.101)
dataDir=/var/lib/zookeeper
clientPort=2181
server.1=192.168.1.101:2888:3888
server.2=192.168.1.102:2888:3888
server.3=192.168.1.103:2888:3888

# 创建myid文件
echo "1" > /var/lib/zookeeper/myid

# 节点2配置 (192.168.1.102)
# 同上，myid设置为2

# 节点3配置 (192.168.1.103)
# 同上，myid设置为3
```

**启动集群**:
```bash
# 在每个节点上启动
bin/zkServer.sh start

# 查看集群状态
bin/zkServer.sh status
# 输出: Mode: leader 或 Mode: follower
```

### 3.2 Java客户端开发

#### 3.2.1 Maven依赖

```xml
<dependency>
    <groupId>org.apache.zookeeper</groupId>
    <artifactId>zookeeper</artifactId>
    <version>3.8.3</version>
</dependency>
```

#### 3.2.2 基础操作

```java
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.util.concurrent.CountDownLatch;

/**
 * ZooKeeper基础操作示例
 * 
 * @author erik.zhou
 */
public class ZooKeeperBasicExample {
    
    private static final String CONNECT_STRING = "localhost:2181";
    private static final int SESSION_TIMEOUT = 5000;
    private ZooKeeper zooKeeper;
    
    /**
     * 连接ZooKeeper
     */
    public void connect() throws Exception {
        CountDownLatch latch = new CountDownLatch(1);
        
        zooKeeper = new ZooKeeper(CONNECT_STRING, SESSION_TIMEOUT, event -> {
            if (event.getState() == Watcher.Event.KeeperState.SyncConnected) {
                System.out.println("连接成功");
                latch.countDown();
            }
        });
        
        // 等待连接成功
        latch.await();
    }
    
    /**
     * 创建节点
     */
    public void createNode(String path, String data) throws Exception {
        String result = zooKeeper.create(
            path,
            data.getBytes(),
            ZooDefs.Ids.OPEN_ACL_UNSAFE,
            CreateMode.PERSISTENT
        );
        System.out.println("创建节点: " + result);
    }
    
    /**
     * 获取节点数据
     */
    public String getData(String path) throws Exception {
        Stat stat = new Stat();
        byte[] data = zooKeeper.getData(path, false, stat);
        System.out.println("版本: " + stat.getVersion());
        return new String(data);
    }
    
    /**
     * 更新节点数据
     */
    public void setData(String path, String data) throws Exception {
        Stat stat = zooKeeper.setData(path, data.getBytes(), -1);
        System.out.println("更新成功，新版本: " + stat.getVersion());
    }
    
    /**
     * 删除节点
     */
    public void deleteNode(String path) throws Exception {
        zooKeeper.delete(path, -1);
        System.out.println("删除节点: " + path);
    }
    
    /**
     * 判断节点是否存在
     */
    public boolean exists(String path) throws Exception {
        Stat stat = zooKeeper.exists(path, false);
        return stat != null;
    }
    
    /**
     * 关闭连接
     */
    public void close() throws Exception {
        if (zooKeeper != null) {
            zooKeeper.close();
        }
    }
}
```

### 3.3 分布式锁实现

#### 3.3.1 排他锁实现

```java
import org.apache.zookeeper.*;
import org.apache.zookeeper.data.Stat;

import java.util.concurrent.CountDownLatch;

/**
 * 基于ZooKeeper的分布式排他锁
 * 
 * @author erik.zhou
 */
public class DistributedLock {
    
    private ZooKeeper zooKeeper;
    private String lockPath;
    private String currentLockPath;
    
    public DistributedLock(ZooKeeper zooKeeper, String lockPath) {
        this.zooKeeper = zooKeeper;
        this.lockPath = lockPath;
    }
    
    /**
     * 获取锁
     */
    public boolean lock() throws Exception {
        // 创建临时顺序节点
        currentLockPath = zooKeeper.create(
            lockPath + "/lock-",
            new byte[0],
            ZooDefs.Ids.OPEN_ACL_UNSAFE,
            CreateMode.EPHEMERAL_SEQUENTIAL
        );
        
        // 检查是否是最小节点
        return checkLock();
    }
    
    /**
     * 检查是否获得锁
     */
    private boolean checkLock() throws Exception {
        List<String> children = zooKeeper.getChildren(lockPath, false);
        Collections.sort(children);
        
        String currentNode = currentLockPath.substring(lockPath.length() + 1);
        
        // 如果是最小节点，获得锁
        if (currentNode.equals(children.get(0))) {
            return true;
        }
        
        // 否则监听前一个节点
        int index = children.indexOf(currentNode);
        String prevNode = children.get(index - 1);
        
        CountDownLatch latch = new CountDownLatch(1);
        
        Stat stat = zooKeeper.exists(lockPath + "/" + prevNode, event -> {
            if (event.getType() == Watcher.Event.EventType.NodeDeleted) {
                latch.countDown();
            }
        });
        
        if (stat != null) {
            // 等待前一个节点删除
            latch.await();
        }
        
        return checkLock();
    }
    
    /**
     * 释放锁
     */
    public void unlock() throws Exception {
        if (currentLockPath != null) {
            zooKeeper.delete(currentLockPath, -1);
            currentLockPath = null;
        }
    }
}
```

#### 3.3.2 读写锁实现

```java
/**
 * 基于ZooKeeper的分布式读写锁
 * 
 * @author erik.zhou
 */
public class DistributedReadWriteLock {
    
    private ZooKeeper zooKeeper;
    private String lockPath;
    
    /**
     * 获取读锁
     */
    public boolean readLock() throws Exception {
        String readLockPath = zooKeeper.create(
            lockPath + "/read-",
            new byte[0],
            ZooDefs.Ids.OPEN_ACL_UNSAFE,
            CreateMode.EPHEMERAL_SEQUENTIAL
        );
        
        // 检查是否有写锁
        List<String> children = zooKeeper.getChildren(lockPath, false);
        for (String child : children) {
            if (child.startsWith("write-") && child.compareTo(readLockPath) < 0) {
                // 有更早的写锁，等待
                return waitForLock(child);
            }
        }
        
        return true;
    }
    
    /**
     * 获取写锁
     */
    public boolean writeLock() throws Exception {
        String writeLockPath = zooKeeper.create(
            lockPath + "/write-",
            new byte[0],
            ZooDefs.Ids.OPEN_ACL_UNSAFE,
            CreateMode.EPHEMERAL_SEQUENTIAL
        );
        
        // 检查是否有其他锁
        List<String> children = zooKeeper.getChildren(lockPath, false);
        Collections.sort(children);
        
        String currentNode = writeLockPath.substring(lockPath.length() + 1);
        
        // 如果是最小节点，获得写锁
        if (currentNode.equals(children.get(0))) {
            return true;
        }
        
        // 否则等待前面的锁释放
        int index = children.indexOf(currentNode);
        return waitForLock(children.get(index - 1));
    }
    
    private boolean waitForLock(String prevNode) throws Exception {
        CountDownLatch latch = new CountDownLatch(1);
        
        Stat stat = zooKeeper.exists(lockPath + "/" + prevNode, event -> {
            if (event.getType() == Watcher.Event.EventType.NodeDeleted) {
                latch.countDown();
            }
        });
        
        if (stat != null) {
            latch.await();
        }
        
        return true;
    }
}
```

### 3.4 服务注册与发现

```java
import org.apache.zookeeper.*;

import java.util.List;

/**
 * 基于ZooKeeper的服务注册与发现
 * 
 * @author erik.zhou
 */
public class ServiceRegistry {
    
    private ZooKeeper zooKeeper;
    private static final String REGISTRY_PATH = "/services";
    
    /**
     * 注册服务
     */
    public void registerService(String serviceName, String serviceAddress) throws Exception {
        String servicePath = REGISTRY_PATH + "/" + serviceName;
        
        // 创建服务节点（持久节点）
        if (zooKeeper.exists(servicePath, false) == null) {
            zooKeeper.create(
                servicePath,
                new byte[0],
                ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.PERSISTENT
            );
        }
        
        // 创建服务实例节点（临时顺序节点）
        String instancePath = zooKeeper.create(
            servicePath + "/instance-",
            serviceAddress.getBytes(),
            ZooDefs.Ids.OPEN_ACL_UNSAFE,
            CreateMode.EPHEMERAL_SEQUENTIAL
        );
        
        System.out.println("服务注册成功: " + instancePath);
    }
    
    /**
     * 发现服务
     */
    public List<String> discoverService(String serviceName) throws Exception {
        String servicePath = REGISTRY_PATH + "/" + serviceName;
        
        // 获取所有服务实例
        List<String> instances = zooKeeper.getChildren(servicePath, true);
        
        List<String> addresses = new ArrayList<>();
        for (String instance : instances) {
            byte[] data = zooKeeper.getData(servicePath + "/" + instance, false, null);
            addresses.add(new String(data));
        }
        
        return addresses;
    }
    
    /**
     * 监听服务变化
     */
    public void watchService(String serviceName, ServiceChangeListener listener) throws Exception {
        String servicePath = REGISTRY_PATH + "/" + serviceName;
        
        zooKeeper.getChildren(servicePath, event -> {
            if (event.getType() == Watcher.Event.EventType.NodeChildrenChanged) {
                try {
                    List<String> addresses = discoverService(serviceName);
                    listener.onChange(addresses);
                    // 重新注册监听
                    watchService(serviceName, listener);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });
    }
    
    interface ServiceChangeListener {
        void onChange(List<String> addresses);
    }
}
```

### 3.5 配置中心实现

```java
/**
 * 基于ZooKeeper的配置中心
 * 
 * @author erik.zhou
 */
public class ConfigCenter {
    
    private ZooKeeper zooKeeper;
    private static final String CONFIG_PATH = "/config";
    private Map<String, String> localCache = new ConcurrentHashMap<>();
    
    /**
     * 获取配置
     */
    public String getConfig(String key) throws Exception {
        String path = CONFIG_PATH + "/" + key;
        
        // 先从本地缓存获取
        if (localCache.containsKey(key)) {
            return localCache.get(key);
        }
        
        // 从ZooKeeper获取
        byte[] data = zooKeeper.getData(path, event -> {
            if (event.getType() == Watcher.Event.EventType.NodeDataChanged) {
                try {
                    // 配置变化，更新本地缓存
                    updateLocalCache(key);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }, null);
        
        String value = new String(data);
        localCache.put(key, value);
        return value;
    }
    
    /**
     * 更新配置
     */
    public void setConfig(String key, String value) throws Exception {
        String path = CONFIG_PATH + "/" + key;
        
        if (zooKeeper.exists(path, false) == null) {
            zooKeeper.create(
                path,
                value.getBytes(),
                ZooDefs.Ids.OPEN_ACL_UNSAFE,
                CreateMode.PERSISTENT
            );
        } else {
            zooKeeper.setData(path, value.getBytes(), -1);
        }
        
        // 更新本地缓存
        localCache.put(key, value);
    }
    
    /**
     * 更新本地缓存
     */
    private void updateLocalCache(String key) throws Exception {
        String path = CONFIG_PATH + "/" + key;
        byte[] data = zooKeeper.getData(path, true, null);
        localCache.put(key, new String(data));
        System.out.println("配置已更新: " + key + " = " + new String(data));
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

#### 4.1.1 客户端优化

**连接池管理**:
```java
/**
 * ZooKeeper连接池
 * 
 * @author erik.zhou
 */
public class ZooKeeperPool {
    
    private static final int MAX_CONNECTIONS = 10;
    private BlockingQueue<ZooKeeper> pool = new LinkedBlockingQueue<>(MAX_CONNECTIONS);
    
    public ZooKeeperPool(String connectString, int sessionTimeout) throws Exception {
        for (int i = 0; i < MAX_CONNECTIONS; i++) {
            ZooKeeper zk = new ZooKeeper(connectString, sessionTimeout, event -> {});
            pool.offer(zk);
        }
    }
    
    public ZooKeeper getConnection() throws InterruptedException {
        return pool.take();
    }
    
    public void returnConnection(ZooKeeper zk) {
        pool.offer(zk);
    }
}
```

**批量操作**:
```java
// 使用multi操作批量提交
List<Op> ops = new ArrayList<>();
ops.add(Op.create("/path1", data1, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT));
ops.add(Op.create("/path2", data2, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT));
ops.add(Op.setData("/path3", data3, -1));

// 原子性批量执行
List<OpResult> results = zooKeeper.multi(ops);
```

**异步操作**:
```java
// 异步创建节点
zooKeeper.create(
    "/async/node",
    data.getBytes(),
    ZooDefs.Ids.OPEN_ACL_UNSAFE,
    CreateMode.PERSISTENT,
    (rc, path, ctx, name) -> {
        if (rc == KeeperException.Code.OK.intValue()) {
            System.out.println("创建成功: " + name);
        }
    },
    null
);
```

#### 4.1.2 服务端优化

**合理配置参数**:
```properties
# zoo.cfg
# 快照保留数量
autopurge.snapRetainCount=3
# 自动清理间隔（小时）
autopurge.purgeInterval=1

# JVM参数优化
# 堆内存
-Xms2g -Xmx2g
# GC配置
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
```

**数据压缩**:
```properties
# 启用数据压缩
jute.maxbuffer=4194304
```

**磁盘优化**:
```bash
# 使用SSD存储数据目录
dataDir=/ssd/zookeeper/data
# 事务日志单独存储
dataLogDir=/ssd/zookeeper/logs
```

### 4.2 高可用配置

#### 4.2.1 集群规模

**节点数量选择**:
```
3节点: 允许1个节点故障
5节点: 允许2个节点故障
7节点: 允许3个节点故障

推荐: 5节点（性能和可用性平衡）
```

#### 4.2.2 Observer节点

**配置Observer**:
```properties
# zoo.cfg
# 添加Observer节点（不参与投票）
server.1=192.168.1.101:2888:3888
server.2=192.168.1.102:2888:3888
server.3=192.168.1.103:2888:3888
server.4=192.168.1.104:2888:3888:observer
server.5=192.168.1.105:2888:3888:observer
```

**使用场景**:
- 跨地域部署
- 提升读性能
- 不影响写性能

#### 4.2.3 会话管理

**合理设置超时时间**:
```java
// 会话超时时间（毫秒）
int sessionTimeout = 30000; // 30秒

// 考虑因素:
// - 网络延迟
// - GC停顿时间
// - 业务处理时间
```

**会话重连机制**:
```java
public class ZooKeeperClient {
    
    private ZooKeeper zooKeeper;
    private CountDownLatch connectedLatch = new CountDownLatch(1);
    
    public void connect(String connectString, int sessionTimeout) throws Exception {
        zooKeeper = new ZooKeeper(connectString, sessionTimeout, event -> {
            if (event.getState() == Watcher.Event.KeeperState.SyncConnected) {
                connectedLatch.countDown();
            } else if (event.getState() == Watcher.Event.KeeperState.Disconnected) {
                // 连接断开，自动重连
                reconnect(connectString, sessionTimeout);
            }
        });
        
        connectedLatch.await();
    }
    
    private void reconnect(String connectString, int sessionTimeout) {
        try {
            Thread.sleep(1000);
            connect(connectString, sessionTimeout);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 4.3 常见陷阱

#### 4.3.1 Watch陷阱

**⚠️ 陷阱1: Watch只触发一次**
```java
// 错误: 只会收到一次通知
zooKeeper.getData("/config", event -> {
    System.out.println("配置变化");
}, null);

// 正确: 重新注册Watch
zooKeeper.getData("/config", event -> {
    System.out.println("配置变化");
    try {
        // 重新注册
        zooKeeper.getData("/config", this, null);
    } catch (Exception e) {
        e.printStackTrace();
    }
}, null);
```

**⚠️ 陷阱2: Watch回调阻塞**
```java
// 错误: 在Watch回调中执行耗时操作
zooKeeper.getData("/config", event -> {
    // 耗时操作会阻塞其他Watch回调
    doHeavyWork();
}, null);

// 正确: 使用线程池异步处理
ExecutorService executor = Executors.newFixedThreadPool(10);

zooKeeper.getData("/config", event -> {
    executor.submit(() -> {
        doHeavyWork();
    });
}, null);
```

#### 4.3.2 节点数据大小限制

**⚠️ 陷阱: 存储大数据**
```java
// 错误: ZNode最大1MB
byte[] largeData = new byte[2 * 1024 * 1024]; // 2MB
zooKeeper.create("/large", largeData, ...); // 失败

// 正确: 存储引用，数据存储在其他系统
String dataRef = "s3://bucket/large-data";
zooKeeper.create("/ref", dataRef.getBytes(), ...);
```

#### 4.3.3 临时节点陷阱

**⚠️ 陷阱: 会话超时导致临时节点删除**
```java
// 场景: 长时间GC导致会话超时
// 临时节点被删除，分布式锁失效

// 解决方案:
// 1. 增加会话超时时间
int sessionTimeout = 60000; // 60秒

// 2. 优化GC配置
// -XX:+UseG1GC -XX:MaxGCPauseMillis=200

// 3. 使用心跳保活
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(1);
scheduler.scheduleAtFixedRate(() -> {
    try {
        zooKeeper.exists("/heartbeat", false);
    } catch (Exception e) {
        e.printStackTrace();
    }
}, 0, 5, TimeUnit.SECONDS);
```

### 4.4 安全配置

#### 4.4.1 ACL权限控制

```java
/**
 * ACL权限控制示例
 * 
 * @author erik.zhou
 */
public class ACLExample {
    
    /**
     * 设置节点权限
     */
    public void setACL(ZooKeeper zk) throws Exception {
        // 创建认证信息
        zk.addAuthInfo("digest", "admin:admin123".getBytes());
        
        // 创建ACL列表
        List<ACL> acls = new ArrayList<>();
        
        // 管理员权限（所有权限）
        Id adminId = new Id("digest", DigestAuthenticationProvider.generateDigest("admin:admin123"));
        acls.add(new ACL(ZooDefs.Perms.ALL, adminId));
        
        // 普通用户权限（只读）
        Id userId = new Id("digest", DigestAuthenticationProvider.generateDigest("user:user123"));
        acls.add(new ACL(ZooDefs.Perms.READ, userId));
        
        // 创建带权限的节点
        zk.create("/secure", "data".getBytes(), acls, CreateMode.PERSISTENT);
    }
    
    /**
     * 使用认证访问节点
     */
    public void accessWithAuth(ZooKeeper zk) throws Exception {
        // 添加认证信息
        zk.addAuthInfo("digest", "admin:admin123".getBytes());
        
        // 访问受保护的节点
        byte[] data = zk.getData("/secure", false, null);
        System.out.println("数据: " + new String(data));
    }
}
```

#### 4.4.2 SSL/TLS加密

```properties
# zoo.cfg
# 启用SSL
secureClientPort=2281
ssl.keyStore.location=/path/to/keystore.jks
ssl.keyStore.password=password
ssl.trustStore.location=/path/to/truststore.jks
ssl.trustStore.password=password
```

```java
// 客户端SSL连接
System.setProperty("zookeeper.client.secure", "true");
System.setProperty("zookeeper.ssl.keyStore.location", "/path/to/keystore.jks");
System.setProperty("zookeeper.ssl.keyStore.password", "password");

ZooKeeper zk = new ZooKeeper("localhost:2281", 5000, event -> {});
```

### 4.5 监控与运维

#### 4.5.1 四字命令

```bash
# 查看服务器状态
echo stat | nc localhost 2181

# 查看配置信息
echo conf | nc localhost 2181

# 查看连接信息
echo cons | nc localhost 2181

# 查看监听信息
echo wchs | nc localhost 2181

# 查看环境变量
echo envi | nc localhost 2181

# 重置统计信息
echo srst | nc localhost 2181
```

#### 4.5.2 JMX监控

```properties
# zoo.cfg
# 启用JMX
jmx.port=9999
```

```java
// 使用JConsole或VisualVM连接
// localhost:9999
```

**关键指标**:
- `OutstandingRequests`: 待处理请求数
- `AvgRequestLatency`: 平均请求延迟
- `NumAliveConnections`: 活跃连接数
- `PacketsReceived`: 接收的数据包数
- `PacketsSent`: 发送的数据包数

#### 4.5.3 日志管理

```properties
# log4j.properties
# 日志级别
log4j.rootLogger=INFO, ROLLINGFILE

# 日志文件
log4j.appender.ROLLINGFILE=org.apache.log4j.RollingFileAppender
log4j.appender.ROLLINGFILE.File=/var/log/zookeeper/zookeeper.log
log4j.appender.ROLLINGFILE.MaxFileSize=100MB
log4j.appender.ROLLINGFILE.MaxBackupIndex=10
```

## ❓ 常见问题

### Q1: ZooKeeper适合存储什么样的数据？

**A**: ZooKeeper适合存储：
- 配置信息（小于1MB）
- 元数据
- 协调信息（锁、选举）
- 服务注册信息

**不适合存储**:
- 大量业务数据
- 频繁变化的数据
- 大文件（>1MB）

### Q2: ZooKeeper集群如何选择节点数量？

**A**: 
- **3节点**: 小规模应用，允许1个节点故障
- **5节点**: 推荐配置，允许2个节点故障
- **7节点**: 大规模应用，允许3个节点故障

**注意**: 
- 使用奇数个节点
- 节点越多，写性能越低
- 可以使用Observer节点提升读性能

### Q3: Watch机制为什么是一次性的？

**A**: 
- **性能考虑**: 避免服务器维护大量持久Watch
- **简化设计**: 降低系统复杂度
- **推荐做法**: 在Watch回调中重新注册

### Q4: 如何处理ZooKeeper的脑裂问题？

**A**: ZooKeeper通过以下机制避免脑裂：
1. **过半机制**: 只有超过半数节点的分区才能选举Leader
2. **Epoch机制**: 每次选举epoch递增，旧Leader无法提交事务
3. **推荐配置**: 使用奇数个节点，合理设置超时时间

### Q5: ZooKeeper和etcd、Consul有什么区别？

**A**: 

| 特性 | ZooKeeper | etcd | Consul |
|------|-----------|------|--------|
| 一致性协议 | ZAB | Raft | Raft |
| 语言 | Java | Go | Go |
| 性能 | 中等 | 高 | 高 |
| 易用性 | 较复杂 | 简单 | 简单 |
| 功能 | 协调服务 | KV存储 | 服务发现+KV |
| 社区 | 成熟 | 活跃 | 活跃 |

**选择建议**:
- **ZooKeeper**: 已有Hadoop/Kafka生态，需要强一致性
- **etcd**: Kubernetes生态，需要高性能KV存储
- **Consul**: 需要服务发现+健康检查+KV存储

### Q6: 如何优化ZooKeeper的性能？

**A**: 
1. **客户端优化**:
   - 使用连接池
   - 批量操作
   - 异步API
   - 本地缓存

2. **服务端优化**:
   - 使用SSD存储
   - 合理配置JVM参数
   - 启用数据压缩
   - 定期清理快照

3. **架构优化**:
   - 使用Observer节点
   - 读写分离
   - 合理的集群规模

### Q7: ZooKeeper会话超时如何处理？

**A**: 
1. **增加超时时间**: 根据网络和GC情况调整
2. **优化GC**: 使用G1GC，减少停顿时间
3. **心跳保活**: 定期发送心跳请求
4. **重连机制**: 实现自动重连逻辑
5. **监控告警**: 监控会话状态，及时告警

### Q8: 如何保证ZooKeeper的数据安全？

**A**: 
1. **ACL权限控制**: 设置节点访问权限
2. **SSL/TLS加密**: 加密客户端和服务器通信
3. **认证机制**: 使用digest或Kerberos认证
4. **数据备份**: 定期备份数据目录
5. **审计日志**: 记录所有操作日志

## 🔗 相关资源

### 官方资源
- **官方网站**: https://zookeeper.apache.org/
- **官方文档**: https://zookeeper.apache.org/doc/current/
- **GitHub仓库**: https://github.com/apache/zookeeper
- **邮件列表**: https://zookeeper.apache.org/mailing_lists.html

### 推荐书籍
- 《从Paxos到ZooKeeper：分布式一致性原理与实践》- 倪超
- 《ZooKeeper: Distributed Process Coordination》- Flavio Junqueira, Benjamin Reed

### 推荐文章
- ZooKeeper官方Wiki: https://cwiki.apache.org/confluence/display/ZOOKEEPER
- ZooKeeper Recipes: https://zookeeper.apache.org/doc/current/recipes.html
- ZooKeeper Internals: https://zookeeper.apache.org/doc/current/zookeeperInternals.html

### 视频教程
- Apache ZooKeeper官方YouTube频道
- 各大技术平台的ZooKeeper实战课程

### 相关技术
- **Curator**: ZooKeeper客户端框架 (https://curator.apache.org/)
- **Kafka**: 使用ZooKeeper做协调 (https://kafka.apache.org/)
- **Hadoop**: 使用ZooKeeper做HA (https://hadoop.apache.org/)
- **Dubbo**: 使用ZooKeeper做注册中心 (https://dubbo.apache.org/)

## 📝 学习检查清单

### 基础知识
- [ ] 理解ZooKeeper的定位和应用场景
- [ ] 掌握ZNode的四种类型及其特点
- [ ] 理解Session的概念和生命周期
- [ ] 掌握Watch机制的特性和使用方法
- [ ] 了解ACL权限控制机制

### 核心原理
- [ ] 理解ZAB协议的工作流程
- [ ] 掌握Leader选举机制
- [ ] 理解ZXID的组成和作用
- [ ] 了解过半机制的原理
- [ ] 理解脑裂问题及其解决方案

### 实战能力
- [ ] 能够搭建单机和集群环境
- [ ] 能够使用Java客户端进行基本操作
- [ ] 能够实现分布式锁
- [ ] 能够实现服务注册与发现
- [ ] 能够实现配置中心

### 进阶能力
- [ ] 掌握性能优化方法
- [ ] 了解高可用配置
- [ ] 掌握监控和运维技巧
- [ ] 能够处理常见问题
- [ ] 了解与其他技术的对比

### 最佳实践
- [ ] 掌握客户端优化技巧
- [ ] 了解服务端配置优化
- [ ] 避免常见陷阱
- [ ] 掌握安全配置方法
- [ ] 了解生产环境运维要点

---

## 附录：Curator框架快速入门

Curator是Netflix开源的ZooKeeper客户端框架，提供了更高级的API和常用的分布式协调功能。

### Maven依赖

```xml
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-framework</artifactId>
    <version>5.5.0</version>
</dependency>
<dependency>
    <groupId>org.apache.curator</groupId>
    <artifactId>curator-recipes</artifactId>
    <version>5.5.0</version>
</dependency>
```

### 基础使用

```java
import org.apache.curator.framework.CuratorFramework;
import org.apache.curator.framework.CuratorFrameworkFactory;
import org.apache.curator.retry.ExponentialBackoffRetry;

/**
 * Curator基础使用示例
 * 
 * @author erik.zhou
 */
public class CuratorExample {
    
    public static void main(String[] args) throws Exception {
        // 创建客户端
        CuratorFramework client = CuratorFrameworkFactory.builder()
            .connectString("localhost:2181")
            .sessionTimeoutMs(5000)
            .connectionTimeoutMs(3000)
            .retryPolicy(new ExponentialBackoffRetry(1000, 3))
            .build();
        
        // 启动客户端
        client.start();
        
        // 创建节点
        client.create()
            .creatingParentsIfNeeded()
            .forPath("/curator/test", "data".getBytes());
        
        // 获取数据
        byte[] data = client.getData().forPath("/curator/test");
        System.out.println("数据: " + new String(data));
        
        // 更新数据
        client.setData().forPath("/curator/test", "newdata".getBytes());
        
        // 删除节点
        client.delete()
            .deletingChildrenIfNeeded()
            .forPath("/curator");
        
        // 关闭客户端
        client.close();
    }
}
```

### 分布式锁

```java
import org.apache.curator.framework.recipes.locks.InterProcessMutex;

/**
 * Curator分布式锁
 * 
 * @author erik.zhou
 */
public class CuratorLockExample {
    
    public static void main(String[] args) throws Exception {
        CuratorFramework client = CuratorFrameworkFactory.builder()
            .connectString("localhost:2181")
            .retryPolicy(new ExponentialBackoffRetry(1000, 3))
            .build();
        client.start();
        
        // 创建分布式锁
        InterProcessMutex lock = new InterProcessMutex(client, "/locks/mylock");
        
        try {
            // 获取锁
            if (lock.acquire(10, TimeUnit.SECONDS)) {
                try {
                    // 执行业务逻辑
                    System.out.println("获得锁，执行业务");
                    Thread.sleep(5000);
                } finally {
                    // 释放锁
                    lock.release();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        
        client.close();
    }
}
```

### 服务发现

```java
import org.apache.curator.x.discovery.ServiceDiscovery;
import org.apache.curator.x.discovery.ServiceDiscoveryBuilder;
import org.apache.curator.x.discovery.ServiceInstance;
import org.apache.curator.x.discovery.UriSpec;

/**
 * Curator服务发现
 * 
 * @author erik.zhou
 */
public class CuratorServiceDiscoveryExample {
    
    public static void main(String[] args) throws Exception {
        CuratorFramework client = CuratorFrameworkFactory.builder()
            .connectString("localhost:2181")
            .retryPolicy(new ExponentialBackoffRetry(1000, 3))
            .build();
        client.start();
        
        // 创建服务发现
        ServiceDiscovery<String> serviceDiscovery = ServiceDiscoveryBuilder.builder(String.class)
            .client(client)
            .basePath("/services")
            .build();
        serviceDiscovery.start();
        
        // 注册服务
        ServiceInstance<String> instance = ServiceInstance.<String>builder()
            .name("myservice")
            .address("192.168.1.100")
            .port(8080)
            .uriSpec(new UriSpec("{scheme}://{address}:{port}"))
            .build();
        serviceDiscovery.registerService(instance);
        
        // 发现服务
        Collection<ServiceInstance<String>> instances = 
            serviceDiscovery.queryForInstances("myservice");
        for (ServiceInstance<String> inst : instances) {
            System.out.println("服务地址: " + inst.buildUriSpec());
        }
        
        // 关闭
        serviceDiscovery.close();
        client.close();
    }
}
```

---

**文档版本**: v1.0  
**最后更新**: 2024-01  
**文档来源**: 官方文档整理  
**@author erik.zhou**
