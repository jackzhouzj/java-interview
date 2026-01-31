# Quartz 完整教程

## 📋 目录
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: Quartz 2.3.x
- **官方文档**: https://www.quartz-scheduler.org/documentation/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、Spring Framework基础、多线程基础、数据库基础
- **文档来源**: Context7 - Quartz官方文档
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Quartz的核心概念和架构设计
- [ ] 掌握Job、Trigger、Scheduler的使用
- [ ] 熟练使用CronTrigger和SimpleTrigger
- [ ] 掌握JobDataMap进行数据传递
- [ ] 了解Quartz的持久化和集群配置
- [ ] 掌握与Spring Boot的集成

## 📖 基础概念

### 1.1 什么是Quartz

Quartz是一个功能强大的开源任务调度框架，完全由Java编写。它提供了丰富的任务调度功能，支持任务持久化、集群部署、任务依赖等高级特性，是企业级应用中最常用的任务调度解决方案。

**核心特点**：
- **功能强大**: 支持复杂的任务调度需求
- **持久化**: 支持将任务配置持久化到数据库
- **集群支持**: 支持多节点集群部署，实现高可用
- **灵活性**: 支持多种触发器类型和任务执行策略
- **可扩展**: 提供丰富的监听器和插件机制

### 1.2 核心概念

- **Job**: 任务接口，定义要执行的具体业务逻辑
- **JobDetail**: 任务详情，包含Job的配置信息
- **Trigger**: 触发器，定义任务的执行时间规则
- **Scheduler**: 调度器，负责管理和调度Job和Trigger
- **JobDataMap**: 数据传递容器，用于向Job传递参数
- **CronTrigger**: 基于Cron表达式的触发器
- **SimpleTrigger**: 简单触发器，支持固定间隔和重复次数

### 1.3 Quartz架构

```
┌─────────────────────────────────────────┐
│           Scheduler（调度器）             │
│  ┌─────────────────────────────────┐   │
│  │    ThreadPool（线程池）          │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │    JobStore（任务存储）          │   │
│  │  - RAMJobStore（内存）           │   │
│  │  - JDBCJobStore（数据库）        │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
         ↓                    ↓
    ┌────────┐          ┌──────────┐
    │  Job   │          │ Trigger  │
    └────────┘          └──────────┘
```

### 1.4 应用场景

- **定时报表**: 定期生成各类业务报表
- **数据同步**: 定时从外部系统同步数据
- **批量处理**: 定时执行批量数据处理任务
- **系统维护**: 定期清理日志、备份数据
- **消息推送**: 定时发送邮件、短信通知
- **任务编排**: 复杂的任务依赖和流程编排
- **集群调度**: 分布式环境下的任务调度

## 🔥 核心特性 (重点)

### 2.1 Job - 任务定义 🔥

Job是Quartz中最核心的接口，定义了要执行的业务逻辑：

```java
package com.example.job;

import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * 简单任务示例
 * @author erik.zhou
 */
public class SimpleJob implements Job {

    private static final Logger logger = LoggerFactory.getLogger(SimpleJob.class);

    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        logger.info("SimpleJob 开始执行");
        
        // 获取任务的唯一标识
        String jobName = context.getJobDetail().getKey().getName();
        String jobGroup = context.getJobDetail().getKey().getGroup();
        
        logger.info("任务名称: {}, 任务组: {}", jobName, jobGroup);
        
        // 执行业务逻辑
        try {
            // 模拟业务处理
            Thread.sleep(2000);
            logger.info("SimpleJob 执行完成");
        } catch (InterruptedException e) {
            logger.error("任务执行被中断", e);
            Thread.currentThread().interrupt();
        }
    }
}
```

### 2.2 JobDetail - 任务详情 🔥

JobDetail包含Job的配置信息和元数据：

```java
package com.example.config;

import com.example.job.SimpleJob;
import org.quartz.JobBuilder;
import org.quartz.JobDetail;

/**
 * JobDetail创建示例
 * @author erik.zhou
 */
public class JobDetailExample {

    public static JobDetail createJobDetail() {
        return JobBuilder.newJob(SimpleJob.class)
                // 设置任务的唯一标识（名称和组）
                .withIdentity("myJob", "myGroup")
                // 设置任务描述
                .withDescription("这是一个简单的任务")
                // 设置任务数据
                .usingJobData("jobParam1", "value1")
                .usingJobData("jobParam2", 100)
                // 设置为持久化任务（即使没有触发器也保留）
                .storeDurably()
                // 构建JobDetail
                .build();
    }
}
```

### 2.3 Trigger - 触发器 🔥

Trigger定义任务的执行时间规则。Quartz提供多种触发器类型：

#### 2.3.1 SimpleTrigger - 简单触发器

