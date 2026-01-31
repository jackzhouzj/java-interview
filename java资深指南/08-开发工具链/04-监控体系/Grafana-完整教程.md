# Grafana 完整教程

## 📋 目录
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: 10.0+
- **官方文档**: https://grafana.com/docs/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 时间序列数据库（如Prometheus）、HTTP协议、JSON基础
- **文档来源**: Context7 - grafana/grafana
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Grafana的架构和核心概念
- [ ] 掌握数据源的配置和管理
- [ ] 能够创建和定制仪表板
- [ ] 掌握各种可视化类型的使用
- [ ] 能够配置告警规则和通知
- [ ] 理解变量和模板的使用

## 📖 基础概念

### 1.1 什么是Grafana

Grafana是一个开源的数据可视化和监控平台，可以将来自多种数据源的数据转换为美观的图表、图形和其他可视化形式。它是云原生监控栈中不可或缺的一部分。

**核心特点**：
- 支持多种数据源（Prometheus、InfluxDB、Elasticsearch等）
- 丰富的可视化选项（图表、仪表盘、热力图等）
- 强大的告警功能
- 灵活的仪表板模板和变量
- 支持插件扩展
- 团队协作和权限管理


### 1.2 核心概念

**仪表板（Dashboard）**：
- 由一个或多个面板组成的可视化集合
- 提供相关信息的一览视图
- 可以包含变量、注释和链接

**面板（Panel）**：
- 仪表板的基本构建块
- 显示来自数据源的可视化数据
- 支持多种可视化类型（时间序列、统计、表格等）

**数据源（Data Source）**：
- Grafana从中查询数据的后端存储
- 支持Prometheus、MySQL、PostgreSQL、Elasticsearch等
- 每个数据源有特定的查询语言

**查询（Query）**：
- 从数据源获取数据的请求
- 使用数据源特定的查询语言（如PromQL、SQL）
- 可以应用转换和过滤

**变量（Variable）**：
- 仪表板中的动态占位符
- 允许创建交互式和可重用的仪表板
- 支持查询变量、自定义变量、区间变量等

### 1.3 应用场景

1. **系统监控**：监控服务器、容器、应用程序的性能指标
2. **业务分析**：可视化业务指标，如销售额、用户活跃度
3. **日志分析**：结合Loki或Elasticsearch展示日志数据
4. **IoT监控**：展示物联网设备的传感器数据
5. **告警管理**：基于指标设置告警规则，及时发现问题


## 🔥 核心特性 (重点)

### 2.1 多数据源支持 🔥

Grafana支持60+种数据源，可以在同一个仪表板中混合使用多个数据源。

**常用数据源**：
- **时间序列数据库**：Prometheus、InfluxDB、Graphite
- **关系型数据库**：MySQL、PostgreSQL、SQL Server
- **日志系统**：Loki、Elasticsearch
- **云服务**：CloudWatch、Azure Monitor、Google Cloud Monitoring
- **APM系统**：Jaeger、Tempo、Zipkin

**数据源配置示例（Prometheus）**：
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    jsonData:
      httpMethod: POST
      timeInterval: 15s
      queryTimeout: 60s
