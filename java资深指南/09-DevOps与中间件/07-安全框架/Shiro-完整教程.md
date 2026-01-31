# Apache Shiro 完整教程

## 📋 目录
- 技术概述
- 学习目标
- 基础概念
- 核心特性
- 实战应用
- 最佳实践
- 常见问题
- 相关资源
- 学习检查清单

## 📚 技术概述
- **版本**: 1.13.x
- **官方文档**: https://shiro.apache.org/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Java基础
  - Servlet规范
  - Web应用开发基础
  - 基本的安全概念
- **文档来源**: Apache Shiro官方文档
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Apache Shiro的核心架构和设计理念
- [ ] 掌握Shiro的认证(Authentication)机制
- [ ] 掌握Shiro的授权(Authorization)机制
- [ ] 熟练使用Realm进行自定义认证授权
- [ ] 掌握会话管理(Session Management)
- [ ] 理解Shiro的加密(Cryptography)功能
- [ ] 能够在实际项目中集成和使用Shiro

## 📖 基础概念

### 1.1 什么是Apache Shiro

Apache Shiro是一个功能强大且易于使用的Java安全框架,提供了认证、授权、加密和会话管理功能。Shiro的设计理念是简单易用,可以在任何环境中使用,不仅限于Web应用。

**核心优势**:
- **简单易用**: API设计直观,学习曲线平缓
- **灵活性强**: 可以在任何环境中使用(Web、桌面、移动等)
- **功能完整**: 提供认证、授权、会话管理、加密等完整功能
- **可扩展**: 支持自定义Realm、Filter等组件
- **轻量级**: 无重依赖,可以独立使用

### 1.2 核心概念

- **Subject**: 当前用户(可以是人或程序),代表安全操作的主体
- **SecurityManager**: 安全管理器,Shiro的核心,管理所有Subject
- **Realm**: 域,用于获取安全数据(用户、角色、权限)
- **Authentication**: 认证,验证用户身份的过程
- **Authorization**: 授权,控制用户访问权限的过程
- **Session**: 会话,用户与应用的交互状态
- **Cryptography**: 加密,数据加密和解密功能

### 1.3 Shiro架构

```
应用代码
    ↓
Subject (当前用户)
    ↓
SecurityManager (安全管理器)
    ↓
Realm (安全数据源)
    ↓
数据存储 (数据库、LDAP等)
```

### 1.4 应用场景
- Web应用的用户认证授权
- 桌面应用的安全控制
- 移动应用的后端安全
- 微服务的权限管理
- 企业内部系统的统一认证
- 需要简单安全框架的中小型项目

## 🔥 核心特性 (重点)

### 2.1 认证机制 🔥

认证是验证用户身份的过程,Shiro提供了简单而强大的认证API。

**认证流程**:
1. 收集用户凭证(用户名/密码)
2. 提交凭证给Subject
3. Subject委托给SecurityManager
4. SecurityManager调用Authenticator
5. Authenticator调用Realm获取认证信息
6. 比对凭证,返回认证结果

**基础认证示例**:
```java
// 1. 获取当前用户
Subject currentUser = SecurityUtils.getSubject();

// 2. 判断是否已认证
if (!currentUser.isAuthenticated()) {
    // 3. 创建认证令牌
    UsernamePasswordToken token = new UsernamePasswordToken("username", "password");
    
    // 4. 设置记住我
    token.setRememberMe(true);
    
    try {
        // 5. 执行登录
        currentUser.login(token);
        System.out.println("登录成功!");
    } catch (UnknownAccountException uae) {
        System.out.println("用户不存在: " + token.getPrincipal());
    } catch (IncorrectCredentialsException ice) {
        System.out.println("密码错误!");
    } catch (LockedAccountException lae) {
        System.out.println("账户已锁定!");
    } catch (AuthenticationException ae) {
        System.out.println("认证失败!");
    }
}

// 6. 获取用户信息
String username = (String) currentUser.getPrincipal();

// 7. 登出
currentUser.logout();
```

### 2.2 授权机制 🔥

授权是控制用户访问权限的过程,Shiro支持多种授权方式。

**授权方式**:
- **编程式授权**: 在代码中直接调用API
- **注解式授权**: 使用注解声明权限
- **JSP标签授权**: 在JSP页面中控制显示

**编程式授权**:
```java
Subject currentUser = SecurityUtils.getSubject();

// 1. 角色检查
if (currentUser.hasRole("admin")) {
    System.out.println("用户是管理员");
} else {
    System.out.println("用户不是管理员");
}

// 2. 多角色检查
boolean[] results = currentUser.hasRoles(Arrays.asList("admin", "user", "guest"));

// 3. 检查是否拥有所有角色
if (currentUser.hasAllRoles(Arrays.asList("admin", "user"))) {
    System.out.println("用户拥有所有指定角色");
}

// 4. 权限检查
if (currentUser.isPermitted("user:create")) {
    System.out.println("用户有创建用户的权限");
}

// 5. 多权限检查
boolean[] permitted = currentUser.isPermitted("user:create", "user:delete", "user:update");

// 6. 检查是否拥有所有权限
if (currentUser.isPermittedAll("user:create", "user:delete")) {
    System.out.println("用户拥有所有指定权限");
}
```

