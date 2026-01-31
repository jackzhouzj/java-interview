# Spring Cloud Gateway 完整教程

## 📋 目录
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: 4.1.x (基于Spring Boot 3.2.x)
- **官方文档**: https://spring.io/projects/spring-cloud-gateway
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Spring Boot 3.x
  - Spring WebFlux (响应式编程基础)
  - 微服务架构基础
  - Nacos/Eureka服务注册与发现

## 🎯 学习目标
- [ ] 理解Spring Cloud Gateway的工作原理和核心概念
- [ ] 掌握路由配置的多种方式（YAML配置和Java代码配置）
- [ ] 熟练使用内置的断言工厂和过滤器工厂
- [ ] 能够实现自定义过滤器和全局过滤器
- [ ] 掌握限流、熔断、重试等高级特性
- [ ] 实现动态路由和路由刷新机制

## 📖 基础概念

### 1.1 什么是Spring Cloud Gateway

Spring Cloud Gateway是Spring Cloud生态系统中的API网关组件，基于Spring Framework 6、Spring Boot 3和Project Reactor构建。它提供了一种简单而有效的方式来路由API请求，并为这些请求提供横切关注点，如安全性、监控/指标和弹性。

**核心特点**：
- 基于Spring Framework和Spring Boot构建
- 支持Spring WebFlux和Spring Web MVC两种模式
- 能够基于任何请求属性进行路由匹配
- 断言和过滤器针对特定路由
- 集成Spring Cloud Circuit Breaker（熔断器）
- 集成Spring Cloud DiscoveryClient（服务发现）
- 易于编写自定义断言和过滤器
- 支持请求限流
- 支持路径重写

### 1.2 核心概念

- **Route（路由）**: 网关的基本构建块，由ID、目标URI、断言集合和过滤器集合组成
- **Predicate（断言）**: 匹配HTTP请求的条件，如请求路径、方法、头部等
- **Filter（过滤器）**: 在发送下游请求之前或之后修改请求和响应
- **Gateway Handler Mapping**: 决定请求应该路由到哪里
- **Gateway Web Handler**: 处理请求并应用过滤器链


### 1.3 工作原理

```
客户端请求 → Gateway Handler Mapping → Gateway Web Handler → 过滤器链 → 代理服务
                    ↓                           ↓
              匹配路由规则              应用Pre/Post过滤器
```

**请求处理流程**：
1. 客户端向Spring Cloud Gateway发送请求
2. Gateway Handler Mapping确定请求与路由匹配
3. 请求被发送到Gateway Web Handler
4. Handler通过特定于该路由的过滤器链运行请求
5. 过滤器可以在发送代理请求之前和之后运行逻辑（Pre和Post过滤器）
6. 执行代理请求
7. 返回响应给客户端

### 1.4 应用场景

- **统一入口**: 为微服务集群提供统一的API入口
- **路由转发**: 根据请求路径、参数等条件将请求路由到不同的后端服务
- **负载均衡**: 与服务发现组件集成，实现客户端负载均衡
- **安全认证**: 统一处理认证和授权逻辑
- **限流熔断**: 保护后端服务，防止过载
- **日志监控**: 统一记录请求日志，便于监控和分析
- **协议转换**: 支持HTTP、WebSocket等多种协议

## 🔥 核心特性 (重点)

### 2.1 路由配置 🔥

Spring Cloud Gateway支持两种路由配置方式：YAML配置和Java代码配置。

#### 2.1.1 YAML配置方式

```yaml
spring:
  cloud:
    gateway:
      routes:
        # 基础路由配置
        - id: user_service_route
          uri: lb://user-service
          predicates:
            - Path=/api/user/**
          filters:
            - StripPrefix=1
        
        # 多条件路由
        - id: order_service_route
          uri: lb://order-service
          predicates:
            - Path=/api/order/**
            - Method=GET,POST
            - Header=X-Request-Id, \d+
          filters:
            - AddRequestHeader=X-Gateway, Spring-Cloud-Gateway
            - AddResponseHeader=X-Response-Time, ${responseTime}
        
        # 带权重的路由（灰度发布）
        - id: product_service_v1
          uri: lb://product-service-v1
          predicates:
            - Path=/api/product/**
            - Weight=group1, 8
        
        - id: product_service_v2
          uri: lb://product-service-v2
          predicates:
            - Path=/api/product/**
            - Weight=group1, 2
```


#### 2.1.2 Java代码配置方式

