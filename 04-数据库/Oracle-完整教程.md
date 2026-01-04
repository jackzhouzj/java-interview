# Oracle 完整教程

## 📋 目录
- 技术概述
- 学习目标
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: Oracle Database 19c/21c
- **官方文档**: https://docs.oracle.com/en/database/
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐ (1-5星)
- **前置知识**: SQL基础、数据库基本概念
- **更新时间**: 2024-12-31

## 🎯 学习目标
- [ ] 掌握Oracle基础操作和SQL语法
- [ ] 理解PL/SQL编程
- [ ] 掌握Oracle性能优化
- [ ] 了解Oracle高可用方案

## 📖 基础概念

### 1.1 什么是Oracle
Oracle Database是企业级关系型数据库管理系统，以高性能、高可用性和安全性著称。广泛应用于金融、电信、政府等关键业务领域。

### 1.2 核心概念
- **实例(Instance)**: 内存结构和后台进程的集合
- **数据库(Database)**: 物理文件的集合
- **表空间(Tablespace)**: 逻辑存储单元
- **数据文件(Datafile)**: 物理存储文件
- **用户(User)**: 数据库账户，拥有schema
- **Schema**: 用户拥有的数据库对象集合

### 1.3 应用场景
- 大型企业ERP系统
- 金融交易系统
- 电信计费系统
- 政府信息系统


## 🔥 核心特性

### 2.1 PL/SQL编程 🔥

**存储过程**
```sql
CREATE OR REPLACE PROCEDURE update_salary(
    p_emp_id IN NUMBER,
    p_increase IN NUMBER
) AS
BEGIN
    UPDATE employees
    SET salary = salary + p_increase
    WHERE employee_id = p_emp_id;
    
    COMMIT;
EXCEPTION
    WHEN OTHERS THEN
        ROLLBACK;
        RAISE;
END;
/

-- 调用存储过程
EXEC update_salary(1001, 1000);
```

**函数**
```sql
CREATE OR REPLACE FUNCTION get_employee_name(
    p_emp_id IN NUMBER
) RETURN VARCHAR2 AS
    v_name VARCHAR2(100);
BEGIN
    SELECT first_name || ' ' || last_name
    INTO v_name
    FROM employees
    WHERE employee_id = p_emp_id;
    
    RETURN v_name;
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN NULL;
END;
/

-- 调用函数
SELECT get_employee_name(1001) FROM DUAL;
```

**触发器**
```sql
CREATE OR REPLACE TRIGGER audit_salary_changes
BEFORE UPDATE OF salary ON employees
FOR EACH ROW
BEGIN
    INSERT INTO salary_audit(
        employee_id,
        old_salary,
        new_salary,
        change_date
    ) VALUES (
        :NEW.employee_id,
        :OLD.salary,
        :NEW.salary,
        SYSDATE
    );
END;
/
```

### 2.2 性能优化 🔥

**执行计划分析**
```sql
-- 查看执行计划
EXPLAIN PLAN FOR
SELECT * FROM employees WHERE department_id = 10;

-- 显示执行计划
SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);
```

**索引优化**
```sql
-- 创建B-Tree索引
CREATE INDEX idx_emp_dept ON employees(department_id);

-- 创建位图索引（适合低基数列）
CREATE BITMAP INDEX idx_emp_gender ON employees(gender);

-- 创建函数索引
CREATE INDEX idx_emp_upper_name ON employees(UPPER(last_name));

-- 重建索引
ALTER INDEX idx_emp_dept REBUILD;
```

**分区表**
```sql
-- 范围分区
CREATE TABLE sales (
    sale_id NUMBER,
    sale_date DATE,
    amount NUMBER
)
PARTITION BY RANGE (sale_date) (
    PARTITION p_2023 VALUES LESS THAN (TO_DATE('2024-01-01', 'YYYY-MM-DD')),
    PARTITION p_2024 VALUES LESS THAN (TO_DATE('2025-01-01', 'YYYY-MM-DD'))
);
```

### 2.3 高可用方案

**Data Guard**
- 主备数据库同步
- 自动故障切换
- 支持物理和逻辑备库

**RAC（Real Application Clusters）**
- 多节点共享存储
- 负载均衡
- 高可用性


## 💻 实战应用

### 3.1 环境搭建

**Docker安装Oracle**
```bash
# 拉取Oracle镜像
docker pull container-registry.oracle.com/database/express:21.3.0-xe

# 启动Oracle容器
docker run -d \
  --name oracle21c \
  -p 1521:1521 \
  -e ORACLE_PWD=Oracle123 \
  container-registry.oracle.com/database/express:21.3.0-xe

# 连接Oracle
docker exec -it oracle21c sqlplus system/Oracle123@//localhost:1521/XE
```

### 3.2 Java集成

**添加依赖**
```xml
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>21.9.0.0</version>
</dependency>
```

**配置连接**
```yaml
spring:
  datasource:
    driver-class-name: oracle.jdbc.OracleDriver
    url: jdbc:oracle:thin:@localhost:1521:XE
    username: system
    password: Oracle123
```

## ✨ 最佳实践

### 4.1 SQL优化
- 使用绑定变量避免硬解析
- 合理使用索引
- 避免全表扫描
- 使用分区表处理大数据量

### 4.2 PL/SQL规范
- 使用异常处理
- 合理使用游标
- 批量操作使用BULK COLLECT
- 避免在循环中执行SQL

## ❓ 常见问题

### Q1: Oracle与MySQL的主要区别？
A: 
- Oracle是商业数据库，MySQL开源
- Oracle功能更强大，适合大型企业
- Oracle支持更复杂的PL/SQL
- MySQL更轻量，易于部署

### Q2: 如何优化Oracle性能？
A: 
1. 合理设计索引
2. 使用分区表
3. 优化SQL语句
4. 调整SGA和PGA参数
5. 定期收集统计信息

## 🔗 相关资源
- Oracle官方文档：https://docs.oracle.com/
- Oracle技术网：https://www.oracle.com/technical-resources/

## 📝 学习检查清单
- [ ] 掌握Oracle基础操作
- [ ] 理解PL/SQL编程
- [ ] 掌握性能优化方法
- [ ] 了解高可用方案

---

**@author erik.zhou**  
**文档版本**: v1.0  
**最后更新**: 2024-12-31
