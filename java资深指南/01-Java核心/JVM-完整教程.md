# JVM 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [JVM架构](#jvm架构)
- [内存模型](#内存模型)
- [垃圾回收](#垃圾回收)
- [类加载机制](#类加载机制)
- [性能调优](#性能调优)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)

## 📚 技术概述
- **版本**: JDK 21
- **官方文档**: https://docs.oracle.com/en/java/javase/21/vm/
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、操作系统基础、计算机组成原理
- **文档来源**: OpenJDK官方文档 + Context7查询
- **更新时间**: 2024-12-31

## 🎯 学习目标
- [ ] 理解JVM的整体架构和运行机制
- [ ] 深入掌握JVM内存模型和各区域的作用
- [ ] 理解垃圾回收算法和各种GC收集器
- [ ] 掌握类加载机制和双亲委派模型
- [ ] 掌握JVM性能调优方法和工具
- [ ] 能够分析和解决内存溢出、内存泄漏等问题

## 📖 JVM架构

### 1.1 什么是JVM

JVM（Java Virtual Machine，Java虚拟机）是Java程序的运行环境，它负责将Java字节码转换为机器码并执行。JVM是Java"一次编写，到处运行"特性的核心。

**JVM的主要功能**：
- 加载和执行字节码
- 内存管理和垃圾回收
- 提供运行时环境
- 确保程序安全性


### 1.2 JVM整体架构

```
┌─────────────────────────────────────────────────────────┐
│                    Java应用程序                          │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                   类加载子系统                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │ 加载     │→ │ 链接     │→ │ 初始化   │             │
│  └──────────┘  └──────────┘  └──────────┘             │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                   运行时数据区                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │ 方法区   │  │ 堆       │  │ 栈       │             │
│  └──────────┘  └──────────┘  └──────────┘             │
│  ┌──────────┐  ┌──────────┐                            │
│  │ 程序计数器│  │ 本地方法栈│                            │
│  └──────────┘  └──────────┘                            │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                   执行引擎                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │ 解释器   │  │ JIT编译器│  │ 垃圾回收器│             │
│  └──────────┘  └──────────┘  └──────────┘             │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                   本地方法接口                           │
└─────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────┐
│                   本地方法库                             │
└─────────────────────────────────────────────────────────┘
```

**三大核心组件**：
1. **类加载子系统**: 负责加载.class文件
2. **运行时数据区**: 存储程序运行时的数据
3. **执行引擎**: 执行字节码指令

## 🔥 内存模型 (重点)

### 2.1 运行时数据区概述

JVM运行时数据区分为以下几个部分：

| 区域 | 线程私有/共享 | 是否会OOM | 作用 |
|------|--------------|-----------|------|
| 程序计数器 | 私有 | 否 | 记录当前线程执行的字节码行号 |
| 虚拟机栈 | 私有 | 是 | 存储局部变量、操作数栈等 |
| 本地方法栈 | 私有 | 是 | 为Native方法服务 |
| 堆 | 共享 | 是 | 存储对象实例 |
| 方法区 | 共享 | 是 | 存储类信息、常量、静态变量 |


### 2.2 程序计数器（Program Counter Register）

**特点**：
- 线程私有
- 占用内存很小
- 唯一不会发生OutOfMemoryError的区域

**作用**：
- 记录当前线程执行的字节码行号
- 线程切换后能恢复到正确的执行位置

### 2.3 虚拟机栈（VM Stack）🔥

**栈帧结构**：
```
┌─────────────────────────┐
│      局部变量表          │ ← 存储方法参数和局部变量
├─────────────────────────┤
│      操作数栈            │ ← 执行字节码指令的工作区
├─────────────────────────┤
│      动态链接            │ ← 指向运行时常量池的方法引用
├─────────────────────────┤
│      方法返回地址        │ ← 方法正常退出或异常退出的地址
└─────────────────────────┘
```

**示例代码**：
```java
/**
 * 虚拟机栈示例
 * @author erik.zhou
 */
public class StackDemo {
    public static void main(String[] args) {
        int result = add(10, 20);
        System.out.println(result);
    }
    
    public static int add(int a, int b) {
        int sum = a + b;
        return sum;
    }
}
```

**栈帧变化**：
```
1. main方法入栈
   ├─ 局部变量表: args, result
   └─ 操作数栈: ...

2. add方法入栈
   ├─ 局部变量表: a=10, b=20, sum=30
   └─ 操作数栈: ...

3. add方法出栈，返回值30

4. main方法继续执行
```

**可能的异常**：
- `StackOverflowError`: 栈深度超过限制（如递归调用过深）
- `OutOfMemoryError`: 无法分配新的栈帧


### 2.4 堆（Heap）🔥 ⚠️ 难点

堆是JVM内存中最大的一块区域，所有对象实例和数组都在堆上分配。

**堆的结构（分代模型）**：
```
┌─────────────────────────────────────────────────────────┐
│                        Java堆                            │
├─────────────────────────────────┬───────────────────────┤
│          新生代（Young Gen）     │   老年代（Old Gen）    │
│                                 │                       │
│  ┌──────┬──────┬──────┐        │                       │
│  │ Eden │ S0   │ S1   │        │                       │
│  │      │      │      │        │                       │
│  │ 8    │ 1    │ 1    │        │                       │
│  └──────┴──────┴──────┘        │                       │
│                                 │                       │
│  新对象分配在Eden区              │  长期存活的对象        │
│  经过Minor GC后存活的对象        │                       │
│  移动到Survivor区                │                       │
└─────────────────────────────────┴───────────────────────┘
```

**内存分配策略**：
1. **对象优先在Eden区分配**
2. **大对象直接进入老年代**（避免在Eden和Survivor之间复制）
3. **长期存活的对象进入老年代**（默认15次GC后）
4. **动态对象年龄判定**

**示例代码**：
```java
/**
 * 堆内存分配示例
 * @author erik.zhou
 */
public class HeapDemo {
    public static void main(String[] args) {
        // 小对象分配在Eden区
        byte[] small = new byte[1024]; // 1KB
        
        // 大对象直接分配在老年代
        byte[] large = new byte[10 * 1024 * 1024]; // 10MB
        
        // 触发Minor GC
        for (int i = 0; i < 1000; i++) {
            byte[] temp = new byte[1024 * 1024]; // 1MB
        }
    }
}
```

**JVM参数配置**：
```bash
# 设置堆的初始大小和最大大小
-Xms512m -Xmx2g

# 设置新生代大小
-Xmn256m

# 设置Eden和Survivor的比例（默认8:1:1）
-XX:SurvivorRatio=8

# 设置新生代和老年代的比例（默认1:2）
-XX:NewRatio=2
```


### 2.5 方法区（Method Area）

方法区用于存储已被虚拟机加载的类信息、常量、静态变量、即时编译器编译后的代码等。

**JDK版本变化**：
- **JDK 7及之前**: 方法区称为"永久代"（PermGen）
- **JDK 8及之后**: 移除永久代，改为"元空间"（Metaspace），使用本地内存

**存储内容**：
- 类的元数据（类名、访问修饰符、字段、方法等）
- 运行时常量池
- 静态变量
- 即时编译器编译后的代码

**JVM参数**：
```bash
# JDK 7及之前：设置永久代大小
-XX:PermSize=64m
-XX:MaxPermSize=256m

# JDK 8及之后：设置元空间大小
-XX:MetaspaceSize=64m
-XX:MaxMetaspaceSize=256m
```

**运行时常量池**：
```java
/**
 * 运行时常量池示例
 * @author erik.zhou
 */
public class ConstantPoolDemo {
    public static void main(String[] args) {
        // 字符串常量池
        String s1 = "hello";
        String s2 = "hello";
        System.out.println(s1 == s2); // true，指向同一个常量池对象
        
        String s3 = new String("hello");
        System.out.println(s1 == s3); // false，s3在堆中
        
        String s4 = s3.intern(); // intern()将字符串放入常量池
        System.out.println(s1 == s4); // true
    }
}
```

## 🔥 垃圾回收（Garbage Collection）⚠️ 难点

### 3.1 如何判断对象可以回收

**引用计数法**（Java未采用）：
- 给对象添加引用计数器
- 引用+1，引用失效-1
- 计数器为0时可回收
- 问题：无法解决循环引用

**可达性分析算法**（Java采用）：
```
GC Roots
   ├─→ 对象A
   │     ├─→ 对象B
   │     └─→ 对象C
   └─→ 对象D
   
对象E ← 对象F （不可达，可回收）
```

**GC Roots包括**：
- 虚拟机栈中引用的对象
- 方法区中静态属性引用的对象
- 方法区中常量引用的对象
- 本地方法栈中引用的对象
- 活跃线程


### 3.2 垃圾回收算法 🔥 ⚠️ 难点

**1. 标记-清除算法（Mark-Sweep）**：
```
标记阶段：
[对象A][对象B][对象C][对象D][对象E]
  ✓      ✗      ✓      ✗      ✓

清除阶段：
[对象A][空闲][对象C][空闲][对象E]
```
- 优点：简单
- 缺点：产生内存碎片，效率不高

**2. 标记-复制算法（Mark-Copy）**：
```
From区：
[对象A][对象B][对象C][对象D][对象E]
  ✓      ✗      ✓      ✗      ✓

To区（复制后）：
[对象A][对象C][对象E][空闲空间...]
```
- 优点：无内存碎片，效率高
- 缺点：浪费一半内存
- 应用：新生代GC（Eden和Survivor）

**3. 标记-整理算法（Mark-Compact）**：
```
标记阶段：
[对象A][对象B][对象C][对象D][对象E]
  ✓      ✗      ✓      ✗      ✓

整理阶段：
[对象A][对象C][对象E][空闲空间...]
```
- 优点：无内存碎片，不浪费内存
- 缺点：效率较低（需要移动对象）
- 应用：老年代GC

**4. 分代收集算法**：
- 新生代：使用标记-复制算法
- 老年代：使用标记-清除或标记-整理算法

### 3.3 垃圾收集器 🔥

**垃圾收集器发展历程**：
```
Serial → Parallel → CMS → G1 → ZGC/Shenandoah
```

**1. Serial收集器**：
- 单线程收集器
- 收集时必须暂停所有工作线程（Stop The World）
- 适用场景：单核CPU、小内存应用

```bash
# 启用Serial收集器
-XX:+UseSerialGC
```


**2. Parallel收集器（吞吐量优先）**：
- 多线程并行收集
- 关注吞吐量（运行用户代码时间 / 总时间）
- 适用场景：后台计算任务、批处理

```bash
# 启用Parallel收集器
-XX:+UseParallelGC

# 设置并行线程数
-XX:ParallelGCThreads=4

# 设置最大GC停顿时间
-XX:MaxGCPauseMillis=100
```

**3. CMS收集器（Concurrent Mark Sweep，停顿时间优先）**：
- 以获取最短停顿时间为目标
- 并发收集，低停顿
- 适用场景：互联网应用、对响应时间敏感的应用

**CMS收集过程**：
```
1. 初始标记（STW）      ← 快速标记GC Roots直接关联的对象
2. 并发标记             ← 与用户线程并发执行
3. 重新标记（STW）      ← 修正并发标记期间的变动
4. 并发清除             ← 与用户线程并发执行
```

```bash
# 启用CMS收集器
-XX:+UseConcMarkSweepGC

# 设置CMS触发百分比
-XX:CMSInitiatingOccupancyFraction=70
```

**CMS的缺点** ⚠️：
- 对CPU资源敏感
- 无法处理浮动垃圾
- 产生内存碎片

**4. G1收集器（Garbage First）🔥**：
- JDK 9+默认收集器
- 面向服务端应用
- 可预测的停顿时间
- 不再区分新生代和老年代

**G1的特点**：
```
堆内存划分为多个Region（默认2048个）：
┌────┬────┬────┬────┬────┬────┬────┬────┐
│ E  │ E  │ S  │ O  │ O  │ H  │ E  │ O  │
└────┴────┴────┴────┴────┴────┴────┴────┘
E: Eden区
S: Survivor区
O: Old区
H: Humongous区（大对象）
```

```bash
# 启用G1收集器
-XX:+UseG1GC

# 设置期望的最大GC停顿时间
-XX:MaxGCPauseMillis=200

# 设置Region大小
-XX:G1HeapRegionSize=4m
```


**5. ZGC收集器（Z Garbage Collector）**：
- JDK 11引入
- 超低延迟（停顿时间<10ms）
- 支持TB级堆内存
- 适用场景：大内存、低延迟应用

```bash
# 启用ZGC收集器
-XX:+UseZGC

# 设置最大堆大小
-Xmx16g
```

**垃圾收集器对比**：

| 收集器 | 类型 | 目标 | 停顿时间 | 适用场景 |
|--------|------|------|----------|----------|
| Serial | 串行 | 简单 | 长 | 单核、小内存 |
| Parallel | 并行 | 吞吐量 | 较长 | 后台计算 |
| CMS | 并发 | 低停顿 | 短 | 互联网应用 |
| G1 | 并发 | 可预测停顿 | 可控 | 服务端应用 |
| ZGC | 并发 | 超低延迟 | 极短 | 大内存应用 |

## 🔥 类加载机制 ⚠️ 难点

### 4.1 类加载过程

类从被加载到虚拟机内存中开始，到卸载出内存为止，整个生命周期包括：

```
加载 → 验证 → 准备 → 解析 → 初始化 → 使用 → 卸载
      └────── 链接 ──────┘
```

**1. 加载（Loading）**：
- 通过类的全限定名获取二进制字节流
- 将字节流转化为方法区的运行时数据结构
- 在内存中生成Class对象

**2. 验证（Verification）**：
- 文件格式验证（魔数、版本号等）
- 元数据验证（语义分析）
- 字节码验证（数据流和控制流分析）
- 符号引用验证

**3. 准备（Preparation）**：
- 为类变量分配内存并设置初始值

```java
/**
 * 准备阶段示例
 * @author erik.zhou
 */
public class PrepareDemo {
    // 准备阶段：value = 0（零值）
    // 初始化阶段：value = 123
    public static int value = 123;
    
    // 准备阶段：CONSTANT = 456（final修饰，直接赋值）
    public static final int CONSTANT = 456;
}
```

**4. 解析（Resolution）**：
- 将符号引用转换为直接引用

**5. 初始化（Initialization）**：
- 执行类构造器`<clinit>()`方法
- 为类变量赋予正确的初始值


### 4.2 类加载器 🔥

**类加载器层次结构**：
```
Bootstrap ClassLoader（启动类加载器）
    ↓
Extension ClassLoader（扩展类加载器）
    ↓
Application ClassLoader（应用程序类加载器）
    ↓
Custom ClassLoader（自定义类加载器）
```

**1. Bootstrap ClassLoader**：
- 加载`<JAVA_HOME>/lib`目录下的核心类库
- 由C++实现，不是Java类

**2. Extension ClassLoader**：
- 加载`<JAVA_HOME>/lib/ext`目录下的扩展类库
- 由`sun.misc.Launcher$ExtClassLoader`实现

**3. Application ClassLoader**：
- 加载用户类路径（ClassPath）上的类
- 由`sun.misc.Launcher$AppClassLoader`实现
- 是程序默认的类加载器

**4. Custom ClassLoader**：
- 用户自定义的类加载器
- 继承`java.lang.ClassLoader`

### 4.3 双亲委派模型 🔥 ⚠️ 难点

**工作原理**：
```
1. 收到类加载请求
2. 委派给父类加载器加载
3. 父类加载器无法加载时，子类加载器才尝试加载
```

**示例代码**：
```java
/**
 * 双亲委派模型示例
 * @author erik.zhou
 */
public class ClassLoaderDemo {
    public static void main(String[] args) {
        // 获取类加载器
        ClassLoader classLoader = ClassLoaderDemo.class.getClassLoader();
        System.out.println("当前类加载器: " + classLoader);
        System.out.println("父类加载器: " + classLoader.getParent());
        System.out.println("祖父类加载器: " + classLoader.getParent().getParent());
        
        // 输出：
        // 当前类加载器: sun.misc.Launcher$AppClassLoader@xxx
        // 父类加载器: sun.misc.Launcher$ExtClassLoader@xxx
        // 祖父类加载器: null（Bootstrap ClassLoader由C++实现）
    }
}
```

**双亲委派的好处**：
- 避免类的重复加载
- 保护核心类库不被篡改

**打破双亲委派**：
- JDBC驱动加载（SPI机制）
- Tomcat类加载器
- OSGi模块化


## 💻 性能调优

### 5.1 JVM参数配置 🔥

**内存参数**：
```bash
# 堆内存设置
-Xms2g              # 初始堆大小
-Xmx4g              # 最大堆大小
-Xmn1g              # 新生代大小
-Xss256k            # 每个线程的栈大小

# 方法区设置（JDK 8+）
-XX:MetaspaceSize=128m
-XX:MaxMetaspaceSize=512m

# 直接内存设置
-XX:MaxDirectMemorySize=1g
```

**GC参数**：
```bash
# 选择垃圾收集器
-XX:+UseG1GC                    # 使用G1收集器
-XX:+UseZGC                     # 使用ZGC收集器
-XX:+UseConcMarkSweepGC         # 使用CMS收集器

# G1收集器参数
-XX:MaxGCPauseMillis=200        # 最大GC停顿时间
-XX:G1HeapRegionSize=4m         # Region大小
-XX:InitiatingHeapOccupancyPercent=45  # 触发并发GC的堆占用百分比

# GC日志参数
-XX:+PrintGCDetails             # 打印GC详细信息
-XX:+PrintGCDateStamps          # 打印GC时间戳
-Xloggc:/path/to/gc.log         # GC日志文件路径
```

**性能监控参数**：
```bash
# 堆转储
-XX:+HeapDumpOnOutOfMemoryError     # OOM时生成堆转储
-XX:HeapDumpPath=/path/to/dump      # 堆转储文件路径

# JMX监控
-Dcom.sun.management.jmxremote
-Dcom.sun.management.jmxremote.port=9999
-Dcom.sun.management.jmxremote.authenticate=false
-Dcom.sun.management.jmxremote.ssl=false
```

**完整配置示例**：
```bash
java -Xms2g -Xmx4g -Xmn1g -Xss256k \
     -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=512m \
     -XX:+UseG1GC -XX:MaxGCPauseMillis=200 \
     -XX:+PrintGCDetails -XX:+PrintGCDateStamps \
     -Xloggc:/var/log/gc.log \
     -XX:+HeapDumpOnOutOfMemoryError \
     -XX:HeapDumpPath=/var/log/heapdump.hprof \
     -jar application.jar
```


### 5.2 性能监控工具 🔥

**1. jps - 查看Java进程**：
```bash
# 查看所有Java进程
jps -l

# 输出示例：
# 12345 com.example.Application
# 12346 org.apache.catalina.startup.Bootstrap
```

**2. jstat - 监控JVM统计信息**：
```bash
# 查看GC统计信息
jstat -gc <pid> 1000 10

# 输出列说明：
# S0C: Survivor 0区容量
# S1C: Survivor 1区容量
# S0U: Survivor 0区使用量
# S1U: Survivor 1区使用量
# EC: Eden区容量
# EU: Eden区使用量
# OC: 老年代容量
# OU: 老年代使用量
# MC: 元空间容量
# MU: 元空间使用量
# YGC: Young GC次数
# YGCT: Young GC总时间
# FGC: Full GC次数
# FGCT: Full GC总时间
# GCT: GC总时间
```

**3. jmap - 生成堆转储快照**：
```bash
# 生成堆转储文件
jmap -dump:format=b,file=heap.hprof <pid>

# 查看堆内存使用情况
jmap -heap <pid>

# 查看对象统计信息
jmap -histo <pid> | head -20
```

**4. jstack - 生成线程快照**：
```bash
# 生成线程快照
jstack <pid> > thread.txt

# 检测死锁
jstack -l <pid>
```

**5. jinfo - 查看和修改JVM参数**：
```bash
# 查看所有JVM参数
jinfo -flags <pid>

# 查看特定参数
jinfo -flag MaxHeapSize <pid>

# 动态修改参数（部分参数支持）
jinfo -flag +PrintGCDetails <pid>
```

**6. VisualVM - 图形化监控工具**：
- 实时监控CPU、内存、线程
- 生成和分析堆转储
- 性能分析和采样
- 插件扩展

**7. JConsole - JMX监控工具**：
- 监控内存使用
- 监控线程状态
- 监控类加载
- 监控MBean


### 5.3 内存溢出排查 🔥 ⚠️ 难点

**1. 堆内存溢出（java.lang.OutOfMemoryError: Java heap space）**：

**原因**：
- 内存泄漏（对象无法被GC回收）
- 堆内存设置过小
- 创建了大量大对象

**排查步骤**：
```bash
# 1. 生成堆转储文件
jmap -dump:format=b,file=heap.hprof <pid>

# 2. 使用MAT（Memory Analyzer Tool）分析
# - 查看Leak Suspects（泄漏嫌疑）
# - 查看Dominator Tree（支配树）
# - 查看对象引用链

# 3. 找出占用内存最多的对象
jmap -histo:live <pid> | head -20
```

**示例代码**：
```java
/**
 * 堆内存溢出示例
 * @author erik.zhou
 */
public class HeapOOMDemo {
    static class OOMObject {}
    
    public static void main(String[] args) {
        List<OOMObject> list = new ArrayList<>();
        while (true) {
            list.add(new OOMObject()); // 不断创建对象，导致OOM
        }
    }
}

// 运行参数：-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
```

**2. 栈溢出（java.lang.StackOverflowError）**：

**原因**：
- 递归调用层次过深
- 方法调用链过长

**示例代码**：
```java
/**
 * 栈溢出示例
 * @author erik.zhou
 */
public class StackOverflowDemo {
    private int stackLength = 0;
    
    public void stackLeak() {
        stackLength++;
        stackLeak(); // 无限递归
    }
    
    public static void main(String[] args) {
        StackOverflowDemo demo = new StackOverflowDemo();
        try {
            demo.stackLeak();
        } catch (Throwable e) {
            System.out.println("栈深度: " + demo.stackLength);
            throw e;
        }
    }
}

// 运行参数：-Xss256k
```

**3. 元空间溢出（java.lang.OutOfMemoryError: Metaspace）**：

**原因**：
- 动态生成大量类（如使用CGLib）
- 元空间设置过小

**示例代码**：
```java
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;

/**
 * 元空间溢出示例
 * @author erik.zhou
 */
public class MetaspaceOOMDemo {
    public static void main(String[] args) {
        while (true) {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(OOMObject.class);
            enhancer.setUseCache(false);
            enhancer.setCallback((MethodInterceptor) (obj, method, args1, proxy) -> 
                proxy.invokeSuper(obj, args1)
            );
            enhancer.create(); // 动态生成类
        }
    }
    
    static class OOMObject {}
}

// 运行参数：-XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m
```


## ✨ 最佳实践

### 6.1 JVM参数调优建议

**生产环境推荐配置**：
```bash
# 1. 堆内存设置
# - 初始堆大小和最大堆大小设置为相同值，避免动态扩容
# - 新生代设置为堆的1/3到1/4
-Xms4g -Xmx4g -Xmn1g

# 2. 使用G1收集器（JDK 8+推荐）
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200
-XX:G1HeapRegionSize=4m

# 3. 启用GC日志
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:/var/log/gc-%t.log
-XX:+UseGCLogFileRotation
-XX:NumberOfGCLogFiles=10
-XX:GCLogFileSize=100M

# 4. OOM时生成堆转储
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/log/heapdump.hprof

# 5. 元空间设置
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m
```

### 6.2 性能优化技巧

**1. 减少Full GC频率**：
```java
/**
 * 避免频繁Full GC
 * @author erik.zhou
 */
public class ReduceFullGC {
    // ❌ 错误：频繁创建大对象
    public void badPractice() {
        for (int i = 0; i < 1000; i++) {
            byte[] largeArray = new byte[10 * 1024 * 1024]; // 10MB
            // 使用后立即丢弃，可能触发Full GC
        }
    }
    
    // ✅ 正确：复用对象或使用对象池
    private byte[] buffer = new byte[10 * 1024 * 1024];
    
    public void goodPractice() {
        for (int i = 0; i < 1000; i++) {
            // 复用buffer
            Arrays.fill(buffer, (byte) 0);
        }
    }
}
```

**2. 合理设置对象大小**：
```java
/**
 * 对象大小优化
 * @author erik.zhou
 */
public class ObjectSizeOptimization {
    // ❌ 错误：使用包装类型
    private Integer count = 0;
    private Long timestamp = 0L;
    
    // ✅ 正确：使用基本类型（节省内存）
    private int count2 = 0;
    private long timestamp2 = 0L;
}
```


**3. 避免内存泄漏**：
```java
import java.util.*;

/**
 * 避免内存泄漏
 * @author erik.zhou
 */
public class AvoidMemoryLeak {
    // ❌ 错误：静态集合持有对象引用
    private static List<Object> cache = new ArrayList<>();
    
    public void badPractice(Object obj) {
        cache.add(obj); // 对象永远不会被GC
    }
    
    // ✅ 正确：使用WeakHashMap或定期清理
    private static Map<Object, Object> cache2 = new WeakHashMap<>();
    
    public void goodPractice(Object key, Object value) {
        cache2.put(key, value); // 弱引用，可以被GC
    }
    
    // ✅ 正确：使用try-with-resources自动关闭资源
    public void closeResource() {
        try (FileInputStream fis = new FileInputStream("file.txt")) {
            // 使用资源
        } catch (IOException e) {
            e.printStackTrace();
        }
        // 资源自动关闭
    }
}
```

### 6.3 监控指标

**关键监控指标**：

| 指标 | 说明 | 告警阈值 |
|------|------|----------|
| 堆内存使用率 | 已使用堆内存 / 最大堆内存 | > 85% |
| Full GC频率 | 每小时Full GC次数 | > 5次/小时 |
| GC停顿时间 | 单次GC的停顿时间 | > 1秒 |
| 元空间使用率 | 已使用元空间 / 最大元空间 | > 90% |
| 线程数 | 活跃线程数量 | > 1000 |

**监控脚本示例**：
```bash
#!/bin/bash
# JVM监控脚本
# @author erik.zhou

PID=$(jps -l | grep "Application" | awk '{print $1}')

if [ -z "$PID" ]; then
    echo "进程未找到"
    exit 1
fi

echo "=== JVM监控信息 ==="
echo "进程ID: $PID"
echo ""

echo "=== 堆内存使用情况 ==="
jmap -heap $PID | grep -A 5 "Heap Usage"
echo ""

echo "=== GC统计信息 ==="
jstat -gc $PID
echo ""

echo "=== 线程统计 ==="
jstack $PID | grep "java.lang.Thread.State" | sort | uniq -c
```


## ❓ 常见问题

### Q1: 什么时候会触发Full GC？
**A**:
1. 老年代空间不足
2. 元空间不足
3. 显式调用`System.gc()`（不推荐）
4. CMS GC出现promotion failed或concurrent mode failure
5. Minor GC晋升到老年代的平均大小大于老年代剩余空间

### Q2: 如何选择垃圾收集器？
**A**:
- **小应用（<100MB堆）**: Serial GC
- **后台计算任务**: Parallel GC（吞吐量优先）
- **互联网应用**: G1 GC（平衡吞吐量和停顿时间）
- **大内存低延迟**: ZGC（停顿时间<10ms）

### Q3: -Xms和-Xmx应该设置为相同值吗？
**A**: 
是的，建议设置为相同值。原因：
- 避免堆动态扩容和收缩的开销
- 减少GC频率
- 提高性能稳定性

### Q4: 如何判断是否发生了内存泄漏？
**A**:
1. 观察堆内存使用趋势，持续上升且不下降
2. Full GC后老年代内存占用仍然很高
3. 使用jmap生成堆转储，用MAT分析
4. 查看Leak Suspects报告

### Q5: 什么是Stop The World（STW）？
**A**:
- STW是指GC时暂停所有应用线程
- 所有GC都会产生STW，只是时间长短不同
- 目标是减少STW时间，而不是完全消除

### Q6: 新生代和老年代的比例如何设置？
**A**:
- 默认比例：新生代:老年代 = 1:2（-XX:NewRatio=2）
- 新生代过小：频繁Minor GC
- 新生代过大：Minor GC时间长，老年代空间不足
- 建议：根据应用特点调整，一般新生代占堆的1/3到1/4

### Q7: 如何分析GC日志？
**A**:
```
[GC (Allocation Failure) [PSYoungGen: 2048K->512K(2560K)] 2048K->520K(9728K), 0.0012345 secs]
```
解读：
- `GC`: Minor GC
- `Allocation Failure`: GC原因（分配失败）
- `PSYoungGen`: 使用Parallel Scavenge收集器
- `2048K->512K(2560K)`: 新生代GC前后大小（总大小）
- `2048K->520K(9728K)`: 堆GC前后大小（总大小）
- `0.0012345 secs`: GC耗时


### Q8: 什么是对象的逃逸分析？
**A**:
- 逃逸分析是JIT编译器的优化技术
- 分析对象的作用域，判断对象是否会逃逸出方法或线程
- 如果对象不逃逸，可以进行以下优化：
  - 栈上分配（减少GC压力）
  - 标量替换（将对象拆分为基本类型）
  - 同步消除（去除不必要的锁）

```java
/**
 * 逃逸分析示例
 * @author erik.zhou
 */
public class EscapeAnalysisDemo {
    // 对象逃逸：返回对象引用
    public User createUser1() {
        User user = new User();
        return user; // 对象逃逸
    }
    
    // 对象不逃逸：对象仅在方法内使用
    public void createUser2() {
        User user = new User();
        System.out.println(user.getName()); // 对象不逃逸，可能栈上分配
    }
}
```

## 🔗 相关资源

### 官方文档
- [Java SE HotSpot Virtual Machine Garbage Collection Tuning Guide](https://docs.oracle.com/en/java/javase/21/gctuning/)
- [JVM Tool Interface](https://docs.oracle.com/en/java/javase/21/docs/specs/jvmti.html)
- [Java Virtual Machine Specification](https://docs.oracle.com/javase/specs/jvms/se21/html/)

### 推荐书籍
- 《深入理解Java虚拟机》- 周志明
- 《Java性能优化权威指南》- Charlie Hunt
- 《垃圾回收算法手册》- Richard Jones

### 在线工具
- [GCeasy](https://gceasy.io/) - GC日志分析工具
- [Eclipse Memory Analyzer (MAT)](https://www.eclipse.org/mat/) - 堆转储分析工具
- [VisualVM](https://visualvm.github.io/) - JVM监控工具

### 推荐文章
- [Understanding Java Garbage Collection](https://www.oracle.com/technical-resources/articles/java/g1gc.html)
- [JVM Performance Optimization](https://www.baeldung.com/jvm-performance-optimization)


## 📝 学习检查清单

### JVM架构
- [ ] 理解JVM的整体架构和各组件的作用
- [ ] 掌握类加载子系统的工作流程
- [ ] 理解运行时数据区的划分
- [ ] 掌握执行引擎的工作原理

### 内存模型
- [ ] 理解程序计数器的作用
- [ ] 掌握虚拟机栈的结构和栈帧的组成
- [ ] 深入理解堆的分代模型
- [ ] 理解方法区和元空间的区别
- [ ] 掌握运行时常量池的作用

### 垃圾回收
- [ ] 理解可达性分析算法
- [ ] 掌握标记-清除、标记-复制、标记-整理算法
- [ ] 理解分代收集算法
- [ ] 掌握Serial、Parallel、CMS、G1、ZGC收集器的特点
- [ ] 能够根据应用场景选择合适的垃圾收集器
- [ ] 理解GC日志的含义

### 类加载机制
- [ ] 掌握类加载的完整过程
- [ ] 理解类加载器的层次结构
- [ ] 深入理解双亲委派模型
- [ ] 了解打破双亲委派的场景

### 性能调优
- [ ] 掌握常用JVM参数的配置
- [ ] 熟练使用jps、jstat、jmap、jstack等工具
- [ ] 能够分析GC日志
- [ ] 能够使用MAT分析堆转储文件
- [ ] 掌握内存溢出的排查方法
- [ ] 理解性能优化的最佳实践

### 实战能力
- [ ] 能够为生产环境配置合适的JVM参数
- [ ] 能够监控JVM运行状态
- [ ] 能够分析和解决内存问题
- [ ] 能够进行JVM性能调优

---

**@author erik.zhou**
**文档版本**: 1.0
**最后更新**: 2024-12-31
**文档来源**: OpenJDK官方文档 + Context7查询