**注解式授权**:
```java
@RequiresAuthentication
public void updateAccount(Account account) {
    // 需要认证才能访问
}

@RequiresRoles("admin")
public void deleteUser(Long userId) {
    // 需要admin角色
}

@RequiresRoles(value = {"admin", "user"}, logical = Logical.OR)
public void viewPage() {
    // 需要admin或user角色
}

@RequiresPermissions("user:create")
public void createUser(User user) {
    // 需要user:create权限
}

@RequiresPermissions(value = {"user:view", "user:edit"}, logical = Logical.AND)
public void editUser(User user) {
    // 需要同时拥有user:view和user:edit权限
}

@RequiresGuest
public void signUp() {
    // 只有游客(未登录)可以访问
}

@RequiresUser
public void viewProfile() {
    // 已登录用户(包括记住我)可以访问
}
```

### 2.3 Realm配置 🔥 (⚠️ 难点)

Realm是Shiro获取安全数据的组件,是连接Shiro和数据源的桥梁。

**自定义Realm**:
```java
public class CustomRealm extends AuthorizingRealm {

    @Autowired
    private UserService userService;

    @Autowired
    private RoleService roleService;

    @Autowired
    private PermissionService permissionService;

    /**
     * 授权方法
     */
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        // 1. 获取用户名
        String username = (String) principals.getPrimaryPrincipal();
        
        // 2. 从数据库获取用户的角色和权限
        User user = userService.findByUsername(username);
        Set<String> roles = roleService.findRolesByUserId(user.getId());
        Set<String> permissions = permissionService.findPermissionsByUserId(user.getId());
        
        // 3. 创建授权信息
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        authorizationInfo.setRoles(roles);
        authorizationInfo.setStringPermissions(permissions);
        
        return authorizationInfo;
    }

    /**
     * 认证方法
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) 
            throws AuthenticationException {
        // 1. 获取用户名和密码
        String username = (String) token.getPrincipal();
        String password = new String((char[]) token.getCredentials());
        
        // 2. 从数据库查询用户
        User user = userService.findByUsername(username);
        
        // 3. 用户不存在
        if (user == null) {
            throw new UnknownAccountException("用户不存在: " + username);
        }
        
        // 4. 账户被锁定
        if (user.isLocked()) {
            throw new LockedAccountException("账户已锁定: " + username);
        }
        
        // 5. 创建认证信息
        // Shiro会自动进行密码比对
        SimpleAuthenticationInfo authenticationInfo = new SimpleAuthenticationInfo(
            username,                    // 用户名(principal)
            user.getPassword(),          // 密码(credentials)
            ByteSource.Util.bytes(user.getSalt()), // 盐值
            getName()                    // realm名称
        );
        
        return authenticationInfo;
    }
}
```

**配置Realm**:
```java
@Configuration
public class ShiroConfig {

    @Bean
    public CustomRealm customRealm() {
        CustomRealm realm = new CustomRealm();
        
        // 配置密码匹配器
        HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
        matcher.setHashAlgorithmName("SHA-256"); // 加密算法
        matcher.setHashIterations(1024);         // 加密次数
        matcher.setStoredCredentialsHexEncoded(true); // 十六进制编码
        
        realm.setCredentialsMatcher(matcher);
        return realm;
    }

    @Bean
    public SecurityManager securityManager(CustomRealm customRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(customRealm);
        return securityManager;
    }
}
```

### 2.4 会话管理 🔥

Shiro提供了完整的会话管理功能,不依赖于Web容器。

**会话管理特性**:
- 支持任何环境(Web、桌面、移动)
- 会话监听器
- 会话存储(内存、缓存、数据库)
- 会话超时控制
- 会话验证调度

**会话操作**:
```java
Subject currentUser = SecurityUtils.getSubject();
Session session = currentUser.getSession();

// 设置会话属性
session.setAttribute("key", "value");

// 获取会话属性
String value = (String) session.getAttribute("key");

// 删除会话属性
session.removeAttribute("key");

// 获取会话ID
Serializable sessionId = session.getId();

// 获取会话创建时间
Date startTime = session.getStartTimestamp();

// 获取最后访问时间
Date lastAccessTime = session.getLastAccessTime();

// 获取会话超时时间(毫秒)
long timeout = session.getTimeout();

// 设置会话超时时间
session.setTimeout(1800000); // 30分钟

// 停止会话
session.stop();
```

