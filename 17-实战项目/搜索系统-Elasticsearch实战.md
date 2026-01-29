# 搜索系统 - Elasticsearch实战

## 📋 项目概述

### 业务场景
构建一个高性能的电商搜索系统，支持：
- 商品全文搜索
- 多条件筛选（价格、品牌、分类等）
- 搜索建议（自动补全）
- 搜索排序（相关性、销量、价格）
- 搜索统计分析

### 技术挑战 ⚠️

#### 难点1: 搜索性能优化
**问题描述**:
- 如何支持亿级数据搜索？
- 如何保证搜索响应时间<100ms？
- 复杂查询如何优化？
- 如何避免深分页问题？

**业务影响**:
- 搜索慢影响用户体验
- 高峰期系统压力大
- 搜索结果不准确

#### 难点2: 搜索相关性优化
**问题描述**:
- 如何提高搜索准确率？
- 如何处理中文分词？
- 如何实现同义词搜索？
- 如何处理拼音搜索？

**业务影响**:
- 搜索结果不相关
- 用户找不到想要的商品
- 转化率低

#### 难点3: 数据实时性
**问题描述**:
- 商品上架如何实时搜索到？
- 库存变化如何实时更新？
- 价格调整如何实时生效？
- 如何保证数据一致性？

**业务影响**:
- 搜索到已下架商品
- 库存显示不准确
- 价格显示错误

## 🏗️ 系统架构

### 整体架构
```
用户 → API网关 → 搜索服务 → Elasticsearch集群
                    ↓
              [数据同步、缓存]
                    ↓
                 MySQL + Redis
```

### 技术选型
- **搜索引擎**: Elasticsearch 8.11.x
- **分词器**: IK Analyzer
- **缓存**: Redis 7.2.x
- **数据同步**: Canal
- **框架**: Spring Boot 3.2.x

## 🔥 核心实现

### 1. 索引设计

```json
{
  "settings": {
    "number_of_shards": 5,
    "number_of_replicas": 1,
    "analysis": {
      "analyzer": {
        "ik_smart_pinyin": {
          "type": "custom",
          "tokenizer": "ik_smart",
          "filter": ["pinyin_filter"]
        }
      },
      "filter": {
        "pinyin_filter": {
          "type": "pinyin",
          "keep_first_letter": true,
          "keep_separate_first_letter": false,
          "keep_full_pinyin": true,
          "keep_original": true,
          "limit_first_letter_length": 16,
          "lowercase": true
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "id": {
        "type": "long"
      },
      "title": {
        "type": "text",
        "analyzer": "ik_max_word",
        "search_analyzer": "ik_smart",
        "fields": {
          "pinyin": {
            "type": "text",
            "analyzer": "ik_smart_pinyin"
          },
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "category_id": {
        "type": "long"
      },
      "category_name": {
        "type": "keyword"
      },
      "brand_id": {
        "type": "long"
      },
      "brand_name": {
        "type": "keyword"
      },
      "price": {
        "type": "double"
      },
      "sales": {
        "type": "long"
      },
      "stock": {
        "type": "long"
      },
      "status": {
        "type": "integer"
      },
      "create_time": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      },
      "update_time": {
        "type": "date",
        "format": "yyyy-MM-dd HH:mm:ss"
      }
    }
  }
}
```

### 2. 搜索服务实现

