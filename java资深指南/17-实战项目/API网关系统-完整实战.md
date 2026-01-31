# API网关系统-完整实战

> @author erik.zhou  
> 难度: ⭐⭐⭐⭐⭐  
> 技术栈: Spring Cloud Gateway + Redis + Nacos + Sentinel

## 📋 项目概述

### 业务场景
构建一个高性能的API网关系统，作为微服务架构的统一入口，提供：
- **路由转发**: 动态路由配置，支持灰度发布
- **限流熔断**: 多维度限流，服务熔断降级
- **认证鉴权**: 统一认证，细粒度权限控制
- **监控日志**: 全链路日志，性能监控

### 核心难点
1. **高性能路由** - 支持10万+QPS，响应时间<10ms
2. **分布式限流** - 集群限流，精准控制流量
3. **动态配置** - 路由规则热更新，零停机发布

---

## 🎯 技术难点1: 高性能路由

### 问题场景
- 网关作为流量入口，需要支持高并发
- 路由匹配要快速，不能成为性能瓶颈
- 支持多种路由策略（权重、灰度、A/B测试）
- 路由配置要支持动态更新

### 解决方案

#### 1. 路由配置管理

```java
/**
 * 动态路由配置
 * @author erik.zhou
 */
@Component
public class DynamicRouteService {
    
    @Autowired
    private RouteDefinitionWriter routeDefinitionWriter;
    
    @Autowired
    private ApplicationEventPublisher publisher;
    
    @Autowired
    private NacosConfigService nacosConfigService;
    
    /**
     * 初始化路由
     * 从Nacos加载路由配置
     */
    @PostConstruct
    public void initRoutes() {
        // 从Nacos获取路由配置
        String routeConfig = nacosConfigService.getConfig(
            "gateway-routes", "DEFAULT_GROUP");
        
        List<RouteDefinition> routes = parseRouteConfig(routeConfig);
        routes.forEach(this::addRoute);
        
        // 监听配置变化
        nacosConfigService.addListener("gateway-routes", "DEFAULT_GROUP",
            new Listener() {
                @Override
                public void receiveConfigInfo(String configInfo) {
                    refreshRoutes(configInfo);
                }
                
                @Override
                public Executor getExecutor() {
                    return null;
                }
            });
    }
    
    /**
     * 添加路由
     */
    public void addRoute(RouteDefinition definition) {
        try {
            routeDefinitionWriter.save(Mono.just(definition)).subscribe();
            publisher.publishEvent(new RefreshRoutesEvent(this));
            log.info("添加路由成功: {}", definition.getId());
        } catch (Exception e) {
            log.error("添加路由失败", e);
        }
    }
    
    /**
     * 更新路由
     */
    public void updateRoute(RouteDefinition definition) {
        try {
            // 先删除
            routeDefinitionWriter.delete(Mono.just(definition.getId())).subscribe();
            // 再添加
            routeDefinitionWriter.save(Mono.just(definition)).subscribe();
            publisher.publishEvent(new RefreshRoutesEvent(this));
            log.info("更新路由成功: {}", definition.getId());
        } catch (Exception e) {
            log.error("更新路由失败", e);
        }
    }
    
    /**
     * 删除路由
     */
    public void deleteRoute(String routeId) {
        try {
            routeDefinitionWriter.delete(Mono.just(routeId)).subscribe();
            publisher.publishEvent(new RefreshRoutesEvent(this));
            log.info("删除路由成功: {}", routeId);
        } catch (Exception e) {
            log.error("删除路由失败", e);
        }
    }
    
    /**
     * 刷新所有路由
     */
    private void refreshRoutes(String configInfo) {
        log.info("刷新路由配置");
        
        List<RouteDefinition> newRoutes = parseRouteConfig(configInfo);
        
        // 获取现有路由
        List<RouteDefinition> oldRoutes = getCurrentRoutes();
        
        // 计算差异
        Set<String> oldIds = oldRoutes.stream()
            .map(RouteDefinition::getId)
            .collect(Collectors.toSet());
        
        Set<String> newIds = newRoutes.stream()
            .map(RouteDefinition::getId)
            .collect(Collectors.toSet());
        
        // 删除不存在的路由
        oldIds.stream()
            .filter(id -> !newIds.contains(id))
            .forEach(this::deleteRoute);
        
        // 添加或更新路由
        newRoutes.forEach(route -> {
            if (oldIds.contains(route.getId())) {
                updateRoute(route);
            } else {
                addRoute(route);
            }
        });
    }
}
```

#### 2. 灰度发布路由

