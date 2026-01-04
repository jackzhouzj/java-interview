# Elasticsearch 完整教程

## 📋 目录
- 技术概述
- 学习目标
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题

## 📚 技术概述
- **版本**: Elasticsearch 8.x
- **官方文档**: https://www.elastic.co/guide/
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: JSON基础、搜索引擎概念
- **文档来源**: Context7 - Elasticsearch Repository
- **更新时间**: 2024-12-31

## 🎯 学习目标
- [ ] 掌握Elasticsearch基础概念和架构
- [ ] 理解倒排索引原理
- [ ] 掌握Query DSL查询语法
- [ ] 理解聚合分析
- [ ] 掌握性能优化方法

## 📖 基础概念

### 1.1 什么是Elasticsearch
Elasticsearch是基于Lucene的分布式搜索和分析引擎，提供近实时搜索、全文检索、结构化搜索和分析功能。

### 1.2 核心概念
- **索引(Index)**: 文档的集合，类似数据库
- **文档(Document)**: 基本数据单元，JSON格式
- **字段(Field)**: 文档中的键值对
- **映射(Mapping)**: 定义文档结构和字段类型
- **分片(Shard)**: 索引的水平分割
- **副本(Replica)**: 分片的备份
- **倒排索引**: 核心数据结构，实现快速全文搜索

### 1.3 应用场景
- 全文搜索引擎
- 日志分析（ELK Stack）
- 实时数据分析
- 应用性能监控(APM)
- 安全分析

## 🔥 核心特性

### 2.1 倒排索引 (⚠️ 难点) 🔥

**倒排索引原理**
```
文档1: "Java is a programming language"
文档2: "Python is also a programming language"

倒排索引:
java -> [文档1]
python -> [文档2]
programming -> [文档1, 文档2]
language -> [文档1, 文档2]
```

**分词器(Analyzer)**
```json
// 创建自定义分词器
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": ["lowercase", "stop", "snowball"]
        }
      }
    }
  }
}

// 测试分词器
POST /my_index/_analyze
{
  "analyzer": "my_analyzer",
  "text": "The Quick Brown Fox Jumps"
}
```

### 2.2 映射(Mapping) 🔥

**创建映射**
```json
PUT /products
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text",
        "analyzer": "standard"
      },
      "description": {
        "type": "text",
        "analyzer": "english"
      },
      "price": {
        "type": "double"
      },
      "stock": {
        "type": "integer"
      },
      "category": {
        "type": "keyword"
      },
      "tags": {
        "type": "keyword"
      },
      "createTime": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "location": {
        "type": "geo_point"
      }
    }
  }
}
```

**字段类型**
- **text**: 全文搜索，会分词
- **keyword**: 精确匹配，不分词
- **数值类型**: long, integer, short, byte, double, float
- **日期类型**: date
- **布尔类型**: boolean
- **地理类型**: geo_point, geo_shape


### 2.3 Query DSL 🔥

**全文查询**
```json
// match查询（分词匹配）
GET /products/_search
{
  "query": {
    "match": {
      "name": "Java programming"
    }
  }
}

// multi_match查询（多字段匹配）
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "Java",
      "fields": ["name", "description"]
    }
  }
}

// match_phrase查询（短语匹配）
GET /products/_search
{
  "query": {
    "match_phrase": {
      "description": "programming language"
    }
  }
}
```

**精确查询**
```json
// term查询（精确匹配）
GET /products/_search
{
  "query": {
    "term": {
      "category": "electronics"
    }
  }
}

// terms查询（多值匹配）
GET /products/_search
{
  "query": {
    "terms": {
      "tags": ["java", "python", "golang"]
    }
  }
}

// range查询（范围查询）
GET /products/_search
{
  "query": {
    "range": {
      "price": {
        "gte": 100,
        "lte": 500
      }
    }
  }
}
```

**复合查询**
```json
// bool查询
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        { "match": { "name": "Java" } }
      ],
      "filter": [
        { "range": { "price": { "gte": 50 } } },
        { "term": { "category": "books" } }
      ],
      "should": [
        { "match": { "description": "advanced" } }
      ],
      "must_not": [
        { "term": { "status": "deleted" } }
      ]
    }
  }
}
```