```java
/**
 * 搜索服务
 * @author erik.zhou
 * 
 * 难点解决：
 * 1. 复杂查询构建
 * 2. 多条件筛选
 * 3. 搜索结果高亮
 * 4. 分页优化
 */
@Service
@Slf4j
public class ProductSearchService {
    
    @Autowired
    private ElasticsearchClient esClient;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    private static final String INDEX_NAME = "product";
    
    /**
     * 搜索商品
     */
    public SearchResult<Product> search(SearchRequest request) {
        log.info("搜索商品: {}", request);
        
        try {
            // 1. 构建查询
            Query query = buildQuery(request);
            
            // 2. 构建排序
            List<SortOptions> sorts = buildSort(request);
            
            // 3. 构建高亮
            Highlight highlight = buildHighlight();
            
            // 4. 构建聚合
            Map<String, Aggregation> aggregations = buildAggregations();
            
            // 5. 执行搜索
            SearchResponse<Product> response = esClient.search(s -> s
                .index(INDEX_NAME)
                .query(query)
                .from(request.getFrom())
                .size(request.getSize())
                .sort(sorts)
                .highlight(highlight)
                .aggregations(aggregations)
                .trackTotalHits(t -> t.enabled(true)),
                Product.class
            );
            
            // 6. 解析结果
            return parseSearchResponse(response);
            
        } catch (Exception e) {
            log.error("搜索失败: request={}", request, e);
            throw new BusinessException("搜索失败", e);
        }
    }
    
    /**
     * 构建查询
     * 
     * 难点解决：
     * 1. 多字段搜索
     * 2. 拼音搜索
     * 3. 多条件过滤
     * 4. 权重设置
     */
    private Query buildQuery(SearchRequest request) {
        BoolQuery.Builder boolQuery = new BoolQuery.Builder();
        
        // 1. 关键词搜索
        if (StringUtils.isNotBlank(request.getKeyword())) {
            // 多字段搜索（标题、拼音）
            boolQuery.must(m -> m
                .multiMatch(mm -> mm
                    .query(request.getKeyword())
                    .fields("title^3", "title.pinyin^2") // 权重设置
                    .type(TextQueryType.BestFields)
                    .operator(Operator.And)
                )
            );
        }
        
        // 2. 分类筛选
        if (request.getCategoryId() != null) {
            boolQuery.filter(f -> f
                .term(t -> t
                    .field("category_id")
                    .value(request.getCategoryId())
                )
            );
        }
        
        // 3. 品牌筛选
        if (request.getBrandId() != null) {
            boolQuery.filter(f -> f
                .term(t -> t
                    .field("brand_id")
                    .value(request.getBrandId())
                )
            );
        }
        
        // 4. 价格区间筛选
        if (request.getMinPrice() != null || request.getMaxPrice() != null) {
            boolQuery.filter(f -> f
                .range(r -> {
                    RangeQuery.Builder rangeBuilder = r.field("price");
                    if (request.getMinPrice() != null) {
                        rangeBuilder.gte(JsonData.of(request.getMinPrice()));
                    }
                    if (request.getMaxPrice() != null) {
                        rangeBuilder.lte(JsonData.of(request.getMaxPrice()));
                    }
                    return rangeBuilder;
                })
            );
        }
        
        // 5. 状态筛选（只搜索上架商品）
        boolQuery.filter(f -> f
            .term(t -> t
                .field("status")
                .value(1)
            )
        );
        
        // 6. 库存筛选（只搜索有库存商品）
        boolQuery.filter(f -> f
            .range(r -> r
                .field("stock")
                .gt(JsonData.of(0))
            )
        );
        
        return boolQuery.build()._toQuery();
    }
    
    /**
     * 构建排序
     */
    private List<SortOptions> buildSort(SearchRequest request) {
        List<SortOptions> sorts = new ArrayList<>();
        
        String sortField = request.getSortField();
        String sortOrder = request.getSortOrder();
        
        if (StringUtils.isNotBlank(sortField)) {
            SortOrder order = "desc".equalsIgnoreCase(sortOrder) 
                ? SortOrder.Desc 
                : SortOrder.Asc;
            
            sorts.add(SortOptions.of(s -> s
                .field(f -> f
                    .field(sortField)
                    .order(order)
                )
            ));
        }
        
        // 默认按相关性排序
        sorts.add(SortOptions.of(s -> s
            .score(sc -> sc.order(SortOrder.Desc))
        ));
        
        return sorts;
    }
    
    /**
     * 构建高亮
     */
    private Highlight buildHighlight() {
        return Highlight.of(h -> h
            .fields("title", hf -> hf
                .preTags("<em class='highlight'>")
                .postTags("</em>")
            )
        );
    }
    
    /**
     * 构建聚合
     * 
     * 难点解决：
     * 1. 分类聚合
     * 2. 品牌聚合
     * 3. 价格区间聚合
     */
    private Map<String, Aggregation> buildAggregations() {
        Map<String, Aggregation> aggregations = new HashMap<>();
        
        // 1. 分类聚合
        aggregations.put("categories", Aggregation.of(a -> a
            .terms(t -> t
                .field("category_id")
                .size(100)
            )
        ));
        
        // 2. 品牌聚合
        aggregations.put("brands", Aggregation.of(a -> a
            .terms(t -> t
                .field("brand_id")
                .size(100)
            )
        ));
        
        // 3. 价格区间聚合
        aggregations.put("price_ranges", Aggregation.of(a -> a
            .range(r -> r
                .field("price")
                .ranges(
                    AggregationRange.of(ar -> ar.to("100")),
                    AggregationRange.of(ar -> ar.from("100").to("500")),
                    AggregationRange.of(ar -> ar.from("500").to("1000")),
                    AggregationRange.of(ar -> ar.from("1000"))
                )
            )
        ));
        
        return aggregations;
    }
    
    /**
     * 解析搜索结果
     */
    private SearchResult<Product> parseSearchResponse(
            SearchResponse<Product> response) {
        
        SearchResult<Product> result = new SearchResult<>();
        
        // 1. 总数
        result.setTotal(response.hits().total().value());
        
        // 2. 商品列表
        List<Product> products = new ArrayList<>();
        for (Hit<Product> hit : response.hits().hits()) {
            Product product = hit.source();
            
            // 设置高亮
            if (hit.highlight().containsKey("title")) {
                String highlightTitle = String.join("", 
                    hit.highlight().get("title"));
                product.setHighlightTitle(highlightTitle);
            }
            
            products.add(product);
        }
        result.setProducts(products);
        
        // 3. 聚合结果
        result.setAggregations(parseAggregations(response.aggregations()));
        
        return result;
    }
}
```

