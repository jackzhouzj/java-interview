# MyBatis-Plus 完整教程

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
- **官方文档**: https://baomidou.com/
- **学习难度**: ⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **学习优先级**: P1
- **预计学习时长**: 15-20小时
- **文档来源**: Context7 + MyBatis-Plus官方文档
- **最后更新**: 2024-12-31

### 前置知识
- MyBatis基础（必须）
- Java基础语法（集合、泛型、Lambda表达式）
- Spring Boot基础
- MySQL数据库

### 相关技术
- MyBatis（基础框架）
- Spring Boot（集成框架）
- MyBatis-Plus Generator（代码生成器）
- Dynamic Datasource（多数据源）

## 🎯 学习目标

完成本教程后，你将能够：

- [ ] 理解MyBatis-Plus的核心架构和设计理念
- [ ] 掌握BaseMapper提供的CRUD接口
- [ ] 熟练使用条件构造器（QueryWrapper、LambdaQueryWrapper）
- [ ] 掌握代码生成器的使用
- [ ] 理解并使用分页插件
- [ ] 掌握性能分析和SQL注入防护
- [ ] 能够快速开发企业级应用


## 📖 基础概念

### 1.1 什么是MyBatis-Plus

MyBatis-Plus（简称MP）是一个**MyBatis的增强工具**，在MyBatis的基础上只做增强不做改变，为简化开发、提高效率而生。

**核心特点**：
- **无侵入**: 只做增强不做改变，引入它不会对现有工程产生影响
- **损耗小**: 启动即会自动注入基本CRUD，性能基本无损耗
- **强大的CRUD操作**: 内置通用Mapper、通用Service，仅需少量配置即可实现单表大部分CRUD操作
- **支持Lambda形式调用**: 通过Lambda表达式，方便地编写各类查询条件
- **支持主键自动生成**: 支持多达4种主键策略
- **支持ActiveRecord模式**: 实体类只需继承Model类即可进行强大的CRUD操作
- **支持自定义全局通用操作**: 支持全局通用方法注入
- **内置代码生成器**: 采用代码或Maven插件可快速生成Mapper、Model、Service、Controller层代码
- **内置分页插件**: 基于MyBatis物理分页，开发者无需关心具体操作
- **内置性能分析插件**: 可输出SQL语句以及其执行时间
- **内置全局拦截插件**: 提供全表delete、update操作智能分析阻断

### 1.2 核心概念

#### BaseMapper
BaseMapper是MyBatis-Plus提供的基础Mapper接口，包含了常用的CRUD方法。只需让自定义Mapper接口继承BaseMapper，即可获得CRUD功能。

#### IService
IService是MyBatis-Plus提供的通用Service接口，包含了常用的业务层方法，支持批量操作。

#### 条件构造器
条件构造器用于构建复杂的查询条件，包括：
- **QueryWrapper**: 普通条件构造器
- **LambdaQueryWrapper**: Lambda条件构造器（类型安全）
- **UpdateWrapper**: 更新条件构造器
- **LambdaUpdateWrapper**: Lambda更新条件构造器

#### 注解
MyBatis-Plus提供了丰富的注解：
- `@TableName`: 表名注解
- `@TableId`: 主键注解
- `@TableField`: 字段注解
- `@Version`: 乐观锁注解
- `@TableLogic`: 逻辑删除注解

### 1.3 应用场景

MyBatis-Plus适用于以下场景：

1. **快速开发**: 单表CRUD操作频繁的项目
2. **标准化项目**: 需要统一CRUD规范的团队项目
3. **微服务项目**: 需要快速构建数据访问层
4. **后台管理系统**: 大量的增删改查操作
5. **原型开发**: 快速验证业务逻辑

**优势场景**：
- ✅ 单表操作为主
- ✅ 标准CRUD操作
- ✅ 需要快速开发
- ✅ 团队协作开发

**不适用场景**：
- ❌ 复杂的多表关联查询（建议使用MyBatis原生XML）
- ❌ 需要精细控制SQL的场景
- ❌ 特殊的数据库操作

## 🔥 核心特性 (重点)

### 2.1 BaseMapper接口 🔥

BaseMapper提供了14个常用的CRUD方法，无需编写XML即可使用。

#### 定义Mapper接口
```java
package com.example.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.example.model.User;
import org.apache.ibatis.annotations.Mapper;

/**
 * 用户Mapper接口
 * 继承BaseMapper即可获得CRUD功能
 * @author erik.zhou
 */
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // 无需编写任何方法，BaseMapper已提供14个常用方法
    // 如需自定义SQL，可以在此添加方法并编写XML
}
```

#### BaseMapper提供的方法

**插入操作**：
```java
// 插入一条记录
int insert(T entity);
```

**删除操作**：
```java
// 根据ID删除
int deleteById(Serializable id);

// 根据条件删除
int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);

// 根据ID批量删除
int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);

// 根据Map条件删除
int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
```

