# Hadoop 完整教程

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

- **版本**: 3.3.6 (LTS)
- **官方文档**: https://hadoop.apache.org/
- **学习难度**: ⭐⭐⭐⭐ (4/5星)
- **重要程度**: ⭐⭐⭐ (3/5星)
- **前置知识**: 
  - Java基础
  - Linux基本操作
  - 分布式系统基本概念
  - 网络编程基础

**文档来源**: Apache Hadoop官方文档 + 社区最佳实践

## 🎯 学习目标

- [ ] 理解Hadoop的核心架构和设计理念
- [ ] 掌握HDFS分布式文件系统的原理和使用
- [ ] 理解MapReduce编程模型
- [ ] 掌握YARN资源管理框架
- [ ] 能够搭建Hadoop集群环境
- [ ] 了解Hadoop在Java后端的应用场景

## 📖 基础概念

### 1.1 什么是Hadoop

Hadoop是Apache软件基金会开发的一个开源分布式计算框架，用于存储和处理大规模数据集。它的核心设计理念是：
- **分布式存储**: 将大文件分块存储在多台机器上
- **分布式计算**: 将计算任务分发到数据所在的节点
- **容错性**: 通过数据副本机制保证可靠性
- **横向扩展**: 通过增加机器来提升存储和计算能力

### 1.2 核心组件

Hadoop生态系统包含三大核心组件：

1. **HDFS (Hadoop Distributed File System)**: 分布式文件系统
2. **MapReduce**: 分布式计算框架
3. **YARN (Yet Another Resource Negotiator)**: 资源管理框架

### 1.3 应用场景

- **日志分析**: 处理海量服务器日志
- **数据仓库**: 构建企业级数据仓库
- **推荐系统**: 离线计算用户推荐
- **数据备份**: 大规模数据的可靠存储
- **ETL处理**: 数据抽取、转换、加载

## 🔥 核心特性 (重点)

### 2.1 HDFS分布式文件系统 🔥

HDFS是Hadoop的核心存储系统，专为大文件存储和批量读取设计。

#### 2.1.1 架构设计

```
Client
  ↓
NameNode (主节点)
  ↓
DataNode1  DataNode2  DataNode3 (从节点)
```

**核心角色**:
- **NameNode**: 管理文件系统的命名空间和元数据
  - 维护文件目录树
  - 记录每个文件的块信息
  - 管理DataNode的心跳和块报告
  
- **DataNode**: 存储实际的数据块
  - 定期向NameNode发送心跳
  - 执行数据块的读写操作
  - 负责数据块的复制

- **SecondaryNameNode**: 辅助NameNode
  - 定期合并编辑日志
  - 不是NameNode的热备份

#### 2.1.2 数据存储机制 (⚠️ 难点)

**块存储**:
- 默认块大小: 128MB (Hadoop 2.x+)
- 大文件被切分成多个块
- 每个块默认复制3份存储在不同节点

**副本放置策略**:
```
第一个副本: 写入客户端所在节点
第二个副本: 不同机架的随机节点
第三个副本: 与第二个副本同机架的不同节点
```

这种策略平衡了可靠性、写入性能和网络带宽。

#### 2.1.3 读写流程

**写入流程**:
```java
1. Client向NameNode请求上传文件
2. NameNode检查权限和文件是否存在
3. NameNode返回可用的DataNode列表
4. Client将文件切分成块，依次写入DataNode
5. DataNode之间建立Pipeline进行副本复制
6. 所有副本写入完成后，通知NameNode
```

**读取流程**:
```java
1. Client向NameNode请求文件位置
2. NameNode返回文件块所在的DataNode列表
3. Client选择最近的DataNode读取数据
4. 如果读取失败，尝试其他副本
```

### 2.2 MapReduce计算框架 🔥

MapReduce是一种编程模型，用于大规模数据集的并行计算。

#### 2.2.1 编程模型

MapReduce将计算分为两个阶段：

**Map阶段**: 处理输入数据，输出键值对
```java
map(K1 key, V1 value) -> list(K2, V2)
```

