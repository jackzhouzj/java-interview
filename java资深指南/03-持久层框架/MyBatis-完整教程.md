# MyBatis 完整教程

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

- **版本**: 3.5.x
- **官方文档**: https://mybatis.org/mybatis-3/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **学习优先级**: P0
- **预计学习时长**: 20-30小时
- **文档来源**: Context7 + MyBatis官方文档
- **最后更新**: 2024-12-31

### 前置知识
- Java基础语法（集合、泛型、注解、反射）
- JDBC基础知识
- SQL语句编写（SELECT、INSERT、UPDATE、DELETE）
- XML基础语法
- Maven/Gradle构建工具

### 相关技术
- Spring Framework（依赖注入、事务管理）
- MyBatis-Plus（MyBatis增强工具）
- MyBatis Generator（代码生成器）
- PageHelper（分页插件）

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解MyBatis的核心架构和工作原理
- [ ] 掌握XML映射器和注解映射器的使用
- [ ] 熟练编写动态SQL语句
- [ ] 理解并正确使用一级缓存和二级缓存
- [ ] 掌握延迟加载和关联查询
- [ ] 能够开发自定义插件
- [ ] 解决常见的性能问题和陷阱


## 📖 基础概念

### 1.1 什么是MyBatis

MyBatis是一款优秀的**持久层框架**，它支持自定义SQL、存储过程以及高级映射。MyBatis免除了几乎所有的JDBC代码以及设置参数和获取结果集的工作。MyBatis可以通过简单的XML或注解来配置和映射原生类型、接口和Java POJO（Plain Old Java Objects，普通老式Java对象）为数据库中的记录。

**核心特点**：
- **SQL与代码分离**: SQL语句可以写在XML文件中，与Java代码解耦
- **灵活的映射**: 支持复杂的结果集映射
- **动态SQL**: 根据条件动态生成SQL语句
- **缓存机制**: 内置一级缓存和二级缓存
- **插件机制**: 支持自定义插件扩展功能

### 1.2 核心概念

#### SqlSessionFactory
SqlSessionFactory是MyBatis的核心对象，用于创建SqlSession实例。通常在应用启动时创建一次，并在整个应用生命周期中重复使用。

#### SqlSession
SqlSession是执行SQL语句的主要接口，包含了所有执行SQL命令、获取映射器和管理事务的方法。每个线程都应该有自己的SqlSession实例，SqlSession实例不是线程安全的。

#### Mapper接口
Mapper接口是MyBatis的映射器，定义了数据库操作的方法。MyBatis会为Mapper接口生成代理实现类。

#### 映射文件（Mapper XML）
映射文件包含了SQL语句和结果映射配置，与Mapper接口对应。

### 1.3 应用场景

MyBatis适用于以下场景：

1. **需要精确控制SQL的项目**: 复杂查询、性能优化要求高
2. **数据库设计已完成的项目**: 表结构固定，需要灵活的SQL
3. **需要动态SQL的场景**: 根据条件动态拼接SQL
4. **遗留系统改造**: 已有数据库设计，需要快速开发
5. **报表查询系统**: 复杂的统计查询和多表关联

**不适用场景**：
- 快速原型开发（推荐MyBatis-Plus）
- 标准化CRUD操作为主（推荐JPA）
- 需要数据库无关性（推荐JPA）


## 🔥 核心特性 (重点)

### 2.1 映射器（Mapper）🔥

MyBatis提供两种方式定义映射器：**XML映射器**和**注解映射器**。

#### XML映射器（推荐）

XML映射器将SQL语句与Java代码分离，便于维护和优化。

**Mapper接口**：
```java
package com.example.mapper;

import com.example.model.User;
import org.apache.ibatis.annotations.Param;
import java.util.List;

/**
 * 用户Mapper接口
 * @author erik.zhou
 */
public interface UserMapper {
    
    /**
     * 根据ID查询用户
     */
    User selectUserById(Integer id);
    
    /**
     * 根据条件查询用户列表
     */
    List<User> searchUsers(@Param("name") String name, 
                          @Param("email") String email, 
                          @Param("status") String status);
    
    /**
     * 插入用户
     */
    int insertUser(User user);
    
    /**
     * 批量插入用户
     */
    int insertUserBatch(List<User> users);
    
    /**
     * 更新用户
     */
    int updateUser(User user);
    
    /**
     * 删除用户
     */
    int deleteUser(Integer id);
}
```

