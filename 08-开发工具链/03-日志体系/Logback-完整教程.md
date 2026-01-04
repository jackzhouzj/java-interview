# Logback 完整教程

## 📋 目录
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: 1.4.x / 1.5.x
- **官方文档**: https://logback.qos.ch/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、SLF4J、XML配置
- **文档来源**: Context7 + 官方文档
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Logback的架构和核心组件
- [ ] 掌握Logback的配置方法
- [ ] 掌握Appender、Encoder、Filter的使用
- [ ] 掌握日志滚动策略的配置
- [ ] 掌握异步日志的配置和优化
- [ ] 理解Logback的性能优化技巧

## 📖 基础概念

### 1.1 什么是Logback

Logback是由Log4j的创始人Ceki Gülcü设计的日志框架，是SLF4J的原生实现。它旨在作为Log4j的继任者，提供更快的性能、更丰富的功能和更灵活的配置。

**核心优势**：
- 性能优异：比Log4j 1.x快约10倍
- 原生支持SLF4J：无需桥接器
- 自动重载配置：修改配置文件后自动生效
- 强大的过滤器：支持复杂的日志过滤逻辑
- 丰富的Appender：支持多种日志输出方式

### 1.2 核心概念

Logback分为三个模块：

- **logback-core**: 核心模块，提供基础功能
- **logback-classic**: 实现SLF4J API，提供Logger功能
- **logback-access**: 集成Servlet容器，提供HTTP访问日志

**核心组件**：

- **Logger**: 日志记录器，负责生成日志事件
- **Appender**: 日志输出目的地（控制台、文件、数据库等）
- **Encoder**: 日志格式化器，将日志事件转换为字节数组
- **Layout**: 日志布局器，定义日志输出格式（已被Encoder替代）
- **Filter**: 日志过滤器，决定日志事件是否被处理

### 1.3 应用场景
- Spring Boot应用的默认日志框架
- 企业级Java应用的日志解决方案
- 需要高性能日志输出的系统
- 需要灵活日志配置的项目
- 微服务架构中的统一日志标准

## 🔥 核心特性 (重点)

### 2.1 配置文件加载机制 🔥

Logback在启动时按以下顺序查找配置文件：

1. **logback-test.xml**: 测试环境配置（优先级最高）
2. **logback.groovy**: Groovy脚本配置
3. **logback.xml**: 标准配置文件
4. **默认配置**: 如果都不存在，使用BasicConfigurator

**配置文件位置**：
- 放在classpath根目录（src/main/resources）
- Spring Boot项目可使用application.yml配置

### 2.2 Appender详解 🔥

Appender定义日志输出的目的地和方式：

#### 常用Appender类型

| Appender | 说明 | 使用场景 |
|----------|------|---------|
| ConsoleAppender | 输出到控制台 | 开发环境、容器化应用 |
| FileAppender | 输出到文件 | 简单的文件日志 |
| RollingFileAppender | 滚动文件输出 | 生产环境（推荐） |
| AsyncAppender | 异步日志输出 | 高并发场景 |
| SMTPAppender | 发送邮件 | 错误告警 |
| DBAppender | 输出到数据库 | 日志审计 |

### 2.3 滚动策略 (⚠️ 难点)

RollingFileAppender支持多种滚动策略：

#### 1. TimeBasedRollingPolicy（基于时间）

按时间滚动日志文件，支持按天、按小时等：

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <!-- 按天滚动 -->
    <fileNamePattern>logs/app.%d{yyyy-MM-dd}.log</fileNamePattern>
    <!-- 保留30天 -->
    <maxHistory>30</maxHistory>
    <!-- 总大小限制3GB -->
    <totalSizeCap>3GB</totalSizeCap>
</rollingPolicy>
```

#### 2. SizeAndTimeBasedRollingPolicy（基于大小和时间）

同时按时间和大小滚动：

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
    <!-- 按天滚动，单个文件超过100MB时分割 -->
    <fileNamePattern>logs/app.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
    <maxFileSize>100MB</maxFileSize>
    <maxHistory>30</maxHistory>
    <totalSizeCap>3GB</totalSizeCap>
</rollingPolicy>
```