```java
package com.example.gateway.config;

import org.springframework.cloud.gateway.route.RouteLocator;
import org.springframework.cloud.gateway.route.builder.RouteLocatorBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Gateway路由配置
 * 
 * @author erik.zhou
 */
@Configuration
public class GatewayRouteConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            // 基础路径路由
            .route("path_route", r -> r.path("/get")
                .uri("https://httpbin.org"))
            
            // 主机路由
            .route("host_route", r -> r.host("*.myhost.org")
                .uri("https://httpbin.org"))
            
            // 路径重写路由
            .route("rewrite_route", r -> r.host("*.rewrite.org")
                .filters(f -> f.rewritePath("/foo/(?<segment>.*)", "/${segment}"))
                .uri("https://httpbin.org"))
            
            // 熔断器路由
            .route("circuit_breaker_route", r -> r.host("*.circuitbreaker.org")
                .filters(f -> f.circuitBreaker(c -> c.setName("slowcmd")))
                .uri("https://httpbin.org"))
            
            // 熔断器带降级路由
            .route("circuit_breaker_fallback_route", r -> r.host("*.circuitbreakerfallback.org")
                .filters(f -> f.circuitBreaker(c -> c
                    .setName("slowcmd")
                    .setFallbackUri("forward:/circuitbreakerfallback")))
                .uri("https://httpbin.org"))
            
            // 限流路由
            .route("limit_route", r -> r
                .host("*.limited.org")
                .and()
                .path("/anything/**")
                .filters(f -> f.requestRateLimiter(c -> c
                    .setRateLimiter(redisRateLimiter())))
                .uri("https://httpbin.org"))
            .build();
    }
    
    // Redis限流器配置
    private org.springframework.cloud.gateway.filter.ratelimit.RateLimiter redisRateLimiter() {
        // 实现Redis限流器
        throw new UnsupportedOperationException("需要配置RedisRateLimiter");
    }
}
```


### 2.2 断言工厂（Predicate Factories）🔥

Spring Cloud Gateway内置了多种断言工厂，用于匹配HTTP请求的各种属性。

#### 2.2.1 常用断言工厂

| 断言工厂 | 说明 | 示例 |
|---------|------|------|
| Path | 匹配请求路径 | `Path=/api/user/**` |
| Method | 匹配HTTP方法 | `Method=GET,POST` |
| Header | 匹配请求头 | `Header=X-Request-Id, \d+` |
| Query | 匹配查询参数 | `Query=name, \w+` |
| Cookie | 匹配Cookie | `Cookie=session, \w+` |
| Host | 匹配主机名 | `Host=**.example.org` |
| RemoteAddr | 匹配远程地址 | `RemoteAddr=192.168.1.1/24` |
| After | 在指定时间之后 | `After=2024-01-01T00:00:00+08:00[Asia/Shanghai]` |
| Before | 在指定时间之前 | `Before=2024-12-31T23:59:59+08:00[Asia/Shanghai]` |
| Between | 在指定时间范围内 | `Between=2024-01-01T00:00:00+08:00[Asia/Shanghai], 2024-12-31T23:59:59+08:00[Asia/Shanghai]` |
| Weight | 权重路由 | `Weight=group1, 8` |

#### 2.2.2 断言组合使用

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: complex_route
          uri: lb://backend-service
          predicates:
            - Path=/api/**
            - Method=GET,POST
            - Header=X-Request-Id, \d+
            - Query=version, v1
            - After=2024-01-01T00:00:00+08:00[Asia/Shanghai]
          filters:
            - StripPrefix=1
```

### 2.3 过滤器工厂（Filter Factories）🔥

过滤器用于在请求被路由前后修改请求和响应。

#### 2.3.1 请求修改过滤器

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: request_filter_route
          uri: lb://backend-service
          predicates:
            - Path=/api/**
          filters:
            # 添加请求头
            - AddRequestHeader=X-Request-Source, Gateway
            # 添加请求参数
            - AddRequestParameter=source, gateway
            # 移除请求头
            - RemoveRequestHeader=X-Sensitive-Header
            # 移除请求参数
            - RemoveRequestParameter=debug
            # 设置请求头
            - SetRequestHeader=X-Request-Id, ${requestId}
            # 路径前缀
            - PrefixPath=/v1
            # 去除路径前缀
            - StripPrefix=1
            # 路径重写
            - RewritePath=/api/(?<segment>.*), /${segment}
            # 设置路径
            - SetPath=/new-path
```


#### 2.3.2 响应修改过滤器

```yaml
filters:
  # 添加响应头
  - AddResponseHeader=X-Response-Source, Gateway
  # 移除响应头
  - RemoveResponseHeader=X-Sensitive-Info
  # 设置响应头
  - SetResponseHeader=X-Response-Time, ${responseTime}
  # 去重响应头
  - DedupeResponseHeader=Access-Control-Allow-Credentials Access-Control-Allow-Origin
  # 重写响应头
  - RewriteResponseHeader=X-Response-Foo, password=[^&]+, password=***
  # 设置响应状态码
  - SetStatus=200
```

#### 2.3.3 高级过滤器

```yaml
filters:
  # 熔断器
  - name: CircuitBreaker
    args:
      name: myCircuitBreaker
      fallbackUri: forward:/fallback
      statusCodes:
        - 500
        - 503
  
  # 限流
  - name: RequestRateLimiter
    args:
      redis-rate-limiter.replenishRate: 10
      redis-rate-limiter.burstCapacity: 20
      redis-rate-limiter.requestedTokens: 1
  
  # 重试
  - name: Retry
    args:
      retries: 3
      statuses: BAD_GATEWAY,GATEWAY_TIMEOUT
      methods: GET,POST
      backoff:
        firstBackoff: 10ms
        maxBackoff: 50ms
        factor: 2
        basedOnPreviousValue: false
  
  # 请求大小限制
  - name: RequestSize
    args:
      maxSize: 5MB
  
  # 修改请求体
  - name: ModifyRequestBody
    args:
      inClass: String
      outClass: String
  
  # 修改响应体
  - name: ModifyResponseBody
    args:
      inClass: String
      outClass: String
```

### 2.4 自定义过滤器（⚠️ 难点）

#### 2.4.1 自定义GatewayFilter

```java
package com.example.gateway.filter;

import org.springframework.cloud.gateway.filter.GatewayFilter;
import org.springframework.cloud.gateway.filter.factory.AbstractGatewayFilterFactory;
import org.springframework.stereotype.Component;
import reactor.core.publisher.Mono;

/**
 * 自定义网关过滤器 - 添加请求日志
 * 
 * @author erik.zhou
 */
