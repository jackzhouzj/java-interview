# MySQL运维-完整教程

> @author erik.zhou

## 📋 目录
- [技术概述](#技术概述)
- [安装部署](#安装部署)
- [主从复制](#主从复制)
- [性能优化](#性能优化)
- [备份恢复](#备份恢复)
- [监控告警](#监控告警)
- [常见问题](#常见问题)

## 📚 技术概述

### 基本信息
- **重要程度**：⭐⭐⭐⭐⭐ (P0必学)
- **难度级别**：⭐⭐⭐⭐
- **前置知识**：Linux、SQL基础
- **学习时长**：30-40小时

### 学习目标
- [ ] 搭建MySQL主从复制
- [ ] 进行性能优化
- [ ] 设计备份方案
- [ ] 排查常见问题


---

## 🔧 安装部署

### YUM安装

```bash
# 添加MySQL仓库
rpm -Uvh https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm

# 安装MySQL
yum install -y mysql-community-server

# 启动服务
systemctl start mysqld
systemctl enable mysqld

# 获取临时密码
grep 'temporary password' /var/log/mysqld.log

# 安全初始化
mysql_secure_installation
```

### 配置文件 🔥

```ini
# /etc/my.cnf
[mysqld]
# 基础配置
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
port=3306
user=mysql

# 字符集
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

# 连接配置
max_connections=1000
max_connect_errors=100
wait_timeout=28800
interactive_timeout=28800

# 缓冲区配置
innodb_buffer_pool_size=4G          # 物理内存的50-70%
innodb_buffer_pool_instances=4
innodb_log_file_size=1G
innodb_log_buffer_size=64M

# 日志配置
log_error=/var/log/mysql/error.log
slow_query_log=1
slow_query_log_file=/var/log/mysql/slow.log
long_query_time=2
log_queries_not_using_indexes=1

# 二进制日志
server-id=1
log-bin=mysql-bin
binlog_format=ROW
expire_logs_days=7
max_binlog_size=500M

# InnoDB配置
innodb_file_per_table=1
innodb_flush_log_at_trx_commit=1
innodb_flush_method=O_DIRECT
innodb_io_capacity=2000
innodb_read_io_threads=8
innodb_write_io_threads=8

[client]
default-character-set=utf8mb4
socket=/var/lib/mysql/mysql.sock
```

---

## 🔄 主从复制

### 主从架构 🔥

```
┌─────────────┐         ┌─────────────┐
│   Master    │ ──────► │   Slave     │
│  (读写)     │  binlog │  (只读)     │
└─────────────┘         └─────────────┘
```

### 主库配置

```ini
# /etc/my.cnf (Master)
[mysqld]
server-id=1
log-bin=mysql-bin
binlog_format=ROW
sync_binlog=1
innodb_flush_log_at_trx_commit=1

# GTID模式（推荐）
gtid_mode=ON
enforce_gtid_consistency=ON
```

```sql
-- 创建复制用户
CREATE USER 'repl'@'%' IDENTIFIED BY 'repl_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl'@'%';
FLUSH PRIVILEGES;

-- 查看主库状态
SHOW MASTER STATUS;
```

### 从库配置

```ini
# /etc/my.cnf (Slave)
[mysqld]
server-id=2
relay-log=relay-bin
read_only=1
super_read_only=1

# GTID模式
gtid_mode=ON
enforce_gtid_consistency=ON
```

```sql
-- 配置主库信息（GTID模式）
CHANGE MASTER TO
    MASTER_HOST='192.168.1.10',
    MASTER_USER='repl',
    MASTER_PASSWORD='repl_password',
    MASTER_AUTO_POSITION=1;

-- 启动复制
START SLAVE;

-- 查看复制状态
SHOW SLAVE STATUS\G
```

### 复制状态检查

```sql
-- 关键指标
SHOW SLAVE STATUS\G

-- 检查项：
-- Slave_IO_Running: Yes
-- Slave_SQL_Running: Yes
-- Seconds_Behind_Master: 0
-- Last_Error: (空)
```

---

## 🚀 性能优化

### 慢查询分析 🔥

```bash
# 开启慢查询日志
SET GLOBAL slow_query_log = 1;
SET GLOBAL long_query_time = 2;

# 分析慢查询日志
mysqldumpslow -s t -t 10 /var/log/mysql/slow.log

# 使用pt-query-digest
pt-query-digest /var/log/mysql/slow.log > slow_report.txt
```

### 执行计划分析

```sql
-- EXPLAIN分析
EXPLAIN SELECT * FROM users WHERE name = 'test';

-- 关注字段：
-- type: 访问类型（ALL < index < range < ref < eq_ref < const）
-- key: 使用的索引
-- rows: 扫描行数
-- Extra: 额外信息

-- EXPLAIN ANALYZE (MySQL 8.0+)
EXPLAIN ANALYZE SELECT * FROM users WHERE name = 'test';
```

### 索引优化 🔥

```sql
-- 查看表索引
SHOW INDEX FROM table_name;

-- 创建索引
CREATE INDEX idx_name ON users(name);
CREATE INDEX idx_composite ON orders(user_id, status, created_at);

-- 查看索引使用情况
SELECT * FROM sys.schema_unused_indexes;
SELECT * FROM sys.schema_redundant_indexes;

-- 索引建议
-- 1. 选择性高的列建索引
-- 2. 组合索引遵循最左前缀原则
-- 3. 避免过多索引（影响写入性能）
-- 4. 定期分析和优化索引
```

### 参数优化

```sql
-- 查看当前配置
SHOW VARIABLES LIKE 'innodb_buffer_pool_size';
SHOW GLOBAL STATUS LIKE 'Innodb_buffer_pool%';

-- 缓冲池命中率
SELECT 
    (1 - (Innodb_buffer_pool_reads / Innodb_buffer_pool_read_requests)) * 100 
    AS buffer_pool_hit_rate
FROM (
    SELECT 
        VARIABLE_VALUE AS Innodb_buffer_pool_reads 
    FROM performance_schema.global_status 
    WHERE VARIABLE_NAME = 'Innodb_buffer_pool_reads'
) a, (
    SELECT 
        VARIABLE_VALUE AS Innodb_buffer_pool_read_requests 
    FROM performance_schema.global_status 
    WHERE VARIABLE_NAME = 'Innodb_buffer_pool_read_requests'
) b;
```

---

## 💾 备份恢复

### mysqldump备份 🔥

```bash
# 全量备份
mysqldump -u root -p --all-databases --single-transaction \
    --master-data=2 --flush-logs > full_backup.sql

# 单库备份
mysqldump -u root -p --single-transaction \
    --databases mydb > mydb_backup.sql

# 单表备份
mysqldump -u root -p mydb users > users_backup.sql

# 压缩备份
mysqldump -u root -p --all-databases | gzip > backup_$(date +%Y%m%d).sql.gz

# 恢复
mysql -u root -p < full_backup.sql
gunzip < backup.sql.gz | mysql -u root -p
```

### xtrabackup备份

```bash
# 安装
yum install -y percona-xtrabackup-80

# 全量备份
xtrabackup --backup --target-dir=/backup/full \
    --user=root --password=xxx

# 准备备份
xtrabackup --prepare --target-dir=/backup/full

# 恢复
systemctl stop mysqld
rm -rf /var/lib/mysql/*
xtrabackup --copy-back --target-dir=/backup/full
chown -R mysql:mysql /var/lib/mysql
systemctl start mysqld

# 增量备份
xtrabackup --backup --target-dir=/backup/inc1 \
    --incremental-basedir=/backup/full \
    --user=root --password=xxx
```

### 备份策略

```
每日备份策略：
├── 周日：全量备份
├── 周一：增量备份
├── 周二：增量备份
├── 周三：增量备份
├── 周四：增量备份
├── 周五：增量备份
└── 周六：增量备份

保留策略：
├── 每日备份保留7天
├── 每周备份保留4周
└── 每月备份保留12个月
```

---

## 📊 监控告警

### 关键监控指标 🔥

```sql
-- 连接数
SHOW GLOBAL STATUS LIKE 'Threads_connected';
SHOW GLOBAL STATUS LIKE 'Max_used_connections';

-- QPS/TPS
SHOW GLOBAL STATUS LIKE 'Questions';
SHOW GLOBAL STATUS LIKE 'Com_commit';
SHOW GLOBAL STATUS LIKE 'Com_rollback';

-- 慢查询
SHOW GLOBAL STATUS LIKE 'Slow_queries';

-- 锁等待
SHOW GLOBAL STATUS LIKE 'Innodb_row_lock%';

-- 复制延迟
SHOW SLAVE STATUS\G  -- Seconds_Behind_Master
```

### Prometheus监控

```yaml
# mysqld_exporter配置
scrape_configs:
  - job_name: 'mysql'
    static_configs:
      - targets: ['mysql-exporter:9104']
```

### 告警规则

```yaml
groups:
  - name: mysql_alerts
    rules:
      - alert: MySQLDown
        expr: mysql_up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "MySQL实例宕机"

      - alert: MySQLSlaveDelay
        expr: mysql_slave_status_seconds_behind_master > 60
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "MySQL复制延迟超过60秒"

      - alert: MySQLConnectionsHigh
        expr: mysql_global_status_threads_connected / mysql_global_variables_max_connections > 0.8
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "MySQL连接数超过80%"
```

---

## ❓ 常见问题

### 问题1：主从复制延迟

**排查**：
```sql
SHOW SLAVE STATUS\G
-- 查看Seconds_Behind_Master
```

**解决**：
- 优化慢SQL
- 增加从库配置
- 使用并行复制
- 考虑分库分表

### 问题2：死锁

**排查**：
```sql
SHOW ENGINE INNODB STATUS\G
-- 查看LATEST DETECTED DEADLOCK

-- 查看当前锁
SELECT * FROM information_schema.INNODB_LOCKS;
SELECT * FROM information_schema.INNODB_LOCK_WAITS;
```

**解决**：
- 优化事务大小
- 统一访问顺序
- 添加合适索引

### 问题3：表空间过大

**排查**：
```sql
SELECT 
    table_schema,
    table_name,
    ROUND(data_length/1024/1024, 2) AS data_mb,
    ROUND(index_length/1024/1024, 2) AS index_mb
FROM information_schema.tables
ORDER BY data_length DESC
LIMIT 20;
```

**解决**：
```sql
-- 优化表（重建表空间）
OPTIMIZE TABLE table_name;

-- 或者
ALTER TABLE table_name ENGINE=InnoDB;
```

---

## 📝 学习检查清单

- [ ] 能够安装配置MySQL
- [ ] 能够搭建主从复制
- [ ] 能够进行性能优化
- [ ] 能够设计备份方案
- [ ] 能够配置监控告警
- [ ] 能够排查常见问题

---

@author erik.zhou
