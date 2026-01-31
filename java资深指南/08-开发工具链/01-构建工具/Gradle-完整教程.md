# Gradle 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [相关资源](#相关资源)

## 📚 技术概述
- **版本**: 8.5+
- **官方文档**: https://gradle.org/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、Groovy/Kotlin基础（可选）
- **文档来源**: Gradle官方文档 (Context7)
- **更新时间**: 2024-01-04

## 🎯 学习目标
- [ ] 理解Gradle的核心概念和工作原理
- [ ] 掌握Gradle构建脚本的编写（Groovy/Kotlin DSL）
- [ ] 熟练使用Gradle进行依赖管理
- [ ] 能够创建和配置自定义任务
- [ ] 掌握多项目构建
- [ ] 了解Gradle性能优化技巧

## 📖 基础概念

### 1.1 什么是Gradle

Gradle是一个开源的构建自动化工具，专注于灵活性和性能。Gradle构建脚本使用Groovy或Kotlin DSL编写，提供了声明式和命令式的混合编程模型。

**核心优势**：
- **高性能**: 增量构建、构建缓存、并行执行
- **灵活性**: 支持Groovy和Kotlin DSL，可编程性强
- **可扩展**: 丰富的插件生态系统
- **多语言支持**: Java、Kotlin、Groovy、Scala、C++等

### 1.2 核心概念

Gradle构建由**项目(Projects)**和**任务(Tasks)**组成，通过**构建脚本(Build Scripts)**配置，使用**Groovy**或**Kotlin** DSL编写。

#### 四大核心概念

1. **Root Project（根项目）**
   - 包含`settings.gradle(.kts)`文件的顶层项目
   - 通常聚合所有子项目

2. **Subprojects（子项目）**
   - 多项目构建中的独立模块
   - 通过`settings.gradle(.kts)`文件包含

3. **Settings File（设置文件）**
   - `settings.gradle(.kts)`配置文件
   - 定义多项目构建的结构

4. **Build Scripts（构建脚本）**
   - `build.gradle(.kts)`文件
   - 定义项目如何构建（插件、依赖、任务等）

#### 项目结构

```
my-project/
├── settings.gradle.kts          # 设置文件
├── build.gradle.kts             # 根项目构建脚本
├── gradle/
│   └── wrapper/                 # Gradle Wrapper
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew                      # Unix Wrapper脚本
├── gradlew.bat                  # Windows Wrapper脚本
└── subprojects/
    ├── app/
    │   └── build.gradle.kts
    └── lib/
        └── build.gradle.kts
```

### 1.3 应用场景
- Java/Kotlin/Groovy项目构建
- Android应用开发
- 多模块项目管理
- 微服务项目构建
- 持续集成和自动化部署

## 🔥 核心特性 (重点)

### 2.1 构建脚本 🔥

Gradle支持两种DSL：Groovy DSL和Kotlin DSL。

#### Groovy DSL示例

```groovy
// build.gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
}

group = 'com.example'
version = '1.0.0'
sourceCompatibility = '17'

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

#### Kotlin DSL示例（推荐）

```kotlin
// build.gradle.kts
plugins {
    java
    id("org.springframework.boot") version "3.2.0"
}

group = "com.example"
version = "1.0.0"