@Component
public class CustomGatewayFilterFactory extends AbstractGatewayFilterFactory<CustomGatewayFilterFactory.Config> {
    
    public CustomGatewayFilterFactory() {
        super(Config.class);
    }
    
    @Override
    public GatewayFilter apply(Config config) {
        return (exchange, chain) -> {
            // Pre过滤器逻辑
            System.out.println("Custom Pre Filter: " + config.getMessage());
            
            return chain.filter(exchange).then(Mono.fromRunnable(() -> {
                // Post过滤器逻辑
                System.out.println("Custom Post Filter: Response Status = " 
                    + exchange.getResponse().getStatusCode());
            }));
        };
    }
    
    public static class Config {
        private String message;
        
        public String getMessage() {
            return message;
        }
        
        public void setMessage(String message) {
            this.message = message;
        }
    }
}
```


**使用自定义过滤器**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: custom_filter_route
          uri: lb://backend-service
          predicates:
            - Path=/api/**
          filters:
            - name: Custom
              args:
                message: "This is a custom filter"
```

#### 2.4.2 自定义GlobalFilter

```java
package com.example.gateway.filter;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.UUID;

/**
 * 全局过滤器 - 添加请求追踪ID
 * 
 * @author erik.zhou
 */
@Component
public class TraceIdGlobalFilter implements GlobalFilter, Ordered {
    
    private static final String TRACE_ID_HEADER = "X-Trace-Id";
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 获取或生成TraceId
        ServerHttpRequest request = exchange.getRequest();
        String traceId = request.getHeaders().getFirst(TRACE_ID_HEADER);
        
        if (traceId == null || traceId.isEmpty()) {
            traceId = UUID.randomUUID().toString();
        }
        
        // 添加TraceId到请求头
        ServerHttpRequest mutatedRequest = request.mutate()
            .header(TRACE_ID_HEADER, traceId)
            .build();
        
        ServerWebExchange mutatedExchange = exchange.mutate()
            .request(mutatedRequest)
            .build();
        
        // 添加TraceId到响应头
        mutatedExchange.getResponse().getHeaders().add(TRACE_ID_HEADER, traceId);
        
        return chain.filter(mutatedExchange);
    }
    
    @Override
    public int getOrder() {
        // 数字越小，优先级越高
        return -100;
    }
}
```

#### 2.4.3 自定义断言工厂

```java
package com.example.gateway.predicate;

import org.springframework.cloud.gateway.handler.predicate.AbstractRoutePredicateFactory;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;

import java.util.Arrays;
import java.util.List;
import java.util.function.Predicate;

/**
 * 自定义断言工厂 - 根据请求时间段匹配
 * 
 * @author erik.zhou
 */
@Component
public class TimeRangeRoutePredicateFactory 
    extends AbstractRoutePredicateFactory<TimeRangeRoutePredicateFactory.Config> {
    
    public TimeRangeRoutePredicateFactory() {
        super(Config.class);
    }
    
    @Override
    public List<String> shortcutFieldOrder() {
        return Arrays.asList("startHour", "endHour");
    }
    
    @Override
    public Predicate<ServerWebExchange> apply(Config config) {
        return exchange -> {
            int currentHour = java.time.LocalTime.now().getHour();
            return currentHour >= config.getStartHour() && currentHour <= config.getEndHour();
        };
    }
    
    public static class Config {
        private int startHour;
        private int endHour;
        
        public int getStartHour() {
            return startHour;
        }
        
        public void setStartHour(int startHour) {
            this.startHour = startHour;
        }
        
        public int getEndHour() {
            return endHour;
        }
        
        public void setEndHour(int endHour) {
            this.endHour = endHour;
        }
    }
}
```


**使用自定义断言**：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: time_range_route
          uri: lb://backend-service
          predicates:
            - Path=/api/**
            - TimeRange=9,18  # 9点到18点之间可访问
```

### 2.5 动态路由（⚠️ 难点）

动态路由允许在运行时添加、修改或删除路由配置，无需重启应用。

#### 2.5.1 启用动态路由端点

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    gateway:
      enabled: true
```

#### 2.5.2 动态路由管理服务

```java
package com.example.gateway.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.gateway.event.RefreshRoutesEvent;
import org.springframework.cloud.gateway.route.RouteDefinition;
import org.springframework.cloud.gateway.route.RouteDefinitionWriter;
import org.springframework.context.ApplicationEventPublisher;
import org.springframework.stereotype.Service;
import reactor.core.publisher.Mono;

/**
 * 动态路由管理服务
 * 
 * @author erik.zhou
 */
