# Prometheus 完整教程

## 📋 目录
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: 2.48+
- **官方文档**: https://prometheus.io/docs/
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: Linux基础、HTTP协议、时间序列数据库概念
- **文档来源**: Context7 - prometheus/prometheus
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Prometheus的架构和工作原理
- [ ] 掌握PromQL查询语言的使用
- [ ] 能够配置和部署Prometheus监控系统
- [ ] 掌握告警规则的编写和配置
- [ ] 理解服务发现机制和Exporter的使用

## 📖 基础概念

### 1.1 什么是Prometheus

Prometheus是一个开源的系统监控和告警工具包，最初由SoundCloud开发，现已成为CNCF（Cloud Native Computing Foundation）的毕业项目。它是云原生应用监控的事实标准。

**核心特点**：
- 多维数据模型（时间序列由指标名称和键值对标识）
- 灵活的查询语言PromQL
- 不依赖分布式存储，单服务器节点是自治的
- 通过HTTP拉取时间序列数据
- 支持推送时间序列数据（通过中间网关）
- 通过服务发现或静态配置发现目标
- 支持多种图形和仪表板模式


### 1.2 核心概念

**时间序列（Time Series）**：
- 由指标名称和一组标签（键值对）唯一标识
- 每个时间序列包含一系列带时间戳的样本值

**指标（Metric）**：
- Counter（计数器）：只增不减的累计指标，如请求总数
- Gauge（仪表盘）：可增可减的瞬时指标，如CPU使用率
- Histogram（直方图）：对观察结果进行采样，如请求持续时间
- Summary（摘要）：类似Histogram，但在客户端计算分位数

**标签（Label）**：
- 用于区分同一指标的不同维度
- 例如：`http_requests_total{method="GET", status="200"}`

**作业（Job）和实例（Instance）**：
- Job：一组具有相同目的的实例集合
- Instance：单个被监控的目标

### 1.3 应用场景

1. **微服务监控**：监控服务的健康状态、请求量、响应时间等
2. **基础设施监控**：监控服务器、容器、数据库等资源使用情况
3. **业务指标监控**：监控订单量、用户活跃度等业务指标
4. **告警通知**：基于监控数据触发告警，及时发现问题
5. **容量规划**：通过历史数据分析，预测资源需求


## 🔥 核心特性 (重点)

### 2.1 多维数据模型 🔥

Prometheus使用多维数据模型，每个时间序列由指标名称和标签集唯一标识。

**数据模型示例**：
```
http_requests_total{method="POST", handler="/api/users", status="200"} 1234
http_requests_total{method="GET", handler="/api/users", status="200"} 5678
```

**标签的优势**：
- 灵活的数据查询和聚合
- 支持多维度分析
- 便于服务发现和动态配置

### 2.2 PromQL查询语言 🔥

PromQL（Prometheus Query Language）是Prometheus的强大查询语言，支持复杂的数据查询和聚合。

**基本查询**：
```promql
# 查询指标
http_requests_total

# 按标签过滤
http_requests_total{job="api-server", method="GET"}

# 正则匹配
http_requests_total{status=~"2.."}
```

**范围查询**：
```promql
# 查询最近5分钟的数据
http_requests_total[5m]

# 计算每秒速率
rate(http_requests_total[5m])
```

**聚合操作**：
```promql
# 按job聚合求和
sum by (job) (rate(http_requests_total[5m]))

# 计算平均值
avg(node_cpu_seconds_total)

# 获取Top 3
topk(3, sum by (app) (rate(http_requests_total[5m])))
```


### 2.3 拉取模型（Pull Model）🔥

Prometheus采用拉取模型，主动从目标抓取指标数据。

**工作流程**：
1. Prometheus定期向配置的目标发送HTTP请求
2. 目标通过`/metrics`端点返回指标数据
3. Prometheus解析并存储数据到时间序列数据库

**优势**：
- 简化目标配置，无需知道Prometheus地址
- 便于检测目标是否存活
- 可以手动访问目标查看指标
- 避免推送模型的网络问题

### 2.4 服务发现（Service Discovery）

Prometheus支持多种服务发现机制，自动发现监控目标。

**支持的服务发现类型**：
- Kubernetes
- Consul
- EC2
- Azure
- GCE
- 文件服务发现（File SD）
- DNS服务发现

**Kubernetes服务发现示例**：
```yaml
scrape_configs:
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
```


### 2.5 告警系统（Alerting）⚠️ 难点

Prometheus的告警系统由两部分组成：Prometheus服务器中的告警规则和Alertmanager。

