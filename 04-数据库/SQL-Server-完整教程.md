# SQL Server 完整教程

## 📋 目录
- 技术概述
- 学习目标  
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: SQL Server 2019/2022
- **官方文档**: https://docs.microsoft.com/sql/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐ (1-5星)
- **前置知识**: SQL基础、数据库基本概念
- **更新时间**: 2024-12-31

## 🎯 学习目标
- [ ] 掌握SQL Server基础操作
- [ ] 理解T-SQL编程
- [ ] 掌握索引和查询优化
- [ ] 了解备份恢复策略

## 📖 基础概念

### 1.1 什么是SQL Server
Microsoft SQL Server是微软开发的关系型数据库管理系统，与Windows生态紧密集成，广泛应用于企业应用开发。

### 1.2 核心概念
- **数据库(Database)**: 数据存储的逻辑单元
- **表(Table)**: 数据的基本存储结构
- **视图(View)**: 虚拟表
- **存储过程(Stored Procedure)**: 预编译的SQL语句集
- **触发器(Trigger)**: 自动执行的特殊存储过程
- **索引(Index)**: 提高查询性能的数据结构

### 1.3 应用场景
- .NET应用后端数据库
- 企业管理系统
- 数据仓库和BI系统
- Web应用数据存储

## 🔥 核心特性

### 2.1 T-SQL编程 🔥

**存储过程**
```sql
CREATE PROCEDURE usp_GetEmployeeByDept
    @DeptId INT
AS
BEGIN
    SET NOCOUNT ON;
    
    SELECT EmployeeId, FirstName, LastName, Salary
    FROM Employees
    WHERE DepartmentId = @DeptId
    ORDER BY LastName;
END;
GO

-- 执行存储过程
EXEC usp_GetEmployeeByDept @DeptId = 10;
```

**函数**
```sql
-- 标量函数
CREATE FUNCTION dbo.fn_GetFullName
(
    @FirstName NVARCHAR(50),
    @LastName NVARCHAR(50)
)
RETURNS NVARCHAR(101)
AS
BEGIN
    RETURN @FirstName + ' ' + @LastName;
END;
GO

-- 使用函数
SELECT dbo.fn_GetFullName(FirstName, LastName) AS FullName
FROM Employees;

-- 表值函数
CREATE FUNCTION dbo.fn_GetEmployeesByDept
(
    @DeptId INT
)
RETURNS TABLE
AS
RETURN
(
    SELECT EmployeeId, FirstName, LastName
    FROM Employees
    WHERE DepartmentId = @DeptId
);
GO
```

**触发器**
```sql
CREATE TRIGGER trg_AuditSalaryChange
ON Employees
AFTER UPDATE
AS
BEGIN
    IF UPDATE(Salary)
    BEGIN
        INSERT INTO SalaryAudit(EmployeeId, OldSalary, NewSalary, ChangeDate)
        SELECT 
            i.EmployeeId,
            d.Salary AS OldSalary,
            i.Salary AS NewSalary,
            GETDATE()
        FROM inserted i
        INNER JOIN deleted d ON i.EmployeeId = d.EmployeeId;
    END;
END;
GO
```


### 2.2 索引优化 🔥

**创建索引**
```sql
-- 聚集索引（每个表只能有一个）
CREATE CLUSTERED INDEX IX_Employee_Id ON Employees(EmployeeId);

-- 非聚集索引
CREATE NONCLUSTERED INDEX IX_Employee_Dept ON Employees(DepartmentId);

-- 包含列索引（覆盖索引）
CREATE NONCLUSTERED INDEX IX_Employee_Dept_Include
ON Employees(DepartmentId)
INCLUDE (FirstName, LastName, Salary);

-- 唯一索引
CREATE UNIQUE INDEX IX_Employee_Email ON Employees(Email);

-- 过滤索引
CREATE NONCLUSTERED INDEX IX_Employee_Active
ON Employees(Status)
WHERE Status = 1;
```