**XML映射文件**：
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.UserMapper">

    <!-- 可复用的结果映射 -->
    <resultMap id="userResultMap" type="com.example.model.User">
        <id property="id" column="id"/>
        <result property="name" column="name"/>
        <result property="email" column="email"/>
        <result property="createdAt" column="created_at"/>
        <!-- 一对一关联 -->
        <association property="profile" column="id"
                     select="selectProfileByUserId" fetchType="lazy"/>
        <!-- 一对多关联 -->
        <collection property="orders" column="id"
                    select="selectOrdersByUserId" fetchType="lazy"/>
    </resultMap>

    <!-- 简单查询 -->
    <select id="selectUserById" parameterType="int" resultMap="userResultMap">
        SELECT id, name, email, created_at
        FROM users
        WHERE id = #{id}
    </select>

    <!-- 插入并返回主键 -->
    <insert id="insertUser" parameterType="User"
            useGeneratedKeys="true" keyProperty="id" keyColumn="id">
        INSERT INTO users (name, email, created_at)
        VALUES (#{name}, #{email}, #{createdAt})
    </insert>

    <!-- 更新 -->
    <update id="updateUser" parameterType="User">
        UPDATE users
        SET name = #{name}, email = #{email}, updated_at = NOW()
        WHERE id = #{id}
    </update>

    <!-- 删除 -->
    <delete id="deleteUser" parameterType="int">
        DELETE FROM users WHERE id = #{id}
    </delete>

</mapper>
```


#### 注解映射器

注解映射器适用于简单的SQL语句，代码更简洁。

```java
@Mapper
public interface UserMapper {
    
    // 简单查询
    @Select("SELECT id, name, email, created_at FROM users WHERE id = #{id}")
    User selectUserById(int id);

    // 多参数查询
    @Select("SELECT * FROM users WHERE name = #{name} AND status = #{status}")
    List<User> selectUsersByNameAndStatus(
        @Param("name") String name,
        @Param("status") String status
    );

    // 动态SQL（使用script标签）
    @Select({
        "<script>",
        "SELECT * FROM users",
        "<where>",
        "  <if test='name != null'>AND name LIKE #{name}</if>",
        "  <if test='email != null'>AND email = #{email}</if>",
        "  <if test='status != null'>AND status = #{status}</if>",
        "</where>",
        "ORDER BY created_at DESC",
        "</script>"
    })
    List<User> searchUsers(
        @Param("name") String name,
        @Param("email") String email,
        @Param("status") String status
    );

    // 复杂结果映射
    @Select("SELECT * FROM users WHERE id = #{id}")
    @Results(id = "userResultMap", value = {
        @Result(property = "id", column = "id", id = true),
        @Result(property = "name", column = "name"),
        @Result(property = "profile", column = "id",
                one = @One(select = "selectProfileByUserId", fetchType = FetchType.LAZY)),
        @Result(property = "orders", column = "id",
                many = @Many(select = "selectOrdersByUserId", fetchType = FetchType.LAZY))
    })
    User selectUserWithDetails(int id);
}
```

**选择建议**：
- ✅ **简单CRUD**: 使用注解映射器
- ✅ **复杂SQL**: 使用XML映射器
- ✅ **动态SQL**: 优先使用XML映射器
- ✅ **团队协作**: 统一使用XML映射器（便于维护）

### 2.2 动态SQL 🔥

动态SQL是MyBatis的强大特性，可以根据条件动态生成SQL语句。

#### if标签
```xml
<select id="searchUsers" resultType="User">
    SELECT * FROM users
    <where>
        <if test="name != null and name != ''">
            AND name LIKE CONCAT('%', #{name}, '%')
        </if>
        <if test="email != null">
            AND email = #{email}
        </if>
        <if test="status != null">
            AND status = #{status}
        </if>
    </where>
    ORDER BY created_at DESC
</select>
```

#### choose/when/otherwise标签
```xml
<select id="findUsers" resultType="User">
    SELECT * FROM users
    WHERE active = true
    <choose>
        <when test="orderBy == 'name'">
            ORDER BY name
        </when>
        <when test="orderBy == 'date'">
            ORDER BY created_at DESC
        </when>
        <otherwise>
            ORDER BY id
        </otherwise>
    </choose>
</select>
```

#### foreach标签（批量操作）
```xml
<!-- 批量插入 -->
<insert id="insertUserBatch" parameterType="list">
    INSERT INTO users (name, email) VALUES
    <foreach collection="list" item="user" separator=",">
        (#{user.name}, #{user.email})
    </foreach>
</insert>

<!-- IN查询 -->
<select id="selectUsersByIds" resultType="User">
    SELECT * FROM users
    WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

#### set标签（动态更新）
```xml
<update id="updateUser" parameterType="User">
    UPDATE users
    <set>
        <if test="name != null">name = #{name},</if>
        <if test="email != null">email = #{email},</if>
        updated_at = NOW()
    </set>
    WHERE id = #{id}
</update>
```

#### trim标签（自定义前缀后缀）
```xml
<select id="searchUsers" resultType="User">
    SELECT * FROM users
    <trim prefix="WHERE" prefixOverrides="AND |OR ">
        <if test="name != null">
            AND name = #{name}
        </if>
        <if test="email != null">
            AND email = #{email}
        </if>
    </trim>
</select>
```


### 2.3 缓存机制 (⚠️ 难点)

MyBatis提供了两级缓存机制来提升查询性能。

#### 一级缓存（默认开启）

一级缓存是**SqlSession级别**的缓存，在同一个SqlSession中，相同的查询会从缓存中获取结果。

**特点**：
- 默认开启，无法关闭
- 作用域：SqlSession
- 生命周期：SqlSession创建到关闭
- 清除时机：执行INSERT、UPDATE、DELETE操作或手动调用clearCache()

**示例**：
```java
// 第一次查询，从数据库获取
User user1 = sqlSession.selectOne("selectUserById", 1);

// 第二次查询，从一级缓存获取（不会发送SQL）
User user2 = sqlSession.selectOne("selectUserById", 1);

// user1 == user2 为true（同一个对象）

// 执行更新操作后，一级缓存被清空
sqlSession.update("updateUser", user1);

// 第三次查询，重新从数据库获取
User user3 = sqlSession.selectOne("selectUserById", 1);
```

**配置**：
```xml
<settings>
    <!-- 一级缓存作用域：SESSION（默认）或STATEMENT -->
    <setting name="localCacheScope" value="SESSION"/>
</settings>
```

**⚠️ 注意事项**：
1. 不同SqlSession之间缓存不共享
2. 在Spring集成中，每个Mapper方法调用都会创建新的SqlSession，一级缓存基本失效
3. 分布式环境下可能导致数据不一致

#### 二级缓存（需手动开启）

二级缓存是**Mapper级别**的缓存，多个SqlSession可以共享同一个Mapper的缓存。

**开启步骤**：

1. 在mybatis-config.xml中开启全局缓存：
```xml
<settings>
    <setting name="cacheEnabled" value="true"/>
</settings>
```

2. 在Mapper XML中添加cache标签：
```xml
<!-- 使用默认配置 -->
<cache/>

<!-- 自定义配置 -->
<cache
  eviction="FIFO"           <!-- 缓存淘汰策略：LRU、FIFO、SOFT、WEAK -->
  flushInterval="60000"     <!-- 刷新间隔（毫秒） -->
  size="512"                <!-- 缓存对象数量 -->
  readOnly="true"/>         <!-- 只读缓存 -->
```

3. 实体类实现Serializable接口：
```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    // ...
}
```

**注解方式开启**：
```java
@CacheNamespace(
    eviction = FifoCache.class,
    flushInterval = 60000,
    size = 512,
    readWrite = true
)
@Mapper
public interface ProductMapper {
    @Select("SELECT * FROM products WHERE id = #{id}")
    @Options(useCache = true)
    Product selectProduct(int id);
}
```

**缓存淘汰策略**：
- **LRU**（默认）: 最近最少使用，移除最长时间不被使用的对象
- **FIFO**: 先进先出，按对象进入缓存的顺序移除
- **SOFT**: 软引用，基于垃圾回收器状态和软引用规则移除
- **WEAK**: 弱引用，更积极地基于垃圾收集器状态和弱引用规则移除

**⚠️ 二级缓存陷阱**：

1. **脏读问题**：
```java
// Mapper A 查询用户
User user = mapperA.selectUserById(1); // 结果被缓存

// Mapper B 更新用户（不同的Mapper）
mapperB.updateUser(user);

// Mapper A 再次查询，仍然返回缓存的旧数据（脏读）
User oldUser = mapperA.selectUserById(1);
```

2. **分布式环境问题**：
   - 多个应用实例之间缓存不同步
   - 建议使用Redis等分布式缓存替代

3. **关联查询问题**：
   - 多表关联查询的缓存管理复杂
   - 容易出现数据不一致

**最佳实践**：
- ✅ 单表查询可以使用二级缓存
- ✅ 只读数据适合使用二级缓存
- ❌ 多表关联查询慎用二级缓存
- ❌ 分布式环境不建议使用MyBatis二级缓存
- ✅ 推荐使用Redis等外部缓存方案


### 2.4 延迟加载 (⚠️ 难点)

延迟加载（Lazy Loading）是指在需要时才加载关联对象，而不是一次性加载所有数据。

**配置延迟加载**：
```xml
<settings>
    <!-- 开启延迟加载 -->
    <setting name="lazyLoadingEnabled" value="true"/>
    <!-- 关闭积极加载（按需加载） -->
    <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

**使用示例**：
```xml
<resultMap id="userResultMap" type="User">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
    
    <!-- 延迟加载用户资料（一对一） -->
    <association property="profile" column="id"
                 select="selectProfileByUserId" 
                 fetchType="lazy"/>
    
    <!-- 延迟加载订单列表（一对多） -->
    <collection property="orders" column="id"
                select="selectOrdersByUserId" 
                fetchType="lazy"/>
</resultMap>

<select id="selectUserById" resultMap="userResultMap">
    SELECT id, name, email FROM users WHERE id = #{id}
</select>

<select id="selectProfileByUserId" resultType="Profile">
    SELECT * FROM profiles WHERE user_id = #{userId}
</select>

<select id="selectOrdersByUserId" resultType="Order">
    SELECT * FROM orders WHERE user_id = #{userId}
</select>
```

**Java代码**：
```java
// 只查询用户基本信息
User user = userMapper.selectUserById(1);
System.out.println(user.getName()); // 不会触发关联查询

// 访问profile时才触发查询
Profile profile = user.getProfile(); // 此时才执行selectProfileByUserId

// 访问orders时才触发查询
List<Order> orders = user.getOrders(); // 此时才执行selectOrdersByUserId
```

**fetchType属性**：
- `lazy`: 延迟加载
- `eager`: 立即加载

**⚠️ N+1查询问题**：

延迟加载可能导致N+1查询问题：

```java
// 查询10个用户（1次查询）
List<User> users = userMapper.selectAllUsers();

// 遍历用户，每次访问orders都会触发一次查询（N次查询）
for (User user : users) {
    List<Order> orders = user.getOrders(); // 触发N次查询
    // 总共：1 + N = 11次查询
}
```

**解决方案**：

1. **使用嵌套结果映射（推荐）**：
```xml
<resultMap id="userWithOrdersMap" type="User">
    <id property="id" column="user_id"/>
    <result property="name" column="user_name"/>
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
        <result property="orderNo" column="order_no"/>
    </collection>
</resultMap>

<select id="selectUsersWithOrders" resultMap="userWithOrdersMap">
    SELECT 
        u.id as user_id,
        u.name as user_name,
        o.id as order_id,
        o.order_no as order_no
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
</select>
```

2. **关闭延迟加载**：
```xml
<association property="profile" column="id"
             select="selectProfileByUserId" 
             fetchType="eager"/>
```

3. **使用批量查询**：
```java
// 先查询所有用户
List<User> users = userMapper.selectAllUsers();

// 提取所有用户ID
List<Integer> userIds = users.stream()
    .map(User::getId)
    .collect(Collectors.toList());

// 批量查询所有订单
List<Order> allOrders = orderMapper.selectOrdersByUserIds(userIds);

// 手动组装数据
Map<Integer, List<Order>> orderMap = allOrders.stream()
    .collect(Collectors.groupingBy(Order::getUserId));

users.forEach(user -> user.setOrders(orderMap.get(user.getId())));
```

### 2.5 插件机制 🔥

MyBatis允许在SQL执行的特定点进行拦截，实现自定义功能。

**可拦截的方法**：
- `Executor`: update、query、flushStatements、commit、rollback、getTransaction、close、isClosed
- `ParameterHandler`: getParameterObject、setParameters
- `ResultSetHandler`: handleResultSets、handleOutputParameters
- `StatementHandler`: prepare、parameterize、batch、update、query

**自定义插件示例**：
```java
@Intercepts({
    @Signature(
        type = Executor.class,
        method = "update",
        args = {MappedStatement.class, Object.class}
    )
})
public class ExamplePlugin implements Interceptor {
    
    private Properties properties = new Properties();

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        // 前置处理
        System.out.println("执行SQL前的处理");
        
        // 执行原方法
        Object returnObject = invocation.proceed();
        
        // 后置处理
        System.out.println("执行SQL后的处理");
        
        return returnObject;
    }

    @Override
    public void setProperties(Properties properties) {
        this.properties = properties;
    }
}
```

**注册插件**：
```xml
<plugins>
    <plugin interceptor="com.example.plugin.ExamplePlugin">
        <property name="someProperty" value="someValue"/>
    </plugin>
</plugins>
```

**常用插件**：
- **PageHelper**: 分页插件
- **MyBatis-Plus**: 增强插件
- **SQL性能监控插件**: 记录SQL执行时间
- **数据权限插件**: 自动添加数据权限过滤条件


## 💻 实战应用

### 3.1 环境搭建

#### Maven依赖
```xml
<dependencies>
    <!-- MyBatis核心 -->
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.13</version>
    </dependency>
    
    <!-- MySQL驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>
    
    <!-- 连接池（可选） -->
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>1.2.18</version>
    </dependency>
    
    <!-- 日志（可选） -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>2.0.7</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.8</version>
    </dependency>
</dependencies>
```

#### MyBatis配置文件（mybatis-config.xml）
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
  PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    
    <!-- 属性配置 -->
    <properties resource="db.properties"/>
    
    <!-- 全局设置 -->
    <settings>
        <!-- 开启驼峰命名转换 -->
        <setting name="mapUnderscoreToCamelCase" value="true"/>
        <!-- 开启二级缓存 -->
        <setting name="cacheEnabled" value="true"/>
        <!-- 延迟加载 -->
        <setting name="lazyLoadingEnabled" value="true"/>
        <setting name="aggressiveLazyLoading" value="false"/>
        <!-- 日志实现 -->
        <setting name="logImpl" value="SLF4J"/>
    </settings>
    
    <!-- 类型别名 -->
    <typeAliases>
        <package name="com.example.model"/>
    </typeAliases>
    
    <!-- 环境配置 -->
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>
    
    <!-- 映射器 -->
    <mappers>
        <package name="com.example.mapper"/>
    </mappers>
    
</configuration>
```

#### 数据库配置文件（db.properties）
```properties
jdbc.driver=com.mysql.cj.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mydb?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
jdbc.username=root
jdbc.password=123456
```

### 3.2 快速开始

#### 1. 创建实体类
```java
package com.example.model;

import java.io.Serializable;
import java.time.LocalDateTime;

/**
 * 用户实体类
 * @author erik.zhou
 */
public class User implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    private Integer id;
    private String name;
    private String email;
    private Integer status;
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    // Getter和Setter方法
    public Integer getId() {
        return id;
    }
    
    public void setId(Integer id) {
        this.id = id;
    }
    
    public String getName() {
        return name;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
    public String getEmail() {
        return email;
    }
    
    public void setEmail(String email) {
        this.email = email;
    }
    
    public Integer getStatus() {
        return status;
    }
    
    public void setStatus(Integer status) {
        this.status = status;
    }
    
    public LocalDateTime getCreatedAt() {
        return createdAt;
    }
    
    public void setCreatedAt(LocalDateTime createdAt) {
        this.createdAt = createdAt;
    }
    
    public LocalDateTime getUpdatedAt() {
        return updatedAt;
    }
    
    public void setUpdatedAt(LocalDateTime updatedAt) {
        this.updatedAt = updatedAt;
    }
    
    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", status=" + status +
                ", createdAt=" + createdAt +
                ", updatedAt=" + updatedAt +
                '}';
    }
}
```

#### 2. 创建Mapper接口
```java
package com.example.mapper;

import com.example.model.User;
import org.apache.ibatis.annotations.Param;
import java.util.List;

/**
 * 用户Mapper接口
 * @author erik.zhou
 */
public interface UserMapper {
    
    /**
     * 根据ID查询用户
     */
    User selectUserById(Integer id);
    
    /**
     * 查询所有用户
     */
    List<User> selectAllUsers();
    
    /**
     * 插入用户
     */
    int insertUser(User user);
    
    /**
     * 更新用户
     */
    int updateUser(User user);
    
    /**
     * 删除用户
     */
    int deleteUser(Integer id);
}
```


#### 3. 创建Mapper XML文件
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.UserMapper">

    <!-- 查询单个用户 -->
    <select id="selectUserById" parameterType="int" resultType="User">
        SELECT id, name, email, status, created_at, updated_at
        FROM users
        WHERE id = #{id}
    </select>

    <!-- 查询所有用户 -->
    <select id="selectAllUsers" resultType="User">
        SELECT id, name, email, status, created_at, updated_at
        FROM users
        ORDER BY created_at DESC
    </select>

    <!-- 插入用户 -->
    <insert id="insertUser" parameterType="User" 
            useGeneratedKeys="true" keyProperty="id">
        INSERT INTO users (name, email, status, created_at)
        VALUES (#{name}, #{email}, #{status}, NOW())
    </insert>

    <!-- 更新用户 -->
    <update id="updateUser" parameterType="User">
        UPDATE users
        SET name = #{name},
            email = #{email},
            status = #{status},
            updated_at = NOW()
        WHERE id = #{id}
    </update>

    <!-- 删除用户 -->
    <delete id="deleteUser" parameterType="int">
        DELETE FROM users WHERE id = #{id}
    </delete>

</mapper>
```

#### 4. 测试代码
```java
package com.example;

import com.example.mapper.UserMapper;
import com.example.model.User;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;

import java.io.IOException;
import java.io.InputStream;
import java.util.List;

/**
 * MyBatis测试类
 * @author erik.zhou
 */
public class MyBatisTest {
    
    public static void main(String[] args) throws IOException {
        // 1. 加载配置文件
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        
        // 2. 创建SqlSessionFactory
        SqlSessionFactory sqlSessionFactory = 
            new SqlSessionFactoryBuilder().build(inputStream);
        
        // 3. 获取SqlSession
        try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
            
            // 4. 获取Mapper接口
            UserMapper userMapper = sqlSession.getMapper(UserMapper.class);
            
            // 5. 执行查询
            User user = userMapper.selectUserById(1);
            System.out.println("查询用户: " + user);
            
            // 6. 插入数据
            User newUser = new User();
            newUser.setName("张三");
            newUser.setEmail("zhangsan@example.com");
            newUser.setStatus(1);
            
            int rows = userMapper.insertUser(newUser);
            System.out.println("插入成功，影响行数: " + rows);
            System.out.println("生成的ID: " + newUser.getId());
            
            // 7. 提交事务
            sqlSession.commit();
            
            // 8. 查询所有用户
            List<User> users = userMapper.selectAllUsers();
            System.out.println("所有用户: " + users);
            
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

### 3.3 进阶案例

#### 案例1：复杂条件查询
```java
// Mapper接口
List<User> searchUsers(@Param("name") String name,
                      @Param("email") String email,
                      @Param("status") Integer status,
                      @Param("startDate") LocalDateTime startDate,
                      @Param("endDate") LocalDateTime endDate);
```

```xml
<!-- Mapper XML -->
<select id="searchUsers" resultType="User">
    SELECT * FROM users
    <where>
        <if test="name != null and name != ''">
            AND name LIKE CONCAT('%', #{name}, '%')
        </if>
        <if test="email != null and email != ''">
            AND email = #{email}
        </if>
        <if test="status != null">
            AND status = #{status}
        </if>
        <if test="startDate != null">
            AND created_at &gt;= #{startDate}
        </if>
        <if test="endDate != null">
            AND created_at &lt;= #{endDate}
        </if>
    </where>
    ORDER BY created_at DESC
</select>
```

#### 案例2：批量操作
```java
// 批量插入
int insertUserBatch(List<User> users);

// 批量更新
int updateUserBatch(List<User> users);

// 批量删除
int deleteUserBatch(List<Integer> ids);
```

```xml
<!-- 批量插入 -->
<insert id="insertUserBatch" parameterType="list">
    INSERT INTO users (name, email, status, created_at)
    VALUES
    <foreach collection="list" item="user" separator=",">
        (#{user.name}, #{user.email}, #{user.status}, NOW())
    </foreach>
</insert>

<!-- 批量更新（使用CASE WHEN） -->
<update id="updateUserBatch" parameterType="list">
    UPDATE users
    SET name = CASE id
        <foreach collection="list" item="user">
            WHEN #{user.id} THEN #{user.name}
        </foreach>
        END,
        email = CASE id
        <foreach collection="list" item="user">
            WHEN #{user.id} THEN #{user.email}
        </foreach>
        END,
        updated_at = NOW()
    WHERE id IN
    <foreach collection="list" item="user" open="(" separator="," close=")">
        #{user.id}
    </foreach>
</update>

<!-- 批量删除 -->
<delete id="deleteUserBatch" parameterType="list">
    DELETE FROM users
    WHERE id IN
    <foreach collection="list" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</delete>
```

#### 案例3：一对多关联查询
```java
// 用户实体类
public class User {
    private Integer id;
    private String name;
    private List<Order> orders; // 一对多关系
    // getter/setter...
}

// 订单实体类
public class Order {
    private Integer id;
    private String orderNo;
    private Integer userId;
    // getter/setter...
}
```

```xml
<!-- 嵌套结果映射（推荐） -->
<resultMap id="userWithOrdersMap" type="User">
    <id property="id" column="user_id"/>
    <result property="name" column="user_name"/>
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
        <result property="orderNo" column="order_no"/>
        <result property="userId" column="user_id"/>
    </collection>
</resultMap>

<select id="selectUsersWithOrders" resultMap="userWithOrdersMap">
    SELECT 
        u.id as user_id,
        u.name as user_name,
        o.id as order_id,
        o.order_no as order_no
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
    ORDER BY u.id, o.id
</select>
```


## ✨ 最佳实践

### 4.1 性能优化

#### 1. 避免SELECT *
```xml
<!-- ❌ 不推荐 -->
<select id="selectUser" resultType="User">
    SELECT * FROM users WHERE id = #{id}
</select>

<!-- ✅ 推荐 -->
<select id="selectUser" resultType="User">
    SELECT id, name, email, status FROM users WHERE id = #{id}
</select>
```

#### 2. 使用批量操作
```java
// ❌ 不推荐：循环单条插入
for (User user : users) {
    userMapper.insertUser(user);
}

// ✅ 推荐：批量插入
userMapper.insertUserBatch(users);
```

#### 3. 合理使用缓存
```java
// ✅ 单表查询使用二级缓存
@CacheNamespace
public interface UserMapper {
    User selectUserById(Integer id);
}

// ❌ 多表关联查询不建议使用二级缓存
// 容易出现数据不一致
```

#### 4. 避免N+1查询
```xml
<!-- ❌ 不推荐：延迟加载导致N+1查询 -->
<resultMap id="userMap" type="User">
    <collection property="orders" column="id"
                select="selectOrdersByUserId" fetchType="lazy"/>
</resultMap>

<!-- ✅ 推荐：使用JOIN一次查询 -->
<resultMap id="userWithOrdersMap" type="User">
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
        <result property="orderNo" column="order_no"/>
    </collection>
</resultMap>

<select id="selectUsersWithOrders" resultMap="userWithOrdersMap">
    SELECT u.*, o.id as order_id, o.order_no
    FROM users u
    LEFT JOIN orders o ON u.id = o.user_id
</select>
```

#### 5. 使用合适的fetchSize
```xml
<select id="selectLargeData" resultType="User" fetchSize="1000">
    SELECT * FROM users
</select>
```

### 4.2 常见陷阱

#### ⚠️ 陷阱1：参数名称不匹配
```java
// ❌ 错误：参数名不匹配
User selectUser(String userName, Integer userStatus);

// XML中使用#{userName}和#{userStatus}会报错
// 因为MyBatis默认使用arg0、arg1或param1、param2

// ✅ 正确：使用@Param注解
User selectUser(@Param("userName") String userName, 
                @Param("userStatus") Integer userStatus);
```

#### ⚠️ 陷阱2：一级缓存导致的脏读
```java
// 同一个SqlSession中
User user1 = userMapper.selectUserById(1);
System.out.println(user1.getName()); // 输出：张三

// 其他线程更新了数据库
// UPDATE users SET name = '李四' WHERE id = 1

// 再次查询，仍然返回缓存的旧数据
User user2 = userMapper.selectUserById(1);
System.out.println(user2.getName()); // 输出：张三（脏读）

// ✅ 解决方案：清空缓存或使用新的SqlSession
sqlSession.clearCache();
User user3 = userMapper.selectUserById(1);
System.out.println(user3.getName()); // 输出：李四
```

#### ⚠️ 陷阱3：动态SQL中的空字符串判断
```xml
<!-- ❌ 错误：只判断null -->
<if test="name != null">
    AND name = #{name}
</if>

<!-- 当name=""时，会生成：AND name = '' -->

<!-- ✅ 正确：同时判断null和空字符串 -->
<if test="name != null and name != ''">
    AND name = #{name}
</if>
```

#### ⚠️ 陷阱4：大于小于符号的转义
```xml
<!-- ❌ 错误：直接使用<和> -->
<select id="selectUsers" resultType="User">
    SELECT * FROM users WHERE age > 18 AND age < 60
</select>
<!-- XML解析会报错 -->

<!-- ✅ 方案1：使用转义字符 -->
<select id="selectUsers" resultType="User">
    SELECT * FROM users WHERE age &gt; 18 AND age &lt; 60
</select>

<!-- ✅ 方案2：使用CDATA -->
<select id="selectUsers" resultType="User">
    <![CDATA[
        SELECT * FROM users WHERE age > 18 AND age < 60
    ]]>
</select>
```

#### ⚠️ 陷阱5：foreach中的collection属性
```java
// 单个List参数
int insertBatch(List<User> users);

// ✅ collection="list"
<foreach collection="list" item="user">
    ...
</foreach>

// 多个参数
int insertBatch(@Param("users") List<User> users, @Param("status") Integer status);

// ✅ collection="users"
<foreach collection="users" item="user">
    ...
</foreach>
```

### 4.3 代码规范

#### 1. Mapper接口规范
```java
/**
 * 用户Mapper接口
 * 
 * 命名规范：
 * - 查询：select/get/find/query + 实体名 + By条件
 * - 插入：insert + 实体名
 * - 更新：update + 实体名
 * - 删除：delete + 实体名 + By条件
 * 
 * @author erik.zhou
 */
public interface UserMapper {
    
    // ✅ 推荐的命名
    User selectUserById(Integer id);
    List<User> selectUsersByStatus(Integer status);
    int insertUser(User user);
    int updateUser(User user);
    int deleteUserById(Integer id);
    
    // ❌ 不推荐的命名
    User getUser(Integer id);  // 不够明确
    List<User> list();         // 太简单
    int add(User user);        // 不够明确
}
```

#### 2. XML文件规范
```xml
<!-- 1. 使用有意义的id -->
<select id="selectUserById" resultType="User">
    SELECT * FROM users WHERE id = #{id}
</select>

<!-- 2. 复杂SQL使用resultMap -->
<resultMap id="userResultMap" type="User">
    <id property="id" column="id"/>
    <result property="name" column="name"/>
</resultMap>

<!-- 3. 动态SQL使用合适的标签 -->
<select id="searchUsers" resultType="User">
    SELECT * FROM users
    <where>
        <if test="name != null and name != ''">
            AND name LIKE CONCAT('%', #{name}, '%')
        </if>
    </where>
</select>

<!-- 4. 添加注释说明 -->
<!-- 根据多个条件查询用户列表，支持分页 -->
<select id="searchUsers" resultType="User">
    ...
</select>
```

#### 3. 事务管理规范
```java
// ✅ 推荐：使用try-with-resources自动关闭
try (SqlSession sqlSession = sqlSessionFactory.openSession()) {
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    mapper.insertUser(user);
    sqlSession.commit();
} catch (Exception e) {
    // 异常会自动回滚
    e.printStackTrace();
}

// ❌ 不推荐：手动管理
SqlSession sqlSession = sqlSessionFactory.openSession();
try {
    UserMapper mapper = sqlSession.getMapper(UserMapper.class);
    mapper.insertUser(user);
    sqlSession.commit();
} catch (Exception e) {
    sqlSession.rollback();
} finally {
    sqlSession.close();
}
```


## ❓ 常见问题

### Q1: MyBatis和Hibernate/JPA有什么区别？

**A**: 主要区别：

| 特性 | MyBatis | Hibernate/JPA |
|------|---------|---------------|
| SQL控制 | 完全控制，手写SQL | 自动生成SQL |
| 学习曲线 | 相对简单 | 较陡峭 |
| 灵活性 | 高，适合复杂SQL | 中等，标准化 |
| 性能优化 | 容易，直接优化SQL | 较难，需要理解ORM机制 |
| 数据库移植性 | 低，SQL依赖数据库 | 高，HQL/JPQL抽象 |
| 适用场景 | 复杂业务、性能要求高 | 标准CRUD、快速开发 |

**选择建议**：
- 复杂SQL、性能要求高 → MyBatis
- 标准CRUD、快速开发 → JPA
- 团队熟悉SQL → MyBatis
- 需要数据库无关性 → JPA

### Q2: 一级缓存和二级缓存的区别？

**A**: 

| 特性 | 一级缓存 | 二级缓存 |
|------|---------|---------|
| 作用域 | SqlSession级别 | Mapper级别 |
| 默认状态 | 默认开启，无法关闭 | 默认关闭，需手动开启 |
| 共享范围 | 同一个SqlSession | 同一个Mapper的所有SqlSession |
| 清除时机 | SqlSession关闭或执行更新操作 | 执行更新操作或手动清除 |
| 线程安全 | 线程安全（每个线程独立） | 需要配置readOnly或实现Serializable |
| 适用场景 | 同一个请求内的重复查询 | 只读数据、单表查询 |

**最佳实践**：
- 一级缓存：默认使用，注意脏读问题
- 二级缓存：谨慎使用，推荐用Redis替代

### Q3: 如何解决N+1查询问题？

**A**: N+1查询问题是指查询N条记录时，触发了N+1次数据库查询。

**解决方案**：

1. **使用嵌套结果映射（推荐）**：
```xml
<resultMap id="userWithOrdersMap" type="User">
    <id property="id" column="user_id"/>
    <collection property="orders" ofType="Order">
        <id property="id" column="order_id"/>
    </collection>
</resultMap>

<select id="selectUsersWithOrders" resultMap="userWithOrdersMap">
    SELECT u.*, o.id as order_id, o.order_no
    FROM users u LEFT JOIN orders o ON u.id = o.user_id
</select>
```

2. **关闭延迟加载**：
```xml
<association property="profile" fetchType="eager"/>
```

3. **使用批量查询**：
```java
// 先查询主表
List<User> users = userMapper.selectAllUsers();
// 批量查询关联表
List<Order> orders = orderMapper.selectOrdersByUserIds(userIds);
// 手动组装
```

### Q4: #{} 和 ${} 有什么区别？

**A**: 

| 特性 | #{} | ${} |
|------|-----|-----|
| 处理方式 | 预编译，使用PreparedStatement | 字符串替换 |
| SQL注入 | 防止SQL注入 | 存在SQL注入风险 |
| 类型处理 | 自动类型转换 | 不进行类型转换 |
| 使用场景 | 参数值 | 表名、列名、ORDER BY |

**示例**：
```xml
<!-- ✅ 使用#{} -->
<select id="selectUser" resultType="User">
    SELECT * FROM users WHERE id = #{id}
</select>
<!-- 生成：SELECT * FROM users WHERE id = ? -->

<!-- ⚠️ 使用${} -->
<select id="selectUser" resultType="User">
    SELECT * FROM users WHERE id = ${id}
</select>
<!-- 生成：SELECT * FROM users WHERE id = 1 -->
<!-- 存在SQL注入风险！ -->

<!-- ✅ ${} 的合理使用场景 -->
<select id="selectFromTable" resultType="User">
    SELECT * FROM ${tableName} WHERE id = #{id}
</select>
```

### Q5: 如何在MyBatis中实现分页？

**A**: 三种方式：

**方式1：使用LIMIT（MySQL）**
```xml
<select id="selectUsers" resultType="User">
    SELECT * FROM users
    LIMIT #{offset}, #{pageSize}
</select>
```

**方式2：使用RowBounds**
```java
// 不推荐：内存分页，性能差
RowBounds rowBounds = new RowBounds(offset, pageSize);
List<User> users = sqlSession.selectList("selectUsers", null, rowBounds);
```

**方式3：使用PageHelper插件（推荐）**
```xml
<!-- 添加依赖 -->
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>5.3.3</version>
</dependency>
```

```java
// 使用PageHelper
PageHelper.startPage(pageNum, pageSize);
List<User> users = userMapper.selectAllUsers();
PageInfo<User> pageInfo = new PageInfo<>(users);

System.out.println("总记录数: " + pageInfo.getTotal());
System.out.println("总页数: " + pageInfo.getPages());
System.out.println("当前页: " + pageInfo.getPageNum());
```

### Q6: MyBatis如何处理枚举类型？

**A**: 两种方式：

**方式1：使用EnumTypeHandler（默认）**
```java
public enum UserStatus {
    ACTIVE,    // 存储为 "ACTIVE"
    INACTIVE,  // 存储为 "INACTIVE"
    DELETED    // 存储为 "DELETED"
}
```

**方式2：使用EnumOrdinalTypeHandler**
```java
public enum UserStatus {
    ACTIVE,    // 存储为 0
    INACTIVE,  // 存储为 1
    DELETED    // 存储为 2
}
```

**配置**：
```xml
<typeHandlers>
    <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler"
                 javaType="com.example.enums.UserStatus"/>
</typeHandlers>
```

**推荐方式：自定义TypeHandler**
```java
public enum UserStatus {
    ACTIVE(1, "激活"),
    INACTIVE(0, "未激活"),
    DELETED(-1, "已删除");
    
    private final int code;
    private final String desc;
    
    UserStatus(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }
    
    public int getCode() {
        return code;
    }
}

@MappedTypes(UserStatus.class)
public class UserStatusTypeHandler extends BaseTypeHandler<UserStatus> {
    
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, 
                                    UserStatus parameter, JdbcType jdbcType) 
                                    throws SQLException {
        ps.setInt(i, parameter.getCode());
    }
    
    @Override
    public UserStatus getNullableResult(ResultSet rs, String columnName) 
                                       throws SQLException {
        int code = rs.getInt(columnName);
        return getStatusByCode(code);
    }
    
    // 其他方法...
    
    private UserStatus getStatusByCode(int code) {
        for (UserStatus status : UserStatus.values()) {
            if (status.getCode() == code) {
                return status;
            }
        }
        return null;
    }
}
```

## 🔗 相关资源

### 官方资源
- [MyBatis官方文档](https://mybatis.org/mybatis-3/)
- [MyBatis GitHub仓库](https://github.com/mybatis/mybatis-3)
- [MyBatis中文文档](https://mybatis.org/mybatis-3/zh/index.html)

### 推荐插件
- [PageHelper](https://github.com/pagehelper/Mybatis-PageHelper) - 分页插件
- [MyBatis-Plus](https://baomidou.com/) - MyBatis增强工具
- [MyBatis Generator](https://mybatis.org/generator/) - 代码生成器
- [MyBatis Dynamic SQL](https://mybatis.org/mybatis-dynamic-sql/) - 动态SQL构建器

### 学习资源
- [MyBatis从入门到精通](https://book.douban.com/subject/27074809/) - 刘增辉著
- [深入浅出MyBatis技术原理与实战](https://book.douban.com/subject/30250459/) - 杨开振著
- [MyBatis源码分析](https://www.cnblogs.com/xrq730/category/1031349.html) - 博客系列

### 相关技术
- [Spring MyBatis集成](https://mybatis.org/spring/)
- [Spring Boot MyBatis Starter](https://mybatis.org/spring-boot-starter/)
- [MyBatis-Spring-Boot-Starter](https://github.com/mybatis/spring-boot-starter)

## 📝 学习检查清单

完成以下检查项，确保你已掌握MyBatis核心知识：

### 基础知识
- [ ] 理解MyBatis的核心架构（SqlSessionFactory、SqlSession、Mapper）
- [ ] 掌握XML映射器和注解映射器的使用
- [ ] 理解参数传递方式（单参数、多参数、@Param）
- [ ] 掌握结果映射（resultType、resultMap）

### 动态SQL
- [ ] 掌握if标签的使用
- [ ] 掌握choose/when/otherwise标签
- [ ] 掌握foreach标签（批量操作）
- [ ] 掌握set标签（动态更新）
- [ ] 掌握where和trim标签

### 高级特性
- [ ] 理解一级缓存的工作原理和注意事项
- [ ] 理解二级缓存的配置和使用场景
- [ ] 掌握延迟加载的配置和使用
- [ ] 理解N+1查询问题及解决方案
- [ ] 掌握关联查询（一对一、一对多）
- [ ] 了解插件机制和常用插件

### 实战能力
- [ ] 能够独立搭建MyBatis项目
- [ ] 能够编写复杂的动态SQL
- [ ] 能够优化SQL性能
- [ ] 能够解决常见的MyBatis问题
- [ ] 能够与Spring/Spring Boot集成

### 最佳实践
- [ ] 遵循Mapper接口命名规范
- [ ] 避免常见陷阱（参数名、缓存、SQL注入）
- [ ] 合理使用缓存机制
- [ ] 正确处理事务
- [ ] 编写可维护的代码

---

**@author** erik.zhou  
**文档版本**: 1.0  
**最后更新**: 2024-12-31  
**文档来源**: Context7 + MyBatis官方文档