```java
package com.example.config;

import org.quartz.SimpleScheduleBuilder;
import org.quartz.Trigger;
import org.quartz.TriggerBuilder;

import java.util.Date;

import static org.quartz.DateBuilder.*;

/**
 * SimpleTrigger示例
 * @author erik.zhou
 */
public class SimpleTriggerExample {

    /**
     * 立即执行一次
     */
    public static Trigger createOnceNowTrigger() {
        return TriggerBuilder.newTrigger()
                .withIdentity("trigger1", "group1")
                .startNow()
                .build();
    }

    /**
     * 10分钟后执行一次
     */
    public static Trigger createOnceFutureTrigger() {
        return TriggerBuilder.newTrigger()
                .withIdentity("trigger2", "group1")
                .startAt(futureDate(10, IntervalUnit.MINUTE))
                .build();
    }

    /**
     * 每5秒执行一次，重复10次
     */
    public static Trigger createRepeatTrigger() {
        return TriggerBuilder.newTrigger()
                .withIdentity("trigger3", "group1")
                .startNow()
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                        .withIntervalInSeconds(5)
                        .withRepeatCount(10))
                .build();
    }

    /**
     * 每小时执行一次，永久重复
     */
    public static Trigger createForeverTrigger() {
        return TriggerBuilder.newTrigger()
                .withIdentity("trigger4", "group1")
                .startAt(futureDate(10, IntervalUnit.MINUTE))
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                        .withIntervalInHours(1)
                        .repeatForever())
                .build();
    }
}
```

#### 2.3.2 CronTrigger - Cron触发器 (难点)

CronTrigger使用Cron表达式定义复杂的调度规则：

```java
package com.example.config;

import org.quartz.CronScheduleBuilder;
import org.quartz.CronTrigger;
import org.quartz.TriggerBuilder;

/**
 * CronTrigger示例
 * @author erik.zhou
 */
public class CronTriggerExample {

    /**
     * 每20秒执行一次
     */
    public static CronTrigger createEvery20SecondsTrigger() {
        return TriggerBuilder.newTrigger()
                .withIdentity("trigger1", "group1")
                .withSchedule(CronScheduleBuilder.cronSchedule("0/20 * * * * ?"))
                .build();
    }

    /**
     * 每天凌晨2点执行
     */
    public static CronTrigger createDailyAt2AMTrigger() {
        return TriggerBuilder.newTrigger()
                .withIdentity("trigger2", "group1")
                .withSchedule(CronScheduleBuilder.cronSchedule("0 0 2 * * ?"))
                .build();
    }

    /**
     * 每个工作日上午9点执行
     */
    public static CronTrigger createWeekdayAt9AMTrigger() {
        return TriggerBuilder.newTrigger()
                .withIdentity("trigger3", "group1")
                .withSchedule(CronScheduleBuilder.cronSchedule("0 0 9 ? * MON-FRI"))
                .build();
    }

    /**
     * 每月1号凌晨执行
     */
    public static CronTrigger createMonthlyTrigger() {
        return TriggerBuilder.newTrigger()
                .withIdentity("trigger4", "group1")
                .withSchedule(CronScheduleBuilder.cronSchedule("0 0 0 1 * ?"))
                .build();
    }

    /**
     * 每15分钟执行一次
     */
    public static CronTrigger createEvery15MinutesTrigger() {
        return TriggerBuilder.newTrigger()
                .withIdentity("trigger5", "group1")
                .withSchedule(CronScheduleBuilder.cronSchedule("0 */15 * * * ?"))
                .build();
    }
}
```

### 2.4 Scheduler - 调度器 🔥

Scheduler是Quartz的核心，负责管理和调度所有的Job和Trigger：

```java
package com.example.config;

import com.example.job.SimpleJob;
import org.quartz.*;
import org.quartz.impl.StdSchedulerFactory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Scheduler使用示例
 * @author erik.zhou
 */
public class SchedulerExample {

    private static final Logger logger = LoggerFactory.getLogger(SchedulerExample.class);

    public static void main(String[] args) throws SchedulerException {
        // 1. 创建Scheduler
        Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();

        // 2. 创建JobDetail
        JobDetail job = JobBuilder.newJob(SimpleJob.class)
                .withIdentity("myJob", "myGroup")
                .build();

        // 3. 创建Trigger
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("myTrigger", "myTriggerGroup")
                .startNow()
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                        .withIntervalInSeconds(10)
                        .repeatForever())
                .build();

        // 4. 调度任务
        scheduler.scheduleJob(job, trigger);

        // 5. 启动调度器
        scheduler.start();

        logger.info("Scheduler 已启动");

        // 6. 运行一段时间后关闭
        try {
            Thread.sleep(60000);  // 运行60秒
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        // 7. 关闭调度器
        scheduler.shutdown(true);  // true表示等待任务执行完成
        logger.info("Scheduler 已关闭");
    }
}
```

