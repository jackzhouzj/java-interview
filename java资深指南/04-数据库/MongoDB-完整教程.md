# MongoDB 完整教程

## 📋 目录
- 技术概述
- 学习目标
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: MongoDB 7.x
- **官方文档**: https://www.mongodb.com/docs/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: JSON基础、NoSQL概念
- **文档来源**: Context7 - MongoDB Documentation
- **更新时间**: 2024-12-31

## 🎯 学习目标
- [ ] 掌握MongoDB文档模型
- [ ] 理解聚合管道(Aggregation Pipeline)
- [ ] 掌握索引和查询优化
- [ ] 了解分片和副本集

## 📖 基础概念

### 1.1 什么是MongoDB
MongoDB是面向文档的NoSQL数据库，以灵活的JSON格式存储数据。支持动态schema、水平扩展和丰富的查询功能。

### 1.2 核心概念
- **文档(Document)**: 基本数据单元，类似关系数据库的行
- **集合(Collection)**: 文档的容器，类似关系数据库的表
- **数据库(Database)**: 集合的容器
- **字段(Field)**: 文档中的键值对
- **_id**: 每个文档的唯一标识符
- **副本集(Replica Set)**: 数据冗余和高可用
- **分片(Sharding)**: 水平扩展

### 1.3 应用场景
- 内容管理系统
- 实时分析
- 物联网数据存储
- 移动应用后端
- 日志和事件数据

## 🔥 核心特性

### 2.1 文档模型 🔥

**CRUD操作**
```javascript
// 插入文档
db.users.insertOne({
    name: "Erik Zhou",
    age: 25,
    email: "erik@example.com",
    tags: ["developer", "java"],
    address: {
        city: "Beijing",
        country: "China"
    }
});

// 批量插入
db.users.insertMany([
    { name: "User1", age: 30 },
    { name: "User2", age: 28 }
]);

// 查询文档
db.users.find({ age: { $gte: 25 } });
db.users.findOne({ name: "Erik Zhou" });

// 更新文档
db.users.updateOne(
    { name: "Erik Zhou" },
    { $set: { age: 26 }, $push: { tags: "mongodb" } }
);

// 删除文档
db.users.deleteOne({ name: "User1" });
db.users.deleteMany({ age: { $lt: 20 } });
```

**查询操作符**
```javascript
// 比较操作符
db.products.find({ price: { $gt: 100, $lt: 500 } });
db.products.find({ category: { $in: ["electronics", "books"] } });

// 逻辑操作符
db.products.find({
    $or: [
        { price: { $lt: 50 } },
        { category: "sale" }
    ]
});

// 数组操作符
db.users.find({ tags: { $all: ["java", "mongodb"] } });
db.users.find({ "tags.0": "developer" });  // 数组第一个元素

// 嵌套文档查询
db.users.find({ "address.city": "Beijing" });
```


### 2.2 聚合管道 (⚠️ 难点) 🔥

聚合管道是MongoDB处理数据的首选方法，通过一系列阶段转换文档。

**基础聚合**
```javascript
// 统计每个分类的商品数量
db.products.aggregate([
    { $group: { 
        _id: "$category", 
        count: { $sum: 1 },
        avgPrice: { $avg: "$price" }
    }},
    { $sort: { count: -1 } }
]);

// 多阶段聚合
db.orders.aggregate([
    // 1. 匹配条件
    { $match: { status: "completed" } },
    
    // 2. 关联用户表
    { $lookup: {
        from: "users",
        localField: "userId",
        foreignField: "_id",
        as: "userInfo"
    }},
    
    // 3. 展开数组
    { $unwind: "$userInfo" },
    
    // 4. 分组统计
    { $group: {
        _id: "$userInfo.city",
        totalAmount: { $sum: "$amount" },
        orderCount: { $sum: 1 }
    }},
    
    // 5. 排序
    { $sort: { totalAmount: -1 } },
    
    // 6. 限制结果
    { $limit: 10 }
]);
```