**Reduce阶段**: 合并相同key的value
```java
reduce(K2 key, list(V2) values) -> list(K3, V3)
```

#### 2.2.2 执行流程 (⚠️ 难点)

```
Input → Split → Map → Shuffle → Sort → Reduce → Output
```

1. **Input**: 读取HDFS上的输入文件
2. **Split**: 将输入文件切分成多个InputSplit
3. **Map**: 每个Split由一个Map任务处理
4. **Shuffle**: 将Map输出按key分组，传输到Reduce节点
5. **Sort**: 对每个Reduce的输入按key排序
6. **Reduce**: 处理每个key的所有value
7. **Output**: 将结果写入HDFS

**Shuffle机制**是MapReduce的核心，也是性能瓶颈：
- Map端: 内存缓冲区 → 溢写到磁盘 → 合并排序
- Reduce端: 拉取数据 → 合并排序 → 传给Reduce函数

### 2.3 YARN资源管理 🔥

YARN是Hadoop 2.x引入的资源管理框架，将资源管理和任务调度分离。

#### 2.3.1 架构组件

```
Client
  ↓
ResourceManager (全局资源管理)
  ↓
NodeManager1  NodeManager2  NodeManager3 (节点资源管理)
  ↓
Container (资源容器)
```

**核心角色**:
- **ResourceManager**: 全局资源调度器
  - 处理客户端请求
  - 启动和监控ApplicationMaster
  - 分配资源给各个应用
  
- **NodeManager**: 节点资源管理器
  - 管理单个节点的资源
  - 启动和监控Container
  - 向ResourceManager汇报资源使用情况
  
- **ApplicationMaster**: 应用管理器
  - 每个应用有一个AM
  - 向RM申请资源
  - 与NM通信启动Container
  
- **Container**: 资源容器
  - 封装CPU、内存等资源
  - 运行具体的任务

#### 2.3.2 任务提交流程

```java
1. Client提交应用到ResourceManager
2. RM分配Container启动ApplicationMaster
3. AM向RM注册并申请资源
4. RM分配Container给AM
5. AM与NM通信，在Container中启动任务
6. 任务执行完成，AM向RM注销
```

## 💻 实战应用

### 3.1 环境搭建

#### 3.1.1 单机伪分布式模式

**系统要求**:
- Linux系统 (推荐CentOS 7+)
- JDK 8+
- SSH免密登录

**安装步骤**:

```bash
# 1. 下载Hadoop
wget https://downloads.apache.org/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz
tar -zxvf hadoop-3.3.6.tar.gz
mv hadoop-3.3.6 /usr/local/hadoop

# 2. 配置环境变量
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

# 3. 配置core-site.xml
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>/usr/local/hadoop/tmp</value>
    </property>
</configuration>

# 4. 配置hdfs-site.xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
</configuration>

# 5. 格式化NameNode
hdfs namenode -format

# 6. 启动HDFS
start-dfs.sh

# 7. 验证
jps  # 应该看到NameNode、DataNode、SecondaryNameNode
```

#### 3.1.2 完全分布式集群

**集群规划** (3节点示例):
```
hadoop01: NameNode, ResourceManager
hadoop02: DataNode, NodeManager
hadoop03: DataNode, NodeManager
```

**配置要点**:
- 配置主机名和hosts映射
- 配置SSH免密登录
- 同步配置文件到所有节点
- 在slaves文件中配置DataNode列表

### 3.2 HDFS基本操作

#### 3.2.1 命令行操作

```bash
# 创建目录
hdfs dfs -mkdir /user/hadoop

# 上传文件
hdfs dfs -put localfile.txt /user/hadoop/

# 查看文件
hdfs dfs -ls /user/hadoop
hdfs dfs -cat /user/hadoop/localfile.txt

# 下载文件
hdfs dfs -get /user/hadoop/localfile.txt ./

# 删除文件
hdfs dfs -rm /user/hadoop/localfile.txt

# 查看文件块信息
hdfs fsck /user/hadoop/localfile.txt -files -blocks -locations
```

