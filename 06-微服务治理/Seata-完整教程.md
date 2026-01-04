# Seata 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [事务模式详解](#事务模式详解)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [相关资源](#相关资源)

## 📚 技术概述
- **版本**: 2.x
- **官方文档**: https://seata.apache.org/
- **GitHub**: https://github.com/apache/incubator-seata
- **学习难度**: ⭐⭐⭐⭐⭐ (5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (5星)
- **前置知识**: 
  - Spring Boot
  - Spring Cloud
  - 分布式系统基础
  - MySQL事务原理
  - 微服务架构

## 🎯 学习目标
- [ ] 理解分布式事务的核心概念和挑战
- [ ] 掌握Seata的架构设计和核心组件
- [ ] 熟练使用AT、TCC、Saga、XA四种事务模式
- [ ] 能够在Spring Cloud项目中集成Seata
- [ ] 掌握Seata的性能优化和故障排查
- [ ] 理解分布式事务的最佳实践

## 📖 基础概念

### 1.1 什么是Seata

Seata（Simple Extensible Autonomous Transaction Architecture）是一款开源的分布式事务解决方案，致力于在微服务架构下提供高性能和简单易用的分布式事务服务。

**核心价值**：
- 对业务无侵入：通过代理数据源自动完成分支事务的注册、提交和回滚
- 高性能：本地事务提交后即可释放数据库连接，不阻塞业务
- 易于使用：一个注解即可开启全局事务

### 1.2 分布式事务的挑战

在微服务架构中，一个业务操作通常需要跨越多个服务和数据库，传统的本地事务无法保证数据一致性：


**典型场景**：
```
用户下单 -> 订单服务创建订单 -> 库存服务扣减库存 -> 账户服务扣减余额
```

**面临的问题**：
1. **原子性问题**：如何保证所有操作要么全部成功，要么全部失败？
2. **一致性问题**：如何保证各个服务的数据状态一致？
3. **隔离性问题**：如何处理并发事务的相互影响？
4. **性能问题**：如何在保证一致性的同时不影响系统性能？

### 1.3 Seata的核心组件

Seata定义了三个核心组件：

**TC (Transaction Coordinator) - 事务协调器**
- 维护全局事务和分支事务的状态
- 驱动全局事务提交或回滚
- 独立部署的服务端

**TM (Transaction Manager) - 事务管理器**
- 定义全局事务的范围
- 开始全局事务、提交或回滚全局事务
- 通常是业务服务的发起方

**RM (Resource Manager) - 资源管理器**
- 管理分支事务处理的资源
- 与TC交互注册分支事务和报告分支事务状态
- 驱动分支事务提交或回滚
- 通常是业务服务的参与方

### 1.4 Seata的工作流程

```
1. TM 向 TC 申请开启一个全局事务，TC 返回全局事务ID (XID)
2. XID 在微服务调用链路的上下文中传播
3. RM 向 TC 注册分支事务，将其纳入 XID 对应的全局事务管辖
4. TM 向 TC 发起针对 XID 的全局提交或回滚决议
5. TC 调度 XID 下管辖的全部分支事务完成提交或回滚
```

## 🔥 核心特性 (重点)

### 2.1 四种事务模式

Seata支持四种事务模式，适用于不同的业务场景：

| 模式 | 适用场景 | 性能 | 侵入性 | 一致性 |
|------|---------|------|--------|--------|
| AT | 基于SQL的关系型数据库 | 高 | 低 | 最终一致性 |
| TCC | 需要自定义事务资源 | 中 | 高 | 强一致性 |
| Saga | 长事务、异步场景 | 高 | 中 | 最终一致性 |
| XA | 需要强一致性 | 低 | 低 | 强一致性 |

### 2.2 AT模式 - 自动补偿 🔥

**核心原理**：
- 基于支持本地ACID事务的关系型数据库
- 一阶段：业务数据和回滚日志记录在同一个本地事务中提交，释放本地锁和连接资源
- 二阶段：
  - 提交：异步化，非常快速地完成
  - 回滚：通过一阶段的回滚日志进行反向补偿

**工作机制**：
```
一阶段：
1. 解析SQL，获取SQL类型、表名、条件等信息
2. 查询前镜像：根据解析得到的条件信息，生成查询语句，定位数据
3. 执行业务SQL
4. 查询后镜像：根据前镜像的结果，通过主键定位数据
5. 插入回滚日志：把前后镜像数据以及业务SQL相关信息组成一条回滚日志记录，插入到UNDO_LOG表中
6. 提交前，向TC注册分支：申请表中涉及记录的全局锁
7. 本地事务提交：业务数据的更新和前面生成的UNDO_LOG一并提交
8. 将本地事务提交的结果上报给TC

二阶段-提交：
1. 收到TC的分支提交请求，把请求放入一个异步任务队列，马上返回提交成功的结果给TC
2. 异步任务阶段的分支提交请求将异步和批量地删除相应UNDO_LOG记录

二阶段-回滚：
1. 收到TC的分支回滚请求，开启一个本地事务，执行如下操作
2. 通过XID和Branch ID查找到相应的UNDO_LOG记录
3. 数据校验：拿UNDO_LOG中的后镜像与当前数据进行比较，如果有不同，说明数据被当前全局事务之外的动作做了修改，需要人工处理
4. 根据UNDO_LOG中的前镜像和业务SQL的相关信息生成并执行回滚的语句
5. 提交本地事务，并把本地事务的执行结果（即分支事务回滚的结果）上报给TC
```

### 2.3 TCC模式 - 手动补偿 (⚠️ 难点)

**核心原理**：
- Try-Confirm-Cancel 三阶段提交协议
- 需要业务系统自己实现Try、Confirm、Cancel三个操作
- 不依赖于底层数据资源的事务支持

**三个阶段**：

1. **Try阶段**：尝试执行，完成所有业务检查，预留必须的业务资源
2. **Confirm阶段**：确认执行真正执行业务，不做任何业务检查，只使用Try阶段预留的业务资源
3. **Cancel阶段**：取消执行，释放Try阶段预留的业务资源

**TCC与AT的区别**：
- AT模式基于SQL拦截，自动生成补偿操作
- TCC模式需要业务自己实现补偿逻辑
- TCC模式可以跨数据库、跨应用资源
- TCC模式对业务侵入性更强，但灵活性更高

**TCC设计要点**：
- **允许空回滚**：Try未执行，Cancel执行了
- **防悬挂控制**：Cancel比Try先执行
- **幂等控制**：重复调用Confirm/Cancel
- **资源预留**：Try阶段预留资源，不直接操作

### 2.4 Saga模式 - 长事务

**核心原理**：
- 将长事务拆分为多个本地短事务
- 由Saga事务协调器协调
- 如果每个本地事务都成功提交，则全局事务成功
- 如果某个本地事务失败，则补偿所有已完成的事务

**适用场景**：
- 业务流程长、业务流程多
- 参与者包含其他公司或遗留系统服务，无法提供TCC模式要求的三个接口
- 典型场景：金融核心系统、互联网微贷业务

**实现方式**：
- 状态机模式：通过状态图定义服务调用的流程
- 注解模式：通过注解定义服务调用的流程

### 2.5 XA模式 - 强一致性

**核心原理**：
- 基于XA协议的两阶段提交
- 利用数据库本身的XA支持
- 强一致性，但性能较差

**工作流程**：
```
一阶段：
1. RM注册分支事务到TC
2. RM执行分支业务SQL但不提交
3. 报告执行状态到TC

二阶段：
- 提交：TC通知RM提交事务
- 回滚：TC通知RM回滚事务
```

**XA与AT的区别**：
- XA模式在一阶段不提交事务，锁定资源
- AT模式在一阶段提交事务，释放资源
- XA模式性能较差，但一致性更强

## 💻 事务模式详解

### 3.1 AT模式实战

#### 3.1.1 环境准备

**1. 创建UNDO_LOG表**

每个业务数据库都需要创建undo_log表：

```sql
CREATE TABLE `undo_log` (
  `id` BIGINT(20) NOT NULL AUTO_INCREMENT,
  `branch_id` BIGINT(20) NOT NULL,
  `xid` VARCHAR(100) NOT NULL,
  `context` VARCHAR(128) NOT NULL,
  `rollback_info` LONGBLOB NOT NULL,
  `log_status` INT(11) NOT NULL,
  `log_created` DATETIME NOT NULL,
  `log_modified` DATETIME NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

**2. 添加依赖**

```xml
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
```

**3. 配置文件**

```yaml
seata:
  enabled: true
  application-id: order-service
  tx-service-group: default_tx_group
  registry:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      namespace: ""
      group: SEATA_GROUP
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
      namespace: ""
      group: SEATA_GROUP
```

#### 3.1.2 业务代码

**订单服务 - 事务发起方**

```java
/**
 * 订单服务
 * @author erik.zhou
 */
@Service
public class OrderService {
    
    @Autowired
    private OrderMapper orderMapper;
    
    @Autowired
    private StorageService storageService;
    
    @Autowired
    private AccountService accountService;
    
    /**
     * 创建订单
     * 使用@GlobalTransactional注解开启全局事务
     */
    @GlobalTransactional(name = "create-order", rollbackFor = Exception.class)
    public void createOrder(OrderDTO orderDTO) {
        // 1. 创建订单
        Order order = new Order();
        order.setUserId(orderDTO.getUserId());
        order.setProductId(orderDTO.getProductId());
        order.setCount(orderDTO.getCount());
        order.setMoney(orderDTO.getMoney());
        order.setStatus(0); // 订单状态：0-创建中
        orderMapper.insert(order);
        
        // 2. 扣减库存
        storageService.deduct(orderDTO.getProductId(), orderDTO.getCount());
        
        // 3. 扣减账户余额
        accountService.deduct(orderDTO.getUserId(), orderDTO.getMoney());
        
        // 4. 更新订单状态
        order.setStatus(1); // 订单状态：1-已完成
        orderMapper.updateById(order);
    }
}
```


**库存服务 - 事务参与方**

```java
/**
 * 库存服务
 * @author erik.zhou
 */
@Service
public class StorageService {
    
    @Autowired
    private StorageMapper storageMapper;
    
    /**
     * 扣减库存
     * 不需要添加@GlobalTransactional注解
     * Seata会自动将此方法纳入全局事务
     */
    @Transactional(rollbackFor = Exception.class)
    public void deduct(Long productId, Integer count) {
        Storage storage = storageMapper.selectById(productId);
        if (storage == null) {
            throw new RuntimeException("商品不存在");
        }
        if (storage.getStock() < count) {
            throw new RuntimeException("库存不足");
        }
        storage.setStock(storage.getStock() - count);
        storageMapper.updateById(storage);
    }
}
```

**账户服务 - 事务参与方**

```java
/**
 * 账户服务
 * @author erik.zhou
 */
@Service
public class AccountService {
    
    @Autowired
    private AccountMapper accountMapper;
    
    /**
     * 扣减账户余额
     */
    @Transactional(rollbackFor = Exception.class)
    public void deduct(Long userId, BigDecimal money) {
        Account account = accountMapper.selectById(userId);
        if (account == null) {
            throw new RuntimeException("账户不存在");
        }
        if (account.getBalance().compareTo(money) < 0) {
            throw new RuntimeException("余额不足");
        }
        account.setBalance(account.getBalance().subtract(money));
        accountMapper.updateById(account);
    }
}
```

### 3.2 TCC模式实战

#### 3.2.1 TCC接口定义

```java
/**
 * TCC账户服务接口
 * @author erik.zhou
 */
@LocalTCC
public interface TccAccountService {
    
    /**
     * Try阶段：冻结金额
     * @param userId 用户ID
     * @param money 金额
     * @param businessKey 业务主键
     * @return 是否成功
     */
    @TwoPhaseBusinessAction(
        name = "deductAccount",
        commitMethod = "commit",
        rollbackMethod = "rollback"
    )
    boolean prepare(
        @BusinessActionContextParameter(paramName = "userId") Long userId,
        @BusinessActionContextParameter(paramName = "money") BigDecimal money,
        @BusinessActionContextParameter(paramName = "businessKey") String businessKey
    );
    
    /**
     * Confirm阶段：确认扣款
     * @param context 上下文
     * @return 是否成功
     */
    boolean commit(BusinessActionContext context);
    
    /**
     * Cancel阶段：解冻金额
     * @param context 上下文
     * @return 是否成功
     */
    boolean rollback(BusinessActionContext context);
}
```

#### 3.2.2 TCC接口实现

```java
/**
 * TCC账户服务实现
 * @author erik.zhou
 */
@Service
public class TccAccountServiceImpl implements TccAccountService {
    
    @Autowired
    private AccountMapper accountMapper;
    
    @Autowired
    private AccountFreezeMapper freezeMapper;
    
    /**
     * Try阶段：冻结金额
     * 1. 检查账户余额是否充足
     * 2. 冻结相应金额
     * 3. 记录冻结记录（用于幂等和防悬挂）
     */
    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean prepare(Long userId, BigDecimal money, String businessKey) {
        // 1. 幂等判断：检查是否已经执行过Try
        AccountFreeze freeze = freezeMapper.selectByBusinessKey(businessKey);
        if (freeze != null) {
            // 已经执行过Try，直接返回成功
            return true;
        }
        
        // 2. 检查账户余额
        Account account = accountMapper.selectById(userId);
        if (account == null) {
            throw new RuntimeException("账户不存在");
        }
        if (account.getBalance().compareTo(money) < 0) {
            throw new RuntimeException("余额不足");
        }
        
        // 3. 冻结金额
        account.setBalance(account.getBalance().subtract(money));
        account.setFrozen(account.getFrozen().add(money));
        accountMapper.updateById(account);
        
        // 4. 记录冻结记录
        freeze = new AccountFreeze();
        freeze.setUserId(userId);
        freeze.setMoney(money);
        freeze.setBusinessKey(businessKey);
        freeze.setState(AccountFreeze.State.TRY);
        freezeMapper.insert(freeze);
        
        return true;
    }
    
    /**
     * Confirm阶段：确认扣款
     * 1. 检查Try是否执行
     * 2. 扣减冻结金额
     * 3. 删除冻结记录
     */
    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean commit(BusinessActionContext context) {
        // 1. 获取Try阶段的参数
        Long userId = Long.valueOf(context.getActionContext("userId").toString());
        String businessKey = context.getActionContext("businessKey").toString();
        
        // 2. 幂等判断：检查冻结记录
        AccountFreeze freeze = freezeMapper.selectByBusinessKey(businessKey);
        if (freeze == null) {
            // 空提交：Try未执行，Confirm执行了
            return true;
        }
        if (freeze.getState() == AccountFreeze.State.CONFIRM) {
            // 已经执行过Confirm
            return true;
        }
        
        // 3. 扣减冻结金额
        Account account = accountMapper.selectById(userId);
        account.setFrozen(account.getFrozen().subtract(freeze.getMoney()));
        accountMapper.updateById(account);
        
        // 4. 更新冻结记录状态
        freeze.setState(AccountFreeze.State.CONFIRM);
        freezeMapper.updateById(freeze);
        
        return true;
    }
    

    /**
     * Cancel阶段：解冻金额
     * 1. 检查Try是否执行
     * 2. 恢复冻结金额到可用余额
     * 3. 删除冻结记录
     */
    @Override
    @Transactional(rollbackFor = Exception.class)
    public boolean rollback(BusinessActionContext context) {
        // 1. 获取Try阶段的参数
        Long userId = Long.valueOf(context.getActionContext("userId").toString());
        String businessKey = context.getActionContext("businessKey").toString();
        
        // 2. 查询冻结记录
        AccountFreeze freeze = freezeMapper.selectByBusinessKey(businessKey);
        if (freeze == null) {
            // 空回滚：Try未执行，Cancel执行了
            // 需要插入一条记录，防止后续Try执行（防悬挂）
            freeze = new AccountFreeze();
            freeze.setUserId(userId);
            freeze.setMoney(BigDecimal.ZERO);
            freeze.setBusinessKey(businessKey);
            freeze.setState(AccountFreeze.State.CANCEL);
            freezeMapper.insert(freeze);
            return true;
        }
        if (freeze.getState() == AccountFreeze.State.CANCEL) {
            // 已经执行过Cancel
            return true;
        }
        
        // 3. 恢复冻结金额
        Account account = accountMapper.selectById(userId);
        account.setBalance(account.getBalance().add(freeze.getMoney()));
        account.setFrozen(account.getFrozen().subtract(freeze.getMoney()));
        accountMapper.updateById(account);
        
        // 4. 更新冻结记录状态
        freeze.setState(AccountFreeze.State.CANCEL);
        freezeMapper.updateById(freeze);
        
        return true;
    }
}
```

#### 3.2.3 TCC事务发起

```java
/**
 * 订单服务 - TCC模式
 * @author erik.zhou
 */
@Service
public class OrderTccService {
    
    @Autowired
    private TccAccountService tccAccountService;
    
    /**
     * 创建订单 - TCC模式
     */
    @GlobalTransactional(name = "create-order-tcc", rollbackFor = Exception.class)
    public void createOrder(OrderDTO orderDTO) {
        String businessKey = UUID.randomUUID().toString();
        
        // 调用TCC服务的Try方法
        boolean result = tccAccountService.prepare(
            orderDTO.getUserId(),
            orderDTO.getMoney(),
            businessKey
        );
        
        if (!result) {
            throw new RuntimeException("账户扣款失败");
        }
        
        // 其他业务逻辑...
    }
}
```

### 3.3 Saga模式实战

#### 3.3.1 状态机定义

创建状态机JSON配置文件：

```json
{
  "Name": "order-saga",
  "Comment": "订单处理Saga状态机",
  "StartState": "CreateOrder",
  "Version": "0.0.1",
  "States": {
    "CreateOrder": {
      "Type": "ServiceTask",
      "ServiceName": "orderService",
      "ServiceMethod": "createOrder",
      "CompensateState": "CancelOrder",
      "Next": "DeductStorage",
      "Input": [
        "$.[orderDTO]"
      ],
      "Output": {
        "orderId": "$.orderId"
      },
      "Status": {
        "#root == true": "SU",
        "#root == false": "FA",
        "$Exception{java.lang.Throwable}": "UN"
      }
    },
    "DeductStorage": {
      "Type": "ServiceTask",
      "ServiceName": "storageService",
      "ServiceMethod": "deduct",
      "CompensateState": "RestoreStorage",
      "Next": "DeductAccount",
      "Input": [
        "$.[orderDTO.productId]",
        "$.[orderDTO.count]"
      ],
      "Status": {
        "#root == true": "SU",
        "#root == false": "FA",
        "$Exception{java.lang.Throwable}": "UN"
      }
    },
    "DeductAccount": {
      "Type": "ServiceTask",
      "ServiceName": "accountService",
      "ServiceMethod": "deduct",
      "CompensateState": "RestoreAccount",
      "Next": "Succeed",
      "Input": [
        "$.[orderDTO.userId]",
        "$.[orderDTO.money]"
      ],
      "Status": {
        "#root == true": "SU",
        "#root == false": "FA",
        "$Exception{java.lang.Throwable}": "UN"
      }
    },
    "CancelOrder": {
      "Type": "ServiceTask",
      "ServiceName": "orderService",
      "ServiceMethod": "cancelOrder",
      "Input": [
        "$.[orderId]"
      ]
    },
    "RestoreStorage": {
      "Type": "ServiceTask",
      "ServiceName": "storageService",
      "ServiceMethod": "restore",
      "Input": [
        "$.[orderDTO.productId]",
        "$.[orderDTO.count]"
      ]
    },
    "RestoreAccount": {
      "Type": "ServiceTask",
      "ServiceName": "accountService",
      "ServiceMethod": "restore",
      "Input": [
        "$.[orderDTO.userId]",
        "$.[orderDTO.money]"
      ]
    },
    "Succeed": {
      "Type": "Succeed"
    }
  }
}
```

#### 3.3.2 Saga服务实现

```java
/**
 * 订单服务 - Saga模式
 * @author erik.zhou
 */
@Service
public class OrderSagaService {
    
    @Autowired
    private OrderMapper orderMapper;
    
    /**
     * 创建订单 - 正向操作
     */
    public boolean createOrder(OrderDTO orderDTO) {
        Order order = new Order();
        order.setUserId(orderDTO.getUserId());
        order.setProductId(orderDTO.getProductId());
        order.setCount(orderDTO.getCount());
        order.setMoney(orderDTO.getMoney());
        order.setStatus(0);
        orderMapper.insert(order);
        return true;
    }
    
    /**
     * 取消订单 - 补偿操作
     */
    public boolean cancelOrder(Long orderId) {
        Order order = orderMapper.selectById(orderId);
        if (order != null) {
            order.setStatus(-1); // 订单状态：-1-已取消
            orderMapper.updateById(order);
        }
        return true;
    }
}
```


#### 3.3.3 Saga状态机执行

```java
/**
 * Saga状态机执行器
 * @author erik.zhou
 */
@Service
public class SagaExecutor {
    
    @Autowired
    private StateMachineEngine stateMachineEngine;
    
    /**
     * 执行Saga状态机
     */
    public void execute(OrderDTO orderDTO) {
        // 准备输入参数
        Map<String, Object> startParams = new HashMap<>();
        startParams.put("orderDTO", orderDTO);
        
        // 执行状态机
        StateMachineInstance instance = stateMachineEngine.startWithBusinessKey(
            "order-saga",
            null,
            UUID.randomUUID().toString(),
            startParams
        );
        
        // 检查执行结果
        if (ExecutionStatus.SU.equals(instance.getStatus())) {
            System.out.println("订单创建成功");
        } else {
            System.out.println("订单创建失败：" + instance.getException());
        }
    }
}
```

### 3.4 XA模式实战

#### 3.4.1 配置XA数据源

```java
/**
 * XA数据源配置
 * @author erik.zhou
 */
@Configuration
public class XADataSourceConfig {
    
    @Bean
    @ConfigurationProperties(prefix = "spring.datasource")
    public DataSource dataSource() {
        // 使用Seata的XA数据源代理
        return new DataSourceProxyXA(createAtomikosDataSource());
    }
    
    private DataSource createAtomikosDataSource() {
        MysqlXADataSource xaDataSource = new MysqlXADataSource();
        xaDataSource.setUrl("jdbc:mysql://localhost:3306/seata_order");
        xaDataSource.setUser("root");
        xaDataSource.setPassword("root");
        
        AtomikosDataSourceBean atomikosDataSource = new AtomikosDataSourceBean();
        atomikosDataSource.setXaDataSource(xaDataSource);
        atomikosDataSource.setUniqueResourceName("orderDataSource");
        
        return atomikosDataSource;
    }
}
```

#### 3.4.2 XA事务使用

```java
/**
 * 订单服务 - XA模式
 * @author erik.zhou
 */
@Service
public class OrderXAService {
    
    @Autowired
    private OrderMapper orderMapper;
    
    @Autowired
    private StorageService storageService;
    
    /**
     * 创建订单 - XA模式
     * 使用@GlobalTransactional注解，Seata会自动使用XA模式
     */
    @GlobalTransactional(name = "create-order-xa", rollbackFor = Exception.class)
    public void createOrder(OrderDTO orderDTO) {
        // 1. 创建订单
        Order order = new Order();
        order.setUserId(orderDTO.getUserId());
        order.setProductId(orderDTO.getProductId());
        order.setCount(orderDTO.getCount());
        order.setMoney(orderDTO.getMoney());
        orderMapper.insert(order);
        
        // 2. 扣减库存
        storageService.deduct(orderDTO.getProductId(), orderDTO.getCount());
        
        // XA模式下，事务在二阶段才提交
        // 一阶段会持有数据库锁，直到全局事务结束
    }
}
```

## ✨ 最佳实践

### 4.1 事务模式选择 🔥

**选择决策树**：

```
是否需要强一致性？
├─ 是 → 使用XA模式（性能较差）
└─ 否 → 是否基于关系型数据库？
    ├─ 是 → 使用AT模式（推荐）
    └─ 否 → 是否需要自定义补偿逻辑？
        ├─ 是 → 使用TCC模式
        └─ 否 → 使用Saga模式
```

**各模式适用场景**：

| 场景 | 推荐模式 | 原因 |
|------|---------|------|
| 电商下单 | AT | 基于MySQL，对业务无侵入 |
| 金融转账 | TCC | 需要精确控制资金流转 |
| 订单审批流程 | Saga | 长事务，多步骤 |
| 跨行转账 | XA | 需要强一致性 |

### 4.2 性能优化 (⚠️ 难点)

#### 4.2.1 AT模式优化

**1. 异步提交优化**

```yaml
seata:
  client:
    rm:
      # 异步提交缓冲区大小
      async-commit-buffer-limit: 10000
      # 报告成功状态（可关闭以提升性能）
      report-success-enable: false
```

**2. 批量操作优化**

```java
/**
 * 批量操作优化
 * @author erik.zhou
 */
@Service
public class BatchOptimizationService {
    
    /**
     * 错误示例：循环调用远程服务
     */
    @GlobalTransactional
    public void badExample(List<OrderDTO> orders) {
        for (OrderDTO order : orders) {
            // 每次循环都会注册一个分支事务
            storageService.deduct(order.getProductId(), order.getCount());
        }
    }
    
    /**
     * 正确示例：批量调用
     */
    @GlobalTransactional
    public void goodExample(List<OrderDTO> orders) {
        // 一次性批量扣减库存，只注册一个分支事务
        Map<Long, Integer> deductMap = orders.stream()
            .collect(Collectors.groupingBy(
                OrderDTO::getProductId,
                Collectors.summingInt(OrderDTO::getCount)
            ));
        storageService.batchDeduct(deductMap);
    }
}
```

**3. UNDO_LOG清理优化**

```yaml
seata:
  server:
    undo:
      # UNDO_LOG保留天数
      log-save-days: 7
      # UNDO_LOG删除周期（毫秒）
      log-delete-period: 86400000
```

#### 4.2.2 TCC模式优化

**1. 减少数据库交互**

```java
/**
 * TCC优化示例
 * @author erik.zhou
 */
@Service
public class TccOptimizationService {
    
    /**
     * 优化：使用Redis缓存减少数据库查询
     */
    @Override
    public boolean prepare(Long userId, BigDecimal money, String businessKey) {
        // 1. 先从Redis检查幂等
        String cacheKey = "tcc:freeze:" + businessKey;
        if (redisTemplate.hasKey(cacheKey)) {
            return true;
        }
        
        // 2. 执行Try逻辑
        // ...
        
        // 3. 写入Redis缓存
        redisTemplate.opsForValue().set(cacheKey, "1", 24, TimeUnit.HOURS);
        
        return true;
    }
}
```

**2. 异步化Confirm/Cancel**

```java
/**
 * 异步化TCC二阶段
 * @author erik.zhou
 */
@Service
public class AsyncTccService {
    
    @Autowired
    private ThreadPoolExecutor executor;
    
    @Override
    public boolean commit(BusinessActionContext context) {
        // 异步执行Confirm，快速返回
        executor.execute(() -> {
            doCommit(context);
        });
        return true;
    }
    
    private void doCommit(BusinessActionContext context) {
        // 实际的Confirm逻辑
        // ...
    }
}
```


#### 4.2.3 全局锁优化

**1. 减少锁冲突**

```java
/**
 * 全局锁优化
 * @author erik.zhou
 */
@Service
public class GlobalLockOptimizationService {
    
    /**
     * 优化前：锁整个订单表
     */
    @GlobalTransactional
    public void badExample(Long orderId) {
        // SELECT * FROM order WHERE id = ? FOR UPDATE
        // 会锁住整行，包括不需要修改的字段
        Order order = orderMapper.selectById(orderId);
        order.setStatus(1);
        orderMapper.updateById(order);
    }
    
    /**
     * 优化后：只锁需要修改的字段
     */
    @GlobalTransactional
    public void goodExample(Long orderId) {
        // UPDATE order SET status = 1 WHERE id = ?
        // 只锁status字段，减少锁冲突
        orderMapper.updateStatus(orderId, 1);
    }
}
```

**2. 全局锁重试配置**

```yaml
seata:
  client:
    rm:
      lock:
        # 获取全局锁重试间隔（毫秒）
        retry-interval: 10
        # 获取全局锁重试次数
        retry-times: 30
        # 分支回滚冲突时的重试策略
        retry-policy-branch-rollback-on-conflict: true
```

### 4.3 高可用部署

#### 4.3.1 Seata Server集群部署

**1. 数据库存储模式**

```yaml
# Seata Server配置
seata:
  config:
    type: nacos
  registry:
    type: nacos
  store:
    mode: db
    db:
      datasource: druid
      db-type: mysql
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://127.0.0.1:3306/seata?useSSL=false
      user: root
      password: root
      min-conn: 5
      max-conn: 100
```

**2. 创建Seata Server数据库表**

```sql
-- 全局事务表
CREATE TABLE `global_table` (
  `xid` VARCHAR(128) NOT NULL,
  `transaction_id` BIGINT,
  `status` TINYINT NOT NULL,
  `application_id` VARCHAR(32),
  `transaction_service_group` VARCHAR(32),
  `transaction_name` VARCHAR(128),
  `timeout` INT,
  `begin_time` BIGINT,
  `application_data` VARCHAR(2000),
  `gmt_create` DATETIME,
  `gmt_modified` DATETIME,
  PRIMARY KEY (`xid`),
  KEY `idx_status_gmt_modified` (`status`, `gmt_modified`),
  KEY `idx_transaction_id` (`transaction_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 分支事务表
CREATE TABLE `branch_table` (
  `branch_id` BIGINT NOT NULL,
  `xid` VARCHAR(128) NOT NULL,
  `transaction_id` BIGINT,
  `resource_group_id` VARCHAR(32),
  `resource_id` VARCHAR(256),
  `branch_type` VARCHAR(8),
  `status` TINYINT,
  `client_id` VARCHAR(64),
  `application_data` VARCHAR(2000),
  `gmt_create` DATETIME(6),
  `gmt_modified` DATETIME(6),
  PRIMARY KEY (`branch_id`),
  KEY `idx_xid` (`xid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- 全局锁表
CREATE TABLE `lock_table` (
  `row_key` VARCHAR(128) NOT NULL,
  `xid` VARCHAR(128),
  `transaction_id` BIGINT,
  `branch_id` BIGINT NOT NULL,
  `resource_id` VARCHAR(256),
  `table_name` VARCHAR(32),
  `pk` VARCHAR(36),
  `status` TINYINT NOT NULL DEFAULT 0,
  `gmt_create` DATETIME,
  `gmt_modified` DATETIME,
  PRIMARY KEY (`row_key`),
  KEY `idx_status` (`status`),
  KEY `idx_branch_id` (`branch_id`),
  KEY `idx_xid` (`xid`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```

**3. 启动多个Seata Server实例**

```bash
# 实例1
sh seata-server.sh -p 8091 -h 192.168.1.100

# 实例2
sh seata-server.sh -p 8092 -h 192.168.1.101

# 实例3
sh seata-server.sh -p 8093 -h 192.168.1.102
```

#### 4.3.2 客户端负载均衡

```yaml
seata:
  registry:
    type: nacos
    nacos:
      # Nacos会自动实现负载均衡
      server-addr: 127.0.0.1:8848
      namespace: ""
      group: SEATA_GROUP
      cluster: default
  service:
    vgroup-mapping:
      # 事务分组映射到集群名
      default_tx_group: default
    grouplist:
      # 直连模式（不推荐生产使用）
      default: 192.168.1.100:8091,192.168.1.101:8092,192.168.1.102:8093
```

### 4.4 监控与告警

#### 4.4.1 Seata Metrics配置

```yaml
seata:
  metrics:
    enabled: true
    registry-type: prometheus
    exporter-list: prometheus
    exporter-prometheus-port: 9898
```

#### 4.4.2 Prometheus采集配置

```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'seata'
    static_configs:
      - targets: ['192.168.1.100:9898', '192.168.1.101:9898', '192.168.1.102:9898']
```

#### 4.4.3 关键指标监控

**核心监控指标**：
- `seata_transaction_total`: 事务总数
- `seata_transaction_active`: 活跃事务数
- `seata_transaction_committed`: 已提交事务数
- `seata_transaction_rollbacked`: 已回滚事务数
- `seata_transaction_timeout`: 超时事务数
- `seata_branch_transaction_total`: 分支事务总数
- `seata_global_lock_wait_time`: 全局锁等待时间

**Grafana Dashboard示例**：

```json
{
  "dashboard": {
    "title": "Seata监控",
    "panels": [
      {
        "title": "事务成功率",
        "targets": [
          {
            "expr": "rate(seata_transaction_committed[5m]) / rate(seata_transaction_total[5m]) * 100"
          }
        ]
      },
      {
        "title": "事务回滚率",
        "targets": [
          {
            "expr": "rate(seata_transaction_rollbacked[5m]) / rate(seata_transaction_total[5m]) * 100"
          }
        ]
      },
      {
        "title": "全局锁等待时间",
        "targets": [
          {
            "expr": "seata_global_lock_wait_time"
          }
        ]
      }
    ]
  }
}
```

### 4.5 故障排查

#### 4.5.1 常见问题诊断

**1. 全局事务未生效**

```java
/**
 * 问题诊断
 * @author erik.zhou
 */
@Service
public class DiagnosisService {
    
    /**
     * 检查点1：@GlobalTransactional注解是否生效
     */
    @GlobalTransactional(name = "test-transaction")
    public void checkAnnotation() {
        // 打印XID，如果为null说明注解未生效
        String xid = RootContext.getXID();
        System.out.println("当前XID: " + xid);
    }
    
    /**
     * 检查点2：数据源是否被代理
     */
    @Autowired
    private DataSource dataSource;
    
    public void checkDataSource() {
        // 检查数据源类型
        System.out.println("数据源类型: " + dataSource.getClass().getName());
        // 应该是 DataSourceProxy 或其子类
    }
}
```

**2. 分支事务未注册**

检查清单：
- [ ] 是否添加了seata-spring-boot-starter依赖
- [ ] 是否配置了正确的事务分组
- [ ] 数据源是否被Seata代理
- [ ] 是否创建了undo_log表

**3. 事务回滚失败**

```sql
-- 查询UNDO_LOG表，检查回滚日志
SELECT * FROM undo_log WHERE xid = 'your-xid';

-- 查询分支事务状态
SELECT * FROM branch_table WHERE xid = 'your-xid';
```


#### 4.5.2 日志分析

**开启Seata调试日志**：

```yaml
logging:
  level:
    io.seata: debug
```

**关键日志分析**：

```
# 全局事务开始
[DEBUG] Begin new global transaction [192.168.1.100:8091:2251799813685248]

# 分支事务注册
[DEBUG] Register branch successfully, xid = 192.168.1.100:8091:2251799813685248, branchId = 2251799813685249

# 全局事务提交
[DEBUG] Global commit successfully, xid = 192.168.1.100:8091:2251799813685248

# 全局事务回滚
[DEBUG] Global rollback successfully, xid = 192.168.1.100:8091:2251799813685248
```

#### 4.5.3 性能分析

**使用Seata控制台**：

```bash
# 启动Seata控制台
java -jar seata-server-console.jar

# 访问控制台
http://localhost:7091
```

**控制台功能**：
- 查看全局事务列表
- 查看分支事务详情
- 查看全局锁信息
- 手动回滚事务
- 查看Seata Server状态

## ❓ 常见问题

### Q1: AT模式和TCC模式如何选择？

**A**: 
- **AT模式**：适用于基于关系型数据库的场景，对业务代码无侵入，性能较好，推荐优先使用
- **TCC模式**：适用于需要自定义补偿逻辑、跨数据库、跨应用的场景，需要业务自己实现Try、Confirm、Cancel三个方法

**选择建议**：
1. 如果业务基于MySQL/Oracle等关系型数据库，优先选择AT模式
2. 如果需要精确控制资源（如金融场景），选择TCC模式
3. 如果涉及非数据库资源（如Redis、消息队列），选择TCC模式

### Q2: Seata性能如何？会不会影响系统性能？

**A**: 
Seata的性能影响主要体现在以下方面：

**AT模式性能开销**：
- 一阶段：需要生成前后镜像，增加约20%-30%的响应时间
- 二阶段：异步提交，对业务几乎无影响
- 全局锁：可能导致并发冲突，需要重试

**优化建议**：
1. 使用批量操作减少分支事务数量
2. 合理设置全局锁重试参数
3. 关闭不必要的功能（如report-success-enable）
4. 使用Redis缓存减少数据库查询

**性能数据**（参考）：
- AT模式：TPS约为本地事务的70%-80%
- TCC模式：TPS约为本地事务的60%-70%
- XA模式：TPS约为本地事务的40%-50%

### Q3: Seata如何保证数据一致性？

**A**: 
Seata通过不同的机制保证数据一致性：

**AT模式**：
- 一阶段：通过UNDO_LOG记录前后镜像
- 二阶段：通过前后镜像对比检测脏写
- 全局锁：防止并发修改

**TCC模式**：
- Try阶段：预留资源
- Confirm阶段：确认提交
- Cancel阶段：释放资源
- 通过业务逻辑保证一致性

**Saga模式**：
- 正向服务：执行业务操作
- 补偿服务：回滚业务操作
- 最终一致性

**XA模式**：
- 基于数据库XA协议
- 强一致性

### Q4: 分布式事务超时如何处理？

**A**: 
Seata提供了多层超时控制：

**1. 全局事务超时**

```java
@GlobalTransactional(
    name = "create-order",
    timeoutMills = 60000, // 全局事务超时时间：60秒
    rollbackFor = Exception.class
)
public void createOrder(OrderDTO orderDTO) {
    // 业务逻辑
}
```

**2. 分支事务超时**

```yaml
seata:
  client:
    tm:
      # 默认全局事务超时时间（毫秒）
      default-global-transaction-timeout: 60000
```

**3. 超时处理策略**

```java
/**
 * 超时处理
 * @author erik.zhou
 */
@Service
public class TimeoutHandlingService {
    
    @GlobalTransactional(timeoutMills = 30000)
    public void handleTimeout() {
        try {
            // 业务逻辑
            longRunningOperation();
        } catch (GlobalTransactionException e) {
            if (e.getCode() == TransactionExceptionCode.TransactionTimeout) {
                // 超时处理逻辑
                logger.error("全局事务超时", e);
                // 可以选择重试或降级
            }
            throw e;
        }
    }
}
```

### Q5: Seata如何处理网络分区和脑裂？

**A**: 
Seata通过以下机制处理网络异常：

**1. 心跳检测**

```yaml
seata:
  transport:
    # 心跳检测开关
    heartbeat: true
    # 心跳周期（毫秒）
    heartbeat-period: 5000
```

**2. 重连机制**

```yaml
seata:
  transport:
    # 重连间隔（毫秒）
    reconnect-interval: 2000
    # 最大重连次数
    max-reconnect-count: 3
```

**3. 事务恢复**

Seata Server会定期扫描超时的全局事务，并尝试恢复：

```yaml
seata:
  server:
    recovery:
      # 提交中事务恢复周期（毫秒）
      committing-retry-period: 1000
      # 回滚中事务恢复周期（毫秒）
      rollbacking-retry-period: 1000
      # 超时事务恢复周期（毫秒）
      timeout-retry-period: 1000
```

### Q6: TCC模式如何实现幂等性？

**A**: 
TCC模式需要业务自己实现幂等性，常见方案：

**1. 使用唯一业务键**

```java
/**
 * 幂等性实现
 * @author erik.zhou
 */
@Override
public boolean prepare(Long userId, BigDecimal money, String businessKey) {
    // 使用businessKey作为唯一键，防止重复执行
    AccountFreeze freeze = freezeMapper.selectByBusinessKey(businessKey);
    if (freeze != null) {
        // 已经执行过，直接返回成功
        return true;
    }
    
    // 执行Try逻辑
    // ...
    
    return true;
}
```

**2. 使用状态机**

```java
/**
 * 状态机实现幂等
 * @author erik.zhou
 */
@Override
public boolean commit(BusinessActionContext context) {
    String businessKey = context.getActionContext("businessKey").toString();
    AccountFreeze freeze = freezeMapper.selectByBusinessKey(businessKey);
    
    if (freeze == null) {
        // 空提交
        return true;
    }
    
    // 检查状态，防止重复执行
    if (freeze.getState() == AccountFreeze.State.CONFIRM) {
        return true;
    }
    
    // 执行Confirm逻辑
    // ...
    
    // 更新状态
    freeze.setState(AccountFreeze.State.CONFIRM);
    freezeMapper.updateById(freeze);
    
    return true;
}
```

**3. 使用分布式锁**

```java
/**
 * 分布式锁实现幂等
 * @author erik.zhou
 */
@Override
public boolean prepare(Long userId, BigDecimal money, String businessKey) {
    String lockKey = "tcc:lock:" + businessKey;
    
    // 获取分布式锁
    boolean locked = redisLock.tryLock(lockKey, 30, TimeUnit.SECONDS);
    if (!locked) {
        throw new RuntimeException("获取锁失败");
    }
    
    try {
        // 执行Try逻辑
        // ...
    } finally {
        redisLock.unlock(lockKey);
    }
    
    return true;
}
```

### Q7: Seata如何与Spring Cloud集成？

**A**: 
Seata与Spring Cloud的集成非常简单：

**1. 添加依赖**

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-seata</artifactId>
</dependency>
```

**2. 配置文件**

```yaml
spring:
  cloud:
    alibaba:
      seata:
        tx-service-group: default_tx_group

seata:
  registry:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
  config:
    type: nacos
    nacos:
      server-addr: 127.0.0.1:8848
```

**3. 使用Feign调用**

```java
/**
 * Feign客户端
 * @author erik.zhou
 */
@FeignClient(name = "storage-service")
public interface StorageClient {
    
    @PostMapping("/storage/deduct")
    Result deduct(@RequestParam("productId") Long productId,
                  @RequestParam("count") Integer count);
}

/**
 * 订单服务
 * @author erik.zhou
 */
@Service
public class OrderService {
    
    @Autowired
    private StorageClient storageClient;
    
    @GlobalTransactional
    public void createOrder(OrderDTO orderDTO) {
        // Feign调用会自动传播XID
        storageClient.deduct(orderDTO.getProductId(), orderDTO.getCount());
    }
}
```


### Q8: 如何排查Seata事务不生效的问题？

**A**: 
按照以下步骤排查：

**1. 检查依赖**

```xml
<!-- 确保添加了Seata依赖 -->
<dependency>
    <groupId>io.seata</groupId>
    <artifactId>seata-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
```

**2. 检查配置**

```yaml
# 确保配置正确
seata:
  enabled: true
  application-id: order-service
  tx-service-group: default_tx_group
```

**3. 检查数据源代理**

```java
/**
 * 数据源检查
 * @author erik.zhou
 */
@Component
public class DataSourceChecker implements ApplicationRunner {
    
    @Autowired
    private DataSource dataSource;
    
    @Override
    public void run(ApplicationArguments args) {
        System.out.println("数据源类型: " + dataSource.getClass().getName());
        // 应该输出: com.zaxxer.hikari.HikariDataSource$HikariProxyConnection
        // 或: io.seata.rm.datasource.DataSourceProxy
    }
}
```

**4. 检查XID传播**

```java
/**
 * XID检查
 * @author erik.zhou
 */
@Service
public class XidChecker {
    
    @GlobalTransactional
    public void checkXid() {
        String xid = RootContext.getXID();
        System.out.println("当前XID: " + xid);
        // 如果为null，说明全局事务未开启
    }
}
```

**5. 检查UNDO_LOG表**

```sql
-- 确保每个业务数据库都创建了undo_log表
SHOW TABLES LIKE 'undo_log';
```

**6. 查看日志**

```yaml
# 开启Seata调试日志
logging:
  level:
    io.seata: debug
```

## 🔗 相关资源

### 官方资源
- **官方网站**: https://seata.apache.org/
- **GitHub仓库**: https://github.com/apache/incubator-seata
- **官方文档**: https://seata.apache.org/docs/overview/what-is-seata/
- **中文文档**: https://seata.apache.org/zh-cn/docs/overview/what-is-seata/

### 学习资源
- **Seata快速开始**: https://seata.apache.org/docs/user/quickstart/
- **Seata示例项目**: https://github.com/apache/incubator-seata-samples
- **Seata博客**: https://seata.apache.org/blog/

### 社区资源
- **Seata钉钉群**: 23171167（1群）、23171824（2群）
- **GitHub Issues**: https://github.com/apache/incubator-seata/issues
- **Stack Overflow**: 标签 `seata`

### 推荐文章
1. 《Seata AT模式原理深度解析》
2. 《TCC分布式事务最佳实践》
3. 《Seata性能优化实战》
4. 《分布式事务Seata源码分析》

### 视频教程
1. 尚硅谷 - Seata分布式事务实战
2. 黑马程序员 - Seata从入门到精通
3. 慕课网 - 分布式事务解决方案

## 📝 学习检查清单

### 基础知识
- [ ] 理解分布式事务的ACID特性
- [ ] 掌握CAP理论和BASE理论
- [ ] 理解2PC和3PC协议
- [ ] 了解Seata的架构设计
- [ ] 掌握TC、TM、RM三个组件的作用

### AT模式
- [ ] 理解AT模式的工作原理
- [ ] 掌握UNDO_LOG的作用
- [ ] 能够配置AT模式
- [ ] 理解全局锁机制
- [ ] 掌握AT模式的性能优化

### TCC模式
- [ ] 理解TCC的三个阶段
- [ ] 掌握TCC的幂等性实现
- [ ] 理解空回滚和防悬挂
- [ ] 能够实现TCC接口
- [ ] 掌握TCC的最佳实践

### Saga模式
- [ ] 理解Saga的补偿机制
- [ ] 掌握状态机的定义
- [ ] 能够实现Saga服务
- [ ] 理解Saga的适用场景

### XA模式
- [ ] 理解XA协议
- [ ] 掌握XA模式的配置
- [ ] 理解XA模式的优缺点

### 实战能力
- [ ] 能够在Spring Boot项目中集成Seata
- [ ] 能够在Spring Cloud项目中使用Seata
- [ ] 能够部署Seata Server集群
- [ ] 能够配置Seata监控
- [ ] 能够排查Seata常见问题

### 高级主题
- [ ] 掌握Seata的性能优化技巧
- [ ] 理解Seata的高可用方案
- [ ] 掌握Seata的故障排查方法
- [ ] 了解Seata的源码架构
- [ ] 能够根据业务场景选择合适的事务模式

## 📊 技术对比

### Seata vs 其他分布式事务方案

| 特性 | Seata | TCC-Transaction | Hmily | LCN |
|------|-------|----------------|-------|-----|
| 事务模式 | AT/TCC/Saga/XA | TCC | TCC/TAC | LCN/TCC/TXC |
| 性能 | 高 | 中 | 中 | 低 |
| 侵入性 | 低（AT模式） | 高 | 高 | 低 |
| 社区活跃度 | 高 | 中 | 中 | 低 |
| 生产可用性 | 高 | 中 | 中 | 低 |
| 学习成本 | 中 | 高 | 高 | 中 |
| 推荐指数 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

### 选择建议

**推荐使用Seata的场景**：
1. 微服务架构，需要跨服务事务
2. 基于Spring Cloud/Dubbo的项目
3. 需要高性能的分布式事务
4. 希望对业务代码侵入性低

**不推荐使用Seata的场景**：
1. 单体应用（使用本地事务即可）
2. 对强一致性要求极高的场景（考虑XA或其他方案）
3. 团队对分布式事务理解不足

## 🎓 进阶学习路径

### 第一阶段：基础入门（1-2周）
1. 学习分布式事务理论基础
2. 理解Seata的架构设计
3. 掌握AT模式的使用
4. 完成简单的示例项目

### 第二阶段：深入实践（2-3周）
1. 掌握TCC模式的使用
2. 学习Saga模式
3. 了解XA模式
4. 在实际项目中应用Seata

### 第三阶段：性能优化（1-2周）
1. 学习Seata的性能优化技巧
2. 掌握全局锁优化方法
3. 学习批量操作优化
4. 进行性能测试和调优

### 第四阶段：高可用部署（1周）
1. 学习Seata Server集群部署
2. 掌握高可用配置
3. 学习监控和告警
4. 掌握故障排查方法

### 第五阶段：源码研究（选修）
1. 阅读Seata核心源码
2. 理解AT模式的实现原理
3. 理解TCC模式的实现原理
4. 参与Seata社区贡献

---

**文档版本**: v2.0  
**最后更新**: 2024-01-04  
**文档来源**: Seata官方文档 + Context7查询  
**作者**: @author erik.zhou

