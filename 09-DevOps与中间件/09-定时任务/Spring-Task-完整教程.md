# Spring Task 完整教程

## 📋 目录
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: Spring Framework 6.1.x / Spring Boot 3.2.x
- **官方文档**: https://docs.spring.io/spring-framework/reference/integration/scheduling.html
- **学习难度**: ⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、Spring Framework基础、多线程基础
- **文档来源**: Context7 - Spring Framework官方文档
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Spring Task的核心概念和工作原理
- [ ] 掌握@Scheduled注解的各种使用方式
- [ ] 熟练使用Cron表达式配置定时任务
- [ ] 掌握任务调度器的配置和优化
- [ ] 了解定时任务的最佳实践和常见陷阱

## 📖 基础概念

### 1.1 什么是Spring Task

Spring Task是Spring Framework提供的轻量级任务调度框架，支持通过注解方式声明式地配置定时任务。它提供了简单易用的API，无需引入第三方依赖即可实现基本的定时任务功能。

**核心特点**：
- **轻量级**: 无需额外依赖，Spring Framework内置支持
- **声明式**: 通过@Scheduled注解配置，代码简洁
- **灵活性**: 支持固定频率、固定延迟、Cron表达式等多种调度方式
- **易集成**: 与Spring生态无缝集成

### 1.2 核心概念

- **@EnableScheduling**: 启用Spring的定时任务支持
- **@Scheduled**: 标记方法为定时任务
- **TaskScheduler**: 任务调度器，负责执行定时任务
- **Cron表达式**: 用于配置复杂的定时规则
- **fixedRate**: 固定频率执行
- **fixedDelay**: 固定延迟执行

### 1.3 应用场景

- **数据同步**: 定期从外部系统同步数据
- **报表生成**: 每日/每周/每月自动生成业务报表
- **缓存刷新**: 定期刷新缓存数据
- **日志清理**: 定期清理过期日志文件
- **健康检查**: 定期检查系统健康状态
- **数据备份**: 定期备份数据库
- **消息推送**: 定时发送通知消息

## 🔥 核心特性 (重点)

### 2.1 启用定时任务支持 🔥

在Spring Boot应用中启用定时任务非常简单，只需在配置类上添加@EnableScheduling注解：

```java
package com.example.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.EnableScheduling;

/**
 * 定时任务配置类
 * @author erik.zhou
 */
@Configuration
@EnableScheduling
public class SchedulingConfig {
    // 启用定时任务支持
}
```

### 2.2 @Scheduled注解详解 🔥

@Scheduled注解支持多种调度方式：

#### 2.2.1 fixedRate - 固定频率执行

```java
package com.example.task;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * 定时任务示例
 * @author erik.zhou
 */
@Component
public class ScheduledTasks {

    private static final Logger logger = LoggerFactory.getLogger(ScheduledTasks.class);

    /**
     * 固定频率执行 - 每5秒执行一次，不管上次执行是否完成
     */
    @Scheduled(fixedRate = 5000)
    public void refreshCache() {
        logger.info("刷新缓存，执行时间: {}", LocalDateTime.now());
        // 缓存刷新逻辑
    }
}
```

#### 2.2.2 fixedDelay - 固定延迟执行

```java
/**
 * 固定延迟执行 - 上次执行完成后等待10秒再执行
 */
@Scheduled(fixedDelay = 10000)
public void cleanupTempFiles() {
    logger.info("清理临时文件");
    // 清理逻辑
}
```

#### 2.2.3 initialDelay - 初始延迟

```java
/**
 * 初始延迟 - 应用启动30秒后开始执行，之后每60秒执行一次
 */
@Scheduled(initialDelay = 30000, fixedRate = 60000)
public void heartbeat() {
    logger.info("心跳检查");
    // 健康检查逻辑
}
```

### 2.3 Cron表达式 🔥 (难点)

Cron表达式提供了强大的定时规则配置能力，格式如下：

```text
┌───────────── 秒 (0-59)
│ ┌───────────── 分钟 (0-59)
│ │ ┌───────────── 小时 (0-23)
│ │ │ ┌───────────── 日期 (1-31)
│ │ │ │ ┌───────────── 月份 (1-12 或 JAN-DEC)
│ │ │ │ │ ┌───────────── 星期 (0-7，0和7都表示周日，或 MON-SUN)
│ │ │ │ │ │
* * * * * *
```

