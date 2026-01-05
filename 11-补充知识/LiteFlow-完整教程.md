# LiteFlow 完整教程

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
- **版本**: LiteFlow 2.12.x
- **官方网站**: https://liteflow.cc/
- **GitHub**: https://github.com/dromara/liteFlow
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Java基础（JDK 8+）
  - Spring Boot基础
  - 设计模式（策略模式、责任链模式）
  - 基本的业务流程概念
- **文档来源**: LiteFlow官方文档 + 社区实践
- **更新时间**: 2025-01-05
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解规则引擎和流程编排的概念
- [ ] 掌握LiteFlow的核心组件定义
- [ ] 理解EL表达式的编写规则
- [ ] 掌握串行、并行、条件编排
- [ ] 理解上下文数据传递机制
- [ ] 掌握脚本组件的使用
- [ ] 能够实现复杂业务流程编排
- [ ] 理解规则热部署和动态刷新

## 📖 基础概念

### 1.1 什么是LiteFlow

LiteFlow是一个轻量级、高性能的编排式规则引擎框架，专注于解决复杂业务逻辑的解耦和编排问题。

**核心理念**：
- 将业务逻辑拆分成独立的组件
- 通过EL表达式灵活编排组件
- 实现业务逻辑的可视化和可配置化


**LiteFlow的优势**：
- 轻量级：核心jar包仅几百KB
- 高性能：支持并行编排，充分利用多核CPU
- 易扩展：组件化设计，易于维护和扩展
- 灵活编排：支持串行、并行、条件、循环等多种编排方式
- 热部署：支持规则动态刷新，无需重启应用
- 多语言脚本：支持Groovy、JavaScript、Python等脚本语言

### 1.2 核心概念

#### 1.2.1 组件（Component）

组件是LiteFlow的最小执行单元，每个组件负责一个独立的业务逻辑片段。

**组件类型**：
- **普通组件**：执行具体业务逻辑
- **条件组件**：用于流程分支判断
- **脚本组件**：使用脚本语言编写的组件
- **子流程组件**：封装一个完整的子流程

#### 1.2.2 EL表达式（Expression Language）

EL表达式是LiteFlow的流程编排语言，用于定义组件的执行顺序和逻辑关系。

**基本语法**：
- `THEN(a, b, c)`：串行执行a、b、c
- `WHEN(a, b, c)`：并行执行a、b、c
- `IF(x, a, b)`：如果x为true执行a，否则执行b
- `SWITCH(x).to(a, b, c)`：根据x的值选择执行
- `FOR(n).DO(a)`：循环执行a，n次

#### 1.2.3 上下文（Context）

上下文用于在组件之间传递数据，是流程执行过程中的数据容器。

**上下文特点**：
- 线程安全
- 支持任意类型数据
- 支持数据隔离
- 支持数据共享

### 1.3 应用场景

- **订单处理流程**：订单校验、库存检查、支付处理、发货通知
- **审批流程**：多级审批、条件审批、并行审批
- **数据处理管道**：数据清洗、转换、校验、存储
- **营销活动规则**：优惠券计算、积分规则、促销策略
- **风控规则引擎**：反欺诈检测、信用评估、风险预警
- **业务编排**：微服务调用编排、异步任务编排

### 1.4 LiteFlow vs 传统代码

**传统代码**：
```java
public void processOrder(Order order) {
    // 大量if-else嵌套
    if (order.getType() == 1) {
        validateOrder(order);
        if (checkStock(order)) {
            if (processPayment(order)) {
                sendNotification(order);
            }
        }
    } else if (order.getType() == 2) {
        // 另一套逻辑
    }
}
```

**LiteFlow方式**：
```java
// 组件化拆分
@LiteflowComponent("validateOrder")
public class ValidateOrderCmp extends NodeComponent {
    @Override
    public void process() {
        // 订单校验逻辑
    }
}

// EL表达式编排
THEN(validateOrder, checkStock, processPayment, sendNotification)
```

## 🔥 核心特性

### 2.1 环境搭建

#### 2.1.1 添加依赖

**Maven依赖**：

```xml
<dependencies>
    <!-- LiteFlow核心 -->
    <dependency>
        <groupId>com.yomahub</groupId>
        <artifactId>liteflow-spring-boot-starter</artifactId>
        <version>2.12.2</version>
    </dependency>
</dependencies>
```

**Gradle依赖**：

```gradle
dependencies {
    implementation 'com.yomahub:liteflow-spring-boot-starter:2.12.2'
}
```

#### 2.1.2 配置文件

**application.yml**：

```yaml
liteflow:
  # 规则文件路径
  rule-source: classpath:flow.el.xml
  # 是否开启监控
  enable-log: true
  # 是否打印banner
  print-banner: true
  # 线程池配置
  when-max-workers: 16
  when-queue-limit: 512
  # 重试配置
  retry-count: 0
```

#### 2.1.3 规则文件

**flow.el.xml**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow>
    <!-- 定义流程链 -->
    <chain name="orderChain">
        THEN(
            validateOrder,
            checkStock,
            processPayment,
            sendNotification
        )
    </chain>
</flow>
```

### 2.2 组件定义 🔥

#### 2.2.1 普通组件

普通组件继承`NodeComponent`类，实现`process()`方法。

```java
import com.yomahub.liteflow.core.NodeComponent;
import com.yomahub.liteflow.slot.DefaultContext;
import org.springframework.stereotype.Component;

/**
 * 订单校验组件
 * 
 * @author erik.zhou
 */
@Component("validateOrder")
public class ValidateOrderComponent extends NodeComponent {

    @Override
    public void process() {
        // 获取上下文
        DefaultContext context = this.getContextBean(DefaultContext.class);
        
        // 获取请求参数
        Order order = context.getData("order");
        
        // 执行校验逻辑
        if (order == null) {
            throw new IllegalArgumentException("订单不能为空");
        }
        
        if (order.getAmount() <= 0) {
            throw new IllegalArgumentException("订单金额必须大于0");
        }
        
        // 将校验结果放入上下文
        context.setData("validateResult", true);
        
        System.out.println("订单校验通过: " + order.getOrderNo());
    }
}
```

#### 2.2.2 条件组件

条件组件继承`NodeSwitchComponent`或`NodeIfComponent`类。

**IF条件组件**：

```java
import com.yomahub.liteflow.core.NodeIfComponent;
import org.springframework.stereotype.Component;