**更新操作**：
```java
// 根据ID更新
int updateById(@Param(Constants.ENTITY) T entity);

// 根据条件更新
int update(@Param(Constants.ENTITY) T entity, 
           @Param(Constants.WRAPPER) Wrapper<T> updateWrapper);
```

**查询操作**：
```java
// 根据ID查询
T selectById(Serializable id);

// 根据ID批量查询
List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);

// 根据Map条件查询
List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);

// 根据条件查询一条记录
T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 根据条件查询记录数
Long selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 根据条件查询列表
List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 根据条件查询Map列表
List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);

// 根据条件分页查询
IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
```

#### 使用示例
```java
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    public void crudExample() {
        // 1. 插入
        User user = new User();
        user.setName("张三");
        user.setEmail("zhangsan@example.com");
        user.setAge(25);
        userMapper.insert(user);
        System.out.println("插入成功，ID: " + user.getId());
        
        // 2. 根据ID查询
        User queryUser = userMapper.selectById(user.getId());
        System.out.println("查询结果: " + queryUser);
        
        // 3. 更新
        user.setAge(26);
        userMapper.updateById(user);
        
        // 4. 批量查询
        List<Long> ids = Arrays.asList(1L, 2L, 3L);
        List<User> users = userMapper.selectBatchIds(ids);
        
        // 5. 条件查询
        Map<String, Object> columnMap = new HashMap<>();
        columnMap.put("name", "张三");
        columnMap.put("age", 26);
        List<User> userList = userMapper.selectByMap(columnMap);
        
        // 6. 删除
        userMapper.deleteById(user.getId());
        
        // 7. 批量删除
        userMapper.deleteBatchIds(ids);
    }
}
```


### 2.2 条件构造器 🔥

条件构造器是MyBatis-Plus最强大的特性之一，用于构建复杂的查询条件。

#### QueryWrapper（普通条件构造器）

```java
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    public void queryWrapperExample() {
        // 1. 基本查询：name = '张三' AND age >= 18
        QueryWrapper<User> wrapper = new QueryWrapper<>();
        wrapper.eq("name", "张三")
               .ge("age", 18);
        List<User> users = userMapper.selectList(wrapper);
        
        // 2. 模糊查询：name LIKE '%张%'
        QueryWrapper<User> wrapper2 = new QueryWrapper<>();
        wrapper2.like("name", "张");
        List<User> users2 = userMapper.selectList(wrapper2);
        
        // 3. 范围查询：age BETWEEN 18 AND 30
        QueryWrapper<User> wrapper3 = new QueryWrapper<>();
        wrapper3.between("age", 18, 30);
        List<User> users3 = userMapper.selectList(wrapper3);
        
        // 4. IN查询：id IN (1, 2, 3)
        QueryWrapper<User> wrapper4 = new QueryWrapper<>();
        wrapper4.in("id", Arrays.asList(1, 2, 3));
        List<User> users4 = userMapper.selectList(wrapper4);
        
        // 5. 排序：ORDER BY age DESC, id ASC
        QueryWrapper<User> wrapper5 = new QueryWrapper<>();
        wrapper5.orderByDesc("age")
                .orderByAsc("id");
        List<User> users5 = userMapper.selectList(wrapper5);
        
        // 6. 分组：GROUP BY age HAVING count(*) > 1
        QueryWrapper<User> wrapper6 = new QueryWrapper<>();
        wrapper6.select("age", "count(*) as count")
                .groupBy("age")
                .having("count(*) > {0}", 1);
        List<Map<String, Object>> maps = userMapper.selectMaps(wrapper6);
        
        // 7. 复杂条件：(name = '张三' OR name = '李四') AND age > 18
        QueryWrapper<User> wrapper7 = new QueryWrapper<>();
        wrapper7.and(w -> w.eq("name", "张三").or().eq("name", "李四"))
                .gt("age", 18);
        List<User> users7 = userMapper.selectList(wrapper7);
    }
}
```

#### LambdaQueryWrapper（Lambda条件构造器，推荐）

LambdaQueryWrapper使用Lambda表达式，避免了字段名硬编码，更加类型安全。