**特殊字符说明**：
- `*`: 匹配所有值
- `?`: 不指定值（仅用于日期和星期字段）
- `-`: 范围，如 1-5
- `,`: 列举，如 1,3,5
- `/`: 步长，如 0/15 表示从0开始每15个单位
- `L`: 最后，如 L 表示月份最后一天
- `W`: 工作日
- `#`: 第几个，如 6#3 表示第3个星期五

#### 2.3.1 常用Cron表达式示例

```java
/**
 * 每天凌晨2点执行
 */
@Scheduled(cron = "0 0 2 * * *")
public void generateDailyReport() {
    logger.info("生成每日报表");
    // 报表生成逻辑
}

/**
 * 每个工作日上午9点执行
 */
@Scheduled(cron = "0 0 9 * * MON-FRI")
public void sendMorningNotifications() {
    logger.info("发送早间通知");
    // 通知逻辑
}

/**
 * 每月1号凌晨执行
 */
@Scheduled(cron = "0 0 0 1 * *")
public void monthlyBilling() {
    logger.info("处理月度账单");
    // 账单处理逻辑
}

/**
 * 每15分钟执行一次
 */
@Scheduled(cron = "0 */15 * * * *")
public void checkSystemStatus() {
    logger.info("检查系统状态");
    // 状态检查逻辑
}

/**
 * 每周一凌晨3点执行
 */
@Scheduled(cron = "0 0 3 * * MON")
public void weeklyDataBackup() {
    logger.info("执行周度数据备份");
    // 备份逻辑
}
```

#### 2.3.2 时区配置

```java
/**
 * 指定时区 - 美国东部时间每天中午12点执行
 */
@Scheduled(cron = "0 0 12 * * *", zone = "America/New_York")
public void noonTaskEasternTime() {
    logger.info("执行东部时间中午任务");
}

/**
 * 中国时区 - 每天上午10点执行
 */
@Scheduled(cron = "0 0 10 * * *", zone = "Asia/Shanghai")
public void morningTaskChinaTime() {
    logger.info("执行中国时间上午任务");
}
```

### 2.4 使用配置文件中的Cron表达式

```java
/**
 * 从配置文件读取Cron表达式
 */
@Scheduled(cron = "${scheduled.backup.cron}")
public void backupDatabase() {
    logger.info("开始数据库备份");
    // 备份逻辑
}
```

```properties
# application.properties
scheduled.backup.cron=0 0 3 * * *
```

```yaml
# application.yml
scheduled:
  backup:
    cron: "0 0 3 * * *"
```

### 2.5 任务调度器配置 (难点)

默认情况下，Spring使用单线程调度器。对于多个定时任务，建议配置线程池：

```java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.TaskScheduler;
import org.springframework.scheduling.annotation.EnableScheduling;
import org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler;

/**
 * 定时任务调度器配置
 * @author erik.zhou
 */
@Configuration
@EnableScheduling
public class SchedulingConfig {

    /**
     * 配置任务调度器线程池
     */
    @Bean(destroyMethod = "shutdown")
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        // 线程池大小
        scheduler.setPoolSize(10);
        // 线程名称前缀
        scheduler.setThreadNamePrefix("scheduled-task-");
        // 等待任务完成后再关闭
        scheduler.setWaitForTasksToCompleteOnShutdown(true);
        // 等待时间（秒）
        scheduler.setAwaitTerminationSeconds(60);
        // 拒绝策略
        scheduler.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        scheduler.initialize();
        return scheduler;
    }
}
```

## 💻 实战应用

### 3.1 环境搭建

#### 3.1.1 Maven依赖

Spring Boot项目默认已包含Spring Task，无需额外依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
```

如果是纯Spring项目，需要添加：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>6.1.3</version>
</dependency>
```

### 3.2 快速开始

#### 3.2.1 创建简单定时任务

```java
package com.example.task;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * 简单定时任务示例
 * @author erik.zhou
 */
@Component
public class SimpleScheduledTask {

    private static final Logger logger = LoggerFactory.getLogger(SimpleScheduledTask.class);
    private static final DateTimeFormatter FORMATTER = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");

    /**
     * 每10秒执行一次
     */
    @Scheduled(fixedRate = 10000)
    public void reportCurrentTime() {
        String currentTime = LocalDateTime.now().format(FORMATTER);
        logger.info("当前时间: {}", currentTime);
    }
}
```