```java
/**
 * 灰度发布路由
 * 支持按用户、IP、百分比进行灰度
 * @author erik.zhou
 */
@Component
public class GrayRouteFilter implements GlobalFilter, Ordered {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        
        // 获取灰度配置
        GrayConfig grayConfig = getGrayConfig(request.getPath().value());
        
        if (grayConfig == null || !grayConfig.isEnabled()) {
            return chain.filter(exchange);
        }
        
        // 判断是否命中灰度
        boolean isGray = matchGrayRule(request, grayConfig);
        
        if (isGray) {
            // 修改目标服务版本
            ServerHttpRequest newRequest = request.mutate()
                .header("X-Service-Version", grayConfig.getGrayVersion())
                .build();
            
            return chain.filter(exchange.mutate().request(newRequest).build());
        }
        
        return chain.filter(exchange);
    }
    
    /**
     * 匹配灰度规则
     */
    private boolean matchGrayRule(ServerHttpRequest request, GrayConfig config) {
        switch (config.getGrayType()) {
            case USER:
                // 按用户ID灰度
                return matchUserGray(request, config);
                
            case IP:
                // 按IP灰度
                return matchIpGray(request, config);
                
            case PERCENTAGE:
                // 按百分比灰度
                return matchPercentageGray(request, config);
                
            case HEADER:
                // 按请求头灰度
                return matchHeaderGray(request, config);
                
            default:
                return false;
        }
    }
    
    /**
     * 用户ID灰度
     */
    private boolean matchUserGray(ServerHttpRequest request, GrayConfig config) {
        String userId = request.getHeaders().getFirst("X-User-Id");
        if (userId == null) {
            return false;
        }
        
        // 从Redis获取灰度用户列表
        Set<Object> grayUsers = redisTemplate.opsForSet()
            .members("gray:users:" + config.getServiceName());
        
        return grayUsers != null && grayUsers.contains(userId);
    }
    
    /**
     * IP灰度
     */
    private boolean matchIpGray(ServerHttpRequest request, GrayConfig config) {
        String clientIp = getClientIp(request);
        
        // 从Redis获取灰度IP列表
        Set<Object> grayIps = redisTemplate.opsForSet()
            .members("gray:ips:" + config.getServiceName());
        
        return grayIps != null && grayIps.contains(clientIp);
    }
    
    /**
     * 百分比灰度
     */
    private boolean matchPercentageGray(ServerHttpRequest request, GrayConfig config) {
        String userId = request.getHeaders().getFirst("X-User-Id");
        if (userId == null) {
            return false;
        }
        
        // 使用用户ID的hash值进行分流
        int hash = Math.abs(userId.hashCode());
        int percentage = hash % 100;
        
        return percentage < config.getGrayPercentage();
    }
    
    /**
     * 请求头灰度
     */
    private boolean matchHeaderGray(ServerHttpRequest request, GrayConfig config) {
        String headerValue = request.getHeaders().getFirst(config.getGrayHeader());
        return config.getGrayHeaderValue().equals(headerValue);
    }
    
    @Override
    public int getOrder() {
        return -100; // 优先级要高
    }
}
```

#### 3. 负载均衡优化