**告警规则示例**：
```yaml
groups:
  - name: example.rules
    interval: 1m
    rules:
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status_code=~"5.."}[5m])) by (job, instance)
          /
          sum(rate(http_requests_total[5m])) by (job, instance)
          > 0.1
        for: 10m
        labels:
          severity: page
        annotations:
          summary: "High error rate on {{ $labels.instance }}"
          description: "Error rate is {{ $value | humanizePercentage }}"
```

**告警状态**：
- Inactive：告警未激活
- Pending：告警条件满足，但未达到`for`持续时间
- Firing：告警条件满足且超过持续时间，发送到Alertmanager

**难点说明**：
- 告警规则的表达式编写需要深入理解PromQL
- 告警阈值的设置需要根据实际业务场景调整
- 避免告警风暴和告警疲劳需要精心设计

### 2.6 记录规则（Recording Rules）

记录规则用于预计算频繁使用或计算成本高的表达式，提高查询性能。

**记录规则示例**：
```yaml
groups:
  - name: example.recording
    interval: 30s
    rules:
      - record: job:http_requests_total:rate5m
        expr: sum(rate(http_requests_total[5m])) by (job, instance)
      
      - record: job:http_request_duration_seconds:p99
        expr: histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (job, le))
```


## 💻 实战应用

### 3.1 环境搭建

**使用Docker快速启动**：
```bash
# 拉取Prometheus镜像
docker pull prom/prometheus:latest

# 创建配置文件目录
mkdir -p /opt/prometheus

# 创建配置文件
cat > /opt/prometheus/prometheus.yml <<EOF
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
EOF

# 启动Prometheus
docker run -d \
  --name prometheus \
  -p 9090:9090 \
  -v /opt/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
  prom/prometheus:latest
```

**访问Web UI**：
- 浏览器访问：http://localhost:9090
- 查看目标状态：http://localhost:9090/targets
- 查看配置：http://localhost:9090/config

### 3.2 配置文件详解

**完整配置示例**：
```yaml
# 全局配置
global:
  scrape_interval: 15s          # 抓取间隔
  evaluation_interval: 15s      # 规则评估间隔
  external_labels:              # 外部标签
    cluster: 'production'
    region: 'us-east-1'

# 告警配置
alerting:
  alertmanagers:
    - static_configs:
        - targets: ['localhost:9093']

# 规则文件
rule_files:
  - 'alerts/*.yml'
  - 'rules/*.yml'

# 抓取配置
scrape_configs:
  # Prometheus自身监控
  - job_name: 'prometheus'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporter监控
  - job_name: 'node'
    static_configs:
      - targets: ['localhost:9100']
        labels:
          env: 'production'

  # 应用监控
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['app1:8080', 'app2:8080']
```


### 3.3 Spring Boot集成Prometheus

**添加依赖**：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

**配置application.yml**：
```yaml
management:
  endpoints:
    web:
      exposure:
        include: prometheus,health,info,metrics
  metrics:
    tags:
      application: ${spring.application.name}
    export:
      prometheus:
        enabled: true
```

**自定义指标**：
```java
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import org.springframework.stereotype.Service;

/**
 * 订单服务
 * @author erik.zhou
 */
@Service
public class OrderService {
    
    private final Counter orderCounter;
    private final MeterRegistry meterRegistry;
    
    public OrderService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.orderCounter = Counter.builder("orders_created_total")
                .description("Total number of orders created")
                .tag("type", "online")
                .register(meterRegistry);
    }
    
    public void createOrder(Order order) {
        // 业务逻辑
        processOrder(order);
        
        // 增加计数器
        orderCounter.increment();
    }
    
    private void processOrder(Order order) {
        // 处理订单逻辑
    }
}
```


### 3.4 常用Exporter

**Node Exporter（主机监控）**：
```bash
# 下载并启动
wget https://github.com/prometheus/node_exporter/releases/download/v1.7.0/node_exporter-1.7.0.linux-amd64.tar.gz
tar xvfz node_exporter-1.7.0.linux-amd64.tar.gz
cd node_exporter-1.7.0.linux-amd64
./node_exporter

# 访问指标
curl http://localhost:9100/metrics
```

**MySQL Exporter（数据库监控）**：
```bash
# 使用Docker启动
docker run -d \
  --name mysql-exporter \
  -p 9104:9104 \
  -e DATA_SOURCE_NAME="user:password@(mysql-host:3306)/" \
  prom/mysqld-exporter
```

**Redis Exporter（缓存监控）**：
```bash
docker run -d \
  --name redis-exporter \
  -p 9121:9121 \
  oliver006/redis_exporter \
  --redis.addr=redis://redis-host:6379
```

### 3.5 PromQL实战查询

