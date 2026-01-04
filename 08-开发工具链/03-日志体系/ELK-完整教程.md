# ELK 完整教程

## 📋 目录
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: Elasticsearch 8.x / Logstash 8.x / Kibana 8.x
- **官方文档**: https://www.elastic.co/
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: Linux基础、日志概念、JSON、RESTful API
- **文档来源**: Context7 + 官方文档
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解ELK Stack的架构和工作原理
- [ ] 掌握Elasticsearch的安装和配置
- [ ] 掌握Logstash的日志收集和处理
- [ ] 掌握Kibana的数据可视化和查询
- [ ] 掌握Filebeat的日志采集
- [ ] 理解ELK在生产环境的部署方案
- [ ] 掌握日志分析和问题排查技巧

## 📖 基础概念

### 1.1 什么是ELK Stack

ELK Stack是由Elasticsearch、Logstash、Kibana三个开源项目组成的日志分析解决方案：

- **Elasticsearch**: 分布式搜索和分析引擎，负责存储和检索日志数据
- **Logstash**: 数据收集和处理管道，负责收集、转换、过滤日志
- **Kibana**: 数据可视化平台，提供日志查询和图表展示

**现代ELK架构**：
- **Beats**: 轻量级数据采集器（Filebeat、Metricbeat等）
- **Elasticsearch**: 核心存储和搜索引擎
- **Kibana**: 可视化和管理界面

### 1.2 核心概念

#### Elasticsearch核心概念

- **Index（索引）**: 类似数据库的表，存储相同类型的文档
- **Document（文档）**: 基本数据单元，以JSON格式存储
- **Field（字段）**: 文档中的键值对
- **Mapping（映射）**: 定义文档字段的类型和属性
- **Shard（分片）**: 索引的水平分割，提高并发性能
- **Replica（副本）**: 分片的备份，提高可用性

#### Logstash核心概念

- **Input（输入）**: 数据源（文件、TCP、Beats等）
- **Filter（过滤器）**: 数据处理和转换（grok、mutate、date等）
- **Output（输出）**: 数据目的地（Elasticsearch、文件等）
- **Pipeline（管道）**: Input → Filter → Output的完整流程

#### Kibana核心概念

- **Discover**: 日志搜索和浏览
- **Visualize**: 创建图表和可视化
- **Dashboard**: 组合多个可视化的仪表盘
- **Dev Tools**: Elasticsearch查询工具

### 1.3 应用场景
- 集中式日志管理和分析
- 应用性能监控（APM）
- 安全日志审计和分析
- 业务数据分析和报表
- 实时监控和告警
- 全文搜索和数据挖掘

## 🔥 核心特性 (重点)

### 2.1 ELK架构 🔥

```
应用服务器 → Filebeat → Logstash → Elasticsearch → Kibana
                ↓           ↓            ↓           ↓
              采集日志    处理过滤      存储索引    可视化查询
```

**数据流转过程**：
1. **Filebeat**: 监控日志文件，实时采集新增日志
2. **Logstash**: 接收日志，进行解析、过滤、转换
3. **Elasticsearch**: 存储处理后的日志，建立索引
4. **Kibana**: 提供查询界面和可视化展示

### 2.2 Elasticsearch核心特性 🔥

#### 1. 分布式架构

- **水平扩展**: 通过添加节点扩展容量
- **自动分片**: 数据自动分布到多个节点
- **高可用**: 副本机制保证数据不丢失

#### 2. 近实时搜索

- **倒排索引**: 快速全文检索
- **秒级延迟**: 数据写入后1秒内可搜索
- **复杂查询**: 支持布尔查询、范围查询、聚合分析

#### 3. RESTful API

```bash
# 创建索引
PUT /logs-2024.01.04

# 写入文档
POST /logs-2024.01.04/_doc
{
  "timestamp": "2024-01-04T10:00:00",
  "level": "ERROR",
  "message": "Database connection failed"
}

# 搜索文档
GET /logs-2024.01.04/_search
{
  "query": {
    "match": {
      "level": "ERROR"
    }
  }
}
```