```java
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    public void lambdaQueryWrapperExample() {
        // 1. 基本查询
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getName, "张三")
               .ge(User::getAge, 18);
        List<User> users = userMapper.selectList(wrapper);
        
        // 2. 链式调用（推荐）
        List<User> users2 = userMapper.selectList(
            new LambdaQueryWrapper<User>()
                .eq(User::getName, "张三")
                .ge(User::getAge, 18)
                .orderByDesc(User::getAge)
        );
        
        // 3. 条件判断
        String name = "张三";
        Integer minAge = 18;
        
        LambdaQueryWrapper<User> wrapper3 = new LambdaQueryWrapper<>();
        wrapper3.eq(StringUtils.isNotBlank(name), User::getName, name)
                .ge(minAge != null, User::getAge, minAge);
        List<User> users3 = userMapper.selectList(wrapper3);
        
        // 4. 查询指定字段
        LambdaQueryWrapper<User> wrapper4 = new LambdaQueryWrapper<>();
        wrapper4.select(User::getId, User::getName, User::getAge)
                .eq(User::getStatus, 1);
        List<User> users4 = userMapper.selectList(wrapper4);
        
        // 5. 复杂条件
        LambdaQueryWrapper<User> wrapper5 = new LambdaQueryWrapper<>();
        wrapper5.nested(w -> w.eq(User::getName, "张三")
                              .or()
                              .eq(User::getName, "李四"))
                .gt(User::getAge, 18);
        List<User> users5 = userMapper.selectList(wrapper5);
    }
}
```

#### UpdateWrapper（更新条件构造器）

```java
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    public void updateWrapperExample() {
        // 1. 更新指定字段
        UpdateWrapper<User> wrapper = new UpdateWrapper<>();
        wrapper.set("age", 26)
               .set("email", "new@example.com")
               .eq("id", 1);
        userMapper.update(null, wrapper);
        // 生成SQL: UPDATE user SET age=26, email='new@example.com' WHERE id=1
        
        // 2. Lambda方式更新
        LambdaUpdateWrapper<User> wrapper2 = new LambdaUpdateWrapper<>();
        wrapper2.set(User::getAge, 26)
                .set(User::getEmail, "new@example.com")
                .eq(User::getId, 1);
        userMapper.update(null, wrapper2);
        
        // 3. 批量更新
        LambdaUpdateWrapper<User> wrapper3 = new LambdaUpdateWrapper<>();
        wrapper3.set(User::getStatus, 0)
                .in(User::getId, Arrays.asList(1, 2, 3));
        userMapper.update(null, wrapper3);
        
        // 4. 条件更新
        LambdaUpdateWrapper<User> wrapper4 = new LambdaUpdateWrapper<>();
        wrapper4.setSql("age = age + 1")
                .eq(User::getStatus, 1);
        userMapper.update(null, wrapper4);
        // 生成SQL: UPDATE user SET age = age + 1 WHERE status=1
    }
}
```

#### 常用方法总结

| 方法 | 说明 | 示例 |
|------|------|------|
| eq | 等于 = | eq("name", "张三") |
| ne | 不等于 <> | ne("name", "张三") |
| gt | 大于 > | gt("age", 18) |
| ge | 大于等于 >= | ge("age", 18) |
| lt | 小于 < | lt("age", 30) |
| le | 小于等于 <= | le("age", 30) |
| between | BETWEEN 值1 AND 值2 | between("age", 18, 30) |
| notBetween | NOT BETWEEN 值1 AND 值2 | notBetween("age", 18, 30) |
| like | LIKE '%值%' | like("name", "张") |
| notLike | NOT LIKE '%值%' | notLike("name", "张") |
| likeLeft | LIKE '%值' | likeLeft("name", "三") |
| likeRight | LIKE '值%' | likeRight("name", "张") |
| isNull | IS NULL | isNull("email") |
| isNotNull | IS NOT NULL | isNotNull("email") |
| in | IN (值1, 值2, ...) | in("id", Arrays.asList(1,2,3)) |
| notIn | NOT IN (值1, 值2, ...) | notIn("id", Arrays.asList(1,2,3)) |
| inSql | IN (sql语句) | inSql("id", "select id from table") |
| groupBy | GROUP BY 字段 | groupBy("age") |
| orderByAsc | ORDER BY 字段 ASC | orderByAsc("age") |
| orderByDesc | ORDER BY 字段 DESC | orderByDesc("age") |
| or | 拼接OR | eq("id",1).or().eq("name","张三") |
| and | 拼接AND | and(i -> i.eq("name","张三")) |
| nested | 嵌套查询 | nested(i -> i.eq("name","张三")) |
| apply | 拼接SQL | apply("date_format(dateColumn,'%Y-%m-%d') = '2024-01-01'") |
| last | 拼接在SQL最后 | last("limit 1") |
| exists | EXISTS (sql语句) | exists("select id from table where age = 1") |


### 2.3 代码生成器 🔥

MyBatis-Plus提供了强大的代码生成器，可以快速生成Entity、Mapper、Service、Controller等代码。

#### Maven依赖
```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.3</version>
</dependency>

<!-- 模板引擎（选择一个） -->
<dependency>
    <groupId>org.apache.velocity</groupId>
    <artifactId>velocity-engine-core</artifactId>
    <version>2.3</version>
</dependency>
```