```

### 2.2 丰富的可视化类型 🔥

Grafana提供多种可视化类型，适应不同的数据展示需求。

**主要可视化类型**：

1. **Time Series（时间序列图）**：
   - 最常用的可视化类型
   - 显示随时间变化的指标
   - 支持多条线、堆叠、填充等

2. **Stat（统计面板）**：
   - 显示单个数值
   - 支持阈值颜色、趋势指示器
   - 适合显示当前状态

3. **Gauge（仪表盘）**：
   - 显示百分比或范围内的值
   - 直观展示资源使用情况

4. **Bar Chart（柱状图）**：
   - 比较不同类别的数据
   - 支持水平和垂直方向

5. **Table（表格）**：
   - 以表格形式展示数据
   - 支持排序、过滤、格式化

6. **Heatmap（热力图）**：
   - 显示数据密度和分布
   - 适合展示直方图数据

7. **Pie Chart（饼图）**：
   - 显示数据的比例关系
   - 适合展示资源分配


### 2.3 变量和模板 🔥

变量使仪表板更加灵活和可重用，用户可以通过下拉菜单动态切换数据。

**变量类型**：

1. **Query变量**：从数据源查询值
```
# Prometheus查询示例
label_values(up, job)  # 获取所有job标签的值
```

2. **Custom变量**：手动定义值列表
```
prod,staging,dev
```

3. **Interval变量**：定义时间间隔
```
1m,5m,10m,30m,1h
```

4. **Data source变量**：动态选择数据源
```
type: datasource
query: prometheus
```

**变量使用示例**：
```promql
# 在查询中使用变量
rate(http_requests_total{job="$job", instance="$instance"}[5m])
```

### 2.4 告警系统 ⚠️ 难点

Grafana的告警系统允许基于查询结果设置告警规则，并通过多种渠道发送通知。

**告警规则配置**：
```yaml
# 告警规则示例
groups:
  - name: example
    interval: 1m
    rules:
      - alert: HighCPUUsage
        expr: |
          100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}%"
```

**通知渠道**：
- Email
- Slack
- PagerDuty
- Webhook
- DingTalk（钉钉）
- WeChat（企业微信）

**难点说明**：
- 告警规则的表达式需要深入理解数据源的查询语言
- 告警阈值的设置需要根据实际情况调整
- 告警通知的去重和分组需要精心配置
- 避免告警风暴需要合理设计告警策略


### 2.5 仪表板模板和共享

Grafana支持导入导出仪表板，社区提供了大量现成的仪表板模板。

**Grafana官方仪表板库**：
- https://grafana.com/grafana/dashboards/
- 提供数千个社区贡献的仪表板
- 按数据源和用途分类

**常用仪表板模板**：
- Node Exporter Full（ID: 1860）：Linux主机监控
- Kubernetes Cluster Monitoring（ID: 7249）：K8s集群监控
- MySQL Overview（ID: 7362）：MySQL数据库监控
- Redis Dashboard（ID: 11835）：Redis监控
- Spring Boot Statistics（ID: 6756）：Spring Boot应用监控

**导入仪表板**：
```bash
# 通过ID导入
Dashboard -> Import -> 输入仪表板ID -> Load

# 通过JSON导入
Dashboard -> Import -> 上传JSON文件或粘贴JSON内容
```

## 💻 实战应用

### 3.1 环境搭建

**使用Docker快速启动**：
```bash
# 拉取Grafana镜像
docker pull grafana/grafana:latest

# 启动Grafana
docker run -d \
  --name grafana \
  -p 3000:3000 \
  -e "GF_SECURITY_ADMIN_PASSWORD=admin" \
  -v grafana-storage:/var/lib/grafana \
  grafana/grafana:latest
```

**访问Grafana**：
- URL：http://localhost:3000
- 默认用户名：admin
- 默认密码：admin（首次登录需要修改）

**使用Docker Compose**：
```yaml
version: '3.8'

services:
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning
    restart: unless-stopped

volumes:
  grafana-data:
```


### 3.2 配置Prometheus数据源

**通过UI配置**：
1. 登录Grafana
2. 点击左侧菜单 Configuration -> Data Sources
3. 点击 Add data source
4. 选择 Prometheus
5. 配置URL：http://prometheus:9090
6. 点击 Save & Test

**通过配置文件自动配置**：
```yaml
# /etc/grafana/provisioning/datasources/prometheus.yml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
    jsonData:
      httpMethod: POST
      timeInterval: 15s
      queryTimeout: 60s
      exemplarTraceIdDestinations:
        - name: trace_id
          datasourceUid: tempo