### 2.3 Logstash配置 (⚠️ 难点)

Logstash使用配置文件定义数据处理管道：

#### Input插件

```ruby
input {
  # 从Filebeat接收数据
  beats {
    port => 5044
  }
  
  # 从文件读取
  file {
    path => "/var/log/app/*.log"
    start_position => "beginning"
  }
  
  # 从TCP接收
  tcp {
    port => 5000
    codec => json
  }
}
```

#### Filter插件（核心难点）

```ruby
filter {
  # Grok解析日志
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{LOGLEVEL:level}\] %{GREEDYDATA:msg}"
    }
  }
  
  # 日期解析
  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss.SSS"]
    target => "@timestamp"
  }
  
  # 字段处理
  mutate {
    remove_field => ["message"]
    add_field => {
      "env" => "production"
    }
  }
  
  # 条件判断
  if [level] == "ERROR" {
    mutate {
      add_tag => ["error"]
    }
  }
}
```

#### Output插件

```ruby
output {
  # 输出到Elasticsearch
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "logs-%{+YYYY.MM.dd}"
  }
  
  # 输出到控制台（调试用）
  stdout {
    codec => rubydebug
  }
}
```

### 2.4 Kibana查询语言（KQL）

Kibana Query Language用于在Kibana中搜索日志：

```
# 精确匹配
level: "ERROR"

# 模糊匹配
message: *connection*

# 范围查询
response_time > 1000

# 布尔查询
level: "ERROR" and service: "user-service"

# 时间范围
@timestamp >= "2024-01-04T00:00:00" and @timestamp < "2024-01-05T00:00:00"
```

### 2.5 索引生命周期管理（ILM）

ILM自动管理索引的生命周期，优化存储和性能：

**生命周期阶段**：
1. **Hot**: 频繁写入和查询（SSD存储）
2. **Warm**: 只读，偶尔查询（普通磁盘）
3. **Cold**: 很少查询（低成本存储）
4. **Delete**: 自动删除过期数据


## 💻 实战应用

### 3.1 环境搭建

#### Docker Compose快速部署

```yaml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - node.name=es-node-1
      - cluster.name=es-docker-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms2g -Xmx2g"
      - xpack.security.enabled=false
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - elk

  logstash:
    image: docker.elastic.co/logstash/logstash:8.11.0
    container_name: logstash
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    ports:
      - "5044:5044"
      - "5000:5000/tcp"
      - "5000:5000/udp"
    environment:
      LS_JAVA_OPTS: "-Xmx512m -Xms512m"
    networks:
      - elk
    depends_on:
      - elasticsearch

  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: kibana
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    ports:
      - "5601:5601"
    networks:
      - elk
    depends_on:
      - elasticsearch

  filebeat:
    image: docker.elastic.co/beats/filebeat:8.11.0
    container_name: filebeat
    user: root
    volumes:
      - ./filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml
      - /var/log:/var/log:ro
      - /var/lib/docker/containers:/var/lib/docker/containers:ro
    networks:
      - elk
    depends_on:
      - logstash

volumes:
  elasticsearch-data:
    driver: local

networks:
  elk:
    driver: bridge
```

#### Filebeat配置（filebeat.yml）

```yaml
filebeat.inputs:
  # 监控应用日志
  - type: log
    enabled: true
    paths:
      - /var/log/app/*.log
    fields:
      app: myapp
      env: production
    multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
    multiline.negate: true
    multiline.match: after

  # 监控Docker容器日志
  - type: container
    enabled: true
    paths:
      - '/var/lib/docker/containers/*/*.log'

# 输出到Logstash
output.logstash:
  hosts: ["logstash:5044"]

# 日志级别
logging.level: info
```

### 3.2 快速开始