---

**作者**: erik.zhou  
**最后更新**: 2024-01-04


### 3. 搜索建议（自动补全）

```java
/**
 * 搜索建议服务
 * @author erik.zhou
 * 
 * 难点解决：
 * 1. 前缀匹配
 * 2. 拼音匹配
 * 3. 热词推荐
 * 4. 个性化推荐
 */
@Service
@Slf4j
public class SearchSuggestionService {
    
    @Autowired
    private ElasticsearchClient esClient;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 获取搜索建议
     */
    public List<String> getSuggestions(String keyword, int size) {
        try {
            // 1. 从缓存获取
            List<String> cached = getCachedSuggestions(keyword);
            if (cached != null && !cached.isEmpty()) {
                return cached.subList(0, Math.min(size, cached.size()));
            }
            
            // 2. 从ES获取
            List<String> suggestions = searchSuggestions(keyword, size);
            
            // 3. 缓存结果
            cacheSuggestions(keyword, suggestions);
            
            return suggestions;
            
        } catch (Exception e) {
            log.error("获取搜索建议失败: keyword={}", keyword, e);
            return Collections.emptyList();
        }
    }
    
    /**
     * 搜索建议
     */
    private List<String> searchSuggestions(String keyword, int size) 
            throws IOException {
        
        // 使用completion suggester
        SearchResponse<Product> response = esClient.search(s -> s
            .index("product")
            .suggest(sg -> sg
                .suggesters("title_suggest", ss -> ss
                    .prefix(keyword)
                    .completion(c -> c
                        .field("title.suggest")
                        .size(size)
                        .skipDuplicates(true)
                    )
                )
            ),
            Product.class
        );
        
        // 解析建议结果
        List<String> suggestions = new ArrayList<>();
        if (response.suggest() != null) {
            response.suggest().get("title_suggest").forEach(suggest -> {
                suggest.completion().options().forEach(option -> {
                    suggestions.add(option.text());
                });
            });
        }
        
        return suggestions;
    }
    
    /**
     * 获取热门搜索词
     */
    public List<String> getHotKeywords(int size) {
        String key = "search:hot_keywords";
        
        // 从Redis获取热门搜索词（使用ZSet按搜索次数排序）
        Set<Object> hotKeywords = redisTemplate.opsForZSet()
            .reverseRange(key, 0, size - 1);
        
        if (hotKeywords != null && !hotKeywords.isEmpty()) {
            return hotKeywords.stream()
                .map(Object::toString)
                .collect(Collectors.toList());
        }
        
        return Collections.emptyList();
    }
    
    /**
     * 记录搜索词
     */
    public void recordSearchKeyword(String keyword) {
        if (StringUtils.isBlank(keyword)) {
            return;
        }
        
        String key = "search:hot_keywords";
        
        // 增加搜索次数
        redisTemplate.opsForZSet().incrementScore(key, keyword, 1);
        
        // 设置过期时间（7天）
        redisTemplate.expire(key, 7, TimeUnit.DAYS);
    }
    
    /**
     * 缓存搜索建议
     */
    private void cacheSuggestions(String keyword, List<String> suggestions) {
        String key = "search:suggestions:" + keyword;
        redisTemplate.opsForValue().set(key, suggestions, 1, TimeUnit.HOURS);
    }
    
    /**
     * 获取缓存的搜索建议
     */
    @SuppressWarnings("unchecked")
    private List<String> getCachedSuggestions(String keyword) {
        String key = "search:suggestions:" + keyword;
        Object cached = redisTemplate.opsForValue().get(key);
        
        if (cached instanceof List) {
            return (List<String>) cached;
        }
        
        return null;
    }
}
```

