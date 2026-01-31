# Spark 完整教程

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

- **版本**: 3.5.0 (最新稳定版)
- **官方文档**: https://spark.apache.org/
- **学习难度**: ⭐⭐⭐⭐ (4/5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (5/5星)
- **前置知识**: 
  - Java/Scala基础
  - Hadoop基础概念
  - 分布式系统原理
  - 函数式编程思想

**文档来源**: Apache Spark官方文档 (Context7)

## 🎯 学习目标

- [ ] 理解Spark的核心架构和设计理念
- [ ] 掌握RDD编程模型
- [ ] 掌握DataFrame和Dataset API
- [ ] 理解Spark SQL的使用
- [ ] 了解Spark的内存计算原理
- [ ] 掌握Spark性能优化技巧
- [ ] 能够在Java项目中集成Spark

## 📖 基础概念

### 1.1 什么是Spark

Apache Spark是一个快速、通用的大规模数据处理引擎，专为大规模数据处理而设计。它的核心特点是：

- **内存计算**: 数据缓存在内存中，比Hadoop MapReduce快10-100倍
- **通用性**: 支持批处理、流处理、机器学习、图计算
- **易用性**: 提供Java、Scala、Python、R等多种语言API
- **兼容性**: 可以运行在Hadoop、Kubernetes等多种平台

### 1.2 Spark vs Hadoop MapReduce

| 特性 | Spark | Hadoop MapReduce |
|------|-------|------------------|
| 计算模型 | 内存计算 | 磁盘计算 |
| 性能 | 快10-100倍 | 较慢 |
| 易用性 | API简洁 | 代码冗长 |
| 实时性 | 支持流处理 | 仅批处理 |
| 迭代计算 | 高效 | 低效 |

### 1.3 Spark生态系统

```
Spark Core (核心引擎)
    ↓
├── Spark SQL (结构化数据处理)
├── Spark Streaming (流处理)
├── MLlib (机器学习)
└── GraphX (图计算)
```

### 1.4 应用场景

- **数据分析**: 交互式数据查询和分析
- **ETL处理**: 数据清洗、转换、加载
- **机器学习**: 大规模机器学习模型训练
- **实时计算**: 实时数据流处理
- **图计算**: 社交网络分析、推荐系统

## 🔥 核心特性 (重点)

### 2.1 RDD (Resilient Distributed Dataset) 🔥

RDD是Spark最基本的数据抽象，代表一个不可变的、可分区的、可并行计算的数据集合。

#### 2.1.1 RDD特性

**核心特性**:
1. **弹性**: 数据丢失可以自动恢复
2. **分布式**: 数据分布在集群的多个节点
3. **不可变**: 一旦创建就不能修改
4. **可分区**: 数据被切分成多个分区并行处理
5. **可缓存**: 可以将数据缓存在内存中

**RDD的五大属性**:
```java
1. 分区列表 (List of Partitions)
2. 计算函数 (Compute Function)
3. 依赖关系 (Dependencies)
4. 分区器 (Partitioner) - 可选
5. 优先位置 (Preferred Locations) - 可选
```

#### 2.1.2 RDD创建方式

**方式1: 并行化集合**
```java
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.SparkConf;

import java.util.Arrays;
import java.util.List;

/**
 * RDD创建示例
 * 
 * @author erik.zhou
 */
public class RDDCreationExample {
    
    public static void main(String[] args) {
        SparkConf conf = new SparkConf()
            .setAppName("RDD Creation")
            .setMaster("local[*]");
        JavaSparkContext sc = new JavaSparkContext(conf);
        
        // 从集合创建RDD
        List<Integer> data = Arrays.asList(1, 2, 3, 4, 5);
        JavaRDD<Integer> rdd = sc.parallelize(data);
        
        System.out.println("Count: " + rdd.count());
        
        sc.close();
    }
}
```

**方式2: 读取外部数据**
```java
// 读取文本文件
JavaRDD<String> textRDD = sc.textFile("hdfs://localhost:9000/data/input.txt");

// 读取多个文件
JavaRDD<String> multiRDD = sc.textFile("hdfs://localhost:9000/data/*.txt");

// 读取整个目录
JavaRDD<String> dirRDD = sc.wholeTextFiles("hdfs://localhost:9000/data/");
```

#### 2.1.3 RDD转换操作 (Transformation) (⚠️ 难点)

转换操作是惰性求值的，只有遇到行动操作才会真正执行。

**常用转换操作**:

```java
/**
 * RDD转换操作示例
 * 
 * @author erik.zhou
 */
public class RDDTransformationExample {
    
    public static void main(String[] args) {
        SparkConf conf = new SparkConf().setAppName("RDD Transformation").setMaster("local[*]");
        JavaSparkContext sc = new JavaSparkContext(conf);
        
        List<Integer> data = Arrays.asList(1, 2, 3, 4, 5);
        JavaRDD<Integer> rdd = sc.parallelize(data);
        
        // map: 对每个元素应用函数
        JavaRDD<Integer> squaredRDD = rdd.map(x -> x * x);
        
        // filter: 过滤元素
        JavaRDD<Integer> evenRDD = rdd.filter(x -> x % 2 == 0);
        
        // flatMap: 将每个元素映射为多个元素
        JavaRDD<String> words = sc.parallelize(Arrays.asList("hello world", "spark java"));
        JavaRDD<String> wordsRDD = words.flatMap(line -> Arrays.asList(line.split(" ")).iterator());
        
        // distinct: 去重
        JavaRDD<Integer> distinctRDD = rdd.distinct();
        
        // union: 合并两个RDD
        JavaRDD<Integer> unionRDD = rdd.union(squaredRDD);
        
        // intersection: 交集
        JavaRDD<Integer> intersectionRDD = rdd.intersection(evenRDD);
        
        // subtract: 差集
        JavaRDD<Integer> subtractRDD = rdd.subtract(evenRDD);
        
        // groupBy: 分组
        JavaPairRDD<Integer, Iterable<Integer>> groupedRDD = rdd.groupBy(x -> x % 2);
        
        sc.close();
    }
}
```

**键值对RDD操作**:
```java
import org.apache.spark.api.java.JavaPairRDD;
import scala.Tuple2;

/**
 * PairRDD操作示例
 * 
 * @author erik.zhou
 */
public class PairRDDExample {
    
    public static void main(String[] args) {
        SparkConf conf = new SparkConf().setAppName("PairRDD").setMaster("local[*]");
        JavaSparkContext sc = new JavaSparkContext(conf);
        
        List<Tuple2<String, Integer>> data = Arrays.asList(
            new Tuple2<>("apple", 3),
            new Tuple2<>("banana", 2),
            new Tuple2<>("apple", 5)
        );
        JavaPairRDD<String, Integer> pairRDD = sc.parallelizePairs(data);
        
        // reduceByKey: 按key聚合
        JavaPairRDD<String, Integer> sumRDD = pairRDD.reduceByKey((a, b) -> a + b);
        
        // groupByKey: 按key分组
        JavaPairRDD<String, Iterable<Integer>> groupedRDD = pairRDD.groupByKey();
        
        // mapValues: 只对value进行map操作
        JavaPairRDD<String, Integer> doubledRDD = pairRDD.mapValues(v -> v * 2);
        
        // sortByKey: 按key排序
        JavaPairRDD<String, Integer> sortedRDD = pairRDD.sortByKey();
        
        // join: 连接两个PairRDD
        JavaPairRDD<String, Integer> otherRDD = sc.parallelizePairs(
            Arrays.asList(new Tuple2<>("apple", 10), new Tuple2<>("orange", 8))
        );
        JavaPairRDD<String, Tuple2<Integer, Integer>> joinedRDD = pairRDD.join(otherRDD);
        
        sc.close();
    }
}
```

#### 2.1.4 RDD行动操作 (Action)

行动操作会触发实际的计算并返回结果。

```java
/**
 * RDD行动操作示例
 * 
 * @author erik.zhou
 */
public class RDDActionExample {
    
    public static void main(String[] args) {
        SparkConf conf = new SparkConf().setAppName("RDD Action").setMaster("local[*]");
        JavaSparkContext sc = new JavaSparkContext(conf);
        
        List<Integer> data = Arrays.asList(1, 2, 3, 4, 5);
        JavaRDD<Integer> rdd = sc.parallelize(data);
        
        // count: 计数
        long count = rdd.count();
        System.out.println("Count: " + count);
        
        // collect: 收集所有元素到Driver
        List<Integer> result = rdd.collect();
        System.out.println("Result: " + result);
        
        // first: 获取第一个元素
        Integer first = rdd.first();
        System.out.println("First: " + first);
        
        // take: 获取前n个元素
        List<Integer> topN = rdd.take(3);
        System.out.println("Top 3: " + topN);
        
        // reduce: 聚合所有元素
        Integer sum = rdd.reduce((a, b) -> a + b);
        System.out.println("Sum: " + sum);
        
        // foreach: 对每个元素执行操作
        rdd.foreach(x -> System.out.println("Element: " + x));
        
        // saveAsTextFile: 保存到文件
        rdd.saveAsTextFile("hdfs://localhost:9000/output");
        
        sc.close();
    }
}
```

### 2.2 DataFrame和Dataset 🔥

DataFrame是Spark SQL的核心抽象，是一个分布式的、按列组织的数据集合，类似于关系数据库的表。

#### 2.2.1 DataFrame vs RDD

| 特性 | DataFrame | RDD |
|------|-----------|-----|
| 数据结构 | 结构化数据 | 非结构化数据 |
| 优化 | Catalyst优化器 | 无优化 |
| 性能 | 更快 | 较慢 |
| API | 声明式 | 命令式 |
| 类型安全 | 运行时检查 | 编译时检查 |

#### 2.2.2 创建DataFrame

```java
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

/**
 * DataFrame创建示例
 * 
 * @author erik.zhou
 */
public class DataFrameCreationExample {
    
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
            .appName("DataFrame Creation")
            .master("local[*]")
            .getOrCreate();
        
        // 方式1: 从JSON文件创建
        Dataset<Row> df1 = spark.read().json("data/users.json");
        
        // 方式2: 从CSV文件创建
        Dataset<Row> df2 = spark.read()
            .option("header", "true")
            .option("inferSchema", "true")
            .csv("data/users.csv");
        
        // 方式3: 从Parquet文件创建
        Dataset<Row> df3 = spark.read().parquet("data/users.parquet");
        
        // 方式4: 从JDBC数据库创建
        Dataset<Row> df4 = spark.read()
            .format("jdbc")
            .option("url", "jdbc:mysql://localhost:3306/test")
            .option("dbtable", "users")
            .option("user", "root")
            .option("password", "password")
            .load();
        
        // 方式5: 从RDD创建
        JavaRDD<String> rdd = spark.sparkContext()
            .textFile("data/users.txt", 1)
            .toJavaRDD();
        Dataset<Row> df5 = spark.read().json(rdd);
        
        spark.stop();
    }
}
```

#### 2.2.3 DataFrame操作

```java
import org.apache.spark.sql.functions;
import static org.apache.spark.sql.functions.*;

/**
 * DataFrame操作示例
 * 
 * @author erik.zhou
 */
public class DataFrameOperationExample {
    
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
            .appName("DataFrame Operations")
            .master("local[*]")
            .getOrCreate();
        
        Dataset<Row> df = spark.read()
            .option("header", "true")
            .option("inferSchema", "true")
            .csv("data/users.csv");
        
        // 查看数据
        df.show();
        df.printSchema();
        
        // 选择列
        df.select("name", "age").show();
        
        // 过滤
        df.filter(col("age").gt(18)).show();
        df.where("age > 18").show();
        
        // 分组聚合
        df.groupBy("gender").count().show();
        df.groupBy("gender").agg(avg("age"), max("salary")).show();
        
        // 排序
        df.orderBy(col("age").desc()).show();
        
        // 添加列
        Dataset<Row> df2 = df.withColumn("age_plus_10", col("age").plus(10));
        
        // 重命名列
        Dataset<Row> df3 = df.withColumnRenamed("name", "user_name");
        
        // 删除列
        Dataset<Row> df4 = df.drop("salary");
        
        // 去重
        df.distinct().show();
        df.dropDuplicates("name").show();
        
        // 连接
        Dataset<Row> orders = spark.read().json("data/orders.json");
        df.join(orders, df.col("id").equalTo(orders.col("user_id"))).show();
        
        spark.stop();
    }
}
```

### 2.3 Spark SQL 🔥

Spark SQL允许使用SQL语句查询结构化数据。

#### 2.3.1 SQL查询

```java
/**
 * Spark SQL示例
 * 
 * @author erik.zhou
 */
public class SparkSQLExample {
    
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
            .appName("Spark SQL")
            .master("local[*]")
            .getOrCreate();
        
        Dataset<Row> df = spark.read().json("data/users.json");
        
        // 注册临时视图
        df.createOrReplaceTempView("users");
        
        // 执行SQL查询
        Dataset<Row> result = spark.sql(
            "SELECT name, age FROM users WHERE age > 18 ORDER BY age DESC"
        );
        result.show();
        
        // 复杂查询
        Dataset<Row> result2 = spark.sql(
            "SELECT gender, COUNT(*) as count, AVG(age) as avg_age " +
            "FROM users " +
            "GROUP BY gender " +
            "HAVING count > 10"
        );
        result2.show();
        
        // 全局临时视图 (跨Session)
        df.createGlobalTempView("global_users");
        spark.sql("SELECT * FROM global_temp.global_users").show();
        
        spark.stop();
    }
}
```

### 2.4 内存计算原理 (⚠️ 难点)

Spark的高性能源于其内存计算模型。

#### 2.4.1 DAG执行引擎

Spark将作业转换为DAG (有向无环图)，然后优化执行计划。

```
Job (作业)
  ↓
Stage (阶段) - 根据Shuffle划分
  ↓
Task (任务) - 每个分区一个Task
```

**执行流程**:
```
1. 构建RDD的Lineage (血统)
2. 根据宽依赖划分Stage
3. 将Stage划分为Task
4. 调度Task到Executor执行
5. 收集结果
```

**依赖关系**:
- **窄依赖**: 父RDD的一个分区最多被子RDD的一个分区使用 (如map、filter)
- **宽依赖**: 父RDD的一个分区被子RDD的多个分区使用 (如groupByKey、reduceByKey)

#### 2.4.2 缓存机制

```java
/**
 * RDD缓存示例
 * 
 * @author erik.zhou
 */
public class CacheExample {
    
    public static void main(String[] args) {
        SparkConf conf = new SparkConf().setAppName("Cache").setMaster("local[*]");
        JavaSparkContext sc = new JavaSparkContext(conf);
        
        JavaRDD<String> rdd = sc.textFile("data/large_file.txt");
        
        // 缓存到内存
        rdd.cache();  // 等价于 rdd.persist(StorageLevel.MEMORY_ONLY)
        
        // 多次使用缓存的RDD
        long count1 = rdd.count();  // 第一次计算并缓存
        long count2 = rdd.count();  // 直接从缓存读取
        
        // 不同的存储级别
        rdd.persist(StorageLevel.MEMORY_AND_DISK());  // 内存+磁盘
        rdd.persist(StorageLevel.MEMORY_ONLY_SER());  // 序列化存储
        rdd.persist(StorageLevel.DISK_ONLY());        // 仅磁盘
        
        // 释放缓存
        rdd.unpersist();
        
        sc.close();
    }
}
```

**存储级别选择**:
- `MEMORY_ONLY`: 默认，性能最好但可能OOM
- `MEMORY_AND_DISK`: 内存不足时溢写到磁盘
- `MEMORY_ONLY_SER`: 序列化存储，节省内存但增加CPU开销
- `DISK_ONLY`: 仅磁盘，性能最差但最可靠

## 💻 实战应用

### 3.1 WordCount完整示例

使用Spark实现经典的WordCount程序。

```java
import org.apache.spark.api.java.JavaPairRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.SparkConf;
import scala.Tuple2;

import java.util.Arrays;

/**
 * Spark WordCount示例
 * 
 * @author erik.zhou
 */
public class SparkWordCount {
    
    public static void main(String[] args) {
        if (args.length < 2) {
            System.err.println("Usage: SparkWordCount <input> <output>");
            System.exit(1);
        }
        
        // 创建SparkConf
        SparkConf conf = new SparkConf()
            .setAppName("Spark WordCount");
        
        // 创建JavaSparkContext
        JavaSparkContext sc = new JavaSparkContext(conf);
        
        // 读取输入文件
        JavaRDD<String> lines = sc.textFile(args[0]);
        
        // 分词
        JavaRDD<String> words = lines.flatMap(
            line -> Arrays.asList(line.split("\\s+")).iterator()
        );
        
        // 转换为(word, 1)
        JavaPairRDD<String, Integer> pairs = words.mapToPair(
            word -> new Tuple2<>(word, 1)
        );
        
        // 按key聚合
        JavaPairRDD<String, Integer> wordCounts = pairs.reduceByKey(
            (a, b) -> a + b
        );
        
        // 保存结果
        wordCounts.saveAsTextFile(args[1]);
        
        sc.close();
    }
}
```

**使用DataFrame实现**:
```java
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import static org.apache.spark.sql.functions.*;

/**
 * DataFrame WordCount示例
 * 
 * @author erik.zhou
 */
public class DataFrameWordCount {
    
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
            .appName("DataFrame WordCount")
            .getOrCreate();
        
        // 读取文件
        Dataset<Row> lines = spark.read().text(args[0]);
        
        // 分词并统计
        Dataset<Row> wordCounts = lines
            .select(explode(split(col("value"), "\\s+")).as("word"))
            .groupBy("word")
            .count()
            .orderBy(col("count").desc());
        
        // 保存结果
        wordCounts.write().csv(args[1]);
        
        spark.stop();
    }
}
```

### 3.2 日志分析实战

分析Web服务器日志，统计访问量、错误率等指标。

```java
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import static org.apache.spark.sql.functions.*;

import java.io.Serializable;

/**
 * 日志分析示例
 * 
 * @author erik.zhou
 */
public class LogAnalysis {
    
    // 日志记录类
    public static class LogRecord implements Serializable {
        private String ip;
        private String timestamp;
        private String method;
        private String url;
        private int statusCode;
        private long responseSize;
        
        // 构造函数、getter、setter省略
    }
    
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
            .appName("Log Analysis")
            .getOrCreate();
        
        // 读取日志文件
        JavaRDD<String> logLines = spark.sparkContext()
            .textFile("hdfs://localhost:9000/logs/*.log", 1)
            .toJavaRDD();
        
        // 解析日志
        JavaRDD<LogRecord> logs = logLines.map(line -> parseLog(line))
            .filter(log -> log != null);
        
        // 转换为DataFrame
        Dataset<Row> logDF = spark.createDataFrame(logs, LogRecord.class);
        
        // 统计分析
        
        // 1. 按状态码统计
        logDF.groupBy("statusCode")
            .count()
            .orderBy(col("count").desc())
            .show();
        
        // 2. 统计错误率
        long totalCount = logDF.count();
        long errorCount = logDF.filter(col("statusCode").geq(400)).count();
        double errorRate = (double) errorCount / totalCount * 100;
        System.out.println("Error Rate: " + errorRate + "%");
        
        // 3. 访问量最高的URL
        logDF.groupBy("url")
            .count()
            .orderBy(col("count").desc())
            .limit(10)
            .show();
        
        // 4. 按小时统计访问量
        logDF.withColumn("hour", hour(col("timestamp")))
            .groupBy("hour")
            .count()
            .orderBy("hour")
            .show();
        
        // 5. 平均响应大小
        logDF.agg(avg("responseSize").as("avg_size")).show();
        
        spark.stop();
    }
    
    private static LogRecord parseLog(String line) {
        // 解析日志行的逻辑
        // 这里简化处理
        return new LogRecord();
    }
}
```

### 3.3 与Spring Boot集成

在Spring Boot项目中集成Spark进行数据处理。

**Maven依赖**:
```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.12</artifactId>
    <version>3.5.0</version>
</dependency>
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-sql_2.12</artifactId>
    <version>3.5.0</version>
</dependency>
```

**配置类**:
```java
import org.apache.spark.SparkConf;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.sql.SparkSession;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Spark配置类
 * 
 * @author erik.zhou
 */
@Configuration
public class SparkConfig {
    
    @Value("${spark.app.name}")
    private String appName;
    
    @Value("${spark.master}")
    private String master;
    
    @Bean
    public SparkConf sparkConf() {
        return new SparkConf()
            .setAppName(appName)
            .setMaster(master)
            .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer");
    }
    
    @Bean
    public JavaSparkContext javaSparkContext() {
        return new JavaSparkContext(sparkConf());
    }
    
    @Bean
    public SparkSession sparkSession() {
        return SparkSession.builder()
            .config(sparkConf())
            .getOrCreate();
    }
}
```

**Service层**:
```java
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.api.java.JavaSparkContext;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

/**
 * Spark数据处理服务
 * 
 * @author erik.zhou
 */
@Service
public class SparkDataService {
    
    @Autowired
    private JavaSparkContext sparkContext;
    
    @Autowired
    private SparkSession sparkSession;
    
    /**
     * 处理文本数据
     */
    public List<String> processTextData(String filePath) {
        JavaRDD<String> lines = sparkContext.textFile(filePath);
        JavaRDD<String> words = lines.flatMap(
            line -> java.util.Arrays.asList(line.split("\\s+")).iterator()
        );
        return words.take(100);
    }
    
    /**
     * 查询结构化数据
     */
    public List<Row> queryData(String tableName, String condition) {
        Dataset<Row> df = sparkSession.table(tableName);
        Dataset<Row> result = df.filter(condition);
        return result.collectAsList();
    }
    
    /**
     * 聚合统计
     */
    public long countRecords(String filePath) {
        JavaRDD<String> rdd = sparkContext.textFile(filePath);
        return rdd.count();
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化技巧

#### 4.1.1 避免Shuffle操作

Shuffle是Spark中最耗时的操作，应尽量避免。

**优化前**:
```java
// groupByKey会产生大量Shuffle
JavaPairRDD<String, Iterable<Integer>> grouped = pairRDD.groupByKey();
JavaPairRDD<String, Integer> result = grouped.mapValues(values -> {
    int sum = 0;
    for (int v : values) {
        sum += v;
    }
    return sum;
});
```

**优化后**:
```java
// reduceByKey在Map端先聚合，减少Shuffle数据量
JavaPairRDD<String, Integer> result = pairRDD.reduceByKey((a, b) -> a + b);
```

#### 4.1.2 合理使用缓存

对于多次使用的RDD，应该缓存到内存中。

```java
/**
 * 缓存优化示例
 * 
 * @author erik.zhou
 */
public class CacheOptimization {
    
    public static void main(String[] args) {
        JavaSparkContext sc = new JavaSparkContext(new SparkConf());
        
        JavaRDD<String> data = sc.textFile("large_file.txt");
        
        // 对多次使用的RDD进行缓存
        JavaRDD<String> filteredData = data.filter(line -> line.contains("ERROR"));
        filteredData.cache();  // 缓存
        
        // 多次使用
        long errorCount = filteredData.count();
        List<String> samples = filteredData.take(10);
        filteredData.saveAsTextFile("errors.txt");
        
        sc.close();
    }
}
```

#### 4.1.3 分区优化

合理设置分区数量可以提高并行度。

```java
/**
 * 分区优化示例
 * 
 * @author erik.zhou
 */
public class PartitionOptimization {
    
    public static void main(String[] args) {
        JavaSparkContext sc = new JavaSparkContext(new SparkConf());
        
        // 读取数据时指定分区数
        JavaRDD<String> rdd = sc.textFile("data.txt", 100);
        
        // 重新分区
        JavaRDD<String> repartitioned = rdd.repartition(200);
        
        // 减少分区数 (不会产生Shuffle)
        JavaRDD<String> coalesced = rdd.coalesce(50);
        
        // 自定义分区器
        JavaPairRDD<String, Integer> pairRDD = rdd.mapToPair(
            line -> new Tuple2<>(line.substring(0, 1), 1)
        );
        JavaPairRDD<String, Integer> partitioned = pairRDD.partitionBy(
            new org.apache.spark.HashPartitioner(100)
        );
        
        sc.close();
    }
}
```

**分区数量建议**:
- 分区数 = CPU核心数 × 2~3
- 每个分区大小: 128MB - 1GB
- 避免分区过多或过少

#### 4.1.4 广播变量

对于小数据集，使用广播变量避免重复传输。

```java
import org.apache.spark.broadcast.Broadcast;

/**
 * 广播变量示例
 * 
 * @author erik.zhou
 */
public class BroadcastExample {
    
    public static void main(String[] args) {
        JavaSparkContext sc = new JavaSparkContext(new SparkConf());
        
        // 小数据集
        Map<String, String> dict = new HashMap<>();
        dict.put("A", "Apple");
        dict.put("B", "Banana");
        
        // 广播变量
        Broadcast<Map<String, String>> broadcastDict = sc.broadcast(dict);
        
        JavaRDD<String> codes = sc.parallelize(Arrays.asList("A", "B", "C"));
        
        // 在Task中使用广播变量
        JavaRDD<String> names = codes.map(code -> {
            Map<String, String> localDict = broadcastDict.value();
            return localDict.getOrDefault(code, "Unknown");
        });
        
        names.collect().forEach(System.out::println);
        
        // 销毁广播变量
        broadcastDict.unpersist();
        
        sc.close();
    }
}
```

#### 4.1.5 数据倾斜处理

数据倾斜会导致某些Task执行时间过长。

**解决方案1: 加盐**
```java
/**
 * 数据倾斜处理 - 加盐法
 * 
 * @author erik.zhou
 */
public class DataSkewSolution {
    
    public static void main(String[] args) {
        JavaSparkContext sc = new JavaSparkContext(new SparkConf());
        
        JavaPairRDD<String, Integer> skewedRDD = sc.parallelizePairs(
            Arrays.asList(
                new Tuple2<>("hot_key", 1),
                new Tuple2<>("hot_key", 2),
                // ... 大量hot_key
                new Tuple2<>("normal_key", 1)
            )
        );
        
        // 加盐: 给key添加随机前缀
        JavaPairRDD<String, Integer> saltedRDD = skewedRDD.mapToPair(
            tuple -> new Tuple2<>(
                tuple._1 + "_" + new Random().nextInt(10),
                tuple._2
            )
        );
        
        // 聚合
        JavaPairRDD<String, Integer> aggregated = saltedRDD.reduceByKey((a, b) -> a + b);
        
        // 去盐: 移除随机前缀
        JavaPairRDD<String, Integer> result = aggregated.mapToPair(
            tuple -> new Tuple2<>(
                tuple._1.split("_")[0],
                tuple._2
            )
        ).reduceByKey((a, b) -> a + b);
        
        sc.close();
    }
}
```

### 4.2 内存管理

#### 4.2.1 内存配置

**关键参数**:
```bash
# Executor内存
spark.executor.memory=4g

# Driver内存
spark.driver.memory=2g

# 堆外内存
spark.memory.offHeap.enabled=true
spark.memory.offHeap.size=2g

# 内存分配比例
spark.memory.fraction=0.6  # 执行和存储内存占比
spark.memory.storageFraction=0.5  # 存储内存占比
```

#### 4.2.2 序列化优化

使用Kryo序列化器提高性能。

```java
SparkConf conf = new SparkConf()
    .set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
    .registerKryoClasses(new Class[]{
        MyClass1.class,
        MyClass2.class
    });
```

### 4.3 监控与调优

#### 4.3.1 Spark UI

访问 `http://driver-node:4040` 查看：
- Jobs: 作业执行情况
- Stages: 阶段详情和DAG
- Storage: RDD缓存情况
- Environment: 配置信息
- Executors: Executor状态

#### 4.3.2 关键指标

**性能指标**:
- Task执行时间
- Shuffle读写量
- GC时间
- 内存使用率

**优化目标**:
- 减少Shuffle数据量
- 降低GC频率
- 提高CPU利用率
- 避免数据倾斜

## ⚠️ 常见陷阱

### 陷阱1: 过度使用collect()

**问题**: collect()会将所有数据拉到Driver，可能导致OOM

**解决方案**:
```java
// 错误: 收集大量数据
List<String> allData = largeRDD.collect();  // 可能OOM

// 正确: 只收集必要的数据
List<String> sample = largeRDD.take(100);
long count = largeRDD.count();
```

### 陷阱2: 在Executor中创建SparkContext

**问题**: SparkContext只能在Driver中创建

**解决方案**:
```java
// 错误: 在map中创建SparkContext
rdd.map(x -> {
    SparkContext sc = new SparkContext();  // 错误!
    return x;
});

// 正确: 使用广播变量或外部资源
```

### 陷阱3: 忘记关闭资源

**问题**: 不关闭SparkContext会导致资源泄漏

**解决方案**:
```java
JavaSparkContext sc = new JavaSparkContext(conf);
try {
    // 处理逻辑
} finally {
    sc.close();  // 确保关闭
}
```

### 陷阱4: 使用非序列化对象

**问题**: 在RDD操作中使用非序列化对象会失败

**解决方案**:
```java
// 错误: 使用非序列化对象
class MyClass {  // 没有实现Serializable
    private int value;
}

// 正确: 实现Serializable
class MyClass implements Serializable {
    private int value;
}
```

## ❓ 常见问题

### Q1: Spark适合什么样的场景？

**A**: Spark适合以下场景：
- **迭代计算**: 机器学习、图计算等需要多次迭代的场景
- **交互式查询**: 数据探索和分析
- **实时流处理**: 准实时数据处理 (Spark Streaming)
- **批处理**: 替代Hadoop MapReduce的批处理任务
- **多语言支持**: 需要使用Java、Scala、Python、R的场景

**不适合的场景**:
- 超低延迟要求 (毫秒级，使用Flink)
- 纯流处理 (使用Flink或Kafka Streams)
- 小数据量 (单机处理更高效)

### Q2: RDD、DataFrame、Dataset有什么区别？

**A**: 三者对比：

| 特性 | RDD | DataFrame | Dataset |
|------|-----|-----------|---------|
| 类型安全 | 编译时 | 运行时 | 编译时 |
| 优化 | 无 | Catalyst优化 | Catalyst优化 |
| API | 函数式 | 声明式 | 两者结合 |
| 性能 | 较慢 | 快 | 最快 |
| 适用场景 | 非结构化数据 | 结构化数据 | 强类型数据 |

**选择建议**:
- 结构化数据优先使用DataFrame/Dataset
- 需要类型安全使用Dataset
- 非结构化数据或需要底层控制使用RDD

### Q3: 如何选择Spark的部署模式？

**A**: Spark支持多种部署模式：

**1. Local模式**:
- 适用场景: 开发测试
- 配置: `setMaster("local[*]")`

**2. Standalone模式**:
- 适用场景: 小规模集群
- 优点: 部署简单
- 缺点: 资源管理功能弱

**3. YARN模式**:
- 适用场景: 已有Hadoop集群
- 优点: 资源管理成熟
- 配置: `setMaster("yarn")`

**4. Kubernetes模式**:
- 适用场景: 云原生环境
- 优点: 容器化部署
- 配置: `setMaster("k8s://...")`

**推荐**: 生产环境优先选择YARN或Kubernetes。

### Q4: Spark和Flink如何选择？

**A**: 两者对比：

| 特性 | Spark | Flink |
|------|-------|-------|
| 计算模型 | 微批处理 | 真流处理 |
| 延迟 | 秒级 | 毫秒级 |
| 吞吐量 | 高 | 较高 |
| 状态管理 | 较弱 | 强 |
| 生态 | 成熟 | 发展中 |
| 学习曲线 | 平缓 | 陡峭 |

**选择建议**:
- **批处理**: Spark
- **实时流处理**: Flink
- **准实时 + 批处理**: Spark
- **复杂事件处理**: Flink
- **机器学习**: Spark (MLlib更成熟)

### Q5: 如何调试Spark程序？

**A**: 调试方法：

**1. 本地模式调试**:
```java
SparkConf conf = new SparkConf()
    .setMaster("local[*]")  // 本地模式
    .setAppName("Debug");
```

**2. 使用日志**:
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MySparkApp {
    private static final Logger logger = LoggerFactory.getLogger(MySparkApp.class);
    
    public static void main(String[] args) {
        logger.info("Starting Spark application");
        // ...
    }
}
```

**3. 查看Spark UI**:
- 访问 `http://driver:4040`
- 查看Stage的DAG图
- 检查Task执行时间
- 查看Shuffle数据量

**4. 采样调试**:
```java
// 使用sample()减少数据量
JavaRDD<String> sample = largeRDD.sample(false, 0.01);
sample.collect().forEach(System.out::println);
```

**5. 使用toDebugString()**:
```java
// 查看RDD的血统信息
System.out.println(rdd.toDebugString());
```

### Q6: Spark在Java后端项目中的典型应用？

**A**: 典型应用场景：

**1. 离线报表生成**:
```java
/**
 * 每日报表生成
 * 
 * @author erik.zhou
 */
@Service
public class ReportService {
    
    @Autowired
    private SparkSession spark;
    
    @Scheduled(cron = "0 0 2 * * ?")  // 每天凌晨2点执行
    public void generateDailyReport() {
        Dataset<Row> orders = spark.read()
            .jdbc("jdbc:mysql://localhost:3306/db", "orders", ...);
        
        Dataset<Row> report = orders
            .filter(col("order_date").equalTo(yesterday()))
            .groupBy("category")
            .agg(
                sum("amount").as("total_amount"),
                count("*").as("order_count")
            );
        
        report.write().mode("overwrite").parquet("hdfs://reports/daily");
    }
}
```

**2. 用户行为分析**:
```java
/**
 * 用户行为分析
 * 
 * @author erik.zhou
 */
@Service
public class UserBehaviorService {
    
    @Autowired
    private SparkSession spark;
    
    public Map<String, Long> analyzeUserBehavior(String userId) {
        Dataset<Row> logs = spark.read().parquet("hdfs://logs/user_behavior");
        
        Dataset<Row> userLogs = logs.filter(col("user_id").equalTo(userId));
        
        Map<String, Long> result = new HashMap<>();
        result.put("pageViews", userLogs.filter(col("action").equalTo("view")).count());
        result.put("clicks", userLogs.filter(col("action").equalTo("click")).count());
        result.put("purchases", userLogs.filter(col("action").equalTo("purchase")).count());
        
        return result;
    }
}
```

**3. 数据ETL**:
```java
/**
 * 数据ETL处理
 * 
 * @author erik.zhou
 */
@Service
public class ETLService {
    
    @Autowired
    private SparkSession spark;
    
    public void etlProcess() {
        // Extract: 从多个数据源读取
        Dataset<Row> mysqlData = spark.read().jdbc(...);
        Dataset<Row> hdfsData = spark.read().parquet("hdfs://raw_data");
        
        // Transform: 数据清洗和转换
        Dataset<Row> cleaned = mysqlData
            .join(hdfsData, "id")
            .filter(col("status").equalTo("valid"))
            .withColumn("processed_time", current_timestamp())
            .select("id", "name", "value", "processed_time");
        
        // Load: 写入目标存储
        cleaned.write()
            .mode("append")
            .partitionBy("date")
            .parquet("hdfs://processed_data");
    }
}
```

**4. 推荐系统离线计算**:
```java
/**
 * 推荐系统离线计算
 * 
 * @author erik.zhou
 */
@Service
public class RecommendationService {
    
    @Autowired
    private SparkSession spark;
    
    public void calculateRecommendations() {
        // 读取用户行为数据
        Dataset<Row> userBehavior = spark.read().parquet("hdfs://user_behavior");
        
        // 计算物品相似度
        Dataset<Row> itemSimilarity = userBehavior
            .groupBy("item_id")
            .agg(collect_list("user_id").as("users"))
            .crossJoin(userBehavior.groupBy("item_id").agg(collect_list("user_id").as("users")))
            .filter(col("item_id").notEqual(col("item_id")))
            // 计算Jaccard相似度
            .withColumn("similarity", calculateJaccardSimilarity(col("users"), col("users")));
        
        // 保存推荐结果
        itemSimilarity.write().mode("overwrite").parquet("hdfs://recommendations");
    }
}
```

### Q7: 如何处理Spark的内存溢出问题？

**A**: 解决方案：

**1. 增加Executor内存**:
```bash
spark-submit \
  --executor-memory 8g \
  --driver-memory 4g \
  myapp.jar
```

**2. 调整内存分配比例**:
```bash
--conf spark.memory.fraction=0.6 \
--conf spark.memory.storageFraction=0.5
```

**3. 使用磁盘溢写**:
```java
// 使用MEMORY_AND_DISK存储级别
rdd.persist(StorageLevel.MEMORY_AND_DISK());
```

**4. 减少数据量**:
```java
// 过滤不需要的数据
JavaRDD<String> filtered = rdd.filter(line -> line.contains("important"));

// 使用sample采样
JavaRDD<String> sampled = rdd.sample(false, 0.1);
```

**5. 增加分区数**:
```java
// 增加分区数，减少每个分区的数据量
JavaRDD<String> repartitioned = rdd.repartition(200);
```

**6. 避免使用collect()**:
```java
// 错误: 收集所有数据
List<String> all = rdd.collect();  // OOM!

// 正确: 只收集必要的数据
List<String> sample = rdd.take(100);
rdd.saveAsTextFile("hdfs://output");  // 直接保存到HDFS
```

## 🔗 相关资源

### 官方文档
- [Apache Spark官网](https://spark.apache.org/)
- [Spark编程指南](https://spark.apache.org/docs/latest/programming-guide.html)
- [Spark SQL指南](https://spark.apache.org/docs/latest/sql-programming-guide.html)
- [Spark Streaming指南](https://spark.apache.org/docs/latest/streaming-programming-guide.html)
- [Spark MLlib指南](https://spark.apache.org/docs/latest/ml-guide.html)

### 推荐书籍
- 《Spark快速大数据分析》
- 《Spark高级数据分析》(第2版)
- 《深入理解Spark核心思想与源码分析》
- 《Spark性能优化指南》

### 在线资源
- [Databricks博客](https://databricks.com/blog)
- [Spark中文社区](http://spark.apache.org/zh-cn/)
- [Spark Summit视频](https://databricks.com/sparkaisummit)

### 相关技术
- **Hadoop**: 分布式存储和计算框架
- **Flink**: 流处理框架
- **Kafka**: 消息队列，常与Spark Streaming配合
- **Hive**: 数据仓库，可通过Spark SQL访问
- **HBase**: NoSQL数据库，可作为Spark数据源

### 开发工具
- **IntelliJ IDEA**: 推荐的Spark开发IDE
- **Zeppelin**: 交互式数据分析笔记本
- **Jupyter**: 支持PySpark的笔记本
- **Spark UI**: 内置的监控界面

## 📝 学习检查清单

- [ ] 理解Spark的核心架构和设计理念
- [ ] 掌握RDD的创建、转换和行动操作
- [ ] 理解RDD的依赖关系和DAG执行流程
- [ ] 掌握DataFrame和Dataset API
- [ ] 能够使用Spark SQL进行数据查询
- [ ] 理解Spark的内存计算原理
- [ ] 掌握Spark性能优化技巧
- [ ] 了解Spark的部署模式
- [ ] 能够在Java项目中集成Spark
- [ ] 掌握Spark程序的调试方法

---

**@author** erik.zhou
**最后更新**: 2024-01-04
**文档版本**: 1.0
**文档来源**: Apache Spark官方文档 (Context7)