#### 代码生成器示例
```java
package com.example.generator;

import com.baomidou.mybatisplus.generator.FastAutoGenerator;
import com.baomidou.mybatisplus.generator.config.OutputFile;
import com.baomidou.mybatisplus.generator.engine.FreemarkerTemplateEngine;

import java.util.Collections;

/**
 * MyBatis-Plus代码生成器
 * @author erik.zhou
 */
public class CodeGenerator {
    
    public static void main(String[] args) {
        // 数据库配置
        String url = "jdbc:mysql://localhost:3306/mydb?useUnicode=true&characterEncoding=utf8";
        String username = "root";
        String password = "123456";
        
        // 输出目录
        String outputDir = System.getProperty("user.dir") + "/src/main/java";
        String mapperXmlDir = System.getProperty("user.dir") + "/src/main/resources/mapper";
        
        FastAutoGenerator.create(url, username, password)
            // 全局配置
            .globalConfig(builder -> {
                builder.author("erik.zhou")              // 作者
                       .outputDir(outputDir)             // 输出目录
                       .commentDate("yyyy-MM-dd")        // 注释日期格式
                       .disableOpenDir();                // 禁止打开输出目录
            })
            // 包配置
            .packageConfig(builder -> {
                builder.parent("com.example")            // 父包名
                       .moduleName("system")             // 模块名
                       .entity("model")                  // Entity包名
                       .mapper("mapper")                 // Mapper包名
                       .service("service")               // Service包名
                       .serviceImpl("service.impl")      // ServiceImpl包名
                       .controller("controller")         // Controller包名
                       .pathInfo(Collections.singletonMap(
                           OutputFile.xml, mapperXmlDir  // Mapper XML路径
                       ));
            })
            // 策略配置
            .strategyConfig(builder -> {
                builder.addInclude("user", "role", "permission")  // 要生成的表名
                       .addTablePrefix("t_", "sys_")              // 表前缀过滤
                       // Entity策略
                       .entityBuilder()
                       .enableLombok()                            // 启用Lombok
                       .enableTableFieldAnnotation()              // 启用字段注解
                       .logicDeleteColumnName("deleted")          // 逻辑删除字段
                       .versionColumnName("version")              // 乐观锁字段
                       // Mapper策略
                       .mapperBuilder()
                       .enableMapperAnnotation()                  // 启用@Mapper注解
                       .enableBaseResultMap()                     // 启用BaseResultMap
                       .enableBaseColumnList()                    // 启用BaseColumnList
                       // Service策略
                       .serviceBuilder()
                       .formatServiceFileName("%sService")        // Service接口名格式
                       .formatServiceImplFileName("%sServiceImpl") // ServiceImpl类名格式
                       // Controller策略
                       .controllerBuilder()
                       .enableRestStyle();                        // 启用REST风格
            })
            // 模板引擎配置
            .templateEngine(new FreemarkerTemplateEngine())
            // 执行生成
            .execute();
        
        System.out.println("代码生成完成！");
    }
}
```

#### 生成的代码结构
```
com.example.system
├── model
│   ├── User.java
│   ├── Role.java
│   └── Permission.java
├── mapper
│   ├── UserMapper.java
│   ├── RoleMapper.java
│   └── PermissionMapper.java
├── service
│   ├── UserService.java
│   ├── RoleService.java
│   └── PermissionService.java
├── service.impl
│   ├── UserServiceImpl.java
│   ├── RoleServiceImpl.java
│   └── PermissionServiceImpl.java
└── controller
    ├── UserController.java
    ├── RoleController.java
    └── PermissionController.java

resources/mapper
├── UserMapper.xml
├── RoleMapper.xml
└── PermissionMapper.xml
```

### 2.4 分页插件

MyBatis-Plus内置了分页插件，使用非常简单。

#### 配置分页插件
```java
package com.example.config;

import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * MyBatis-Plus配置类
 * @author erik.zhou
 */
@Configuration
public class MybatisPlusConfig {
    
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 添加分页插件
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}
```

#### 使用分页
```java
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    /**
     * 分页查询用户
     */
    public IPage<User> getUserPage(int pageNum, int pageSize, String name) {
        // 创建分页对象
        Page<User> page = new Page<>(pageNum, pageSize);
        
        // 构建查询条件
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.like(StringUtils.isNotBlank(name), User::getName, name)
               .orderByDesc(User::getCreatedAt);
        
        // 执行分页查询
        IPage<User> userPage = userMapper.selectPage(page, wrapper);
        
        System.out.println("总记录数: " + userPage.getTotal());
        System.out.println("总页数: " + userPage.getPages());
        System.out.println("当前页: " + userPage.getCurrent());
        System.out.println("每页大小: " + userPage.getSize());
        System.out.println("当前页数据: " + userPage.getRecords());
        
        return userPage;
    }
    
    /**
     * 自定义SQL分页查询
     */
    public IPage<User> customPage(int pageNum, int pageSize) {
        Page<User> page = new Page<>(pageNum, pageSize);
        // 调用自定义的分页方法
        return userMapper.selectUserPage(page, "张三");
    }
}
```