java {
    sourceCompatibility = JavaVersion.VERSION_17
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.named<Test>("test") {
    useJUnitPlatform()
}
```

**Kotlin DSL优势**：
- 类型安全
- IDE支持更好（自动补全、重构）
- 编译时检查
- 更好的代码导航

### 2.2 依赖管理 🔥

#### 依赖配置

```kotlin
dependencies {
    // 编译和运行时都需要
    implementation("org.springframework.boot:spring-boot-starter-web")
    
    // API依赖（会传递给消费者）
    api("com.google.guava:guava:32.1.3-jre")
    
    // 仅编译时需要
    compileOnly("org.projectlombok:lombok:1.18.30")
    
    // 注解处理器
    annotationProcessor("org.projectlombok:lombok:1.18.30")
    
    // 运行时需要
    runtimeOnly("com.mysql:mysql-connector-j")
    
    // 测试依赖
    testImplementation("org.junit.jupiter:junit-jupiter:5.10.0")
    testRuntimeOnly("org.junit.platform:junit-platform-launcher")
}
```

#### 依赖配置类型对比

| 配置 | 编译 | 运行 | 传递 | 说明 |
|------|------|------|------|------|
| implementation | ✓ | ✓ | ✗ | 实现依赖，不传递给消费者 |
| api | ✓ | ✓ | ✓ | API依赖，传递给消费者 |
| compileOnly | ✓ | ✗ | ✗ | 仅编译时需要 |
| runtimeOnly | ✗ | ✓ | ✗ | 仅运行时需要 |
| testImplementation | ✓ | ✓ | ✗ | 测试实现依赖 |

#### 版本管理

```kotlin
// 方式1：直接指定版本
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web:3.2.0")
}

// 方式2：使用版本目录（推荐）
// gradle/libs.versions.toml
[versions]
spring-boot = "3.2.0"
junit = "5.10.0"

[libraries]
spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web", version.ref = "spring-boot" }
junit-jupiter = { module = "org.junit.jupiter:junit-jupiter", version.ref = "junit" }

// build.gradle.kts
dependencies {
    implementation(libs.spring.boot.starter.web)
    testImplementation(libs.junit.jupiter)
}

// 方式3：使用平台BOM
dependencies {
    implementation(platform("org.springframework.boot:spring-boot-dependencies:3.2.0"))
    implementation("org.springframework.boot:spring-boot-starter-web")
}
```

#### 依赖约束

```kotlin
dependencies {
    constraints {
        implementation("org.apache.commons:commons-lang3:3.13.0")
    }
}
```

### 2.3 任务系统 🔥

#### 内置任务

```bash
# 查看所有任务
./gradlew tasks

# 编译主代码
./gradlew compileJava

# 编译测试代码
./gradlew compileTestJava

# 运行测试
./gradlew test

# 构建项目
./gradlew build

# 清理构建产物
./gradlew clean

# 查看依赖树
./gradlew dependencies
```

#### 自定义任务

```kotlin
// 简单任务
tasks.register("hello") {
    doLast {
        println("Hello, Gradle!")
    }
}

// 带参数的任务
abstract class GreetingTask : DefaultTask() {
    @Input
    var greeting: String = "Hello"
    
    @Input
    var name: String = "World"
    
    @TaskAction
    fun greet() {
        println("$greeting, $name!")
    }
}

tasks.register<GreetingTask>("greet") {
    greeting = "Hi"
    name = "Gradle"
}

// 任务依赖
tasks.register("taskA") {
    doLast {
        println("Task A")
    }
}

tasks.register("taskB") {
    dependsOn("taskA")
    doLast {
        println("Task B")
    }
}
```

#### 任务配置

```kotlin
tasks.named<Test>("test") {
    useJUnitPlatform()
    
    // 设置JVM参数
    jvmArgs("-Xmx1024m")
    
    // 设置系统属性
    systemProperty("spring.profiles.active", "test")
    
    // 测试日志
    testLogging {
        events("passed", "skipped", "failed")
        showStandardStreams = true
    }
}

tasks.named<JavaCompile>("compileJava") {
    options.encoding = "UTF-8"
    options.compilerArgs.add("-parameters")
}
```

### 2.4 多项目构建 (⚠️ 难点)

#### 设置文件配置

```kotlin
// settings.gradle.kts
rootProject.name = "my-project"

include("app")
include("lib")
include("common")

// 自定义子项目路径
project(":app").projectDir = file("modules/application")
```

#### 根项目配置

```kotlin
// build.gradle.kts (root)
plugins {
    java
    id("org.springframework.boot") version "3.2.0" apply false
}