### 2.4 聚合分析 (⚠️ 难点) 🔥

**指标聚合**
```json
// 统计聚合
GET /products/_search
{
  "size": 0,
  "aggs": {
    "avg_price": { "avg": { "field": "price" } },
    "max_price": { "max": { "field": "price" } },
    "min_price": { "min": { "field": "price" } },
    "sum_price": { "sum": { "field": "price" } },
    "stats_price": { "stats": { "field": "price" } }
  }
}
```

**桶聚合**
```json
// terms聚合（分组统计）
GET /products/_search
{
  "size": 0,
  "aggs": {
    "categories": {
      "terms": {
        "field": "category",
        "size": 10
      },
      "aggs": {
        "avg_price": {
          "avg": { "field": "price" }
        }
      }
    }
  }
}

// histogram聚合（直方图）
GET /products/_search
{
  "size": 0,
  "aggs": {
    "price_ranges": {
      "histogram": {
        "field": "price",
        "interval": 100
      }
    }
  }
}

// date_histogram聚合（时间直方图）
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "sales_over_time": {
      "date_histogram": {
        "field": "createTime",
        "calendar_interval": "day"
      },
      "aggs": {
        "total_amount": {
          "sum": { "field": "amount" }
        }
      }
    }
  }
}
```

### 2.5 评分机制 (⚠️ 难点)

**TF-IDF算法**
- TF (Term Frequency): 词频
- IDF (Inverse Document Frequency): 逆文档频率
- Score = TF * IDF

**BM25算法**（Elasticsearch 5.0+默认）
- 改进的TF-IDF算法
- 考虑文档长度归一化
- 更好的相关性评分

**自定义评分**
```json
GET /products/_search
{
  "query": {
    "function_score": {
      "query": { "match": { "name": "Java" } },
      "functions": [
        {
          "filter": { "term": { "category": "books" } },
          "weight": 2
        },
        {
          "field_value_factor": {
            "field": "sales",
            "factor": 0.1,
            "modifier": "log1p"
          }
        }
      ],
      "score_mode": "sum",
      "boost_mode": "multiply"
    }
  }
}
```


## 💻 实战应用

### 3.1 环境搭建

**Docker安装Elasticsearch**
```bash
# 拉取Elasticsearch镜像
docker pull elasticsearch:8.11.0

# 启动Elasticsearch容器
docker run -d \
  --name elasticsearch \
  -p 9200:9200 \
  -p 9300:9300 \
  -e "discovery.type=single-node" \
  -e "xpack.security.enabled=false" \
  -e "ES_JAVA_OPTS=-Xms512m -Xmx512m" \
  elasticsearch:8.11.0

# 验证安装
curl http://localhost:9200
```

### 3.2 Spring Boot集成

**添加依赖**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-elasticsearch</artifactId>
</dependency>
```

**配置文件**
```yaml
spring:
  elasticsearch:
    uris: http://localhost:9200
    connection-timeout: 3s
    socket-timeout: 5s
```

**实体类**
```java
@Document(indexName = "products")
public class Product {
    @Id
    private String id;
    
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String name;
    
    @Field(type = FieldType.Text, analyzer = "ik_max_word")
    private String description;
    
    @Field(type = FieldType.Double)
    private Double price;
    
    @Field(type = FieldType.Keyword)
    private String category;
    
    @Field(type = FieldType.Keyword)
    private List<String> tags;
    
    @Field(type = FieldType.Date, format = DateFormat.date_time)
    private LocalDateTime createTime;
    
    // getters and setters
}
```

**Repository接口**
```java
public interface ProductRepository extends ElasticsearchRepository<Product, String> {
    
    // 方法名查询
    List<Product> findByName(String name);
    List<Product> findByPriceBetween(Double min, Double max);
    List<Product> findByCategory(String category);
    
    // @Query注解
    @Query("{\"match\": {\"name\": \"?0\"}}")
    List<Product> searchByName(String name);
}
```

**Service实现**
```java
@Service
public class ProductSearchService {
    
    @Autowired
    private ElasticsearchRestTemplate elasticsearchTemplate;
    