#### Logstash管道配置（logstash.conf）

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  # 解析JSON日志
  if [message] =~ /^\{/ {
    json {
      source => "message"
    }
  }
  
  # 解析Java日志
  grok {
    match => {
      "message" => "%{TIMESTAMP_ISO8601:timestamp} \[%{DATA:thread}\] %{LOGLEVEL:level} %{DATA:logger} - %{GREEDYDATA:msg}"
    }
  }
  
  # 解析时间戳
  date {
    match => ["timestamp", "yyyy-MM-dd HH:mm:ss.SSS", "ISO8601"]
    target => "@timestamp"
  }
  
  # 添加字段
  mutate {
    add_field => {
      "[@metadata][index_prefix]" => "logs"
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "%{[@metadata][index_prefix]}-%{+YYYY.MM.dd}"
  }
  
  # 调试输出
  stdout {
    codec => rubydebug
  }
}
```

### 3.3 进阶配置

#### Java应用日志收集完整方案

**1. Logback配置（输出JSON格式）**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>/var/log/app/application.log</file>
        
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/var/log/app/application.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>7</maxHistory>
        </rollingPolicy>
        
        <!-- JSON格式输出 -->
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <customFields>{"app":"myapp","env":"production"}</customFields>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

**Maven依赖**：

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```


**2. Filebeat配置（采集日志）**

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/app/application*.log
    json.keys_under_root: true
    json.add_error_key: true
    fields:
      app: myapp
      env: production

output.logstash:
  hosts: ["logstash:5044"]
```

**3. Logstash配置（处理日志）**

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  # JSON已经被Filebeat解析，直接使用
  
  # 添加地理位置信息（如果有IP字段）
  if [client_ip] {
    geoip {
      source => "client_ip"
      target => "geoip"
    }
  }
  
  # 错误日志添加标签
  if [level] == "ERROR" {
    mutate {
      add_tag => ["error", "alert"]
    }
  }
  
  # 慢查询标记
  if [response_time] and [response_time] > 1000 {
    mutate {
      add_tag => ["slow_query"]
    }
  }
}

output {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "logs-%{[app]}-%{+YYYY.MM.dd}"
    template_name => "logs"
    template_pattern => "logs-*"
  }
}
```

#### Elasticsearch索引模板配置

```json
PUT _index_template/logs-template
{
  "index_patterns": ["logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 3,
      "number_of_replicas": 1,
      "index.lifecycle.name": "logs-policy",
      "index.lifecycle.rollover_alias": "logs"
    },
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "level": {
          "type": "keyword"
        },
        "logger": {
          "type": "keyword"
        },
        "message": {
          "type": "text",
          "fields": {
            "keyword": {
              "type": "keyword",
              "ignore_above": 256
            }
          }
        },
        "thread": {
          "type": "keyword"
        },
        "response_time": {
          "type": "long"
        }
      }
    }
  }
}
```

#### 索引生命周期策略配置

```json
PUT _ilm/policy/logs-policy
{
  "policy": {
    "phases": {
      "hot": {
        "min_age": "0ms",
        "actions": {
          "rollover": {
            "max_size": "50GB",
            "max_age": "1d"
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "shrink": {
            "number_of_shards": 1
          },
          "forcemerge": {
            "max_num_segments": 1
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

#### Kibana可视化配置

**1. 创建索引模式**

```
Management → Stack Management → Index Patterns → Create index pattern
索引模式：logs-*
时间字段：@timestamp
```

**2. 常用查询示例**

```
# 查询ERROR日志
level: "ERROR"

# 查询特定应用的日志
app: "user-service" and level: "ERROR"

# 查询慢查询
response_time > 1000

# 查询特定时间范围
@timestamp >= "2024-01-04T00:00:00" and @timestamp < "2024-01-05T00:00:00"

# 组合查询
level: "ERROR" and (app: "user-service" or app: "order-service")
```

**3. 创建可视化图表**

```
Visualize → Create visualization

常用图表类型：
- Line Chart: 日志趋势图
- Pie Chart: 日志级别分布
- Data Table: 日志详情表
- Metric: 错误数量统计
- Heat Map: 时间热力图
```

**4. 创建Dashboard**

```
Dashboard → Create dashboard → Add panels

推荐面板：
- 日志总量趋势
- 错误日志数量
- 日志级别分布
- Top 10 错误消息
- 响应时间分布
- 应用日志分布
```


## ✨ 最佳实践

### 4.1 性能优化

#### 1. Elasticsearch优化

```yaml
# elasticsearch.yml
# JVM堆内存设置（不超过物理内存的50%，不超过32GB）
ES_JAVA_OPTS: "-Xms4g -Xmx4g"

# 分片数量优化
# 单个分片大小控制在20-50GB
# 分片数 = 数据总量 / 单分片大小

# 副本数量
number_of_replicas: 1  # 生产环境至少1个副本

# 刷新间隔（提高写入性能）
index.refresh_interval: 30s  # 默认1s

# 批量写入
bulk:
  size: 5MB
  actions: 1000
```

#### 2. Logstash优化

```yaml
# logstash.yml
# 管道工作线程数
pipeline.workers: 4  # CPU核心数

# 批量大小
pipeline.batch.size: 125

# 批量延迟
pipeline.batch.delay: 50

# 队列类型（持久化队列）
queue.type: persisted
queue.max_bytes: 1GB
```

#### 3. Filebeat优化

```yaml
# filebeat.yml
# 批量发送
output.logstash:
  hosts: ["logstash:5044"]
  bulk_max_size: 2048
  worker: 2

# 日志采集优化
filebeat.inputs:
  - type: log
    paths:
      - /var/log/app/*.log
    # 扫描频率
    scan_frequency: 10s
    # 关闭文件超时
    close_inactive: 5m
```

### 4.2 配置规范

#### 1. 索引命名规范

```
# 推荐格式：{类型}-{应用}-{日期}
logs-user-service-2024.01.04
logs-order-service-2024.01.04
metrics-system-2024.01.04

# 使用别名
POST /_aliases
{
  "actions": [
    {
      "add": {
        "index": "logs-user-service-2024.01.04",
        "alias": "logs-user-service"
      }
    }
  ]
}
```

#### 2. 字段映射规范

```json
{
  "mappings": {
    "properties": {
      "@timestamp": {"type": "date"},
      "level": {"type": "keyword"},
      "message": {
        "type": "text",
        "fields": {
          "keyword": {"type": "keyword", "ignore_above": 256}
        }
      },
      "app": {"type": "keyword"},
      "env": {"type": "keyword"},
      "host": {"type": "keyword"},
      "response_time": {"type": "long"},
      "user_id": {"type": "keyword"},
      "request_id": {"type": "keyword"}
    }
  }
}
```

#### 3. Grok模式规范

```ruby
# 定义自定义Grok模式
filter {
  grok {
    patterns_dir => ["/etc/logstash/patterns"]
    match => {
      "message" => "%{CUSTOM_PATTERN}"
    }
  }
}

# /etc/logstash/patterns/custom
CUSTOM_TIMESTAMP %{YEAR}-%{MONTHNUM}-%{MONTHDAY} %{TIME}
CUSTOM_LOGLEVEL (TRACE|DEBUG|INFO|WARN|ERROR|FATAL)
CUSTOM_PATTERN %{CUSTOM_TIMESTAMP:timestamp} \[%{DATA:thread}\] %{CUSTOM_LOGLEVEL:level} %{DATA:logger} - %{GREEDYDATA:msg}
```

### 4.3 常见陷阱

#### ⚠️ 陷阱1：索引爆炸

```bash
# ❌ 错误：每个应用实例创建独立索引
logs-user-service-instance1-2024.01.04
logs-user-service-instance2-2024.01.04

# ✅ 正确：按应用和日期创建索引
logs-user-service-2024.01.04

# 使用字段区分实例
{
  "app": "user-service",
  "instance": "instance1"
}
```

#### ⚠️ 陷阱2：字段映射冲突

```bash
# ❌ 错误：同一字段不同类型
# 索引1：user_id 为 long
# 索引2：user_id 为 keyword

# ✅ 正确：使用索引模板统一字段类型
PUT _index_template/logs-template
{
  "index_patterns": ["logs-*"],
  "template": {
    "mappings": {
      "properties": {
        "user_id": {"type": "keyword"}
      }
    }
  }
}
```

#### ⚠️ 陷阱3：Logstash配置错误导致数据丢失

```ruby
# ❌ 错误：Grok解析失败时丢弃数据
filter {
  grok {
    match => {"message" => "%{PATTERN}"}
  }
}

# ✅ 正确：添加标签，保留原始数据
filter {
  grok {
    match => {"message" => "%{PATTERN}"}
    tag_on_failure => ["_grokparsefailure"]
  }
  
  # 解析失败时保留原始消息
  if "_grokparsefailure" in [tags] {
    mutate {
      add_field => {"parse_error" => "true"}
    }
  }
}
```

#### ⚠️ 陷阱4：磁盘空间不足

```bash
# 监控磁盘使用率
GET _cat/allocation?v

# 设置磁盘水位线
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.disk.watermark.low": "85%",
    "cluster.routing.allocation.disk.watermark.high": "90%",
    "cluster.routing.allocation.disk.watermark.flood_stage": "95%"
  }
}

# 配置ILM自动删除旧数据
PUT _ilm/policy/logs-policy
{
  "policy": {
    "phases": {
      "delete": {
        "min_age": "30d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

#### ⚠️ 陷阱5：查询性能差

```bash
# ❌ 错误：使用通配符查询
GET logs-*/_search
{
  "query": {
    "wildcard": {
      "message": "*error*"
    }
  }
}

# ✅ 正确：使用match查询或keyword字段
GET logs-*/_search
{
  "query": {
    "match": {
      "message": "error"
    }
  }
}

# 或者使用keyword字段精确匹配
GET logs-*/_search
{
  "query": {
    "term": {
      "level": "ERROR"
    }
  }
}
```


## ❓ 常见问题

### Q1: ELK Stack与传统日志方案的区别？

**A**: 

| 对比项 | 传统方案 | ELK Stack |
|--------|---------|-----------|
| 日志存储 | 分散在各服务器 | 集中存储 |
| 日志查询 | grep/awk命令 | 强大的搜索引擎 |
| 可视化 | 无 | Kibana图表 |
| 实时性 | 需要登录服务器 | 实时查看 |
| 扩展性 | 难以扩展 | 水平扩展 |
| 分析能力 | 有限 | 聚合分析 |

### Q2: 如何选择Filebeat还是Logstash采集日志？

**A**: 

**Filebeat优势**：
- 轻量级，资源占用少
- 部署简单
- 适合大规模部署

**Logstash优势**：
- 功能强大，支持复杂处理
- 插件丰富
- 适合复杂日志解析

**推荐方案**：
```
应用服务器 → Filebeat → Logstash → Elasticsearch
              (采集)    (处理)      (存储)
```

Filebeat负责采集，Logstash负责处理，各司其职。

### Q3: 如何处理多行日志（如Java异常堆栈）？

**A**: 

**Filebeat配置**：

```yaml
filebeat.inputs:
  - type: log
    paths:
      - /var/log/app/*.log
    # 多行配置
    multiline.type: pattern
    multiline.pattern: '^[0-9]{4}-[0-9]{2}-[0-9]{2}'
    multiline.negate: true
    multiline.match: after
    multiline.max_lines: 500
```

**Logstash配置**：

```ruby
filter {
  # 如果Filebeat未处理多行，在Logstash处理
  multiline {
    pattern => "^%{TIMESTAMP_ISO8601}"
    negate => true
    what => "previous"
  }
}
```

### Q4: 如何实现日志告警？

**A**: 

**方式1：使用Kibana Alerting**

```
Stack Management → Alerts and Insights → Rules and Connectors

创建规则：
- 条件：level: "ERROR" 
- 阈值：5分钟内超过10条
- 动作：发送邮件/Webhook
```

**方式2：使用ElastAlert**

```yaml
# elastalert_rule.yaml
name: Error Log Alert
type: frequency
index: logs-*
num_events: 10
timeframe:
  minutes: 5

filter:
  - term:
      level: "ERROR"

alert:
  - email
  - slack

email:
  - "ops@example.com"

slack:
  slack_webhook_url: "https://hooks.slack.com/services/xxx"
```

### Q5: 如何优化Elasticsearch查询性能？

**A**: 

**1. 使用时间范围过滤**

```json
GET logs-*/_search
{
  "query": {
    "bool": {
      "must": [
        {"match": {"message": "error"}}
      ],
      "filter": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-1h",
              "lte": "now"
            }
          }
        }
      ]
    }
  }
}
```

**2. 使用keyword字段**

```json
# ❌ 慢：使用text字段
{"term": {"message": "ERROR"}}