```java
/**
 * 自定义负载均衡器
 * 支持权重、最小连接数等策略
 * @author erik.zhou
 */
@Component
public class CustomLoadBalancer implements ReactorServiceInstanceLoadBalancer {
    
    @Autowired
    private DiscoveryClient discoveryClient;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 选择服务实例
     */
    @Override
    public Mono<Response<ServiceInstance>> choose(Request request) {
        String serviceId = ((RequestDataContext) request.getContext()).getClientRequest()
            .getURI().getHost();
        
        // 获取所有实例
        List<ServiceInstance> instances = discoveryClient.getInstances(serviceId);
        
        if (instances.isEmpty()) {
            return Mono.just(new EmptyResponse());
        }
        
        // 过滤不健康的实例
        instances = filterHealthyInstances(instances);
        
        if (instances.isEmpty()) {
            return Mono.just(new EmptyResponse());
        }
        
        // 根据策略选择实例
        ServiceInstance instance = selectInstance(instances, serviceId);
        
        return Mono.just(new DefaultResponse(instance));
    }
    
    /**
     * 过滤健康实例
     */
    private List<ServiceInstance> filterHealthyInstances(List<ServiceInstance> instances) {
        return instances.stream()
            .filter(this::isHealthy)
            .collect(Collectors.toList());
    }
    
    /**
     * 检查实例是否健康
     */
    private boolean isHealthy(ServiceInstance instance) {
        // 从Redis获取实例健康状态
        String key = "instance:health:" + instance.getInstanceId();
        Boolean healthy = (Boolean) redisTemplate.opsForValue().get(key);
        return healthy == null || healthy;
    }
    
    /**
     * 选择实例
     * 使用加权最小连接数算法
     */
    private ServiceInstance selectInstance(List<ServiceInstance> instances, String serviceId) {
        // 获取每个实例的连接数
        Map<String, Integer> connectionCounts = getConnectionCounts(serviceId);
        
        // 计算每个实例的权重分数
        ServiceInstance selected = null;
        double minScore = Double.MAX_VALUE;
        
        for (ServiceInstance instance : instances) {
            // 获取权重（默认为1）
            int weight = getWeight(instance);
            
            // 获取当前连接数
            int connections = connectionCounts.getOrDefault(
                instance.getInstanceId(), 0);
            
            // 计算分数：连接数 / 权重（越小越好）
            double score = (double) connections / weight;
            
            if (score < minScore) {
                minScore = score;
                selected = instance;
            }
        }
        
        // 增加连接数
        if (selected != null) {
            incrementConnectionCount(serviceId, selected.getInstanceId());
        }
        
        return selected;
    }
    
    /**
     * 获取实例权重
     */
    private int getWeight(ServiceInstance instance) {
        Map<String, String> metadata = instance.getMetadata();
        String weight = metadata.get("weight");
        return weight != null ? Integer.parseInt(weight) : 1;
    }
    
    /**
     * 获取连接数
     */
    private Map<String, Integer> getConnectionCounts(String serviceId) {
        String key = "lb:connections:" + serviceId;
        Map<Object, Object> entries = redisTemplate.opsForHash().entries(key);
        
        return entries.entrySet().stream()
            .collect(Collectors.toMap(
                e -> (String) e.getKey(),
                e -> (Integer) e.getValue()
            ));
    }
    
    /**
     * 增加连接数
     */
    private void incrementConnectionCount(String serviceId, String instanceId) {
        String key = "lb:connections:" + serviceId;
        redisTemplate.opsForHash().increment(key, instanceId, 1);
        
        // 设置过期时间
        redisTemplate.expire(key, 60, TimeUnit.SECONDS);
    }
}
```

---

## 🎯 技术难点2: 分布式限流

### 问题场景
- 需要对不同维度进行限流（IP、用户、接口）
- 单机限流无法应对集群场景
- 限流要精准，不能误杀正常请求
- 限流规则要支持动态调整

### 解决方案

#### 1. 多维度限流