#### 3.2.2 启动类配置

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableScheduling;

/**
 * 应用启动类
 * @author erik.zhou
 */
@SpringBootApplication
@EnableScheduling
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 3.3 进阶案例

#### 3.3.1 数据同步任务

```java
package com.example.task;

import com.example.service.DataSyncService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDateTime;

/**
 * 数据同步定时任务
 * @author erik.zhou
 */
@Component
public class DataSyncTask {

    private static final Logger logger = LoggerFactory.getLogger(DataSyncTask.class);

    private final DataSyncService dataSyncService;

    public DataSyncTask(DataSyncService dataSyncService) {
        this.dataSyncService = dataSyncService;
    }

    /**
     * 每小时同步一次用户数据
     */
    @Scheduled(cron = "0 0 * * * *")
    public void syncUserData() {
        logger.info("开始同步用户数据: {}", LocalDateTime.now());
        try {
            int syncCount = dataSyncService.syncUsers();
            logger.info("用户数据同步完成，同步数量: {}", syncCount);
        } catch (Exception e) {
            logger.error("用户数据同步失败", e);
        }
    }

    /**
     * 每天凌晨2点同步订单数据
     */
    @Scheduled(cron = "0 0 2 * * *")
    public void syncOrderData() {
        logger.info("开始同步订单数据: {}", LocalDateTime.now());
        try {
            int syncCount = dataSyncService.syncOrders();
            logger.info("订单数据同步完成，同步数量: {}", syncCount);
        } catch (Exception e) {
            logger.error("订单数据同步失败", e);
        }
    }
}
```

#### 3.3.2 缓存刷新任务

```java
package com.example.task;

import com.example.service.CacheService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

/**
 * 缓存刷新定时任务
 * @author erik.zhou
 */
@Component
public class CacheRefreshTask {

    private static final Logger logger = LoggerFactory.getLogger(CacheRefreshTask.class);

    private final CacheService cacheService;

    public CacheRefreshTask(CacheService cacheService) {
        this.cacheService = cacheService;
    }

    /**
     * 每5分钟刷新热点数据缓存
     */
    @Scheduled(fixedRate = 300000)
    public void refreshHotDataCache() {
        logger.info("开始刷新热点数据缓存");
        cacheService.refreshHotData();
        logger.info("热点数据缓存刷新完成");
    }

    /**
     * 每天凌晨3点清理过期缓存
     */
    @Scheduled(cron = "0 0 3 * * *")
    public void cleanExpiredCache() {
        logger.info("开始清理过期缓存");
        long cleanedCount = cacheService.cleanExpiredCache();
        logger.info("过期缓存清理完成，清理数量: {}", cleanedCount);
    }
}
```

#### 3.3.3 报表生成任务

```java
package com.example.task;

import com.example.service.ReportService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.time.LocalDate;

/**
 * 报表生成定时任务
 * @author erik.zhou
 */
@Component
public class ReportGenerationTask {

    private static final Logger logger = LoggerFactory.getLogger(ReportGenerationTask.class);

    private final ReportService reportService;

    public ReportGenerationTask(ReportService reportService) {
        this.reportService = reportService;
    }

    /**
     * 每天凌晨1点生成昨日报表
     */
    @Scheduled(cron = "0 0 1 * * *")
    public void generateDailyReport() {
        LocalDate yesterday = LocalDate.now().minusDays(1);
        logger.info("开始生成日报: {}", yesterday);
        try {
            reportService.generateDailyReport(yesterday);
            logger.info("日报生成完成: {}", yesterday);
        } catch (Exception e) {
            logger.error("日报生成失败: {}", yesterday, e);
        }
    }

    /**
     * 每周一凌晨2点生成上周报表
     */
    @Scheduled(cron = "0 0 2 * * MON")
    public void generateWeeklyReport() {
        logger.info("开始生成周报");
        try {
            reportService.generateWeeklyReport();
            logger.info("周报生成完成");
        } catch (Exception e) {
            logger.error("周报生成失败", e);
        }
    }

    /**
     * 每月1号凌晨3点生成上月报表
     */
    @Scheduled(cron = "0 0 3 1 * *")
    public void generateMonthlyReport() {
        logger.info("开始生成月报");
        try {
            reportService.generateMonthlyReport();
            logger.info("月报生成完成");
        } catch (Exception e) {
            logger.error("月报生成失败", e);
        }
    }
}
```