#### 3.2.2 Java API操作

```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;
import org.apache.hadoop.io.IOUtils;

/**
 * HDFS文件操作工具类
 * 
 * @author erik.zhou
 */
public class HdfsUtil {
    
    private static final String HDFS_URI = "hdfs://localhost:9000";
    
    /**
     * 获取FileSystem实例
     */
    public static FileSystem getFileSystem() throws Exception {
        Configuration conf = new Configuration();
        conf.set("fs.defaultFS", HDFS_URI);
        return FileSystem.get(conf);
    }
    
    /**
     * 上传文件到HDFS
     */
    public static void uploadFile(String localPath, String hdfsPath) throws Exception {
        FileSystem fs = getFileSystem();
        Path src = new Path(localPath);
        Path dst = new Path(hdfsPath);
        fs.copyFromLocalFile(src, dst);
        fs.close();
    }
    
    /**
     * 从HDFS下载文件
     */
    public static void downloadFile(String hdfsPath, String localPath) throws Exception {
        FileSystem fs = getFileSystem();
        Path src = new Path(hdfsPath);
        Path dst = new Path(localPath);
        fs.copyToLocalFile(src, dst);
        fs.close();
    }
    
    /**
     * 读取HDFS文件内容
     */
    public static String readFile(String hdfsPath) throws Exception {
        FileSystem fs = getFileSystem();
        Path path = new Path(hdfsPath);
        FSDataInputStream in = fs.open(path);
        
        byte[] buffer = new byte[1024];
        StringBuilder content = new StringBuilder();
        int bytesRead;
        
        while ((bytesRead = in.read(buffer)) > 0) {
            content.append(new String(buffer, 0, bytesRead));
        }
        
        IOUtils.closeStream(in);
        fs.close();
        
        return content.toString();
    }
    
    /**
     * 写入内容到HDFS文件
     */
    public static void writeFile(String hdfsPath, String content) throws Exception {
        FileSystem fs = getFileSystem();
        Path path = new Path(hdfsPath);
        FSDataOutputStream out = fs.create(path);
        out.write(content.getBytes());
        out.close();
        fs.close();
    }
    
    /**
     * 删除HDFS文件或目录
     */
    public static boolean deleteFile(String hdfsPath) throws Exception {
        FileSystem fs = getFileSystem();
        Path path = new Path(hdfsPath);
        boolean result = fs.delete(path, true);
        fs.close();
        return result;
    }
}
```

### 3.3 MapReduce编程实战

#### 3.3.1 WordCount示例

经典的单词计数程序，统计文本中每个单词出现的次数。

**Mapper类**:
```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

/**
 * WordCount Mapper
 * 
 * @author erik.zhou
 */
public class WordCountMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    
    private final static IntWritable ONE = new IntWritable(1);
    private Text word = new Text();
    
    @Override
    protected void map(LongWritable key, Text value, Context context) 
            throws IOException, InterruptedException {
        // 获取一行文本
        String line = value.toString();
        
        // 分割单词
        String[] words = line.split("\\s+");
        
        // 输出每个单词和计数1
        for (String w : words) {
            word.set(w);
            context.write(word, ONE);
        }
    }
}
```

**Reducer类**:
```java
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

/**
 * WordCount Reducer
 * 
 * @author erik.zhou
 */
public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    
    private IntWritable result = new IntWritable();
    
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) 
            throws IOException, InterruptedException {
        // 累加相同单词的计数
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        
        result.set(sum);
        context.write(key, result);
    }
}
```

**Driver类**:
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

/**
 * WordCount主程序
 * 
 * @author erik.zhou
 */
public class WordCountDriver {
    
    public static void main(String[] args) throws Exception {
        if (args.length != 2) {
            System.err.println("Usage: WordCountDriver <input path> <output path>");
            System.exit(-1);
        }
        
        // 创建配置
        Configuration conf = new Configuration();
        
        // 创建Job
        Job job = Job.getInstance(conf, "word count");
        job.setJarByClass(WordCountDriver.class);
        
        // 设置Mapper和Reducer
        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);
        