### 2.5 JobDataMap - 数据传递 (难点)

JobDataMap用于在Job、JobDetail和Trigger之间传递数据：

```java
package com.example.job;

import org.quartz.Job;
import org.quartz.JobDataMap;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.ArrayList;
import java.util.Date;

/**
 * 使用JobDataMap的任务
 * @author erik.zhou
 */
public class DataMapJob implements Job {

    private static final Logger logger = LoggerFactory.getLogger(DataMapJob.class);

    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        // 获取合并后的JobDataMap（包含JobDetail和Trigger的数据）
        JobDataMap dataMap = context.getMergedJobDataMap();

        // 读取数据
        String jobSays = dataMap.getString("jobSays");
        float myFloatValue = dataMap.getFloat("myFloatValue");
        ArrayList<Date> state = (ArrayList<Date>) dataMap.get("myStateData");

        // 修改数据（会保存到JobDataMap中）
        state.add(new Date());

        logger.info("任务说: {}, 浮点值: {}, 状态数据大小: {}", 
                    jobSays, myFloatValue, state.size());
    }
}
```

创建带数据的JobDetail和Trigger：

```java
// 创建JobDetail并添加数据
JobDetail job = JobBuilder.newJob(DataMapJob.class)
        .withIdentity("dataJob", "group1")
        .usingJobData("jobSays", "Hello from JobDetail")
        .usingJobData("myFloatValue", 3.14f)
        .usingJobData("myStateData", new ArrayList<Date>())
        .build();

// 创建Trigger并添加数据（会覆盖JobDetail中的同名数据）
Trigger trigger = TriggerBuilder.newTrigger()
        .withIdentity("dataTrigger", "group1")
        .startNow()
        .usingJobData("jobSays", "Hello from Trigger")  // 覆盖JobDetail中的值
        .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                .withIntervalInSeconds(10)
                .repeatForever())
        .build();
```

## 💻 实战应用

### 3.1 环境搭建

#### 3.1.1 Maven依赖

```xml
<dependencies>
    <!-- Quartz核心依赖 -->
    <dependency>
        <groupId>org.quartz-scheduler</groupId>
        <artifactId>quartz</artifactId>
        <version>2.3.2</version>
    </dependency>

    <!-- Spring Boot集成Quartz -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-quartz</artifactId>
    </dependency>

    <!-- 数据库持久化（可选） -->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>

    <!-- 日志 -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
    </dependency>
</dependencies>
```

### 3.2 快速开始

#### 3.2.1 纯Quartz应用

```java
package com.example;

import com.example.job.SimpleJob;
import org.quartz.*;
import org.quartz.impl.StdSchedulerFactory;

/**
 * Quartz快速开始示例
 * @author erik.zhou
 */
public class QuartzQuickStart {

    public static void main(String[] args) throws SchedulerException {
        // 创建调度器
        Scheduler scheduler = StdSchedulerFactory.getDefaultScheduler();

        // 创建任务
        JobDetail job = JobBuilder.newJob(SimpleJob.class)
                .withIdentity("job1", "group1")
                .build();

        // 创建触发器 - 每10秒执行一次
        Trigger trigger = TriggerBuilder.newTrigger()
                .withIdentity("trigger1", "group1")
                .startNow()
                .withSchedule(SimpleScheduleBuilder.simpleSchedule()
                        .withIntervalInSeconds(10)
                        .repeatForever())
                .build();

        // 调度任务
        scheduler.scheduleJob(job, trigger);

        // 启动调度器
        scheduler.start();

        System.out.println("Quartz调度器已启动");
    }
}
```

#### 3.2.2 Spring Boot集成

```java
package com.example.config;

import com.example.job.DataSyncJob;
import org.quartz.*;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Quartz配置类
 * @author erik.zhou
 */
@Configuration
public class QuartzConfig {

    /**
     * 创建JobDetail
     */
    @Bean
    public JobDetail dataSyncJobDetail() {
        return JobBuilder.newJob(DataSyncJob.class)
                .withIdentity("dataSyncJob", "syncGroup")
                .withDescription("数据同步任务")
                .storeDurably()
                .build();
    }

    /**
     * 创建Trigger
     */
    @Bean
    public Trigger dataSyncTrigger() {
        // 每天凌晨2点执行
        CronScheduleBuilder scheduleBuilder = CronScheduleBuilder.cronSchedule("0 0 2 * * ?");

        return TriggerBuilder.newTrigger()
                .forJob(dataSyncJobDetail())
                .withIdentity("dataSyncTrigger", "syncGroup")
                .withDescription("数据同步触发器")
                .withSchedule(scheduleBuilder)
                .build();
    }
}
```

