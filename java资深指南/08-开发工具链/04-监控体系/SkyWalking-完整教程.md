# SkyWalking 完整教程

## 📋 目录
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: 9.5+
- **官方文档**: https://skywalking.apache.org/docs/
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 分布式系统、微服务架构、Java基础、网络协议
- **文档来源**: Apache SkyWalking官方文档
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解SkyWalking的架构和工作原理
- [ ] 掌握Agent的部署和配置
- [ ] 能够进行分布式链路追踪
- [ ] 掌握性能分析和问题定位
- [ ] 理解服务拓扑和依赖分析
- [ ] 能够配置告警规则
- [ ] 掌握日志和指标的关联分析

## 📖 基础概念

### 1.1 什么是SkyWalking

Apache SkyWalking是一个开源的应用性能监控（APM）系统，专为微服务、云原生和容器化架构设计。它提供分布式追踪、服务网格遥测分析、指标聚合和可视化一体化解决方案。

**核心特点**：
- 分布式链路追踪
- 服务性能指标监控
- 服务拓扑图自动生成
- 服务依赖分析
- 告警功能
- 日志分析
- 支持多语言（Java、.NET、Node.js、Python等）
- 无侵入式探针（Java Agent）


### 1.2 核心概念

**服务（Service）**：
- 一组提供相同功能的工作负载
- 例如：订单服务、用户服务、支付服务

**服务实例（Service Instance）**：
- 服务的一个运行实例
- 例如：订单服务的3个Pod实例

**端点（Endpoint）**：
- 服务中的一个具体操作
- 例如：HTTP接口、RPC方法、MQ消费者

**追踪（Trace）**：
- 一次完整的请求调用链路
- 由多个Span组成

**跨度（Span）**：
- 追踪中的一个操作单元
- 包含开始时间、结束时间、标签等信息

**拓扑（Topology）**：
- 服务之间的调用关系图
- 自动生成，实时更新

### 1.3 架构组件

**SkyWalking Agent**：
- 基于JavaAgent技术的探针
- 无侵入式数据采集
- 自动追踪调用链路

**SkyWalking OAP（Observability Analysis Platform）**：
- 后端分析平台
- 接收Agent发送的数据
- 进行数据聚合和分析
- 提供查询API

**SkyWalking UI**：
- Web可视化界面
- 展示服务拓扑、追踪、指标等
- 提供告警管理

**Storage**：
- 数据存储层
- 支持Elasticsearch、MySQL、TiDB、H2等


### 1.4 应用场景

1. **分布式链路追踪**：追踪请求在微服务间的完整调用路径
2. **性能瓶颈分析**：识别慢查询、慢接口、慢服务
3. **服务依赖分析**：了解服务间的依赖关系
4. **故障定位**：快速定位异常和错误
5. **容量规划**：基于性能数据进行容量评估
6. **SLA监控**：监控服务可用性和响应时间

## 🔥 核心特性 (重点)

### 2.1 分布式链路追踪 🔥

SkyWalking的核心功能是分布式链路追踪，可以追踪请求在微服务架构中的完整调用路径。

**追踪原理**：
```
用户请求 -> API Gateway -> 订单服务 -> 库存服务 -> 数据库
                         -> 支付服务 -> 第三方支付
```

**Trace结构**：
```
Trace ID: 1234567890
├── Span 1: API Gateway [0-100ms]
│   └── Span 2: 订单服务 [10-90ms]
│       ├── Span 3: 库存服务 [20-40ms]
│       │   └── Span 4: MySQL查询 [25-35ms]
│       └── Span 5: 支付服务 [50-80ms]
│           └── Span 6: 第三方支付API [55-75ms]
```

**追踪信息包含**：
- TraceId：全局唯一的追踪ID
- SpanId：Span的唯一标识
- ParentSpanId：父Span的ID
- 开始时间和结束时间
- 服务名称和实例
- 端点名称
- 标签（Tags）
- 日志（Logs）
- 错误信息

