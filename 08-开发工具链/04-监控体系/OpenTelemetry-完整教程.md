# OpenTelemetry 完整教程

> **作者**: erik.zhou  
> **创建时间**: 2025-01-31  
> **技术栈**: OpenTelemetry 1.35+, Spring Boot 3.x, Jaeger, Prometheus

## 📋 目录

- [1. OpenTelemetry简介](#1-opentelemetry简介)
- [2. 核心概念](#2-核心概念)
- [3. 快速开始](#3-快速开始)
- [4. Traces追踪](#4-traces追踪)
- [5. Metrics指标](#5-metrics指标)
- [6. Logs日志](#6-logs日志)
- [7. 实战案例](#7-实战案例)
- [8. 最佳实践](#8-最佳实践)

---

## 1. OpenTelemetry简介

### 1.1 什么是OpenTelemetry

OpenTelemetry (OTel) 是CNCF的可观测性框架，统一了Traces、Metrics、Logs三大支柱。

**核心特性**:
- 🔍 分布式追踪
- 📊 指标收集
- 📝 日志关联
- 🔌 厂商中立
- 🚀 自动化埋点

### 1.2 为什么选择OpenTelemetry

| 特性 | OpenTelemetry | 传统方案 |
|------|---------------|---------|
| 标准化 | ✅ CNCF标准 | ❌ 各自为政 |
| 厂商锁定 | ✅ 无锁定 | ❌ 强绑定 |
| 自动埋点 | ✅ 支持 | ⚠️ 手动 |
| 生态系统 | ✅ 丰富 | ⚠️ 有限 |
| 学习成本 | ✅ 统一API | ❌ 多套体系 |

### 1.3 三大支柱

```
┌─────────────────────────────────────────────────────┐
│              OpenTelemetry 可观测性                  │
├─────────────────┬─────────────────┬─────────────────┤
│    Traces       │    Metrics      │      Logs       │
│   (追踪链路)     │   (性能指标)     │    (日志记录)    │
│                 │                 │                 │
│  请求流转路径    │  CPU/内存/QPS   │   错误/警告     │
│  耗时分析       │  业务指标       │   调试信息      │
│  依赖关系       │  SLI/SLO       │   审计日志      │
└─────────────────┴─────────────────┴─────────────────┘
```

---

## 2. 核心概念

### 2.1 Traces (追踪)

#### Span (跨度)
```java
/**
 * Span是追踪的基本单元
 * 
 * @author erik.zhou
 */
public class SpanConcept {
    /*
     * Span包含:
     * - 操作名称
     * - 开始和结束时间
     * - 属性(Attributes)
     * - 事件(Events)
     * - 状态(Status)
     * - 父Span引用
     */
}
```

#### Trace (追踪链)
```
HTTP Request → Service A → Service B → Database
     │              │            │          │
   Span 1        Span 2       Span 3    Span 4
     └──────────────┴────────────┴──────────┘
                  Trace (追踪链)
```

### 2.2 Metrics (指标)

- **Counter**: 只增不减的计数器
- **Gauge**: 可增可减的仪表盘
- **Histogram**: 直方图，记录分布

### 2.3 Context Propagation (上下文传播)

```java
/**
 * 上下文在分布式系统中传播
 * 
 * @author erik.zhou
 */
public class ContextPropagation {
    /*
     * W3C Trace Context标准:
     * 
     * traceparent: 00-{trace-id}-{span-id}-{flags}
     * tracestate: vendor1=value1,vendor2=value2
     */
}
```

---

## 3. 快速开始

### 3.1 Maven依赖

```xml
<properties>
    <opentelemetry.version>1.35.0</opentelemetry.version>
</properties>

<dependencies>
    <!-- OpenTelemetry API -->
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-api</artifactId>
        <version>${opentelemetry.version}</version>
    </dependency>
    
    <!-- OpenTelemetry SDK -->
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-sdk</artifactId>
        <version>${opentelemetry.version}</version>
    </dependency>
    
    <!-- Spring Boot集成 -->
    <dependency>
        <groupId>io.opentelemetry.instrumentation</groupId>
        <artifactId>opentelemetry-spring-boot-starter</artifactId>
        <version>1.35.0-alpha</version>
    </dependency>

    
    <!-- Jaeger导出器 -->
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-exporter-jaeger</artifactId>
        <version>${opentelemetry.version}</version>
    </dependency>
    
    <!-- Prometheus导出器 -->
    <dependency>
        <groupId>io.opentelemetry</groupId>
        <artifactId>opentelemetry-exporter-prometheus</artifactId>
        <version>${opentelemetry.version}</version>
    </dependency>
    
    <!-- 自动埋点 -->
    <dependency>
        <groupId>io.opentelemetry.instrumentation</groupId>
        <artifactId>opentelemetry-instrumentation-annotations</artifactId>
        <version>1.35.0-alpha</version>
    </dependency>
</dependencies>
```

### 3.2 配置文件

```yaml
# application.yml
spring:
  application:
    name: otel-demo-service

otel:
  service:
    name: ${spring.application.name}
  traces:
    exporter: jaeger
  metrics:
    exporter: prometheus
  exporter:
    jaeger:
      endpoint: http://localhost:14250
    prometheus:
      port: 9464

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  metrics:
    export:
      prometheus:
        enabled: true
```

### 3.3 基础配置类

```java
package com.example.otel.config;

import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.common.Attributes;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.exporter.jaeger.JaegerGrpcSpanExporter;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.resources.Resource;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.BatchSpanProcessor;
import io.opentelemetry.semconv.resource.attributes.ResourceAttributes;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * OpenTelemetry配置
 * 
 * @author erik.zhou
 */
@Configuration
public class OpenTelemetryConfig {
    
    @Value("${otel.service.name}")
    private String serviceName;
    
    @Value("${otel.exporter.jaeger.endpoint}")
    private String jaegerEndpoint;
    
    @Bean
    public OpenTelemetry openTelemetry() {
        // 1. 创建Resource(服务标识)
        Resource resource = Resource.getDefault()
            .merge(Resource.create(
                Attributes.of(ResourceAttributes.SERVICE_NAME, serviceName)
            ));
        
        // 2. 创建Jaeger导出器
        JaegerGrpcSpanExporter jaegerExporter = JaegerGrpcSpanExporter.builder()
            .setEndpoint(jaegerEndpoint)
            .build();
        
        // 3. 创建TracerProvider
        SdkTracerProvider tracerProvider = SdkTracerProvider.builder()
            .addSpanProcessor(BatchSpanProcessor.builder(jaegerExporter).build())
            .setResource(resource)
            .build();
        
        // 4. 创建OpenTelemetry实例
        return OpenTelemetrySdk.builder()
            .setTracerProvider(tracerProvider)
            .buildAndRegisterGlobal();
    }
    
    @Bean
    public Tracer tracer(OpenTelemetry openTelemetry) {
        return openTelemetry.getTracer(serviceName);
    }
}
```

---

## 4. Traces追踪

### 4.1 手动埋点

```java
package com.example.otel.service;

import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.SpanKind;
import io.opentelemetry.api.trace.StatusCode;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Context;
import io.opentelemetry.context.Scope;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

/**
 * 手动埋点示例
 * 
 * @author erik.zhou
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class OrderService {
    
    private final Tracer tracer;
    private final PaymentService paymentService;
    
    /**
     * 创建订单 - 手动创建Span
     */
    public String createOrder(String userId, String productId) {
        // 创建Span
        Span span = tracer.spanBuilder("createOrder")
            .setSpanKind(SpanKind.SERVER)
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            // 添加属性
            span.setAttribute("user.id", userId);
            span.setAttribute("product.id", productId);
            
            // 业务逻辑
            log.info("创建订单: userId={}, productId={}", userId, productId);
            
            // 添加事件
            span.addEvent("订单验证完成");
            
            // 调用支付服务
            String paymentResult = paymentService.processPayment(userId, 100.0);
            
            span.addEvent("支付处理完成");
            
            // 设置成功状态
            span.setStatus(StatusCode.OK);
            
            return "ORDER-" + System.currentTimeMillis();
            
        } catch (Exception e) {
            // 记录异常
            span.recordException(e);
            span.setStatus(StatusCode.ERROR, "订单创建失败");
            throw e;
        } finally {
            // 结束Span
            span.end();
        }
    }
}
```

### 4.2 自动埋点

```java
package com.example.otel.service;

import io.opentelemetry.instrumentation.annotations.WithSpan;
import io.opentelemetry.instrumentation.annotations.SpanAttribute;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

/**
 * 自动埋点示例 - 使用注解
 * 
 * @author erik.zhou
 */
@Slf4j
@Service
public class PaymentService {
    
    /**
     * 使用@WithSpan自动创建Span
     */
    @WithSpan
    public String processPayment(
        @SpanAttribute("user.id") String userId,
        @SpanAttribute("amount") Double amount
    ) {
        log.info("处理支付: userId={}, amount={}", userId, amount);
        
        // 模拟支付处理
        try {
            Thread.sleep(100);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        
        return "PAYMENT-SUCCESS";
    }
    
    /**
     * 嵌套Span
     */
    @WithSpan
    public void validatePayment(@SpanAttribute("payment.id") String paymentId) {
        log.info("验证支付: paymentId={}", paymentId);
        
        // 调用其他方法会创建子Span
        checkBalance(paymentId);
        checkRisk(paymentId);
    }
    
    @WithSpan
    private void checkBalance(String paymentId) {
        log.info("检查余额: paymentId={}", paymentId);
    }
    
    @WithSpan
    private void checkRisk(String paymentId) {
        log.info("风险检查: paymentId={}", paymentId);
    }
}
```

### 4.3 HTTP请求追踪

```java
package com.example.otel.controller;

import com.example.otel.service.OrderService;
import io.opentelemetry.api.trace.Span;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.bind.annotation.*;

/**
 * HTTP请求自动追踪
 * 
 * @author erik.zhou
 */
@Slf4j
@RestController
@RequestMapping("/api/orders")
@RequiredArgsConstructor
public class OrderController {
    
    private final OrderService orderService;
    
    @PostMapping
    public OrderResponse createOrder(@RequestBody OrderRequest request) {
        // 获取当前Span
        Span currentSpan = Span.current();
        currentSpan.setAttribute("http.method", "POST");
        currentSpan.setAttribute("http.route", "/api/orders");
        
        log.info("收到创建订单请求: {}", request);
        
        String orderId = orderService.createOrder(
            request.userId(),
            request.productId()
        );
        
        return new OrderResponse(orderId, "SUCCESS");
    }
    
    record OrderRequest(String userId, String productId) {}
    record OrderResponse(String orderId, String status) {}
}
```

### 4.4 数据库查询追踪

```java
package com.example.otel.repository;

import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Scope;
import lombok.RequiredArgsConstructor;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

/**
 * 数据库操作追踪
 * 
 * @author erik.zhou
 */
@Repository
@RequiredArgsConstructor
public class OrderRepository {
    
    private final JdbcTemplate jdbcTemplate;
    private final Tracer tracer;
    
    public void saveOrder(String orderId, String userId) {
        Span span = tracer.spanBuilder("db.insert.orders")
            .setAttribute("db.system", "mysql")
            .setAttribute("db.operation", "INSERT")
            .setAttribute("db.table", "orders")
            .startSpan();
        
        try (Scope scope = span.makeCurrent()) {
            String sql = "INSERT INTO orders (order_id, user_id) VALUES (?, ?)";
            jdbcTemplate.update(sql, orderId, userId);
            
            span.setAttribute("db.rows_affected", 1);
        } finally {
            span.end();
        }
    }
}
```

---

## 5. Metrics指标

### 5.1 Counter (计数器)

```java
package com.example.otel.metrics;

import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.metrics.LongCounter;
import io.opentelemetry.api.metrics.Meter;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

/**
 * Counter指标示例
 * 
 * @author erik.zhou
 */
@Slf4j
@Component
public class OrderMetrics {
    
    private final LongCounter orderCreatedCounter;
    private final LongCounter orderFailedCounter;
    
    public OrderMetrics(OpenTelemetry openTelemetry) {
        Meter meter = openTelemetry.getMeter("order-service");
        
        // 创建订单成功计数器
        this.orderCreatedCounter = meter
            .counterBuilder("orders.created")
            .setDescription("订单创建成功数量")
            .setUnit("1")
            .build();
        
        // 创建订单失败计数器
        this.orderFailedCounter = meter
            .counterBuilder("orders.failed")
            .setDescription("订单创建失败数量")
            .setUnit("1")
            .build();
    }
    
    public void recordOrderCreated(String userId, String productId) {
        orderCreatedCounter.add(1,
            io.opentelemetry.api.common.Attributes.builder()
                .put("user.id", userId)
                .put("product.id", productId)
                .build()
        );
    }
    
    public void recordOrderFailed(String reason) {
        orderFailedCounter.add(1,
            io.opentelemetry.api.common.Attributes.builder()
                .put("failure.reason", reason)
                .build()
        );
    }
}
```

### 5.2 Gauge (仪表盘)

```java
package com.example.otel.metrics;

import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.metrics.Meter;
import org.springframework.stereotype.Component;

import java.util.concurrent.atomic.AtomicLong;

/**
 * Gauge指标示例
 * 
 * @author erik.zhou
 */
@Component
public class SystemMetrics {
    
    private final AtomicLong activeConnections = new AtomicLong(0);
    
    public SystemMetrics(OpenTelemetry openTelemetry) {
        Meter meter = openTelemetry.getMeter("system-metrics");
        
        // 创建活跃连接数Gauge
        meter.gaugeBuilder("connections.active")
            .setDescription("当前活跃连接数")
            .setUnit("1")
            .buildWithCallback(measurement -> 
                measurement.record(activeConnections.get())
            );
        
        // JVM内存使用Gauge
        meter.gaugeBuilder("jvm.memory.used")
            .setDescription("JVM内存使用量")
            .setUnit("bytes")
            .buildWithCallback(measurement -> {
                Runtime runtime = Runtime.getRuntime();
                long usedMemory = runtime.totalMemory() - runtime.freeMemory();
                measurement.record(usedMemory);
            });
    }
    
    public void incrementConnections() {
        activeConnections.incrementAndGet();
    }
    
    public void decrementConnections() {
        activeConnections.decrementAndGet();
    }
}
```

### 5.3 Histogram (直方图)

```java
package com.example.otel.metrics;

import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.metrics.DoubleHistogram;
import io.opentelemetry.api.metrics.Meter;
import org.springframework.stereotype.Component;

/**
 * Histogram指标示例
 * 
 * @author erik.zhou
 */
@Component
public class PerformanceMetrics {
    
    private final DoubleHistogram requestDuration;
    private final DoubleHistogram paymentAmount;
    
    public PerformanceMetrics(OpenTelemetry openTelemetry) {
        Meter meter = openTelemetry.getMeter("performance-metrics");
        
        // 请求耗时直方图
        this.requestDuration = meter
            .histogramBuilder("http.request.duration")
            .setDescription("HTTP请求耗时")
            .setUnit("ms")
            .build();
        
        // 支付金额直方图
        this.paymentAmount = meter
            .histogramBuilder("payment.amount")
            .setDescription("支付金额分布")
            .setUnit("CNY")
            .build();
    }
    
    public void recordRequestDuration(double durationMs, String endpoint) {
        requestDuration.record(durationMs,
            io.opentelemetry.api.common.Attributes.builder()
                .put("http.endpoint", endpoint)
                .build()
        );
    }
    
    public void recordPaymentAmount(double amount, String paymentMethod) {
        paymentAmount.record(amount,
            io.opentelemetry.api.common.Attributes.builder()
                .put("payment.method", paymentMethod)
                .build()
        );
    }
}
```

---

## 6. Logs日志

### 6.1 日志关联

```java
package com.example.otel.logging;

import io.opentelemetry.api.trace.Span;
import lombok.extern.slf4j.Slf4j;
import org.slf4j.MDC;
import org.springframework.stereotype.Component;

/**
 * 日志与Trace关联
 * 
 * @author erik.zhou
 */
@Slf4j
@Component
public class TraceLoggingService {
    
    public void logWithTrace(String message) {
        // 获取当前Span
        Span currentSpan = Span.current();
        
        // 将TraceId和SpanId添加到MDC
        MDC.put("trace_id", currentSpan.getSpanContext().getTraceId());
        MDC.put("span_id", currentSpan.getSpanContext().getSpanId());
        
        try {
            log.info(message);
        } finally {
            MDC.clear();
        }
    }
}
```

### 6.2 Logback配置

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>
                %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - 
                [TraceId: %X{trace_id}] [SpanId: %X{span_id}] - %msg%n
            </pattern>
        </encoder>
    </appender>
    
    <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="net.logstash.logback.encoder.LogstashEncoder">
            <includeMdcKeyName>trace_id</includeMdcKeyName>
            <includeMdcKeyName>span_id</includeMdcKeyName>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE" />
    </root>
</configuration>
```

---

## 7. 实战案例

### 7.1 微服务链路追踪

```java
package com.example.otel.microservice;

import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Context;
import io.opentelemetry.context.propagation.TextMapGetter;
import io.opentelemetry.context.propagation.TextMapPropagator;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpHeaders;
import org.springframework.stereotype.Service;
import org.springframework.web.client.RestTemplate;

/**
 * 微服务间追踪传播
 * 
 * @author erik.zhou
 */
@Service
@RequiredArgsConstructor
public class MicroserviceClient {
    
    private final RestTemplate restTemplate;
    private final Tracer tracer;
    private final TextMapPropagator propagator;
    
    public String callDownstreamService(String endpoint, String data) {
        Span span = tracer.spanBuilder("call.downstream")
            .setAttribute("http.url", endpoint)
            .startSpan();
        
        try (var scope = span.makeCurrent()) {
            // 创建HTTP头
            HttpHeaders headers = new HttpHeaders();
            
            // 注入Trace上下文到HTTP头
            propagator.inject(Context.current(), headers, 
                (carrier, key, value) -> carrier.add(key, value)
            );
            
            // 发送HTTP请求
            HttpEntity<String> request = new HttpEntity<>(data, headers);
            return restTemplate.postForObject(endpoint, request, String.class);
            
        } finally {
            span.end();
        }
    }
}
```

### 7.2 异步任务追踪

```java
package com.example.otel.async;

import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Context;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Service;

import java.util.concurrent.CompletableFuture;

/**
 * 异步任务追踪
 * 
 * @author erik.zhou
 */
@Slf4j
@Service
@RequiredArgsConstructor
public class AsyncTaskService {
    
    private final Tracer tracer;
    
    @Async
    public CompletableFuture<String> processAsync(String taskId) {
        // 捕获当前上下文
        Context parentContext = Context.current();
        
        return CompletableFuture.supplyAsync(() -> {
            // 在新线程中恢复上下文
            try (var scope = parentContext.makeCurrent()) {
                Span span = tracer.spanBuilder("async.task")
                    .setAttribute("task.id", taskId)
                    .startSpan();
                
                try (var spanScope = span.makeCurrent()) {
                    log.info("处理异步任务: {}", taskId);
                    Thread.sleep(1000);
                    return "TASK-COMPLETED";
                } catch (Exception e) {
                    span.recordException(e);
                    throw new RuntimeException(e);
                } finally {
                    span.end();
                }
            }
        });
    }
}
```

---

## 8. 最佳实践

### 8.1 Span命名规范

```java
/**
 * Span命名最佳实践
 * 
 * @author erik.zhou
 */
public class SpanNamingBestPractices {
    
    // ✅ 好的命名
    String goodName1 = "GET /api/users/{id}";
    String goodName2 = "db.query.users";
    String goodName3 = "kafka.produce.order-events";
    
    // ❌ 不好的命名
    String badName1 = "process";  // 太模糊
    String badName2 = "GET /api/users/123";  // 包含具体值
    String badName3 = "doSomething";  // 不清晰
}
```

### 8.2 属性添加规范

```java
package com.example.otel.best;

import io.opentelemetry.api.common.AttributeKey;
import io.opentelemetry.api.common.Attributes;
import io.opentelemetry.api.trace.Span;

/**
 * 属性添加最佳实践
 * 
 * @author erik.zhou
 */
public class AttributeBestPractices {
    
    // 定义常量
    private static final AttributeKey<String> USER_ID = 
        AttributeKey.stringKey("user.id");
    private static final AttributeKey<Long> ORDER_AMOUNT = 
        AttributeKey.longKey("order.amount");
    
    public void addAttributes(Span span, String userId, Long amount) {
        // ✅ 使用类型安全的AttributeKey
        span.setAttribute(USER_ID, userId);
        span.setAttribute(ORDER_AMOUNT, amount);
        
        // ✅ 批量添加属性
        Attributes attrs = Attributes.builder()
            .put("http.method", "POST")
            .put("http.status_code", 200L)
            .build();
        span.setAllAttributes(attrs);
    }
}
```

### 8.3 采样策略

```java
package com.example.otel.sampling;

import io.opentelemetry.sdk.trace.samplers.Sampler;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * 采样策略配置
 * 
 * @author erik.zhou
 */
@Configuration
public class SamplingConfig {
    
    @Bean
    public Sampler sampler() {
        // 1. 始终采样 (开发环境)
        // return Sampler.alwaysOn();
        
        // 2. 从不采样
        // return Sampler.alwaysOff();
        
        // 3. 概率采样 (10%采样率)
        // return Sampler.traceIdRatioBased(0.1);
        
        // 4. 父级采样 (跟随父Span的采样决策)
        return Sampler.parentBased(Sampler.traceIdRatioBased(0.1));
    }
}
```

---

## 9. 总结

OpenTelemetry是现代可观测性的标准:

### 核心优势
- ✅ 统一的可观测性标准
- ✅ 厂商中立，避免锁定
- ✅ 自动埋点，降低成本
- ✅ 丰富的生态系统
- ✅ 生产级性能

### 实施建议
1. 从Traces开始，逐步添加Metrics和Logs
2. 优先使用自动埋点，必要时手动埋点
3. 合理设置采样率，平衡成本和可见性
4. 统一命名规范和属性标准
5. 关联日志和追踪，提升问题定位效率

---

**作者**: erik.zhou  
**最后更新**: 2025-01-31  
**版本**: 1.0.0