#### 3.3.4 动态控制任务启停

```java
package com.example.task;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

/**
 * 可配置的定时任务
 * @author erik.zhou
 */
@Component
@ConditionalOnProperty(prefix = "scheduled.task", name = "enabled", havingValue = "true", matchIfMissing = true)
public class ConfigurableTask {

    private static final Logger logger = LoggerFactory.getLogger(ConfigurableTask.class);

    @Scheduled(cron = "${scheduled.task.cron:0 0 * * * *}")
    public void executeTask() {
        logger.info("执行可配置定时任务");
        // 任务逻辑
    }
}
```

```yaml
# application.yml
scheduled:
  task:
    enabled: true
    cron: "0 */30 * * * *"  # 每30分钟执行一次
```

## ✨ 最佳实践

### 4.1 性能优化

#### 4.1.1 配置合适的线程池大小

```java
@Bean(destroyMethod = "shutdown")
public TaskScheduler taskScheduler() {
    ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
    // 根据定时任务数量和执行时间配置线程池大小
    // 建议：线程池大小 = 定时任务数量 * 1.5
    scheduler.setPoolSize(15);
    scheduler.setThreadNamePrefix("scheduled-task-");
    scheduler.setWaitForTasksToCompleteOnShutdown(true);
    scheduler.setAwaitTerminationSeconds(60);
    scheduler.initialize();
    return scheduler;
}
```

#### 4.1.2 避免长时间阻塞

```java
/**
 * 错误示例：任务执行时间过长
 */
@Scheduled(fixedRate = 5000)
public void badTask() {
    // 执行耗时操作，可能超过5秒
    processLargeDataSet();  // ❌ 不推荐
}

/**
 * 正确示例：使用异步执行
 */
@Scheduled(fixedRate = 5000)
public void goodTask() {
    // 提交到异步线程池执行
    asyncExecutor.execute(() -> {
        processLargeDataSet();
    });
}
```

#### 4.1.3 合理选择fixedRate和fixedDelay

```java
/**
 * fixedRate：适用于执行时间短且稳定的任务
 * 不管上次执行是否完成，都会按固定频率触发
 */
@Scheduled(fixedRate = 10000)
public void quickTask() {
    // 快速任务，执行时间 < 10秒
}

/**
 * fixedDelay：适用于执行时间不确定的任务
 * 等待上次执行完成后，再等待指定时间后执行
 */
@Scheduled(fixedDelay = 10000)
public void slowTask() {
    // 耗时任务，执行时间不确定
}
```

### 4.2 常见陷阱

#### 4.2.1 ⚠️ 单线程调度器导致任务阻塞

**问题**：默认情况下，Spring使用单线程调度器，如果一个任务执行时间过长，会阻塞其他任务。

```java
// 任务1：执行时间10秒
@Scheduled(fixedRate = 5000)
public void task1() {
    Thread.sleep(10000);  // 模拟耗时操作
}

// 任务2：会被任务1阻塞
@Scheduled(fixedRate = 5000)
public void task2() {
    logger.info("任务2执行");
}
```

**解决方案**：配置多线程调度器（见4.1.1）

#### 4.2.2 ⚠️ Cron表达式错误

```java
// 错误：日期和星期同时指定
@Scheduled(cron = "0 0 12 15 * MON")  // ❌ 错误

// 正确：使用?表示不指定
@Scheduled(cron = "0 0 12 15 * ?")    // ✅ 正确：每月15号中午12点
@Scheduled(cron = "0 0 12 ? * MON")   // ✅ 正确：每周一中午12点
```

#### 4.2.3 ⚠️ 任务执行异常未处理

```java
/**
 * 错误示例：异常未捕获，导致后续任务不再执行
 */
@Scheduled(fixedRate = 10000)
public void badTask() {
    // 如果抛出异常，后续调度会停止
    processData();  // ❌ 可能抛出异常
}

/**
 * 正确示例：捕获并记录异常
 */
@Scheduled(fixedRate = 10000)
public void goodTask() {
    try {
        processData();
    } catch (Exception e) {
        logger.error("任务执行失败", e);
        // 可以发送告警通知
    }
}
```