@Service
public class DynamicRouteService {
    
    @Autowired
    private RouteDefinitionWriter routeDefinitionWriter;
    
    @Autowired
    private ApplicationEventPublisher publisher;
    
    /**
     * 添加路由
     */
    public String addRoute(RouteDefinition definition) {
        routeDefinitionWriter.save(Mono.just(definition)).subscribe();
        // 发布路由刷新事件
        publisher.publishEvent(new RefreshRoutesEvent(this));
        return "success";
    }
    
    /**
     * 更新路由
     */
    public String updateRoute(RouteDefinition definition) {
        try {
            // 先删除后添加
            deleteRoute(definition.getId());
            addRoute(definition);
            return "success";
        } catch (Exception e) {
            return "update fail: " + e.getMessage();
        }
    }
    
    /**
     * 删除路由
     */
    public String deleteRoute(String id) {
        routeDefinitionWriter.delete(Mono.just(id)).subscribe();
        publisher.publishEvent(new RefreshRoutesEvent(this));
        return "success";
    }
}
```

#### 2.5.3 动态路由REST API

```java
package com.example.gateway.controller;

import com.example.gateway.service.DynamicRouteService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cloud.gateway.route.RouteDefinition;
import org.springframework.web.bind.annotation.*;

/**
 * 动态路由管理接口
 * 
 * @author erik.zhou
 */
@RestController
@RequestMapping("/gateway/routes")
public class DynamicRouteController {
    
    @Autowired
    private DynamicRouteService dynamicRouteService;
    
    /**
     * 添加路由
     */
    @PostMapping
    public String addRoute(@RequestBody RouteDefinition definition) {
        return dynamicRouteService.addRoute(definition);
    }
    
    /**
     * 更新路由
     */
    @PutMapping("/{id}")
    public String updateRoute(@PathVariable String id, 
                             @RequestBody RouteDefinition definition) {
        definition.setId(id);
        return dynamicRouteService.updateRoute(definition);
    }
    
    /**
     * 删除路由
     */
    @DeleteMapping("/{id}")
    public String deleteRoute(@PathVariable String id) {
        return dynamicRouteService.deleteRoute(id);
    }
}
```


#### 2.5.4 使用Actuator管理路由

```bash
# 获取所有路由
curl http://localhost:8080/actuator/gateway/routes

# 获取特定路由
curl http://localhost:8080/actuator/gateway/routes/user_service_route

# 刷新路由缓存
curl -X POST http://localhost:8080/actuator/gateway/refresh

# 创建新路由
curl -X POST http://localhost:8080/actuator/gateway/routes/new-route \
  -H "Content-Type: application/json" \
  -d '{
    "id": "new-route",
    "uri": "http://example.org",
    "predicates": [
      {
        "name": "Path",
        "args": {"_genkey_0": "/new-path"}
      }
    ],
    "filters": [
      {
        "name": "AddRequestHeader",
        "args": {
          "_genkey_0": "X-Custom-Header",
          "_genkey_1": "CustomValue"
        }
      }
    ],
    "order": 0
  }'

# 删除路由
curl -X DELETE http://localhost:8080/actuator/gateway/routes/new-route
```

### 2.6 限流

#### 2.6.1 Redis限流配置

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
</dependency>
```

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: 
  cloud:
    gateway:
      routes:
        - id: rate_limit_route
          uri: lb://backend-service
          predicates:
            - Path=/api/**
          filters:
            - name: RequestRateLimiter
              args:
                # 令牌桶每秒填充速率
                redis-rate-limiter.replenishRate: 10
                # 令牌桶容量
                redis-rate-limiter.burstCapacity: 20
                # 每个请求消耗的令牌数
                redis-rate-limiter.requestedTokens: 1
                # 限流键解析器
                key-resolver: "#{@userKeyResolver}"
```

#### 2.6.2 自定义限流键解析器

```java
package com.example.gateway.config;

import org.springframework.cloud.gateway.filter.ratelimit.KeyResolver;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import reactor.core.publisher.Mono;

/**
 * 限流键解析器配置
 * 
 * @author erik.zhou
 */
@Configuration
public class RateLimiterConfig {
    
    /**
     * 基于用户ID限流
     */
    @Bean
    public KeyResolver userKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest().getQueryParams().getFirst("userId")
        );
    }
    
    /**
     * 基于IP地址限流
     */
    @Bean
    public KeyResolver ipKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest().getRemoteAddress().getAddress().getHostAddress()
        );
    }
    
    /**
     * 基于请求路径限流
     */
    @Bean
    public KeyResolver pathKeyResolver() {
        return exchange -> Mono.just(
            exchange.getRequest().getPath().value()
        );
    }
}
```


### 2.7 熔断器集成

#### 2.7.1 添加依赖

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
</dependency>
```