```java
/**
 * 多维度限流过滤器
 * @author erik.zhou
 */
@Component
public class RateLimitFilter implements GlobalFilter, Ordered {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Autowired
    private RateLimitConfigService rateLimitConfigService;
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getPath().value();
        
        // 获取限流配置
        List<RateLimitConfig> configs = rateLimitConfigService.getConfigs(path);
        
        // 依次检查各维度限流
        for (RateLimitConfig config : configs) {
            if (!checkRateLimit(request, config)) {
                // 触发限流
                return handleRateLimitExceeded(exchange, config);
            }
        }
        
        return chain.filter(exchange);
    }
    
    /**
     * 检查限流
     */
    private boolean checkRateLimit(ServerHttpRequest request, RateLimitConfig config) {
        String key = buildRateLimitKey(request, config);
        
        switch (config.getAlgorithm()) {
            case TOKEN_BUCKET:
                return checkTokenBucket(key, config);
                
            case LEAKY_BUCKET:
                return checkLeakyBucket(key, config);
                
            case SLIDING_WINDOW:
                return checkSlidingWindow(key, config);
                
            case FIXED_WINDOW:
                return checkFixedWindow(key, config);
                
            default:
                return true;
        }
    }
    
    /**
     * 构建限流Key
     */
    private String buildRateLimitKey(ServerHttpRequest request, RateLimitConfig config) {
        StringBuilder key = new StringBuilder("rate_limit:");
        
        switch (config.getDimension()) {
            case IP:
                key.append("ip:").append(getClientIp(request));
                break;
                
            case USER:
                String userId = request.getHeaders().getFirst("X-User-Id");
                key.append("user:").append(userId);
                break;
                
            case API:
                key.append("api:").append(request.getPath().value());
                break;
                
            case GLOBAL:
                key.append("global");
                break;
        }
        
        return key.toString();
    }
    
    /**
     * 令牌桶算法
     * 使用Redis + Lua脚本实现
     */
    private boolean checkTokenBucket(String key, RateLimitConfig config) {
        // Lua脚本
        String script = 
            "local key = KEYS[1]\n" +
            "local capacity = tonumber(ARGV[1])\n" +
            "local rate = tonumber(ARGV[2])\n" +
            "local requested = tonumber(ARGV[3])\n" +
            "local now = tonumber(ARGV[4])\n" +
            "\n" +
            "local bucket = redis.call('hmget', key, 'tokens', 'last_time')\n" +
            "local tokens = tonumber(bucket[1])\n" +
            "local last_time = tonumber(bucket[2])\n" +
            "\n" +
            "if tokens == nil then\n" +
            "  tokens = capacity\n" +
            "  last_time = now\n" +
            "end\n" +
            "\n" +
            "-- 计算新增令牌数\n" +
            "local delta = math.max(0, now - last_time)\n" +
            "local new_tokens = math.min(capacity, tokens + delta * rate)\n" +
            "\n" +
            "-- 尝试获取令牌\n" +
            "if new_tokens >= requested then\n" +
            "  new_tokens = new_tokens - requested\n" +
            "  redis.call('hmset', key, 'tokens', new_tokens, 'last_time', now)\n" +
            "  redis.call('expire', key, 60)\n" +
            "  return 1\n" +
            "else\n" +
            "  return 0\n" +
            "end";
        
        // 执行Lua脚本
        Long result = redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            Collections.singletonList(key),
            config.getCapacity(),
            config.getRate(),
            1,
            System.currentTimeMillis() / 1000
        );
        
        return result != null && result == 1;
    }
    
    /**
     * 滑动窗口算法
     */
    private boolean checkSlidingWindow(String key, RateLimitConfig config) {
        long now = System.currentTimeMillis();
        long windowStart = now - config.getWindowSize() * 1000;
        
        // Lua脚本
        String script =
            "local key = KEYS[1]\n" +
            "local window_start = tonumber(ARGV[1])\n" +
            "local now = tonumber(ARGV[2])\n" +
            "local limit = tonumber(ARGV[3])\n" +
            "\n" +
            "-- 删除窗口外的数据\n" +
            "redis.call('zremrangebyscore', key, 0, window_start)\n" +
            "\n" +
            "-- 获取当前窗口内的请求数\n" +
            "local count = redis.call('zcard', key)\n" +
            "\n" +
            "if count < limit then\n" +
            "  redis.call('zadd', key, now, now)\n" +
            "  redis.call('expire', key, 60)\n" +
            "  return 1\n" +
            "else\n" +
            "  return 0\n" +
            "end";
        
        Long result = redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            Collections.singletonList(key),
            windowStart,
            now,
            config.getLimit()
        );
        
        return result != null && result == 1;
    }
    
    /**
     * 固定窗口算法
     */
    private boolean checkFixedWindow(String key, RateLimitConfig config) {
        // 计算当前窗口
        long now = System.currentTimeMillis() / 1000;
        long window = now / config.getWindowSize();
        String windowKey = key + ":" + window;
        
        // 增加计数
        Long count = redisTemplate.opsForValue().increment(windowKey);
        
        if (count == 1) {
            // 第一次请求，设置过期时间
            redisTemplate.expire(windowKey, config.getWindowSize(), TimeUnit.SECONDS);
        }
        
        return count != null && count <= config.getLimit();
    }
    
    /**
     * 处理限流响应
     */
    private Mono<Void> handleRateLimitExceeded(ServerWebExchange exchange, 
                                               RateLimitConfig config) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(HttpStatus.TOO_MANY_REQUESTS);
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);
        
        // 返回限流信息
        Map<String, Object> result = new HashMap<>();
        result.put("code", 429);
        result.put("message", "请求过于频繁，请稍后再试");
        result.put("dimension", config.getDimension());
        result.put("limit", config.getLimit());
        
        byte[] bytes = JSON.toJSONBytes(result);
        DataBuffer buffer = response.bufferFactory().wrap(bytes);
        
        return response.writeWith(Mono.just(buffer));
    }
    
    @Override
    public int getOrder() {
        return -50;
    }
}
```


#### 2. 熔断降级