**自定义会话管理器**:
```java
@Bean
public SessionManager sessionManager() {
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
    
    // 设置会话超时时间(毫秒)
    sessionManager.setGlobalSessionTimeout(1800000); // 30分钟
    
    // 设置会话验证调度器
    sessionManager.setSessionValidationSchedulerEnabled(true);
    sessionManager.setSessionValidationInterval(900000); // 15分钟检查一次
    
    // 设置会话ID Cookie配置
    SimpleCookie sessionIdCookie = new SimpleCookie("SHIROSESSIONID");
    sessionIdCookie.setHttpOnly(true);
    sessionIdCookie.setMaxAge(-1); // 浏览器关闭时失效
    sessionManager.setSessionIdCookie(sessionIdCookie);
    
    // 禁用URL重写(不在URL中显示sessionId)
    sessionManager.setSessionIdUrlRewritingEnabled(false);
    
    return sessionManager;
}
```

### 2.5 加密功能 🔥

Shiro提供了强大的加密功能,支持多种加密算法。

**密码加密**:
```java
// 1. 使用SHA-256加密
String password = "123456";
String salt = new SecureRandomNumberGenerator().nextBytes().toHex();

SimpleHash hash = new SimpleHash(
    "SHA-256",           // 算法
    password,            // 原始密码
    salt,                // 盐值
    1024                 // 加密次数
);

String encryptedPassword = hash.toHex();

// 2. 使用BCrypt加密(推荐)
String bcryptPassword = new Bcrypt().hashpw(password, Bcrypt.gensalt());

// 3. 验证密码
boolean matches = Bcrypt.checkpw(password, bcryptPassword);
```

**数据加密解密**:
```java
// AES加密
AesCipherService aesCipherService = new AesCipherService();
aesCipherService.setKeySize(128); // 设置密钥长度

// 生成密钥
Key key = aesCipherService.generateNewKey();

// 加密
String plainText = "敏感数据";
ByteSource cipherText = aesCipherService.encrypt(
    plainText.getBytes(), 
    key.getEncoded()
);

// 解密
ByteSource decrypted = aesCipherService.decrypt(
    cipherText.getBytes(), 
    key.getEncoded()
);
String decryptedText = new String(decrypted.getBytes());
```

### 2.6 缓存管理

Shiro支持缓存来提高性能,减少数据库访问。

**配置缓存**:
```java
@Bean
public CacheManager cacheManager() {
    // 使用EhCache
    EhCacheManager cacheManager = new EhCacheManager();
    cacheManager.setCacheManagerConfigFile("classpath:ehcache.xml");
    return cacheManager;
    
    // 或使用Redis
    // RedisCacheManager cacheManager = new RedisCacheManager();
    // cacheManager.setRedisManager(redisManager());
    // return cacheManager;
}

@Bean
public SecurityManager securityManager(CustomRealm customRealm, 
                                       CacheManager cacheManager) {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    securityManager.setRealm(customRealm);
    securityManager.setCacheManager(cacheManager);
    return securityManager;
}
```

## 💻 实战应用

### 3.1 环境搭建

**Maven依赖**:
```xml
<dependencies>
    <!-- Shiro核心 -->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
        <version>1.13.0</version>
    </dependency>
    
    <!-- Shiro Web支持 -->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-web</artifactId>
        <version>1.13.0</version>
    </dependency>
    
    <!-- Shiro Spring集成 -->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-spring</artifactId>
        <version>1.13.0</version>
    </dependency>
    
    <!-- Shiro EhCache支持(可选) -->
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-ehcache</artifactId>
        <version>1.13.0</version>
    </dependency>
</dependencies>
```

**Gradle依赖**:
```gradle
dependencies {
    implementation 'org.apache.shiro:shiro-core:1.13.0'
    implementation 'org.apache.shiro:shiro-web:1.13.0'
    implementation 'org.apache.shiro:shiro-spring:1.13.0'
    implementation 'org.apache.shiro:shiro-ehcache:1.13.0'
}
```

### 3.2 快速开始

**Spring Boot集成配置**:
```java
@Configuration
public class ShiroConfig {

    /**
     * 自定义Realm
     */
    @Bean
    public CustomRealm customRealm() {
        CustomRealm realm = new CustomRealm();
        HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
        matcher.setHashAlgorithmName("SHA-256");
        matcher.setHashIterations(1024);
        realm.setCredentialsMatcher(matcher);
        return realm;
    }

    /**
     * 安全管理器
     */
    @Bean
    public SecurityManager securityManager(CustomRealm customRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        securityManager.setRealm(customRealm);
        return securityManager;
    }

    /**
     * Shiro过滤器
     */
    @Bean
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilter = new ShiroFilterFactoryBean();
        shiroFilter.setSecurityManager(securityManager);
        
        // 设置登录页面
        shiroFilter.setLoginUrl("/login");
        // 设置登录成功页面
        shiroFilter.setSuccessUrl("/index");
        // 设置未授权页面
        shiroFilter.setUnauthorizedUrl("/unauthorized");
        
        // 配置过滤器链
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        
        // 公开资源
        filterChainDefinitionMap.put("/login", "anon");
        filterChainDefinitionMap.put("/logout", "logout");
        filterChainDefinitionMap.put("/public/**", "anon");
        filterChainDefinitionMap.put("/static/**", "anon");
        
        // 需要认证的资源
        filterChainDefinitionMap.put("/user/**", "authc");
        
        // 需要特定角色的资源
        filterChainDefinitionMap.put("/admin/**", "roles[admin]");
        
        // 需要特定权限的资源
        filterChainDefinitionMap.put("/api/user/create", "perms[user:create]");
        filterChainDefinitionMap.put("/api/user/delete", "perms[user:delete]");
        
        // 其他资源需要认证
        filterChainDefinitionMap.put("/**", "authc");
        
        shiroFilter.setFilterChainDefinitionMap(filterChainDefinitionMap);
        
        return shiroFilter;
    }

    /**
     * 启用Shiro注解
     */
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(
            SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }

    /**
     * Shiro生命周期处理器
     */
    @Bean
    public LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }
}
```