#### 2.7.2 配置熔断器

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: circuit_breaker_route
          uri: lb://backend-service
          predicates:
            - Path=/api/**
          filters:
            - name: CircuitBreaker
              args:
                name: backendCircuitBreaker
                fallbackUri: forward:/fallback
                statusCodes:
                  - 500
                  - 503
                  - INTERNAL_SERVER_ERROR

# Resilience4j配置
resilience4j:
  circuitbreaker:
    configs:
      default:
        # 滑动窗口大小
        slidingWindowSize: 10
        # 失败率阈值
        failureRateThreshold: 50
        # 慢调用时间阈值
        slowCallDurationThreshold: 2s
        # 慢调用率阈值
        slowCallRateThreshold: 50
        # 等待时间（开启到半开启）
        waitDurationInOpenState: 10s
        # 半开启状态允许的调用数
        permittedNumberOfCallsInHalfOpenState: 5
    instances:
      backendCircuitBreaker:
        baseConfig: default
```

#### 2.7.3 降级处理

```java
package com.example.gateway.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.HashMap;
import java.util.Map;

/**
 * 熔断降级处理
 * 
 * @author erik.zhou
 */
@RestController
public class FallbackController {
    
    @GetMapping("/fallback")
    public Map<String, Object> fallback() {
        Map<String, Object> result = new HashMap<>();
        result.put("code", 503);
        result.put("message", "服务暂时不可用，请稍后重试");
        result.put("data", null);
        return result;
    }
}
```

## 💻 实战应用

### 3.1 环境搭建

#### 3.1.1 创建Spring Boot项目

```xml
<!-- pom.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.0</version>
        <relativePath/>
    </parent>
    
    <groupId>com.example</groupId>
    <artifactId>gateway-service</artifactId>
    <version>1.0.0</version>
    
    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2023.0.0</spring-cloud.version>
    </properties>
    
    <dependencies>
        <!-- Spring Cloud Gateway -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        
        <!-- Nacos服务发现 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        
        <!-- 负载均衡 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-loadbalancer</artifactId>
        </dependency>
        
        <!-- Redis限流 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis-reactive</artifactId>
        </dependency>
        
        <!-- 熔断器 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId>
        </dependency>
        
        <!-- Actuator监控 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    </dependencies>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>com.alibaba.cloud</groupId>
                <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                <version>2022.0.0.0</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```


#### 3.1.2 配置文件

```yaml
# application.yml
server:
  port: 8080

spring:
  application:
    name: gateway-service
  
  # Nacos配置
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
        namespace: dev
    
    # Gateway配置
    gateway:
      # 服务发现路由
      discovery:
        locator:
          enabled: true
          lower-case-service-id: true
      
      # 全局默认过滤器
      default-filters:
        - AddResponseHeader=X-Response-Default, Gateway
      
      # 路由配置
      routes:
        # 用户服务路由
        - id: user_service
          uri: lb://user-service
          predicates:
            - Path=/api/user/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
                key-resolver: "#{@ipKeyResolver}"
        
        # 订单服务路由
        - id: order_service
          uri: lb://order-service
          predicates:
            - Path=/api/order/**
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: orderCircuitBreaker
                fallbackUri: forward:/fallback
      
      # 全局CORS配置
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "*"
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
            allowedHeaders: "*"
            maxAge: 3600
  
  # Redis配置
  redis:
    host: localhost
    port: 6379
    password: 
    database: 0

# Actuator配置
management:
  endpoints:
    web:
      exposure:
        include: "*"
  endpoint:
    gateway:
      enabled: true

# 日志配置
logging:
  level:
    org.springframework.cloud.gateway: DEBUG
    reactor.netty: INFO
```

#### 3.1.3 启动类

```java
package com.example.gateway;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

/**
 * Gateway网关启动类
 * 
 * @author erik.zhou
 */
@SpringBootApplication
@EnableDiscoveryClient
public class GatewayApplication {
    
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }
}
```

### 3.2 完整案例：统一认证网关

#### 3.2.1 认证过滤器

```java
package com.example.gateway.filter;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.HttpStatus;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.http.server.reactive.ServerHttpResponse;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

import java.util.Arrays;
import java.util.List;

/**
 * 统一认证过滤器
 * 
 * @author erik.zhou
 */
@Component
public class AuthenticationFilter implements GlobalFilter, Ordered {
    
    // 白名单路径
    private static final List<String> WHITE_LIST = Arrays.asList(
        "/api/auth/login",
        "/api/auth/register",
        "/actuator/**"
    );
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getPath().value();
        
        // 检查是否在白名单中
        if (isWhiteList(path)) {
            return chain.filter(exchange);
        }
        
        // 获取token
        String token = request.getHeaders().getFirst("Authorization");
        
        if (token == null || token.isEmpty()) {
            return unauthorized(exchange);
        }
        
        // 验证token（这里简化处理，实际应该调用认证服务）
        if (!validateToken(token)) {
            return unauthorized(exchange);
        }
        
        // 解析用户信息并添加到请求头
        String userId = parseUserId(token);
        ServerHttpRequest mutatedRequest = request.mutate()
            .header("X-User-Id", userId)
            .build();
        