/**
 * VIP用户判断组件
 * 
 * @author erik.zhou
 */
@Component("isVipUser")
public class VipUserConditionComponent extends NodeIfComponent {

    @Override
    public boolean processIf() throws Exception {
        DefaultContext context = this.getContextBean(DefaultContext.class);
        Order order = context.getData("order");
        
        // 判断是否为VIP用户
        boolean isVip = order.getUserLevel() >= 5;
        
        System.out.println("用户VIP状态: " + isVip);
        return isVip;
    }
}
```

**SWITCH条件组件**：

```java
import com.yomahub.liteflow.core.NodeSwitchComponent;
import org.springframework.stereotype.Component;

/**
 * 订单类型路由组件
 * 
 * @author erik.zhou
 */
@Component("orderTypeRouter")
public class OrderTypeRouterComponent extends NodeSwitchComponent {

    @Override
    public String processSwitch() throws Exception {
        DefaultContext context = this.getContextBean(DefaultContext.class);
        Order order = context.getData("order");
        
        // 根据订单类型返回不同的节点ID
        switch (order.getType()) {
            case 1:
                return "normalOrder";
            case 2:
                return "groupOrder";
            case 3:
                return "seckillOrder";
            default:
                return "defaultOrder";
        }
    }
}
```


#### 2.2.3 脚本组件

LiteFlow支持使用脚本语言编写组件，无需编译即可动态修改。

**Groovy脚本组件**：

```xml
<flow>
    <nodes>
        <!-- Groovy脚本组件 -->
        <node id="calculateDiscount" type="script" language="groovy">
            <![CDATA[
                def context = defaultContext
                def order = context.getData("order")
                def discount = 1.0
                
                // 根据金额计算折扣
                if (order.amount > 1000) {
                    discount = 0.9
                } else if (order.amount > 500) {
                    discount = 0.95
                }
                
                context.setData("discount", discount)
                println "计算折扣: ${discount}"
            ]]>
        </node>
    </nodes>
</flow>
```

**添加Groovy依赖**：

```xml
<dependency>
    <groupId>com.yomahub</groupId>
    <artifactId>liteflow-script-groovy</artifactId>
    <version>2.12.2</version>
</dependency>
```

### 2.3 EL表达式详解 🔥 (⚠️ 难点)

#### 2.3.1 串行编排（THEN）

串行执行多个组件，按顺序依次执行。

```xml
<chain name="serialChain">
    THEN(a, b, c, d)
</chain>
```

**执行顺序**：a → b → c → d

#### 2.3.2 并行编排（WHEN）

并行执行多个组件，充分利用多核CPU。

```xml
<chain name="parallelChain">
    WHEN(a, b, c)
</chain>
```

**执行特点**：
- a、b、c同时执行
- 等待所有组件执行完成
- 任一组件异常，整个流程失败

#### 2.3.3 条件编排（IF）

根据条件选择执行分支。

```xml
<chain name="conditionChain">
    IF(condition, THEN(a, b), THEN(c, d))
</chain>
```

**执行逻辑**：
- 如果condition返回true，执行a→b
- 如果condition返回false，执行c→d

#### 2.3.4 选择编排（SWITCH）

根据条件值选择执行不同的分支。

```xml
<chain name="switchChain">
    SWITCH(router).to(
        a,
        b,
        c
    ).DEFAULT(d)
</chain>
```

**执行逻辑**：
- router返回"a"，执行组件a
- router返回"b"，执行组件b
- router返回"c"，执行组件c
- 其他情况执行组件d

#### 2.3.5 循环编排（FOR）

循环执行组件。

```xml
<chain name="loopChain">
    FOR(5).DO(THEN(a, b))
</chain>
```

**执行逻辑**：
- 循环执行5次
- 每次执行a→b

**动态循环次数**：

```xml
<chain name="dynamicLoopChain">
    FOR(loopCount).DO(THEN(a, b))
</chain>
```

```java
@Component("loopCount")
public class LoopCountComponent extends NodeForComponent {
    @Override
    public int processFor() throws Exception {
        DefaultContext context = this.getContextBean(DefaultContext.class);
        // 动态返回循环次数
        return context.getData("count");
    }
}
```

#### 2.3.6 迭代编排（ITERATOR）

遍历集合执行组件。

```xml
<chain name="iteratorChain">
    ITERATOR(items).DO(THEN(a, b))
</chain>
```

```java
@Component("items")
public class ItemsIteratorComponent extends NodeIteratorComponent {
    @Override
    public Iterator<?> processIterator() throws Exception {
        DefaultContext context = this.getContextBean(DefaultContext.class);
        List<String> items = context.getData("itemList");
        return items.iterator();
    }
}
```

#### 2.3.7 复杂嵌套编排

```xml
<chain name="complexChain">
    THEN(
        a,
        WHEN(b, c, d),
        IF(condition,
            THEN(e, f),
            WHEN(g, h)
        ),
        SWITCH(router).to(i, j, k),
        FOR(3).DO(THEN(l, m))
    )
</chain>
```

**执行流程**：
1. 执行a
2. 并行执行b、c、d
3. 根据condition条件选择执行e→f或并行执行g、h
4. 根据router选择执行i、j或k
5. 循环3次执行l→m

### 2.4 上下文数据传递

#### 2.4.1 默认上下文

```java
import com.yomahub.liteflow.slot.DefaultContext;

@Component("componentA")
public class ComponentA extends NodeComponent {
    
    @Override
    public void process() {
        // 获取上下文
        DefaultContext context = this.getContextBean(DefaultContext.class);
        
        // 存储数据
        context.setData("key1", "value1");
        context.setData("order", new Order());
        
        // 获取数据
        String value = context.getData("key1");
        Order order = context.getData("order");
    }
}
```

#### 2.4.2 自定义上下文

```java
import com.yomahub.liteflow.slot.DefaultContext;