    /**
     * 复杂搜索
     */
    public SearchHits<Product> search(String keyword, String category, 
                                      Double minPrice, Double maxPrice,
                                      int page, int size) {
        // 构建查询条件
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        
        if (keyword != null) {
            boolQuery.must(QueryBuilders.multiMatchQuery(keyword, "name", "description"));
        }
        
        if (category != null) {
            boolQuery.filter(QueryBuilders.termQuery("category", category));
        }
        
        if (minPrice != null || maxPrice != null) {
            RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("price");
            if (minPrice != null) rangeQuery.gte(minPrice);
            if (maxPrice != null) rangeQuery.lte(maxPrice);
            boolQuery.filter(rangeQuery);
        }
        
        // 构建查询
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
            .withQuery(boolQuery)
            .withPageable(PageRequest.of(page, size))
            .withSort(SortBuilders.scoreSort().order(SortOrder.DESC))
            .withHighlightFields(
                new HighlightBuilder.Field("name"),
                new HighlightBuilder.Field("description")
            )
            .build();
        
        return elasticsearchTemplate.search(searchQuery, Product.class);
    }
    
    /**
     * 聚合统计
     */
    public Map<String, Long> aggregateByCategory() {
        NativeSearchQuery searchQuery = new NativeSearchQueryBuilder()
            .addAggregation(
                AggregationBuilders.terms("categories")
                    .field("category")
                    .size(10)
            )
            .build();
        
        SearchHits<Product> searchHits = elasticsearchTemplate.search(
            searchQuery, Product.class);
        
        Map<String, Long> result = new HashMap<>();
        Aggregations aggregations = searchHits.getAggregations();
        if (aggregations != null) {
            Terms terms = aggregations.get("categories");
            for (Terms.Bucket bucket : terms.getBuckets()) {
                result.put(bucket.getKeyAsString(), bucket.getDocCount());
            }
        }
        return result;
    }
}
```

## ✨ 最佳实践

### 4.1 索引设计
- 合理设置分片数（小索引1个分片，大索引根据数据量）
- 设置副本数（至少1个副本保证高可用）
- 使用别名管理索引版本
- 定期清理过期数据

### 4.2 查询优化
- 使用filter代替query（filter有缓存）
- 避免深分页（使用scroll或search_after）
- 使用_source过滤返回字段
- 合理使用高亮功能

### 4.3 性能优化 (⚠️ 难点)

**批量操作**
```java
// 批量索引
BulkRequest bulkRequest = new BulkRequest();
for (Product product : products) {
    bulkRequest.add(new IndexRequest("products")
        .id(product.getId())
        .source(JSON.toJSONString(product), XContentType.JSON));
}
BulkResponse bulkResponse = client.bulk(bulkRequest, RequestOptions.DEFAULT);
```

**避免深分页**
```json
// ❌ 深分页性能差
GET /products/_search
{
  "from": 10000,
  "size": 10
}

// ✅ 使用search_after
GET /products/_search
{
  "size": 10,
  "query": { "match_all": {} },
  "search_after": [1463538857, "654323"],
  "sort": [
    { "createTime": "asc" },
    { "_id": "asc" }
  ]
}
```

## ❓ 常见问题

### Q1: Elasticsearch与传统数据库的区别？
A: 
- ES擅长全文搜索和分析，数据库擅长事务
- ES使用倒排索引，数据库使用B-Tree
- ES是分布式的，易于水平扩展
- ES不支持事务和JOIN

### Q2: 如何选择分片数？
A: 
- 小索引（<5GB）：1个分片
- 中等索引（5-50GB）：3-5个分片
- 大索引（>50GB）：根据数据量，每个分片20-50GB
- 分片数不宜过多，影响性能

### Q3: 如何优化搜索性能？
A: 
1. 合理设计映射和分词器
2. 使用filter代替query
3. 避免深分页
4. 使用routing减少查询分片数
5. 增加副本数提高查询并发

## 🔗 相关资源
- 官方文档：https://www.elastic.co/guide/
- Kibana：可视化工具
- Logstash：数据采集工具

## 📝 学习检查清单
- [ ] 理解倒排索引原理
- [ ] 掌握Query DSL语法
- [ ] 掌握聚合分析
- [ ] 了解评分机制
- [ ] 掌握性能优化方法

---

**@author erik.zhou**  
**文档版本**: v1.0  
**最后更新**: 2024-12-31