### 2.2 服务拓扑图 🔥

SkyWalking自动生成服务拓扑图，展示服务间的调用关系和性能指标。

**拓扑图信息**：
- 服务节点：显示服务名称和健康状态
- 调用关系：箭头表示调用方向
- 调用量：显示每分钟调用次数（CPM）
- 响应时间：显示平均响应时间
- 错误率：显示错误请求的百分比

**拓扑图类型**：
- 全局拓扑：所有服务的调用关系
- 服务拓扑：单个服务的上下游关系
- 实例拓扑：服务实例级别的调用关系


### 2.3 性能指标监控 🔥

SkyWalking收集和展示多维度的性能指标。

**服务级别指标**：
- SLA（Service Level Agreement）：服务可用性
- CPM（Calls Per Minute）：每分钟调用次数
- 平均响应时间
- P50、P75、P90、P95、P99响应时间
- 错误率

**实例级别指标**：
- JVM指标：堆内存、非堆内存、GC次数、GC时间
- CPU使用率
- 线程数
- 类加载数

**端点级别指标**：
- 每个接口的调用量
- 每个接口的响应时间
- 每个接口的错误率

**数据库指标**：
- SQL执行次数
- SQL执行时间
- 慢SQL识别

### 2.4 探针原理 ⚠️ 难点

SkyWalking使用Java Agent技术实现无侵入式监控。

**工作原理**：
```java
// 1. JVM启动时加载Agent
java -javaagent:/path/to/skywalking-agent.jar -jar app.jar

// 2. Agent使用字节码增强技术（Byte Buddy）
// 拦截目标方法，在方法执行前后插入追踪代码

// 原始代码
public String getUserInfo(Long userId) {
    return userService.getUser(userId);
}

// Agent增强后的逻辑（伪代码）
public String getUserInfo(Long userId) {
    Span span = createSpan("getUserInfo");
    try {
        String result = userService.getUser(userId);
        span.setSuccess();
        return result;
    } catch (Exception e) {
        span.setError(e);
        throw e;
    } finally {
        span.finish();
    }
}
```

**支持的框架和组件**：
- Web框架：Spring MVC、Spring WebFlux、Servlet
- RPC框架：Dubbo、gRPC、Feign
- 数据库：MySQL、PostgreSQL、MongoDB、Redis
- 消息队列：Kafka、RabbitMQ、RocketMQ
- HTTP客户端：HttpClient、OkHttp、RestTemplate

**难点说明**：
- 字节码增强技术较为复杂，需要深入理解JVM
- 探针会对应用性能产生一定影响（通常<5%）
- 某些框架可能需要特殊配置才能正确追踪
- 自定义插件开发需要掌握SkyWalking插件机制


### 2.5 告警功能

SkyWalking提供灵活的告警规则配置和多种通知方式。

**告警规则示例**：
```yaml
# alarm-settings.yml
rules:
  # 服务响应时间告警
  service_resp_time_rule:
    metrics-name: service_resp_time
    op: ">"
    threshold: 1000
    period: 10
    count: 3
    silence-period: 5
    message: "服务 {name} 响应时间超过1秒"
  
  # 服务错误率告警
  service_sla_rule:
    metrics-name: service_sla
    op: "<"
    threshold: 95
    period: 10
    count: 2
    message: "服务 {name} SLA低于95%"
  
  # 端点响应时间告警
  endpoint_resp_time_rule:
    metrics-name: endpoint_resp_time
    op: ">"
    threshold: 2000
    period: 10
    count: 3
    message: "端点 {name} 响应时间超过2秒"
```

**通知方式**：
- Webhook
- gRPC
- 钉钉
- 企业微信
- Slack
- 邮件

## 💻 实战应用

### 3.1 环境搭建