/**
 * 订单上下文
 * 
 * @author erik.zhou
 */
public class OrderContext extends DefaultContext {
    
    private Order order;
    private User user;
    private List<Product> products;
    
    // Getter和Setter方法
    public Order getOrder() {
        return order;
    }
    
    public void setOrder(Order order) {
        this.order = order;
    }
    
    public User getUser() {
        return user;
    }
    
    public void setUser(User user) {
        this.user = user;
    }
    
    public List<Product> getProducts() {
        return products;
    }
    
    public void setProducts(List<Product> products) {
        this.products = products;
    }
}
```

**使用自定义上下文**：

```java
@Component("componentB")
public class ComponentB extends NodeComponent {
    
    @Override
    public void process() {
        // 获取自定义上下文
        OrderContext context = this.getContextBean(OrderContext.class);
        
        // 直接访问属性
        Order order = context.getOrder();
        User user = context.getUser();
    }
}
```

### 2.5 流程执行

#### 2.5.1 执行流程

```java
import com.yomahub.liteflow.core.FlowExecutor;
import com.yomahub.liteflow.flow.LiteflowResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

/**
 * 订单服务
 * 
 * @author erik.zhou
 */
@Service
public class OrderService {
    
    @Autowired
    private FlowExecutor flowExecutor;
    
    /**
     * 处理订单
     */
    public void processOrder(Order order) {
        // 创建上下文
        OrderContext context = new OrderContext();
        context.setOrder(order);
        
        // 执行流程
        LiteflowResponse response = flowExecutor.execute2Resp(
            "orderChain",  // 流程链名称
            null,          // 请求参数（可选）
            context        // 上下文
        );
        
        // 检查执行结果
        if (response.isSuccess()) {
            System.out.println("订单处理成功");
        } else {
            System.out.println("订单处理失败: " + response.getMessage());
            Exception exception = response.getCause();
            exception.printStackTrace();
        }
    }
}
```

#### 2.5.2 获取执行结果

```java
// 执行流程并获取响应
LiteflowResponse response = flowExecutor.execute2Resp("orderChain", null, context);

// 判断是否成功
boolean success = response.isSuccess();

// 获取异常信息
String message = response.getMessage();
Exception cause = response.getCause();

// 获取执行时间
long executeTime = response.getExecuteStepStr();

// 获取上下文数据
OrderContext resultContext = response.getContextBean(OrderContext.class);
```

### 2.6 组件高级特性

#### 2.6.1 组件重试

```java
import com.yomahub.liteflow.annotation.LiteflowRetry;

/**
 * 支付组件（带重试）
 * 
 * @author erik.zhou
 */
@Component("payment")
@LiteflowRetry(retry = 3, forExceptions = {PaymentException.class})
public class PaymentComponent extends NodeComponent {
    
    @Override
    public void process() {
        // 支付逻辑
        // 如果抛出PaymentException，会自动重试3次
    }
}
```

#### 2.6.2 组件超时控制

```java
import com.yomahub.liteflow.annotation.LiteflowMethod;
import com.yomahub.liteflow.enums.LiteFlowMethodEnum;

@Component("timeoutComponent")
public class TimeoutComponent extends NodeComponent {
    
    @Override
    @LiteflowMethod(LiteFlowMethodEnum.PROCESS)
    public void process() {
        // 业务逻辑
    }
    
    @Override
    @LiteflowMethod(LiteFlowMethodEnum.PROCESS_TIMEOUT)
    public void processTimeout() {
        // 超时处理逻辑
        System.out.println("组件执行超时");
    }
}
```

**配置超时时间**：

```xml
<chain name="timeoutChain">
    THEN(a, b.timeout(5000), c)
</chain>
```

#### 2.6.3 组件回滚

```java
@Component("rollbackComponent")
public class RollbackComponent extends NodeComponent {
    
    @Override
    public void process() {
        // 正常执行逻辑
        System.out.println("执行业务逻辑");
    }
    
    @Override
    public void rollback() throws Exception {
        // 回滚逻辑
        System.out.println("执行回滚操作");
    }
}
```


## 💻 实战应用

### 3.1 电商订单处理流程

#### 3.1.1 业务场景

实现一个完整的电商订单处理流程，包括：
1. 订单校验
2. 库存检查
3. 优惠计算
4. 支付处理
5. 库存扣减
6. 发送通知

#### 3.1.2 组件实现

**订单校验组件**：

```java
import com.yomahub.liteflow.core.NodeComponent;
import org.springframework.stereotype.Component;

/**
 * 订单校验组件
 * 
 * @author erik.zhou
 */
@Component("validateOrder")
public class ValidateOrderComponent extends NodeComponent {
    
    @Override
    public void process() {
        OrderContext context = this.getContextBean(OrderContext.class);
        Order order = context.getOrder();
        
        // 校验订单基本信息
        if (order == null || order.getOrderNo() == null) {
            throw new IllegalArgumentException("订单信息不完整");
        }
        
        // 校验商品信息
        if (order.getProducts() == null || order.getProducts().isEmpty()) {
            throw new IllegalArgumentException("订单商品不能为空");
        }
        
        // 校验金额
        if (order.getTotalAmount() <= 0) {
            throw new IllegalArgumentException("订单金额必须大于0");
        }
        
        System.out.println("订单校验通过: " + order.getOrderNo());
    }
}
```

**库存检查组件**：

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * 库存检查组件
 * 
 * @author erik.zhou
 */
@Component("checkStock")
public class CheckStockComponent extends NodeComponent {
    
    @Autowired
    private StockService stockService;
    
    @Override
    public void process() {
        OrderContext context = this.getContextBean(OrderContext.class);
        Order order = context.getOrder();
        
        // 检查每个商品的库存
        for (OrderItem item : order.getItems()) {
            int stock = stockService.getStock(item.getProductId());
            if (stock < item.getQuantity()) {
                throw new RuntimeException(
                    "商品库存不足: " + item.getProductName() + 
                    ", 需要: " + item.getQuantity() + 
                    ", 库存: " + stock
                );
            }
        }
        
        System.out.println("库存检查通过");
    }
}
```