### 2.4 Filter过滤器

Filter用于精确控制日志输出：

#### 常用Filter类型

- **LevelFilter**: 精确匹配日志级别
- **ThresholdFilter**: 过滤低于指定级别的日志
- **EvaluatorFilter**: 基于表达式的复杂过滤
- **MarkerFilter**: 基于Marker的过滤

### 2.5 异步日志 🔥

AsyncAppender通过队列实现异步日志输出，提升性能：

**工作原理**：
1. 日志事件放入队列
2. 后台线程从队列取出并写入
3. 主线程不阻塞，继续执行

**性能提升**：
- 减少I/O阻塞
- 提高应用吞吐量
- 适合高并发场景


## 💻 实战应用

### 3.1 环境搭建

#### Maven依赖

```xml
<!-- Logback Classic（包含logback-core和slf4j-api） -->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.4.11</version>
</dependency>

<!-- Logback Access（可选，用于HTTP访问日志） -->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-access</artifactId>
    <version>1.4.11</version>
</dependency>
```

#### Gradle依赖

```groovy
dependencies {
    implementation 'ch.qos.logback:logback-classic:1.4.11'
    implementation 'ch.qos.logback:logback-access:1.4.11'
}
```

### 3.2 快速开始

#### 基础配置示例（logback.xml）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 定义日志输出格式 -->
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"/>
    
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    
    <!-- 文件输出 -->
    <appender name="FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/application.log</file>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    
    <!-- Root Logger -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

#### Java代码使用

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Logback使用示例
 * 
 * @author erik.zhou
 */
public class UserService {
    private static final Logger logger = LoggerFactory.getLogger(UserService.class);
    
    public void createUser(String username) {
        logger.info("创建用户：{}", username);
        logger.debug("用户详细信息：{}", getUserDetails(username));
    }
    
    private String getUserDetails(String username) {
        return "User details for " + username;
    }
}
```

### 3.3 进阶配置

#### 滚动文件配置（生产环境推荐）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 定义变量 -->
    <property name="LOG_HOME" value="logs"/>
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n"/>
    
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>
    
    <!-- INFO级别日志 -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/info.log</file>
        
        <!-- 滚动策略：按天滚动，单文件超过100MB时分割 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/info.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>3GB</totalSizeCap>
        </rollingPolicy>
        
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
        
        <!-- 只记录INFO级别 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    
    <!-- ERROR级别日志 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/error.log</file>
        
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>90</maxHistory>
        </rollingPolicy>
        
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
        
        <!-- 只记录ERROR级别 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>
    
    <!-- Root Logger -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="INFO_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>
</configuration>
```


#### 异步日志配置（高性能场景）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="LOG_HOME" value="logs"/>
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n"/>
    
    <!-- 同步文件Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/application.log</file>
        
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/application.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
            <totalSizeCap>3GB</totalSizeCap>
        </rollingPolicy>
        
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
        </encoder>
    </appender>
    
    <!-- 异步Appender -->
    <appender name="ASYNC_FILE" class="ch.qos.logback.classic.AsyncAppender">
        <!-- 队列大小 -->
        <queueSize>512</queueSize>
        <!-- 丢弃阈值：队列剩余容量小于此值时，丢弃TRACE/DEBUG/INFO日志 -->
        <discardingThreshold>0</discardingThreshold>
        <!-- 是否包含调用者信息（性能影响较大，建议false） -->
        <includeCallerData>false</includeCallerData>
        <!-- 引用同步Appender -->
        <appender-ref ref="FILE"/>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="ASYNC_FILE"/>
    </root>