allprojects {
    group = "com.example"
    version = "1.0.0"
    
    repositories {
        mavenCentral()
    }
}

subprojects {
    apply(plugin = "java")
    
    java {
        sourceCompatibility = JavaVersion.VERSION_17
    }
    
    dependencies {
        testImplementation("org.junit.jupiter:junit-jupiter:5.10.0")
    }
    
    tasks.named<Test>("test") {
        useJUnitPlatform()
    }
}
```

#### 子项目配置

```kotlin
// app/build.gradle.kts
plugins {
    id("org.springframework.boot")
    id("io.spring.dependency-management")
}

dependencies {
    // 依赖同级模块
    implementation(project(":lib"))
    implementation(project(":common"))
    
    implementation("org.springframework.boot:spring-boot-starter-web")
}

// lib/build.gradle.kts
plugins {
    `java-library`
}

dependencies {
    api("com.google.guava:guava:32.1.3-jre")
    implementation(project(":common"))
}
```

### 2.5 插件系统

#### 应用插件

```kotlin
plugins {
    // 核心插件
    java
    `java-library`
    application
    
    // 社区插件
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
    kotlin("jvm") version "1.9.21"
}
```

#### 常用插件配置

```kotlin
// Java插件
java {
    sourceCompatibility = JavaVersion.VERSION_17
    targetCompatibility = JavaVersion.VERSION_17
    
    // 生成源码jar
    withSourcesJar()
    // 生成javadoc jar
    withJavadocJar()
}

// Application插件
application {
    mainClass.set("com.example.Main")
}

// Spring Boot插件
springBoot {
    buildInfo()
}
```

## 💻 实战应用

### 3.1 环境搭建

#### 安装Gradle

**方式一：使用SDKMAN（推荐）**
```bash
# 安装SDKMAN
curl -s "https://get.sdkman.io" | bash

# 安装Gradle
sdk install gradle

# 验证安装
gradle -v
```

**方式二：手动安装**
```bash
# 下载Gradle
wget https://services.gradle.org/distributions/gradle-8.5-bin.zip

# 解压
unzip gradle-8.5-bin.zip

# 配置环境变量
export GRADLE_HOME=/path/to/gradle-8.5
export PATH=$GRADLE_HOME/bin:$PATH

# 验证安装
gradle -v
```

**方式三：使用Gradle Wrapper（推荐）**
```bash
# 项目中已有wrapper，直接使用
./gradlew -v

# 生成wrapper
gradle wrapper --gradle-version 8.5
```

### 3.2 快速开始

#### 创建新项目

```bash
# 使用init任务创建项目
gradle init

# 选择项目类型
# 1: basic
# 2: application
# 3: library
# 4: Gradle plugin

# 选择DSL
# 1: Groovy
# 2: Kotlin

# 选择测试框架
# 1: JUnit 4
# 2: TestNG
# 3: Spock
# 4: JUnit Jupiter
```

#### 基本命令

```bash
# 构建项目
./gradlew build

# 清理构建
./gradlew clean

# 运行应用
./gradlew run

# 运行测试
./gradlew test

# 查看任务列表
./gradlew tasks

# 查看项目依赖
./gradlew dependencies

# 查看项目属性
./gradlew properties

# 刷新依赖
./gradlew build --refresh-dependencies
```

### 3.3 进阶案例

#### Spring Boot项目配置

```kotlin
// build.gradle.kts
plugins {
    java
    id("org.springframework.boot") version "3.2.0"
    id("io.spring.dependency-management") version "1.1.4"
}

group = "com.example"
version = "1.0.0"

java {
    sourceCompatibility = JavaVersion.VERSION_17
}

configurations {
    compileOnly {
        extendsFrom(configurations.annotationProcessor.get())
    }
}

repositories {
    mavenCentral()
}