**优惠计算组件**：

```java
/**
 * 优惠计算组件
 * 
 * @author erik.zhou
 */
@Component("calculateDiscount")
public class CalculateDiscountComponent extends NodeComponent {
    
    @Autowired
    private CouponService couponService;
    
    @Override
    public void process() {
        OrderContext context = this.getContextBean(OrderContext.class);
        Order order = context.getOrder();
        
        double originalAmount = order.getTotalAmount();
        double discount = 0;
        
        // 计算优惠券折扣
        if (order.getCouponCode() != null) {
            discount += couponService.calculateDiscount(order.getCouponCode(), originalAmount);
        }
        
        // 计算会员折扣
        if (order.getUserLevel() >= 5) {
            discount += originalAmount * 0.05;  // VIP用户额外5%折扣
        }
        
        // 计算满减优惠
        if (originalAmount >= 1000) {
            discount += 100;
        } else if (originalAmount >= 500) {
            discount += 50;
        }
        
        double finalAmount = originalAmount - discount;
        context.setData("discount", discount);
        context.setData("finalAmount", finalAmount);
        
        System.out.println("优惠计算完成 - 原价: " + originalAmount + 
                         ", 优惠: " + discount + 
                         ", 实付: " + finalAmount);
    }
}
```

**支付处理组件**：

```java
import com.yomahub.liteflow.annotation.LiteflowRetry;

/**
 * 支付处理组件
 * 
 * @author erik.zhou
 */
@Component("processPayment")
@LiteflowRetry(retry = 3, forExceptions = {PaymentException.class})
public class ProcessPaymentComponent extends NodeComponent {
    
    @Autowired
    private PaymentService paymentService;
    
    @Override
    public void process() {
        OrderContext context = this.getContextBean(OrderContext.class);
        Order order = context.getOrder();
        Double finalAmount = context.getData("finalAmount");
        
        // 调用支付接口
        PaymentResult result = paymentService.pay(
            order.getOrderNo(),
            finalAmount,
            order.getPaymentMethod()
        );
        
        if (!result.isSuccess()) {
            throw new PaymentException("支付失败: " + result.getMessage());
        }
        
        context.setData("paymentResult", result);
        System.out.println("支付成功 - 订单号: " + order.getOrderNo() + 
                         ", 金额: " + finalAmount);
    }
    
    @Override
    public void rollback() throws Exception {
        // 支付回滚逻辑
        OrderContext context = this.getContextBean(OrderContext.class);
        PaymentResult result = context.getData("paymentResult");
        
        if (result != null && result.isSuccess()) {
            paymentService.refund(result.getTransactionId());
            System.out.println("支付已回滚");
        }
    }
}
```

**库存扣减组件**：

```java
/**
 * 库存扣减组件
 * 
 * @author erik.zhou
 */
@Component("deductStock")
public class DeductStockComponent extends NodeComponent {
    
    @Autowired
    private StockService stockService;
    
    @Override
    public void process() {
        OrderContext context = this.getContextBean(OrderContext.class);
        Order order = context.getOrder();
        
        // 扣减库存
        for (OrderItem item : order.getItems()) {
            stockService.deduct(item.getProductId(), item.getQuantity());
        }
        
        System.out.println("库存扣减成功");
    }
    
    @Override
    public void rollback() throws Exception {
        // 库存回滚
        OrderContext context = this.getContextBean(OrderContext.class);
        Order order = context.getOrder();
        
        for (OrderItem item : order.getItems()) {
            stockService.restore(item.getProductId(), item.getQuantity());
        }
        
        System.out.println("库存已回滚");
    }
}
```

**通知发送组件**：

```java
/**
 * 通知发送组件
 * 
 * @author erik.zhou
 */
@Component("sendNotification")
public class SendNotificationComponent extends NodeComponent {
    
    @Autowired
    private NotificationService notificationService;
    
    @Override
    public void process() {
        OrderContext context = this.getContextBean(OrderContext.class);
        Order order = context.getOrder();
        
        // 并行发送多种通知
        notificationService.sendSms(order.getUserPhone(), "订单支付成功");
        notificationService.sendEmail(order.getUserEmail(), "订单确认");
        notificationService.sendPush(order.getUserId(), "订单处理中");
        
        System.out.println("通知发送成功");
    }
}
```

#### 3.1.3 流程编排

**flow.el.xml**：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<flow>
    <!-- 订单处理主流程 -->
    <chain name="orderProcessChain">
        THEN(
            validateOrder,
            checkStock,
            calculateDiscount,
            processPayment,
            WHEN(
                deductStock,
                sendNotification
            )
        )
    </chain>
</flow>
```

#### 3.1.4 服务调用

```java
import com.yomahub.liteflow.core.FlowExecutor;
import com.yomahub.liteflow.flow.LiteflowResponse;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

/**
 * 订单服务
 * 
 * @author erik.zhou
 */
@Service
public class OrderService {
    
    @Autowired
    private FlowExecutor flowExecutor;
    
    /**
     * 处理订单
     */
    @Transactional(rollbackFor = Exception.class)
    public OrderResult processOrder(Order order) {
        // 创建上下文
        OrderContext context = new OrderContext();
        context.setOrder(order);
        
        // 执行流程
        LiteflowResponse response = flowExecutor.execute2Resp(
            "orderProcessChain",
            null,
            context
        );
        
        // 处理结果
        OrderResult result = new OrderResult();
        result.setSuccess(response.isSuccess());
        
        if (response.isSuccess()) {
            result.setMessage("订单处理成功");
            result.setFinalAmount(context.getData("finalAmount"));
            result.setDiscount(context.getData("discount"));
        } else {
            result.setMessage("订单处理失败: " + response.getMessage());
            // 触发回滚
            throw new RuntimeException(response.getMessage(), response.getCause());
        }
        
        return result;
    }
}
```

### 3.2 多级审批流程

#### 3.2.1 业务场景

实现一个灵活的多级审批流程：
- 金额小于1000：部门经理审批
- 金额1000-5000：部门经理 + 财务审批
- 金额大于5000：部门经理 + 财务 + 总经理审批

#### 3.2.2 组件实现

**审批路由组件**：

```java
import com.yomahub.liteflow.core.NodeSwitchComponent;