</configuration>
```

#### 多环境配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 读取Spring Boot配置 -->
    <springProperty scope="context" name="APP_NAME" source="spring.application.name"/>
    <springProperty scope="context" name="LOG_LEVEL" source="logging.level.root" defaultValue="INFO"/>
    
    <!-- 开发环境 -->
    <springProfile name="dev">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
        
        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
    
    <!-- 生产环境 -->
    <springProfile name="prod">
        <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>logs/${APP_NAME}.log</file>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>logs/${APP_NAME}.%d{yyyy-MM-dd}.log</fileNamePattern>
                <maxHistory>30</maxHistory>
            </rollingPolicy>
            <encoder>
                <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            </encoder>
        </appender>
        
        <root level="${LOG_LEVEL}">
            <appender-ref ref="FILE"/>
        </root>
    </springProfile>
</configuration>
```

#### JSON格式日志配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- JSON格式输出（适合ELK收集） -->
    <appender name="JSON_FILE" class="ch.qos.logback.core.FileAppender">
        <file>logs/application.json</file>
        <encoder class="ch.qos.logback.classic.encoder.JsonEncoder">
            <withSequenceNumber>true</withSequenceNumber>
            <withTimestamp>true</withTimestamp>
            <withNanoseconds>true</withNanoseconds>
            <withLevel>true</withLevel>
            <withThreadName>true</withThreadName>
            <withLoggerName>true</withLoggerName>
            <withContext>true</withContext>
            <withMarkers>true</withMarkers>
            <withMDC>true</withMDC>
            <withKVPList>true</withKVPList>
            <withMessage>true</withMessage>
            <withArguments>true</withArguments>
            <withThrowable>true</withThrowable>
            <withFormattedMessage>false</withFormattedMessage>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="JSON_FILE"/>
    </root>
</configuration>
```

#### 自定义Logger配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- 为特定包配置日志级别 -->
    <logger name="com.example.service" level="DEBUG"/>
    <logger name="com.example.dao" level="TRACE"/>
    
    <!-- 关闭第三方库的日志 -->
    <logger name="org.springframework" level="WARN"/>
    <logger name="org.hibernate" level="WARN"/>
    
    <!-- SQL日志（MyBatis） -->
    <logger name="com.example.mapper" level="DEBUG"/>
    
    <!-- 不继承root的appender -->
    <logger name="com.example.special" level="INFO" additivity="false">
        <appender-ref ref="CONSOLE"/>
    </logger>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```


## ✨ 最佳实践

### 4.1 性能优化

#### 1. 使用异步日志

```xml
<!-- 高并发场景必须使用异步日志 -->
<appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
    <queueSize>512</queueSize>
    <discardingThreshold>0</discardingThreshold>
    <includeCallerData>false</includeCallerData>
    <appender-ref ref="FILE"/>
</appender>
```

**注意事项**：
- queueSize根据日志量调整（默认256）
- includeCallerData=false可提升性能
- discardingThreshold=0避免丢失日志

#### 2. 合理设置日志级别

```xml
<!-- 生产环境 -->
<root level="INFO">
    <appender-ref ref="FILE"/>
</root>

<!-- 开发环境 -->
<root level="DEBUG">
    <appender-ref ref="CONSOLE"/>
</root>
```

#### 3. 避免频繁创建Logger

```java
// ❌ 错误：每次调用都创建Logger
public void method() {
    Logger logger = LoggerFactory.getLogger(MyClass.class);
    logger.info("message");
}

// ✅ 正确：使用静态常量
public class MyClass {
    private static final Logger logger = LoggerFactory.getLogger(MyClass.class);
    
    public void method() {
        logger.info("message");
    }
}
```

### 4.2 配置规范

#### 1. 日志文件命名规范

```xml
<!-- 按日志级别分文件 -->
<file>logs/info.log</file>
<file>logs/error.log</file>

<!-- 按模块分文件 -->
<file>logs/user-service.log</file>
<file>logs/order-service.log</file>

<!-- 滚动文件命名 -->
<fileNamePattern>logs/app.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
```

#### 2. 日志保留策略

```xml
<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
    <!-- 保留天数 -->
    <maxHistory>30</maxHistory>
    <!-- 总大小限制 -->
    <totalSizeCap>3GB</totalSizeCap>
    <!-- 启动时清理 -->
    <cleanHistoryOnStart>true</cleanHistoryOnStart>
</rollingPolicy>
```