**查看执行计划**
```sql
-- 显示实际执行计划
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

SELECT * FROM Employees WHERE DepartmentId = 10;

-- 查看索引使用情况
SELECT 
    OBJECT_NAME(s.object_id) AS TableName,
    i.name AS IndexName,
    s.user_seeks,
    s.user_scans,
    s.user_lookups,
    s.user_updates
FROM sys.dm_db_index_usage_stats s
INNER JOIN sys.indexes i ON s.object_id = i.object_id AND s.index_id = i.index_id
WHERE OBJECT_NAME(s.object_id) = 'Employees';
```

### 2.3 备份恢复 🔥

**完整备份**
```sql
-- 完整备份
BACKUP DATABASE MyDatabase
TO DISK = 'C:\Backup\MyDatabase_Full.bak'
WITH FORMAT, INIT, NAME = 'Full Backup';

-- 差异备份
BACKUP DATABASE MyDatabase
TO DISK = 'C:\Backup\MyDatabase_Diff.bak'
WITH DIFFERENTIAL, INIT, NAME = 'Differential Backup';

-- 事务日志备份
BACKUP LOG MyDatabase
TO DISK = 'C:\Backup\MyDatabase_Log.trn'
WITH INIT, NAME = 'Log Backup';
```

**恢复数据库**
```sql
-- 恢复完整备份
RESTORE DATABASE MyDatabase
FROM DISK = 'C:\Backup\MyDatabase_Full.bak'
WITH NORECOVERY;

-- 恢复差异备份
RESTORE DATABASE MyDatabase
FROM DISK = 'C:\Backup\MyDatabase_Diff.bak'
WITH NORECOVERY;

-- 恢复事务日志
RESTORE LOG MyDatabase
FROM DISK = 'C:\Backup\MyDatabase_Log.trn'
WITH RECOVERY;
```

## 💻 实战应用

### 3.1 Java集成

**添加依赖**
```xml
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>mssql-jdbc</artifactId>
    <version>12.4.2.jre11</version>
</dependency>
```

**配置连接**
```yaml
spring:
  datasource:
    driver-class-name: com.microsoft.sqlserver.jdbc.SQLServerDriver
    url: jdbc:sqlserver://localhost:1433;databaseName=MyDatabase;encrypt=true;trustServerCertificate=true
    username: sa
    password: YourPassword123
```

### 3.2 性能优化案例

**分页查询优化**
```sql
-- SQL Server 2012+ 使用OFFSET-FETCH
SELECT EmployeeId, FirstName, LastName
FROM Employees
ORDER BY EmployeeId
OFFSET 100 ROWS
FETCH NEXT 20 ROWS ONLY;

-- 使用ROW_NUMBER()
WITH EmployeeCTE AS (
    SELECT 
        EmployeeId,
        FirstName,
        LastName,
        ROW_NUMBER() OVER (ORDER BY EmployeeId) AS RowNum
    FROM Employees
)
SELECT EmployeeId, FirstName, LastName
FROM EmployeeCTE
WHERE RowNum BETWEEN 101 AND 120;
```

## ✨ 最佳实践

### 4.1 查询优化
- 避免SELECT *
- 使用参数化查询防止SQL注入
- 合理使用索引
- 避免在WHERE子句中使用函数
- 使用EXISTS代替IN

### 4.2 索引设计
- 主键自动创建聚集索引
- 外键字段创建非聚集索引
- 经常查询的字段创建索引
- 避免过多索引影响写入性能

## ❓ 常见问题

### Q1: SQL Server与MySQL的区别？
A: 
- SQL Server使用T-SQL，MySQL使用标准SQL
- SQL Server商业授权，MySQL开源
- SQL Server与Windows集成更好
- 语法差异：TOP vs LIMIT，GETDATE() vs NOW()

### Q2: 如何优化SQL Server性能？
A: 
1. 合理设计索引
2. 定期更新统计信息
3. 使用执行计划分析慢查询
4. 配置合适的内存和CPU
5. 定期维护数据库（重建索引、更新统计）

## 🔗 相关资源
- 官方文档：https://docs.microsoft.com/sql/
- SQL Server Management Studio (SSMS)

## 📝 学习检查清单
- [ ] 掌握T-SQL基础语法
- [ ] 理解存储过程和函数
- [ ] 掌握索引优化
- [ ] 了解备份恢复策略

---

**@author erik.zhou**  
**文档版本**: v1.0  
**最后更新**: 2024-12-31