```

### 3.3 创建第一个仪表板

**步骤**：
1. 点击左侧菜单 + -> Dashboard
2. 点击 Add new panel
3. 选择数据源：Prometheus
4. 输入查询：
```promql
rate(http_requests_total[5m])
```
5. 设置面板标题：HTTP Request Rate
6. 选择可视化类型：Time series
7. 点击 Apply

**完整仪表板JSON示例**：
```json
{
  "dashboard": {
    "title": "Application Monitoring",
    "panels": [
      {
        "id": 1,
        "title": "CPU Usage",
        "type": "timeseries",
        "gridPos": {
          "h": 8,
          "w": 12,
          "x": 0,
          "y": 0
        },
        "targets": [
          {
            "expr": "100 - (avg by (instance) (irate(node_cpu_seconds_total{mode=\"idle\"}[5m])) * 100)",
            "refId": "A"
          }
        ],
        "fieldConfig": {
          "defaults": {
            "unit": "percent",
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {"color": "green", "value": null},
                {"color": "yellow", "value": 70},
                {"color": "red", "value": 90}
              ]
            }
          }
        }
      }
    ],
    "time": {
      "from": "now-6h",
      "to": "now"
    },
    "timezone": "browser"
  }
}
```


### 3.4 使用变量创建动态仪表板

**创建Query变量**：
1. Dashboard Settings -> Variables -> Add variable
2. 配置变量：
   - Name: `instance`
   - Type: Query
   - Data source: Prometheus
   - Query: `label_values(up, instance)`
   - Multi-value: 启用
   - Include All option: 启用

**在查询中使用变量**：
```promql
# 单个变量
rate(http_requests_total{instance="$instance"}[5m])

# 多个变量
rate(http_requests_total{job="$job", instance="$instance"}[5m])

# 使用All选项
rate(http_requests_total{instance=~"$instance"}[5m])
```

**变量链式依赖**：
```promql
# 变量1：job
label_values(up, job)

# 变量2：instance（依赖job）
label_values(up{job="$job"}, instance)

# 查询中使用
rate(http_requests_total{job="$job", instance="$instance"}[5m])
```

### 3.5 配置告警

**创建告警规则**：
1. 编辑面板 -> Alert标签
2. 点击 Create alert rule from this panel
3. 配置告警条件：
```
WHEN avg() OF query(A, 5m, now) IS ABOVE 80
```
4. 设置评估间隔：1m
5. 设置持续时间：5m
6. 添加标签和注释
7. 选择通知渠道

**通知渠道配置（Slack）**：
```yaml
# /etc/grafana/provisioning/notifiers/slack.yml
notifiers:
  - name: Slack
    type: slack
    uid: slack-notifier
    org_id: 1
    is_default: true
    send_reminder: true
    settings:
      url: https://hooks.slack.com/services/YOUR/WEBHOOK/URL
      recipient: '#alerts'
      username: Grafana
```

**告警规则示例（钉钉）**：
```yaml
notifiers:
  - name: DingTalk
    type: dingding
    uid: dingtalk-notifier
    settings:
      url: https://oapi.dingtalk.com/robot/send?access_token=YOUR_TOKEN
      msgtype: markdown
```


## ✨ 最佳实践

### 4.1 仪表板设计原则

**1. 遵循信息层次**：
- 最重要的指标放在顶部
- 使用大面板显示关键指标
- 相关指标分组展示

**2. 合理使用颜色**：
```json
{
  "thresholds": {
    "mode": "absolute",
    "steps": [
      {"color": "green", "value": null},      // 正常
      {"color": "yellow", "value": 70},       // 警告
      {"color": "red", "value": 90}           // 严重
    ]
  }
}
```

**3. 统一时间范围**：
- 使用仪表板级别的时间选择器
- 避免面板使用不同的时间范围
- 提供常用的时间范围快捷选项

**4. 添加说明文档**：
```markdown
# 在面板描述中添加说明
- 指标含义
- 正常范围
- 异常处理方法
- 相关文档链接
```

### 4.2 性能优化

**1. 优化查询**：
```promql
# 不推荐：查询所有实例
rate(http_requests_total[5m])

# 推荐：使用变量过滤
rate(http_requests_total{instance="$instance"}[5m])