**登录控制器**:
```java
@RestController
@RequestMapping("/api/auth")
public class AuthController {

    @PostMapping("/login")
    public Result login(@RequestBody LoginRequest request) {
        Subject subject = SecurityUtils.getSubject();
        
        try {
            UsernamePasswordToken token = new UsernamePasswordToken(
                request.getUsername(), 
                request.getPassword()
            );
            token.setRememberMe(request.isRememberMe());
            
            subject.login(token);
            
            return Result.success("登录成功");
        } catch (UnknownAccountException e) {
            return Result.error("用户不存在");
        } catch (IncorrectCredentialsException e) {
            return Result.error("密码错误");
        } catch (LockedAccountException e) {
            return Result.error("账户已锁定");
        } catch (AuthenticationException e) {
            return Result.error("认证失败");
        }
    }

    @PostMapping("/logout")
    public Result logout() {
        Subject subject = SecurityUtils.getSubject();
        subject.logout();
        return Result.success("登出成功");
    }

    @GetMapping("/info")
    @RequiresAuthentication
    public Result getUserInfo() {
        Subject subject = SecurityUtils.getSubject();
        String username = (String) subject.getPrincipal();
        
        // 获取用户角色和权限
        Session session = subject.getSession();
        
        Map<String, Object> userInfo = new HashMap<>();
        userInfo.put("username", username);
        userInfo.put("sessionId", session.getId());
        
        return Result.success(userInfo);
    }
}
```

### 3.3 进阶案例

**案例1: 动态权限控制**

**权限过滤器**:
```java
public class DynamicPermissionFilter extends AccessControlFilter {

    @Autowired
    private PermissionService permissionService;

    @Override
    protected boolean isAccessAllowed(ServletRequest request, 
                                      ServletResponse response, 
                                      Object mappedValue) throws Exception {
        Subject subject = getSubject(request, response);
        
        // 获取请求URI
        String requestURI = getPathWithinApplication(request);
        
        // 从数据库获取该URI需要的权限
        List<String> requiredPermissions = permissionService.getPermissionsByUri(requestURI);
        
        // 如果没有配置权限,则允许访问
        if (requiredPermissions == null || requiredPermissions.isEmpty()) {
            return true;
        }
        
        // 检查用户是否拥有所需权限
        for (String permission : requiredPermissions) {
            if (!subject.isPermitted(permission)) {
                return false;
            }
        }
        
        return true;
    }

    @Override
    protected boolean onAccessDenied(ServletRequest request, 
                                     ServletResponse response) throws Exception {
        Subject subject = getSubject(request, response);
        
        if (subject.getPrincipal() == null) {
            // 未登录,跳转到登录页
            saveRequestAndRedirectToLogin(request, response);
        } else {
            // 已登录但无权限,返回403
            HttpServletResponse httpResponse = (HttpServletResponse) response;
            httpResponse.setStatus(HttpServletResponse.SC_FORBIDDEN);
            httpResponse.getWriter().write("无权限访问");
        }
        
        return false;
    }
}
```

**案例2: 多Realm配置**

```java
@Configuration
public class MultiRealmConfig {

    @Bean
    public DatabaseRealm databaseRealm() {
        return new DatabaseRealm();
    }

    @Bean
    public LdapRealm ldapRealm() {
        return new LdapRealm();
    }

    @Bean
    public SecurityManager securityManager(DatabaseRealm databaseRealm, 
                                           LdapRealm ldapRealm) {
        DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
        
        // 配置多个Realm
        List<Realm> realms = new ArrayList<>();
        realms.add(databaseRealm);
        realms.add(ldapRealm);
        securityManager.setRealms(realms);
        
        // 配置认证策略
        ModularRealmAuthenticator authenticator = new ModularRealmAuthenticator();
        // 至少一个Realm认证成功即可
        authenticator.setAuthenticationStrategy(new AtLeastOneSuccessfulStrategy());
        // 或者所有Realm都要认证成功
        // authenticator.setAuthenticationStrategy(new AllSuccessfulStrategy());
        // 或者第一个Realm认证成功即可
        // authenticator.setAuthenticationStrategy(new FirstSuccessfulStrategy());
        
        securityManager.setAuthenticator(authenticator);
        
        return securityManager;
    }
}
```