```java
package com.example.job;

import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

/**
 * 数据同步任务
 * @author erik.zhou
 */
@Component
public class DataSyncJob implements Job {

    private static final Logger logger = LoggerFactory.getLogger(DataSyncJob.class);

    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        logger.info("开始执行数据同步任务");
        
        try {
            // 执行数据同步逻辑
            syncData();
            
            logger.info("数据同步任务执行完成");
        } catch (Exception e) {
            logger.error("数据同步任务执行失败", e);
            throw new JobExecutionException(e);
        }
    }

    private void syncData() {
        // 数据同步逻辑
    }
}
```

### 3.3 进阶案例

#### 3.3.1 数据库持久化配置

```properties
# application.properties

# Quartz配置
spring.quartz.job-store-type=jdbc
spring.quartz.jdbc.initialize-schema=always

# 数据源配置
spring.datasource.url=jdbc:mysql://localhost:3306/quartz_db?useUnicode=true&characterEncoding=utf8
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Quartz属性配置
spring.quartz.properties.org.quartz.scheduler.instanceName=MyScheduler
spring.quartz.properties.org.quartz.scheduler.instanceId=AUTO
spring.quartz.properties.org.quartz.threadPool.threadCount=10
spring.quartz.properties.org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX
spring.quartz.properties.org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
spring.quartz.properties.org.quartz.jobStore.tablePrefix=QRTZ_
spring.quartz.properties.org.quartz.jobStore.isClustered=true
spring.quartz.properties.org.quartz.jobStore.clusterCheckinInterval=20000
```

#### 3.3.2 动态管理任务

```java
package com.example.service;

import org.quartz.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Service;

/**
 * 任务管理服务
 * @author erik.zhou
 */
@Service
public class JobManagementService {

    private static final Logger logger = LoggerFactory.getLogger(JobManagementService.class);

    private final Scheduler scheduler;

    public JobManagementService(Scheduler scheduler) {
        this.scheduler = scheduler;
    }

    /**
     * 添加任务
     */
    public void addJob(Class<? extends Job> jobClass, String jobName, String jobGroup, 
                       String cronExpression) throws SchedulerException {
        // 创建JobDetail
        JobDetail jobDetail = JobBuilder.newJob(jobClass)
                .withIdentity(jobName, jobGroup)
                .storeDurably()
                .build();

        // 创建Trigger
        CronTrigger trigger = TriggerBuilder.newTrigger()
                .withIdentity(jobName + "Trigger", jobGroup)
                .withSchedule(CronScheduleBuilder.cronSchedule(cronExpression))
                .build();

        // 调度任务
        scheduler.scheduleJob(jobDetail, trigger);
        logger.info("任务添加成功: {}.{}", jobGroup, jobName);
    }

    /**
     * 暂停任务
     */
    public void pauseJob(String jobName, String jobGroup) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName, jobGroup);
        scheduler.pauseJob(jobKey);
        logger.info("任务已暂停: {}.{}", jobGroup, jobName);
    }

    /**
     * 恢复任务
     */
    public void resumeJob(String jobName, String jobGroup) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName, jobGroup);
        scheduler.resumeJob(jobKey);
        logger.info("任务已恢复: {}.{}", jobGroup, jobName);
    }

    /**
     * 删除任务
     */
    public void deleteJob(String jobName, String jobGroup) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName, jobGroup);
        scheduler.deleteJob(jobKey);
        logger.info("任务已删除: {}.{}", jobGroup, jobName);
    }

    /**
     * 修改任务的Cron表达式
     */
    public void updateJobCron(String jobName, String jobGroup, String newCronExpression) 
            throws SchedulerException {
        TriggerKey triggerKey = TriggerKey.triggerKey(jobName + "Trigger", jobGroup);
        CronTrigger oldTrigger = (CronTrigger) scheduler.getTrigger(triggerKey);

        if (oldTrigger != null) {
            // 创建新的Trigger
            CronTrigger newTrigger = oldTrigger.getTriggerBuilder()
                    .withSchedule(CronScheduleBuilder.cronSchedule(newCronExpression))
                    .build();

            // 重新调度
            scheduler.rescheduleJob(triggerKey, newTrigger);
            logger.info("任务Cron表达式已更新: {}.{}, 新表达式: {}", 
                       jobGroup, jobName, newCronExpression);
        }
    }

    /**
     * 立即执行任务
     */
    public void triggerJob(String jobName, String jobGroup) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName, jobGroup);
        scheduler.triggerJob(jobKey);
        logger.info("任务已触发执行: {}.{}", jobGroup, jobName);
    }

    /**
     * 检查任务是否存在
     */
    public boolean checkJobExists(String jobName, String jobGroup) throws SchedulerException {
        JobKey jobKey = JobKey.jobKey(jobName, jobGroup);
        return scheduler.checkExists(jobKey);
    }
}
```