**使用Docker快速部署**：
```bash
# 1. 拉取镜像
docker pull apache/skywalking-oap-server:9.5.0
docker pull apache/skywalking-ui:9.5.0
docker pull elasticsearch:7.17.0

# 2. 启动Elasticsearch
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -e "discovery.type=single-node" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  elasticsearch:7.17.0

# 3. 启动OAP Server
docker run -d \
  --name oap-server \
  --link elasticsearch:elasticsearch \
  -e SW_STORAGE=elasticsearch \
  -e SW_STORAGE_ES_CLUSTER_NODES=elasticsearch:9200 \
  -p 11800:11800 \
  -p 12800:12800 \
  apache/skywalking-oap-server:9.5.0

# 4. 启动UI
docker run -d \
  --name skywalking-ui \
  --link oap-server:oap-server \
  -e SW_OAP_ADDRESS=http://oap-server:12800 \
  -p 8080:8080 \
  apache/skywalking-ui:9.5.0
```

**访问SkyWalking UI**：
- URL：http://localhost:8080
- 无需登录，直接访问


**使用Docker Compose部署**：
```yaml
version: '3.8'

services:
  elasticsearch:
    image: elasticsearch:7.17.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ports:
      - "9200:9200"
    volumes:
      - es-data:/usr/share/elasticsearch/data

  oap-server:
    image: apache/skywalking-oap-server:9.5.0
    container_name: oap-server
    depends_on:
      - elasticsearch
    environment:
      - SW_STORAGE=elasticsearch
      - SW_STORAGE_ES_CLUSTER_NODES=elasticsearch:9200
      - SW_HEALTH_CHECKER=default
      - SW_TELEMETRY=prometheus
      - JAVA_OPTS=-Xms512m -Xmx512m
    ports:
      - "11800:11800"
      - "12800:12800"

  skywalking-ui:
    image: apache/skywalking-ui:9.5.0
    container_name: skywalking-ui
    depends_on:
      - oap-server
    environment:
      - SW_OAP_ADDRESS=http://oap-server:12800
      - SW_ZIPKIN_ADDRESS=http://oap-server:9412
    ports:
      - "8080:8080"

volumes:
  es-data:
```

### 3.2 配置Java Agent

**下载Agent**：
```bash
# 下载SkyWalking Agent
wget https://archive.apache.org/dist/skywalking/9.5.0/apache-skywalking-apm-9.5.0.tar.gz
tar -zxvf apache-skywalking-apm-9.5.0.tar.gz
```

**配置Agent**：
```bash
# 编辑配置文件
vi apache-skywalking-apm-bin/agent/config/agent.config

# 关键配置项
agent.service_name=${SW_AGENT_NAME:order-service}  # 服务名称
collector.backend_service=${SW_AGENT_COLLECTOR_BACKEND_SERVICES:127.0.0.1:11800}  # OAP地址
agent.sample_n_per_3_secs=${SW_AGENT_SAMPLE:-1}  # 采样率（-1表示全量采集）
logging.level=${SW_LOGGING_LEVEL:INFO}  # 日志级别
```

**启动应用**：
```bash
# 方式1：命令行参数
java -javaagent:/path/to/skywalking-agent.jar \
     -Dskywalking.agent.service_name=order-service \
     -Dskywalking.collector.backend_service=127.0.0.1:11800 \
     -jar order-service.jar

# 方式2：环境变量
export SW_AGENT_NAME=order-service
export SW_AGENT_COLLECTOR_BACKEND_SERVICES=127.0.0.1:11800
java -javaagent:/path/to/skywalking-agent.jar -jar order-service.jar
```


### 3.3 Spring Boot集成

**Maven依赖**（可选，用于手动埋点）：
```xml
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-trace</artifactId>
    <version>9.0.0</version>
</dependency>
```