# ✅ 快：使用keyword字段
{"term": {"level": "ERROR"}}
```

**3. 限制返回字段**

```json
GET logs-*/_search
{
  "_source": ["@timestamp", "level", "message"],
  "query": {...}
}
```

**4. 使用聚合代替搜索**

```json
# 统计错误数量
GET logs-*/_search
{
  "size": 0,
  "aggs": {
    "error_count": {
      "filter": {
        "term": {"level": "ERROR"}
      }
    }
  }
}
```

### Q6: 如何备份和恢复Elasticsearch数据？

**A**: 

**1. 配置快照仓库**

```json
PUT _snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "/mount/backups/elasticsearch"
  }
}
```

**2. 创建快照**

```json
PUT _snapshot/my_backup/snapshot_1
{
  "indices": "logs-*",
  "ignore_unavailable": true,
  "include_global_state": false
}
```

**3. 恢复快照**

```json
POST _snapshot/my_backup/snapshot_1/_restore
{
  "indices": "logs-2024.01.04",
  "ignore_unavailable": true
}
```

**4. 自动快照策略**

```json
PUT _slm/policy/daily-snapshots
{
  "schedule": "0 30 1 * * ?",
  "name": "<daily-snap-{now/d}>",
  "repository": "my_backup",
  "config": {
    "indices": ["logs-*"],
    "ignore_unavailable": false,
    "include_global_state": false
  },
  "retention": {
    "expire_after": "30d",
    "min_count": 5,
    "max_count": 50
  }
}
```


## 🔗 相关资源

### 官方资源
- [Elastic官方网站](https://www.elastic.co/)
- [Elasticsearch文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html)
- [Logstash文档](https://www.elastic.co/guide/en/logstash/current/index.html)
- [Kibana文档](https://www.elastic.co/guide/en/kibana/current/index.html)
- [Filebeat文档](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)

### 推荐文章
- [ELK Stack架构详解](https://www.elastic.co/guide/en/elastic-stack/current/index.html)
- [Elasticsearch性能优化](https://www.elastic.co/guide/en/elasticsearch/reference/current/tune-for-search-speed.html)
- [Logstash Grok模式](https://www.elastic.co/guide/en/logstash/current/plugins-filters-grok.html)
- [Kibana可视化指南](https://www.elastic.co/guide/en/kibana/current/visualize.html)

### 相关技术
- [SLF4J完整教程](SLF4J-完整教程.md) - 日志门面
- [Logback完整教程](Logback-完整教程.md) - 日志框架
- [Prometheus完整教程](../04-监控体系/Prometheus-完整教程.md) - 监控系统

## 📝 学习检查清单

- [ ] 理解ELK Stack的架构和工作原理
- [ ] 掌握Elasticsearch的基本操作和查询
- [ ] 掌握Logstash的配置和日志处理
- [ ] 掌握Kibana的查询和可视化
- [ ] 掌握Filebeat的日志采集配置
- [ ] 理解Grok模式的编写方法
- [ ] 掌握索引生命周期管理
- [ ] 了解ELK性能优化技巧
- [ ] 掌握日志告警的配置方法
- [ ] 了解数据备份和恢复方案
- [ ] 能够在生产环境部署和维护ELK Stack

## 📊 ELK部署架构

### 小型部署（单机）

```
┌─────────────────────────────────┐
│      单台服务器（8GB+ RAM）        │
├─────────────────────────────────┤
│  Elasticsearch (4GB)             │
│  Logstash (2GB)                  │
│  Kibana (1GB)                    │
│  Filebeat (轻量级)                │
└─────────────────────────────────┘