**案例3: 记住我功能**

```java
@Bean
public SimpleCookie rememberMeCookie() {
    SimpleCookie cookie = new SimpleCookie("rememberMe");
    cookie.setHttpOnly(true);
    cookie.setMaxAge(2592000); // 30天
    return cookie;
}

@Bean
public CookieRememberMeManager rememberMeManager(SimpleCookie rememberMeCookie) {
    CookieRememberMeManager manager = new CookieRememberMeManager();
    manager.setCookie(rememberMeCookie);
    
    // 设置加密密钥
    byte[] cipherKey = Base64.decode("4AvVhmFLUs0KTA3Kprsdag==");
    manager.setCipherKey(cipherKey);
    
    return manager;
}

@Bean
public SecurityManager securityManager(CustomRealm customRealm,
                                       CookieRememberMeManager rememberMeManager) {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    securityManager.setRealm(customRealm);
    securityManager.setRememberMeManager(rememberMeManager);
    return securityManager;
}
```

## ✨ 最佳实践

### 4.1 性能优化

**1. 启用缓存**
```java
@Bean
public CacheManager cacheManager() {
    EhCacheManager cacheManager = new EhCacheManager();
    cacheManager.setCacheManagerConfigFile("classpath:ehcache-shiro.xml");
    return cacheManager;
}

@Bean
public CustomRealm customRealm(CacheManager cacheManager) {
    CustomRealm realm = new CustomRealm();
    realm.setCacheManager(cacheManager);
    
    // 启用认证缓存
    realm.setAuthenticationCachingEnabled(true);
    realm.setAuthenticationCacheName("authenticationCache");
    
    // 启用授权缓存
    realm.setAuthorizationCachingEnabled(true);
    realm.setAuthorizationCacheName("authorizationCache");
    
    return realm;
}
```

**EhCache配置(ehcache-shiro.xml)**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ehcache>
    <diskStore path="java.io.tmpdir/shiro-ehcache"/>
    
    <!-- 认证缓存 -->
    <cache name="authenticationCache"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="3600"
           timeToLiveSeconds="0"
           overflowToDisk="false"
           statistics="true"/>
    
    <!-- 授权缓存 -->
    <cache name="authorizationCache"
           maxEntriesLocalHeap="2000"
           eternal="false"
           timeToIdleSeconds="3600"
           timeToLiveSeconds="0"
           overflowToDisk="false"
           statistics="true"/>
</ehcache>
```

**2. 会话优化**
```java
@Bean
public SessionManager sessionManager() {
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
    
    // 设置会话超时时间
    sessionManager.setGlobalSessionTimeout(1800000); // 30分钟
    
    // 删除无效会话
    sessionManager.setDeleteInvalidSessions(true);
    
    // 设置会话验证调度器
    sessionManager.setSessionValidationSchedulerEnabled(true);
    sessionManager.setSessionValidationInterval(900000); // 15分钟
    
    // 禁用URL重写
    sessionManager.setSessionIdUrlRewritingEnabled(false);
    
    return sessionManager;
}
```

**3. 密码加密强度**
```java
@Bean
public HashedCredentialsMatcher hashedCredentialsMatcher() {
    HashedCredentialsMatcher matcher = new HashedCredentialsMatcher();
    
    // 使用SHA-256算法
    matcher.setHashAlgorithmName("SHA-256");
    
    // 设置加密次数(越高越安全,但性能越低)
    matcher.setHashIterations(1024);
    
    // 使用十六进制编码
    matcher.setStoredCredentialsHexEncoded(true);
    
    return matcher;
}
```

### 4.2 常见陷阱

**⚠️ 陷阱1: 权限字符串格式错误**

错误示例:
```java
// 错误: 权限字符串格式不规范
@RequiresPermissions("userCreate")
public void createUser(User user) { }
```

正确示例:
```java
// 正确: 使用冒号分隔的格式 资源:操作
@RequiresPermissions("user:create")
public void createUser(User user) { }

