# PostgreSQL 完整教程

## 📋 目录
- 技术概述
- 学习目标
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: PostgreSQL 15/16
- **官方文档**: https://www.postgresql.org/docs/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: SQL基础、数据库基本概念
- **更新时间**: 2024-12-31

## 🎯 学习目标
- [ ] 掌握PostgreSQL基础操作
- [ ] 理解高级特性（JSON、全文搜索）
- [ ] 掌握性能优化方法
- [ ] 了解扩展和插件

## 📖 基础概念

### 1.1 什么是PostgreSQL
PostgreSQL是功能强大的开源对象关系型数据库系统，以稳定性、数据完整性和正确性著称。支持SQL标准和众多高级特性。

### 1.2 核心概念
- **数据库(Database)**: 独立的数据存储空间
- **Schema**: 数据库内的命名空间
- **表(Table)**: 数据存储的基本单位
- **视图(View)**: 虚拟表
- **函数(Function)**: 可重用的代码块
- **扩展(Extension)**: 增强功能的插件

### 1.3 应用场景
- Web应用后端数据库
- 地理信息系统(GIS)
- 数据仓库和分析
- 时序数据存储

## 🔥 核心特性

### 2.1 JSON支持 🔥

**JSON数据类型**
```sql
-- 创建包含JSON字段的表
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    attributes JSONB  -- JSONB比JSON性能更好
);

-- 插入JSON数据
INSERT INTO products (name, attributes) VALUES
('Laptop', '{"brand": "Dell", "cpu": "Intel i7", "ram": "16GB"}'),
('Phone', '{"brand": "Apple", "model": "iPhone 15", "storage": "256GB"}');

-- 查询JSON字段
SELECT name, attributes->>'brand' AS brand
FROM products;

-- JSON路径查询
SELECT name, attributes->'cpu' AS cpu
FROM products
WHERE attributes->>'brand' = 'Dell';

-- JSON数组操作
SELECT name, jsonb_array_elements(attributes->'features') AS feature
FROM products;

-- 创建JSON索引
CREATE INDEX idx_products_brand ON products ((attributes->>'brand'));
```

### 2.2 全文搜索 🔥

```sql
-- 创建全文搜索列
ALTER TABLE articles ADD COLUMN tsv tsvector;

-- 更新全文搜索列
UPDATE articles
SET tsv = to_tsvector('english', title || ' ' || content);

-- 创建GIN索引
CREATE INDEX idx_articles_tsv ON articles USING GIN(tsv);

-- 全文搜索查询
SELECT title, ts_rank(tsv, query) AS rank
FROM articles, to_tsquery('english', 'postgresql & database') query
WHERE tsv @@ query
ORDER BY rank DESC;

-- 中文全文搜索（需要zhparser扩展）
CREATE EXTENSION zhparser;
CREATE TEXT SEARCH CONFIGURATION chinese (PARSER = zhparser);

SELECT * FROM articles
WHERE to_tsvector('chinese', content) @@ to_tsquery('chinese', '数据库');
```

### 2.3 高级特性

**数组类型**
```sql
-- 创建数组字段
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50),
    tags TEXT[]
);

-- 插入数组数据
INSERT INTO users (name, tags) VALUES
('Erik', ARRAY['developer', 'java', 'postgresql']);

-- 查询数组
SELECT * FROM users WHERE 'java' = ANY(tags);
SELECT * FROM users WHERE tags @> ARRAY['java'];
```

**窗口函数**
```sql
-- 排名函数
SELECT 
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;

-- 分组排名
SELECT 
    department_id,
    name,
    salary,
    RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) AS dept_rank
FROM employees;
```

**CTE（公共表表达式）**
```sql
-- 递归查询组织结构
WITH RECURSIVE org_tree AS (
    -- 基础查询：顶层节点
    SELECT id, name, parent_id, 1 AS level
    FROM departments
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- 递归查询：子节点
    SELECT d.id, d.name, d.parent_id, ot.level + 1
    FROM departments d
    INNER JOIN org_tree ot ON d.parent_id = ot.id
)
SELECT * FROM org_tree ORDER BY level, id;
```