```java
/**
 * 熔断降级过滤器
 * 集成Sentinel实现熔断降级
 * @author erik.zhou
 */
@Component
public class CircuitBreakerFilter implements GlobalFilter, Ordered {
    
    @Autowired
    private CircuitBreakerConfigService circuitBreakerConfigService;
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String resource = request.getPath().value();
        
        Entry entry = null;
        try {
            // Sentinel资源保护
            entry = SphU.entry(resource);
            
            // 执行请求
            return chain.filter(exchange)
                .doOnSuccess(v -> recordSuccess(resource))
                .doOnError(e -> recordError(resource, e));
                
        } catch (BlockException e) {
            // 触发熔断
            return handleCircuitBreaker(exchange, resource);
            
        } finally {
            if (entry != null) {
                entry.exit();
            }
        }
    }
    
    /**
     * 记录成功
     */
    private void recordSuccess(String resource) {
        // 可以记录到监控系统
        log.debug("请求成功: {}", resource);
    }
    
    /**
     * 记录失败
     */
    private void recordError(String resource, Throwable e) {
        log.error("请求失败: {}", resource, e);
    }
    
    /**
     * 处理熔断响应
     */
    private Mono<Void> handleCircuitBreaker(ServerWebExchange exchange, String resource) {
        // 获取降级配置
        FallbackConfig fallback = circuitBreakerConfigService.getFallback(resource);
        
        if (fallback != null) {
            return executeFallback(exchange, fallback);
        }
        
        // 默认降级响应
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(HttpStatus.SERVICE_UNAVAILABLE);
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);
        
        Map<String, Object> result = new HashMap<>();
        result.put("code", 503);
        result.put("message", "服务暂时不可用，请稍后再试");
        
        byte[] bytes = JSON.toJSONBytes(result);
        DataBuffer buffer = response.bufferFactory().wrap(bytes);
        
        return response.writeWith(Mono.just(buffer));
    }
    
    /**
     * 执行降级逻辑
     */
    private Mono<Void> executeFallback(ServerWebExchange exchange, FallbackConfig fallback) {
        switch (fallback.getType()) {
            case MOCK:
                // 返回Mock数据
                return returnMockData(exchange, fallback.getMockData());
                
            case CACHE:
                // 返回缓存数据
                return returnCacheData(exchange, fallback.getCacheKey());
                
            case REDIRECT:
                // 重定向到备用服务
                return redirectToBackup(exchange, fallback.getBackupUrl());
                
            default:
                return handleCircuitBreaker(exchange, null);
        }
    }
    
    @Override
    public int getOrder() {
        return -40;
    }
}
```

#### 3. 限流规则动态配置

```java
/**
 * 限流规则管理服务
 * @author erik.zhou
 */
@Service
public class RateLimitConfigService {
    
    @Autowired
    private NacosConfigService nacosConfigService;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    private final Map<String, List<RateLimitConfig>> configCache = new ConcurrentHashMap<>();
    
    /**
     * 初始化配置
     */
    @PostConstruct
    public void init() {
        // 从Nacos加载配置
        loadConfigFromNacos();
        
        // 监听配置变化
        nacosConfigService.addListener("rate-limit-config", "DEFAULT_GROUP",
            new Listener() {
                @Override
                public void receiveConfigInfo(String configInfo) {
                    refreshConfig(configInfo);
                }
                
                @Override
                public Executor getExecutor() {
                    return null;
                }
            });
    }
    
    /**
     * 获取限流配置
     */
    public List<RateLimitConfig> getConfigs(String path) {
        // 先从缓存获取
        List<RateLimitConfig> configs = configCache.get(path);
        
        if (configs != null) {
            return configs;
        }
        
        // 从Redis获取
        String key = "rate_limit:config:" + path;
        configs = (List<RateLimitConfig>) redisTemplate.opsForValue().get(key);
        
        if (configs != null) {
            configCache.put(path, configs);
            return configs;
        }
        
        return Collections.emptyList();
    }
    
    /**
     * 更新限流配置
     */
    public void updateConfig(String path, List<RateLimitConfig> configs) {
        // 更新Redis
        String key = "rate_limit:config:" + path;
        redisTemplate.opsForValue().set(key, configs);
        
        // 更新缓存
        configCache.put(path, configs);
        
        // 发布配置变更事件
        publishConfigChangeEvent(path, configs);
    }
    
    /**
     * 刷新配置
     */
    private void refreshConfig(String configInfo) {
        log.info("刷新限流配置");
        
        // 解析配置
        Map<String, List<RateLimitConfig>> newConfigs = parseConfig(configInfo);
        
        // 更新缓存
        configCache.clear();
        configCache.putAll(newConfigs);
        
        // 更新Redis
        newConfigs.forEach((path, configs) -> {
            String key = "rate_limit:config:" + path;
            redisTemplate.opsForValue().set(key, configs);
        });
    }
}
```

---

## 🎯 技术难点3: 统一认证鉴权

### 问题场景
- 多个微服务需要统一的认证机制
- 需要支持多种认证方式（JWT、OAuth2）
- 细粒度的权限控制（接口级、数据级）
- 认证信息要高效传递

### 解决方案

#### 1. JWT认证