/**
 * 审批路由组件
 * 
 * @author erik.zhou
 */
@Component("approvalRouter")
public class ApprovalRouterComponent extends NodeSwitchComponent {
    
    @Override
    public String processSwitch() throws Exception {
        ApprovalContext context = this.getContextBean(ApprovalContext.class);
        double amount = context.getApprovalRequest().getAmount();
        
        if (amount < 1000) {
            return "level1";  // 一级审批
        } else if (amount < 5000) {
            return "level2";  // 二级审批
        } else {
            return "level3";  // 三级审批
        }
    }
}
```

**审批组件**：

```java
/**
 * 部门经理审批组件
 * 
 * @author erik.zhou
 */
@Component("managerApproval")
public class ManagerApprovalComponent extends NodeComponent {
    
    @Autowired
    private ApprovalService approvalService;
    
    @Override
    public void process() {
        ApprovalContext context = this.getContextBean(ApprovalContext.class);
        ApprovalRequest request = context.getApprovalRequest();
        
        // 执行审批逻辑
        boolean approved = approvalService.approve(
            request.getId(),
            "MANAGER",
            request.getManagerId()
        );
        
        if (!approved) {
            throw new ApprovalRejectedException("部门经理审批拒绝");
        }
        
        context.addApprovalRecord("部门经理审批通过");
        System.out.println("部门经理审批通过");
    }
}
```

#### 3.2.3 流程编排

```xml
<flow>
    <!-- 一级审批流程 -->
    <chain name="level1">
        THEN(managerApproval)
    </chain>
    
    <!-- 二级审批流程 -->
    <chain name="level2">
        THEN(managerApproval, financeApproval)
    </chain>
    
    <!-- 三级审批流程 -->
    <chain name="level3">
        THEN(
            managerApproval,
            WHEN(financeApproval, hrApproval),
            ceoApproval
        )
    </chain>
    
    <!-- 主审批流程 -->
    <chain name="approvalChain">
        THEN(
            validateRequest,
            SWITCH(approvalRouter).to(level1, level2, level3),
            notifyResult
        )
    </chain>
</flow>
```

### 3.3 数据处理管道

#### 3.3.1 业务场景

实现一个数据ETL管道：数据提取 → 数据清洗 → 数据转换 → 数据校验 → 数据存储

#### 3.3.2 组件实现

```java
/**
 * 数据提取组件
 * 
 * @author erik.zhou
 */
@Component("extractData")
public class ExtractDataComponent extends NodeComponent {
    
    @Override
    public void process() {
        DataContext context = this.getContextBean(DataContext.class);
        
        // 从数据源提取数据
        List<RawData> rawDataList = dataSource.extract();
        context.setRawDataList(rawDataList);
        
        System.out.println("数据提取完成，共 " + rawDataList.size() + " 条");
    }
}
```

**使用迭代器处理数据**：

```xml
<flow>
    <chain name="dataProcessChain">
        THEN(
            extractData,
            ITERATOR(dataIterator).DO(
                THEN(
                    cleanData,
                    transformData,
                    validateData
                )
            ),
            saveData
        )
    </chain>
</flow>
```

```java
/**
 * 数据迭代器组件
 * 
 * @author erik.zhou
 */
@Component("dataIterator")
public class DataIteratorComponent extends NodeIteratorComponent {
    
    @Override
    public Iterator<?> processIterator() throws Exception {
        DataContext context = this.getContextBean(DataContext.class);
        return context.getRawDataList().iterator();
    }
}
```


## ✨ 最佳实践

### 4.1 组件设计原则

#### 4.1.1 单一职责原则

每个组件只负责一个明确的业务功能。

```java
// ❌ 不推荐：组件职责过多
@Component("processOrder")
public class ProcessOrderComponent extends NodeComponent {
    @Override
    public void process() {
        // 校验订单
        validateOrder();
        // 检查库存
        checkStock();
        // 处理支付
        processPayment();
        // 发送通知
        sendNotification();
    }
}

// ✅ 推荐：拆分成多个组件
@Component("validateOrder")
public class ValidateOrderComponent extends NodeComponent {
    @Override
    public void process() {
        // 只负责订单校验
    }
}

@Component("checkStock")
public class CheckStockComponent extends NodeComponent {
    @Override
    public void process() {
        // 只负责库存检查
    }
}
```

#### 4.1.2 组件命名规范

```java
// 组件命名要清晰表达功能
@Component("validateOrder")        // ✅ 清晰
@Component("checkStock")           // ✅ 清晰
@Component("calculateDiscount")    // ✅ 清晰

@Component("process")              // ❌ 不清晰
@Component("handle")               // ❌ 不清晰
@Component("doSomething")          // ❌ 不清晰
```

#### 4.1.3 避免组件间直接依赖

```java
// ❌ 不推荐：组件间直接依赖
@Component("componentA")
public class ComponentA extends NodeComponent {
    @Autowired
    private ComponentB componentB;  // 直接依赖其他组件
    
    @Override
    public void process() {
        componentB.doSomething();
    }
}

// ✅ 推荐：通过上下文传递数据
@Component("componentA")
public class ComponentA extends NodeComponent {
    @Override
    public void process() {
        DefaultContext context = this.getContextBean(DefaultContext.class);
        context.setData("result", "data");
    }
}