**手动埋点示例**：
```java
import org.apache.skywalking.apm.toolkit.trace.Trace;
import org.apache.skywalking.apm.toolkit.trace.TraceContext;
import org.springframework.stereotype.Service;

/**
 * 订单服务
 * @author erik.zhou
 */
@Service
public class OrderService {
    
    /**
     * 创建订单
     * @Trace注解会创建一个新的Span
     */
    @Trace
    public Order createOrder(OrderRequest request) {
        // 获取当前TraceId
        String traceId = TraceContext.traceId();
        System.out.println("TraceId: " + traceId);
        
        // 业务逻辑
        Order order = new Order();
        order.setOrderNo(generateOrderNo());
        order.setUserId(request.getUserId());
        
        // 调用其他服务
        checkInventory(order);
        processPayment(order);
        
        return order;
    }
    
    @Trace
    private void checkInventory(Order order) {
        // 库存检查逻辑
    }
    
    @Trace
    private void processPayment(Order order) {
        // 支付处理逻辑
    }
    
    private String generateOrderNo() {
        return "ORD" + System.currentTimeMillis();
    }
}
```

**添加自定义标签**：
```java
import org.apache.skywalking.apm.toolkit.trace.ActiveSpan;

@Service
public class UserService {
    
    @Trace
    public User getUser(Long userId) {
        // 添加自定义标签
        ActiveSpan.tag("userId", String.valueOf(userId));
        ActiveSpan.tag("operation", "getUser");
        
        User user = userRepository.findById(userId);
        
        if (user == null) {
            // 记录错误
            ActiveSpan.error(new RuntimeException("User not found"));
        }
        
        return user;
    }
}
```

### 3.4 查看追踪数据

**查看服务拓扑**：
1. 访问SkyWalking UI
2. 点击"Topology"菜单
3. 选择时间范围
4. 查看服务调用关系图

**查看追踪详情**：
1. 点击"Trace"菜单
2. 设置查询条件：
   - 服务名称
   - 端点名称
   - 追踪状态（成功/失败）
   - 响应时间范围
3. 点击具体的Trace查看详细信息
4. 查看Span列表和时间线

**追踪详情包含**：
- 完整的调用链路
- 每个Span的开始时间和持续时间
- 服务和实例信息
- 标签和日志
- 异常堆栈（如果有）


### 3.5 配置告警

**配置钉钉告警**：
```yaml
# alarm-settings.yml
dingtalkHooks:
  textTemplate: |-
    {
      "msgtype": "markdown",
      "markdown": {
        "title": "SkyWalking告警",
        "text": "## SkyWalking告警\n\n**告警规则**: %s\n\n**告警消息**: %s\n\n**告警时间**: %s"
      }
    }
  webhooks:
    - url: https://oapi.dingtalk.com/robot/send?access_token=YOUR_TOKEN
```

**配置企业微信告警**：
```yaml
# alarm-settings.yml
wechatHooks:
  textTemplate: |-
    {
      "msgtype": "markdown",
      "markdown": {
        "content": "## SkyWalking告警\n\n**告警规则**: %s\n\n**告警消息**: %s\n\n**告警时间**: %s"
      }
    }
  webhooks:
    - url: https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=YOUR_KEY
```

**自定义告警规则**：
```yaml
rules:
  # 慢接口告警
  slow_endpoint_rule:
    metrics-name: endpoint_resp_time
    op: ">"
    threshold: 3000
    period: 5
    count: 2
    silence-period: 10
    message: "端点 {name} 响应时间超过3秒，当前值: {value}ms"
    tags:
      level: warning
  
  # JVM内存告警
  jvm_memory_rule:
    metrics-name: instance_jvm_memory_heap_used
    op: ">"
    threshold: 80
    period: 10
    count: 3
    message: "实例 {name} JVM堆内存使用率超过80%"
    tags:
      level: critical
```

## ✨ 最佳实践

### 4.1 Agent配置优化

**1. 采样率配置**：
```properties
# 生产环境建议配置采样率，减少性能影响
agent.sample_n_per_3_secs=3  # 每3秒采样3次

# 或使用百分比采样
agent.sample_percentage=10  # 采样10%的请求
```

**2. 插件选择性启用**：
```properties
# 禁用不需要的插件
plugin.mount=default,mysql,redis,kafka
plugin.exclude_plugins=mongodb,elasticsearch
```