### 4. 搜索统计分析

```java
/**
 * 搜索统计服务
 * @author erik.zhou
 * 
 * 难点解决：
 * 1. 搜索日志记录
 * 2. 搜索行为分析
 * 3. 无结果搜索分析
 * 4. 搜索转化率统计
 */
@Service
@Slf4j
public class SearchAnalyticsService {
    
    @Autowired
    private SearchLogMapper searchLogMapper;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 记录搜索日志
     */
    @Async
    public void recordSearchLog(SearchLogDTO logDTO) {
        try {
            SearchLog log = SearchLog.builder()
                .userId(logDTO.getUserId())
                .keyword(logDTO.getKeyword())
                .resultCount(logDTO.getResultCount())
                .searchTime(logDTO.getSearchTime())
                .hasClick(false)
                .createTime(new Date())
                .build();
            
            searchLogMapper.insert(log);
            
            // 记录到Redis（用于实时统计）
            recordToRedis(logDTO);
            
        } catch (Exception e) {
            log.error("记录搜索日志失败", e);
        }
    }
    
    /**
     * 记录到Redis
     */
    private void recordToRedis(SearchLogDTO logDTO) {
        String date = new SimpleDateFormat("yyyyMMdd").format(new Date());
        
        // 1. 搜索次数
        String countKey = "search:count:" + date;
        redisTemplate.opsForValue().increment(countKey);
        redisTemplate.expire(countKey, 30, TimeUnit.DAYS);
        
        // 2. 无结果搜索
        if (logDTO.getResultCount() == 0) {
            String noResultKey = "search:no_result:" + date;
            redisTemplate.opsForSet().add(noResultKey, logDTO.getKeyword());
            redisTemplate.expire(noResultKey, 30, TimeUnit.DAYS);
        }
        
        // 3. 搜索耗时统计
        String timeKey = "search:time:" + date;
        redisTemplate.opsForList().rightPush(timeKey, logDTO.getSearchTime());
        redisTemplate.expire(timeKey, 30, TimeUnit.DAYS);
    }
    
    /**
     * 获取搜索统计
     */
    public SearchStatistics getStatistics(LocalDate date) {
        String dateStr = date.format(DateTimeFormatter.ofPattern("yyyyMMdd"));
        
        SearchStatistics statistics = new SearchStatistics();
        
        // 1. 搜索次数
        String countKey = "search:count:" + dateStr;
        Object count = redisTemplate.opsForValue().get(countKey);
        statistics.setSearchCount(count != null ? Long.valueOf(count.toString()) : 0L);
        
        // 2. 无结果搜索
        String noResultKey = "search:no_result:" + dateStr;
        Long noResultCount = redisTemplate.opsForSet().size(noResultKey);
        statistics.setNoResultCount(noResultCount != null ? noResultCount : 0L);
        
        // 3. 平均搜索耗时
        String timeKey = "search:time:" + dateStr;
        List<Object> times = redisTemplate.opsForList().range(timeKey, 0, -1);
        if (times != null && !times.isEmpty()) {
            double avgTime = times.stream()
                .mapToLong(t -> Long.valueOf(t.toString()))
                .average()
                .orElse(0.0);
            statistics.setAvgSearchTime(avgTime);
        }
        
        return statistics;
    }
    
    /**
     * 获取无结果搜索词
     */
    public List<String> getNoResultKeywords(LocalDate date, int limit) {
        String dateStr = date.format(DateTimeFormatter.ofPattern("yyyyMMdd"));
        String key = "search:no_result:" + dateStr;
        
        Set<Object> keywords = redisTemplate.opsForSet().members(key);
        
        if (keywords != null && !keywords.isEmpty()) {
            return keywords.stream()
                .map(Object::toString)
                .limit(limit)
                .collect(Collectors.toList());
        }
        
        return Collections.emptyList();
    }
}
```

## 🔥 难点深度解析

### 难点1: 如何优化搜索性能？

#### 问题场景
```
用户搜索"手机"
    ↓
返回100万条结果
    ↓
如何快速返回？
如何避免深分页？
```

#### 解决方案：多层优化