// 更细粒度的权限
@RequiresPermissions("user:create:admin")
public void createAdminUser(User user) { }
```

**⚠️ 陷阱2: Realm中的密码比对错误**

错误示例:
```java
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) 
        throws AuthenticationException {
    String username = (String) token.getPrincipal();
    String password = new String((char[]) token.getCredentials());
    
    User user = userService.findByUsername(username);
    
    // 错误: 手动比对密码
    if (!user.getPassword().equals(password)) {
        throw new IncorrectCredentialsException();
    }
    
    return new SimpleAuthenticationInfo(username, user.getPassword(), getName());
}
```

正确示例:
```java
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) 
        throws AuthenticationException {
    String username = (String) token.getPrincipal();
    User user = userService.findByUsername(username);
    
    if (user == null) {
        throw new UnknownAccountException();
    }
    
    // 正确: 返回认证信息,让Shiro自动比对密码
    return new SimpleAuthenticationInfo(
        username,
        user.getPassword(),
        ByteSource.Util.bytes(user.getSalt()),
        getName()
    );
}
```

**⚠️ 陷阱3: 过滤器链顺序错误**

错误示例:
```java
// 错误: 顺序不当,导致规则不生效
filterChainDefinitionMap.put("/**", "authc");
filterChainDefinitionMap.put("/login", "anon");
filterChainDefinitionMap.put("/public/**", "anon");
```

正确示例:
```java
// 正确: 具体路径在前,通配符在后
filterChainDefinitionMap.put("/login", "anon");
filterChainDefinitionMap.put("/public/**", "anon");
filterChainDefinitionMap.put("/**", "authc");
```

**⚠️ 陷阱4: 忘记清除缓存**

错误示例:
```java
// 错误: 修改用户权限后未清除缓存
public void updateUserRoles(Long userId, List<Long> roleIds) {
    userRoleService.updateUserRoles(userId, roleIds);
    // 缓存中的权限信息未更新
}
```

正确示例:
```java
// 正确: 修改权限后清除缓存
public void updateUserRoles(Long userId, List<Long> roleIds) {
    userRoleService.updateUserRoles(userId, roleIds);
    
    // 清除授权缓存
    User user = userService.findById(userId);
    SimplePrincipalCollection principals = new SimplePrincipalCollection(
        user.getUsername(), 
        "customRealm"
    );
    
    SecurityManager securityManager = SecurityUtils.getSecurityManager();
    if (securityManager instanceof DefaultWebSecurityManager) {
        DefaultWebSecurityManager webSecurityManager = 
            (DefaultWebSecurityManager) securityManager;
        Realm realm = webSecurityManager.getRealms().iterator().next();
        if (realm instanceof AuthorizingRealm) {
            ((AuthorizingRealm) realm).clearCachedAuthorizationInfo(principals);
        }
    }
}
```

**⚠️ 陷阱5: Session并发问题**

错误示例:
```java
// 错误: 未限制并发会话数
@Bean
public SecurityManager securityManager(CustomRealm customRealm) {
    DefaultWebSecurityManager securityManager = new DefaultWebSecurityManager();
    securityManager.setRealm(customRealm);
    return securityManager;
}
```

正确示例:
```java
// 正确: 限制并发会话数
@Bean
public SessionManager sessionManager() {
    DefaultWebSessionManager sessionManager = new DefaultWebSessionManager();
    
    // 配置会话监听器
    Collection<SessionListener> listeners = new ArrayList<>();
    listeners.add(new SessionListener() {
        @Override
        public void onStart(Session session) {
            // 会话创建时的处理
        }
        
        @Override
        public void onStop(Session session) {
            // 会话停止时的处理
        }
        
        @Override
        public void onExpiration(Session session) {
            // 会话过期时的处理
        }
    });
    sessionManager.setSessionListeners(listeners);
    
    return sessionManager;
}
```

### 4.3 安全建议

**1. 使用HTTPS**
```yaml
# application.yml
server:
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: password
    key-store-type: PKCS12
```

**2. 密码加盐**
```java
// 注册时生成盐值
public void register(User user) {
    // 生成随机盐值
    String salt = new SecureRandomNumberGenerator().nextBytes().toHex();
    user.setSalt(salt);
    
    // 加密密码
    SimpleHash hash = new SimpleHash(
        "SHA-256",
        user.getPassword(),
        salt,
        1024
    );
    user.setPassword(hash.toHex());
    
    userRepository.save(user);
}
```

**3. 登录失败次数限制**
```java
@Component
public class LoginRetryLimit {

    private Cache<String, AtomicInteger> passwordRetryCache;

    public LoginRetryLimit() {
        passwordRetryCache = CacheBuilder.newBuilder()
            .expireAfterWrite(10, TimeUnit.MINUTES)
            .build();
    }

    public void recordFailure(String username) {
        AtomicInteger retryCount = passwordRetryCache.getIfPresent(username);
        if (retryCount == null) {
            retryCount = new AtomicInteger(0);
            passwordRetryCache.put(username, retryCount);
        }
        retryCount.incrementAndGet();
    }

    public boolean isLocked(String username) {
        AtomicInteger retryCount = passwordRetryCache.getIfPresent(username);
        return retryCount != null && retryCount.get() >= 5;
    }

    public void clearRetry(String username) {
        passwordRetryCache.invalidate(username);
    }
}
```

**4. 审计日志**
```java
@Aspect
@Component
public class ShiroAuditAspect {

    @Autowired
    private AuditLogService auditLogService;

