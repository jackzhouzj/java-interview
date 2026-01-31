# Prometheus+Grafana-完整教程

> @author erik.zhou

## 📋 目录
- [技术概述](#技术概述)
- [Prometheus基础](#prometheus基础)
- [PromQL查询](#promql查询)
- [告警配置](#告警配置)
- [Grafana可视化](#grafana可视化)
- [常用监控配置](#常用监控配置)
- [最佳实践](#最佳实践)

## 📚 技术概述

### 基本信息
- **重要程度**：⭐⭐⭐⭐⭐ (P0必学)
- **难度级别**：⭐⭐⭐⭐
- **前置知识**：Linux、Docker/K8s
- **学习时长**：30-40小时
- **官方文档**：https://prometheus.io/docs/

### 学习目标
- [ ] 理解Prometheus架构
- [ ] 掌握PromQL查询语言
- [ ] 能够配置告警规则
- [ ] 能够创建Grafana仪表盘


---

## 🏗️ Prometheus基础

### 架构概览 🔥

```
┌─────────────────────────────────────────────────────────────┐
│                      Prometheus Server                       │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐  │
│  │  Retrieval  │  │    TSDB     │  │    HTTP Server      │  │
│  │  (Pull)     │  │  (Storage)  │  │    (PromQL API)     │  │
│  └──────┬──────┘  └─────────────┘  └──────────┬──────────┘  │
└─────────┼────────────────────────────────────┼──────────────┘
          │                                     │
          ▼                                     ▼
┌─────────────────────┐              ┌─────────────────────┐
│      Targets        │              │      Grafana        │
│  ┌───────────────┐  │              │   (Visualization)   │
│  │ Node Exporter │  │              └─────────────────────┘
│  │ MySQL Exporter│  │                        │
│  │ Redis Exporter│  │              ┌─────────▼───────────┐
│  │ Custom App    │  │              │   AlertManager      │
│  └───────────────┘  │              │   (Alert Routing)   │
└─────────────────────┘              └─────────────────────┘
```

### 安装部署

```yaml
# docker-compose.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:v2.47.0
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - ./prometheus/rules:/etc/prometheus/rules
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--storage.tsdb.retention.time=15d'
      - '--web.enable-lifecycle'
    restart: always

  alertmanager:
    image: prom/alertmanager:v0.26.0
    container_name: alertmanager
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager/alertmanager.yml:/etc/alertmanager/alertmanager.yml
    restart: always

  grafana:
    image: grafana/grafana:10.1.0
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin123
      - GF_USERS_ALLOW_SIGN_UP=false
    restart: always

  node-exporter:
    image: prom/node-exporter:v1.6.1
    container_name: node-exporter
    ports:
      - "9100:9100"
    volumes:
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /:/rootfs:ro
    command:
      - '--path.procfs=/host/proc'
      - '--path.sysfs=/host/sys'
      - '--path.rootfs=/rootfs'
    restart: always

volumes:
  prometheus_data:
  grafana_data:
```

### Prometheus配置 🔥

```yaml
# prometheus.yml
global:
  scrape_interval: 15s          # 采集间隔
  evaluation_interval: 15s      # 规则评估间隔
  scrape_timeout: 10s           # 采集超时

alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - alertmanager:9093

rule_files:
  - /etc/prometheus/rules/*.yml

scrape_configs:
  # Prometheus自身监控
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node Exporter
  - job_name: 'node'
    static_configs:
      - targets: 
        - '192.168.1.10:9100'
        - '192.168.1.11:9100'
        - '192.168.1.12:9100'
    relabel_configs:
      - source_labels: [__address__]
        target_label: instance
        regex: '([^:]+):\d+'
        replacement: '${1}'

  # 服务发现 - 文件
  - job_name: 'file_sd'
    file_sd_configs:
      - files:
        - /etc/prometheus/targets/*.json
        refresh_interval: 30s

  # 服务发现 - Kubernetes
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__

  # MySQL Exporter
  - job_name: 'mysql'
    static_configs:
      - targets: ['mysql-exporter:9104']

  # Redis Exporter
  - job_name: 'redis'
    static_configs:
      - targets: ['redis-exporter:9121']
```

---

## 📊 PromQL查询

### 基础语法 🔥

```promql
# 即时向量 - 查询当前值
node_cpu_seconds_total

# 带标签过滤
node_cpu_seconds_total{mode="idle"}
node_cpu_seconds_total{instance="server1", mode!="idle"}
node_cpu_seconds_total{mode=~"user|system"}    # 正则匹配
node_cpu_seconds_total{mode!~"idle|iowait"}    # 正则排除

# 范围向量 - 查询时间范围内的值
node_cpu_seconds_total[5m]      # 最近5分钟
node_cpu_seconds_total[1h]      # 最近1小时

# 时间偏移
node_cpu_seconds_total offset 1h    # 1小时前的值
```

### 常用函数 🔥

```promql
# rate - 计算增长率（用于Counter类型）
rate(node_cpu_seconds_total{mode="idle"}[5m])

# irate - 瞬时增长率
irate(node_cpu_seconds_total{mode="idle"}[5m])

# increase - 增长量
increase(http_requests_total[1h])

# sum - 求和
sum(rate(http_requests_total[5m]))
sum by (instance) (rate(http_requests_total[5m]))
sum without (cpu) (rate(node_cpu_seconds_total[5m]))

# avg - 平均值
avg(node_memory_MemAvailable_bytes)
avg by (instance) (node_load1)

# max/min
max(node_filesystem_avail_bytes)
min by (instance) (node_load1)

# count - 计数
count(up == 1)

# topk/bottomk - 前N/后N
topk(5, rate(http_requests_total[5m]))
bottomk(3, node_filesystem_avail_bytes)

# histogram_quantile - 分位数
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket[5m])))

# predict_linear - 线性预测
predict_linear(node_filesystem_avail_bytes[1h], 4*3600)  # 预测4小时后

# delta - 差值（用于Gauge类型）
delta(node_memory_MemAvailable_bytes[1h])

# absent - 检测指标是否存在
absent(up{job="myapp"})
```

### 常用监控指标 🔥

```promql
# === CPU监控 ===
# CPU使用率
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# 各模式CPU使用率
sum by (instance, mode) (irate(node_cpu_seconds_total[5m])) * 100

# === 内存监控 ===
# 内存使用率
(1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100

# 可用内存
node_memory_MemAvailable_bytes / 1024 / 1024 / 1024  # GB

# === 磁盘监控 ===
# 磁盘使用率
(1 - node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes) * 100

# 磁盘IO
rate(node_disk_read_bytes_total[5m])
rate(node_disk_written_bytes_total[5m])

# === 网络监控 ===
# 网络流量
rate(node_network_receive_bytes_total{device!~"lo|veth.*"}[5m]) * 8  # bps
rate(node_network_transmit_bytes_total{device!~"lo|veth.*"}[5m]) * 8

# === 进程监控 ===
# 进程数
node_procs_running
node_procs_blocked

# === 系统负载 ===
node_load1
node_load5
node_load15

# === HTTP监控 ===
# QPS
sum(rate(http_requests_total[5m]))

# 错误率
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100

# 响应时间P99
histogram_quantile(0.99, sum by (le) (rate(http_request_duration_seconds_bucket[5m])))
```

---

## 🚨 告警配置

### AlertManager配置 🔥

```yaml
# alertmanager.yml
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.example.com:587'
  smtp_from: 'alertmanager@example.com'
  smtp_auth_username: 'alertmanager@example.com'
  smtp_auth_password: 'password'

route:
  group_by: ['alertname', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 4h
  receiver: 'default'
  routes:
    - match:
        severity: critical
      receiver: 'critical-alerts'
      continue: true
    - match:
        severity: warning
      receiver: 'warning-alerts'
    - match_re:
        service: mysql|redis
      receiver: 'db-team'

receivers:
  - name: 'default'
    email_configs:
      - to: 'ops@example.com'

  - name: 'critical-alerts'
    email_configs:
      - to: 'oncall@example.com'
    webhook_configs:
      - url: 'http://webhook.example.com/alert'
        send_resolved: true

  - name: 'warning-alerts'
    email_configs:
      - to: 'ops@example.com'

  - name: 'db-team'
    email_configs:
      - to: 'dba@example.com'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
```

### 告警规则 🔥

```yaml
# rules/node_alerts.yml
groups:
  - name: node_alerts
    rules:
      # 实例宕机
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "实例 {{ $labels.instance }} 宕机"
          description: "{{ $labels.instance }} 已经宕机超过1分钟"

      # CPU使用率过高
      - alert: HighCpuUsage
        expr: 100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "CPU使用率过高"
          description: "{{ $labels.instance }} CPU使用率超过80%，当前值: {{ $value | printf \"%.2f\" }}%"

      # 内存使用率过高
      - alert: HighMemoryUsage
        expr: (1 - node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes) * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "内存使用率过高"
          description: "{{ $labels.instance }} 内存使用率超过85%，当前值: {{ $value | printf \"%.2f\" }}%"

      # 磁盘空间不足
      - alert: DiskSpaceLow
        expr: (1 - node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"} / node_filesystem_size_bytes) * 100 > 85
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "磁盘空间不足"
          description: "{{ $labels.instance }} {{ $labels.mountpoint }} 磁盘使用率超过85%"

      # 磁盘空间即将耗尽
      - alert: DiskSpaceCritical
        expr: predict_linear(node_filesystem_avail_bytes{fstype!~"tmpfs|overlay"}[1h], 4*3600) < 0
        for: 30m
        labels:
          severity: critical
        annotations:
          summary: "磁盘空间即将耗尽"
          description: "{{ $labels.instance }} {{ $labels.mountpoint }} 预计4小时内磁盘空间将耗尽"

  - name: application_alerts
    rules:
      # 服务错误率过高
      - alert: HighErrorRate
        expr: sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100 > 5
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "服务错误率过高"
          description: "HTTP 5xx错误率超过5%，当前值: {{ $value | printf \"%.2f\" }}%"

      # 响应时间过长
      - alert: HighLatency
        expr: histogram_quantile(0.95, sum by (le) (rate(http_request_duration_seconds_bucket[5m]))) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "响应时间过长"
          description: "P95响应时间超过1秒，当前值: {{ $value | printf \"%.2f\" }}秒"
```

---

## 📈 Grafana可视化

### 数据源配置

```yaml
# grafana/provisioning/datasources/datasources.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: false
```

### 常用面板配置

```json
// CPU使用率面板
{
  "title": "CPU Usage",
  "type": "timeseries",
  "targets": [
    {
      "expr": "100 - (avg by (instance) (irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
      "legendFormat": "{{ instance }}"
    }
  ],
  "fieldConfig": {
    "defaults": {
      "unit": "percent",
      "max": 100,
      "thresholds": {
        "mode": "absolute",
        "steps": [
          {"color": "green", "value": null},
          {"color": "yellow", "value": 70},
          {"color": "red", "value": 85}
        ]
      }
    }
  }
}
```

### 推荐Dashboard

- **Node Exporter Full**: ID 1860
- **Kubernetes Cluster**: ID 6417
- **MySQL Overview**: ID 7362
- **Redis Dashboard**: ID 763
- **Nginx**: ID 9614

---

## 💡 最佳实践

### 监控设计原则

1. **USE方法**（资源监控）
   - Utilization（使用率）
   - Saturation（饱和度）
   - Errors（错误）

2. **RED方法**（服务监控）
   - Rate（请求速率）
   - Errors（错误率）
   - Duration（响应时间）

3. **四个黄金指标**
   - 延迟（Latency）
   - 流量（Traffic）
   - 错误（Errors）
   - 饱和度（Saturation）

### 告警设计原则

1. **告警分级**：Critical、Warning、Info
2. **告警收敛**：避免告警风暴
3. **告警抑制**：高级别抑制低级别
4. **告警静默**：维护期间静默
5. **告警升级**：未处理自动升级

---

## 📝 学习检查清单

- [ ] 理解Prometheus架构
- [ ] 能够编写PromQL查询
- [ ] 能够配置告警规则
- [ ] 能够配置AlertManager
- [ ] 能够创建Grafana仪表盘
- [ ] 理解监控设计原则

---

@author erik.zhou