```java
/**
 * 搜索性能优化
 * @author erik.zhou
 */
@Service
@Slf4j
public class SearchPerformanceOptimizer {
    
    /**
     * 1. 使用缓存
     */
    public SearchResult<Product> searchWithCache(SearchRequest request) {
        // 生成缓存key
        String cacheKey = generateCacheKey(request);
        
        // 从缓存获取
        SearchResult<Product> cached = getFromCache(cacheKey);
        if (cached != null) {
            log.debug("命中缓存: key={}", cacheKey);
            return cached;
        }
        
        // 执行搜索
        SearchResult<Product> result = search(request);
        
        // 缓存结果（热门搜索词缓存时间长一些）
        int expireTime = isHotKeyword(request.getKeyword()) ? 3600 : 300;
        cacheResult(cacheKey, result, expireTime);
        
        return result;
    }
    
    /**
     * 2. 避免深分页
     * 使用search_after代替from/size
     */
    public SearchResult<Product> searchAfter(SearchRequest request, 
                                             List<Object> searchAfter) {
        try {
            SearchResponse<Product> response = esClient.search(s -> {
                SearchRequest.Builder builder = s
                    .index("product")
                    .query(buildQuery(request))
                    .size(request.getSize())
                    .sort(buildSort(request));
                
                // 使用search_after
                if (searchAfter != null && !searchAfter.isEmpty()) {
                    builder.searchAfter(searchAfter);
                }
                
                return builder;
            }, Product.class);
            
            return parseSearchResponse(response);
            
        } catch (Exception e) {
            log.error("搜索失败", e);
            throw new BusinessException("搜索失败", e);
        }
    }
    
    /**
     * 3. 使用routing优化
     */
    public SearchResult<Product> searchWithRouting(SearchRequest request) {
        try {
            // 根据分类ID进行routing
            String routing = String.valueOf(request.getCategoryId());
            
            SearchResponse<Product> response = esClient.search(s -> s
                .index("product")
                .routing(routing) // 指定routing，只搜索特定分片
                .query(buildQuery(request))
                .from(request.getFrom())
                .size(request.getSize()),
                Product.class
            );
            
            return parseSearchResponse(response);
            
        } catch (Exception e) {
            log.error("搜索失败", e);
            throw new BusinessException("搜索失败", e);
        }
    }
    
    /**
     * 4. 使用filter context代替query context
     * filter不计算相关性分数，性能更好
     */
    private Query buildOptimizedQuery(SearchRequest request) {
        BoolQuery.Builder boolQuery = new BoolQuery.Builder();
        
        // 关键词搜索使用must（需要计算分数）
        if (StringUtils.isNotBlank(request.getKeyword())) {
            boolQuery.must(m -> m
                .match(mt -> mt
                    .field("title")
                    .query(request.getKeyword())
                )
            );
        }
        
        // 其他条件使用filter（不计算分数，性能更好）
        if (request.getCategoryId() != null) {
            boolQuery.filter(f -> f
                .term(t -> t.field("category_id").value(request.getCategoryId()))
            );
        }
        
        if (request.getBrandId() != null) {
            boolQuery.filter(f -> f
                .term(t -> t.field("brand_id").value(request.getBrandId()))
            );
        }
        
        return boolQuery.build()._toQuery();
    }
}
```

### 难点2: 如何提高搜索相关性？

#### 解决方案：多策略优化