        return chain.filter(exchange.mutate().request(mutatedRequest).build());
    }
    
    private boolean isWhiteList(String path) {
        return WHITE_LIST.stream().anyMatch(pattern -> 
            path.matches(pattern.replace("**", ".*"))
        );
    }
    
    private boolean validateToken(String token) {
        // 实际应该调用认证服务验证token
        return token.startsWith("Bearer ");
    }
    
    private String parseUserId(String token) {
        // 实际应该解析JWT token获取用户ID
        return "user123";
    }
    
    private Mono<Void> unauthorized(ServerWebExchange exchange) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(HttpStatus.UNAUTHORIZED);
        return response.setComplete();
    }
    
    @Override
    public int getOrder() {
        return -200;
    }
}
```


#### 3.2.2 日志记录过滤器

```java
package com.example.gateway.filter;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.http.server.reactive.ServerHttpRequest;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

/**
 * 请求日志记录过滤器
 * 
 * @author erik.zhou
 */
@Component
public class LoggingFilter implements GlobalFilter, Ordered {
    
    private static final Logger logger = LoggerFactory.getLogger(LoggingFilter.class);
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        long startTime = System.currentTimeMillis();
        
        String requestId = request.getHeaders().getFirst("X-Trace-Id");
        String method = request.getMethod().name();
        String path = request.getPath().value();
        String clientIp = request.getRemoteAddress().getAddress().getHostAddress();
        
        logger.info("请求开始 - TraceId: {}, Method: {}, Path: {}, ClientIP: {}", 
            requestId, method, path, clientIp);
        
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            long endTime = System.currentTimeMillis();
            long duration = endTime - startTime;
            int statusCode = exchange.getResponse().getStatusCode().value();
            
            logger.info("请求结束 - TraceId: {}, StatusCode: {}, Duration: {}ms", 
                requestId, statusCode, duration);
        }));
    }
    
    @Override
    public int getOrder() {
        return -300;
    }
}
```

### 3.3 灰度发布案例

```yaml
spring:
  cloud:
    gateway:
      routes:
        # 生产版本（80%流量）
        - id: product_service_v1
          uri: lb://product-service-v1
          predicates:
            - Path=/api/product/**
            - Weight=product-group, 80
          filters:
            - StripPrefix=1
        
        # 灰度版本（20%流量）
        - id: product_service_v2
          uri: lb://product-service-v2
          predicates:
            - Path=/api/product/**
            - Weight=product-group, 20
          filters:
            - StripPrefix=1
            - AddResponseHeader=X-Version, v2
```

## ✨ 最佳实践

### 4.1 性能优化

#### 4.1.1 连接池配置

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        # 连接超时时间
        connect-timeout: 1000
        # 响应超时时间
        response-timeout: 5s
        # 连接池配置
        pool:
          # 连接池类型
          type: ELASTIC
          # 最大连接数
          max-connections: 1000
          # 获取连接超时时间
          acquire-timeout: 45000
```

#### 4.1.2 WebFlux配置

```yaml
spring:
  webflux:
    # 最大内存大小
    max-in-memory-size: 10MB
```

#### 4.1.3 Netty配置

```java
package com.example.gateway.config;

import io.netty.channel.ChannelOption;
import org.springframework.boot.web.embedded.netty.NettyReactiveWebServerFactory;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.context.annotation.Configuration;

/**
 * Netty服务器配置
 * 
 * @author erik.zhou
 */
@Configuration
public class NettyConfig implements WebServerFactoryCustomizer<NettyReactiveWebServerFactory> {
    
    @Override
    public void customize(NettyReactiveWebServerFactory factory) {
        factory.addServerCustomizers(httpServer -> httpServer
            .option(ChannelOption.SO_BACKLOG, 1024)
            .option(ChannelOption.SO_KEEPALIVE, true)
            .option(ChannelOption.TCP_NODELAY, true)
        );
    }
}
```


### 4.2 安全配置

#### 4.2.1 HTTPS配置

```yaml
server:
  port: 8443
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: changeit
    key-store-type: PKCS12
    key-alias: gateway
```

#### 4.2.2 敏感信息脱敏

```java
package com.example.gateway.filter;

import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

/**
 * 敏感信息脱敏过滤器
 * 
 * @author erik.zhou
 */
@Component
public class SensitiveDataFilter implements GlobalFilter, Ordered {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        // 移除敏感请求头
        exchange.getRequest().mutate()
            .headers(headers -> {
                headers.remove("X-Internal-Token");
                headers.remove("X-Admin-Key");
            })
            .build();
        
        return chain.filter(exchange).then(Mono.fromRunnable(() -> {
            // 移除敏感响应头
            exchange.getResponse().getHeaders().remove("X-Internal-Info");
        }));
    }
    
    @Override
    public int getOrder() {
        return -400;
    }
}
```

### 4.3 监控与告警

#### 4.3.1 自定义监控指标

```java
package com.example.gateway.metrics;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.cloud.gateway.filter.GatewayFilterChain;
import org.springframework.cloud.gateway.filter.GlobalFilter;
import org.springframework.core.Ordered;
import org.springframework.stereotype.Component;
import org.springframework.web.server.ServerWebExchange;
import reactor.core.publisher.Mono;

/**
 * 自定义监控指标过滤器
 * 
 * @author erik.zhou
 */