#### 3. 日志格式规范

```xml
<!-- 标准格式 -->
<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>

<!-- 包含MDC信息 -->
<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{requestId}] %-5level %logger{50} - %msg%n</pattern>

<!-- 彩色输出（控制台） -->
<pattern>%d{HH:mm:ss.SSS} [%thread] %highlight(%-5level) %cyan(%logger{36}) - %msg%n</pattern>
```

### 4.3 常见陷阱

#### ⚠️ 陷阱1：日志文件权限问题

```bash
# 确保应用有写权限
chmod 755 logs/
chown app:app logs/

# Docker容器中注意挂载目录权限
docker run -v /host/logs:/app/logs myapp
```

#### ⚠️ 陷阱2：日志文件过大导致磁盘满

```xml
<!-- ✅ 必须配置滚动策略和大小限制 -->
<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
    <fileNamePattern>logs/app.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
    <maxFileSize>100MB</maxFileSize>
    <maxHistory>30</maxHistory>
    <totalSizeCap>3GB</totalSizeCap>
</rollingPolicy>
```

#### ⚠️ 陷阱3：异步日志丢失

```xml
<!-- ❌ 错误：队列满时会丢失日志 -->
<appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
    <queueSize>256</queueSize>
    <discardingThreshold>20</discardingThreshold>
</appender>

<!-- ✅ 正确：设置合理的队列大小和丢弃阈值 -->
<appender name="ASYNC" class="ch.qos.logback.classic.AsyncAppender">
    <queueSize>1024</queueSize>
    <discardingThreshold>0</discardingThreshold>
    <!-- 队列满时阻塞，不丢失日志 -->
    <neverBlock>false</neverBlock>
</appender>
```

#### ⚠️ 陷阱4：配置文件未生效

```java
// 检查配置文件位置
// 1. src/main/resources/logback.xml
// 2. src/main/resources/logback-spring.xml (Spring Boot)

// 启动时查看日志
// Logback会打印配置文件加载信息
```

#### ⚠️ 陷阱5：循环依赖导致StackOverflowError

```xml
<!-- ❌ 错误：Logger的additivity导致循环 -->
<logger name="com.example" level="DEBUG" additivity="true">
    <appender-ref ref="FILE"/>
</logger>

<root level="INFO">
    <appender-ref ref="FILE"/>
</root>

<!-- ✅ 正确：根据需要设置additivity -->
<logger name="com.example" level="DEBUG" additivity="false">
    <appender-ref ref="FILE"/>
</logger>
```


## ❓ 常见问题

### Q1: Logback与Log4j2如何选择？

**A**: 

| 对比项 | Logback | Log4j2 |
|--------|---------|--------|
| 性能 | ⭐⭐⭐⭐ 优秀 | ⭐⭐⭐⭐⭐ 更优秀 |
| 配置 | ⭐⭐⭐⭐⭐ 简单直观 | ⭐⭐⭐⭐ 功能更强 |
| Spring Boot | ⭐⭐⭐⭐⭐ 默认集成 | ⭐⭐⭐ 需要额外配置 |
| 社区 | ⭐⭐⭐⭐⭐ 广泛使用 | ⭐⭐⭐⭐ 持续增长 |
| 异步日志 | ⭐⭐⭐⭐ AsyncAppender | ⭐⭐⭐⭐⭐ 原生异步 |

**推荐**：
- Spring Boot项目：优先选择Logback（默认集成）
- 高性能要求：选择Log4j2（异步性能更好）
- 新项目：两者都可以，根据团队熟悉度选择

### Q2: 如何在Spring Boot中自定义Logback配置？

**A**: 

```yaml
# application.yml
logging:
  config: classpath:logback-spring.xml
  level:
    root: INFO
    com.example: DEBUG
```

```xml
<!-- logback-spring.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    
    <springProperty scope="context" name="APP_NAME" source="spring.application.name"/>
    
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
        </encoder>
    </appender>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

### Q3: 如何实现日志脱敏？

**A**: 

```java
import ch.qos.logback.classic.PatternLayout;
import ch.qos.logback.classic.spi.ILoggingEvent;