#### 4.2.4 ⚠️ 时区问题

```java
/**
 * 问题：未指定时区，使用服务器默认时区
 */
@Scheduled(cron = "0 0 9 * * *")  // 服务器时区的9点
public void task1() { }

/**
 * 解决：明确指定时区
 */
@Scheduled(cron = "0 0 9 * * *", zone = "Asia/Shanghai")  // 中国时区的9点
public void task2() { }
```

### 4.3 监控与日志

#### 4.3.1 添加任务执行监控

```java
package com.example.task;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

/**
 * 带监控的定时任务
 * @author erik.zhou
 */
@Component
public class MonitoredTask {

    private static final Logger logger = LoggerFactory.getLogger(MonitoredTask.class);

    @Scheduled(fixedRate = 60000)
    public void monitoredTask() {
        long startTime = System.currentTimeMillis();
        String taskName = "数据同步任务";
        
        try {
            logger.info("[{}] 开始执行", taskName);
            
            // 执行任务逻辑
            executeBusinessLogic();
            
            long duration = System.currentTimeMillis() - startTime;
            logger.info("[{}] 执行成功，耗时: {}ms", taskName, duration);
            
            // 可以将执行结果上报到监控系统
            reportToMonitoring(taskName, true, duration);
            
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            logger.error("[{}] 执行失败，耗时: {}ms", taskName, duration, e);
            
            // 上报失败信息
            reportToMonitoring(taskName, false, duration);
            
            // 发送告警
            sendAlert(taskName, e);
        }
    }

    private void executeBusinessLogic() {
        // 业务逻辑
    }

    private void reportToMonitoring(String taskName, boolean success, long duration) {
        // 上报到监控系统（如Prometheus）
    }

    private void sendAlert(String taskName, Exception e) {
        // 发送告警（如邮件、短信、钉钉等）
    }
}
```

#### 4.3.2 使用AOP统一处理

```java
package com.example.aspect;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

/**
 * 定时任务切面
 * @author erik.zhou
 */
@Aspect
@Component
public class ScheduledTaskAspect {

    private static final Logger logger = LoggerFactory.getLogger(ScheduledTaskAspect.class);

    @Around("@annotation(org.springframework.scheduling.annotation.Scheduled)")
    public Object aroundScheduledTask(ProceedingJoinPoint joinPoint) throws Throwable {
        String methodName = joinPoint.getSignature().getName();
        String className = joinPoint.getTarget().getClass().getSimpleName();
        String taskName = className + "." + methodName;
        
        long startTime = System.currentTimeMillis();
        logger.info("[定时任务] {} 开始执行", taskName);
        
        try {
            Object result = joinPoint.proceed();
            long duration = System.currentTimeMillis() - startTime;
            logger.info("[定时任务] {} 执行成功，耗时: {}ms", taskName, duration);
            return result;
        } catch (Throwable e) {
            long duration = System.currentTimeMillis() - startTime;
            logger.error("[定时任务] {} 执行失败，耗时: {}ms", taskName, duration, e);
            throw e;
        }
    }
}
```

### 4.4 分布式环境注意事项

#### 4.4.1 ⚠️ 避免重复执行

在分布式环境中，多个实例会同时执行定时任务，需要使用分布式锁：

```java
package com.example.task;

import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

import java.util.concurrent.TimeUnit;

/**
 * 分布式环境下的定时任务
 * @author erik.zhou
 */
@Component
public class DistributedTask {

    private static final Logger logger = LoggerFactory.getLogger(DistributedTask.class);

    private final RedissonClient redissonClient;

    public DistributedTask(RedissonClient redissonClient) {
        this.redissonClient = redissonClient;
    }

    @Scheduled(cron = "0 0 2 * * *")
    public void distributedTask() {
        String lockKey = "scheduled:task:distributedTask";
        RLock lock = redissonClient.getLock(lockKey);
        
        try {
            // 尝试获取锁，最多等待0秒，锁自动释放时间10分钟
            boolean locked = lock.tryLock(0, 600, TimeUnit.SECONDS);
            
            if (locked) {
                logger.info("获取分布式锁成功，开始执行任务");
                // 执行任务逻辑
                executeTask();
                logger.info("任务执行完成");
            } else {
                logger.info("未获取到分布式锁，跳过本次执行");
            }
        } catch (InterruptedException e) {
            logger.error("获取分布式锁被中断", e);
            Thread.currentThread().interrupt();
        } catch (Exception e) {
            logger.error("任务执行失败", e);
        } finally {
            if (lock.isHeldByCurrentThread()) {
                lock.unlock();
                logger.info("释放分布式锁");
            }
        }
    }

    private void executeTask() {
        // 任务逻辑
    }
}
```

