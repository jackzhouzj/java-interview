# Dubbo 源码解析 - 完整教程

> 深入理解 Dubbo 的服务治理和 RPC 调用原理
> 
> @author erik.zhou

## 📚 技术概述

| 项目 | 说明 |
|------|------|
| **框架名称** | Apache Dubbo |
| **当前版本** | 3.2.x |
| **源码地址** | https://github.com/apache/dubbo |
| **学习难度** | ⭐⭐⭐⭐ |
| **重要程度** | ⭐⭐⭐⭐ |
| **预计时长** | 30-40 小时 |
| **前置知识** | RPC 原理、网络编程、动态代理、SPI |

## 🎯 学习目标

- [ ] 理解 Dubbo 的整体架构
- [ ] 掌握服务暴露的完整流程
- [ ] 掌握服务引用的完整流程
- [ ] 理解负载均衡的实现
- [ ] 掌握集群容错机制
- [ ] 理解 SPI 扩展机制
- [ ] 能够自定义 Dubbo 扩展

## 📖 目录

1. [Dubbo 整体架构](#1-dubbo-整体架构)
2. [服务暴露流程](#2-服务暴露流程)
3. [服务引用流程](#3-服务引用流程)
4. [服务调用流程](#4-服务调用流程)
5. [负载均衡实现](#5-负载均衡实现)
6. [集群容错机制](#6-集群容错机制)
7. [SPI 扩展机制](#7-spi-扩展机制)

---

## 1. Dubbo 整体架构

### 1.1 核心角色

```
┌─────────────┐
│  Registry   │  注册中心（Nacos、ZooKeeper）
└──────┬──────┘
       │
   register/subscribe
       │
┌──────┴──────┐         ┌─────────────┐
│  Provider   │ ◄────── │  Consumer   │
│  (服务提供者) │  invoke  │  (服务消费者) │
└─────────────┘         └─────────────┘
       │                       │
       └───────────┬───────────┘
                   │
            ┌──────┴──────┐
            │  Monitor    │  监控中心
            └─────────────┘
```

### 1.2 核心组件

```java
// 1. Protocol - 协议层
public interface Protocol {
    // 暴露服务
    <T> Exporter<T> export(Invoker<T> invoker) throws RpcException;
    
    // 引用服务
    <T> Invoker<T> refer(Class<T> type, URL url) throws RpcException;
}

// 2. Invoker - 调用者
public interface Invoker<T> extends Node {
    // 获取服务接口
    Class<T> getInterface();
    
    // 执行调用
    Result invoke(Invocation invocation) throws RpcException;
}

// 3. Exporter - 暴露者
public interface Exporter<T> {
    // 获取 Invoker
    Invoker<T> getInvoker();
    
    // 取消暴露
    void unexport();
}

// 4. Invocation - 调用信息
public interface Invocation {
    // 获取方法名
    String getMethodName();
    
    // 获取参数类型
    Class<?>[] getParameterTypes();
    
    // 获取参数值
    Object[] getArguments();
}

// 5. Result - 调用结果
public interface Result {
    // 获取返回值
    Object getValue();
    
    // 获取异常
    Throwable getException();
}
```

### 1.3 分层架构

```
┌─────────────────────────────────────┐
│         Service (业务层)              │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│         Config (配置层)               │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│         Proxy (代理层)                │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│         Registry (注册层)             │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│         Cluster (集群层)              │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│         Monitor (监控层)              │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│         Protocol (协议层)             │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│         Exchange (交换层)             │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│         Transport (传输层)            │
└─────────────────────────────────────┘
┌─────────────────────────────────────┐
│         Serialize (序列化层)          │
└─────────────────────────────────────┘
```

---

## 2. 服务暴露流程 🔥

### 2.1 服务暴露示例

```java
// 服务接口
public interface UserService {
    User getUser(Long id);
}

// 服务实现
@DubboService
public class UserServiceImpl implements UserService {
    @Override
    public User getUser(Long id) {
        return new User(id, "张三");
    }
}

// Spring Boot 配置
dubbo:
  application:
    name: dubbo-provider
  registry:
    address: nacos://127.0.0.1:8848
  protocol:
    name: dubbo
    port: 20880
```

### 2.2 服务暴露流程

```
1. Spring 容器启动
   ↓
2. 解析 @DubboService 注解
   ↓
3. 创建 ServiceConfig
   ↓
4. 调用 ServiceConfig.export()
   ↓
5. 创建 Invoker（代理服务实现类）
   ↓
6. 通过 Protocol 暴露服务
   ↓
7. 启动 Netty Server（监听端口）
   ↓
8. 注册服务到注册中心
   ↓
9. 服务暴露完成
```

### 2.3 ServiceConfig.export() 源码

```java
// ServiceConfig.export() - 暴露服务
public synchronized void export() {
    if (!shouldExport()) {
        return;
    }
    
    if (bootstrap == null) {
        bootstrap = DubboBootstrap.getInstance();
        bootstrap.initialize();
    }
    
    // 检查和更新配置
    checkAndUpdateSubConfigs();
    
    // 初始化元数据
    initServiceMetadata(provider);
    serviceMetadata.setServiceType(getInterfaceClass());
    serviceMetadata.setTarget(getRef());
    serviceMetadata.generateServiceKey();
    
    // 执行暴露
    doExport();
}

// ServiceConfig.doExport()
protected synchronized void doExport() {
    if (unexported) {
        throw new IllegalStateException("The service " + interfaceClass.getName() + " has already unexported!");
    }
    if (exported) {
        return;
    }
    exported = true;
    
    if (StringUtils.isEmpty(path)) {
        path = interfaceName;
    }
    
    // 暴露服务
    doExportUrls();
}

// ServiceConfig.doExportUrls()
private void doExportUrls() {
    // 加载注册中心 URL
    List<URL> registryURLs = ConfigValidationUtils.loadRegistries(this, true);
    
    // 遍历所有协议，分别暴露
    for (ProtocolConfig protocolConfig : protocols) {
        String pathKey = URL.buildKey(getContextPath(protocolConfig)
            .map(p -> p + "/" + path).orElse(path), group, version);
        
        // 暴露服务
        doExportUrlsFor1Protocol(protocolConfig, registryURLs);
    }
}

// ServiceConfig.doExportUrlsFor1Protocol() - 核心暴露逻辑
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
    String name = protocolConfig.getName();
    if (StringUtils.isEmpty(name)) {
        name = DUBBO;
    }
    
    // 构建服务 URL
    Map<String, String> map = new HashMap<>();
    map.put(SIDE_KEY, PROVIDER_SIDE);
    // ... 添加各种参数
    
    URL url = new URL(name, host, port, getContextPath(protocolConfig)
        .map(p -> p + "/" + path).orElse(path), map);
    
    // 1. 本地暴露（injvm 协议）
    if (!SCOPE_REMOTE.equalsIgnoreCase(scope)) {
        exportLocal(url);
    }
    
    // 2. 远程暴露
    if (!SCOPE_LOCAL.equalsIgnoreCase(scope)) {
        if (CollectionUtils.isNotEmpty(registryURLs)) {
            for (URL registryURL : registryURLs) {
                // 创建 Invoker
                Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, 
                    registryURL.addParameterAndEncoded(EXPORT_KEY, url.toFullString()));
                
                // 包装 Invoker（添加过滤器链）
                DelegateProviderMetaDataInvoker wrapperInvoker = 
                    new DelegateProviderMetaDataInvoker(invoker, this);
                
                // 通过 Protocol 暴露服务
                Exporter<?> exporter = PROTOCOL.export(wrapperInvoker);
                exporters.add(exporter);
            }
        }
    }
}
```

### 2.4 Protocol.export() 源码

```java
// RegistryProtocol.export() - 注册中心协议暴露
@Override
public <T> Exporter<T> export(final Invoker<T> originInvoker) throws RpcException {
    // 获取注册中心 URL
    URL registryUrl = getRegistryUrl(originInvoker);
    // 获取服务提供者 URL
    URL providerUrl = getProviderUrl(originInvoker);
    
    // 1. 通过 DubboProtocol 暴露服务（启动 Netty Server）
    final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker, providerUrl);
    
    // 2. 获取注册中心
    final Registry registry = getRegistry(registryUrl);
    
    // 3. 注册服务到注册中心
    final URL registeredProviderUrl = getUrlToRegistry(providerUrl, registryUrl);
    registry.register(registeredProviderUrl);
    
    // 4. 订阅 override 配置
    registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);
    
    return new DestroyableExporter<>(exporter);
}

// DubboProtocol.export() - Dubbo 协议暴露
@Override
public <T> Exporter<T> export(Invoker<T> invoker) throws RpcException {
    URL url = invoker.getUrl();
    
    // 生成服务 key：group/interface:version:port
    String key = serviceKey(url);
    
    // 创建 Exporter
    DubboExporter<T> exporter = new DubboExporter<T>(invoker, key, exporterMap);
    exporterMap.put(key, exporter);
    
    // 启动 Server（Netty）
    openServer(url);
    
    // 优化序列化
    optimizeSerialization(url);
    
    return exporter;
}

// DubboProtocol.openServer() - 启动服务器
private void openServer(URL url) {
    String key = url.getAddress();
    boolean isServer = url.getParameter(IS_SERVER_KEY, true);
    if (isServer) {
        ProtocolServer server = serverMap.get(key);
        if (server == null) {
            synchronized (this) {
                server = serverMap.get(key);
                if (server == null) {
                    // 创建服务器
                    serverMap.put(key, createServer(url));
                }
            }
        } else {
            // 重置服务器
            server.reset(url);
        }
    }
}

// DubboProtocol.createServer() - 创建服务器
private ProtocolServer createServer(URL url) {
    // 默认使用 Netty
    url = url.addParameterIfAbsent(SERVER_KEY, getDefaultServer());
    
    // 创建 Exchanger（默认 HeaderExchanger）
    ExchangeServer server;
    try {
        server = Exchangers.bind(url, requestHandler);
    } catch (RemotingException e) {
        throw new RpcException("Fail to start server", e);
    }
    
    return new DubboProtocolServer(server);
}
```

---

## 3. 服务引用流程 🔥

### 3.1 服务引用示例

```java
// 服务消费者
@Service
public class OrderService {
    
    @DubboReference
    private UserService userService;
    
    public void createOrder(Long userId) {
        User user = userService.getUser(userId);
        // 创建订单逻辑
    }
}

// Spring Boot 配置
dubbo:
  application:
    name: dubbo-consumer
  registry:
    address: nacos://127.0.0.1:8848
```

### 3.2 服务引用流程

```
1. Spring 容器启动
   ↓
2. 解析 @DubboReference 注解
   ↓
3. 创建 ReferenceConfig
   ↓
4. 调用 ReferenceConfig.get()
   ↓
5. 从注册中心订阅服务
   ↓
6. 通过 Protocol 引用服务
   ↓
7. 创建 Invoker
   ↓
8. 创建代理对象（JDK 或 Javassist）
   ↓
9. 注入到 Spring Bean
   ↓
10. 服务引用完成
```

### 3.3 ReferenceConfig.get() 源码

```java
// ReferenceConfig.get() - 获取代理对象
public synchronized T get() {
    if (destroyed) {
        throw new IllegalStateException("The invoker of ReferenceConfig(" + url + ") has already destroyed!");
    }
    if (ref == null) {
        // 初始化
        init();
    }
    return ref;
}

// ReferenceConfig.init()
public synchronized void init() {
    if (initialized) {
        return;
    }
    
    if (bootstrap == null) {
        bootstrap = DubboBootstrap.getInstance();
        bootstrap.initialize();
    }
    
    // 检查和更新配置
    checkAndUpdateSubConfigs();
    
    // 初始化元数据
    initServiceMetadata(consumer);
    serviceMetadata.setServiceType(getInterfaceClass());
    
    // 创建代理
    ref = createProxy(map);
    
    initialized = true;
}

// ReferenceConfig.createProxy() - 创建代理
private T createProxy(Map<String, String> map) {
    // 是否本地引用
    if (shouldJvmRefer(map)) {
        // 本地引用（injvm 协议）
        URL url = new URL(LOCAL_PROTOCOL, LOCALHOST_VALUE, 0, interfaceClass.getName()).addParameters(map);
        invoker = REF_PROTOCOL.refer(interfaceClass, url);
    } else {
        // 远程引用
        urls.clear();
        
        // 用户指定 URL
        if (url != null && url.length() > 0) {
            String[] us = SEMICOLON_SPLIT_PATTERN.split(url);
            if (us != null && us.length > 0) {
                for (String u : us) {
                    URL url = URL.valueOf(u);
                    urls.add(url);
                }
            }
        } else {
            // 从注册中心获取 URL
            List<URL> us = ConfigValidationUtils.loadRegistries(this, false);
            if (CollectionUtils.isNotEmpty(us)) {
                for (URL u : us) {
                    URL monitorUrl = ConfigValidationUtils.loadMonitor(this, u);
                    if (monitorUrl != null) {
                        map.put(MONITOR_KEY, URL.encode(monitorUrl.toFullString()));
                    }
                    urls.add(u.addParameterAndEncoded(REFER_KEY, StringUtils.toQueryString(map)));
                }
            }
        }
        
        if (urls.size() == 1) {
            // 单个注册中心
            invoker = REF_PROTOCOL.refer(interfaceClass, urls.get(0));
        } else {
            // 多个注册中心
            List<Invoker<?>> invokers = new ArrayList<>();
            URL registryURL = null;
            for (URL url : urls) {
                invokers.add(REF_PROTOCOL.refer(interfaceClass, url));
                if (UrlUtils.isRegistry(url)) {
                    registryURL = url;
                }
            }
            if (registryURL != null) {
                // 使用 ZoneAwareCluster
                invoker = CLUSTER.join(new StaticDirectory(registryURL, invokers));
            } else {
                invoker = CLUSTER.join(new StaticDirectory(invokers));
            }
        }
    }
    
    // 创建代理对象
    return (T) PROXY_FACTORY.getProxy(invoker, ProtocolUtils.isGeneric(generic));
}
```