# 推荐：使用记录规则
job:http_requests:rate5m{instance="$instance"}
```

**2. 合理设置刷新间隔**：
```
# 实时监控：5s-10s
# 一般监控：30s-1m
# 历史分析：不自动刷新
```

**3. 限制数据点数量**：
```json
{
  "maxDataPoints": 1000,  // 限制返回的数据点数量
  "interval": "30s"       // 设置最小查询间隔
}
```

**4. 使用查询缓存**：
```yaml
# grafana.ini
[dataproxy]
timeout = 30
keep_alive_seconds = 30

[caching]
enabled = true
```


### 4.3 权限管理

**组织和团队**：
```
# 组织结构
Organization (组织)
  └── Team (团队)
      └── User (用户)
```

**权限级别**：
- **Viewer**：只能查看仪表板
- **Editor**：可以编辑仪表板
- **Admin**：完全管理权限

**文件夹权限**：
```
# 为团队设置文件夹权限
1. 创建文件夹
2. 文件夹设置 -> Permissions
3. 添加团队并设置权限级别
```

### 4.4 备份和恢复

**备份仪表板**：
```bash
# 导出单个仪表板
curl -H "Authorization: Bearer YOUR_API_KEY" \
  http://localhost:3000/api/dashboards/uid/DASHBOARD_UID \
  > dashboard.json