适用场景：
- 开发/测试环境
- 日志量 < 10GB/天
- 单应用日志收集
```

### 中型部署（集群）

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ ES Master 1  │  │ ES Master 2  │  │ ES Master 3  │
└──────────────┘  └──────────────┘  └──────────────┘
        │                 │                 │
┌───────┴─────────────────┴─────────────────┴───────┐
│                                                    │
├──────────────┬──────────────┬──────────────┐      │
│ ES Data 1    │ ES Data 2    │ ES Data 3    │      │
│ (16GB RAM)   │ (16GB RAM)   │ (16GB RAM)   │      │
└──────────────┴──────────────┴──────────────┘      │
                                                     │
┌──────────────┬──────────────┐                     │
│ Logstash 1   │ Logstash 2   │                     │
│ (8GB RAM)    │ (8GB RAM)    │                     │
└──────────────┴──────────────┘                     │
                                                     │
┌──────────────┐                                    │
│   Kibana     │                                    │
│  (4GB RAM)   │                                    │
└──────────────┘                                    │
                                                     │
┌──────────────────────────────────────────────────┐
│         应用服务器（Filebeat）                      │
└──────────────────────────────────────────────────┘

适用场景：
- 生产环境
- 日志量 10-100GB/天
- 多应用日志收集
- 需要高可用
```