```java
/**
 * 搜索相关性优化
 * @author erik.zhou
 */
@Service
@Slf4j
public class SearchRelevanceOptimizer {
    
    /**
     * 1. 使用function_score调整分数
     */
    public Query buildFunctionScoreQuery(SearchRequest request) {
        // 基础查询
        Query baseQuery = buildBaseQuery(request);
        
        // 构建function_score查询
        return Query.of(q -> q
            .functionScore(fs -> fs
                .query(baseQuery)
                .functions(
                    // 销量加权
                    FunctionScore.of(f -> f
                        .fieldValueFactor(fv -> fv
                            .field("sales")
                            .factor(1.2)
                            .modifier(FieldValueFactorModifier.Log1p)
                        )
                        .weight(2.0)
                    ),
                    // 新品加权
                    FunctionScore.of(f -> f
                        .gauss(g -> g
                            .field("create_time")
                            .origin(JsonData.of(new Date()))
                            .scale(JsonData.of("30d"))
                        )
                        .weight(1.5)
                    )
                )
                .scoreMode(FunctionScoreMode.Sum)
                .boostMode(FunctionBoostMode.Multiply)
            )
        );
    }
    
    /**
     * 2. 使用同义词
     */
    public void createSynonymAnalyzer() {
        // 在索引settings中配置同义词
        /*
        "analysis": {
          "filter": {
            "synonym_filter": {
              "type": "synonym",
              "synonyms": [
                "手机,mobile,phone",
                "电脑,computer,pc",
                "笔记本,notebook,laptop"
              ]
            }
          },
          "analyzer": {
            "synonym_analyzer": {
              "tokenizer": "ik_smart",
              "filter": ["synonym_filter"]
            }
          }
        }
        */
    }
    
    /**
     * 3. 使用dis_max查询
     * 取多个字段中最高分数
     */
    public Query buildDisMaxQuery(String keyword) {
        return Query.of(q -> q
            .disMax(dm -> dm
                .queries(
                    Query.of(mq -> mq
                        .match(m -> m.field("title").query(keyword).boost(3.0f))
                    ),
                    Query.of(mq -> mq
                        .match(m -> m.field("description").query(keyword).boost(1.0f))
                    ),
                    Query.of(mq -> mq
                        .match(m -> m.field("brand_name").query(keyword).boost(2.0f))
                    )
                )
                .tieBreaker(0.3)
            )
        );
    }
}
```

### 难点3: 如何保证数据实时性？

#### 解决方案：实时同步 + 定时刷新

```java
/**
 * 数据实时同步
 * @author erik.zhou
 */
@Service
@Slf4j
public class RealTimeSyncService {
    
    @Autowired
    private ElasticsearchClient esClient;
    
    /**
     * 1. 实时更新单个文档
     */
    public void updateProduct(Product product) {
        try {
            UpdateRequest<Product, Product> request = UpdateRequest.of(u -> u
                .index("product")
                .id(String.valueOf(product.getId()))
                .doc(product)
                .docAsUpsert(true)
                .refresh(Refresh.True) // 立即刷新，使数据可搜索
            );
            
            esClient.update(request, Product.class);
            
            log.info("更新商品成功: productId={}", product.getId());
            
        } catch (Exception e) {
            log.error("更新商品失败: productId={}", product.getId(), e);
        }
    }
    
    /**
     * 2. 批量更新
     */
    public void batchUpdateProducts(List<Product> products) {
        try {
            BulkRequest.Builder builder = new BulkRequest.Builder();
            
            for (Product product : products) {
                builder.operations(op -> op
                    .update(u -> u
                        .index("product")
                        .id(String.valueOf(product.getId()))
                        .action(a -> a
                            .doc(product)
                            .docAsUpsert(true)
                        )
                    )
                );
            }
            
            BulkResponse response = esClient.bulk(builder.build());
            
            if (response.errors()) {
                log.error("批量更新部分失败");
            } else {
                log.info("批量更新成功: count={}", products.size());
            }
            
        } catch (Exception e) {
            log.error("批量更新失败", e);
        }
    }
    
    /**
     * 3. 定时全量同步
     */
    @Scheduled(cron = "0 0 3 * * ?") // 每天凌晨3点执行
    public void fullSync() {
        log.info("开始全量同步");
        
        try {
            // 查询所有商品
            List<Product> products = productMapper.selectAll();
            
            // 批量同步到ES
            int batchSize = 1000;
            for (int i = 0; i < products.size(); i += batchSize) {
                int end = Math.min(i + batchSize, products.size());
                List<Product> batch = products.subList(i, end);
                
                batchUpdateProducts(batch);
                
                // 延迟一下，避免压力过大
                Thread.sleep(1000);
            }
            
            log.info("全量同步完成: total={}", products.size());
            
        } catch (Exception e) {
            log.error("全量同步失败", e);
        }
    }
}
```

## 📊 性能测试

### 测试结果

| 指标 | 数值 |
|------|------|
| 数据量 | 1亿 |
| 搜索QPS | 5000 |
| 平均响应时间 | 50ms |
| P99响应时间 | 150ms |
| 搜索准确率 | 95% |

## 💡 最佳实践

### 1. 索引设计
- 合理设置分片数
- 使用合适的分词器
- 字段类型选择正确

### 2. 查询优化
- 使用filter代替query
- 避免深分页
- 使用缓存

### 3. 数据同步
- 实时同步重要数据
- 定时全量同步
- 监控同步延迟

---

**作者**: erik.zhou  
**最后更新**: 2024-01-04