# 导出所有仪表板
for dash in $(curl -H "Authorization: Bearer YOUR_API_KEY" \
  http://localhost:3000/api/search?type=dash-db | jq -r '.[].uid'); do
  curl -H "Authorization: Bearer YOUR_API_KEY" \
    http://localhost:3000/api/dashboards/uid/$dash \
    > ${dash}.json
done
```

**备份数据库**：
```bash
# SQLite（默认）
cp /var/lib/grafana/grafana.db /backup/grafana.db

# MySQL
mysqldump -u grafana -p grafana > grafana_backup.sql

# PostgreSQL
pg_dump -U grafana grafana > grafana_backup.sql
```

### 4.5 常见陷阱

**⚠️ 陷阱1：过度使用变量**
```
# 问题：变量过多导致加载缓慢
$region, $cluster, $namespace, $pod, $container...

# 建议：只使用必要的变量，考虑使用级联变量
$cluster -> $namespace -> $pod
```

**⚠️ 陷阱2：查询范围过大**
```promql
# 问题：查询1年的数据
rate(http_requests_total[365d])

# 建议：使用合理的时间范围
rate(http_requests_total[5m])
```

**⚠️ 陷阱3：忽略告警测试**
```
# 问题：告警规则配置后从未触发
# 建议：
1. 使用测试数据验证告警规则
2. 定期检查告警状态
3. 设置告警规则的健康检查
```


## ❓ 常见问题

### Q1: Grafana和Kibana有什么区别？

**A**: 主要区别：
- **定位**：Grafana专注于指标可视化，Kibana专注于日志分析
- **数据源**：Grafana支持多种数据源，Kibana主要用于Elasticsearch
- **可视化**：Grafana的时间序列图表更强大，Kibana的日志查询更便捷
- **生态**：Grafana与Prometheus生态集成更好，Kibana是ELK栈的一部分
- **使用场景**：Grafana适合监控指标，Kibana适合日志分析

### Q2: 如何优化Grafana的加载速度？

**A**: 优化建议：
1. **减少面板数量**：一个仪表板不超过20个面板
2. **优化查询**：使用记录规则预计算复杂查询
3. **合理设置刷新间隔**：避免过于频繁的刷新
4. **使用变量过滤**：减少查询的数据量
5. **启用查询缓存**：配置数据源缓存
6. **升级硬件**：增加内存和CPU资源

### Q3: 如何实现Grafana的高可用？

**A**: 高可用方案：
1. **使用外部数据库**：
```yaml
# grafana.ini
[database]
type = mysql
host = mysql-cluster:3306
name = grafana
user = grafana
password = password
```

2. **部署多个Grafana实例**：
```yaml
# 使用负载均衡器分发请求
nginx -> grafana-1
      -> grafana-2
      -> grafana-3
```

3. **共享存储**：
```yaml
# 使用NFS或对象存储共享插件和配置
volumes:
  - nfs-storage:/var/lib/grafana/plugins
```

### Q4: 如何在Grafana中使用SQL数据源？

**A**: SQL数据源配置：
```sql
-- MySQL查询示例
SELECT
  UNIX_TIMESTAMP(created_at) as time_sec,
  COUNT(*) as value,
  'orders' as metric
FROM orders
WHERE $__timeFilter(created_at)
GROUP BY time_sec
ORDER BY time_sec
```

**时间宏**：
- `$__timeFilter(column)`：时间范围过滤
- `$__timeFrom()`：开始时间
- `$__timeTo()`：结束时间
- `$__timeGroup(column, interval)`：时间分组


### Q5: 如何创建自定义插件？

**A**: 插件开发步骤：
```bash
# 1. 安装Grafana插件SDK
npm install -g @grafana/toolkit

# 2. 创建插件
npx @grafana/create-plugin

# 3. 选择插件类型
- Panel plugin（面板插件）
- Data source plugin（数据源插件）
- App plugin（应用插件）

# 4. 开发插件
cd my-plugin
npm install
npm run dev

# 5. 构建插件
npm run build

# 6. 安装插件
cp -r dist /var/lib/grafana/plugins/my-plugin
```

**插件配置**：
```yaml
# grafana.ini
[plugins]
allow_loading_unsigned_plugins = my-plugin
```

### Q6: 如何实现单点登录（SSO）？

**A**: SSO配置示例（OAuth）：
```ini
# grafana.ini
[auth.generic_oauth]
enabled = true
name = OAuth
allow_sign_up = true
client_id = YOUR_CLIENT_ID
client_secret = YOUR_CLIENT_SECRET
scopes = openid profile email
auth_url = https://oauth.example.com/authorize
token_url = https://oauth.example.com/token
api_url = https://oauth.example.com/userinfo
```

**LDAP集成**：
```toml
# ldap.toml
[[servers]]
host = "ldap.example.com"
port = 389
use_ssl = false
bind_dn = "cn=admin,dc=example,dc=com"
bind_password = "password"
search_filter = "(uid=%s)"
search_base_dns = ["dc=example,dc=com"]

[servers.attributes]
name = "givenName"
surname = "sn"
username = "uid"
member_of = "memberOf"
email = "mail"
```

## 🔗 相关资源

### 官方资源
- **官方文档**: https://grafana.com/docs/
- **GitHub仓库**: https://github.com/grafana/grafana
- **官方博客**: https://grafana.com/blog/
- **仪表板库**: https://grafana.com/grafana/dashboards/

### 学习资源
- **Grafana Tutorials**: https://grafana.com/tutorials/
- **Grafana Labs YouTube**: 官方视频教程
- **Awesome Grafana**: https://github.com/rtfpessoa/awesome-grafana

### 相关工具
- **Prometheus**: 时间序列数据库
- **Loki**: 日志聚合系统
- **Tempo**: 分布式追踪系统
- **Mimir**: 长期存储方案

## 📝 学习检查清单

- [ ] 理解Grafana的架构和核心概念
- [ ] 能够配置多种数据源
- [ ] 掌握创建和编辑仪表板
- [ ] 了解各种可视化类型的使用场景
- [ ] 能够使用变量创建动态仪表板
- [ ] 掌握告警规则的配置
- [ ] 了解通知渠道的设置
- [ ] 能够导入和导出仪表板
- [ ] 理解权限管理机制
- [ ] 掌握性能优化技巧
- [ ] 了解高可用部署方案
- [ ] 能够进行备份和恢复

---

**学习建议**：
1. 先在本地搭建Grafana环境，熟悉UI操作
2. 配置Prometheus数据源，创建简单的仪表板
3. 学习使用变量，创建动态仪表板
4. 导入社区仪表板，学习优秀的设计
5. 配置告警规则，测试通知功能
6. 研究生产环境的最佳实践

**下一步学习**：
- 深入学习PromQL查询语言
- 学习Loki进行日志可视化
- 了解Tempo进行链路追踪
- 学习Grafana插件开发
- 研究Grafana Cloud的高级特性