```java
// Mapper接口
@Mapper
public interface UserMapper extends BaseMapper<User> {
    
    /**
     * 自定义分页查询
     */
    IPage<User> selectUserPage(Page<?> page, @Param("name") String name);
}
```

```xml
<!-- Mapper XML -->
<select id="selectUserPage" resultType="User">
    SELECT * FROM user
    WHERE name LIKE CONCAT('%', #{name}, '%')
    ORDER BY created_at DESC
</select>
```

### 2.5 常用注解

#### @TableName（表名注解）
```java
@TableName("sys_user")  // 指定表名
public class User {
    // ...
}
```

#### @TableId（主键注解）
```java
public class User {
    
    @TableId(type = IdType.AUTO)  // 主键自增
    private Long id;
    
    // 其他主键策略：
    // IdType.NONE: 无状态
    // IdType.INPUT: 手动输入
    // IdType.ASSIGN_ID: 雪花算法（默认）
    // IdType.ASSIGN_UUID: UUID
}
```

#### @TableField（字段注解）
```java
public class User {
    
    @TableField("user_name")  // 指定数据库字段名
    private String name;
    
    @TableField(exist = false)  // 表示该字段不是数据库字段
    private String remark;
    
    @TableField(select = false)  // 查询时不返回该字段
    private String password;
    
    @TableField(fill = FieldFill.INSERT)  // 插入时自动填充
    private LocalDateTime createdAt;
    
    @TableField(fill = FieldFill.INSERT_UPDATE)  // 插入和更新时自动填充
    private LocalDateTime updatedAt;
}
```

#### @Version（乐观锁注解）
```java
public class User {
    
    @Version  // 乐观锁版本号
    private Integer version;
}
```

配置乐观锁插件：
```java
@Configuration
public class MybatisPlusConfig {
    
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        // 添加乐观锁插件
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}
```

#### @TableLogic（逻辑删除注解）
```java
public class User {
    
    @TableLogic  // 逻辑删除标记
    private Integer deleted;  // 0-未删除，1-已删除
}
```

配置逻辑删除：
```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: deleted  # 全局逻辑删除字段名
      logic-delete-value: 1        # 逻辑已删除值
      logic-not-delete-value: 0    # 逻辑未删除值
```


## 💻 实战应用

### 3.1 环境搭建

#### Maven依赖
```xml
<dependencies>
    <!-- Spring Boot Starter -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- MyBatis-Plus -->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.3.1</version>
    </dependency>
    
    <!-- MySQL驱动 -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.33</version>
    </dependency>
    
    <!-- Lombok（可选） -->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
</dependencies>
```

#### application.yml配置
```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/mydb?useUnicode=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
    username: root
    password: 123456

mybatis-plus:
  # 配置Mapper XML文件路径
  mapper-locations: classpath:mapper/**/*.xml
  # 配置类型别名包路径
  type-aliases-package: com.example.model
  # 全局配置
  global-config:
    db-config:
      # 主键类型（AUTO-自增，ASSIGN_ID-雪花算法）
      id-type: AUTO
      # 表名前缀
      table-prefix: t_
      # 逻辑删除配置
      logic-delete-field: deleted
      logic-delete-value: 1
      logic-not-delete-value: 0
  # MyBatis配置
  configuration:
    # 驼峰命名转换
    map-underscore-to-camel-case: true
    # 日志输出
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

### 3.2 快速开始

#### 1. 创建实体类
```java
package com.example.model;

import com.baomidou.mybatisplus.annotation.*;
import lombok.Data;
import java.time.LocalDateTime;

/**
 * 用户实体类
 * @author erik.zhou
 */
@Data
@TableName("user")
public class User {
    
    @TableId(type = IdType.AUTO)
    private Long id;
    
    private String name;
    
    private String email;
    
    private Integer age;
    
    private Integer status;
    
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createdAt;
    
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updatedAt;
    
    @TableLogic
    private Integer deleted;
    
    @Version
    private Integer version;
}
```

#### 2. 创建Mapper接口
```java
package com.example.mapper;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.example.model.User;
import org.apache.ibatis.annotations.Mapper;

/**
 * 用户Mapper接口
 * @author erik.zhou
 */
@Mapper
public interface UserMapper extends BaseMapper<User> {
    // 继承BaseMapper即可，无需编写任何方法
}
```

#### 3. 创建Service接口和实现类
```java
package com.example.service;

import com.baomidou.mybatisplus.extension.service.IService;
import com.example.model.User;

/**
 * 用户Service接口
 * @author erik.zhou
 */
public interface UserService extends IService<User> {
    // 继承IService即可获得常用方法
}
```

```java
package com.example.service.impl;