#### 3.3.3 任务监听器

```java
package com.example.listener;

import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.quartz.JobListener;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

/**
 * 任务执行监听器
 * @author erik.zhou
 */
@Component
public class CustomJobListener implements JobListener {

    private static final Logger logger = LoggerFactory.getLogger(CustomJobListener.class);

    @Override
    public String getName() {
        return "CustomJobListener";
    }

    /**
     * 任务执行前调用
     */
    @Override
    public void jobToBeExecuted(JobExecutionContext context) {
        String jobName = context.getJobDetail().getKey().getName();
        logger.info("任务即将执行: {}", jobName);
    }

    /**
     * 任务被否决时调用（例如被TriggerListener否决）
     */
    @Override
    public void jobExecutionVetoed(JobExecutionContext context) {
        String jobName = context.getJobDetail().getKey().getName();
        logger.warn("任务执行被否决: {}", jobName);
    }

    /**
     * 任务执行后调用
     */
    @Override
    public void jobWasExecuted(JobExecutionContext context, JobExecutionException jobException) {
        String jobName = context.getJobDetail().getKey().getName();
        long runTime = context.getJobRunTime();

        if (jobException != null) {
            logger.error("任务执行失败: {}, 耗时: {}ms", jobName, runTime, jobException);
        } else {
            logger.info("任务执行成功: {}, 耗时: {}ms", jobName, runTime);
        }
    }
}
```

注册监听器：

```java
package com.example.config;

import com.example.listener.CustomJobListener;
import org.quartz.Scheduler;
import org.springframework.context.annotation.Configuration;

import javax.annotation.PostConstruct;

/**
 * 监听器配置
 * @author erik.zhou
 */
@Configuration
public class ListenerConfig {

    private final Scheduler scheduler;
    private final CustomJobListener customJobListener;

    public ListenerConfig(Scheduler scheduler, CustomJobListener customJobListener) {
        this.scheduler = scheduler;
        this.customJobListener = customJobListener;
    }

    @PostConstruct
    public void init() throws Exception {
        // 注册全局监听器
        scheduler.getListenerManager().addJobListener(customJobListener);
    }
}
```

#### 3.3.4 集群配置示例

```java
package com.example.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.quartz.SchedulerFactoryBean;

import javax.sql.DataSource;
import java.util.Properties;

/**
 * Quartz集群配置
 * @author erik.zhou
 */
@Configuration
public class QuartzClusterConfig {

    @Bean
    public SchedulerFactoryBean schedulerFactoryBean(DataSource dataSource) {
        SchedulerFactoryBean factory = new SchedulerFactoryBean();
        factory.setDataSource(dataSource);
        factory.setOverwriteExistingJobs(true);
        factory.setStartupDelay(20);  // 延迟20秒启动

        Properties properties = new Properties();
        
        // 调度器实例配置
        properties.setProperty("org.quartz.scheduler.instanceName", "ClusterScheduler");
        properties.setProperty("org.quartz.scheduler.instanceId", "AUTO");
        
        // 线程池配置
        properties.setProperty("org.quartz.threadPool.class", "org.quartz.simpl.SimpleThreadPool");
        properties.setProperty("org.quartz.threadPool.threadCount", "20");
        properties.setProperty("org.quartz.threadPool.threadPriority", "5");
        
        // JobStore配置
        properties.setProperty("org.quartz.jobStore.class", "org.quartz.impl.jdbcjobstore.JobStoreTX");
        properties.setProperty("org.quartz.jobStore.driverDelegateClass", 
                              "org.quartz.impl.jdbcjobstore.StdJDBCDelegate");
        properties.setProperty("org.quartz.jobStore.tablePrefix", "QRTZ_");
        properties.setProperty("org.quartz.jobStore.useProperties", "false");
        
        // 集群配置
        properties.setProperty("org.quartz.jobStore.isClustered", "true");
        properties.setProperty("org.quartz.jobStore.clusterCheckinInterval", "20000");
        
        factory.setQuartzProperties(properties);
        
        return factory;
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

#### 4.1.1 合理配置线程池

```properties
# 根据任务数量和执行时间配置线程池大小
org.quartz.threadPool.threadCount=20

# 线程优先级（1-10）
org.quartz.threadPool.threadPriority=5

# 线程池类型
org.quartz.threadPool.class=org.quartz.simpl.SimpleThreadPool
```

#### 4.1.2 使用持久化JobStore

```properties
# 使用数据库持久化，避免任务丢失
org.quartz.jobStore.class=org.quartz.impl.jdbcjobstore.JobStoreTX

# 不要使用RAMJobStore在生产环境
# org.quartz.jobStore.class=org.quartz.simpl.RAMJobStore  # ❌ 仅用于开发测试
```

#### 4.1.3 避免任务执行时间过长

```java
/**
 * 错误示例：任务执行时间过长
 */