**常用聚合操作符**
```javascript
// $project: 字段投影
db.users.aggregate([
    { $project: {
        name: 1,
        age: 1,
        isAdult: { $gte: ["$age", 18] }
    }}
]);

// $addFields: 添加字段
db.orders.aggregate([
    { $addFields: {
        totalWithTax: { $multiply: ["$amount", 1.1] }
    }}
]);

// $bucket: 分桶
db.products.aggregate([
    { $bucket: {
        groupBy: "$price",
        boundaries: [0, 100, 500, 1000, 5000],
        default: "Other",
        output: {
            count: { $sum: 1 },
            products: { $push: "$name" }
        }
    }}
]);
```

### 2.3 索引优化 🔥

**创建索引**
```javascript
// 单字段索引
db.users.createIndex({ email: 1 });  // 1升序，-1降序

// 复合索引
db.orders.createIndex({ userId: 1, createTime: -1 });

// 唯一索引
db.users.createIndex({ email: 1 }, { unique: true });

// 部分索引
db.orders.createIndex(
    { status: 1 },
    { partialFilterExpression: { amount: { $gt: 100 } } }
);

// 文本索引（全文搜索）
db.articles.createIndex({ title: "text", content: "text" });

// 地理空间索引
db.locations.createIndex({ location: "2dsphere" });

// TTL索引（自动过期）
db.sessions.createIndex(
    { createdAt: 1 },
    { expireAfterSeconds: 3600 }
);
```

**查看和分析索引**
```javascript
// 查看所有索引
db.users.getIndexes();

// 查看执行计划
db.users.find({ email: "erik@example.com" }).explain("executionStats");

// 删除索引
db.users.dropIndex("email_1");
```

### 2.4 分片策略 (⚠️ 难点)

**分片键选择**
```javascript
// 启用分片
sh.enableSharding("mydb");

// 基于哈希的分片（数据分布均匀）
sh.shardCollection("mydb.users", { _id: "hashed" });

// 基于范围的分片（适合范围查询）
sh.shardCollection("mydb.orders", { userId: 1, createTime: 1 });

// 查看分片状态
sh.status();
```

**分片键选择原则**
1. 高基数（Cardinality）：值的种类要多
2. 低频率（Frequency）：避免热点数据
3. 单调性（Monotonicity）：避免单调递增的键


## 💻 实战应用

### 3.1 环境搭建

**Docker安装MongoDB**
```bash
# 拉取MongoDB镜像
docker pull mongo:7.0

# 启动MongoDB容器
docker run -d \
  --name mongodb7 \
  -p 27017:27017 \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password \
  -v /data/mongodb:/data/db \
  mongo:7.0

# 连接MongoDB
docker exec -it mongodb7 mongosh -u admin -p password
```

### 3.2 Spring Boot集成

**添加依赖**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

**配置文件**
```yaml
spring:
  data:
    mongodb:
      uri: mongodb://admin:password@localhost:27017/mydb?authSource=admin
      # 或分开配置
      host: localhost
      port: 27017
      database: mydb
      username: admin
      password: password
      authentication-database: admin
```

**实体类**
```java
@Document(collection = "users")
public class User {
    @Id
    private String id;
    
    @Indexed(unique = true)
    private String email;
    
    private String name;
    private Integer age;
    private List<String> tags;
    private Address address;
    
    @CreatedDate
    private LocalDateTime createTime;
    
    @LastModifiedDate
    private LocalDateTime updateTime;
    
    // getters and setters
}

@Data
public class Address {
    private String city;
    private String country;
}
```

**Repository接口**
```java
public interface UserRepository extends MongoRepository<User, String> {
    
    // 方法名查询
    User findByEmail(String email);
    List<User> findByAgeGreaterThan(Integer age);
    List<User> findByTagsContaining(String tag);
    
    // @Query注解
    @Query("{ 'address.city': ?0 }")
    List<User> findByCity(String city);
    
    // 聚合查询
    @Aggregation(pipeline = {
        "{ $match: { age: { $gte: ?0 } } }",
        "{ $group: { _id: '$address.city', count: { $sum: 1 } } }",
        "{ $sort: { count: -1 } }"
    })
    List<CityCount> countUsersByCity(Integer minAge);
}
```