import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.mapper.UserMapper;
import com.example.model.User;
import com.example.service.UserService;
import org.springframework.stereotype.Service;

/**
 * 用户Service实现类
 * @author erik.zhou
 */
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    // 继承ServiceImpl即可获得常用方法
}
```

#### 4. 创建Controller
```java
package com.example.controller;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.example.model.User;
import com.example.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

/**
 * 用户Controller
 * @author erik.zhou
 */
@RestController
@RequestMapping("/user")
public class UserController {
    
    @Autowired
    private UserService userService;
    
    /**
     * 根据ID查询用户
     */
    @GetMapping("/{id}")
    public User getById(@PathVariable Long id) {
        return userService.getById(id);
    }
    
    /**
     * 查询所有用户
     */
    @GetMapping("/list")
    public List<User> list() {
        return userService.list();
    }
    
    /**
     * 分页查询用户
     */
    @GetMapping("/page")
    public IPage<User> page(@RequestParam(defaultValue = "1") int pageNum,
                           @RequestParam(defaultValue = "10") int pageSize,
                           @RequestParam(required = false) String name) {
        Page<User> page = new Page<>(pageNum, pageSize);
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.like(name != null, User::getName, name);
        return userService.page(page, wrapper);
    }
    
    /**
     * 新增用户
     */
    @PostMapping
    public boolean save(@RequestBody User user) {
        return userService.save(user);
    }
    
    /**
     * 更新用户
     */
    @PutMapping
    public boolean update(@RequestBody User user) {
        return userService.updateById(user);
    }
    
    /**
     * 删除用户
     */
    @DeleteMapping("/{id}")
    public boolean delete(@PathVariable Long id) {
        return userService.removeById(id);
    }
}
```

### 3.3 进阶案例

#### 案例1：自动填充
```java
package com.example.handler;

import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;
import org.apache.ibatis.reflection.MetaObject;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * 自动填充处理器
 * @author erik.zhou
 */
@Component
public class MyMetaObjectHandler implements MetaObjectHandler {
    
    @Override
    public void insertFill(MetaObject metaObject) {
        // 插入时自动填充
        this.strictInsertFill(metaObject, "createdAt", LocalDateTime.class, LocalDateTime.now());
        this.strictInsertFill(metaObject, "updatedAt", LocalDateTime.class, LocalDateTime.now());
    }
    
    @Override
    public void updateFill(MetaObject metaObject) {
        // 更新时自动填充
        this.strictUpdateFill(metaObject, "updatedAt", LocalDateTime.class, LocalDateTime.now());
    }
}
```

#### 案例2：批量操作
```java
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    /**
     * 批量插入（推荐）
     */
    public boolean saveBatch(List<User> users) {
        // 使用IService的saveBatch方法，默认每批1000条
        return userService.saveBatch(users);
    }
    
    /**
     * 批量更新
     */
    public boolean updateBatchById(List<User> users) {
        return userService.updateBatchById(users);
    }
    
    /**
     * 批量删除
     */
    public boolean removeBatchByIds(List<Long> ids) {
        return userService.removeBatchByIds(ids);
    }
}
```

#### 案例3：多表关联查询
```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    
    /**
     * 查询用户及其角色信息
     */
    @Select("SELECT u.*, r.role_name " +
            "FROM user u " +
            "LEFT JOIN user_role ur ON u.id = ur.user_id " +
            "LEFT JOIN role r ON ur.role_id = r.id " +
            "WHERE u.id = #{userId}")
    UserVO selectUserWithRoles(@Param("userId") Long userId);
}
```

## ✨ 最佳实践

### 4.1 性能优化

#### 1. 使用Lambda条件构造器
```java
// ✅ 推荐：使用Lambda，类型安全
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(User::getName, "张三");

// ❌ 不推荐：使用字符串，容易出错
QueryWrapper<User> wrapper = new QueryWrapper<>();
wrapper.eq("name", "张三");
```

#### 2. 批量操作
```java
// ❌ 不推荐：循环单条插入
for (User user : users) {
    userService.save(user);
}

// ✅ 推荐：批量插入
userService.saveBatch(users);
```

#### 3. 只查询需要的字段
```java
// ❌ 不推荐：查询所有字段
List<User> users = userMapper.selectList(null);

// ✅ 推荐：只查询需要的字段
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.select(User::getId, User::getName, User::getEmail);
List<User> users = userMapper.selectList(wrapper);
```

#### 4. 合理使用分页
```java
// ✅ 推荐：使用分页查询大量数据
Page<User> page = new Page<>(1, 100);
IPage<User> userPage = userMapper.selectPage(page, null);

// ❌ 不推荐：一次性查询所有数据
List<User> users = userMapper.selectList(null);
```

### 4.2 代码规范

#### 1. Service层使用IService
```java
// ✅ 推荐：使用IService提供的方法
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
    
    public boolean saveUser(User user) {
        // 使用IService的save方法
        return this.save(user);
    }
}