dependencies {
    // Spring Boot Starters
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    
    // Database
    runtimeOnly("com.mysql:mysql-connector-j")
    
    // Lombok
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")
    
    // Test
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.named<Test>("test") {
    useJUnitPlatform()
}

tasks.named<org.springframework.boot.gradle.tasks.bundling.BootJar>("bootJar") {
    archiveFileName.set("${project.name}.jar")
}
```

#### 多环境配置

```kotlin
// build.gradle.kts
val profile: String by project

tasks.named<ProcessResources>("processResources") {
    filesMatching("application.yml") {
        expand(project.properties)
    }
}

tasks.register<Copy>("copyConfig") {
    from("src/main/resources/application-${profile}.yml")
    into("build/resources/main")
    rename { "application.yml" }
}
```

```bash
# 使用指定环境构建
./gradlew build -Pprofile=prod
```

#### 自定义发布配置

```kotlin
// build.gradle.kts
plugins {
    `maven-publish`
}

publishing {
    publications {
        create<MavenPublication>("maven") {
            from(components["java"])
            
            groupId = "com.example"
            artifactId = "my-library"
            version = "1.0.0"
            
            pom {
                name.set("My Library")
                description.set("A concise description of my library")
                url.set("https://github.com/example/my-library")
                
                licenses {
                    license {
                        name.set("The Apache License, Version 2.0")
                        url.set("http://www.apache.org/licenses/LICENSE-2.0.txt")
                    }
                }
                
                developers {
                    developer {
                        id.set("erik.zhou")
                        name.set("Erik Zhou")
                        email.set("erik.zhou@example.com")
                    }
                }
            }
        }
    }
    
    repositories {
        maven {
            url = uri("https://nexus.example.com/repository/maven-releases/")
            credentials {
                username = project.findProperty("nexusUsername") as String?
                password = project.findProperty("nexusPassword") as String?
            }
        }
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

#### 启用构建缓存

```kotlin
// gradle.properties
org.gradle.caching=true
org.gradle.parallel=true
org.gradle.configureondemand=true
```

#### 使用Gradle Daemon

```bash
# Daemon默认启用，可在gradle.properties中配置
org.gradle.daemon=true
org.gradle.jvmargs=-Xmx2048m -XX:MaxMetaspaceSize=512m
```

#### 增量编译

```kotlin
tasks.withType<JavaCompile> {
    options.isIncremental = true
}
```

#### 并行执行

```bash
# 使用多个worker并行执行
./gradlew build --parallel --max-workers=4
```

### 4.2 依赖管理最佳实践

#### 1. 使用版本目录（Version Catalog）

```toml
# gradle/libs.versions.toml
[versions]
spring-boot = "3.2.0"
lombok = "1.18.30"

[libraries]
spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web", version.ref = "spring-boot" }
lombok = { module = "org.projectlombok:lombok", version.ref = "lombok" }

[plugins]
spring-boot = { id = "org.springframework.boot", version.ref = "spring-boot" }
```

#### 2. 锁定依赖版本

```kotlin
dependencyLocking {
    lockAllConfigurations()
}
```

```bash
# 生成锁定文件
./gradlew dependencies --write-locks

# 更新锁定文件
./gradlew dependencies --update-locks
```

#### 3. 分析依赖

```bash
# 查看依赖树
./gradlew dependencies

# 查看特定配置的依赖
./gradlew dependencies --configuration implementation

# 分析依赖冲突
./gradlew dependencyInsight --dependency spring-core
```

### 4.3 常见陷阱

#### ⚠️ 陷阱1：配置阶段执行耗时操作

**问题**：在配置阶段执行耗时操作会拖慢所有Gradle命令

```kotlin
// ❌ 错误：配置阶段执行
val result = exec {
    commandLine("some-slow-command")
}

// ✅ 正确：在任务执行阶段执行
tasks.register("myTask") {
    doLast {
        exec {
            commandLine("some-slow-command")
        }
    }
}
```

#### ⚠️ 陷阱2：使用动态版本

**问题**：动态版本（如`1.+`）会导致构建不可重现

```kotlin
// ❌ 避免使用动态版本
dependencies {
    implementation("com.example:lib:1.+")
    implementation("com.example:lib:latest.release")
}

// ✅ 使用具体版本
dependencies {
    implementation("com.example:lib:1.2.3")
}
```

#### ⚠️ 陷阱3：忽略Gradle Wrapper

**问题**：不同开发者使用不同Gradle版本导致构建不一致

```bash
# ✅ 始终使用wrapper
./gradlew build

# ❌ 避免直接使用gradle命令
gradle build
```

## ❓ 常见问题

### Q1: Gradle和Maven有什么区别？

**A**: 主要区别

| 特性 | Gradle | Maven |
|------|--------|-------|
| 配置语言 | Groovy/Kotlin DSL | XML |
| 性能 | 更快（增量构建、缓存） | 较慢 |
| 灵活性 | 高（可编程） | 低（约定优于配置） |
| 学习曲线 | 较陡 | 较平缓 |
| 生态系统 | 丰富 | 更丰富 |

### Q2: 如何加速Gradle构建？

**A**: 优化建议

```kotlin
// gradle.properties
org.gradle.caching=true
org.gradle.parallel=true
org.gradle.configureondemand=true
org.gradle.jvmargs=-Xmx2048m -XX:MaxMetaspaceSize=512m

# 使用本地缓存
org.gradle.caching.local=true
```

### Q3: 如何解决依赖冲突？

**A**: 使用依赖约束或强制版本

```kotlin
configurations.all {
    resolutionStrategy {
        // 强制使用特定版本
        force("com.google.guava:guava:32.1.3-jre")
        
        // 失败快速策略
        failOnVersionConflict()
    }
}

// 或使用依赖约束
dependencies {
    constraints {
        implementation("com.google.guava:guava:32.1.3-jre")
    }
}
```

### Q4: 如何跳过测试？

**A**: 使用命令行参数

```bash
# 跳过测试
./gradlew build -x test

# 或在build.gradle.kts中配置
tasks.named<Test>("test") {
    enabled = false
}
```

### Q5: 如何查看Gradle使用的JDK版本？

**A**: 使用以下命令

```bash
./gradlew -version
```

### Q6: 如何配置代理？

**A**: 在gradle.properties中配置

```properties
# gradle.properties
systemProp.http.proxyHost=proxy.example.com
systemProp.http.proxyPort=8080
systemProp.https.proxyHost=proxy.example.com
systemProp.https.proxyPort=8080
```

## 🔗 相关资源

### 官方文档
- [Gradle官方网站](https://gradle.org/)
- [Gradle用户手册](https://docs.gradle.org/current/userguide/userguide.html)
- [Gradle插件门户](https://plugins.gradle.org/)

### 推荐阅读
- [Gradle实战](https://www.manning.com/books/gradle-in-action)
- [Gradle最佳实践](https://docs.gradle.org/current/userguide/best_practices.html)
- [Gradle性能优化指南](https://docs.gradle.org/current/userguide/performance.html)

### 常用工具
- [Gradle Build Scan](https://scans.gradle.com/) - 构建分析工具
- [Gradle Profiler](https://github.com/gradle/gradle-profiler) - 性能分析工具
- [Gradle Enterprise](https://gradle.com/enterprise/) - 企业级构建加速

## 📝 学习检查清单
- [ ] 理解Gradle的核心概念（项目、任务、构建脚本）
- [ ] 掌握Kotlin DSL的基本语法
- [ ] 熟练配置依赖和管理依赖版本
- [ ] 能够创建和配置自定义任务
- [ ] 理解多项目构建的配置
- [ ] 掌握Gradle性能优化技巧
- [ ] 能够使用Gradle Wrapper
- [ ] 了解Gradle插件的使用和开发

---

**@author** erik.zhou