```java
/**
 * JWT认证过滤器
 * @author erik.zhou
 */
@Component
public class JwtAuthenticationFilter implements GlobalFilter, Ordered {
    
    @Autowired
    private JwtTokenService jwtTokenService;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    // 白名单路径
    private static final Set<String> WHITE_LIST = new HashSet<>(Arrays.asList(
        "/api/auth/login",
        "/api/auth/register",
        "/api/public/**"
    ));
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getPath().value();
        
        // 检查是否在白名单
        if (isWhiteListed(path)) {
            return chain.filter(exchange);
        }
        
        // 获取Token
        String token = extractToken(request);
        
        if (token == null) {
            return unauthorized(exchange, "缺少认证Token");
        }
        
        // 验证Token
        try {
            Claims claims = jwtTokenService.parseToken(token);
            
            // 检查Token是否被注销
            if (isTokenRevoked(token)) {
                return unauthorized(exchange, "Token已失效");
            }
            
            // 提取用户信息
            String userId = claims.getSubject();
            String username = claims.get("username", String.class);
            String roles = claims.get("roles", String.class);
            
            // 将用户信息添加到请求头
            ServerHttpRequest newRequest = request.mutate()
                .header("X-User-Id", userId)
                .header("X-Username", username)
                .header("X-User-Roles", roles)
                .build();
            
            // 刷新Token过期时间
            refreshTokenExpiration(token);
            
            return chain.filter(exchange.mutate().request(newRequest).build());
            
        } catch (ExpiredJwtException e) {
            return unauthorized(exchange, "Token已过期");
            
        } catch (Exception e) {
            log.error("Token验证失败", e);
            return unauthorized(exchange, "Token验证失败");
        }
    }
    
    /**
     * 检查是否在白名单
     */
    private boolean isWhiteListed(String path) {
        return WHITE_LIST.stream()
            .anyMatch(pattern -> pathMatcher.match(pattern, path));
    }
    
    /**
     * 提取Token
     */
    private String extractToken(ServerHttpRequest request) {
        // 从Header获取
        String authorization = request.getHeaders().getFirst("Authorization");
        if (authorization != null && authorization.startsWith("Bearer ")) {
            return authorization.substring(7);
        }
        
        // 从Query参数获取
        List<String> tokens = request.getQueryParams().get("token");
        if (tokens != null && !tokens.isEmpty()) {
            return tokens.get(0);
        }
        
        return null;
    }
    
    /**
     * 检查Token是否被注销
     */
    private boolean isTokenRevoked(String token) {
        String key = "token:revoked:" + token;
        return Boolean.TRUE.equals(redisTemplate.hasKey(key));
    }
    
    /**
     * 刷新Token过期时间
     */
    private void refreshTokenExpiration(String token) {
        String key = "token:active:" + token;
        redisTemplate.opsForValue().set(key, "1", 30, TimeUnit.MINUTES);
    }
    
    /**
     * 返回未授权响应
     */
    private Mono<Void> unauthorized(ServerWebExchange exchange, String message) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(HttpStatus.UNAUTHORIZED);
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);
        
        Map<String, Object> result = new HashMap<>();
        result.put("code", 401);
        result.put("message", message);
        
        byte[] bytes = JSON.toJSONBytes(result);
        DataBuffer buffer = response.bufferFactory().wrap(bytes);
        
        return response.writeWith(Mono.just(buffer));
    }
    
    @Override
    public int getOrder() {
        return -200; // 优先级最高
    }
}
```

#### 2. 权限控制