    @Around("@annotation(requiresPermissions)")
    public Object logPermissionCheck(ProceedingJoinPoint joinPoint, 
                                     RequiresPermissions requiresPermissions) 
            throws Throwable {
        Subject subject = SecurityUtils.getSubject();
        String username = (String) subject.getPrincipal();
        String method = joinPoint.getSignature().toShortString();
        String[] permissions = requiresPermissions.value();
        
        AuditLog log = new AuditLog();
        log.setUsername(username);
        log.setAction(method);
        log.setPermissions(Arrays.toString(permissions));
        log.setTimestamp(LocalDateTime.now());
        
        try {
            Object result = joinPoint.proceed();
            log.setSuccess(true);
            return result;
        } catch (Exception e) {
            log.setSuccess(false);
            log.setErrorMessage(e.getMessage());
            throw e;
        } finally {
            auditLogService.save(log);
        }
    }
}
```

## ❓ 常见问题

### Q1: Shiro与Spring Security的区别是什么?

A: 主要区别:

| 特性 | Shiro | Spring Security |
|------|-------|-----------------|
| 学习曲线 | 简单易学 | 相对复杂 |
| 依赖性 | 独立,可在任何环境使用 | 与Spring深度集成 |
| 功能完整性 | 基础功能完善 | 功能更全面 |
| 社区支持 | 活跃但较小 | 非常活跃 |
| OAuth2支持 | 需要额外集成 | 原生支持 |
| 适用场景 | 中小型项目 | 企业级大型项目 |

**选择建议**:
- 项目简单,需要快速上手: 选择Shiro
- 企业级项目,需要完整的安全方案: 选择Spring Security
- 非Spring项目: 选择Shiro
- 需要OAuth2/JWT: 优先考虑Spring Security

### Q2: 如何实现单点登录(SSO)?

A: Shiro实现SSO的方案:

**方案1: 使用CAS集成**
```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-cas</artifactId>
    <version>1.13.0</version>
</dependency>
```

```java
@Bean
public CasRealm casRealm() {
    CasRealm realm = new CasRealm();
    realm.setCasServerUrlPrefix("https://cas.example.com/cas");
    realm.setCasService("http://app.example.com/cas");
    return realm;
}
```

**方案2: 自定义Token共享**
```java
// 使用Redis存储Token
@Component
public class SsoTokenManager {

    @Autowired
    private RedisTemplate<String, String> redisTemplate;

    public String createToken(String username) {
        String token = UUID.randomUUID().toString();
        redisTemplate.opsForValue().set(
            "sso:token:" + token, 
            username, 
            30, 
            TimeUnit.MINUTES
        );
        return token;
    }

    public String getUsernameByToken(String token) {
        return redisTemplate.opsForValue().get("sso:token:" + token);
    }

    public void removeToken(String token) {
        redisTemplate.delete("sso:token:" + token);
    }
}
```

### Q3: 如何实现动态权限加载?

A: 从数据库动态加载权限:

```java
@Service
public class DynamicPermissionService {

    @Autowired
    private PermissionRepository permissionRepository;

    /**
     * 刷新权限配置
     */
    public void refreshPermissions() {
        ShiroFilterFactoryBean shiroFilter = 
            SpringContextHolder.getBean(ShiroFilterFactoryBean.class);
        
        // 获取所有权限配置
        List<Permission> permissions = permissionRepository.findAll();
        
        // 构建过滤器链
        Map<String, String> filterChainDefinitionMap = new LinkedHashMap<>();
        for (Permission permission : permissions) {
            String url = permission.getUrl();
            String perms = permission.getPerms();
            
            if (StringUtils.hasText(perms)) {
                filterChainDefinitionMap.put(url, "perms[" + perms + "]");
            }
        }
        
        // 更新过滤器链
        shiroFilter.setFilterChainDefinitionMap(filterChainDefinitionMap);
        
        // 重新加载过滤器链
        AbstractShiroFilter shiroFilterInstance = null;
        try {
            shiroFilterInstance = (AbstractShiroFilter) shiroFilter.getObject();
            PathMatchingFilterChainResolver resolver = 
                (PathMatchingFilterChainResolver) shiroFilterInstance
                    .getFilterChainResolver();
            
            DefaultFilterChainManager manager = 
                (DefaultFilterChainManager) resolver.getFilterChainManager();
            
            // 清空旧的权限配置
            manager.getFilterChains().clear();
            
            // 重新构建
            shiroFilter.getFilterChainDefinitionMap().forEach((url, filter) -> {
                manager.createChain(url, filter);
            });
        } catch (Exception e) {
            throw new RuntimeException("刷新权限配置失败", e);
        }
    }
}
```

### Q4: 如何处理跨域请求?

A: 配置CORS过滤器:

```java
@Bean
public FilterRegistrationBean<CorsFilter> corsFilter() {
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    CorsConfiguration config = new CorsConfiguration();
    
    config.setAllowCredentials(true);
    config.addAllowedOriginPattern("*");
    config.addAllowedHeader("*");
    config.addAllowedMethod("*");
    
    source.registerCorsConfiguration("/**", config);
    
    FilterRegistrationBean<CorsFilter> bean = 
        new FilterRegistrationBean<>(new CorsFilter(source));
    bean.setOrder(Ordered.HIGHEST_PRECEDENCE);
    
    return bean;
}
```

### Q5: 如何实现前后端分离的认证?

A: 使用Token认证:

**自定义Token过滤器**:
```java
public class JwtAuthenticationFilter extends BasicHttpAuthenticationFilter {