@Component
public class MetricsFilter implements GlobalFilter, Ordered {
    
    private final Counter requestCounter;
    private final Timer requestTimer;
    
    public MetricsFilter(MeterRegistry meterRegistry) {
        this.requestCounter = Counter.builder("gateway.requests.total")
            .description("Total number of requests")
            .register(meterRegistry);
        
        this.requestTimer = Timer.builder("gateway.requests.duration")
            .description("Request duration")
            .register(meterRegistry);
    }
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        requestCounter.increment();
        
        return requestTimer.record(() -> chain.filter(exchange));
    }
    
    @Override
    public int getOrder() {
        return -500;
    }
}
```

#### 4.3.2 健康检查

```yaml
management:
  health:
    gateway:
      enabled: true
  endpoint:
    health:
      show-details: always
```

### 4.4 常见陷阱

#### ⚠️ 陷阱1：过滤器执行顺序混乱

**问题**：多个过滤器的执行顺序不符合预期

**解决方案**：
- 实现`Ordered`接口，明确指定order值
- order值越小，优先级越高
- 建议预留足够的order间隔，便于后续插入新过滤器

```java
@Override
public int getOrder() {
    return -100;  // 数字越小，优先级越高
}
```

#### ⚠️ 陷阱2：路由配置不生效

**问题**：修改路由配置后不生效

**解决方案**：
- 使用动态路由时，需要发布`RefreshRoutesEvent`事件
- 或者调用`/actuator/gateway/refresh`端点刷新路由
- 检查路由的order值，确保路由匹配顺序正确

#### ⚠️ 陷阱3：内存溢出

**问题**：处理大文件上传/下载时内存溢出

**解决方案**：
- 配置`spring.webflux.max-in-memory-size`限制内存大小
- 使用流式处理，避免将整个文件加载到内存
- 考虑使用对象存储服务处理大文件

```yaml
spring:
  webflux:
    max-in-memory-size: 10MB
  codec:
    max-in-memory-size: 10MB
```


#### ⚠️ 陷阱4：响应式编程理解不足

**问题**：在过滤器中使用阻塞式代码，导致性能下降

**解决方案**：
- 避免在过滤器中使用阻塞式IO操作
- 使用响应式的WebClient替代RestTemplate
- 理解Mono和Flux的使用场景

```java
// ❌ 错误示例：阻塞式调用
@Override
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
    String result = restTemplate.getForObject("http://service/api", String.class);
    // 处理result...
    return chain.filter(exchange);
}

// ✅ 正确示例：响应式调用
@Override
public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
    return webClient.get()
        .uri("http://service/api")
        .retrieve()
        .bodyToMono(String.class)
        .flatMap(result -> {
            // 处理result...
            return chain.filter(exchange);
        });
}
```

## ❓ 常见问题

### Q1: Spring Cloud Gateway与Zuul的区别？

**A**: 主要区别如下：

| 特性 | Spring Cloud Gateway | Zuul 1.x |
|------|---------------------|----------|
| 底层框架 | Spring WebFlux（响应式） | Servlet（阻塞式） |
| 性能 | 高（非阻塞IO） | 相对较低 |
| 编程模型 | 响应式编程 | 传统Servlet |
| 维护状态 | 活跃维护 | 已停止更新 |
| 功能丰富度 | 更丰富（断言、过滤器等） | 相对简单 |

**建议**：新项目优先选择Spring Cloud Gateway。

### Q2: 如何实现灰度发布？

**A**: 使用Weight断言实现流量分配：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: service_v1
          uri: lb://service-v1
          predicates:
            - Path=/api/**
            - Weight=group1, 90  # 90%流量
        
        - id: service_v2
          uri: lb://service-v2
          predicates:
            - Path=/api/**
            - Weight=group1, 10  # 10%流量
```

### Q3: 如何处理跨域问题？

**A**: 配置全局CORS：

```yaml
spring:
  cloud:
    gateway:
      globalcors:
        cors-configurations:
          '[/**]':
            allowedOrigins: "*"
            allowedMethods:
              - GET
              - POST
              - PUT
              - DELETE
            allowedHeaders: "*"
            allowCredentials: true
            maxAge: 3600
```

### Q4: 如何实现接口版本控制？

**A**: 使用Header断言或Path断言：

```yaml
spring:
  cloud:
    gateway:
      routes:
        # 基于Header的版本控制
        - id: api_v1
          uri: lb://service-v1
          predicates:
            - Path=/api/**
            - Header=X-API-Version, v1
        
        # 基于Path的版本控制
        - id: api_v2
          uri: lb://service-v2
          predicates:
            - Path=/api/v2/**
```

### Q5: 如何实现请求重试？

**A**: 使用Retry过滤器：

```yaml
filters:
  - name: Retry
    args:
      retries: 3
      statuses: BAD_GATEWAY,GATEWAY_TIMEOUT
      methods: GET,POST
      backoff:
        firstBackoff: 10ms
        maxBackoff: 50ms
        factor: 2
        basedOnPreviousValue: false
```

### Q6: 如何监控网关性能？

**A**: 集成Actuator和Prometheus：