@Component("componentB")
public class ComponentB extends NodeComponent {
    @Override
    public void process() {
        DefaultContext context = this.getContextBean(DefaultContext.class);
        String result = context.getData("result");
    }
}
```

### 4.2 性能优化

#### 4.2.1 合理使用并行编排

```xml
<!-- 将可以并行执行的组件放在WHEN中 -->
<chain name="optimizedChain">
    THEN(
        validateOrder,
        WHEN(
            checkStock,
            checkCoupon,
            checkUserLevel
        ),
        processPayment
    )
</chain>
```

#### 4.2.2 配置线程池

```yaml
liteflow:
  # 并行线程池配置
  when-max-workers: 32          # 最大线程数
  when-queue-limit: 1024        # 队列大小
  when-max-wait-seconds: 15     # 最大等待时间（秒）
```

#### 4.2.3 避免重复计算

```java
/**
 * 使用上下文缓存计算结果
 * 
 * @author erik.zhou
 */
@Component("calculatePrice")
public class CalculatePriceComponent extends NodeComponent {
    
    @Override
    public void process() {
        DefaultContext context = this.getContextBean(DefaultContext.class);
        
        // 检查缓存
        Double cachedPrice = context.getData("calculatedPrice");
        if (cachedPrice != null) {
            return;  // 使用缓存结果
        }
        
        // 执行计算
        Double price = doCalculate();
        
        // 缓存结果
        context.setData("calculatedPrice", price);
    }
}
```

### 4.3 异常处理

#### 4.3.1 全局异常处理

```java
import com.yomahub.liteflow.exception.LiteFlowException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;

/**
 * LiteFlow全局异常处理
 * 
 * @author erik.zhou
 */
@RestControllerAdvice
public class LiteFlowExceptionHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(LiteFlowExceptionHandler.class);
    
    @ExceptionHandler(LiteFlowException.class)
    public Result handleLiteFlowException(LiteFlowException e) {
        logger.error("LiteFlow执行异常", e);
        return Result.error("流程执行失败: " + e.getMessage());
    }
    
    @ExceptionHandler(Exception.class)
    public Result handleException(Exception e) {
        logger.error("系统异常", e);
        return Result.error("系统异常，请稍后重试");
    }
}
```

#### 4.3.2 组件异常处理

```java
/**
 * 带异常处理的组件
 * 
 * @author erik.zhou
 */
@Component("safeComponent")
public class SafeComponent extends NodeComponent {
    
    private static final Logger logger = LoggerFactory.getLogger(SafeComponent.class);
    
    @Override
    public void process() {
        try {
            // 业务逻辑
            doBusinessLogic();
        } catch (BusinessException e) {
            // 业务异常处理
            logger.warn("业务异常: {}", e.getMessage());
            DefaultContext context = this.getContextBean(DefaultContext.class);
            context.setData("error", e.getMessage());
            // 不抛出异常，继续执行
        } catch (Exception e) {
            // 系统异常处理
            logger.error("系统异常", e);
            throw new RuntimeException("组件执行失败", e);
        }
    }
}
```

### 4.4 规则管理

#### 4.4.1 规则文件组织

```
resources/
├── flow/
│   ├── order/
│   │   ├── order-process.el.xml
│   │   └── order-refund.el.xml
│   ├── approval/
│   │   ├── approval-flow.el.xml
│   │   └── approval-route.el.xml
│   └── common/
│       └── common-flow.el.xml
```

**配置多个规则文件**：

```yaml
liteflow:
  rule-source: 
    - classpath:flow/order/*.el.xml
    - classpath:flow/approval/*.el.xml
    - classpath:flow/common/*.el.xml
```

#### 4.4.2 规则热部署

**从数据库加载规则**：

```java
import com.yomahub.liteflow.parser.el.ClassXmlFlowELParser;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * 规则动态加载器
 * 
 * @author erik.zhou
 */
@Component
public class RuleLoader {
    
    @Autowired
    private FlowExecutor flowExecutor;
    
    @Autowired
    private RuleRepository ruleRepository;
    
    /**
     * 从数据库加载规则
     */
    public void loadRulesFromDatabase() {
        // 从数据库查询规则
        List<Rule> rules = ruleRepository.findAll();
        
        // 构建规则XML
        StringBuilder xmlBuilder = new StringBuilder();
        xmlBuilder.append("<?xml version=\"1.0\" encoding=\"UTF-8\"?>");
        xmlBuilder.append("<flow>");
        
        for (Rule rule : rules) {
            xmlBuilder.append("<chain name=\"").append(rule.getName()).append("\">");
            xmlBuilder.append(rule.getElExpression());
            xmlBuilder.append("</chain>");
        }
        
        xmlBuilder.append("</flow>");
        
        // 重新加载规则
        flowExecutor.reloadRule(xmlBuilder.toString());
    }
    
    /**
     * 刷新单个规则链
     */
    public void refreshChain(String chainName) {
        Rule rule = ruleRepository.findByName(chainName);
        if (rule != null) {
            String xml = buildChainXml(rule);
            flowExecutor.reloadRule(xml);
        }
    }
    
    private String buildChainXml(Rule rule) {
        return String.format(
            "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" +
            "<flow>" +
            "<chain name=\"%s\">%s</chain>" +
            "</flow>",
            rule.getName(),
            rule.getElExpression()
        );
    }
}
```

#### 4.4.3 规则版本管理

```java
/**
 * 规则版本管理
 * 
 * @author erik.zhou
 */
@Service
public class RuleVersionService {
    
    @Autowired
    private RuleVersionRepository versionRepository;
    
    @Autowired
    private FlowExecutor flowExecutor;
    
    /**
     * 发布新版本规则
     */
    @Transactional(rollbackFor = Exception.class)
    public void publishVersion(String chainName, String elExpression) {
        // 保存新版本
        RuleVersion version = new RuleVersion();
        version.setChainName(chainName);
        version.setElExpression(elExpression);
        version.setVersion(generateVersion());
        version.setStatus("ACTIVE");
        version.setCreateTime(LocalDateTime.now());
        
        versionRepository.save(version);
        
        // 停用旧版本
        versionRepository.deactivateOldVersions(chainName);
        
        // 重新加载规则
        String xml = buildChainXml(version);
        flowExecutor.reloadRule(xml);
    }
    
