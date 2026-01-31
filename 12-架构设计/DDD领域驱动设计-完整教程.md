# DDD领域驱动设计-完整教程

> @author erik.zhou  
> 难度: ⭐⭐⭐⭐⭐  
> 适用场景: 复杂业务系统、微服务架构

## 📋 目录

- [DDD核心概念](#ddd核心概念)
- [DDD分层架构](#ddd分层架构)
- [DDD战术设计](#ddd战术设计)
- [DDD战略设计](#ddd战略设计)
- [DDD项目搭建](#ddd项目搭建)
- [DDD最佳实践](#ddd最佳实践)
- [实战案例](#实战案例)

---

## 🎯 DDD核心概念

### 什么是DDD

领域驱动设计（Domain-Driven Design，DDD）是一种软件开发方法论，由Eric Evans在2003年提出。DDD的核心思想是：
- **以业务领域为核心**，而不是技术实现
- **通过统一语言**连接业务和技术
- **将复杂业务拆分**为多个限界上下文
- **用领域模型**表达业务规则

### DDD解决什么问题

1. **业务复杂性** - 将复杂业务拆分为可管理的模块
2. **沟通成本** - 通过统一语言减少业务和技术的沟通障碍
3. **代码腐化** - 通过清晰的架构保持代码质量
4. **需求变更** - 通过领域模型快速响应业务变化

### DDD核心概念图

```
┌─────────────────────────────────────────────────────────┐
│                    战略设计 (Strategic Design)            │
├─────────────────────────────────────────────────────────┤
│  限界上下文 (Bounded Context)                             │
│  上下文映射 (Context Mapping)                             │
│  子域划分 (Subdomain)                                     │
└─────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────┐
│                    战术设计 (Tactical Design)             │
├─────────────────────────────────────────────────────────┤
│  实体 (Entity)                                           │
│  值对象 (Value Object)                                   │
│  聚合 (Aggregate)                                        │
│  聚合根 (Aggregate Root)                                 │
│  领域服务 (Domain Service)                               │
│  领域事件 (Domain Event)                                 │
│  仓储 (Repository)                                       │
│  工厂 (Factory)                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 🏗️ DDD分层架构

### 经典四层架构

```
┌─────────────────────────────────────────┐
│         用户接口层 (User Interface)       │
│  Controller / API / Web / RPC           │
│  职责: 接收请求、参数校验、返回响应        │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│         应用层 (Application)              │
│  ApplicationService / Facade             │
│  职责: 编排业务流程、事务控制、权限校验    │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│         领域层 (Domain)                   │
│  Entity / ValueObject / DomainService    │
│  职责: 核心业务逻辑、业务规则             │
└─────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────┐
│         基础设施层 (Infrastructure)       │
│  Repository / Mapper / MQ / Cache        │
│  职责: 数据持久化、外部服务调用           │
└─────────────────────────────────────────┘
```

### 依赖关系

```
用户接口层 ──→ 应用层 ──→ 领域层 ←── 基础设施层
                              ↑
                              │
                         (依赖倒置)
```

**核心原则**：
- 上层依赖下层
- 领域层不依赖任何层（通过接口实现依赖倒置）
- 基础设施层实现领域层定义的接口

---

## ⚔️ DDD战术设计

### 1. 实体 (Entity)

**定义**：具有唯一标识的对象，标识在整个生命周期中保持不变。

**特点**：
- 有唯一ID
- 可变的
- 通过ID判断相等性



```java
/**
 * 实体示例：订单
 * @author erik.zhou
 */
@Entity
public class Order {
    
    // 唯一标识
    private OrderId id;
    
    // 订单号
    private String orderNo;
    
    // 用户ID
    private UserId userId;
    
    // 订单状态
    private OrderStatus status;
    
    // 订单金额
    private Money totalAmount;
    
    // 订单项列表
    private List<OrderItem> items;
    
    // 创建时间
    private LocalDateTime createTime;
    
    /**
     * 创建订单（工厂方法）
     */
    public static Order create(UserId userId, List<OrderItem> items) {
        // 业务规则校验
        if (items == null || items.isEmpty()) {
            throw new DomainException("订单项不能为空");
        }
        
        Order order = new Order();
        order.id = OrderId.generate();
        order.orderNo = generateOrderNo();
        order.userId = userId;
        order.items = new ArrayList<>(items);
        order.status = OrderStatus.CREATED;
        order.totalAmount = calculateTotalAmount(items);
        order.createTime = LocalDateTime.now();
        
        // 发布领域事件
        order.addDomainEvent(new OrderCreatedEvent(order.id));
        
        return order;
    }
    
    /**
     * 支付订单
     */
    public void pay(PaymentMethod paymentMethod) {
        // 业务规则校验
        if (this.status != OrderStatus.CREATED) {
            throw new DomainException("订单状态不正确，无法支付");
        }
        
        // 状态变更
        this.status = OrderStatus.PAID;
        
        // 发布领域事件
        this.addDomainEvent(new OrderPaidEvent(this.id, paymentMethod));
    }
    
    /**
     * 取消订单
     */
    public void cancel(String reason) {
        // 业务规则校验
        if (this.status == OrderStatus.COMPLETED) {
            throw new DomainException("订单已完成，无法取消");
        }
        
        if (this.status == OrderStatus.CANCELLED) {
            throw new DomainException("订单已取消");
        }
        
        // 状态变更
        this.status = OrderStatus.CANCELLED;
        
        // 发布领域事件
        this.addDomainEvent(new OrderCancelledEvent(this.id, reason));
    }
    
    /**
     * 计算总金额
     */
    private static Money calculateTotalAmount(List<OrderItem> items) {
        return items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
    
    /**
     * 生成订单号
     */
    private static String generateOrderNo() {
        return "ORD" + System.currentTimeMillis();
    }
    
    // 相等性判断（基于ID）
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Order order = (Order) o;
        return Objects.equals(id, order.id);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

### 2. 值对象 (Value Object)

**定义**：没有唯一标识的对象，通过属性值判断相等性。

**特点**：
- 无ID
- 不可变的
- 通过属性值判断相等性
- 可以被共享

```java
/**
 * 值对象示例：金额
 * @author erik.zhou
 */
public class Money {
    
    private final BigDecimal amount;
    private final Currency currency;
    
    public static final Money ZERO = new Money(BigDecimal.ZERO, Currency.CNY);
    
    /**
     * 构造函数私有化
     */
    private Money(BigDecimal amount, Currency currency) {
        if (amount == null) {
            throw new IllegalArgumentException("金额不能为空");
        }
        if (currency == null) {
            throw new IllegalArgumentException("币种不能为空");
        }
        if (amount.compareTo(BigDecimal.ZERO) < 0) {
            throw new IllegalArgumentException("金额不能为负数");
        }
        
        this.amount = amount.setScale(2, RoundingMode.HALF_UP);
        this.currency = currency;
    }
    
    /**
     * 工厂方法
     */
    public static Money of(BigDecimal amount, Currency currency) {
        return new Money(amount, currency);
    }
    
    public static Money ofCNY(BigDecimal amount) {
        return new Money(amount, Currency.CNY);
    }
    
    /**
     * 加法
     */
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("币种不同，无法相加");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }
    
    /**
     * 减法
     */
    public Money subtract(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("币种不同，无法相减");
        }
        return new Money(this.amount.subtract(other.amount), this.currency);
    }
    
    /**
     * 乘法
     */
    public Money multiply(BigDecimal multiplier) {
        return new Money(this.amount.multiply(multiplier), this.currency);
    }
    
    /**
     * 比较
     */
    public boolean greaterThan(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("币种不同，无法比较");
        }
        return this.amount.compareTo(other.amount) > 0;
    }
    
    // Getter方法
    public BigDecimal getAmount() {
        return amount;
    }
    
    public Currency getCurrency() {
        return currency;
    }
    
    // 相等性判断（基于属性值）
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Money money = (Money) o;
        return Objects.equals(amount, money.amount) &&
               currency == money.currency;
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(amount, currency);
    }
    
    @Override
    public String toString() {
        return currency.getSymbol() + amount;
    }
}

/**
 * 值对象示例：地址
 * @author erik.zhou
 */
public class Address {
    
    private final String province;
    private final String city;
    private final String district;
    private final String street;
    private final String detail;
    private final String zipCode;
    
    private Address(String province, String city, String district, 
                   String street, String detail, String zipCode) {
        this.province = province;
        this.city = city;
        this.district = district;
        this.street = street;
        this.detail = detail;
        this.zipCode = zipCode;
    }
    
    public static Address of(String province, String city, String district,
                           String street, String detail, String zipCode) {
        // 参数校验
        if (province == null || province.isEmpty()) {
            throw new IllegalArgumentException("省份不能为空");
        }
        if (city == null || city.isEmpty()) {
            throw new IllegalArgumentException("城市不能为空");
        }
        
        return new Address(province, city, district, street, detail, zipCode);
    }
    
    /**
     * 获取完整地址
     */
    public String getFullAddress() {
        return province + city + district + street + detail;
    }
    
    // Getter方法（只读）
    public String getProvince() { return province; }
    public String getCity() { return city; }
    public String getDistrict() { return district; }
    public String getStreet() { return street; }
    public String getDetail() { return detail; }
    public String getZipCode() { return zipCode; }
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Address address = (Address) o;
        return Objects.equals(province, address.province) &&
               Objects.equals(city, address.city) &&
               Objects.equals(district, address.district) &&
               Objects.equals(street, address.street) &&
               Objects.equals(detail, address.detail) &&
               Objects.equals(zipCode, address.zipCode);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(province, city, district, street, detail, zipCode);
    }
}
```

### 3. 聚合 (Aggregate) 和聚合根 (Aggregate Root)

**定义**：
- **聚合**：一组相关对象的集合，作为数据修改的单元
- **聚合根**：聚合的根实体，外部只能通过聚合根访问聚合内部对象

**特点**：
- 聚合根负责维护聚合内部的一致性
- 外部只能持有聚合根的引用
- 聚合之间通过ID引用，而不是对象引用
- 聚合是事务边界

```java
/**
 * 聚合根示例：订单
 * @author erik.zhou
 */
@Entity
@AggregateRoot
public class Order {
    
    // 聚合根ID
    private OrderId id;
    
    // 订单项（聚合内部实体）
    private List<OrderItem> items;
    
    // 收货地址（值对象）
    private Address shippingAddress;
    
    // 订单状态
    private OrderStatus status;
    
    /**
     * 添加订单项
     * 通过聚合根操作聚合内部对象
     */
    public void addItem(ProductId productId, int quantity, Money price) {
        // 业务规则校验
        if (this.status != OrderStatus.CREATED) {
            throw new DomainException("订单状态不正确，无法添加商品");
        }
        
        // 检查是否已存在该商品
        Optional<OrderItem> existingItem = items.stream()
            .filter(item -> item.getProductId().equals(productId))
            .findFirst();
        
        if (existingItem.isPresent()) {
            // 已存在，增加数量
            existingItem.get().increaseQuantity(quantity);
        } else {
            // 不存在，新增订单项
            OrderItem newItem = OrderItem.create(productId, quantity, price);
            items.add(newItem);
        }
        
        // 重新计算总金额
        recalculateTotalAmount();
    }
    
    /**
     * 移除订单项
     */
    public void removeItem(OrderItemId itemId) {
        if (this.status != OrderStatus.CREATED) {
            throw new DomainException("订单状态不正确，无法移除商品");
        }
        
        items.removeIf(item -> item.getId().equals(itemId));
        recalculateTotalAmount();
    }
    
    /**
     * 修改收货地址
     */
    public void changeShippingAddress(Address newAddress) {
        if (this.status != OrderStatus.CREATED) {
            throw new DomainException("订单状态不正确，无法修改地址");
        }
        
        this.shippingAddress = newAddress;
    }
    
    /**
     * 重新计算总金额
     * 维护聚合内部一致性
     */
    private void recalculateTotalAmount() {
        this.totalAmount = items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
    
    // 不允许外部直接访问订单项
    // 只提供只读访问
    public List<OrderItem> getItems() {
        return Collections.unmodifiableList(items);
    }
}

/**
 * 聚合内部实体：订单项
 * @author erik.zhou
 */
@Entity
public class OrderItem {
    
    private OrderItemId id;
    private ProductId productId;
    private int quantity;
    private Money price;
    private Money subtotal;
    
    /**
     * 创建订单项
     */
    public static OrderItem create(ProductId productId, int quantity, Money price) {
        if (quantity <= 0) {
            throw new DomainException("数量必须大于0");
        }
        
        OrderItem item = new OrderItem();
        item.id = OrderItemId.generate();
        item.productId = productId;
        item.quantity = quantity;
        item.price = price;
        item.subtotal = price.multiply(BigDecimal.valueOf(quantity));
        
        return item;
    }
    
    /**
     * 增加数量
     */
    public void increaseQuantity(int amount) {
        if (amount <= 0) {
            throw new DomainException("增加数量必须大于0");
        }
        
        this.quantity += amount;
        this.subtotal = this.price.multiply(BigDecimal.valueOf(this.quantity));
    }
    
    /**
     * 减少数量
     */
    public void decreaseQuantity(int amount) {
        if (amount <= 0) {
            throw new DomainException("减少数量必须大于0");
        }
        
        if (this.quantity - amount < 0) {
            throw new DomainException("数量不足");
        }
        
        this.quantity -= amount;
        this.subtotal = this.price.multiply(BigDecimal.valueOf(this.quantity));
    }
    
    // Getter方法
    public OrderItemId getId() { return id; }
    public ProductId getProductId() { return productId; }
    public int getQuantity() { return quantity; }
    public Money getPrice() { return price; }
    public Money getSubtotal() { return subtotal; }
}
```

### 4. 领域服务 (Domain Service)

**定义**：当某个业务逻辑不适合放在实体或值对象中时，使用领域服务。

**使用场景**：
- 涉及多个聚合的业务逻辑
- 无状态的业务操作
- 领域计算和转换

```java
/**
 * 领域服务示例：订单定价服务
 * @author erik.zhou
 */
@DomainService
public class OrderPricingService {
    
    /**
     * 计算订单价格
     * 涉及多个聚合：订单、商品、优惠券
     */
    public Money calculateOrderPrice(Order order, List<Product> products, 
                                    Coupon coupon) {
        // 1. 计算商品总价
        Money subtotal = Money.ZERO;
        for (OrderItem item : order.getItems()) {
            Product product = findProduct(products, item.getProductId());
            Money itemPrice = product.getPrice().multiply(
                BigDecimal.valueOf(item.getQuantity()));
            subtotal = subtotal.add(itemPrice);
        }
        
        // 2. 应用优惠券
        Money discount = Money.ZERO;
        if (coupon != null && coupon.isApplicable(order)) {
            discount = coupon.calculateDiscount(subtotal);
        }
        
        // 3. 计算最终价格
        Money finalPrice = subtotal.subtract(discount);
        
        return finalPrice;
    }
    
    private Product findProduct(List<Product> products, ProductId productId) {
        return products.stream()
            .filter(p -> p.getId().equals(productId))
            .findFirst()
            .orElseThrow(() -> new DomainException("商品不存在"));
    }
}

/**
 * 领域服务示例：库存服务
 * @author erik.zhou
 */
@DomainService
public class InventoryService {
    
    /**
     * 检查库存是否充足
     */
    public boolean checkInventory(Order order, List<Inventory> inventories) {
        for (OrderItem item : order.getItems()) {
            Inventory inventory = findInventory(inventories, item.getProductId());
            
            if (inventory.getAvailableQuantity() < item.getQuantity()) {
                return false;
            }
        }
        
        return true;
    }
    
    /**
     * 扣减库存
     */
    public void deductInventory(Order order, List<Inventory> inventories) {
        for (OrderItem item : order.getItems()) {
            Inventory inventory = findInventory(inventories, item.getProductId());
            inventory.deduct(item.getQuantity());
        }
    }
    
    private Inventory findInventory(List<Inventory> inventories, ProductId productId) {
        return inventories.stream()
            .filter(i -> i.getProductId().equals(productId))
            .findFirst()
            .orElseThrow(() -> new DomainException("库存不存在"));
    }
}
```

### 5. 领域事件 (Domain Event)

**定义**：领域中发生的重要业务事件。

**特点**：
- 不可变的
- 过去式命名
- 包含事件发生的时间和相关数据

```java
/**
 * 领域事件基类
 * @author erik.zhou
 */
public abstract class DomainEvent {
    
    private final String eventId;
    private final LocalDateTime occurredOn;
    
    protected DomainEvent() {
        this.eventId = UUID.randomUUID().toString();
        this.occurredOn = LocalDateTime.now();
    }
    
    public String getEventId() {
        return eventId;
    }
    
    public LocalDateTime getOccurredOn() {
        return occurredOn;
    }
}

/**
 * 订单创建事件
 * @author erik.zhou
 */
public class OrderCreatedEvent extends DomainEvent {
    
    private final OrderId orderId;
    private final UserId userId;
    private final Money totalAmount;
    
    public OrderCreatedEvent(OrderId orderId, UserId userId, Money totalAmount) {
        super();
        this.orderId = orderId;
        this.userId = userId;
        this.totalAmount = totalAmount;
    }
    
    public OrderId getOrderId() { return orderId; }
    public UserId getUserId() { return userId; }
    public Money getTotalAmount() { return totalAmount; }
}

/**
 * 订单支付事件
 * @author erik.zhou
 */
public class OrderPaidEvent extends DomainEvent {
    
    private final OrderId orderId;
    private final PaymentMethod paymentMethod;
    private final Money amount;
    
    public OrderPaidEvent(OrderId orderId, PaymentMethod paymentMethod, Money amount) {
        super();
        this.orderId = orderId;
        this.paymentMethod = paymentMethod;
        this.amount = amount;
    }
    
    public OrderId getOrderId() { return orderId; }
    public PaymentMethod getPaymentMethod() { return paymentMethod; }
    public Money getAmount() { return amount; }
}

/**
 * 领域事件发布器
 * @author erik.zhou
 */
@Component
public class DomainEventPublisher {
    
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    /**
     * 发布领域事件
     */
    public void publish(DomainEvent event) {
        eventPublisher.publishEvent(event);
    }
    
    /**
     * 批量发布领域事件
     */
    public void publishAll(List<DomainEvent> events) {
        events.forEach(this::publish);
    }
}

/**
 * 领域事件监听器
 * @author erik.zhou
 */
@Component
public class OrderEventListener {
    
    @Autowired
    private NotificationService notificationService;
    
    @Autowired
    private InventoryService inventoryService;
    
    /**
     * 监听订单创建事件
     */
    @EventListener
    @Async
    public void handleOrderCreated(OrderCreatedEvent event) {
        // 发送通知
        notificationService.sendOrderCreatedNotification(
            event.getUserId(), event.getOrderId());
    }
    
    /**
     * 监听订单支付事件
     */
    @EventListener
    @Transactional
    public void handleOrderPaid(OrderPaidEvent event) {
        // 扣减库存
        inventoryService.deductInventory(event.getOrderId());
        
        // 发送通知
        notificationService.sendOrderPaidNotification(
            event.getOrderId(), event.getAmount());
    }
}
```

### 6. 仓储 (Repository)

**定义**：提供聚合的持久化和查询接口。

**特点**：
- 面向聚合根
- 隐藏持久化细节
- 提供领域语言的查询方法

```java
/**
 * 仓储接口
 * @author erik.zhou
 */
public interface OrderRepository {
    
    /**
     * 保存订单
     */
    void save(Order order);
    
    /**
     * 根据ID查找订单
     */
    Optional<Order> findById(OrderId orderId);
    
    /**
     * 根据订单号查找订单
     */
    Optional<Order> findByOrderNo(String orderNo);
    
    /**
     * 查找用户的订单列表
     */
    List<Order> findByUserId(UserId userId, int page, int size);
    
    /**
     * 查找待支付订单
     */
    List<Order> findPendingOrders(LocalDateTime before);
    
    /**
     * 删除订单
     */
    void remove(Order order);
}

/**
 * 仓储实现
 * @author erik.zhou
 */
@Repository
public class OrderRepositoryImpl implements OrderRepository {
    
    @Autowired
    private OrderMapper orderMapper;
    
    @Autowired
    private OrderItemMapper orderItemMapper;
    
    @Autowired
    private DomainEventPublisher eventPublisher;
    
    @Override
    @Transactional
    public void save(Order order) {
        // 1. 转换为数据模型
        OrderDO orderDO = OrderConverter.toDataObject(order);
        
        // 2. 保存订单
        if (orderDO.getId() == null) {
            orderMapper.insert(orderDO);
        } else {
            orderMapper.update(orderDO);
        }
        
        // 3. 保存订单项
        List<OrderItemDO> itemDOs = OrderConverter.toItemDataObjects(order.getItems());
        orderItemMapper.batchInsert(itemDOs);
        
        // 4. 发布领域事件
        eventPublisher.publishAll(order.getDomainEvents());
        order.clearDomainEvents();
    }
    
    @Override
    public Optional<Order> findById(OrderId orderId) {
        // 1. 查询订单
        OrderDO orderDO = orderMapper.selectById(orderId.getValue());
        if (orderDO == null) {
            return Optional.empty();
        }
        
        // 2. 查询订单项
        List<OrderItemDO> itemDOs = orderItemMapper.selectByOrderId(orderId.getValue());
        
        // 3. 转换为领域模型
        Order order = OrderConverter.toDomainObject(orderDO, itemDOs);
        
        return Optional.of(order);
    }
    
    @Override
    public Optional<Order> findByOrderNo(String orderNo) {
        OrderDO orderDO = orderMapper.selectByOrderNo(orderNo);
        if (orderDO == null) {
            return Optional.empty();
        }
        
        List<OrderItemDO> itemDOs = orderItemMapper.selectByOrderId(orderDO.getId());
        Order order = OrderConverter.toDomainObject(orderDO, itemDOs);
        
        return Optional.of(order);
    }
    
    @Override
    public List<Order> findByUserId(UserId userId, int page, int size) {
        int offset = (page - 1) * size;
        List<OrderDO> orderDOs = orderMapper.selectByUserId(
            userId.getValue(), offset, size);
        
        return orderDOs.stream()
            .map(orderDO -> {
                List<OrderItemDO> itemDOs = orderItemMapper.selectByOrderId(orderDO.getId());
                return OrderConverter.toDomainObject(orderDO, itemDOs);
            })
            .collect(Collectors.toList());
    }
    
    @Override
    public List<Order> findPendingOrders(LocalDateTime before) {
        List<OrderDO> orderDOs = orderMapper.selectPendingOrders(before);
        
        return orderDOs.stream()
            .map(orderDO -> {
                List<OrderItemDO> itemDOs = orderItemMapper.selectByOrderId(orderDO.getId());
                return OrderConverter.toDomainObject(orderDO, itemDOs);
            })
            .collect(Collectors.toList());
    }
    
    @Override
    @Transactional
    public void remove(Order order) {
        orderItemMapper.deleteByOrderId(order.getId().getValue());
        orderMapper.deleteById(order.getId().getValue());
    }
}
```


### 7. 工厂 (Factory)

**定义**：负责创建复杂对象和聚合。

```java
/**
 * 订单工厂
 * @author erik.zhou
 */
@Component
public class OrderFactory {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private OrderPricingService pricingService;
    
    /**
     * 创建订单
     */
    public Order createOrder(UserId userId, List<OrderItemRequest> itemRequests,
                           Address shippingAddress, Coupon coupon) {
        // 1. 验证商品
        List<Product> products = validateProducts(itemRequests);
        
        // 2. 创建订单项
        List<OrderItem> items = createOrderItems(itemRequests, products);
        
        // 3. 创建订单
        Order order = Order.create(userId, items);
        order.changeShippingAddress(shippingAddress);
        
        // 4. 计算价格
        Money totalPrice = pricingService.calculateOrderPrice(order, products, coupon);
        order.setTotalAmount(totalPrice);
        
        return order;
    }
    
    private List<Product> validateProducts(List<OrderItemRequest> requests) {
        List<ProductId> productIds = requests.stream()
            .map(OrderItemRequest::getProductId)
            .collect(Collectors.toList());
        
        List<Product> products = productRepository.findByIds(productIds);
        
        if (products.size() != productIds.size()) {
            throw new DomainException("部分商品不存在");
        }
        
        return products;
    }
    
    private List<OrderItem> createOrderItems(List<OrderItemRequest> requests,
                                            List<Product> products) {
        return requests.stream()
            .map(request -> {
                Product product = findProduct(products, request.getProductId());
                return OrderItem.create(
                    request.getProductId(),
                    request.getQuantity(),
                    product.getPrice()
                );
            })
            .collect(Collectors.toList());
    }
}
```

---

## 🗺️ DDD战略设计

### 1. 限界上下文 (Bounded Context)

**定义**：明确的边界，在边界内统一语言和模型。

**示例：电商系统的限界上下文划分**

```
┌─────────────────────────────────────────────────────────┐
│                      电商系统                             │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  订单上下文   │  │  商品上下文   │  │  用户上下文   │  │
│  │              │  │              │  │              │  │
│  │  - 订单      │  │  - 商品      │  │  - 用户      │  │
│  │  - 订单项    │  │  - 类目      │  │  - 地址      │  │
│  │  - 支付      │  │  - 品牌      │  │  - 积分      │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  库存上下文   │  │  营销上下文   │  │  物流上下文   │  │
│  │              │  │              │  │              │  │
│  │  - 库存      │  │  - 优惠券    │  │  - 物流单    │  │
│  │  - 仓库      │  │  - 活动      │  │  - 配送      │  │
│  │  - 调拨      │  │  - 积分      │  │  - 签收      │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 2. 上下文映射 (Context Mapping)

**上下文之间的关系模式**：

1. **共享内核 (Shared Kernel)**
   - 两个上下文共享部分模型
   - 需要紧密协作

2. **客户-供应商 (Customer-Supplier)**
   - 上游供应商，下游客户
   - 供应商提供API

3. **遵奉者 (Conformist)**
   - 下游完全遵循上游模型
   - 无法影响上游

4. **防腐层 (Anti-Corruption Layer)**
   - 使用适配器隔离外部模型
   - 保护自己的领域模型

5. **开放主机服务 (Open Host Service)**
   - 提供标准化的API
   - 供多个下游使用

6. **发布语言 (Published Language)**
   - 使用标准化的数据格式
   - 如JSON、XML

```java
/**
 * 防腐层示例：订单上下文访问商品上下文
 * @author erik.zhou
 */
@Component
public class ProductServiceAdapter {
    
    @Autowired
    private ProductServiceClient productServiceClient;
    
    /**
     * 获取商品信息
     * 将外部模型转换为内部模型
     */
    public Product getProduct(ProductId productId) {
        // 1. 调用外部服务
        ProductDTO productDTO = productServiceClient.getProduct(productId.getValue());
        
        // 2. 转换为领域模型（防腐层）
        return convertToDomainModel(productDTO);
    }
    
    /**
     * 批量获取商品信息
     */
    public List<Product> getProducts(List<ProductId> productIds) {
        List<String> ids = productIds.stream()
            .map(ProductId::getValue)
            .collect(Collectors.toList());
        
        List<ProductDTO> productDTOs = productServiceClient.batchGetProducts(ids);
        
        return productDTOs.stream()
            .map(this::convertToDomainModel)
            .collect(Collectors.toList());
    }
    
    /**
     * 转换为领域模型
     * 隔离外部模型的变化
     */
    private Product convertToDomainModel(ProductDTO dto) {
        return Product.builder()
            .id(ProductId.of(dto.getId()))
            .name(dto.getName())
            .price(Money.ofCNY(dto.getPrice()))
            .categoryId(CategoryId.of(dto.getCategoryId()))
            .brandId(BrandId.of(dto.getBrandId()))
            .build();
    }
}
```

### 3. 子域划分 (Subdomain)

**子域类型**：

1. **核心域 (Core Domain)**
   - 业务核心竞争力
   - 投入最多资源
   - 示例：推荐算法、定价策略

2. **支撑域 (Supporting Domain)**
   - 支持核心域
   - 有一定业务价值
   - 示例：订单管理、库存管理

3. **通用域 (Generic Domain)**
   - 通用功能
   - 可以使用现成方案
   - 示例：用户认证、消息通知

```
电商系统子域划分：

核心域：
  - 商品推荐
  - 动态定价
  - 智能营销

支撑域：
  - 订单管理
  - 库存管理
  - 物流管理

通用域：
  - 用户认证
  - 消息通知
  - 文件存储
```

---

## 🏗️ DDD项目搭建

### 项目结构

```
ecommerce-order/
├── order-api/                    # API层
│   ├── controller/
│   │   └── OrderController.java
│   └── dto/
│       ├── OrderRequest.java
│       └── OrderResponse.java
│
├── order-application/            # 应用层
│   ├── service/
│   │   └── OrderApplicationService.java
│   ├── assembler/
│   │   └── OrderAssembler.java
│   └── command/
│       ├── CreateOrderCommand.java
│       └── PayOrderCommand.java
│
├── order-domain/                 # 领域层
│   ├── model/
│   │   ├── aggregate/
│   │   │   ├── Order.java
│   │   │   └── OrderItem.java
│   │   ├── entity/
│   │   ├── valueobject/
│   │   │   ├── OrderId.java
│   │   │   ├── Money.java
│   │   │   └── Address.java
│   │   └── event/
│   │       ├── OrderCreatedEvent.java
│   │       └── OrderPaidEvent.java
│   ├── service/
│   │   ├── OrderPricingService.java
│   │   └── InventoryService.java
│   └── repository/
│       └── OrderRepository.java
│
└── order-infrastructure/         # 基础设施层
    ├── persistence/
    │   ├── mapper/
    │   │   ├── OrderMapper.java
    │   │   └── OrderItemMapper.java
    │   ├── dataobject/
    │   │   ├── OrderDO.java
    │   │   └── OrderItemDO.java
    │   └── repository/
    │       └── OrderRepositoryImpl.java
    ├── client/
    │   ├── ProductServiceClient.java
    │   └── UserServiceClient.java
    └── mq/
        ├── OrderEventPublisher.java
        └── OrderEventListener.java
```

### Maven依赖配置

```xml
<!-- 父POM -->
<project>
    <groupId>com.example</groupId>
    <artifactId>ecommerce-order</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <modules>
        <module>order-api</module>
        <module>order-application</module>
        <module>order-domain</module>
        <module>order-infrastructure</module>
    </modules>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
    </dependencies>
</project>

<!-- order-domain模块 -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>ecommerce-order</artifactId>
        <version>1.0.0</version>
    </parent>
    
    <artifactId>order-domain</artifactId>
    
    <!-- 领域层不依赖任何其他层 -->
    <dependencies>
        <!-- 只依赖基础工具 -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
    </dependencies>
</project>

<!-- order-application模块 -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>ecommerce-order</artifactId>
        <version>1.0.0</version>
    </parent>
    
    <artifactId>order-application</artifactId>
    
    <dependencies>
        <!-- 依赖领域层 -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>order-domain</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>
</project>

<!-- order-infrastructure模块 -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>ecommerce-order</artifactId>
        <version>1.0.0</version>
    </parent>
    
    <artifactId>order-infrastructure</artifactId>
    
    <dependencies>
        <!-- 依赖领域层 -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>order-domain</artifactId>
        </dependency>
        
        <!-- 持久化 -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
        </dependency>
        
        <!-- 消息队列 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        
        <!-- Redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
    </dependencies>
</project>

<!-- order-api模块 -->
<project>
    <parent>
        <groupId>com.example</groupId>
        <artifactId>ecommerce-order</artifactId>
        <version>1.0.0</version>
    </parent>
    
    <artifactId>order-api</artifactId>
    
    <dependencies>
        <!-- 依赖应用层 -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>order-application</artifactId>
        </dependency>
        
        <!-- 依赖基础设施层 -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>order-infrastructure</artifactId>
        </dependency>
        
        <!-- Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```

### 完整代码示例

#### 1. API层

```java
/**
 * 订单控制器
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    @Autowired
    private OrderApplicationService orderApplicationService;
    
    @Autowired
    private OrderAssembler orderAssembler;
    
    /**
     * 创建订单
     */
    @PostMapping
    public Result<OrderResponse> createOrder(@RequestBody @Valid CreateOrderRequest request) {
        // 1. 转换为命令
        CreateOrderCommand command = orderAssembler.toCommand(request);
        
        // 2. 执行命令
        OrderId orderId = orderApplicationService.createOrder(command);
        
        // 3. 查询订单
        Order order = orderApplicationService.getOrder(orderId);
        
        // 4. 转换为响应
        OrderResponse response = orderAssembler.toResponse(order);
        
        return Result.success(response);
    }
    
    /**
     * 支付订单
     */
    @PostMapping("/{orderId}/pay")
    public Result<Void> payOrder(@PathVariable String orderId,
                                @RequestBody @Valid PayOrderRequest request) {
        PayOrderCommand command = new PayOrderCommand(
            OrderId.of(orderId),
            request.getPaymentMethod()
        );
        
        orderApplicationService.payOrder(command);
        
        return Result.success();
    }
    
    /**
     * 取消订单
     */
    @PostMapping("/{orderId}/cancel")
    public Result<Void> cancelOrder(@PathVariable String orderId,
                                   @RequestBody @Valid CancelOrderRequest request) {
        CancelOrderCommand command = new CancelOrderCommand(
            OrderId.of(orderId),
            request.getReason()
        );
        
        orderApplicationService.cancelOrder(command);
        
        return Result.success();
    }
    
    /**
     * 查询订单
     */
    @GetMapping("/{orderId}")
    public Result<OrderResponse> getOrder(@PathVariable String orderId) {
        Order order = orderApplicationService.getOrder(OrderId.of(orderId));
        OrderResponse response = orderAssembler.toResponse(order);
        return Result.success(response);
    }
    
    /**
     * 查询用户订单列表
     */
    @GetMapping("/user/{userId}")
    public Result<PageResult<OrderResponse>> getUserOrders(
            @PathVariable String userId,
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "10") int size) {
        
        PageResult<Order> orders = orderApplicationService.getUserOrders(
            UserId.of(userId), page, size);
        
        PageResult<OrderResponse> response = orders.map(orderAssembler::toResponse);
        
        return Result.success(response);
    }
}
```

#### 2. 应用层

```java
/**
 * 订单应用服务
 * @author erik.zhou
 */
@Service
public class OrderApplicationService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private OrderFactory orderFactory;
    
    @Autowired
    private ProductServiceAdapter productServiceAdapter;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private DomainEventPublisher eventPublisher;
    
    /**
     * 创建订单
     */
    @Transactional
    public OrderId createOrder(CreateOrderCommand command) {
        // 1. 获取商品信息
        List<ProductId> productIds = command.getItems().stream()
            .map(OrderItemRequest::getProductId)
            .collect(Collectors.toList());
        List<Product> products = productServiceAdapter.getProducts(productIds);
        
        // 2. 检查库存
        boolean hasInventory = inventoryService.checkInventory(
            command.getItems(), products);
        if (!hasInventory) {
            throw new ApplicationException("库存不足");
        }
        
        // 3. 创建订单
        Order order = orderFactory.createOrder(
            command.getUserId(),
            command.getItems(),
            command.getShippingAddress(),
            command.getCoupon()
        );
        
        // 4. 保存订单
        orderRepository.save(order);
        
        return order.getId();
    }
    
    /**
     * 支付订单
     */
    @Transactional
    public void payOrder(PayOrderCommand command) {
        // 1. 查询订单
        Order order = orderRepository.findById(command.getOrderId())
            .orElseThrow(() -> new ApplicationException("订单不存在"));
        
        // 2. 支付订单
        order.pay(command.getPaymentMethod());
        
        // 3. 保存订单
        orderRepository.save(order);
    }
    
    /**
     * 取消订单
     */
    @Transactional
    public void cancelOrder(CancelOrderCommand command) {
        // 1. 查询订单
        Order order = orderRepository.findById(command.getOrderId())
            .orElseThrow(() -> new ApplicationException("订单不存在"));
        
        // 2. 取消订单
        order.cancel(command.getReason());
        
        // 3. 保存订单
        orderRepository.save(order);
    }
    
    /**
     * 查询订单
     */
    public Order getOrder(OrderId orderId) {
        return orderRepository.findById(orderId)
            .orElseThrow(() -> new ApplicationException("订单不存在"));
    }
    
    /**
     * 查询用户订单列表
     */
    public PageResult<Order> getUserOrders(UserId userId, int page, int size) {
        List<Order> orders = orderRepository.findByUserId(userId, page, size);
        long total = orderRepository.countByUserId(userId);
        
        return new PageResult<>(orders, total, page, size);
    }
}
```


#### 3. 领域层（核心）

```java
/**
 * 订单聚合根
 * @author erik.zhou
 */
@Entity
@AggregateRoot
public class Order extends BaseEntity {
    
    private OrderId id;
    private String orderNo;
    private UserId userId;
    private OrderStatus status;
    private Money totalAmount;
    private Address shippingAddress;
    private List<OrderItem> items;
    private LocalDateTime createTime;
    private LocalDateTime payTime;
    
    // 领域事件列表
    private List<DomainEvent> domainEvents = new ArrayList<>();
    
    /**
     * 私有构造函数
     */
    private Order() {}
    
    /**
     * 创建订单（工厂方法）
     */
    public static Order create(UserId userId, List<OrderItem> items) {
        // 业务规则校验
        if (items == null || items.isEmpty()) {
            throw new DomainException("订单项不能为空");
        }
        
        Order order = new Order();
        order.id = OrderId.generate();
        order.orderNo = generateOrderNo();
        order.userId = userId;
        order.items = new ArrayList<>(items);
        order.status = OrderStatus.CREATED;
        order.totalAmount = calculateTotalAmount(items);
        order.createTime = LocalDateTime.now();
        
        // 发布领域事件
        order.addDomainEvent(new OrderCreatedEvent(
            order.id, order.userId, order.totalAmount));
        
        return order;
    }
    
    /**
     * 支付订单
     */
    public void pay(PaymentMethod paymentMethod) {
        // 业务规则校验
        if (this.status != OrderStatus.CREATED) {
            throw new DomainException("订单状态不正确，无法支付");
        }
        
        // 状态变更
        this.status = OrderStatus.PAID;
        this.payTime = LocalDateTime.now();
        
        // 发布领域事件
        this.addDomainEvent(new OrderPaidEvent(
            this.id, paymentMethod, this.totalAmount));
    }
    
    /**
     * 取消订单
     */
    public void cancel(String reason) {
        // 业务规则校验
        if (this.status == OrderStatus.COMPLETED) {
            throw new DomainException("订单已完成，无法取消");
        }
        
        if (this.status == OrderStatus.CANCELLED) {
            throw new DomainException("订单已取消");
        }
        
        // 状态变更
        OrderStatus oldStatus = this.status;
        this.status = OrderStatus.CANCELLED;
        
        // 发布领域事件
        this.addDomainEvent(new OrderCancelledEvent(
            this.id, oldStatus, reason));
    }
    
    /**
     * 添加订单项
     */
    public void addItem(ProductId productId, int quantity, Money price) {
        if (this.status != OrderStatus.CREATED) {
            throw new DomainException("订单状态不正确，无法添加商品");
        }
        
        // 检查是否已存在
        Optional<OrderItem> existingItem = items.stream()
            .filter(item -> item.getProductId().equals(productId))
            .findFirst();
        
        if (existingItem.isPresent()) {
            existingItem.get().increaseQuantity(quantity);
        } else {
            OrderItem newItem = OrderItem.create(productId, quantity, price);
            items.add(newItem);
        }
        
        // 重新计算总金额
        recalculateTotalAmount();
    }
    
    /**
     * 移除订单项
     */
    public void removeItem(OrderItemId itemId) {
        if (this.status != OrderStatus.CREATED) {
            throw new DomainException("订单状态不正确，无法移除商品");
        }
        
        items.removeIf(item -> item.getId().equals(itemId));
        recalculateTotalAmount();
    }
    
    /**
     * 修改收货地址
     */
    public void changeShippingAddress(Address newAddress) {
        if (this.status != OrderStatus.CREATED) {
            throw new DomainException("订单状态不正确，无法修改地址");
        }
        
        this.shippingAddress = newAddress;
    }
    
    /**
     * 设置总金额
     */
    public void setTotalAmount(Money amount) {
        this.totalAmount = amount;
    }
    
    /**
     * 重新计算总金额
     */
    private void recalculateTotalAmount() {
        this.totalAmount = items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
    
    /**
     * 计算总金额
     */
    private static Money calculateTotalAmount(List<OrderItem> items) {
        return items.stream()
            .map(OrderItem::getSubtotal)
            .reduce(Money.ZERO, Money::add);
    }
    
    /**
     * 生成订单号
     */
    private static String generateOrderNo() {
        return "ORD" + System.currentTimeMillis() + 
               RandomStringUtils.randomNumeric(6);
    }
    
    /**
     * 添加领域事件
     */
    protected void addDomainEvent(DomainEvent event) {
        this.domainEvents.add(event);
    }
    
    /**
     * 获取领域事件
     */
    public List<DomainEvent> getDomainEvents() {
        return Collections.unmodifiableList(domainEvents);
    }
    
    /**
     * 清空领域事件
     */
    public void clearDomainEvents() {
        this.domainEvents.clear();
    }
    
    // Getter方法
    public OrderId getId() { return id; }
    public String getOrderNo() { return orderNo; }
    public UserId getUserId() { return userId; }
    public OrderStatus getStatus() { return status; }
    public Money getTotalAmount() { return totalAmount; }
    public Address getShippingAddress() { return shippingAddress; }
    public List<OrderItem> getItems() { 
        return Collections.unmodifiableList(items); 
    }
    public LocalDateTime getCreateTime() { return createTime; }
    public LocalDateTime getPayTime() { return payTime; }
}

/**
 * 订单状态枚举
 * @author erik.zhou
 */
public enum OrderStatus {
    CREATED("已创建"),
    PAID("已支付"),
    SHIPPED("已发货"),
    COMPLETED("已完成"),
    CANCELLED("已取消");
    
    private final String description;
    
    OrderStatus(String description) {
        this.description = description;
    }
    
    public String getDescription() {
        return description;
    }
}
```

#### 4. 基础设施层

```java
/**
 * 订单仓储实现
 * @author erik.zhou
 */
@Repository
public class OrderRepositoryImpl implements OrderRepository {
    
    @Autowired
    private OrderMapper orderMapper;
    
    @Autowired
    private OrderItemMapper orderItemMapper;
    
    @Autowired
    private DomainEventPublisher eventPublisher;
    
    @Override
    @Transactional
    public void save(Order order) {
        // 1. 转换为数据模型
        OrderDO orderDO = toDataObject(order);
        
        // 2. 保存订单
        if (orderMapper.selectById(orderDO.getId()) == null) {
            orderMapper.insert(orderDO);
        } else {
            orderMapper.updateById(orderDO);
        }
        
        // 3. 删除旧的订单项
        orderItemMapper.deleteByOrderId(orderDO.getId());
        
        // 4. 保存新的订单项
        List<OrderItemDO> itemDOs = toItemDataObjects(order);
        if (!itemDOs.isEmpty()) {
            orderItemMapper.batchInsert(itemDOs);
        }
        
        // 5. 发布领域事件
        eventPublisher.publishAll(order.getDomainEvents());
        order.clearDomainEvents();
    }
    
    @Override
    public Optional<Order> findById(OrderId orderId) {
        OrderDO orderDO = orderMapper.selectById(orderId.getValue());
        if (orderDO == null) {
            return Optional.empty();
        }
        
        List<OrderItemDO> itemDOs = orderItemMapper
            .selectByOrderId(orderId.getValue());
        
        Order order = toDomainObject(orderDO, itemDOs);
        return Optional.of(order);
    }
    
    @Override
    public Optional<Order> findByOrderNo(String orderNo) {
        OrderDO orderDO = orderMapper.selectByOrderNo(orderNo);
        if (orderDO == null) {
            return Optional.empty();
        }
        
        List<OrderItemDO> itemDOs = orderItemMapper
            .selectByOrderId(orderDO.getId());
        
        Order order = toDomainObject(orderDO, itemDOs);
        return Optional.of(order);
    }
    
    @Override
    public List<Order> findByUserId(UserId userId, int page, int size) {
        int offset = (page - 1) * size;
        List<OrderDO> orderDOs = orderMapper.selectByUserId(
            userId.getValue(), offset, size);
        
        return orderDOs.stream()
            .map(orderDO -> {
                List<OrderItemDO> itemDOs = orderItemMapper
                    .selectByOrderId(orderDO.getId());
                return toDomainObject(orderDO, itemDOs);
            })
            .collect(Collectors.toList());
    }
    
    @Override
    public long countByUserId(UserId userId) {
        return orderMapper.countByUserId(userId.getValue());
    }
    
    /**
     * 转换为数据对象
     */
    private OrderDO toDataObject(Order order) {
        OrderDO orderDO = new OrderDO();
        orderDO.setId(order.getId().getValue());
        orderDO.setOrderNo(order.getOrderNo());
        orderDO.setUserId(order.getUserId().getValue());
        orderDO.setStatus(order.getStatus().name());
        orderDO.setTotalAmount(order.getTotalAmount().getAmount());
        orderDO.setCurrency(order.getTotalAmount().getCurrency().name());
        
        if (order.getShippingAddress() != null) {
            orderDO.setProvince(order.getShippingAddress().getProvince());
            orderDO.setCity(order.getShippingAddress().getCity());
            orderDO.setDistrict(order.getShippingAddress().getDistrict());
            orderDO.setStreet(order.getShippingAddress().getStreet());
            orderDO.setDetail(order.getShippingAddress().getDetail());
            orderDO.setZipCode(order.getShippingAddress().getZipCode());
        }
        
        orderDO.setCreateTime(order.getCreateTime());
        orderDO.setPayTime(order.getPayTime());
        
        return orderDO;
    }
    
    /**
     * 转换订单项为数据对象
     */
    private List<OrderItemDO> toItemDataObjects(Order order) {
        return order.getItems().stream()
            .map(item -> {
                OrderItemDO itemDO = new OrderItemDO();
                itemDO.setId(item.getId().getValue());
                itemDO.setOrderId(order.getId().getValue());
                itemDO.setProductId(item.getProductId().getValue());
                itemDO.setQuantity(item.getQuantity());
                itemDO.setPrice(item.getPrice().getAmount());
                itemDO.setSubtotal(item.getSubtotal().getAmount());
                return itemDO;
            })
            .collect(Collectors.toList());
    }
    
    /**
     * 转换为领域对象
     */
    private Order toDomainObject(OrderDO orderDO, List<OrderItemDO> itemDOs) {
        // 使用反射或Builder模式重建领域对象
        // 这里简化处理
        Order order = new Order();
        // 设置字段...
        return order;
    }
}
```

---

## 🎓 DDD最佳实践

### 1. 统一语言 (Ubiquitous Language)

**原则**：
- 业务和技术使用相同的术语
- 术语在代码中体现
- 避免技术术语污染业务

**示例**：

```java
// ❌ 不好的命名（技术术语）
public class OrderDO {
    private String id;
    private String userId;
    private int status;  // 0-创建, 1-支付, 2-完成
}

// ✅ 好的命名（业务术语）
public class Order {
    private OrderId id;
    private UserId userId;
    private OrderStatus status;  // CREATED, PAID, COMPLETED
}

// ❌ 不好的方法名
public void updateStatus(int newStatus) {
    this.status = newStatus;
}

// ✅ 好的方法名（体现业务意图）
public void pay(PaymentMethod paymentMethod) {
    if (this.status != OrderStatus.CREATED) {
        throw new DomainException("订单状态不正确，无法支付");
    }
    this.status = OrderStatus.PAID;
}
```

### 2. 充血模型 vs 贫血模型

**贫血模型**（反模式）：
```java
// 贫血模型：只有数据，没有行为
public class Order {
    private String id;
    private String userId;
    private int status;
    
    // 只有getter/setter
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    // ...
}

// 业务逻辑在Service中
public class OrderService {
    public void payOrder(Order order) {
        if (order.getStatus() != 0) {
            throw new Exception("状态不正确");
        }
        order.setStatus(1);
    }
}
```

**充血模型**（DDD推荐）：
```java
// 充血模型：数据 + 行为
public class Order {
    private OrderId id;
    private UserId userId;
    private OrderStatus status;
    
    // 业务行为
    public void pay(PaymentMethod paymentMethod) {
        // 业务规则校验
        if (this.status != OrderStatus.CREATED) {
            throw new DomainException("订单状态不正确，无法支付");
        }
        
        // 状态变更
        this.status = OrderStatus.PAID;
        
        // 发布领域事件
        this.addDomainEvent(new OrderPaidEvent(this.id, paymentMethod));
    }
    
    // 只提供必要的getter
    public OrderId getId() { return id; }
    public OrderStatus getStatus() { return status; }
}
```

### 3. 聚合设计原则

**原则**：
1. **聚合要小** - 只包含必要的对象
2. **通过ID引用** - 聚合之间通过ID引用，不持有对象引用
3. **一个事务一个聚合** - 一个事务只修改一个聚合
4. **最终一致性** - 聚合之间通过事件实现最终一致性

```java
// ❌ 不好的设计：聚合过大
public class Order {
    private OrderId id;
    private User user;  // 持有User对象
    private List<Product> products;  // 持有Product对象
    private Payment payment;  // 持有Payment对象
    // ...
}

// ✅ 好的设计：聚合小，通过ID引用
public class Order {
    private OrderId id;
    private UserId userId;  // 只持有ID
    private List<OrderItem> items;  // 聚合内部实体
    // ...
}

public class OrderItem {
    private OrderItemId id;
    private ProductId productId;  // 只持有ID
    private int quantity;
    private Money price;
}
```

### 4. 领域事件使用

**原则**：
- 用于聚合之间的解耦
- 实现最终一致性
- 记录重要的业务事件

```java
/**
 * 使用领域事件实现最终一致性
 * @author erik.zhou
 */

// 1. 订单支付后发布事件
public class Order {
    public void pay(PaymentMethod paymentMethod) {
        this.status = OrderStatus.PAID;
        
        // 发布领域事件
        this.addDomainEvent(new OrderPaidEvent(this.id, this.totalAmount));
    }
}

// 2. 监听事件，扣减库存
@Component
public class InventoryEventListener {
    
    @Autowired
    private InventoryService inventoryService;
    
    @EventListener
    @Transactional
    public void handleOrderPaid(OrderPaidEvent event) {
        // 扣减库存
        inventoryService.deductInventory(event.getOrderId());
    }
}

// 3. 监听事件，发送通知
@Component
public class NotificationEventListener {
    
    @Autowired
    private NotificationService notificationService;
    
    @EventListener
    @Async
    public void handleOrderPaid(OrderPaidEvent event) {
        // 发送支付成功通知
        notificationService.sendPaymentSuccessNotification(event);
    }
}
```

### 5. 仓储模式

**原则**：
- 面向聚合根
- 隐藏持久化细节
- 使用领域语言

```java
// ✅ 好的仓储接口
public interface OrderRepository {
    void save(Order order);
    Optional<Order> findById(OrderId orderId);
    Optional<Order> findByOrderNo(String orderNo);
    List<Order> findPendingOrders(LocalDateTime before);
    List<Order> findByUserId(UserId userId, int page, int size);
}

// ❌ 不好的仓储接口（暴露了持久化细节）
public interface OrderRepository {
    void insert(OrderDO orderDO);
    void update(OrderDO orderDO);
    OrderDO selectById(String id);
    List<OrderDO> selectByCondition(Map<String, Object> params);
}
```

### 6. 防腐层使用

**原则**：
- 隔离外部系统
- 保护领域模型
- 转换外部模型

```java
/**
 * 防腐层：隔离外部服务
 * @author erik.zhou
 */
@Component
public class PaymentServiceAdapter {
    
    @Autowired
    private PaymentServiceClient paymentServiceClient;
    
    /**
     * 创建支付
     * 将外部模型转换为内部模型
     */
    public Payment createPayment(Order order, PaymentMethod method) {
        // 1. 构建外部请求
        CreatePaymentRequest request = new CreatePaymentRequest();
        request.setOrderNo(order.getOrderNo());
        request.setAmount(order.getTotalAmount().getAmount());
        request.setPaymentMethod(method.name());
        
        // 2. 调用外部服务
        CreatePaymentResponse response = paymentServiceClient.createPayment(request);
        
        // 3. 转换为领域模型（防腐层）
        return Payment.builder()
            .id(PaymentId.of(response.getPaymentId()))
            .orderId(order.getId())
            .amount(Money.ofCNY(response.getAmount()))
            .method(method)
            .status(convertPaymentStatus(response.getStatus()))
            .build();
    }
    
    /**
     * 转换支付状态
     * 隔离外部状态码的变化
     */
    private PaymentStatus convertPaymentStatus(String externalStatus) {
        switch (externalStatus) {
            case "PENDING":
                return PaymentStatus.PENDING;
            case "SUCCESS":
                return PaymentStatus.SUCCESS;
            case "FAILED":
                return PaymentStatus.FAILED;
            default:
                throw new IllegalArgumentException("未知的支付状态: " + externalStatus);
        }
    }
}
```

### 7. 值对象不可变性

**原则**：
- 值对象必须不可变
- 通过工厂方法创建
- 修改时返回新对象

```java
/**
 * 不可变的值对象
 * @author erik.zhou
 */
public final class Money {
    
    private final BigDecimal amount;
    private final Currency currency;
    
    // 私有构造函数
    private Money(BigDecimal amount, Currency currency) {
        this.amount = amount;
        this.currency = currency;
    }
    
    // 工厂方法
    public static Money of(BigDecimal amount, Currency currency) {
        return new Money(amount, currency);
    }
    
    // 返回新对象，而不是修改当前对象
    public Money add(Money other) {
        if (!this.currency.equals(other.currency)) {
            throw new IllegalArgumentException("币种不同");
        }
        return new Money(this.amount.add(other.amount), this.currency);
    }
    
    // 只提供getter，没有setter
    public BigDecimal getAmount() {
        return amount;
    }
    
    public Currency getCurrency() {
        return currency;
    }
}
```

---

## 📝 实战案例：电商订单系统

### 业务场景

构建一个电商订单系统，包含以下功能：
1. 创建订单
2. 支付订单
3. 取消订单
4. 订单发货
5. 订单完成

### 领域模型设计

```
订单聚合 (Order Aggregate)
├── Order (聚合根)
│   ├── OrderId
│   ├── UserId
│   ├── OrderStatus
│   ├── Money (总金额)
│   ├── Address (收货地址)
│   └── List<OrderItem>
└── OrderItem (实体)
    ├── OrderItemId
    ├── ProductId
    ├── quantity
    ├── price
    └── subtotal
```

### 完整实现

由于篇幅限制，完整代码已在前面章节展示。

### 关键设计决策

1. **聚合边界**：订单和订单项作为一个聚合，通过订单聚合根统一管理
2. **ID引用**：订单只持有UserId和ProductId，不持有完整对象
3. **领域事件**：使用OrderCreatedEvent、OrderPaidEvent等事件实现解耦
4. **值对象**：Money、Address等使用值对象，保证不可变性
5. **防腐层**：使用Adapter隔离外部服务（商品服务、支付服务）

---

## 📚 总结

### DDD核心价值

1. **业务驱动** - 以业务为核心，而不是技术
2. **模型驱动** - 用领域模型表达业务规则
3. **统一语言** - 减少沟通成本
4. **清晰架构** - 分层清晰，职责明确

### DDD适用场景

**适合使用DDD**：
- 业务复杂度高
- 需求变化频繁
- 长期维护的系统
- 团队规模较大

**不适合使用DDD**：
- 简单的CRUD系统
- 短期项目
- 技术驱动的系统
- 团队规模很小

### 学习建议

1. **理解概念** - 先理解DDD的核心概念和原则
2. **小步实践** - 从小项目开始，逐步应用DDD
3. **持续重构** - DDD是演进式的，需要持续重构
4. **团队协作** - DDD需要业务和技术的紧密协作

---

**作者**: erik.zhou  
**最后更新**: 2024-01-04