**CPU使用率**：
```promql
# 计算CPU使用率
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

**内存使用率**：
```promql
# 计算内存使用率
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100
```

**HTTP请求QPS**：
```promql
# 计算每秒请求数
sum(rate(http_requests_total[1m])) by (job)
```

**HTTP请求P99延迟**：
```promql
# 计算99分位延迟
histogram_quantile(0.99, 
  sum(rate(http_request_duration_seconds_bucket[5m])) by (job, le)
)
```

**错误率**：
```promql
# 计算错误率
sum(rate(http_requests_total{status=~"5.."}[5m])) 
/ 
sum(rate(http_requests_total[5m]))
```


## ✨ 最佳实践

### 4.1 指标命名规范

**遵循命名约定**：
```
# 格式：<namespace>_<subsystem>_<name>_<unit>
http_requests_total              # 总请求数
http_request_duration_seconds    # 请求持续时间（秒）
process_cpu_seconds_total        # CPU使用时间（秒）
node_memory_bytes                # 内存大小（字节）
```

**命名原则**：
- 使用小写字母和下划线
- 指标名称应该描述被测量的内容
- 单位应该作为后缀（seconds, bytes, ratio等）
- 累计指标使用`_total`后缀

### 4.2 标签使用建议

**合理使用标签**：
```promql
# 好的标签设计
http_requests_total{method="GET", path="/api/users", status="200"}

# 避免高基数标签（会产生大量时间序列）
# 不推荐：使用用户ID作为标签
http_requests_total{user_id="12345"}  # ❌

# 推荐：使用用户类型
http_requests_total{user_type="premium"}  # ✅
```

**标签原则**：
- 避免使用高基数标签（如用户ID、订单号）
- 标签值应该是有限的枚举值
- 不要在标签中存储动态数据

### 4.3 性能优化

**1. 使用记录规则**：
```yaml
# 预计算复杂查询
groups:
  - name: performance
    interval: 30s
    rules:
      - record: instance:node_cpu:avg_rate5m
        expr: avg(rate(node_cpu_seconds_total[5m])) by (instance)
```

**2. 合理设置抓取间隔**：
```yaml
global:
  scrape_interval: 15s  # 默认15秒，根据需求调整

scrape_configs:
  - job_name: 'high-frequency'
    scrape_interval: 5s   # 高频监控
  
  - job_name: 'low-frequency'
    scrape_interval: 60s  # 低频监控
```

**3. 数据保留策略**：
```bash
# 启动时设置保留时间
prometheus \
  --storage.tsdb.retention.time=30d \
  --storage.tsdb.retention.size=50GB
```


### 4.4 告警设计原则

**1. 告警分级**：
```yaml
groups:
  - name: alerts
    rules:
      # 严重告警：立即处理
      - alert: ServiceDown
        expr: up{job="api-server"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.instance }} is down"
      
      # 警告告警：需要关注
      - alert: HighMemoryUsage
        expr: (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) < 0.1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
```

**2. 避免告警疲劳**：
- 设置合理的`for`持续时间，避免瞬时波动触发告警
- 使用告警分组和抑制规则
- 定期审查和优化告警规则

**3. 告警信息完整**：
```yaml
annotations:
  summary: "{{ $labels.instance }} CPU usage is {{ $value | humanizePercentage }}"
  description: |
    CPU usage has been above 80% for more than 5 minutes.
    Current value: {{ $value | humanizePercentage }}
    Instance: {{ $labels.instance }}
    Job: {{ $labels.job }}
  runbook_url: "https://wiki.company.com/runbooks/high-cpu"
```

### 4.5 常见陷阱

**⚠️ 陷阱1：时间序列爆炸**
```promql
# 错误：使用高基数标签
http_requests{request_id="uuid-12345"}  # 每个请求都会创建新的时间序列

# 正确：使用低基数标签
http_requests{endpoint="/api/users", method="GET"}
```

**⚠️ 陷阱2：查询性能问题**
```promql
# 低效查询：大范围聚合
sum(rate(http_requests_total[1h]))  # 1小时范围太大

