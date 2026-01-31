# Code Review最佳实践

> @author erik.zhou  
> 难度: ⭐⭐⭐  
> 重要性: ⭐⭐⭐⭐⭐

## 📋 目录

- [Code Review价值](#code-review价值)
- [Review流程](#review流程)
- [Review检查清单](#review检查清单)
- [Review技巧](#review技巧)
- [常见问题](#常见问题)

---

## 🎯 Code Review价值

### 为什么要做Code Review

```
1. 提高代码质量
   - 发现潜在bug
   - 改进代码设计
   - 统一代码风格
   
2. 知识分享
   - 团队成员互相学习
   - 新人快速成长
   - 技术经验传承
   
3. 降低风险
   - 减少生产事故
   - 提前发现问题
   - 降低维护成本
   
4. 团队协作
   - 增进团队沟通
   - 建立代码所有权
   - 提升团队凝聚力
```

---

## 🔄 Review流程

### 标准流程

```
1. 开发者提交代码
   ├─ 自测通过
   ├─ 单元测试通过
   ├─ 代码格式化
   └─ 提交MR/PR
   
2. 自动化检查
   ├─ CI/CD构建
   ├─ 单元测试
   ├─ 代码扫描（SonarQube）
   └─ 代码覆盖率检查
   
3. 人工Review
   ├─ Reviewer审查代码
   ├─ 提出修改意见
   ├─ 开发者修改
   └─ 再次Review
   
4. 合并代码
   ├─ Review通过
   ├─ 合并到主分支
   └─ 部署到测试环境
```

### GitLab MR流程

```bash
# 1. 创建功能分支
git checkout -b feature/user-login

# 2. 开发并提交
git add .
git commit -m "feat: 实现用户登录功能"
git push origin feature/user-login

# 3. 创建Merge Request
# 在GitLab界面创建MR
# 填写MR描述，关联Issue

# 4. 等待Review
# Reviewer审查代码，提出意见

# 5. 修改代码
git add .
git commit -m "fix: 修复登录逻辑问题"
git push origin feature/user-login

# 6. 合并代码
# Review通过后，合并到主分支
```

---

## ✅ Review检查清单

### 1. 功能正确性

```java
/**
 * 功能正确性检查
 * @author erik.zhou
 */

// ❌ 错误：逻辑错误
public boolean isAdult(int age) {
    return age > 18;  // 应该是 >= 18
}

// ✅ 正确
public boolean isAdult(int age) {
    return age >= 18;
}

// ❌ 错误：边界条件未处理
public int divide(int a, int b) {
    return a / b;  // b为0时会抛异常
}

// ✅ 正确
public int divide(int a, int b) {
    if (b == 0) {
        throw new IllegalArgumentException("除数不能为0");
    }
    return a / b;
}

// ❌ 错误：空指针未处理
public String getUserName(User user) {
    return user.getName();  // user可能为null
}

// ✅ 正确
public String getUserName(User user) {
    if (user == null) {
        return "匿名用户";
    }
    return user.getName();
}
```

### 2. 代码设计

```java
/**
 * 代码设计检查
 * @author erik.zhou
 */

// ❌ 错误：职责不单一
public class UserService {
    public void register(User user) {
        // 保存用户
        userRepository.save(user);
        
        // 发送邮件
        emailService.sendWelcomeEmail(user.getEmail());
        
        // 发送短信
        smsService.sendWelcomeSms(user.getPhone());
        
        // 记录日志
        logService.log("用户注册: " + user.getUsername());
    }
}

// ✅ 正确：职责分离
public class UserService {
    
    @Autowired
    private ApplicationEventPublisher eventPublisher;
    
    public void register(User user) {
        // 保存用户
        userRepository.save(user);
        
        // 发布事件
        eventPublisher.publishEvent(new UserRegisteredEvent(user));
    }
}

@Component
public class UserEventListener {
    
    @EventListener
    @Async
    public void handleUserRegistered(UserRegisteredEvent event) {
        User user = event.getUser();
        
        // 发送邮件
        emailService.sendWelcomeEmail(user.getEmail());
        
        // 发送短信
        smsService.sendWelcomeSms(user.getPhone());
        
        // 记录日志
        logService.log("用户注册: " + user.getUsername());
    }
}

// ❌ 错误：过度设计
public interface UserFactory {
    User createUser();
}

public class SimpleUserFactory implements UserFactory {
    @Override
    public User createUser() {
        return new User();
    }
}

// ✅ 正确：简单直接
public class User {
    public static User create() {
        return new User();
    }
}
```

### 3. 性能问题

```java
/**
 * 性能问题检查
 * @author erik.zhou
 */

// ❌ 错误：N+1查询
public List<OrderVO> getOrders() {
    List<Order> orders = orderRepository.findAll();
    
    return orders.stream()
        .map(order -> {
            OrderVO vo = new OrderVO();
            vo.setOrderId(order.getId());
            
            // 每个订单都查询一次用户（N+1问题）
            User user = userRepository.findById(order.getUserId()).get();
            vo.setUserName(user.getName());
            
            return vo;
        })
        .collect(Collectors.toList());
}

// ✅ 正确：批量查询
public List<OrderVO> getOrders() {
    List<Order> orders = orderRepository.findAll();
    
    // 批量查询用户
    Set<Long> userIds = orders.stream()
        .map(Order::getUserId)
        .collect(Collectors.toSet());
    
    Map<Long, User> userMap = userRepository.findByIdIn(userIds).stream()
        .collect(Collectors.toMap(User::getId, u -> u));
    
    return orders.stream()
        .map(order -> {
            OrderVO vo = new OrderVO();
            vo.setOrderId(order.getId());
            
            User user = userMap.get(order.getUserId());
            vo.setUserName(user != null ? user.getName() : "");
            
            return vo;
        })
        .collect(Collectors.toList());
}

// ❌ 错误：在循环中操作数据库
public void updateUsers(List<User> users) {
    for (User user : users) {
        userRepository.save(user);  // 每次都访问数据库
    }
}

// ✅ 正确：批量操作
public void updateUsers(List<User> users) {
    userRepository.saveAll(users);  // 批量保存
}
```

### 4. 安全问题

```java
/**
 * 安全问题检查
 * @author erik.zhou
 */

// ❌ 错误：SQL注入
public List<User> searchUsers(String keyword) {
    String sql = "SELECT * FROM user WHERE username LIKE '%" + keyword + "%'";
    return jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(User.class));
}

// ✅ 正确：使用参数化查询
public List<User> searchUsers(String keyword) {
    String sql = "SELECT * FROM user WHERE username LIKE ?";
    return jdbcTemplate.query(sql, new BeanPropertyRowMapper<>(User.class), "%" + keyword + "%");
}

// ❌ 错误：密码明文存储
public void register(User user) {
    user.setPassword(user.getPassword());  // 明文密码
    userRepository.save(user);
}

// ✅ 正确：密码加密
public void register(User user) {
    String encryptedPassword = BCrypt.hashpw(user.getPassword(), BCrypt.gensalt());
    user.setPassword(encryptedPassword);
    userRepository.save(user);
}

// ❌ 错误：敏感信息日志
public void login(String username, String password) {
    log.info("用户登录: username={}, password={}", username, password);  // 密码泄露
}

// ✅ 正确：不记录敏感信息
public void login(String username, String password) {
    log.info("用户登录: username={}", username);
}
```

### 5. 异常处理

```java
/**
 * 异常处理检查
 * @author erik.zhou
 */

// ❌ 错误：吞掉异常
public void processOrder(Order order) {
    try {
        orderService.process(order);
    } catch (Exception e) {
        // 什么都不做
    }
}

// ✅ 正确：正确处理异常
public void processOrder(Order order) {
    try {
        orderService.process(order);
    } catch (Exception e) {
        log.error("订单处理失败: orderId={}", order.getId(), e);
        throw new BusinessException("订单处理失败", e);
    }
}

// ❌ 错误：捕获Exception
public void readFile(String path) {
    try {
        Files.readAllLines(Paths.get(path));
    } catch (Exception e) {  // 太宽泛
        log.error("读取文件失败", e);
    }
}

// ✅ 正确：捕获具体异常
public void readFile(String path) {
    try {
        Files.readAllLines(Paths.get(path));
    } catch (IOException e) {
        log.error("读取文件失败: path={}", path, e);
        throw new BusinessException("文件读取失败", e);
    }
}
```

### 6. 测试覆盖

```java
/**
 * 测试覆盖检查
 * @author erik.zhou
 */

// ✅ 正确：编写单元测试
@Test
public void testCalculateDiscount() {
    // Given
    Order order = new Order();
    order.setTotalAmount(new BigDecimal("100"));
    
    Coupon coupon = new Coupon();
    coupon.setDiscountRate(new BigDecimal("0.1"));
    
    // When
    BigDecimal discount = orderService.calculateDiscount(order, coupon);
    
    // Then
    assertEquals(new BigDecimal("10.00"), discount);
}

// ✅ 正确：测试边界条件
@Test
public void testCalculateDiscount_NullCoupon() {
    Order order = new Order();
    order.setTotalAmount(new BigDecimal("100"));
    
    BigDecimal discount = orderService.calculateDiscount(order, null);
    
    assertEquals(BigDecimal.ZERO, discount);
}

// ✅ 正确：测试异常情况
@Test(expected = IllegalArgumentException.class)
public void testCalculateDiscount_NegativeAmount() {
    Order order = new Order();
    order.setTotalAmount(new BigDecimal("-100"));
    
    orderService.calculateDiscount(order, null);
}
```

---

## 💡 Review技巧

### 1. Reviewer技巧

```
1. 及时Review
   - 收到Review请求后尽快处理
   - 不要让开发者等待太久
   
2. 关注重点
   - 先看整体设计
   - 再看具体实现
   - 最后看代码细节
   
3. 提建设性意见
   - 说明问题所在
   - 给出改进建议
   - 提供示例代码
   
4. 保持友善
   - 对事不对人
   - 使用礼貌用语
   - 肯定好的地方
   
5. 自动化检查
   - 使用工具检查格式
   - 使用工具检查规范
   - 人工关注逻辑
```

### 2. 开发者技巧

```
1. 提交前自查
   - 自己先Review一遍
   - 运行单元测试
   - 检查代码格式
   
2. 小步提交
   - 每次提交不要太大
   - 一个MR只做一件事
   - 便于Review
   
3. 写好描述
   - 说明改动内容
   - 说明改动原因
   - 关联相关Issue
   
4. 积极响应
   - 及时回复Review意见
   - 虚心接受建议
   - 不要争论细节
   
5. 持续改进
   - 从Review中学习
   - 总结常见问题
   - 提升代码质量
```

### 3. Review评论模板

```
# 发现问题
❌ 问题：这里存在空指针风险
💡 建议：添加null检查或使用Optional
📝 示例：
if (user == null) {
    return "匿名用户";
}

# 性能问题
⚠️ 性能：这里存在N+1查询问题
💡 建议：使用批量查询优化
📝 示例：见上面的代码

# 安全问题
🔒 安全：密码不应该明文存储
💡 建议：使用BCrypt加密
📝 示例：见上面的代码

# 代码风格
📐 风格：变量命名不符合规范
💡 建议：使用驼峰命名
📝 示例：userName 而不是 user_name

# 肯定好的地方
✅ 很好：这个设计很优雅
👍 点赞：测试覆盖很全面
```

---

## 🚨 常见问题

### 1. Review太慢

```
问题：
- Review请求积压
- 开发者等待时间长
- 影响开发效率

解决方案：
1. 设置Review时限（如2小时内响应）
2. 轮流担任Reviewer
3. 使用自动化工具减少人工Review
4. 小步提交，减少Review工作量
```

### 2. Review流于形式

```
问题：
- 只看代码格式
- 不关注逻辑问题
- 走过场

解决方案：
1. 制定Review检查清单
2. 定期Review培训
3. Review质量考核
4. 分享Review案例
```

### 3. Review意见不统一

```
问题：
- 不同Reviewer意见不同
- 开发者不知道听谁的
- 浪费时间争论

解决方案：
1. 制定团队代码规范
2. 使用自动化工具统一格式
3. 重大问题团队讨论
4. 建立技术决策机制
```

### 4. Review冲突

```
问题：
- Reviewer和开发者意见不合
- 争论不休
- 影响团队氛围

解决方案：
1. 对事不对人
2. 用数据说话
3. 寻求第三方意见
4. 必要时升级到技术负责人
```

---

## 📝 总结

### 核心要点

1. **及时Review** - 不要让开发者等待
2. **关注重点** - 先看设计，再看实现
3. **建设性意见** - 给出具体建议和示例
4. **保持友善** - 对事不对人
5. **持续改进** - 从Review中学习

### 最佳实践

1. **自动化** - 使用工具自动检查格式和规范
2. **小步提交** - 每次提交不要太大
3. **写好描述** - 说明改动内容和原因
4. **及时响应** - 快速回复Review意见
5. **知识分享** - 通过Review传播最佳实践

---

**作者**: erik.zhou  
**最后更新**: 2024-01-04