**3. 日志配置**：
```properties
# 生产环境使用WARN级别
logging.level=WARN
logging.max_file_size=100MB
logging.max_history_files=5
```

### 4.2 性能影响最小化

**Agent性能影响**：
- CPU：增加1-3%
- 内存：增加50-100MB
- 响应时间：增加1-5ms

**优化建议**：
1. 使用采样而非全量采集
2. 禁用不需要的插件
3. 调整批量发送参数
4. 使用异步发送模式

**批量发送配置**：
```properties
# 批量发送配置
collector.buffer_size=5000
collector.batch_size=3000
collector.get_profile_task_interval=20
```


### 4.3 存储优化

**Elasticsearch配置**：
```yaml
# application.yml
storage:
  elasticsearch:
    namespace: ${SW_NAMESPACE:"skywalking"}
    clusterNodes: ${SW_STORAGE_ES_CLUSTER_NODES:localhost:9200}
    protocol: ${SW_STORAGE_ES_HTTP_PROTOCOL:"http"}
    connectTimeout: ${SW_STORAGE_ES_CONNECT_TIMEOUT:3000}
    socketTimeout: ${SW_STORAGE_ES_SOCKET_TIMEOUT:30000}
    responseTimeout: ${SW_STORAGE_ES_RESPONSE_TIMEOUT:15000}
    numHttpClientThread: ${SW_STORAGE_ES_NUM_HTTP_CLIENT_THREAD:0}
    user: ${SW_ES_USER:""}
    password: ${SW_ES_PASSWORD:""}
    trustStorePath: ${SW_STORAGE_ES_SSL_JKS_PATH:""}
    trustStorePass: ${SW_STORAGE_ES_SSL_JKS_PASS:""}
    secretsManagementFile: ${SW_ES_SECRETS_MANAGEMENT_FILE:""} # Secrets management file in the properties format includes the username, password, which are managed by 3rd party tool.
    dayStep: ${SW_STORAGE_DAY_STEP:1} # Represent the number of days in the one minute/hour/day index.
    indexShardsNumber: ${SW_STORAGE_ES_INDEX_SHARDS_NUMBER:1} # Shard number of new indexes
    indexReplicasNumber: ${SW_STORAGE_ES_INDEX_REPLICAS_NUMBER:1} # Replicas number of new indexes
    # Specify the settings for each index individually.
    # If configured, this setting has the highest priority and overrides the generic settings.
    specificIndexSettings: ${SW_STORAGE_ES_SPECIFIC_INDEX_SETTINGS:""}
    superDatasetDayStep: ${SW_SUPERDATASET_STORAGE_DAY_STEP:-1} # Represent the number of days in the super size dataset record index, the default value is the same as dayStep when the value is less than 0
    superDatasetIndexShardsFactor: ${SW_STORAGE_ES_SUPER_DATASET_INDEX_SHARDS_FACTOR:5} #  This factor provides more shards for the super size dataset, shards number = indexShardsNumber * superDatasetIndexShardsFactor. Also, this factor effects Zipkin and Jaeger traces.
    superDatasetIndexReplicasNumber: ${SW_STORAGE_ES_SUPER_DATASET_INDEX_REPLICAS_NUMBER:0} # Represent the replicas number in the super size dataset record index, the default value is 0.
    indexTemplateOrder: ${SW_STORAGE_ES_INDEX_TEMPLATE_ORDER:0} # the order of index template
    bulkActions: ${SW_STORAGE_ES_BULK_ACTIONS:5000} # Execute the async bulk record data every ${SW_STORAGE_ES_BULK_ACTIONS} requests
    batchOfBytes: ${SW_STORAGE_ES_BATCH_OF_BYTES:10485760} # A threshold to control the max body size of ElasticSearch Bulk flush.
    flushInterval: ${SW_STORAGE_ES_FLUSH_INTERVAL:15} # flush the bulk every 15 seconds whatever the number of requests
    concurrentRequests: ${SW_STORAGE_ES_CONCURRENT_REQUESTS:2} # the number of concurrent requests
    resultWindowMaxSize: ${SW_STORAGE_ES_QUERY_MAX_WINDOW_SIZE:10000}
    metadataQueryMaxSize: ${SW_STORAGE_ES_QUERY_MAX_SIZE:10000}
    scrollingBatchSize: ${SW_STORAGE_ES_SCROLLING_BATCH_SIZE:5000}
    segmentQueryMaxSize: ${SW_STORAGE_ES_QUERY_SEGMENT_SIZE:200}
    profileTaskQueryMaxSize: ${SW_STORAGE_ES_QUERY_PROFILE_TASK_SIZE:200}
    oapAnalyzer: ${SW_STORAGE_ES_OAP_ANALYZER:"{\"analyzer\":{\"oap_analyzer\":{\"type\":\"stop\"}}}"} # the oap analyzer.
    oapLogAnalyzer: ${SW_STORAGE_ES_OAP_LOG_ANALYZER:"{\"analyzer\":{\"oap_log_analyzer\":{\"type\":\"standard\"}}}"} # the oap log analyzer. It could be customized by the ES analyzer configuration to support more language log formats, such as Chinese log, Japanese log and etc.
    advanced: ${SW_STORAGE_ES_ADVANCED:""}
```