    /**
     * 回滚到指定版本
     */
    @Transactional(rollbackFor = Exception.class)
    public void rollbackToVersion(Long versionId) {
        RuleVersion version = versionRepository.findById(versionId)
            .orElseThrow(() -> new IllegalArgumentException("版本不存在"));
        
        // 激活指定版本
        version.setStatus("ACTIVE");
        versionRepository.save(version);
        
        // 停用其他版本
        versionRepository.deactivateOtherVersions(version.getChainName(), versionId);
        
        // 重新加载规则
        String xml = buildChainXml(version);
        flowExecutor.reloadRule(xml);
    }
    
    private String generateVersion() {
        return "v" + System.currentTimeMillis();
    }
    
    private String buildChainXml(RuleVersion version) {
        return String.format(
            "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" +
            "<flow>" +
            "<chain name=\"%s\">%s</chain>" +
            "</flow>",
            version.getChainName(),
            version.getElExpression()
        );
    }
}
```

### 4.5 监控和日志

#### 4.5.1 启用监控

```yaml
liteflow:
  # 启用监控日志
  enable-log: true
  # 打印执行步骤
  print-execution-log: true
```

#### 4.5.2 自定义监控

```java
import com.yomahub.liteflow.monitor.MonitorBus;
import com.yomahub.liteflow.slot.Slot;
import org.springframework.stereotype.Component;

/**
 * 流程执行监控
 * 
 * @author erik.zhou
 */
@Component
public class FlowMonitor {
    
    private static final Logger logger = LoggerFactory.getLogger(FlowMonitor.class);
    
    /**
     * 监控流程执行
     */
    public void monitorExecution(String chainName, Slot slot) {
        // 记录执行时间
        long startTime = System.currentTimeMillis();
        
        try {
            // 执行流程
            // ...
        } finally {
            long endTime = System.currentTimeMillis();
            long duration = endTime - startTime;
            
            // 记录监控数据
            logger.info("流程执行监控 - 链名称: {}, 执行时间: {}ms", 
                       chainName, duration);
            
            // 如果执行时间过长，发送告警
            if (duration > 5000) {
                sendAlert(chainName, duration);
            }
        }
    }
    
    private void sendAlert(String chainName, long duration) {
        logger.warn("流程执行超时告警 - 链名称: {}, 执行时间: {}ms", 
                   chainName, duration);
        // 发送告警通知
    }
}
```

#### 4.5.3 组件执行日志

```java
/**
 * 带日志的组件基类
 * 
 * @author erik.zhou
 */
public abstract class LoggableComponent extends NodeComponent {
    
    protected final Logger logger = LoggerFactory.getLogger(getClass());
    
    @Override
    public final void process() {
        String componentName = this.getNodeId();
        logger.info("组件开始执行: {}", componentName);
        
        long startTime = System.currentTimeMillis();
        try {
            doProcess();
            long duration = System.currentTimeMillis() - startTime;
            logger.info("组件执行成功: {}, 耗时: {}ms", componentName, duration);
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            logger.error("组件执行失败: {}, 耗时: {}ms", componentName, duration, e);
            throw e;
        }
    }
    
    /**
     * 子类实现具体业务逻辑
     */
    protected abstract void doProcess();
}
```

### 4.6 测试最佳实践

#### 4.6.1 单元测试

```java
import com.yomahub.liteflow.core.FlowExecutor;
import com.yomahub.liteflow.flow.LiteflowResponse;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import static org.junit.jupiter.api.Assertions.*;

/**
 * LiteFlow流程测试
 * 
 * @author erik.zhou
 */
@SpringBootTest
public class OrderFlowTest {
    
    @Autowired
    private FlowExecutor flowExecutor;
    
    @Test
    public void testOrderProcessFlow() {
        // 准备测试数据
        OrderContext context = new OrderContext();
        Order order = new Order();
        order.setOrderNo("TEST001");
        order.setTotalAmount(1000.0);
        context.setOrder(order);
        
        // 执行流程
        LiteflowResponse response = flowExecutor.execute2Resp(
            "orderProcessChain",
            null,
            context
        );
        
        // 验证结果
        assertTrue(response.isSuccess(), "流程应该执行成功");
        assertNotNull(context.getData("finalAmount"), "应该计算出最终金额");
    }
    
    @Test
    public void testOrderValidationFail() {
        // 准备无效订单
        OrderContext context = new OrderContext();
        Order order = new Order();
        order.setTotalAmount(-100.0);  // 无效金额
        context.setOrder(order);
        
        // 执行流程
        LiteflowResponse response = flowExecutor.execute2Resp(
            "orderProcessChain",
            null,
            context
        );
        
        // 验证失败
        assertFalse(response.isSuccess(), "流程应该执行失败");
        assertNotNull(response.getCause(), "应该有异常信息");
    }
}
```

#### 4.6.2 Mock组件测试

```java
import org.mockito.Mockito;
import org.springframework.boot.test.mock.mockito.MockBean;

@SpringBootTest
public class MockComponentTest {
    
    @Autowired
    private FlowExecutor flowExecutor;
    
    @MockBean
    private PaymentService paymentService;
    
    @Test
    public void testPaymentFlow() {
        // Mock支付服务
        PaymentResult mockResult = new PaymentResult();
        mockResult.setSuccess(true);
        Mockito.when(paymentService.pay(Mockito.any(), Mockito.any(), Mockito.any()))
               .thenReturn(mockResult);
        
        // 执行流程
        OrderContext context = new OrderContext();
        context.setOrder(createTestOrder());
        
        LiteflowResponse response = flowExecutor.execute2Resp(
            "orderProcessChain",
            null,
            context
        );
        
        // 验证
        assertTrue(response.isSuccess());
        Mockito.verify(paymentService, Mockito.times(1))
               .pay(Mockito.any(), Mockito.any(), Mockito.any());
    }
}
```

## ❓ 常见问题

### Q1: LiteFlow和传统if-else相比有什么优势？

**A**: 
- **解耦性**：业务逻辑拆分成独立组件，易于维护
- **可读性**：EL表达式清晰表达流程逻辑
- **可扩展性**：新增业务只需添加组件，无需修改原有代码
- **可配置性**：流程可以通过配置动态调整
- **可测试性**：组件独立，易于单元测试
- **性能**：支持并行编排，提高执行效率

### Q2: 如何选择串行还是并行编排？

**A**: 
- **串行（THEN）**：组件之间有依赖关系，必须按顺序执行
- **并行（WHEN）**：组件之间无依赖，可以同时执行

```xml
<!-- 串行：后续组件依赖前面组件的结果 -->
<chain name="serial">
    THEN(validateOrder, checkStock, processPayment)