    @Override
    protected boolean isAccessAllowed(ServletRequest request, 
                                      ServletResponse response, 
                                      Object mappedValue) {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String token = httpRequest.getHeader("Authorization");
        
        if (StringUtils.hasText(token)) {
            try {
                JwtToken jwtToken = new JwtToken(token);
                getSubject(request, response).login(jwtToken);
                return true;
            } catch (Exception e) {
                return false;
            }
        }
        
        return false;
    }

    @Override
    protected boolean onAccessDenied(ServletRequest request, 
                                     ServletResponse response) throws Exception {
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        httpResponse.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        httpResponse.setContentType("application/json;charset=utf-8");
        httpResponse.getWriter().write("{\"code\":401,\"message\":\"未授权\"}");
        return false;
    }
}
```

**JWT Realm**:
```java
public class JwtRealm extends AuthorizingRealm {

    @Autowired
    private JwtTokenProvider tokenProvider;

    @Autowired
    private UserService userService;

    @Override
    public boolean supports(AuthenticationToken token) {
        return token instanceof JwtToken;
    }

    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        String username = (String) principals.getPrimaryPrincipal();
        User user = userService.findByUsername(username);
        
        SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();
        authorizationInfo.setRoles(user.getRoles());
        authorizationInfo.setStringPermissions(user.getPermissions());
        
        return authorizationInfo;
    }

    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) 
            throws AuthenticationException {
        String jwt = (String) token.getCredentials();
        
        if (!tokenProvider.validateToken(jwt)) {
            throw new AuthenticationException("Token无效");
        }
        
        String username = tokenProvider.getUsernameFromToken(jwt);
        User user = userService.findByUsername(username);
        
        if (user == null) {
            throw new UnknownAccountException("用户不存在");
        }
        
        return new SimpleAuthenticationInfo(username, jwt, getName());
    }
}
```

## 🔗 相关资源

### 官方资源
- **官方文档**: https://shiro.apache.org/documentation.html
- **GitHub仓库**: https://github.com/apache/shiro
- **参考手册**: https://shiro.apache.org/reference.html
- **示例项目**: https://github.com/apache/shiro/tree/main/samples

### 推荐文章
- Apache Shiro架构详解
- Shiro与Spring Boot集成实战
- Shiro权限管理最佳实践
- Shiro源码分析系列

### 视频教程
- Apache Shiro入门到精通
- Shiro实战权限管理系统
- Shiro安全框架深度解析

### 相关技术
- **Spring Security**: 另一个流行的安全框架
- **CAS**: 单点登录解决方案
- **OAuth2**: 授权框架标准
- **JWT**: JSON Web Token
- **LDAP**: 企业目录服务

## 📝 学习检查清单

### 基础知识
- [ ] 理解Shiro的核心架构(Subject、SecurityManager、Realm)
- [ ] 掌握认证(Authentication)的基本流程
- [ ] 掌握授权(Authorization)的基本概念
- [ ] 了解Shiro的四大核心功能

### 核心功能
- [ ] 能够实现基于表单的登录认证
- [ ] 能够配置基于角色和权限的访问控制
- [ ] 掌握自定义Realm的实现
- [ ] 理解密码加密和验证机制

### 进阶特性
- [ ] 掌握Shiro注解的使用
- [ ] 能够配置过滤器链
- [ ] 理解会话管理机制
- [ ] 掌握缓存配置和使用

### 实战能力
- [ ] 能够实现完整的用户认证授权系统
- [ ] 能够实现动态权限控制
- [ ] 能够实现记住我功能
- [ ] 能够实现登录失败次数限制
- [ ] 能够实现前后端分离的Token认证

### 安全最佳实践
- [ ] 了解密码加密的最佳实践
- [ ] 掌握会话安全配置
- [ ] 了解常见的安全威胁和防护
- [ ] 掌握审计日志的实现

### 性能优化
- [ ] 了解Shiro的性能特点
- [ ] 掌握缓存优化策略
- [ ] 了解会话管理优化

---

**学习建议**:
1. 先理解Shiro的核心架构和设计理念
2. 通过简单示例掌握基本的认证授权
3. 深入学习Realm的自定义实现
4. 实践过滤器链和注解的使用
5. 关注安全最佳实践和性能优化
6. 对比学习Spring Security,了解各自优势

**下一步学习**:
- 深入学习Spring Security进行对比
- 研究CAS单点登录解决方案
- 学习OAuth2和JWT的集成
- 探索微服务架构下的统一认证方案

---

**文档版本**: v1.0  
**最后更新**: 2024-01-04  
**维护者**: @author erik.zhou  
**反馈**: 如有问题或建议,请提交Issue