**数据保留策略**：
```yaml
# TTL配置
core:
  default:
    # 记录数据保留7天
    recordDataTTL: ${SW_CORE_RECORD_DATA_TTL:7}
    # 指标数据保留7天
    metricsDataTTL: ${SW_CORE_METRICS_DATA_TTL:7}
```

### 4.4 告警策略设计

**告警分级**：
```yaml
rules:
  # P0：核心服务不可用
  service_down_rule:
    metrics-name: service_sla
    op: "<"
    threshold: 50
    period: 1
    count: 1
    message: "[P0] 服务 {name} 不可用"
    tags:
      level: critical
  
  # P1：服务性能严重下降
  service_slow_rule:
    metrics-name: service_resp_time
    op: ">"
    threshold: 5000
    period: 5
    count: 3
    message: "[P1] 服务 {name} 响应时间严重超标"
    tags:
      level: high
  
  # P2：服务性能下降
  service_warning_rule:
    metrics-name: service_resp_time
    op: ">"
    threshold: 2000
    period: 10
    count: 5
    message: "[P2] 服务 {name} 响应时间超标"
    tags:
      level: warning
```

### 4.5 常见陷阱

**⚠️ 陷阱1：全量采集导致性能问题**
```
# 问题：生产环境全量采集
agent.sample_n_per_3_secs=-1

# 解决：使用采样
agent.sample_n_per_3_secs=3
```

**⚠️ 陷阱2：存储空间不足**
```
# 问题：数据保留时间过长
recordDataTTL: 30
metricsDataTTL: 30

# 解决：根据实际需求调整
recordDataTTL: 7
metricsDataTTL: 7
```

**⚠️ 陷阱3：忽略Agent版本兼容性**
```
# 问题：Agent版本与OAP版本不匹配
Agent: 8.x
OAP: 9.x

# 解决：保持版本一致
Agent: 9.5.0
OAP: 9.5.0
```


## ❓ 常见问题

### Q1: SkyWalking和Zipkin有什么区别？

**A**: 主要区别：
- **功能范围**：SkyWalking提供完整的APM解决方案（追踪+指标+日志），Zipkin只专注于分布式追踪
- **探针方式**：SkyWalking使用无侵入式Agent，Zipkin需要代码埋点
- **UI功能**：SkyWalking提供更丰富的可视化和分析功能
- **性能指标**：SkyWalking自动收集JVM、服务、端点等多维度指标
- **告警功能**：SkyWalking内置告警系统，Zipkin需要第三方工具
- **生态系统**：SkyWalking更适合Java生态，Zipkin更通用

### Q2: 如何减少Agent对应用的性能影响？

