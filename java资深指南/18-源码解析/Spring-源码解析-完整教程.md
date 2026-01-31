# Spring 源码解析 - 完整教程

> 深入理解 Spring 框架的核心原理和设计思想
> 
> @author erik.zhou

## 📚 技术概述

| 项目 | 说明 |
|------|------|
| **框架名称** | Spring Framework |
| **当前版本** | 6.1.x (Spring Boot 3.2.x) |
| **源码地址** | https://github.com/spring-projects/spring-framework |
| **学习难度** | ⭐⭐⭐⭐⭐ |
| **重要程度** | ⭐⭐⭐⭐⭐ |
| **预计时长** | 40-60 小时 |
| **前置知识** | Java 基础、设计模式、反射、代理 |

## 🎯 学习目标

- [ ] 理解 Spring IoC 容器的启动流程
- [ ] 掌握 Bean 的完整生命周期
- [ ] 深入理解 AOP 的实现原理
- [ ] 掌握 Spring 事务管理机制
- [ ] 理解 Spring MVC 的请求处理流程
- [ ] 学习 Spring 中的设计模式应用
- [ ] 能够解决 Spring 相关的复杂问题

## 📖 目录

1. [Spring 整体架构](#1-spring-整体架构)
2. [IoC 容器启动流程](#2-ioc-容器启动流程)
3. [Bean 生命周期](#3-bean-生命周期)
4. [依赖注入原理](#4-依赖注入原理)
5. [AOP 实现原理](#5-aop-实现原理)
6. [事务管理机制](#6-事务管理机制)
7. [Spring MVC 原理](#7-spring-mvc-原理)
8. [设计模式应用](#8-设计模式应用)

---

## 1. Spring 整体架构

### 1.1 核心模块

```
spring-framework/
├── spring-core          # 核心工具类
├── spring-beans         # Bean 定义和管理
├── spring-context       # 应用上下文
├── spring-aop           # AOP 实现
├── spring-aspects       # AspectJ 集成
├── spring-tx            # 事务管理
├── spring-web           # Web 基础
├── spring-webmvc        # Spring MVC
└── spring-jdbc          # JDBC 支持
```

### 1.2 核心概念

**IoC (Inversion of Control) - 控制反转**
- 对象的创建和管理由 Spring 容器负责
- 降低代码耦合度

**DI (Dependency Injection) - 依赖注入**
- 通过构造器、Setter、字段注入依赖
- IoC 的具体实现方式

**AOP (Aspect Oriented Programming) - 面向切面编程**
- 横切关注点的模块化
- 通过动态代理实现

**Bean**
- Spring 管理的对象
- 由 IoC 容器创建、配置和管理


### 1.3 核心接口

```java
// BeanFactory - Bean 工厂接口
public interface BeanFactory {
    Object getBean(String name) throws BeansException;
    <T> T getBean(String name, Class<T> requiredType) throws BeansException;
    boolean containsBean(String name);
    boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
}

// ApplicationContext - 应用上下文（BeanFactory 的子接口）
public interface ApplicationContext extends 
    EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
    MessageSource, ApplicationEventPublisher, ResourcePatternResolver {
    // 提供更多企业级功能
}

// BeanDefinition - Bean 定义
public interface BeanDefinition extends AttributeAccessor, BeanMetadataElement {
    String getBeanClassName();
    String getScope();
    boolean isSingleton();
    boolean isPrototype();
}
```

**BeanFactory vs ApplicationContext**

| 特性 | BeanFactory | ApplicationContext |
|------|-------------|-------------------|
| Bean 实例化 | 延迟加载 | 立即加载 |
| 国际化 | 不支持 | 支持 |
| 事件发布 | 不支持 | 支持 |
| AOP | 需手动配置 | 自动支持 |
| 使用场景 | 资源受限环境 | 企业应用（推荐） |

---

## 2. IoC 容器启动流程 🔥

### 2.1 启动流程概览

```java
// Spring Boot 启动入口
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}

// 底层调用 AbstractApplicationContext.refresh()
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // 1. 准备刷新上下文
        prepareRefresh();
        
        // 2. 获取 BeanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        
        // 3. 准备 BeanFactory
        prepareBeanFactory(beanFactory);
        
        try {
            // 4. 后置处理 BeanFactory
            postProcessBeanFactory(beanFactory);
            
            // 5. 调用 BeanFactory 后置处理器
            invokeBeanFactoryPostProcessors(beanFactory);
            
            // 6. 注册 Bean 后置处理器
            registerBeanPostProcessors(beanFactory);
            
            // 7. 初始化消息源
            initMessageSource();
            
            // 8. 初始化事件广播器
            initApplicationEventMulticaster();
            
            // 9. 刷新（模板方法，子类实现）
            onRefresh();
            
            // 10. 注册监听器
            registerListeners();
            
            // 11. 实例化所有非懒加载的单例 Bean
            finishBeanFactoryInitialization(beanFactory);
            
            // 12. 完成刷新
            finishRefresh();
        } catch (BeansException ex) {
            // 销毁已创建的单例 Bean
            destroyBeans();
            cancelRefresh(ex);
            throw ex;
        }
    }
}
```

### 2.2 核心步骤详解

#### 步骤 1: prepareRefresh() - 准备刷新

```java
protected void prepareRefresh() {
    // 设置启动时间
    this.startupDate = System.currentTimeMillis();
    // 设置关闭标志为 false
    this.closed.set(false);
    // 设置激活标志为 true
    this.active.set(true);
    
    // 初始化属性源（可由子类覆盖）
    initPropertySources();
    
    // 验证必需的属性
    getEnvironment().validateRequiredProperties();
    
    // 存储预刷新的 ApplicationListener
    this.earlyApplicationListeners = new LinkedHashSet<>(this.applicationListeners);
    this.earlyApplicationEvents = new LinkedHashSet<>();
}
```

#### 步骤 2: obtainFreshBeanFactory() - 获取 BeanFactory

```java
protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {
    // 刷新 BeanFactory（创建或刷新）
    refreshBeanFactory();
    // 返回 BeanFactory
    return getBeanFactory();
}

// AbstractRefreshableApplicationContext 实现
protected final void refreshBeanFactory() throws BeansException {
    // 如果已存在 BeanFactory，先销毁
    if (hasBeanFactory()) {
        destroyBeans();
        closeBeanFactory();
    }
    
    try {
        // 创建 DefaultListableBeanFactory
        DefaultListableBeanFactory beanFactory = createBeanFactory();
        beanFactory.setSerializationId(getId());
        
        // 定制 BeanFactory（是否允许覆盖、循环依赖）
        customizeBeanFactory(beanFactory);
        
        // 加载 BeanDefinition
        loadBeanDefinitions(beanFactory);
        
        this.beanFactory = beanFactory;
    } catch (IOException ex) {
        throw new ApplicationContextException("I/O error parsing bean definition source", ex);
    }
}
```

#### 步骤 5: invokeBeanFactoryPostProcessors() - 调用 BeanFactory 后置处理器 🔥

```java
protected void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory) {
    // 委托给 PostProcessorRegistrationDelegate
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory, 
        getBeanFactoryPostProcessors());
}

// 核心逻辑
public static void invokeBeanFactoryPostProcessors(
        ConfigurableListableBeanFactory beanFactory, 
        List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
    
    // 1. 先处理 BeanDefinitionRegistryPostProcessor
    // 这个接口可以注册新的 BeanDefinition
    
    // 2. 再处理 BeanFactoryPostProcessor
    // 这个接口可以修改 BeanDefinition
    
    // 重要实现：ConfigurationClassPostProcessor
    // 负责处理 @Configuration、@ComponentScan、@Import 等注解
}
```

**ConfigurationClassPostProcessor 的作用**：
- 解析 `@Configuration` 类
- 处理 `@ComponentScan` 扫描包
- 处理 `@Import` 导入配置
- 处理 `@Bean` 方法
- 这是 Spring Boot 自动配置的核心！

#### 步骤 11: finishBeanFactoryInitialization() - 实例化单例 Bean 🔥

```java
protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    // 初始化转换服务
    if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME)) {
        beanFactory.setConversionService(
            beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
    }
    
    // 冻结配置（不再修改 BeanDefinition）
    beanFactory.freezeConfiguration();
    
    // 实例化所有非懒加载的单例 Bean
    beanFactory.preInstantiateSingletons();
}

// DefaultListableBeanFactory 实现
public void preInstantiateSingletons() throws BeansException {
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
    
    // 遍历所有 BeanDefinition
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        
        // 非抽象、单例、非懒加载
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            if (isFactoryBean(beanName)) {
                // 处理 FactoryBean
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
                // ...
            } else {
                // 普通 Bean，调用 getBean() 创建
                getBean(beanName);
            }
        }
    }
}
```

### 2.3 流程图

```
启动 Spring 应用
    ↓
prepareRefresh() - 准备刷新
    ↓
obtainFreshBeanFactory() - 创建 BeanFactory
    ↓
prepareBeanFactory() - 配置 BeanFactory
    ↓
invokeBeanFactoryPostProcessors() - 处理 @Configuration 等
    ↓
registerBeanPostProcessors() - 注册 Bean 后置处理器
    ↓
finishBeanFactoryInitialization() - 实例化所有单例 Bean
    ↓
finishRefresh() - 完成刷新，发布事件
    ↓
Spring 容器启动完成
```

---

## 3. Bean 生命周期 🔥

### 3.1 完整生命周期

```java
// Bean 生命周期的完整流程
1. 实例化 Bean (Instantiation)
   ↓
2. 设置属性 (Populate Properties)
   ↓
3. 调用 BeanNameAware.setBeanName()
   ↓
4. 调用 BeanFactoryAware.setBeanFactory()
   ↓
5. 调用 ApplicationContextAware.setApplicationContext()
   ↓
6. 调用 BeanPostProcessor.postProcessBeforeInitialization()
   ↓
7. 调用 @PostConstruct 注解的方法
   ↓
8. 调用 InitializingBean.afterPropertiesSet()
   ↓
9. 调用自定义的 init-method
   ↓
10. 调用 BeanPostProcessor.postProcessAfterInitialization()
   ↓
Bean 可以使用了
   ↓
容器关闭
   ↓
11. 调用 @PreDestroy 注解的方法
   ↓
12. 调用 DisposableBean.destroy()
   ↓
13. 调用自定义的 destroy-method
```


### 3.2 Bean 创建核心源码

```java
// AbstractAutowireCapableBeanFactory.doCreateBean()
protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
    // 1. 实例化 Bean
    BeanWrapper instanceWrapper = createBeanInstance(beanName, mbd, args);
    Object bean = instanceWrapper.getWrappedInstance();
    
    // 2. 允许后置处理器修改 BeanDefinition
    synchronized (mbd.postProcessingLock) {
        if (!mbd.postProcessed) {
            applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
            mbd.postProcessed = true;
        }
    }
    
    // 3. 提前暴露单例 Bean（解决循环依赖）
    boolean earlySingletonExposure = (mbd.isSingleton() && 
        this.allowCircularReferences && isSingletonCurrentlyInCreation(beanName));
    if (earlySingletonExposure) {
        addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
    }
    
    // 4. 填充属性（依赖注入）
    Object exposedObject = bean;
    populateBean(beanName, mbd, instanceWrapper);
    
    // 5. 初始化 Bean
    exposedObject = initializeBean(beanName, exposedObject, mbd);
    
    return exposedObject;
}

// 初始化 Bean
protected Object initializeBean(String beanName, Object bean, @Nullable RootBeanDefinition mbd) {
    // 调用 Aware 接口
    invokeAwareMethods(beanName, bean);
    
    // 调用 BeanPostProcessor.postProcessBeforeInitialization()
    Object wrappedBean = applyBeanPostProcessorsBeforeInitialization(bean, beanName);
    
    // 调用初始化方法
    invokeInitMethods(beanName, wrappedBean, mbd);
    
    // 调用 BeanPostProcessor.postProcessAfterInitialization()
    wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    
    return wrappedBean;
}
```

### 3.3 循环依赖解决 🔥

**什么是循环依赖？**
```java
@Service
public class A {
    @Autowired
    private B b;  // A 依赖 B
}

@Service
public class B {
    @Autowired
    private A a;  // B 依赖 A
}
```

**Spring 如何解决？**

使用三级缓存：

```java
public class DefaultSingletonBeanRegistry {
    // 一级缓存：完整的单例 Bean
    private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);
    
    // 二级缓存：早期的单例 Bean（已实例化，未初始化）
    private final Map<String, Object> earlySingletonObjects = new ConcurrentHashMap<>(16);
    
    // 三级缓存：单例工厂
    private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);
}

// 获取单例 Bean
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
    // 1. 从一级缓存获取
    Object singletonObject = this.singletonObjects.get(beanName);
    
    if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
        // 2. 从二级缓存获取
        singletonObject = this.earlySingletonObjects.get(beanName);
        
        if (singletonObject == null && allowEarlyReference) {
            synchronized (this.singletonObjects) {
                // 3. 从三级缓存获取
                ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
                if (singletonFactory != null) {
                    singletonObject = singletonFactory.getObject();
                    // 放入二级缓存
                    this.earlySingletonObjects.put(beanName, singletonObject);
                    // 从三级缓存移除
                    this.singletonFactories.remove(beanName);
                }
            }
        }
    }
    return singletonObject;
}
```

**循环依赖解决流程**：

```
1. 创建 A，实例化 A（未初始化）
2. 将 A 的工厂放入三级缓存
3. 填充 A 的属性，发现依赖 B
4. 创建 B，实例化 B（未初始化）
5. 将 B 的工厂放入三级缓存
6. 填充 B 的属性，发现依赖 A
7. 从三级缓存获取 A 的早期引用
8. B 初始化完成，放入一级缓存
9. A 获取到 B，继续初始化
10. A 初始化完成，放入一级缓存
```

**注意**：
- 只能解决单例 Bean 的循环依赖
- 不能解决构造器注入的循环依赖
- 不能解决 prototype 作用域的循环依赖

---

## 4. 依赖注入原理

### 4.1 注入方式

```java
// 1. 构造器注入（推荐）
@Service
public class UserService {
    private final UserRepository userRepository;
    
    @Autowired  // Spring 4.3+ 可省略
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// 2. Setter 注入
@Service
public class UserService {
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// 3. 字段注入（不推荐）
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

### 4.2 依赖注入源码

```java
// AbstractAutowireCapableBeanFactory.populateBean()
protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
    // 1. 给 InstantiationAwareBeanPostProcessor 机会修改 Bean
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
            if (!bp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
                return;
            }
        }
    }
    
    // 2. 获取属性值
    PropertyValues pvs = mbd.getPropertyValues();
    
    // 3. 自动装配（byName 或 byType）
    int resolvedAutowireMode = mbd.getResolvedAutowireMode();
    if (resolvedAutowireMode == AUTOWIRE_BY_NAME || resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
        MutablePropertyValues newPvs = new MutablePropertyValues(pvs);
        if (resolvedAutowireMode == AUTOWIRE_BY_NAME) {
            autowireByName(beanName, mbd, bw, newPvs);
        }
        if (resolvedAutowireMode == AUTOWIRE_BY_TYPE) {
            autowireByType(beanName, mbd, bw, newPvs);
        }
        pvs = newPvs;
    }
    
    // 4. 处理 @Autowired、@Resource 等注解
    // 由 AutowiredAnnotationBeanPostProcessor 处理
    if (hasInstantiationAwareBeanPostProcessors()) {
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
            PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
            if (pvsToUse == null) {
                return;
            }
            pvs = pvsToUse;
        }
    }
    
    // 5. 应用属性值
    applyPropertyValues(beanName, mbd, bw, pvs);
}
```

### 4.3 @Autowired 处理

```java
// AutowiredAnnotationBeanPostProcessor
public class AutowiredAnnotationBeanPostProcessor implements 
    InstantiationAwareBeanPostProcessor, BeanFactoryAware {
    
    @Override
    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
        // 1. 查找需要注入的元数据（字段、方法）
        InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
        
        // 2. 执行注入
        metadata.inject(bean, beanName, pvs);
        
        return pvs;
    }
    
    // 查找 @Autowired 注解的字段和方法
    private InjectionMetadata findAutowiringMetadata(String beanName, Class<?> clazz, 
            @Nullable PropertyValues pvs) {
        // 从缓存获取
        InjectionMetadata metadata = this.injectionMetadataCache.get(cacheKey);
        if (metadata == null) {
            // 构建注入元数据
            metadata = buildAutowiringMetadata(clazz);
            this.injectionMetadataCache.put(cacheKey, metadata);
        }
        return metadata;
    }
    
    // 构建注入元数据
    private InjectionMetadata buildAutowiringMetadata(Class<?> clazz) {
        List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();
        Class<?> targetClass = clazz;
        
        do {
            List<InjectionMetadata.InjectedElement> currElements = new ArrayList<>();
            
            // 处理字段
            ReflectionUtils.doWithLocalFields(targetClass, field -> {
                MergedAnnotation<?> ann = findAutowiredAnnotation(field);
                if (ann != null) {
                    if (Modifier.isStatic(field.getModifiers())) {
                        return;  // 忽略静态字段
                    }
                    boolean required = determineRequiredStatus(ann);
                    currElements.add(new AutowiredFieldElement(field, required));
                }
            });
            
            // 处理方法
            ReflectionUtils.doWithLocalMethods(targetClass, method -> {
                Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
                if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
                    return;
                }
                MergedAnnotation<?> ann = findAutowiredAnnotation(bridgedMethod);
                if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
                    if (Modifier.isStatic(method.getModifiers())) {
                        return;  // 忽略静态方法
                    }
                    boolean required = determineRequiredStatus(ann);
                    PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
                    currElements.add(new AutowiredMethodElement(method, required, pd));
                }
            });
            
            elements.addAll(0, currElements);
            targetClass = targetClass.getSuperclass();
        }
        while (targetClass != null && targetClass != Object.class);
        
        return InjectionMetadata.forElements(elements, clazz);
    }
}
```

---

## 5. AOP 实现原理 🔥

### 5.1 AOP 核心概念

```java
// 切面（Aspect）
@Aspect
@Component
public class LogAspect {
    
    // 切点（Pointcut）
    @Pointcut("execution(* com.example.service.*.*(..))")
    public void serviceLayer() {}
    
    // 前置通知（Before Advice）
    @Before("serviceLayer()")
    public void before(JoinPoint joinPoint) {
        System.out.println("Before: " + joinPoint.getSignature());
    }
    
    // 后置通知（After Advice）
    @After("serviceLayer()")
    public void after(JoinPoint joinPoint) {
        System.out.println("After: " + joinPoint.getSignature());
    }
    
    // 返回通知（AfterReturning Advice）
    @AfterReturning(pointcut = "serviceLayer()", returning = "result")
    public void afterReturning(JoinPoint joinPoint, Object result) {
        System.out.println("AfterReturning: " + result);
    }
    
    // 异常通知（AfterThrowing Advice）
    @AfterThrowing(pointcut = "serviceLayer()", throwing = "ex")
    public void afterThrowing(JoinPoint joinPoint, Exception ex) {
        System.out.println("AfterThrowing: " + ex.getMessage());
    }
    
    // 环绕通知（Around Advice）
    @Around("serviceLayer()")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("Around Before");
        Object result = pjp.proceed();  // 执行目标方法
        System.out.println("Around After");
        return result;
    }
}
```

### 5.2 AOP 代理创建

```java
// AbstractAutoProxyCreator.postProcessAfterInitialization()
@Override
public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
    if (bean != null) {
        Object cacheKey = getCacheKey(bean.getClass(), beanName);
        if (this.earlyProxyReferences.remove(cacheKey) != bean) {
            // 如果需要代理，创建代理对象
            return wrapIfNecessary(bean, beanName, cacheKey);
        }
    }
    return bean;
}

protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
    // 1. 获取该 Bean 的所有 Advisor（增强器）
    Object[] specificInterceptors = getAdvicesAndAdvisorsForBean(
        bean.getClass(), beanName, null);
    
    // 2. 如果有 Advisor，创建代理
    if (specificInterceptors != DO_NOT_PROXY) {
        this.advisedBeans.put(cacheKey, Boolean.TRUE);
        // 创建代理对象
        Object proxy = createProxy(
            bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
        this.proxyTypes.put(cacheKey, proxy.getClass());
        return proxy;
    }
    
    this.advisedBeans.put(cacheKey, Boolean.FALSE);
    return bean;
}
```