/**
 * 自定义脱敏Layout
 * 
 * @author erik.zhou
 */
public class MaskingPatternLayout extends PatternLayout {
    
    private static final Pattern PHONE_PATTERN = Pattern.compile("(1[3-9]\\d)(\\d{4})(\\d{4})");
    private static final Pattern ID_CARD_PATTERN = Pattern.compile("(\\d{6})(\\d{8})(\\d{4})");
    
    @Override
    public String doLayout(ILoggingEvent event) {
        String message = super.doLayout(event);
        
        // 手机号脱敏：138****5678
        message = PHONE_PATTERN.matcher(message).replaceAll("$1****$3");
        
        // 身份证脱敏：110101********1234
        message = ID_CARD_PATTERN.matcher(message).replaceAll("$1********$3");
        
        return message;
    }
}
```

配置使用：

```xml
<appender name="FILE" class="ch.qos.logback.core.FileAppender">
    <file>logs/app.log</file>
    <encoder class="ch.qos.logback.core.encoder.LayoutWrappingEncoder">
        <layout class="com.example.MaskingPatternLayout">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </layout>
    </encoder>
</appender>
```

### Q4: 如何动态修改日志级别？

**A**: 

```java
import ch.qos.logback.classic.Level;
import ch.qos.logback.classic.Logger;
import ch.qos.logback.classic.LoggerContext;
import org.slf4j.LoggerFactory;

/**
 * 动态修改日志级别
 * 
 * @author erik.zhou
 */
public class LogLevelManager {
    
    public static void setLogLevel(String loggerName, String level) {
        LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
        Logger logger = loggerContext.getLogger(loggerName);
        logger.setLevel(Level.toLevel(level));
    }
    
    public static void setRootLogLevel(String level) {
        LoggerContext loggerContext = (LoggerContext) LoggerFactory.getILoggerFactory();
        Logger rootLogger = loggerContext.getLogger(Logger.ROOT_LOGGER_NAME);
        rootLogger.setLevel(Level.toLevel(level));
    }
}

// 使用示例
LogLevelManager.setLogLevel("com.example.service", "DEBUG");
LogLevelManager.setRootLogLevel("INFO");
```

Spring Boot Actuator方式：

```yaml
# application.yml
management:
  endpoints:
    web:
      exposure:
        include: loggers
```

```bash
# 查看日志级别
curl http://localhost:8080/actuator/loggers/com.example.service

# 修改日志级别
curl -X POST http://localhost:8080/actuator/loggers/com.example.service \
  -H "Content-Type: application/json" \
  -d '{"configuredLevel": "DEBUG"}'
```

### Q5: 如何监控日志输出性能？

**A**: 

```xml
<!-- 开启Logback内部日志 -->
<configuration debug="true">
    <!-- 配置内容 -->
</configuration>

<!-- 或者只开启状态监听 -->
<configuration>
    <statusListener class="ch.qos.logback.core.status.OnConsoleStatusListener"/>
    <!-- 配置内容 -->
</configuration>
```

使用JMX监控：

```xml
<configuration>
    <!-- 启用JMX -->
    <jmxConfigurator/>
    
    <!-- 配置内容 -->
</configuration>
```

通过JConsole或VisualVM连接应用，查看Logback的MBean信息。

### Q6: 如何处理日志中的异常堆栈？

**A**: 

```xml
<!-- 限制异常堆栈深度 -->
<encoder>
    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n%ex{10}</pattern>
</encoder>

<!-- 完整堆栈 -->
<encoder>
    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n%ex{full}</pattern>
</encoder>

<!-- 短堆栈（只显示异常消息） -->
<encoder>
    <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n%ex{short}</pattern>