**A**: 优化建议：
```properties
# 1. 使用采样
agent.sample_n_per_3_secs=3

# 2. 禁用不需要的插件
plugin.exclude_plugins=mongodb,elasticsearch

# 3. 调整批量发送
collector.buffer_size=5000
collector.batch_size=3000

# 4. 降低日志级别
logging.level=WARN

# 5. 使用异步发送
collector.is_resolve_dns_periodically=true
```

**性能测试结果**：
- 默认配置：CPU +3%，内存 +100MB，响应时间 +5ms
- 优化配置：CPU +1%，内存 +50MB，响应时间 +2ms

### Q3: 如何追踪跨线程的异步调用？

**A**: 使用@TraceCrossThread注解：
```java
import org.apache.skywalking.apm.toolkit.trace.TraceCrossThread;
import java.util.concurrent.CompletableFuture;

@Service
public class AsyncService {
    
    @Trace
    public void processAsync() {
        // 使用@TraceCrossThread标记异步任务
        CompletableFuture.runAsync(new TracedTask());
    }
    
    @TraceCrossThread
    static class TracedTask implements Runnable {
        @Override
        public void run() {
            // 这里的操作会被追踪
            System.out.println("Async task");
        }
    }
}
```

### Q4: 如何集成日志系统？

**A**: 集成Logback：
```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.apache.skywalking</groupId>
    <artifactId>apm-toolkit-logback-1.x</artifactId>
    <version>9.0.0</version>
</dependency>
```

```xml
<!-- logback.xml -->
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
            <layout class="org.apache.skywalking.apm.toolkit.log.logback.v1.x.TraceIdPatternLogbackLayout">
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%tid] [%thread] %-5level %logger{36} - %msg%n</pattern>
            </layout>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
    </root>
</configuration>
```

**日志中会自动包含TraceId**：
```
2024-01-04 10:30:45.123 [TID:1234567890] [http-nio-8080-exec-1] INFO  c.e.OrderService - 创建订单成功
```


### Q5: 如何在Kubernetes中部署SkyWalking？

**A**: 使用Helm部署：
```bash
# 1. 添加Helm仓库
helm repo add skywalking https://apache.jfrog.io/artifactory/skywalking-helm
helm repo update

# 2. 创建values.yaml
cat > values.yaml <<EOF
oap:
  image:
    tag: 9.5.0
  replicas: 2
  resources:
    requests:
      memory: 2Gi
      cpu: 1000m
    limits:
      memory: 4Gi
      cpu: 2000m
  storageType: elasticsearch
  
ui:
  image:
    tag: 9.5.0
  replicas: 1

elasticsearch:
  enabled: true
  replicas: 3
  minimumMasterNodes: 2
EOF

# 3. 安装SkyWalking
helm install skywalking skywalking/skywalking -f values.yaml -n skywalking --create-namespace

# 4. 配置应用使用SkyWalking
# 在Deployment中添加initContainer下载Agent
```

**Kubernetes Deployment示例**：
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-service
spec:
  replicas: 3
  template:
    spec:
      initContainers:
        - name: skywalking-agent
          image: apache/skywalking-java-agent:9.0.0-java17
          command:
            - sh
            - -c
            - cp -r /skywalking/agent /agent
          volumeMounts:
            - name: skywalking-agent
              mountPath: /agent
      containers:
        - name: order-service
          image: order-service:latest
          env:
            - name: JAVA_TOOL_OPTIONS
              value: "-javaagent:/agent/skywalking-agent.jar"
            - name: SW_AGENT_NAME
              value: "order-service"
            - name: SW_AGENT_COLLECTOR_BACKEND_SERVICES
              value: "skywalking-oap:11800"
          volumeMounts:
            - name: skywalking-agent
              mountPath: /agent
      volumes:
        - name: skywalking-agent
          emptyDir: {}
```

### Q6: 如何自定义插件？

**A**: 创建自定义插件：
```java
// 1. 创建插件定义类
public class MyPluginDefine extends ClassInstanceMethodsEnhancePluginDefine {
    