# 高效查询：使用记录规则
sum(job:http_requests:rate5m)  # 使用预计算的5分钟速率
```

**⚠️ 陷阱3：忽略数据保留**
- 默认保留15天，生产环境建议30天以上
- 监控磁盘使用情况，避免存储空间不足
- 考虑使用远程存储（如Thanos、Cortex）


## ❓ 常见问题

### Q1: Prometheus和Zabbix有什么区别？

**A**: 主要区别：
- **数据模型**：Prometheus使用多维时间序列，Zabbix使用传统的主机-项目模型
- **采集方式**：Prometheus采用拉取模型，Zabbix支持推拉两种模式
- **查询语言**：Prometheus有强大的PromQL，Zabbix查询相对简单
- **适用场景**：Prometheus更适合云原生和微服务，Zabbix更适合传统IT基础设施
- **生态系统**：Prometheus与Kubernetes、Grafana等云原生工具集成更好

### Q2: 如何处理Prometheus的高可用？

**A**: 高可用方案：
1. **联邦集群**：多个Prometheus实例，通过联邦机制聚合数据
2. **Thanos**：提供长期存储、全局查询和高可用能力
3. **Cortex**：多租户、水平扩展的Prometheus即服务
4. **VictoriaMetrics**：高性能、低成本的Prometheus替代方案

**简单高可用配置**：
```yaml
# 部署两个相同配置的Prometheus实例
# 使用Alertmanager的集群模式去重告警
```

### Q3: 如何监控Prometheus自身？

**A**: 监控Prometheus：
```yaml
# 在配置中添加自监控
scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

**关键指标**：
```promql
# 抓取目标数量
prometheus_sd_discovered_targets

# 抓取持续时间
prometheus_target_scrape_duration_seconds

# TSDB大小
prometheus_tsdb_storage_blocks_bytes

# 查询持续时间
prometheus_engine_query_duration_seconds
```

### Q4: 如何优化PromQL查询性能？

**A**: 优化建议：
1. **使用记录规则**：预计算复杂查询
2. **减少查询范围**：避免过大的时间范围
3. **使用合适的函数**：`rate()`比`increase()`更高效
4. **避免高基数聚合**：不要在高基数标签上聚合
5. **使用子查询谨慎**：子查询会显著增加计算量


### Q5: Counter和Gauge应该如何选择？

**A**: 选择原则：
- **Counter**：用于只增不减的指标
  - 请求总数、错误总数、任务完成数
  - 使用`rate()`或`increase()`函数查询
  
- **Gauge**：用于可增可减的指标
  - CPU使用率、内存使用量、队列长度、温度
  - 直接查询当前值或使用`avg_over_time()`等函数

**示例**：
```java
// Counter示例
Counter requestCounter = Counter.builder("http_requests_total")
    .description("Total HTTP requests")
    .register(registry);
requestCounter.increment();

// Gauge示例
Gauge queueSize = Gauge.builder("queue_size", queue, Queue::size)
    .description("Current queue size")
    .register(registry);
```

### Q6: 如何处理Prometheus的数据迁移？

**A**: 数据迁移方案：
1. **快照备份**：
```bash
# 创建快照
curl -XPOST http://localhost:9090/api/v1/admin/tsdb/snapshot

# 快照位置：data/snapshots/<snapshot-name>
```

2. **远程存储**：
```yaml
# 配置远程写入
remote_write:
  - url: "http://remote-storage:9201/write"

# 配置远程读取
remote_read:
  - url: "http://remote-storage:9201/read"
```

3. **使用Thanos或Cortex**：提供长期存储和数据迁移能力

## 🔗 相关资源

### 官方资源
- **官方文档**: https://prometheus.io/docs/
- **GitHub仓库**: https://github.com/prometheus/prometheus
- **官方博客**: https://prometheus.io/blog/

### 学习资源
- **Prometheus Up & Running** (书籍)
- **PromLabs**: https://promlabs.com/ (官方培训)
- **Awesome Prometheus**: https://github.com/roaldnefs/awesome-prometheus

### 相关工具
- **Grafana**: 可视化工具，与Prometheus完美集成
- **Alertmanager**: 告警管理和路由
- **Thanos**: 高可用和长期存储方案
- **VictoriaMetrics**: 高性能Prometheus兼容方案

## 📝 学习检查清单

- [ ] 理解Prometheus的架构和工作原理
- [ ] 掌握四种指标类型的使用场景
- [ ] 能够编写基本的PromQL查询
- [ ] 掌握聚合函数和范围查询
- [ ] 能够配置Prometheus抓取目标
- [ ] 理解服务发现机制
- [ ] 能够编写告警规则和记录规则
- [ ] 掌握Spring Boot集成Prometheus
- [ ] 了解常用Exporter的使用
- [ ] 理解高可用和数据持久化方案
- [ ] 掌握性能优化和最佳实践
- [ ] 能够排查常见问题

---

**学习建议**：
1. 先在本地搭建Prometheus环境，熟悉Web UI
2. 学习PromQL，从简单查询开始逐步深入
3. 实践Spring Boot集成，监控自己的应用
4. 学习编写告警规则，理解告警机制
5. 研究生产环境的高可用方案
6. 结合Grafana学习数据可视化

**下一步学习**：
- 学习Grafana进行数据可视化
- 学习Alertmanager进行告警管理
- 了解Thanos实现长期存储
- 学习Kubernetes中的Prometheus Operator