## ❓ 常见问题

### Q1: @Scheduled方法可以有参数吗？

A: 不可以。@Scheduled标注的方法必须是无参方法，且返回值类型必须是void。

```java
// ❌ 错误
@Scheduled(fixedRate = 5000)
public void task(String param) { }

// ❌ 错误
@Scheduled(fixedRate = 5000)
public String task() { return "result"; }

// ✅ 正确
@Scheduled(fixedRate = 5000)
public void task() { }
```

### Q2: fixedRate和fixedDelay的区别？

A: 
- **fixedRate**: 固定频率，从上次任务**开始**时间计算下次执行时间
- **fixedDelay**: 固定延迟，从上次任务**结束**时间计算下次执行时间

```java
// fixedRate: 如果任务执行3秒，每5秒触发一次
// 时间轴: 0s(开始) -> 3s(结束) -> 5s(开始) -> 8s(结束) -> 10s(开始)
@Scheduled(fixedRate = 5000)
public void task1() { Thread.sleep(3000); }

// fixedDelay: 如果任务执行3秒，结束后等5秒再执行
// 时间轴: 0s(开始) -> 3s(结束) -> 8s(开始) -> 11s(结束) -> 16s(开始)
@Scheduled(fixedDelay = 5000)
public void task2() { Thread.sleep(3000); }
```

### Q3: 如何动态修改Cron表达式？

A: Spring Task本身不支持动态修改，需要使用ScheduledTaskRegistrar：

```java
package com.example.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.annotation.SchedulingConfigurer;
import org.springframework.scheduling.config.ScheduledTaskRegistrar;
import org.springframework.scheduling.support.CronTrigger;

/**
 * 动态Cron配置
 * @author erik.zhou
 */
@Configuration
public class DynamicSchedulingConfig implements SchedulingConfigurer {

    private String cronExpression = "0 0 * * * *";  // 默认每小时执行

    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.addTriggerTask(
            () -> {
                // 任务逻辑
                System.out.println("执行动态定时任务");
            },
            triggerContext -> {
                // 动态获取Cron表达式
                CronTrigger trigger = new CronTrigger(getCronExpression());
                return trigger.nextExecution(triggerContext);
            }
        );
    }

    public String getCronExpression() {
        // 可以从数据库或配置中心读取
        return cronExpression;
    }

    public void setCronExpression(String cronExpression) {
        this.cronExpression = cronExpression;
    }
}
```

### Q4: 定时任务执行失败后会自动重试吗？

A: 不会。Spring Task不提供自动重试机制，需要自己实现：

```java
@Scheduled(fixedRate = 60000)
public void taskWithRetry() {
    int maxRetries = 3;
    int retryCount = 0;
    
    while (retryCount < maxRetries) {
        try {
            executeTask();
            break;  // 成功则退出
        } catch (Exception e) {
            retryCount++;
            logger.error("任务执行失败，重试次数: {}/{}", retryCount, maxRetries, e);
            
            if (retryCount >= maxRetries) {
                logger.error("任务执行失败，已达最大重试次数");
                // 发送告警
            } else {
                // 等待一段时间后重试
                try {
                    Thread.sleep(5000);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                    break;
                }
            }
        }
    }
}
```

### Q5: 如何在测试环境禁用定时任务？

A: 使用配置文件控制：

```java
@Component
@ConditionalOnProperty(prefix = "scheduling", name = "enabled", havingValue = "true", matchIfMissing = true)
public class MyScheduledTask {
    @Scheduled(fixedRate = 10000)
    public void task() { }
}
```

```yaml
# application-dev.yml (开发环境)
scheduling:
  enabled: false

# application-prod.yml (生产环境)
scheduling:
  enabled: true
```

### Q6: Spring Task适合什么场景？什么时候应该使用Quartz？