        // 设置输出类型
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        
        // 设置输入输出路径
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        
        // 提交任务
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

**运行任务**:
```bash
# 编译
javac -classpath `hadoop classpath` WordCount*.java
jar -cvf wordcount.jar WordCount*.class

# 准备输入数据
echo "hello world hello hadoop" > input.txt
hdfs dfs -put input.txt /input/

# 运行MapReduce任务
hadoop jar wordcount.jar WordCountDriver /input /output

# 查看结果
hdfs dfs -cat /output/part-r-00000
```

### 3.4 与Java后端集成

#### 3.4.1 Spring Boot集成Hadoop

**Maven依赖**:
```xml
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-client</artifactId>
    <version>3.3.6</version>
</dependency>
```

**配置类**:
```java
import org.apache.hadoop.fs.FileSystem;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Hadoop配置类
 * 
 * @author erik.zhou
 */
@Configuration
public class HadoopConfig {
    
    @Value("${hadoop.hdfs.uri}")
    private String hdfsUri;
    
    @Bean
    public FileSystem fileSystem() throws Exception {
        org.apache.hadoop.conf.Configuration conf = new org.apache.hadoop.conf.Configuration();
        conf.set("fs.defaultFS", hdfsUri);
        conf.set("dfs.client.use.datanode.hostname", "true");
        return FileSystem.get(conf);
    }
}
```

**Service层**:
```java
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.web.multipart.MultipartFile;

import java.io.IOException;

/**
 * HDFS文件服务
 * 
 * @author erik.zhou
 */
@Service
public class HdfsService {
    
    @Autowired
    private FileSystem fileSystem;
    
    /**
     * 上传文件到HDFS
     */
    public String uploadFile(MultipartFile file, String destPath) throws IOException {
        Path path = new Path(destPath);
        fileSystem.copyFromLocalFile(false, true, 
            new Path(file.getOriginalFilename()), path);
        return destPath;
    }
    
    /**
     * 删除HDFS文件
     */
    public boolean deleteFile(String filePath) throws IOException {
        return fileSystem.delete(new Path(filePath), true);
    }
    
    /**
     * 检查文件是否存在
     */
    public boolean exists(String filePath) throws IOException {
        return fileSystem.exists(new Path(filePath));
    }
}
```

## ✨ 最佳实践

### 4.1 HDFS使用建议

#### 4.1.1 文件大小优化

**推荐做法**:
- 单个文件大小: 128MB - 1GB
- 避免大量小文件 (< 10MB)
- 使用SequenceFile或Avro合并小文件

**原因**:
- NameNode内存有限，每个文件/块都占用内存
- 小文件会导致大量的元数据开销
- MapReduce处理小文件效率低

**小文件合并示例**:
```java
/**
 * 合并小文件到SequenceFile
 * 
 * @author erik.zhou
 */
public class SmallFileMerger {
    
    public static void mergeFiles(String inputDir, String outputFile) throws Exception {
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(conf);
        
        // 创建SequenceFile Writer
        SequenceFile.Writer writer = SequenceFile.createWriter(
            conf,
            SequenceFile.Writer.file(new Path(outputFile)),
            SequenceFile.Writer.keyClass(Text.class),
            SequenceFile.Writer.valueClass(BytesWritable.class)
        );
        
        // 遍历输入目录
        FileStatus[] files = fs.listStatus(new Path(inputDir));
        for (FileStatus file : files) {
            if (file.isFile()) {
                // 读取文件内容
                FSDataInputStream in = fs.open(file.getPath());
                byte[] buffer = new byte[(int) file.getLen()];
                in.readFully(buffer);
                in.close();
                
                // 写入SequenceFile
                writer.append(new Text(file.getPath().getName()), 
                             new BytesWritable(buffer));
            }
        }
        
        writer.close();
        fs.close();
    }
}
```

#### 4.1.2 副本数量设置

**推荐配置**:
- 重要数据: 3副本 (默认)
- 临时数据: 1-2副本
- 冷数据: 2副本

**动态调整副本数**:
```bash
# 设置文件副本数为2
hdfs dfs -setrep -w 2 /user/hadoop/data.txt
```

#### 4.1.3 数据压缩

**推荐压缩格式**:
- **Snappy**: 压缩速度快，适合中间数据
- **LZO**: 支持分片，适合MapReduce输入
- **Gzip**: 压缩率高，适合归档数据

**启用压缩**:
```java
// 在MapReduce中启用压缩
Configuration conf = new Configuration();
conf.setBoolean("mapreduce.map.output.compress", true);
conf.setClass("mapreduce.map.output.compress.codec", 
              SnappyCodec.class, CompressionCodec.class);
```

### 4.2 MapReduce性能优化

#### 4.2.1 Combiner优化

使用Combiner在Map端进行局部聚合，减少网络传输：

```java
// 在Driver中设置Combiner
job.setCombinerClass(WordCountReducer.class);
```

#### 4.2.2 分区优化

自定义Partitioner实现数据均衡分布：

```java
/**
 * 自定义分区器
 * 
 * @author erik.zhou
 */
public class CustomPartitioner extends Partitioner<Text, IntWritable> {
    
    @Override
    public int getPartition(Text key, IntWritable value, int numPartitions) {
        // 根据key的首字母分区
        char firstChar = key.toString().charAt(0);
        return (firstChar % numPartitions);
    }
}
```

#### 4.2.3 内存调优

**关键参数**:
```xml
<!-- Map任务内存 -->
<property>
    <name>mapreduce.map.memory.mb</name>
    <value>2048</value>
</property>

<!-- Reduce任务内存 -->
<property>
    <name>mapreduce.reduce.memory.mb</name>
    <value>4096</value>
</property>

<!-- Map输出缓冲区大小 -->
<property>
    <name>mapreduce.task.io.sort.mb</name>
    <value>200</value>
</property>
```

### 4.3 集群运维建议

#### 4.3.1 监控指标

**HDFS监控**:
- NameNode堆内存使用率
- DataNode磁盘使用率
- 块损坏数量
- 副本不足的块数量

**YARN监控**:
- 集群资源使用率
- 任务队列长度
- 任务失败率
- 节点健康状态

**监控工具**:
- Hadoop自带Web UI (NameNode: 9870, ResourceManager: 8088)
- Ambari
- Cloudera Manager
- Prometheus + Grafana

#### 4.3.2 容量规划

**存储容量**:
```
实际可用容量 = 总磁盘容量 × (1 - 预留空间) / 副本数
```

**计算资源**:
- CPU: 每个节点8-16核
- 内存: 每个节点64-128GB
- 磁盘: 每个节点12-24块SATA盘

#### 4.3.3 备份策略

**NameNode元数据备份**:
```bash
# 配置多个元数据存储目录
<property>
    <name>dfs.namenode.name.dir</name>
    <value>file:///data1/hadoop/name,file:///data2/hadoop/name</value>
</property>

# 定期备份fsimage和edits
hdfs dfsadmin -fetchImage /backup/fsimage
```

## ⚠️ 常见陷阱

### 陷阱1: NameNode单点故障

**问题**: NameNode宕机导致整个集群不可用

**解决方案**:
- 配置NameNode HA (High Availability)
- 使用ZooKeeper实现自动故障转移
- 定期备份元数据

### 陷阱2: 数据倾斜

**问题**: 某些Reduce任务处理的数据量远大于其他任务

**解决方案**:
- 使用自定义Partitioner
- 对热点key进行预处理
- 增加Reduce任务数量

### 陷阱3: 小文件问题

**问题**: 大量小文件导致NameNode内存不足

**解决方案**:
- 使用HAR (Hadoop Archive)归档小文件
- 使用SequenceFile合并小文件
- 在数据写入时就避免产生小文件

### 陷阱4: 磁盘空间不均衡

**问题**: 某些DataNode磁盘满，其他节点空闲

**解决方案**:
```bash
# 启动数据均衡器
hdfs balancer -threshold 10
```

## ❓ 常见问题

### Q1: Hadoop适合什么样的场景？

**A**: Hadoop适合以下场景：
- **大规模数据存储**: TB/PB级别的数据
- **批处理计算**: 离线数据分析、ETL
- **一次写入多次读取**: 数据不频繁修改
- **高吞吐量**: 对延迟要求不高

**不适合的场景**:
- 实时计算 (使用Spark/Flink)
- 小文件存储
- 随机读写 (使用HBase)
- 低延迟查询 (使用Elasticsearch)

### Q2: HDFS和传统文件系统有什么区别？

**A**: 主要区别：
- **文件大小**: HDFS适合大文件，传统FS适合各种大小
- **访问模式**: HDFS一次写入多次读取，传统FS支持随机读写
- **数据位置**: HDFS数据分布在多台机器，传统FS在单机
- **容错性**: HDFS通过副本机制保证可靠性
- **性能**: HDFS优化了大文件的顺序读写

### Q3: MapReduce和Spark有什么区别？

**A**: 主要区别：
- **计算模型**: MapReduce基于磁盘，Spark基于内存
- **性能**: Spark比MapReduce快10-100倍
- **易用性**: Spark API更简洁，支持多种语言
- **适用场景**: MapReduce适合批处理，Spark适合迭代计算和实时处理

**建议**: 新项目优先考虑Spark，但Hadoop生态仍然重要。

### Q4: 如何选择合适的块大小？

**A**: 块大小选择考虑因素：
- **文件大小**: 块大小应该是文件大小的1/100到1/10
- **网络带宽**: 块越大，网络传输时间占比越小
- **Map任务数**: 块数量决定Map任务数量

**推荐配置**:
- 默认128MB适合大多数场景
- 超大文件 (>10GB) 可以使用256MB或512MB
- 小文件场景考虑使用64MB

### Q5: Hadoop在Java后端项目中的应用场景？

**A**: 典型应用场景：
1. **日志分析**: 收集和分析应用日志
2. **数据仓库**: 构建离线数据仓库
3. **报表生成**: 生成每日/每月业务报表
4. **推荐系统**: 离线计算用户推荐
5. **数据备份**: 大规模数据的可靠存储

**集成方式**:
- 使用Hadoop Client API直接操作HDFS
- 通过Hive进行SQL查询
- 使用Sqoop导入导出数据
- 结合Spark进行数据处理

## 🔗 相关资源

### 官方文档
- [Apache Hadoop官网](https://hadoop.apache.org/)
- [Hadoop文档](https://hadoop.apache.org/docs/stable/)
- [HDFS架构指南](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsDesign.html)
- [MapReduce教程](https://hadoop.apache.org/docs/stable/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)

### 推荐书籍
- 《Hadoop权威指南》(第4版)
- 《Hadoop实战》
- 《大数据技术原理与应用》

### 在线资源
- [Hadoop中文社区](http://hadoop.apache.org/zh-cn/)
- [Cloudera文档](https://docs.cloudera.com/)
- [Hortonworks文档](https://docs.hortonworks.com/)

### 相关技术
- **Hive**: 数据仓库工具，提供SQL接口
- **HBase**: 分布式NoSQL数据库
- **Spark**: 内存计算框架
- **Flink**: 流处理框架
- **Sqoop**: 数据导入导出工具

## 📝 学习检查清单

- [ ] 理解Hadoop的核心架构和设计理念
- [ ] 掌握HDFS的读写流程和副本机制
- [ ] 理解MapReduce的执行流程和Shuffle机制
- [ ] 掌握YARN的资源管理和任务调度
- [ ] 能够编写简单的MapReduce程序
- [ ] 了解Hadoop的性能优化方法
- [ ] 掌握Hadoop集群的运维要点
- [ ] 了解Hadoop在Java后端的应用场景

---

**@author** erik.zhou
**最后更新**: 2024-01-04
**文档版本**: 1.0