// ❌ 不推荐：直接使用Mapper
@Service
public class UserService {
    
    @Autowired
    private UserMapper userMapper;
    
    public int saveUser(User user) {
        return userMapper.insert(user);
    }
}
```

#### 2. 条件构造器的条件判断
```java
// ✅ 推荐：使用条件判断
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.eq(name != null, User::getName, name)
       .ge(minAge != null, User::getAge, minAge);

// ❌ 不推荐：手动判断
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
if (name != null) {
    wrapper.eq(User::getName, name);
}
if (minAge != null) {
    wrapper.ge(User::getAge, minAge);
}
```

#### 3. 实体类使用Lombok
```java
// ✅ 推荐：使用Lombok简化代码
@Data
@TableName("user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
}

// ❌ 不推荐：手写getter/setter
@TableName("user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String name;
    
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}
```


## ❓ 常见问题

### Q1: MyBatis-Plus和MyBatis有什么区别？

**A**: MyBatis-Plus是MyBatis的增强工具，主要区别：

| 特性 | MyBatis | MyBatis-Plus |
|------|---------|--------------|
| CRUD操作 | 需要手写SQL | 内置通用CRUD方法 |
| 条件构造 | 手写SQL或动态SQL | 条件构造器 |
| 分页 | 需要插件或手写 | 内置分页插件 |
| 代码生成 | MyBatis Generator | 更强大的代码生成器 |
| 学习成本 | 中等 | 低 |
| 开发效率 | 中等 | 高 |

**选择建议**：
- MyBatis-Plus完全兼容MyBatis
- 可以在同一个项目中混用
- 简单CRUD用MyBatis-Plus，复杂SQL用MyBatis

### Q2: 如何在MyBatis-Plus中使用自定义SQL？

**A**: 三种方式：

**方式1：在Mapper接口中使用注解**
```java
@Mapper
public interface UserMapper extends BaseMapper<User> {
    
    @Select("SELECT * FROM user WHERE name = #{name}")
    List<User> selectByName(@Param("name") String name);
}
```

**方式2：在Mapper XML中编写SQL**
```xml
<mapper namespace="com.example.mapper.UserMapper">
    <select id="selectByName" resultType="User">
        SELECT * FROM user WHERE name = #{name}
    </select>
</mapper>
```

**方式3：使用apply方法**
```java
LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
wrapper.apply("date_format(created_at,'%Y-%m-%d') = {0}", "2024-01-01");
List<User> users = userMapper.selectList(wrapper);
```

### Q3: 逻辑删除是如何工作的？

**A**: 逻辑删除不会真正删除数据，而是修改删除标记字段。

**配置**：
```java
@TableLogic
private Integer deleted;  // 0-未删除，1-已删除
```

**行为**：
```java
// 执行删除操作
userMapper.deleteById(1);
// 实际执行SQL: UPDATE user SET deleted=1 WHERE id=1

// 查询操作会自动过滤已删除数据
List<User> users = userMapper.selectList(null);
// 实际执行SQL: SELECT * FROM user WHERE deleted=0
```

**注意事项**：
- 逻辑删除只对MyBatis-Plus的方法有效
- 自定义SQL需要手动添加deleted条件
- 如需查询已删除数据，使用自定义SQL

### Q4: 如何实现乐观锁？

**A**: 使用@Version注解：

**1. 实体类添加版本字段**
```java
@Data
public class User {
    private Long id;
    private String name;
    
    @Version
    private Integer version;
}
```

**2. 配置乐观锁插件**
```java
@Configuration
public class MybatisPlusConfig {
    
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
        return interceptor;
    }
}
```

**3. 更新操作**
```java
// 查询用户（version=1）
User user = userMapper.selectById(1);

// 更新用户
user.setName("新名称");
int rows = userMapper.updateById(user);
// 执行SQL: UPDATE user SET name='新名称', version=2 WHERE id=1 AND version=1

if (rows == 0) {
    // 更新失败，说明数据已被其他线程修改
    System.out.println("更新失败，请重试");
}
```

### Q5: 如何处理枚举类型？

**A**: MyBatis-Plus提供了两种枚举处理方式：

**方式1：实现IEnum接口（推荐）**
```java
public enum UserStatus implements IEnum<Integer> {
    ACTIVE(1, "激活"),
    INACTIVE(0, "未激活"),
    DELETED(-1, "已删除");
    
    private final int code;
    private final String desc;
    
    UserStatus(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }
    
    @Override
    public Integer getValue() {
        return this.code;
    }
    
    public String getDesc() {
        return this.desc;
    }
}
```

**方式2：使用@EnumValue注解**
```java
public enum UserStatus {
    ACTIVE(1, "激活"),
    INACTIVE(0, "未激活"),
    DELETED(-1, "已删除");
    