### 大型部署（分层集群）

```
┌─────────────────────────────────────────────────┐
│              负载均衡（Nginx/HAProxy）             │
└─────────────────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
┌───────▼──────┐ ┌──────▼──────┐ ┌─────▼───────┐
│ Logstash 1   │ │ Logstash 2  │ │ Logstash 3  │
│ (16GB RAM)   │ │ (16GB RAM)  │ │ (16GB RAM)  │
└──────────────┘ └─────────────┘ └─────────────┘
        │               │               │
        └───────────────┼───────────────┘
                        │
┌─────────────────────────────────────────────────┐
│           Elasticsearch Cluster                  │
├─────────────────────────────────────────────────┤
│  Master Nodes (3台)                              │
│  Hot Data Nodes (5台, SSD, 32GB RAM)            │
│  Warm Data Nodes (5台, HDD, 32GB RAM)           │
│  Cold Data Nodes (3台, HDD, 16GB RAM)           │
└─────────────────────────────────────────────────┘
                        │
┌─────────────────────────────────────────────────┐
│              Kibana Cluster (2台)                │
└─────────────────────────────────────────────────┘

适用场景：
- 大型生产环境
- 日志量 > 100GB/天
- 多租户、多应用
- 需要高可用和高性能
```

## 🎓 进阶学习路径