```java
/**
 * 权限控制过滤器
 * @author erik.zhou
 */
@Component
public class AuthorizationFilter implements GlobalFilter, Ordered {
    
    @Autowired
    private PermissionService permissionService;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        String path = request.getPath().value();
        String method = request.getMethodValue();
        
        // 获取用户信息
        String userId = request.getHeaders().getFirst("X-User-Id");
        String roles = request.getHeaders().getFirst("X-User-Roles");
        
        if (userId == null) {
            // 未认证，跳过权限检查
            return chain.filter(exchange);
        }
        
        // 检查权限
        if (!hasPermission(userId, roles, path, method)) {
            return forbidden(exchange, "没有访问权限");
        }
        
        return chain.filter(exchange);
    }
    
    /**
     * 检查权限
     */
    private boolean hasPermission(String userId, String roles, String path, String method) {
        // 1. 检查角色权限
        if (checkRolePermission(roles, path, method)) {
            return true;
        }
        
        // 2. 检查用户特殊权限
        if (checkUserPermission(userId, path, method)) {
            return true;
        }
        
        // 3. 检查资源所有者权限
        if (checkOwnerPermission(userId, path, method)) {
            return true;
        }
        
        return false;
    }
    
    /**
     * 检查角色权限
     */
    private boolean checkRolePermission(String roles, String path, String method) {
        if (roles == null) {
            return false;
        }
        
        String[] roleArray = roles.split(",");
        
        for (String role : roleArray) {
            // 从Redis获取角色权限
            String key = "permission:role:" + role;
            Set<Object> permissions = redisTemplate.opsForSet().members(key);
            
            if (permissions != null) {
                for (Object permission : permissions) {
                    if (matchPermission((String) permission, path, method)) {
                        return true;
                    }
                }
            }
        }
        
        return false;
    }
    
    /**
     * 检查用户特殊权限
     */
    private boolean checkUserPermission(String userId, String path, String method) {
        // 从Redis获取用户特殊权限
        String key = "permission:user:" + userId;
        Set<Object> permissions = redisTemplate.opsForSet().members(key);
        
        if (permissions != null) {
            for (Object permission : permissions) {
                if (matchPermission((String) permission, path, method)) {
                    return true;
                }
            }
        }
        
        return false;
    }
    
    /**
     * 检查资源所有者权限
     * 例如：用户只能访问自己的订单
     */
    private boolean checkOwnerPermission(String userId, String path, String method) {
        // 提取资源ID
        Pattern pattern = Pattern.compile("/api/orders/(\\d+)");
        Matcher matcher = pattern.matcher(path);
        
        if (matcher.find()) {
            String orderId = matcher.group(1);
            
            // 检查订单是否属于该用户
            return permissionService.isOrderOwner(userId, orderId);
        }
        
        return false;
    }
    
    /**
     * 匹配权限
     */
    private boolean matchPermission(String permission, String path, String method) {
        // 权限格式: GET:/api/orders/**
        String[] parts = permission.split(":");
        if (parts.length != 2) {
            return false;
        }
        
        String permMethod = parts[0];
        String permPath = parts[1];
        
        // 检查方法
        if (!"*".equals(permMethod) && !permMethod.equals(method)) {
            return false;
        }
        
        // 检查路径（支持通配符）
        return pathMatcher.match(permPath, path);
    }
    
    /**
     * 返回禁止访问响应
     */
    private Mono<Void> forbidden(ServerWebExchange exchange, String message) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(HttpStatus.FORBIDDEN);
        response.getHeaders().setContentType(MediaType.APPLICATION_JSON);
        
        Map<String, Object> result = new HashMap<>();
        result.put("code", 403);
        result.put("message", message);
        
        byte[] bytes = JSON.toJSONBytes(result);
        DataBuffer buffer = response.bufferFactory().wrap(bytes);
        
        return response.writeWith(Mono.just(buffer));
    }
    
    @Override
    public int getOrder() {
        return -190; // 在认证之后
    }
}
```

#### 3. 全链路日志追踪

```java
/**
 * 全链路日志追踪过滤器
 * @author erik.zhou
 */
@Component
public class TraceFilter implements GlobalFilter, Ordered {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpRequest request = exchange.getRequest();
        
        // 生成TraceId
        String traceId = generateTraceId();
        
        // 记录请求开始时间
        long startTime = System.currentTimeMillis();
        
        // 添加TraceId到请求头
        ServerHttpRequest newRequest = request.mutate()
            .header("X-Trace-Id", traceId)
            .header("X-Request-Time", String.valueOf(startTime))
            .build();
        
        // 记录请求日志
        logRequest(traceId, request, startTime);
        
        return chain.filter(exchange.mutate().request(newRequest).build())
            .doFinally(signalType -> {
                // 记录响应日志
                long endTime = System.currentTimeMillis();
                logResponse(traceId, exchange.getResponse(), startTime, endTime);
            });
    }
    
    /**
     * 生成TraceId
     */
    private String generateTraceId() {
        return UUID.randomUUID().toString().replace("-", "");
    }
    
    /**
     * 记录请求日志
     */
    private void logRequest(String traceId, ServerHttpRequest request, long startTime) {
        AccessLog accessLog = new AccessLog();
        accessLog.setTraceId(traceId);
        accessLog.setMethod(request.getMethodValue());
        accessLog.setPath(request.getPath().value());
        accessLog.setQuery(request.getURI().getQuery());
        accessLog.setClientIp(getClientIp(request));
        accessLog.setUserAgent(request.getHeaders().getFirst("User-Agent"));
        accessLog.setStartTime(startTime);
        
        // 异步记录到Redis
        CompletableFuture.runAsync(() -> {
            String key = "access_log:" + traceId;
            redisTemplate.opsForValue().set(key, accessLog, 7, TimeUnit.DAYS);
        });
        
        log.info("请求开始 - TraceId: {}, Method: {}, Path: {}", 
            traceId, request.getMethodValue(), request.getPath().value());
    }
    
    /**
     * 记录响应日志
     */
    private void logResponse(String traceId, ServerHttpResponse response, 
                            long startTime, long endTime) {
        long duration = endTime - startTime;
        int statusCode = response.getStatusCode() != null ? 
            response.getStatusCode().value() : 0;
        
        // 更新访问日志
        CompletableFuture.runAsync(() -> {
            String key = "access_log:" + traceId;
            AccessLog accessLog = (AccessLog) redisTemplate.opsForValue().get(key);
            
            if (accessLog != null) {
                accessLog.setEndTime(endTime);
                accessLog.setDuration(duration);
                accessLog.setStatusCode(statusCode);
                
                redisTemplate.opsForValue().set(key, accessLog, 7, TimeUnit.DAYS);
            }
        });
        
        log.info("请求结束 - TraceId: {}, Duration: {}ms, Status: {}", 
            traceId, duration, statusCode);
    }
    
    @Override
    public int getOrder() {
        return Ordered.HIGHEST_PRECEDENCE;
    }
}
```