A: 
**Spring Task适合**：
- 简单的定时任务
- 单机环境
- 任务数量较少（< 50个）
- 不需要持久化任务状态
- 不需要复杂的任务依赖关系

**应该使用Quartz的场景**：
- 需要持久化任务配置
- 需要集群支持
- 任务数量多（> 50个）
- 需要动态添加/删除任务
- 需要任务依赖和编排
- 需要任务执行历史记录

### Q7: 如何优雅停止定时任务？

A: 配置等待任务完成：

```java
@Bean(destroyMethod = "shutdown")
public TaskScheduler taskScheduler() {
    ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
    scheduler.setPoolSize(10);
    // 关闭时等待任务完成
    scheduler.setWaitForTasksToCompleteOnShutdown(true);
    // 最多等待60秒
    scheduler.setAwaitTerminationSeconds(60);
    scheduler.initialize();
    return scheduler;
}
```

## 🔗 相关资源

### 官方文档
- [Spring Framework - Task Execution and Scheduling](https://docs.spring.io/spring-framework/reference/integration/scheduling.html)
- [Spring Boot - Task Execution and Scheduling](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.task-execution-and-scheduling)

### 推荐文章
- [Spring @Scheduled注解详解](https://www.baeldung.com/spring-scheduled-tasks)
- [Cron表达式完全指南](https://www.baeldung.com/cron-expressions)
- [Spring Task vs Quartz对比](https://www.baeldung.com/spring-task-scheduler-vs-quartz)

### 相关技术
- **Quartz**: 功能更强大的任务调度框架
- **XXL-Job**: 分布式任务调度平台
- **Redisson**: 提供分布式锁支持

## 📝 学习检查清单

- [ ] 理解Spring Task的核心概念和工作原理
- [ ] 掌握@EnableScheduling和@Scheduled注解的使用
- [ ] 熟练使用fixedRate、fixedDelay、initialDelay配置
- [ ] 掌握Cron表达式的编写规则
- [ ] 理解Cron表达式中各字段的含义和特殊字符
- [ ] 掌握时区配置的使用
- [ ] 了解如何配置任务调度器线程池
- [ ] 掌握任务执行监控和日志记录
- [ ] 理解分布式环境下的任务调度问题
- [ ] 掌握使用分布式锁避免重复执行
- [ ] 了解Spring Task的适用场景和局限性
- [ ] 能够处理任务执行异常和实现重试机制
- [ ] 掌握动态控制任务启停的方法
- [ ] 了解Spring Task与Quartz的区别

## 📊 常用Cron表达式速查表

| 说明 | Cron表达式 |
|------|-----------|
| 每秒执行 | `* * * * * *` |
| 每分钟执行 | `0 * * * * *` |
| 每小时执行 | `0 0 * * * *` |
| 每天凌晨执行 | `0 0 0 * * *` |
| 每天上午10点执行 | `0 0 10 * * *` |
| 每天下午2点30分执行 | `0 30 14 * * *` |
| 每5分钟执行 | `0 */5 * * * *` |
| 每15分钟执行 | `0 */15 * * * *` |
| 每30分钟执行 | `0 */30 * * * *` |
| 每小时的第10分钟执行 | `0 10 * * * *` |
| 每天上午9点到下午5点每小时执行 | `0 0 9-17 * * *` |
| 每个工作日上午9点执行 | `0 0 9 * * MON-FRI` |
| 每周一上午10点执行 | `0 0 10 ? * MON` |
| 每月1号凌晨执行 | `0 0 0 1 * *` |
| 每月最后一天凌晨执行 | `0 0 0 L * *` |
| 每季度第一天凌晨执行 | `0 0 0 1 1,4,7,10 *` |
| 每年1月1号凌晨执行 | `0 0 0 1 1 *` |

## 🎓 进阶学习建议

1. **深入理解调度原理**: 研究Spring Task的源码，了解任务调度的底层实现
2. **学习Quartz**: 对于复杂场景，学习Quartz框架
3. **分布式调度**: 学习XXL-Job等分布式任务调度平台
4. **监控告警**: 集成Prometheus、Grafana等监控工具
5. **性能优化**: 学习如何优化大量定时任务的性能
6. **容错设计**: 学习如何设计高可用的定时任务系统

---

**文档版本**: v1.0  
**最后更新**: 2024-01-04  
**维护者**: @author erik.zhou