1. **基础阶段**（1周）
   - 理解ELK架构
   - 掌握Docker部署
   - 学习基本查询

2. **进阶阶段**（2周）
   - 掌握Logstash配置
   - 学习Grok模式
   - 掌握Kibana可视化

3. **高级阶段**（1个月）
   - 性能优化
   - 集群部署
   - 监控告警
   - 数据备份

4. **实战阶段**（持续）
   - 生产环境部署
   - 问题排查
   - 容量规划
   - 安全加固

## 🔧 常用运维命令

### Elasticsearch

```bash
# 查看集群健康状态
GET _cluster/health

# 查看节点信息
GET _cat/nodes?v

# 查看索引列表
GET _cat/indices?v

# 查看分片分配
GET _cat/shards?v

# 删除索引
DELETE logs-2024.01.01

# 强制合并段
POST logs-*/_forcemerge?max_num_segments=1

# 清理缓存
POST logs-*/_cache/clear
```

### Logstash

```bash
# 测试配置文件
bin/logstash -f config/logstash.conf --config.test_and_exit

# 启动Logstash
bin/logstash -f config/logstash.conf

# 查看管道状态
curl -XGET 'localhost:9600/_node/stats/pipelines?pretty'
```

### Filebeat

```bash
# 测试配置
filebeat test config

# 测试输出
filebeat test output

# 启动Filebeat
filebeat -e -c filebeat.yml
```

---

**文档版本**: v1.0  
**最后更新**: 2024-01-04  
**维护者**: @author erik.zhou