## 💻 实战应用

### 3.1 环境搭建

**Docker安装PostgreSQL**
```bash
# 拉取PostgreSQL镜像
docker pull postgres:16

# 启动PostgreSQL容器
docker run -d \
  --name postgres16 \
  -p 5432:5432 \
  -e POSTGRES_PASSWORD=postgres \
  -v /data/postgres:/var/lib/postgresql/data \
  postgres:16

# 连接PostgreSQL
docker exec -it postgres16 psql -U postgres
```

### 3.2 Java集成

**添加依赖**
```xml
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.1</version>
</dependency>
```

**配置连接**
```yaml
spring:
  datasource:
    driver-class-name: org.postgresql.Driver
    url: jdbc:postgresql://localhost:5432/mydb
    username: postgres
    password: postgres
  jpa:
    database-platform: org.hibernate.dialect.PostgreSQLDialect
    show-sql: true
```

### 3.3 性能优化案例

**EXPLAIN分析**
```sql
-- 查看执行计划
EXPLAIN ANALYZE
SELECT * FROM orders WHERE user_id = 1001;

-- 详细执行计划
EXPLAIN (ANALYZE, BUFFERS, VERBOSE)
SELECT * FROM orders WHERE user_id = 1001;
```

**索引优化**
```sql
-- B-Tree索引（默认）
CREATE INDEX idx_orders_user ON orders(user_id);

-- 部分索引
CREATE INDEX idx_orders_pending ON orders(status)
WHERE status = 'pending';

-- 表达式索引
CREATE INDEX idx_users_lower_email ON users(LOWER(email));

-- GiST索引（用于地理数据）
CREATE INDEX idx_locations_point ON locations USING GIST(point);
```

## ✨ 最佳实践

### 4.1 性能优化
- 使用EXPLAIN ANALYZE分析查询
- 合理设计索引
- 使用连接池
- 定期VACUUM清理
- 调整shared_buffers和work_mem

### 4.2 数据类型选择
- 使用SERIAL代替手动序列
- 使用JSONB代替JSON
- 使用TEXT代替VARCHAR(无长度限制时)
- 使用TIMESTAMP WITH TIME ZONE存储时间

### 4.3 安全规范
```sql
-- 创建只读用户
CREATE USER readonly WITH PASSWORD 'password';
GRANT CONNECT ON DATABASE mydb TO readonly;
GRANT USAGE ON SCHEMA public TO readonly;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly;

-- 行级安全策略
ALTER TABLE orders ENABLE ROW LEVEL SECURITY;

CREATE POLICY user_orders ON orders
FOR SELECT
USING (user_id = current_user_id());
```

## ❓ 常见问题

### Q1: PostgreSQL与MySQL的区别？
A: 
- PostgreSQL更符合SQL标准
- PostgreSQL支持更多高级特性（JSON、数组、全文搜索）
- PostgreSQL支持复杂查询和窗口函数
- MySQL在简单查询上性能更好
- PostgreSQL更适合复杂业务逻辑

### Q2: 如何优化PostgreSQL性能？
A: 
1. 合理设计索引
2. 使用EXPLAIN分析慢查询
3. 定期VACUUM和ANALYZE
4. 调整配置参数（shared_buffers、work_mem）
5. 使用连接池
6. 考虑分区表

### Q3: JSONB与JSON的区别？
A: 
- JSONB以二进制格式存储，JSON以文本格式
- JSONB支持索引，查询更快
- JSONB插入稍慢（需要转换）
- 推荐使用JSONB

## 🔗 相关资源
- 官方文档：https://www.postgresql.org/docs/
- pgAdmin：图形化管理工具
- PostGIS：地理信息扩展

## 📝 学习检查清单
- [ ] 掌握PostgreSQL基础操作
- [ ] 理解JSON和数组类型
- [ ] 掌握全文搜索
- [ ] 了解窗口函数和CTE
- [ ] 掌握性能优化方法

---

**@author erik.zhou**  
**文档版本**: v1.0  
**最后更新**: 2024-12-31