</chain>

<!-- 并行：组件之间无依赖 -->
<chain name="parallel">
    WHEN(sendSms, sendEmail, sendPush)
</chain>
```

### Q3: 上下文数据如何在组件间传递？

**A**: 
通过上下文的`setData`和`getData`方法：

```java
// 组件A存储数据
context.setData("key", value);

// 组件B获取数据
Object value = context.getData("key");
```

### Q4: 如何实现组件的条件执行？

**A**: 
使用IF或SWITCH表达式：

```xml
<!-- IF条件 -->
<chain name="conditionalChain">
    IF(isVip, THEN(vipProcess), THEN(normalProcess))
</chain>

<!-- SWITCH选择 -->
<chain name="switchChain">
    SWITCH(orderType).to(type1, type2, type3)
</chain>
```

### Q5: 如何处理组件执行异常？

**A**: 
1. 在组件内部try-catch处理
2. 实现rollback方法进行回滚
3. 使用全局异常处理器
4. 配置组件重试机制

```java
@Component("safeComponent")
@LiteflowRetry(retry = 3)
public class SafeComponent extends NodeComponent {
    @Override
    public void process() {
        try {
            // 业务逻辑
        } catch (Exception e) {
            // 异常处理
        }
    }
    
    @Override
    public void rollback() {
        // 回滚逻辑
    }
}
```

### Q6: LiteFlow支持哪些脚本语言？

**A**: 
- Groovy（推荐）
- JavaScript
- Python
- Lua
- QLExpress

需要添加对应的脚本引擎依赖。

### Q7: 如何实现规则的动态更新？

**A**: 
```java
// 从数据库或配置中心加载规则
String ruleXml = loadRuleFromDatabase();

// 重新加载规则
flowExecutor.reloadRule(ruleXml);
```

### Q8: 并行编排的线程池如何配置？

**A**: 
```yaml
liteflow:
  when-max-workers: 32        # 最大线程数
  when-queue-limit: 1024      # 队列大小
  when-max-wait-seconds: 15   # 最大等待时间
```

## 🔗 相关资源

### 官方资源
- [LiteFlow官网](https://liteflow.cc/)
- [GitHub仓库](https://github.com/dromara/liteFlow)
- [Gitee仓库](https://gitee.com/dromara/liteFlow)
- [官方文档](https://liteflow.cc/pages/5816c5/)

### 社区资源
- [LiteFlow社区](https://liteflow.cc/pages/community/)
- [问题反馈](https://github.com/dromara/liteFlow/issues)
- [讨论区](https://github.com/dromara/liteFlow/discussions)

### 推荐文章
- [LiteFlow框架分析系列](https://www.cnblogs.com/wasp520/p/19398574.html)
- [LiteFlow并行编排与异步超时](https://www.cnblogs.com/dalelee/p/17558175.html)
- [流程编排LiteFlow-业务代码解耦](https://www.cnblogs.com/w1570631036/p/18534261)

### 相关技术
- [Drools](https://www.drools.org/) - 规则引擎
- [Easy Rules](https://github.com/j-easy/easy-rules) - 轻量级规则引擎
- [Activiti](https://www.activiti.org/) - 工作流引擎
- [Camunda](https://camunda.com/) - BPM平台

## 📝 学习检查清单

### 基础知识
- [ ] 理解规则引擎和流程编排的概念
- [ ] 掌握LiteFlow的核心理念
- [ ] 理解组件的概念和类型
- [ ] 掌握上下文的作用和使用
- [ ] 了解LiteFlow的应用场景

### 核心特性
- [ ] 掌握普通组件的定义和使用
- [ ] 掌握条件组件（IF/SWITCH）
- [ ] 理解EL表达式语法
- [ ] 掌握串行编排（THEN）
- [ ] 掌握并行编排（WHEN）
- [ ] 掌握条件编排（IF/SWITCH）
- [ ] 掌握循环编排（FOR/ITERATOR）
- [ ] 理解脚本组件的使用

### 实战能力
- [ ] 能够设计和实现业务流程
- [ ] 能够编写复杂的EL表达式
- [ ] 能够处理组件间数据传递
- [ ] 能够实现异常处理和回滚
- [ ] 能够进行性能优化
- [ ] 能够实现规则热部署

### 最佳实践
- [ ] 掌握组件设计原则
- [ ] 理解性能优化技巧
- [ ] 掌握异常处理模式
- [ ] 理解规则管理方法
- [ ] 掌握监控和日志记录
- [ ] 能够编写单元测试

---

**学习建议**：
1. 先理解规则引擎的基本概念
2. 从简单的串行流程开始实践
3. 逐步学习并行、条件等复杂编排
4. 在实际项目中应用所学知识
5. 关注性能优化和最佳实践
6. 参与社区讨论，学习他人经验

**预计学习时长**: 15-25小时（基础学习）+ 30-50小时（进阶学习）

**下一步学习**：
- [设计模式](./设计模式-完整教程.md)：深入理解责任链、策略等模式
- [并发编程](./并发编程-完整教程.md)：理解并行编排的底层原理
- [Spring Boot](../02-Spring生态/02-Spring-Boot-完整教程.md)：集成到Spring Boot项目

**实战项目推荐**：
1. 电商订单处理系统
2. 企业审批流程系统
3. 数据ETL处理管道
4. 营销规则引擎
5. 风控决策引擎