</encoder>
```


## 🔗 相关资源

### 官方资源
- [Logback官方网站](https://logback.qos.ch/)
- [Logback文档](https://logback.qos.ch/manual/index.html)
- [Logback GitHub仓库](https://github.com/qos-ch/logback)
- [Logback配置示例](https://logback.qos.ch/manual/configuration.html)

### 推荐文章
- [Logback架构详解](https://logback.qos.ch/manual/architecture.html)
- [Logback性能优化](https://logback.qos.ch/manual/appenders.html#AsyncAppender)
- [Spring Boot日志配置](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging)
- [阿里巴巴Java开发手册 - 日志规约](https://github.com/alibaba/p3c)

### 相关技术
- [SLF4J完整教程](SLF4J-完整教程.md) - 日志门面
- [ELK完整教程](ELK-完整教程.md) - 日志收集与分析
- [Log4j2完整教程](./Log4j2-完整教程.md) - 高性能日志框架

## 📝 学习检查清单

- [ ] 理解Logback的架构和核心组件
- [ ] 掌握Logback配置文件的编写
- [ ] 掌握常用Appender的配置和使用
- [ ] 理解日志滚动策略的配置方法
- [ ] 掌握异步日志的配置和优化
- [ ] 了解Filter的使用场景和配置
- [ ] 掌握多环境配置的方法
- [ ] 理解日志性能优化技巧
- [ ] 掌握日志脱敏的实现方法
- [ ] 了解动态修改日志级别的方法
- [ ] 能够在实际项目中正确配置和使用Logback

## 📊 Logback配置模板

### 生产环境完整配置模板

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 定义变量 -->
    <property name="LOG_HOME" value="logs"/>
    <property name="APP_NAME" value="myapp"/>
    <property name="LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] [%X{requestId}] %-5level %logger{50} - %msg%n"/>
    
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %highlight(%-5level) %cyan(%logger{36}) - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    
    <!-- INFO日志 -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/${APP_NAME}-info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/${APP_NAME}-info.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>3GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    
    <!-- ERROR日志 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_HOME}/${APP_NAME}-error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/${APP_NAME}-error.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>90</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>
    
    <!-- 异步INFO日志 -->
    <appender name="ASYNC_INFO" class="ch.qos.logback.classic.AsyncAppender">
        <queueSize>1024</queueSize>
        <discardingThreshold>0</discardingThreshold>
        <includeCallerData>false</includeCallerData>
        <neverBlock>false</neverBlock>
        <appender-ref ref="INFO_FILE"/>
    </appender>
    
    <!-- 异步ERROR日志 -->
    <appender name="ASYNC_ERROR" class="ch.qos.logback.classic.AsyncAppender">
        <queueSize>512</queueSize>
        <discardingThreshold>0</discardingThreshold>
        <includeCallerData>false</includeCallerData>
        <neverBlock>false</neverBlock>
        <appender-ref ref="ERROR_FILE"/>
    </appender>
    
    <!-- 业务日志 -->
    <logger name="com.example.service" level="INFO" additivity="false">
        <appender-ref ref="ASYNC_INFO"/>
        <appender-ref ref="ASYNC_ERROR"/>
    </logger>
    
    <!-- SQL日志 -->
    <logger name="com.example.mapper" level="DEBUG" additivity="false">
        <appender-ref ref="CONSOLE"/>
    </logger>
    
    <!-- 第三方库日志 -->
    <logger name="org.springframework" level="WARN"/>
    <logger name="org.hibernate" level="WARN"/>
    <logger name="com.zaxxer.hikari" level="WARN"/>
    
    <!-- Root Logger -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="ASYNC_INFO"/>
        <appender-ref ref="ASYNC_ERROR"/>
    </root>
</configuration>
```

## 🎓 进阶学习路径

1. **基础阶段**（1-2天）
   - 理解Logback架构
   - 掌握基本配置
   - 学习常用Appender

2. **进阶阶段**（3-5天）
   - 掌握滚动策略
   - 学习异步日志
   - 理解Filter机制

3. **高级阶段**（1周）
   - 性能优化
   - 日志脱敏
   - 动态配置
   - 监控告警

4. **实战阶段**（持续）
   - 生产环境配置
   - 问题排查
   - 性能调优

---

**文档版本**: v1.0  
**最后更新**: 2024-01-04  
**维护者**: @author erik.zhou