public class BadJob implements Job {
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        // 执行耗时操作，可能阻塞线程池
        processLargeDataSet();  // ❌ 不推荐
    }
}

/**
 * 正确示例：使用异步处理
 */
public class GoodJob implements Job {
    
    @Autowired
    private AsyncService asyncService;
    
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        // 提交到异步线程池处理
        asyncService.processAsync();  // ✅ 推荐
    }
}
```

### 4.2 常见陷阱

#### 4.2.1 ⚠️ Job类必须有无参构造函数

```java
// ❌ 错误：没有无参构造函数
public class BadJob implements Job {
    private final String param;
    
    public BadJob(String param) {
        this.param = param;
    }
    
    @Override
    public void execute(JobExecutionContext context) { }
}

// ✅ 正确：提供无参构造函数，使用JobDataMap传递参数
public class GoodJob implements Job {
    
    public GoodJob() {
        // 无参构造函数
    }
    
    @Override
    public void execute(JobExecutionContext context) {
        JobDataMap dataMap = context.getJobDetail().getJobDataMap();
        String param = dataMap.getString("param");
    }
}
```

#### 4.2.2 ⚠️ 集群环境下的时钟同步

在集群环境中，各节点的系统时钟必须同步，否则会导致任务调度混乱：

```bash
# 使用NTP同步时钟
sudo ntpdate -u ntp.aliyun.com

# 或配置自动同步
sudo systemctl enable ntpd
sudo systemctl start ntpd
```

#### 4.2.3 ⚠️ 任务并发执行问题

```java
package com.example.job;

import org.quartz.DisallowConcurrentExecution;
import org.quartz.Job;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;

/**
 * 禁止并发执行的任务
 * @author erik.zhou
 */
@DisallowConcurrentExecution  // 防止同一任务并发执行
public class NonConcurrentJob implements Job {
    
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        // 任务逻辑
        // 即使上次执行未完成，也不会启动新的执行
    }
}
```

#### 4.2.4 ⚠️ 异常处理

```java
/**
 * 正确的异常处理
 * @author erik.zhou
 */
public class SafeJob implements Job {
    
    private static final Logger logger = LoggerFactory.getLogger(SafeJob.class);
    
    @Override
    public void execute(JobExecutionContext context) throws JobExecutionException {
        try {
            // 执行任务逻辑
            executeTask();
        } catch (Exception e) {
            logger.error("任务执行失败", e);
            
            // 创建JobExecutionException并设置是否重新调度
            JobExecutionException jee = new JobExecutionException(e);
            
            // 设置为true表示立即重新执行
            jee.setRefireImmediately(false);
            
            // 设置为true表示取消所有触发器
            jee.setUnscheduleAllTriggers(false);
            
            throw jee;
        }
    }
    
    private void executeTask() {
        // 任务逻辑
    }
}
```

### 4.3 监控与运维

#### 4.3.1 任务执行监控

```java
package com.example.monitor;

import org.quartz.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

import java.util.Date;

/**
 * 任务监控监听器
 * @author erik.zhou
 */
@Component
public class JobMonitorListener implements JobListener {

    private static final Logger logger = LoggerFactory.getLogger(JobMonitorListener.class);

    @Override
    public String getName() {
        return "JobMonitorListener";
    }

    @Override
    public void jobToBeExecuted(JobExecutionContext context) {
        String jobName = context.getJobDetail().getKey().getName();
        Date fireTime = context.getFireTime();
        logger.info("[监控] 任务开始执行 - 任务名: {}, 触发时间: {}", jobName, fireTime);
    }

    @Override
    public void jobExecutionVetoed(JobExecutionContext context) {
        String jobName = context.getJobDetail().getKey().getName();
        logger.warn("[监控] 任务被否决 - 任务名: {}", jobName);
        // 发送告警
        sendAlert("任务被否决", jobName);
    }

    @Override
    public void jobWasExecuted(JobExecutionContext context, JobExecutionException jobException) {
        String jobName = context.getJobDetail().getKey().getName();
        long runTime = context.getJobRunTime();
        Date nextFireTime = context.getNextFireTime();

        if (jobException != null) {
            logger.error("[监控] 任务执行失败 - 任务名: {}, 耗时: {}ms", jobName, runTime, jobException);
            // 发送告警
            sendAlert("任务执行失败", jobName + ", 错误: " + jobException.getMessage());
        } else {
            logger.info("[监控] 任务执行成功 - 任务名: {}, 耗时: {}ms, 下次执行: {}", 
                       jobName, runTime, nextFireTime);
            
            // 如果执行时间超过阈值，发送告警
            if (runTime > 60000) {  // 超过60秒
                sendAlert("任务执行时间过长", jobName + ", 耗时: " + runTime + "ms");
            }
        }

        // 上报监控指标
        reportMetrics(jobName, runTime, jobException == null);
    }