    @Override
    protected ClassMatch enhanceClass() {
        return NameMatch.byName("com.example.MyClass");
    }
    
    @Override
    public ConstructorInterceptPoint[] getConstructorsInterceptPoints() {
        return new ConstructorInterceptPoint[0];
    }
    
    @Override
    public InstanceMethodsInterceptPoint[] getInstanceMethodsInterceptPoints() {
        return new InstanceMethodsInterceptPoint[] {
            new InstanceMethodsInterceptPoint() {
                @Override
                public ElementMatcher<MethodDescription> getMethodsMatcher() {
                    return named("myMethod");
                }
                
                @Override
                public String getMethodsInterceptor() {
                    return "com.example.MyMethodInterceptor";
                }
                
                @Override
                public boolean isOverrideArgs() {
                    return false;
                }
            }
        };
    }
}

// 2. 创建拦截器
public class MyMethodInterceptor implements InstanceMethodsAroundInterceptor {
    
    @Override
    public void beforeMethod(EnhancedInstance objInst, Method method, 
                            Object[] allArguments, Class<?>[] argumentsTypes,
                            MethodInterceptResult result) {
        // 方法执行前的逻辑
        AbstractSpan span = ContextManager.createLocalSpan("MyMethod");
        span.tag("arg1", String.valueOf(allArguments[0]));
    }
    
    @Override
    public Object afterMethod(EnhancedInstance objInst, Method method,
                             Object[] allArguments, Class<?>[] argumentsTypes,
                             Object ret) {
        // 方法执行后的逻辑
        ContextManager.stopSpan();
        return ret;
    }
    
    @Override
    public void handleMethodException(EnhancedInstance objInst, Method method,
                                      Object[] allArguments, Class<?>[] argumentsTypes,
                                      Throwable t) {
        // 异常处理逻辑
        ContextManager.activeSpan().log(t);
    }
}
```

## 🔗 相关资源

### 官方资源
- **官方文档**: https://skywalking.apache.org/docs/
- **GitHub仓库**: https://github.com/apache/skywalking
- **官方博客**: https://skywalking.apache.org/blog/
- **插件列表**: https://skywalking.apache.org/docs/skywalking-java/latest/en/setup/service-agent/java-agent/supported-list/

### 学习资源
- **SkyWalking极简入门**: https://skywalking.apache.org/docs/skywalking-showcase/latest/readme/
- **SkyWalking实战**: 社区提供的实战案例
- **Apache SkyWalking社区**: https://skywalking.apache.org/community/

### 相关工具
- **SkyWalking CLI**: 命令行工具
- **SkyWalking Kubernetes**: K8s集成方案
- **SkyWalking Satellite**: 数据收集器

## 📝 学习检查清单

- [ ] 理解SkyWalking的架构和工作原理
- [ ] 掌握Agent的部署和配置
- [ ] 能够查看和分析追踪数据
- [ ] 理解服务拓扑图的含义
- [ ] 掌握性能指标的查看和分析
- [ ] 能够配置告警规则
- [ ] 了解探针的工作原理
- [ ] 掌握手动埋点的使用
- [ ] 理解采样策略
- [ ] 能够优化Agent性能
- [ ] 了解存储优化方法
- [ ] 掌握日志集成
- [ ] 理解Kubernetes部署方案

---

**学习建议**：
1. 先在本地搭建SkyWalking环境，熟悉UI界面
2. 部署一个简单的Spring Boot应用，配置Agent
3. 学习查看追踪数据和服务拓扑
4. 尝试手动埋点，添加自定义标签
5. 配置告警规则，测试告警功能
6. 研究性能优化和采样策略
7. 学习在生产环境的部署方案

**下一步学习**：
- 深入学习字节码增强技术
- 了解OpenTelemetry标准
- 学习自定义插件开发
- 研究大规模部署的最佳实践
- 学习与其他监控工具的集成
