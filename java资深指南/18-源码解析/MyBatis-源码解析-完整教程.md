# MyBatis 源码解析 - 完整教程

> 深入理解 MyBatis 的核心原理和执行流程
> 
> @author erik.zhou

## 📚 技术概述

| 项目 | 说明 |
|------|------|
| **框架名称** | MyBatis |
| **当前版本** | 3.5.x |
| **源码地址** | https://github.com/mybatis/mybatis-3 |
| **学习难度** | ⭐⭐⭐⭐ |
| **重要程度** | ⭐⭐⭐⭐⭐ |
| **预计时长** | 30-40 小时 |
| **前置知识** | JDBC、SQL、反射、动态代理 |

## 🎯 学习目标

- [ ] 理解 MyBatis 的整体架构
- [ ] 掌握配置文件的解析流程
- [ ] 深入理解 SQL 执行的完整流程
- [ ] 掌握一级缓存和二级缓存的实现
- [ ] 理解插件机制的原理
- [ ] 掌握结果集映射的实现
- [ ] 能够自定义 TypeHandler 和插件

## 📖 目录

1. [MyBatis 整体架构](#1-mybatis-整体架构)
2. [配置文件解析](#2-配置文件解析)
3. [SQL 执行流程](#3-sql-执行流程)
4. [缓存机制](#4-缓存机制)
5. [插件机制](#5-插件机制)
6. [结果集映射](#6-结果集映射)

---

## 1. MyBatis 整体架构

### 1.1 核心组件

```
mybatis-3/
├── SqlSessionFactoryBuilder  # 构建 SqlSessionFactory
├── SqlSessionFactory          # 创建 SqlSession
├── SqlSession                 # 执行 SQL 的会话
├── Executor                   # SQL 执行器
├── StatementHandler           # JDBC Statement 处理器
├── ParameterHandler           # 参数处理器
├── ResultSetHandler           # 结果集处理器
└── TypeHandler                # 类型处理器
```

### 1.2 四大对象 🔥

```java
// 1. Executor - 执行器
public interface Executor {
    ResultHandler NO_RESULT_HANDLER = null;
    
    int update(MappedStatement ms, Object parameter) throws SQLException;
    <E> List<E> query(MappedStatement ms, Object parameter, 
        RowBounds rowBounds, ResultHandler resultHandler) throws SQLException;
    void commit(boolean required) throws SQLException;
    void rollback(boolean required) throws SQLException;
}

// 2. StatementHandler - 语句处理器
public interface StatementHandler {
    Statement prepare(Connection connection, Integer transactionTimeout) throws SQLException;
    void parameterize(Statement statement) throws SQLException;
    <E> List<E> query(Statement statement, ResultHandler resultHandler) throws SQLException;
    int update(Statement statement) throws SQLException;
}

// 3. ParameterHandler - 参数处理器
public interface ParameterHandler {
    Object getParameterObject();
    void setParameters(PreparedStatement ps) throws SQLException;
}

// 4. ResultSetHandler - 结果集处理器
public interface ResultSetHandler {
    <E> List<E> handleResultSets(Statement stmt) throws SQLException;
    <E> Cursor<E> handleCursorResultSets(Statement stmt) throws SQLException;
    void handleOutputParameters(CallableStatement cs) throws SQLException;
}
```

### 1.3 执行流程概览

```
1. 创建 SqlSessionFactory
   ↓
2. 创建 SqlSession
   ↓
3. 获取 Mapper 代理对象
   ↓
4. 执行 Mapper 方法
   ↓
5. 通过 Executor 执行 SQL
   ↓
6. StatementHandler 处理 JDBC Statement
   ↓
7. ParameterHandler 设置参数
   ↓
8. 执行 SQL
   ↓
9. ResultSetHandler 处理结果集
   ↓
10. 返回结果
```

---

## 2. 配置文件解析

### 2.1 SqlSessionFactory 创建

```java
// 1. 使用 SqlSessionFactoryBuilder 构建
String resource = "mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = 
    new SqlSessionFactoryBuilder().build(inputStream);

// 2. SqlSessionFactoryBuilder.build() 源码
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
    try {
        // 创建 XML 配置构建器
        XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
        // 解析配置文件，返回 Configuration 对象
        return build(parser.parse());
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
        ErrorContext.instance().reset();
        try {
            inputStream.close();
        } catch (IOException e) {
            // Intentionally ignore. Prefer previous error.
        }
    }
}

public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
}
```

### 2.2 配置文件解析流程

```java
// XMLConfigBuilder.parse()
public Configuration parse() {
    if (parsed) {
        throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    // 解析 <configuration> 根节点
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
}

// 解析配置
private void parseConfiguration(XNode root) {
    try {
        // 1. 解析 <properties>
        propertiesElement(root.evalNode("properties"));
        
        // 2. 解析 <settings>
        Properties settings = settingsAsProperties(root.evalNode("settings"));
        loadCustomVfs(settings);
        loadCustomLogImpl(settings);
        
        // 3. 解析 <typeAliases>
        typeAliasesElement(root.evalNode("typeAliases"));
        
        // 4. 解析 <plugins>
        pluginElement(root.evalNode("plugins"));
        
        // 5. 解析 <objectFactory>
        objectFactoryElement(root.evalNode("objectFactory"));
        
        // 6. 解析 <objectWrapperFactory>
        objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
        
        // 7. 解析 <reflectorFactory>
        reflectorFactoryElement(root.evalNode("reflectorFactory"));
        
        // 8. 应用 settings
        settingsElement(settings);
        
        // 9. 解析 <environments>
        environmentsElement(root.evalNode("environments"));
        
        // 10. 解析 <databaseIdProvider>
        databaseIdProviderElement(root.evalNode("databaseIdProvider"));
        
        // 11. 解析 <typeHandlers>
        typeHandlerElement(root.evalNode("typeHandlers"));
        
        // 12. 解析 <mappers> 🔥
        mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
}
```

### 2.3 Mapper 文件解析 🔥

```java
// XMLMapperBuilder.parse()
public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
        // 解析 <mapper> 节点
        configurationElement(parser.evalNode("/mapper"));
        configuration.addLoadedResource(resource);
        // 绑定 Mapper 接口
        bindMapperForNamespace();
    }
    
    parsePendingResultMaps();
    parsePendingCacheRefs();
    parsePendingStatements();
}

// 解析 <mapper> 节点
private void configurationElement(XNode context) {
    try {
        String namespace = context.getStringAttribute("namespace");
        if (namespace == null || namespace.isEmpty()) {
            throw new BuilderException("Mapper's namespace cannot be empty");
        }
        builderAssistant.setCurrentNamespace(namespace);
        
        // 解析 <cache-ref>
        cacheRefElement(context.evalNode("cache-ref"));
        
        // 解析 <cache>
        cacheElement(context.evalNode("cache"));
        
        // 解析 <parameterMap>（已废弃）
        parameterMapElement(context.evalNodes("/mapper/parameterMap"));
        
        // 解析 <resultMap>
        resultMapElements(context.evalNodes("/mapper/resultMap"));
        
        // 解析 <sql>
        sqlElement(context.evalNodes("/mapper/sql"));
        
        // 解析 <select|insert|update|delete>
        buildStatementFromContext(context.evalNodes("select|insert|update|delete"));
    } catch (Exception e) {
        throw new BuilderException("Error parsing Mapper XML. The XML location is '" 
            + resource + "'. Cause: " + e, e);
    }
}
```

---

## 3. SQL 执行流程 🔥

### 3.1 获取 Mapper 代理对象

```java
// 1. 从 SqlSession 获取 Mapper
UserMapper userMapper = sqlSession.getMapper(UserMapper.class);

// 2. DefaultSqlSession.getMapper() 源码
@Override
public <T> T getMapper(Class<T> type) {
    return configuration.getMapper(type, this);
}

// 3. Configuration.getMapper()
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    return mapperRegistry.getMapper(type, sqlSession);
}

// 4. MapperRegistry.getMapper()
public <T> T getMapper(Class<T> type, SqlSession sqlSession) {
    // 从缓存获取 MapperProxyFactory
    final MapperProxyFactory<T> mapperProxyFactory = 
        (MapperProxyFactory<T>) knownMappers.get(type);
    if (mapperProxyFactory == null) {
        throw new BindingException("Type " + type + " is not known to the MapperRegistry.");
    }
    try {
        // 创建代理对象
        return mapperProxyFactory.newInstance(sqlSession);
    } catch (Exception e) {
        throw new BindingException("Error getting mapper instance. Cause: " + e, e);
    }
}

// 5. MapperProxyFactory.newInstance()
public T newInstance(SqlSession sqlSession) {
    // 创建 MapperProxy（InvocationHandler）
    final MapperProxy<T> mapperProxy = new MapperProxy<>(sqlSession, mapperInterface, methodCache);
    return newInstance(mapperProxy);
}

protected T newInstance(MapperProxy<T> mapperProxy) {
    // 使用 JDK 动态代理创建代理对象
    return (T) Proxy.newProxyInstance(
        mapperInterface.getClassLoader(), 
        new Class[] { mapperInterface }, 
        mapperProxy);
}
```

### 3.2 执行 Mapper 方法

```java
// MapperProxy.invoke() - 代理方法调用
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
        // 如果是 Object 的方法，直接执行
        if (Object.class.equals(method.getDeclaringClass())) {
            return method.invoke(this, args);
        } else {
            // 缓存 MapperMethod
            return cachedInvoker(method).invoke(proxy, method, args, sqlSession);
        }
    } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
    }
}

// MapperMethod.execute() - 执行 SQL
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {
        case INSERT: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.insert(command.getName(), param));
            break;
        }
        case UPDATE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.update(command.getName(), param));
            break;
        }
        case DELETE: {
            Object param = method.convertArgsToSqlCommandParam(args);
            result = rowCountResult(sqlSession.delete(command.getName(), param));
            break;
        }
        case SELECT:
            if (method.returnsVoid() && method.hasResultHandler()) {
                executeWithResultHandler(sqlSession, args);
                result = null;
            } else if (method.returnsMany()) {
                result = executeForMany(sqlSession, args);
            } else if (method.returnsMap()) {
                result = executeForMap(sqlSession, args);
            } else if (method.returnsCursor()) {
                result = executeForCursor(sqlSession, args);
            } else {
                Object param = method.convertArgsToSqlCommandParam(args);
                result = sqlSession.selectOne(command.getName(), param);
                if (method.returnsOptional() && 
                    (result == null || !method.getReturnType().equals(result.getClass()))) {
                    result = Optional.ofNullable(result);
                }
            }
            break;
        case FLUSH:
            result = sqlSession.flushStatements();
            break;
        default:
            throw new BindingException("Unknown execution method for: " + command.getName());
    }
    return result;
}
```