    private void sendAlert(String title, String message) {
        // 发送告警（邮件、短信、钉钉等）
        logger.warn("告警: {} - {}", title, message);
    }

    private void reportMetrics(String jobName, long runTime, boolean success) {
        // 上报到监控系统（如Prometheus）
    }
}
```

#### 4.3.2 查询任务状态

```java
package com.example.service;

import org.quartz.*;
import org.quartz.impl.matchers.GroupMatcher;
import org.springframework.stereotype.Service;

import java.util.*;

/**
 * 任务查询服务
 * @author erik.zhou
 */
@Service
public class JobQueryService {

    private final Scheduler scheduler;

    public JobQueryService(Scheduler scheduler) {
        this.scheduler = scheduler;
    }

    /**
     * 获取所有任务
     */
    public List<Map<String, Object>> getAllJobs() throws SchedulerException {
        List<Map<String, Object>> jobList = new ArrayList<>();

        for (String groupName : scheduler.getJobGroupNames()) {
            for (JobKey jobKey : scheduler.getJobKeys(GroupMatcher.jobGroupEquals(groupName))) {
                JobDetail jobDetail = scheduler.getJobDetail(jobKey);
                List<? extends Trigger> triggers = scheduler.getTriggersOfJob(jobKey);

                Map<String, Object> jobInfo = new HashMap<>();
                jobInfo.put("jobName", jobKey.getName());
                jobInfo.put("jobGroup", jobKey.getGroup());
                jobInfo.put("description", jobDetail.getDescription());
                jobInfo.put("jobClass", jobDetail.getJobClass().getName());

                if (!triggers.isEmpty()) {
                    Trigger trigger = triggers.get(0);
                    jobInfo.put("triggerState", scheduler.getTriggerState(trigger.getKey()).name());
                    jobInfo.put("previousFireTime", trigger.getPreviousFireTime());
                    jobInfo.put("nextFireTime", trigger.getNextFireTime());
                }

                jobList.add(jobInfo);
            }
        }

        return jobList;
    }

    /**
     * 获取正在执行的任务
     */
    public List<String> getRunningJobs() throws SchedulerException {
        List<String> runningJobs = new ArrayList<>();
        List<JobExecutionContext> executingJobs = scheduler.getCurrentlyExecutingJobs();

        for (JobExecutionContext context : executingJobs) {
            String jobName = context.getJobDetail().getKey().getName();
            runningJobs.add(jobName);
        }

        return runningJobs;
    }
}
```

## ❓ 常见问题

### Q1: Quartz和Spring Task有什么区别？

A: 
| 特性 | Spring Task | Quartz |
|------|------------|--------|
| 复杂度 | 简单 | 复杂 |
| 持久化 | 不支持 | 支持 |
| 集群 | 需要额外实现 | 原生支持 |
| 动态管理 | 困难 | 容易 |
| 适用场景 | 简单定时任务 | 企业级任务调度 |

### Q2: 如何实现任务失败重试？

A: 使用JobExecutionException：

```java
@Override
public void execute(JobExecutionContext context) throws JobExecutionException {
    try {
        executeTask();
    } catch (Exception e) {
        JobExecutionException jee = new JobExecutionException(e);
        jee.setRefireImmediately(true);  // 立即重新执行
        throw jee;
    }
}
```

### Q3: 如何防止任务重复执行？

A: 使用@DisallowConcurrentExecution注解：

```java
@DisallowConcurrentExecution
public class MyJob implements Job {
    @Override
    public void execute(JobExecutionContext context) {
        // 任务逻辑
    }
}
```

### Q4: 集群环境下任务会重复执行吗？

A: 不会。Quartz使用数据库锁机制确保同一时刻只有一个节点执行任务。

### Q5: 如何优雅停止Quartz？

A: 使用shutdown方法：

```java
// true表示等待任务执行完成
scheduler.shutdown(true);

// false表示立即停止
scheduler.shutdown(false);
```

### Q6: Quartz支持任务依赖吗？

A: Quartz本身不直接支持任务依赖，需要通过JobListener或自定义逻辑实现。

### Q7: 如何处理Misfire（错过触发）？

A: 配置Misfire策略：

```java
CronTrigger trigger = TriggerBuilder.newTrigger()
    .withSchedule(CronScheduleBuilder.cronSchedule("0 0 2 * * ?")
        .withMisfireHandlingInstructionDoNothing())  // 不执行错过的任务
    .build();