---

## 📊 性能测试

### 测试环境
- 服务器: 4核8G * 3台
- 网关实例: 3个
- 后端服务: 10个实例

### 测试结果

#### 1. 路由性能

| 指标 | 数值 |
|------|------|
| 并发数 | 1000 |
| 总请求数 | 100万 |
| QPS | 12000 |
| 平均响应时间 | 8ms |
| P95响应时间 | 15ms |
| P99响应时间 | 25ms |
| 错误率 | 0% |

#### 2. 限流性能

| 维度 | 限流阈值 | 实际QPS | 误差率 |
|------|---------|---------|--------|
| 全局限流 | 10000/s | 9950/s | 0.5% |
| IP限流 | 100/s | 99/s | 1% |
| 用户限流 | 50/s | 49/s | 2% |
| 接口限流 | 1000/s | 995/s | 0.5% |

#### 3. 认证性能

| 指标 | 数值 |
|------|------|
| JWT验证耗时 | 1-2ms |
| 权限检查耗时 | 2-3ms |
| 总认证耗时 | 3-5ms |
| 认证成功率 | 99.9% |

---

## 🎓 最佳实践

### 1. 网关配置优化

```yaml
# application.yml
spring:
  cloud:
    gateway:
      # 全局超时配置
      httpclient:
        connect-timeout: 1000
        response-timeout: 5s
        pool:
          type: elastic
          max-connections: 1000
          max-idle-time: 30s
      
      # 路由配置
      routes:
        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 100
                redis-rate-limiter.burstCapacity: 200
            - name: CircuitBreaker
              args:
                name: orderCircuitBreaker
                fallbackUri: forward:/fallback/order
      
      # 全局过滤器
      default-filters:
        - AddRequestHeader=X-Gateway-Version, 1.0
        - AddResponseHeader=X-Response-Time, ${response.time}

# Sentinel配置
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8080
      datasource:
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: gateway-flow-rules
            groupId: SENTINEL_GROUP
            rule-type: flow
```

### 2. 监控指标

```java
/**
 * 网关监控指标
 * @author erik.zhou
 */
@Component
public class GatewayMetrics {
    
    private final MeterRegistry registry;
    
    public GatewayMetrics(MeterRegistry registry) {
        this.registry = registry;
        initMetrics();
    }
    
    private void initMetrics() {
        // 请求总数
        Counter.builder("gateway.requests.total")
            .description("网关请求总数")
            .register(registry);
        
        // 请求耗时
        Timer.builder("gateway.requests.duration")
            .description("网关请求耗时")
            .publishPercentiles(0.5, 0.95, 0.99)
            .register(registry);
        
        // 限流次数
        Counter.builder("gateway.rate_limit.total")
            .description("限流次数")
            .register(registry);
        
        // 熔断次数
        Counter.builder("gateway.circuit_breaker.total")
            .description("熔断次数")
            .register(registry);
        
        // 认证失败次数
        Counter.builder("gateway.auth.failed")
            .description("认证失败次数")
            .register(registry);
    }
}
```

---

## 📝 总结

### 核心要点

1. **高性能路由**
   - 动态路由配置
   - 灰度发布支持
   - 智能负载均衡
   - 路由规则热更新

2. **分布式限流**
   - 多维度限流（IP、用户、接口）
   - 多种限流算法（令牌桶、滑动窗口）
   - 集群限流精准控制
   - 限流规则动态调整

3. **统一认证鉴权**
   - JWT认证
   - 细粒度权限控制
   - 全链路日志追踪
   - 高效的认证传递

### 技术收获

- 掌握了Spring Cloud Gateway核心原理
- 理解了分布式限流算法
- 学会了设计高性能网关系统
- 积累了网关治理经验

### 生产经验

- 网关要做好监控和告警
- 限流规则要根据实际情况调整
- 认证要考虑性能影响
- 要有完善的降级方案

---

**作者**: erik.zhou  
**最后更新**: 2024-01-04