```yaml
management:
  endpoints:
    web:
      exposure:
        include: "*"
  metrics:
    export:
      prometheus:
        enabled: true
```

访问`/actuator/prometheus`获取监控指标。

### Q7: 如何实现动态路由？

**A**: 参考2.5节的动态路由实现，核心步骤：
1. 注入`RouteDefinitionWriter`
2. 使用`save()`方法保存路由
3. 发布`RefreshRoutesEvent`事件刷新路由

### Q8: 如何处理WebSocket请求？

**A**: Gateway原生支持WebSocket：

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: websocket_route
          uri: ws://localhost:9000
          predicates:
            - Path=/ws/**
```


## 🔗 相关资源

### 官方文档
- [Spring Cloud Gateway官方文档](https://spring.io/projects/spring-cloud-gateway)
- [Spring Cloud Gateway参考指南](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)
- [Spring WebFlux文档](https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html)

### 推荐文章
- [Spring Cloud Gateway深度解析](https://spring.io/blog)
- [响应式编程入门指南](https://projectreactor.io/docs)
- [微服务网关设计模式](https://microservices.io/patterns/apigateway.html)

### 视频教程
- [Spring Cloud Gateway实战教程](https://www.youtube.com/springdevelopers)
- [响应式编程与WebFlux](https://www.youtube.com/springdevelopers)

### 源码仓库
- [Spring Cloud Gateway GitHub](https://github.com/spring-cloud/spring-cloud-gateway)
- [示例项目](https://github.com/spring-cloud-samples/spring-cloud-gateway-sample)

## 📝 学习检查清单

- [ ] 理解Spring Cloud Gateway的核心概念（Route、Predicate、Filter）
- [ ] 掌握YAML和Java代码两种路由配置方式
- [ ] 熟练使用常用的断言工厂（Path、Method、Header等）
- [ ] 熟练使用常用的过滤器工厂（AddRequestHeader、RewritePath等）
- [ ] 能够实现自定义GatewayFilter
- [ ] 能够实现自定义GlobalFilter
- [ ] 理解过滤器的执行顺序和优先级
- [ ] 掌握动态路由的实现方式
- [ ] 掌握限流配置和自定义限流键解析器
- [ ] 掌握熔断器集成和降级处理
- [ ] 理解响应式编程的基本概念
- [ ] 能够配置CORS和HTTPS
- [ ] 能够集成Nacos实现服务发现
- [ ] 能够实现统一认证和鉴权
- [ ] 能够配置监控和健康检查
- [ ] 了解性能优化的最佳实践
- [ ] 能够排查常见问题和陷阱

## 📊 技术对比

### Gateway vs Nginx

| 特性 | Spring Cloud Gateway | Nginx |
|------|---------------------|-------|
| 定位 | 应用层网关 | 反向代理服务器 |
| 服务发现 | 原生支持 | 需要额外配置 |
| 动态路由 | 支持 | 需要reload |
| 限流熔断 | 内置支持 | 需要lua脚本 |
| 开发语言 | Java | C |
| 配置方式 | YAML/Java代码 | 配置文件 |
| 性能 | 较高 | 极高 |
| 适用场景 | 微服务网关 | 通用反向代理 |

**建议**：
- 微服务架构：优先选择Spring Cloud Gateway
- 静态资源代理：优先选择Nginx
- 混合架构：Nginx作为入口，Gateway作为微服务网关

## 🎓 进阶学习路径

1. **基础阶段**（1-2周）
   - 掌握路由配置
   - 熟悉常用断言和过滤器
   - 完成基础实战案例

2. **进阶阶段**（2-3周）
   - 实现自定义过滤器
   - 掌握限流和熔断
   - 实现动态路由

3. **高级阶段**（3-4周）
   - 深入理解响应式编程
   - 性能调优
   - 生产环境部署

4. **专家阶段**（持续学习）
   - 源码分析
   - 架构设计
   - 最佳实践总结

## 📌 重点回顾

### 🔥 核心知识点
1. **路由配置**：掌握YAML和Java代码两种配置方式
2. **断言工厂**：理解各种断言的使用场景
3. **过滤器工厂**：熟练使用内置过滤器
4. **自定义过滤器**：能够根据业务需求实现自定义逻辑
5. **动态路由**：实现运行时路由管理
6. **限流熔断**：保护后端服务

### ⚠️ 难点突破
1. **响应式编程**：理解Mono和Flux的使用
2. **过滤器顺序**：掌握order值的设置规则
3. **动态路由**：理解路由刷新机制
4. **性能优化**：避免阻塞式代码

### 💡 实战技巧
1. 使用全局过滤器处理通用逻辑（认证、日志等）
2. 合理设置过滤器优先级，避免执行顺序混乱
3. 使用限流保护后端服务，防止过载
4. 配置熔断器，提高系统容错能力
5. 使用动态路由实现灰度发布
6. 集成监控，及时发现和解决问题

---

**文档版本**: 1.0.0  
**最后更新**: 2024-01-04  
**文档来源**: Context7 - Spring Cloud Gateway官方文档  
**适用版本**: Spring Cloud Gateway 4.1.x, Spring Boot 3.2.x  
**作者**: @author erik.zhou