```

Misfire策略：
- `withMisfireHandlingInstructionDoNothing()`: 不执行错过的任务
- `withMisfireHandlingInstructionFireAndProceed()`: 立即执行一次，然后继续
- `withMisfireHandlingInstructionIgnoreMisfires()`: 忽略所有misfire，按原计划执行

## 🔗 相关资源

### 官方文档
- [Quartz官方网站](https://www.quartz-scheduler.org/)
- [Quartz文档](https://www.quartz-scheduler.org/documentation/)
- [Quartz GitHub](https://github.com/quartz-scheduler/quartz)
- [Spring Boot Quartz集成](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.quartz)

### 推荐文章
- [Quartz完全指南](https://www.baeldung.com/quartz)
- [Spring Boot集成Quartz](https://www.baeldung.com/spring-quartz-schedule)
- [Quartz集群配置详解](https://www.baeldung.com/quartz-cluster-hazelcast)

### 数据库脚本
- [Quartz数据库表结构](https://github.com/quartz-scheduler/quartz/tree/main/quartz-core/src/main/resources/org/quartz/impl/jdbcjobstore)

### 相关技术
- **Spring Task**: 轻量级任务调度
- **XXL-Job**: 分布式任务调度平台
- **Elastic-Job**: 分布式任务调度框架

## 📝 学习检查清单

- [ ] 理解Quartz的核心概念和架构
- [ ] 掌握Job接口的实现
- [ ] 掌握JobDetail的创建和配置
- [ ] 熟练使用SimpleTrigger和CronTrigger
- [ ] 理解Scheduler的生命周期管理
- [ ] 掌握JobDataMap的使用
- [ ] 了解任务持久化配置
- [ ] 掌握集群配置和部署
- [ ] 了解JobListener和TriggerListener
- [ ] 掌握动态管理任务的方法
- [ ] 理解Misfire机制和处理策略
- [ ] 掌握任务监控和告警
- [ ] 了解@DisallowConcurrentExecution注解
- [ ] 掌握与Spring Boot的集成

## 📊 Quartz Cron表达式速查表

| 说明 | Cron表达式 |
|------|-----------|
| 每秒执行 | `* * * * * ?` |
| 每分钟执行 | `0 * * * * ?` |
| 每小时执行 | `0 0 * * * ?` |
| 每天凌晨执行 | `0 0 0 * * ?` |
| 每天上午10点执行 | `0 0 10 * * ?` |
| 每20秒执行 | `0/20 * * * * ?` |
| 每5分钟执行 | `0 */5 * * * ?` |
| 每15分钟执行 | `0 */15 * * * ?` |
| 每30分钟执行 | `0 */30 * * * ?` |
| 每个工作日上午9点执行 | `0 0 9 ? * MON-FRI` |
| 每周一上午10点执行 | `0 0 10 ? * MON` |
| 每月1号凌晨执行 | `0 0 0 1 * ?` |
| 每月最后一天凌晨执行 | `0 0 0 L * ?` |
| 每季度第一天凌晨执行 | `0 0 0 1 1,4,7,10 ?` |
| 每年1月1号凌晨执行 | `0 0 0 1 1 ?` |

**注意**: Quartz的Cron表达式有7个字段（包含年份），与Linux Cron（6个字段）不同。

## 🎓 进阶学习建议

1. **深入源码**: 研究Quartz的调度原理和线程模型
2. **集群实战**: 搭建Quartz集群环境，理解分布式调度
3. **性能优化**: 学习如何优化大规模任务调度的性能
4. **监控集成**: 集成Prometheus、Grafana等监控工具
5. **高可用设计**: 学习如何设计高可用的任务调度系统
6. **任务编排**: 研究复杂任务依赖和流程编排的实现
7. **对比学习**: 对比XXL-Job、Elastic-Job等其他调度框架

## 🔧 Quartz数据库表说明

Quartz持久化需要11张表：

| 表名 | 说明 |
|------|------|
| QRTZ_JOB_DETAILS | 存储JobDetail信息 |
| QRTZ_TRIGGERS | 存储Trigger信息 |
| QRTZ_SIMPLE_TRIGGERS | 存储SimpleTrigger信息 |
| QRTZ_CRON_TRIGGERS | 存储CronTrigger信息 |
| QRTZ_BLOB_TRIGGERS | 存储Blob类型的Trigger |
| QRTZ_CALENDARS | 存储Calendar信息 |
| QRTZ_PAUSED_TRIGGER_GRPS | 存储暂停的Trigger组 |
| QRTZ_FIRED_TRIGGERS | 存储正在执行的Trigger |
| QRTZ_SCHEDULER_STATE | 存储Scheduler状态 |
| QRTZ_LOCKS | 存储锁信息（集群使用） |
| QRTZ_SIMPROP_TRIGGERS | 存储简单属性Trigger |

---

**文档版本**: v1.0  
**最后更新**: 2024-01-04  
**维护者**: @author erik.zhou