### 3.3 进阶案例

**案例1：实现分页查询**
```java
@Service
public class UserService {
    
    @Autowired
    private MongoTemplate mongoTemplate;
    
    public Page<User> findUsers(String city, int page, int size) {
        Query query = new Query();
        
        if (city != null) {
            query.addCriteria(Criteria.where("address.city").is(city));
        }
        
        // 总数
        long total = mongoTemplate.count(query, User.class);
        
        // 分页
        query.with(PageRequest.of(page, size, Sort.by("createTime").descending()));
        List<User> users = mongoTemplate.find(query, User.class);
        
        return new PageImpl<>(users, PageRequest.of(page, size), total);
    }
}
```

**案例2：复杂聚合查询**
```java
public List<OrderStatistics> getOrderStatistics() {
    Aggregation aggregation = Aggregation.newAggregation(
        // 匹配已完成订单
        Aggregation.match(Criteria.where("status").is("completed")),
        
        // 关联用户表
        Aggregation.lookup("users", "userId", "_id", "userInfo"),
        Aggregation.unwind("userInfo"),
        
        // 分组统计
        Aggregation.group("userInfo.city")
            .sum("amount").as("totalAmount")
            .count().as("orderCount")
            .avg("amount").as("avgAmount"),
        
        // 排序
        Aggregation.sort(Sort.Direction.DESC, "totalAmount"),
        
        // 限制结果
        Aggregation.limit(10)
    );
    
    return mongoTemplate.aggregate(
        aggregation, "orders", OrderStatistics.class
    ).getMappedResults();
}
```

## ✨ 最佳实践

### 4.1 Schema设计
- 嵌入式文档 vs 引用：一对少用嵌入，一对多用引用
- 避免过深的嵌套（建议不超过3层）
- 合理使用数组（避免无限增长）
- 考虑查询模式设计schema

### 4.2 性能优化
- 为常用查询字段创建索引
- 使用投影减少返回字段
- 使用聚合管道代替多次查询
- 避免$where和JavaScript表达式
- 使用explain()分析查询性能

### 4.3 常见陷阱

**⚠️ 陷阱1：N+1查询问题**
```java
// ❌ 错误：循环查询
List<Order> orders = orderRepository.findAll();
for (Order order : orders) {
    User user = userRepository.findById(order.getUserId());
    order.setUser(user);
}

// ✅ 正确：使用聚合或批量查询
Aggregation aggregation = Aggregation.newAggregation(
    Aggregation.lookup("users", "userId", "_id", "user")
);
```

**⚠️ 陷阱2：大数组问题**
```javascript
// ❌ 错误：数组无限增长
db.users.updateOne(
    { _id: userId },
    { $push: { loginHistory: loginRecord } }
);

// ✅ 正确：限制数组大小
db.users.updateOne(
    { _id: userId },
    { 
        $push: { 
            loginHistory: {
                $each: [loginRecord],
                $slice: -100  // 只保留最近100条
            }
        }
    }
);
```

## ❓ 常见问题

### Q1: MongoDB适合什么场景？
A: 
- 需要灵活schema的应用
- 读多写多的场景
- 需要水平扩展的系统
- 存储JSON格式数据
- 不适合：复杂事务、多表关联

### Q2: 如何选择分片键？
A: 
1. 高基数：值的种类要多
2. 低频率：避免热点
3. 非单调：避免单调递增
4. 考虑查询模式

### Q3: MongoDB的事务支持？
A: 
- MongoDB 4.0+支持多文档事务
- 副本集和分片集群都支持
- 性能开销较大，谨慎使用

## 🔗 相关资源
- 官方文档：https://www.mongodb.com/docs/
- MongoDB Compass：图形化管理工具
- MongoDB University：免费在线课程

## 📝 学习检查清单
- [ ] 掌握CRUD操作
- [ ] 理解聚合管道
- [ ] 掌握索引优化
- [ ] 了解分片和副本集
- [ ] 完成Spring Boot集成

---

**@author erik.zhou**  
**文档版本**: v1.0  
**最后更新**: 2024-12-31