    @EnumValue  // 标记数据库存储的值
    private final int code;
    private final String desc;
    
    UserStatus(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }
}
```

**配置**：
```yaml
mybatis-plus:
  configuration:
    # 配置默认枚举处理器
    default-enum-type-handler: com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
```

### Q6: 如何实现多租户？

**A**: 使用多租户插件：

**1. 配置多租户插件**
```java
@Configuration
public class MybatisPlusConfig {
    
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        
        // 添加多租户插件
        TenantLineInnerInterceptor tenantInterceptor = new TenantLineInnerInterceptor();
        tenantInterceptor.setTenantLineHandler(new TenantLineHandler() {
            @Override
            public Expression getTenantId() {
                // 从上下文获取租户ID
                Long tenantId = TenantContext.getTenantId();
                return new LongValue(tenantId);
            }
            
            @Override
            public String getTenantIdColumn() {
                return "tenant_id";  // 租户ID字段名
            }
            
            @Override
            public boolean ignoreTable(String tableName) {
                // 忽略某些表
                return "sys_config".equalsIgnoreCase(tableName);
            }
        });
        
        interceptor.addInnerInterceptor(tenantInterceptor);
        return interceptor;
    }
}
```

**2. 租户上下文**
```java
public class TenantContext {
    
    private static final ThreadLocal<Long> TENANT_ID = new ThreadLocal<>();
    
    public static void setTenantId(Long tenantId) {
        TENANT_ID.set(tenantId);
    }
    
    public static Long getTenantId() {
        return TENANT_ID.get();
    }
    
    public static void clear() {
        TENANT_ID.remove();
    }
}
```

**3. 使用**
```java
// 设置租户ID
TenantContext.setTenantId(1L);

// 查询操作会自动添加租户条件
List<User> users = userMapper.selectList(null);
// 执行SQL: SELECT * FROM user WHERE tenant_id = 1
```

## 🔗 相关资源

### 官方资源
- [MyBatis-Plus官方文档](https://baomidou.com/)
- [MyBatis-Plus GitHub仓库](https://github.com/baomidou/mybatis-plus)
- [MyBatis-Plus示例项目](https://github.com/baomidou/mybatis-plus-samples)

### 推荐插件
- [Dynamic Datasource](https://github.com/baomidou/dynamic-datasource-spring-boot-starter) - 多数据源
- [MyBatis-Plus Join](https://github.com/yulichang/mybatis-plus-join) - 多表关联查询
- [MyBatis-Plus Extension](https://github.com/baomidou/mybatis-plus-ext) - 扩展功能

### 学习资源
- [MyBatis-Plus快速入门](https://baomidou.com/pages/226c21/) - 官方教程
- [MyBatis-Plus实战](https://www.bilibili.com/video/BV1Xu411A7tL/) - 视频教程
- [MyBatis-Plus源码分析](https://www.cnblogs.com/youzhibing/category/1348199.html) - 博客系列

### 相关技术
- [MyBatis](https://mybatis.org/mybatis-3/) - 基础框架
- [Spring Boot](https://spring.io/projects/spring-boot) - 集成框架
- [Lombok](https://projectlombok.org/) - 代码简化工具

## 📝 学习检查清单

完成以下检查项，确保你已掌握MyBatis-Plus核心知识：

### 基础知识
- [ ] 理解MyBatis-Plus的设计理念和核心特性
- [ ] 掌握BaseMapper提供的14个CRUD方法
- [ ] 理解IService接口的常用方法
- [ ] 掌握常用注解的使用（@TableName、@TableId、@TableField等）

### 条件构造器
- [ ] 掌握QueryWrapper的使用
- [ ] 掌握LambdaQueryWrapper的使用（推荐）
- [ ] 掌握UpdateWrapper的使用
- [ ] 理解各种查询条件方法（eq、like、in等）
- [ ] 掌握复杂条件的构建（and、or、nested等）

### 高级特性
- [ ] 掌握代码生成器的使用
- [ ] 掌握分页插件的配置和使用
- [ ] 理解逻辑删除的原理和使用
- [ ] 理解乐观锁的实现和使用
- [ ] 掌握自动填充的配置
- [ ] 了解多租户插件的使用

### 实战能力
- [ ] 能够快速搭建MyBatis-Plus项目
- [ ] 能够使用代码生成器生成代码
- [ ] 能够编写复杂的查询条件
- [ ] 能够实现批量操作
- [ ] 能够与Spring Boot集成

### 最佳实践
- [ ] 优先使用Lambda条件构造器
- [ ] 合理使用批量操作
- [ ] 只查询需要的字段
- [ ] 使用IService简化Service层代码
- [ ] 使用Lombok简化实体类代码

---

**@author** erik.zhou  
**文档版本**: 1.0  
**最后更新**: 2024-12-31  
**文档来源**: Context7 + MyBatis-Plus官方文档
