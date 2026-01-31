# Spring Security 完整教程

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
- **版本**: 6.2.x (Spring Security 6)
- **官方文档**: https://spring.io/projects/spring-security
- **学习难度**: ⭐⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Java基础
  - Spring Framework
  - Spring Boot
  - Servlet规范
  - HTTP协议基础
- **文档来源**: Context7 - Spring Security官方文档
- **更新时间**: 2024-01-04
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Spring Security的核心架构和工作原理
- [ ] 掌握认证(Authentication)和授权(Authorization)机制
- [ ] 熟练使用过滤器链(Filter Chain)进行安全配置
- [ ] 掌握OAuth2和JWT的集成与应用
- [ ] 理解方法级安全和权限表达式
- [ ] 能够在实际项目中实现完整的安全方案

## 📖 基础概念

### 1.1 什么是Spring Security

Spring Security是一个功能强大且高度可定制的身份验证和访问控制框架。它是保护基于Spring的应用程序的事实标准。

**核心功能**:
- **认证(Authentication)**: 验证用户身份
- **授权(Authorization)**: 控制用户访问权限
- **防护**: 防止常见攻击(CSRF、会话固定等)
- **集成**: 支持多种认证方式(表单、OAuth2、SAML等)

### 1.2 核心概念

- **SecurityContext**: 安全上下文,存储当前认证信息
- **Authentication**: 认证对象,包含用户凭证和权限
- **SecurityFilterChain**: 安全过滤器链,处理HTTP请求
- **UserDetails**: 用户详情接口,封装用户信息
- **GrantedAuthority**: 授权信息,表示用户权限
- **AuthenticationManager**: 认证管理器,处理认证请求

### 1.3 应用场景
- Web应用的用户登录认证
- RESTful API的安全保护
- 微服务架构的统一认证授权
- OAuth2社交登录集成
- 企业级权限管理系统
- 单点登录(SSO)解决方案

## 🔥 核心特性 (重点)

### 2.1 安全过滤器链 🔥 (⚠️ 难点)

Spring Security的核心是基于Servlet过滤器的安全过滤器链。每个HTTP请求都会经过一系列过滤器的处理。

**过滤器链架构**:
```
HTTP请求 
  ↓
DelegatingFilterProxy (Servlet容器)
  ↓
FilterChainProxy (Spring Security)
  ↓
SecurityFilterChain (多个安全过滤器)
  ↓
应用程序
```

**关键过滤器**:
- `SecurityContextPersistenceFilter`: 管理SecurityContext的持久化
- `UsernamePasswordAuthenticationFilter`: 处理表单登录
- `BasicAuthenticationFilter`: 处理HTTP Basic认证
- `ExceptionTranslationFilter`: 处理认证和授权异常
- `FilterSecurityInterceptor`: 执行访问控制决策

**配置示例**:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(Customizer.withDefaults())
            .httpBasic(Customizer.withDefaults())
            .formLogin(Customizer.withDefaults())
            .authorizeHttpRequests((authorize) -> authorize
                .anyRequest().authenticated()
            );
        return http.build();
    }
}
```

### 2.2 认证机制 🔥

认证是验证用户身份的过程。Spring Security支持多种认证方式。

**认证流程**:
1. 用户提交凭证(用户名/密码)
2. `AuthenticationManager`处理认证请求
3. `AuthenticationProvider`执行实际认证逻辑
4. 认证成功后创建`Authentication`对象
5. 将`Authentication`存入`SecurityContext`

**基础配置**:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests((authorize) -> authorize
                .requestMatchers("/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin((form) -> form
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .permitAll()
            )
            .logout((logout) -> logout
                .logoutSuccessUrl("/login?logout")
                .permitAll()
            );
        return http.build();
    }

    @Bean
    public UserDetailsService userDetailsService() {
        return new InMemoryUserDetailsManager(
            User.withDefaultPasswordEncoder()
                .username("user")
                .password("password")
                .roles("USER")
                .build(),
            User.withDefaultPasswordEncoder()
                .username("admin")
                .password("admin")
                .roles("USER", "ADMIN")
                .build()
        );
    }
}
```

### 2.3 授权机制 🔥

授权是控制用户访问资源权限的过程。

**授权方式**:
- **URL级别授权**: 基于请求路径的访问控制
- **方法级别授权**: 基于注解的方法安全
- **对象级别授权**: 基于领域对象的访问控制

**URL授权示例**:
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .authorizeHttpRequests((authorize) -> authorize
            .requestMatchers("/public/**").permitAll()
            .requestMatchers("/user/**").hasRole("USER")
            .requestMatchers("/admin/**").hasRole("ADMIN")
            .requestMatchers("/api/**").hasAuthority("SCOPE_read")
            .anyRequest().authenticated()
        );
    return http.build();
}
```

### 2.4 方法级安全 🔥 (⚠️ 难点)

使用注解在方法级别进行安全控制,提供更细粒度的权限管理。

**启用方法安全**:
```java
@Configuration
@EnableMethodSecurity
public class MethodSecurityConfig {
    // 方法安全已启用
}
```

**常用注解**:

**@PreAuthorize** - 方法执行前检查权限:
```java
@Service
public class DocumentService {

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteAll() {
        // 只有ADMIN角色可以执行
    }

    @PreAuthorize("hasAuthority('SCOPE_read')")
    public Document findById(Long id) {
        // 需要read权限
    }

    @PreAuthorize("hasRole('USER') and #username == authentication.name")
    public List<Document> findByUsername(String username) {
        // 用户只能查询自己的文档
    }
}
```

**@PostAuthorize** - 方法执行后检查权限:
```java
@Service
public class DocumentService {

    @PostAuthorize("returnObject.owner == authentication.name or hasRole('ADMIN')")
    public Document getDocument(Long id) {
        // 只能访问自己的文档或管理员可以访问所有文档
    }
}
```

**@PreFilter** - 过滤方法参数:
```java
@Service
public class DocumentService {

    @PreFilter("filterObject.owner == authentication.name or hasRole('ADMIN')")
    public void updateDocuments(List<Document> documents) {
        // 只处理用户拥有的文档
        documents.forEach(documentRepository::save);
    }
}
```

**@PostFilter** - 过滤返回结果:
```java
@Service
public class DocumentService {

    @PostFilter("filterObject.isPublic or filterObject.owner == authentication.name")
    public List<Document> findAll() {
        // 只返回公开的或用户拥有的文档
    }
}
```

### 2.5 权限表达式 (⚠️ 难点)

Spring Security使用SpEL(Spring Expression Language)表达式进行权限判断。

**常用表达式**:
- `hasRole('ROLE')`: 检查是否有指定角色
- `hasAuthority('AUTHORITY')`: 检查是否有指定权限
- `hasAnyRole('ROLE1', 'ROLE2')`: 检查是否有任意角色
- `hasAnyAuthority('AUTH1', 'AUTH2')`: 检查是否有任意权限
- `permitAll()`: 允许所有访问
- `denyAll()`: 拒绝所有访问
- `isAnonymous()`: 是否匿名用户
- `isAuthenticated()`: 是否已认证
- `isFullyAuthenticated()`: 是否完全认证(非记住我)
- `principal`: 当前用户对象
- `authentication`: 当前认证对象

**自定义权限表达式**:
```java
@Component("orderSecurity")
public class OrderSecurityExpression {

    @Autowired
    private OrderRepository orderRepository;

    public boolean canAccessOrder(Long orderId) {
        Authentication auth = SecurityContextHolder.getContext()
            .getAuthentication();
        Order order = orderRepository.findById(orderId).orElse(null);
        return order != null && order.getUsername()
            .equals(auth.getName());
    }
}

@Service
public class OrderService {

    @PreAuthorize("@orderSecurity.canAccessOrder(#orderId)")
    public Order getOrder(Long orderId) {
        return orderRepository.findById(orderId).orElse(null);
    }
}
```

### 2.6 OAuth2集成 🔥

Spring Security提供完整的OAuth2支持,包括授权服务器和资源服务器。

**OAuth2授权服务器配置**:
```java
@Configuration
@EnableWebSecurity
public class AuthorizationServerConfig {

    @Bean
    @Order(1)
    public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http)
            throws Exception {
        http
            .oauth2AuthorizationServer((authorizationServer) -> {
                http.securityMatcher(authorizationServer.getEndpointsMatcher());
                authorizationServer
                    .oidc(Customizer.withDefaults());  // 启用OpenID Connect
            })
            .authorizeHttpRequests((authorize) ->
                authorize.anyRequest().authenticated()
            )
            .exceptionHandling((exceptions) -> exceptions
                .defaultAuthenticationEntryPointFor(
                    new LoginUrlAuthenticationEntryPoint("/login"),
                    new MediaTypeRequestMatcher(MediaType.TEXT_HTML)
                )
            );
        return http.build();
    }

    @Bean
    public RegisteredClientRepository registeredClientRepository() {
        RegisteredClient oidcClient = RegisteredClient.withId(UUID.randomUUID().toString())
                .clientId("oidc-client")
                .clientSecret("{noop}secret")
                .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
                .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
                .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
                .redirectUri("http://127.0.0.1:8080/login/oauth2/code/oidc-client")
                .postLogoutRedirectUri("http://127.0.0.1:8080/")
                .scope(OidcScopes.OPENID)
                .scope(OidcScopes.PROFILE)
                .clientSettings(ClientSettings.builder().requireAuthorizationConsent(true).build())
                .build();
        return new InMemoryRegisteredClientRepository(oidcClient);
    }

    @Bean
    public JWKSource<SecurityContext> jwkSource() {
        KeyPair keyPair = generateRsaKey();
        RSAPublicKey publicKey = (RSAPublicKey) keyPair.getPublic();
        RSAPrivateKey privateKey = (RSAPrivateKey) keyPair.getPrivate();
        RSAKey rsaKey = new RSAKey.Builder(publicKey)
                .privateKey(privateKey)
                .keyID(UUID.randomUUID().toString())
                .build();
        JWKSet jwkSet = new JWKSet(rsaKey);
        return new ImmutableJWKSet<>(jwkSet);
    }

    private static KeyPair generateRsaKey() {
        KeyPair keyPair;
        try {
            KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("RSA");
            keyPairGenerator.initialize(2048);
            keyPair = keyPairGenerator.generateKeyPair();
        } catch (Exception ex) {
            throw new IllegalStateException(ex);
        }
        return keyPair;
    }

    @Bean
    public AuthorizationServerSettings authorizationServerSettings() {
        return AuthorizationServerSettings.builder().build();
    }
}
```

### 2.7 JWT资源服务器 🔥

配置资源服务器验证JWT令牌。

**基础JWT配置**:
```java
@Configuration
@EnableWebSecurity
public class ResourceServerConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests((authorize) -> authorize
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer((oauth2) -> oauth2
                .jwt(Customizer.withDefaults())
            );
        return http.build();
    }
}
```

**自定义JWT配置**:
```java
import static org.springframework.security.oauth2.core.authorization.OAuth2AuthorizationManagers.hasScope;

@Configuration
@EnableWebSecurity
public class CustomJwtSecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests((authorize) -> authorize
                .requestMatchers("/messages/**").access(hasScope("message:read"))
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer((oauth2) -> oauth2
                .jwt((jwt) -> jwt
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())
                )
            );
        return http.build();
    }

    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtGrantedAuthoritiesConverter grantedAuthoritiesConverter = 
            new JwtGrantedAuthoritiesConverter();
        grantedAuthoritiesConverter.setAuthorityPrefix("SCOPE_");
        
        JwtAuthenticationConverter jwtAuthenticationConverter = 
            new JwtAuthenticationConverter();
        jwtAuthenticationConverter.setJwtGrantedAuthoritiesConverter(
            grantedAuthoritiesConverter);
        return jwtAuthenticationConverter;
    }
}
```

## 💻 实战应用

### 3.1 环境搭建

**Maven依赖**:
```xml
<dependencies>
    <!-- Spring Boot Starter Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <!-- Spring Boot Starter Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <!-- OAuth2 Authorization Server (可选) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-authorization-server</artifactId>
    </dependency>
    
    <!-- OAuth2 Resource Server (可选) -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
    </dependency>
    
    <!-- JWT支持 -->
    <dependency>
        <groupId>com.nimbusds</groupId>
        <artifactId>nimbus-jose-jwt</artifactId>
    </dependency>
</dependencies>
```

**Gradle依赖**:
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-security'
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-authorization-server'
    implementation 'org.springframework.boot:spring-boot-starter-oauth2-resource-server'
    implementation 'com.nimbusds:nimbus-jose-jwt'
}
```

### 3.2 快速开始

**基础安全配置**:
```java
@Configuration
@EnableWebSecurity
public class BasicSecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/", "/home", "/public/**").permitAll()
                .requestMatchers("/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            .logout(logout -> logout
                .permitAll()
            );
        return http.build();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

**自定义UserDetailsService**:
```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) 
            throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException(
                "用户不存在: " + username));
        
        return org.springframework.security.core.userdetails.User
            .withUsername(user.getUsername())
            .password(user.getPassword())
            .authorities(getAuthorities(user))
            .accountExpired(false)
            .accountLocked(false)
            .credentialsExpired(false)
            .disabled(false)
            .build();
    }

    private Collection<? extends GrantedAuthority> getAuthorities(User user) {
        return user.getRoles().stream()
            .map(role -> new SimpleGrantedAuthority("ROLE_" + role.getName()))
            .collect(Collectors.toList());
    }
}
```

### 3.3 进阶案例

**案例1: JWT认证实现**

**JWT工具类**:
```java
@Component
public class JwtTokenProvider {

    @Value("${jwt.secret}")
    private String jwtSecret;

    @Value("${jwt.expiration}")
    private long jwtExpirationMs;

    public String generateToken(Authentication authentication) {
        UserDetails userDetails = (UserDetails) authentication.getPrincipal();
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + jwtExpirationMs);

        return Jwts.builder()
            .setSubject(userDetails.getUsername())
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .signWith(SignatureAlgorithm.HS512, jwtSecret)
            .compact();
    }

    public String getUsernameFromToken(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey(jwtSecret)
            .parseClaimsJws(token)
            .getBody();
        return claims.getSubject();
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parser().setSigningKey(jwtSecret).parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }
}
```

**JWT认证过滤器**:
```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtTokenProvider tokenProvider;

    @Autowired
    private CustomUserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {
        try {
            String jwt = getJwtFromRequest(request);

            if (StringUtils.hasText(jwt) && tokenProvider.validateToken(jwt)) {
                String username = tokenProvider.getUsernameFromToken(jwt);
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                
                UsernamePasswordAuthenticationToken authentication =
                    new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                authentication.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request));

                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        } catch (Exception ex) {
            logger.error("无法设置用户认证", ex);
        }

        filterChain.doFilter(request, response);
    }

    private String getJwtFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```

**JWT安全配置**:
```java
@Configuration
@EnableWebSecurity
public class JwtSecurityConfig {

    @Autowired
    private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;

    @Bean
    public JwtAuthenticationFilter jwtAuthenticationFilter() {
        return new JwtAuthenticationFilter();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .exceptionHandling(exception -> exception
                .authenticationEntryPoint(jwtAuthenticationEntryPoint)
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .anyRequest().authenticated()
            );

        http.addFilterBefore(jwtAuthenticationFilter(), 
            UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration authConfig) throws Exception {
        return authConfig.getAuthenticationManager();
    }
}
```

**案例2: 基于数据库的动态权限控制**

**权限实体**:
```java
@Entity
@Table(name = "permissions")
public class Permission {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String resource;
    private String action;
    
    // getters and setters
}

@Entity
@Table(name = "roles")
public class Role {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "role_permissions",
        joinColumns = @JoinColumn(name = "role_id"),
        inverseJoinColumns = @JoinColumn(name = "permission_id")
    )
    private Set<Permission> permissions = new HashSet<>();
    
    // getters and setters
}

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String username;
    private String password;
    
    @ManyToMany(fetch = FetchType.EAGER)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    // getters and setters
}
```

**动态权限加载**:
```java
@Service
public class DynamicUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) 
            throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new UsernameNotFoundException(
                "用户不存在: " + username));
        
        Set<GrantedAuthority> authorities = new HashSet<>();
        
        // 添加角色
        user.getRoles().forEach(role -> {
            authorities.add(new SimpleGrantedAuthority("ROLE_" + role.getName()));
            
            // 添加权限
            role.getPermissions().forEach(permission -> {
                authorities.add(new SimpleGrantedAuthority(
                    permission.getResource() + ":" + permission.getAction()));
            });
        });
        
        return new org.springframework.security.core.userdetails.User(
            user.getUsername(),
            user.getPassword(),
            authorities
        );
    }
}
```

**案例3: 记住我功能**

```java
@Configuration
@EnableWebSecurity
public class RememberMeSecurityConfig {

    @Autowired
    private DataSource dataSource;

    @Autowired
    private UserDetailsService userDetailsService;

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .anyRequest().authenticated()
            )
            .formLogin(form -> form
                .loginPage("/login")
                .permitAll()
            )
            .rememberMe(remember -> remember
                .key("uniqueAndSecret")
                .tokenRepository(persistentTokenRepository())
                .tokenValiditySeconds(86400) // 24小时
                .userDetailsService(userDetailsService)
            );
        return http.build();
    }

    @Bean
    public PersistentTokenRepository persistentTokenRepository() {
        JdbcTokenRepositoryImpl tokenRepository = new JdbcTokenRepositoryImpl();
        tokenRepository.setDataSource(dataSource);
        // 首次启动时创建表
        // tokenRepository.setCreateTableOnStartup(true);
        return tokenRepository;
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

**1. 密码加密策略**
```java
@Bean
public PasswordEncoder passwordEncoder() {
    // 使用BCrypt,自动加盐
    return new BCryptPasswordEncoder(10); // 强度10
}

// 或使用委托密码编码器支持多种算法
@Bean
public PasswordEncoder passwordEncoder() {
    String idForEncode = "bcrypt";
    Map<String, PasswordEncoder> encoders = new HashMap<>();
    encoders.put(idForEncode, new BCryptPasswordEncoder());
    encoders.put("pbkdf2", Pbkdf2PasswordEncoder.defaultsForSpringSecurity_v5_8());
    encoders.put("scrypt", SCryptPasswordEncoder.defaultsForSpringSecurity_v5_8());
    
    return new DelegatingPasswordEncoder(idForEncode, encoders);
}
```

**2. 会话管理**
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .sessionManagement(session -> session
            .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
            .maximumSessions(1) // 同一用户最多1个会话
            .maxSessionsPreventsLogin(true) // 阻止新登录
            .expiredUrl("/login?expired")
        );
    return http.build();
}
```

**3. CSRF保护**
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .csrf(csrf -> csrf
            .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            .ignoringRequestMatchers("/api/**") // API接口可以忽略CSRF
        );
    return http.build();
}
```

### 4.2 常见陷阱

**⚠️ 陷阱1: 过滤器链顺序错误**

错误示例:
```java
// 错误: 自定义过滤器位置不当
http.addFilterAfter(jwtFilter, UsernamePasswordAuthenticationFilter.class);
```

正确示例:
```java
// 正确: JWT过滤器应该在UsernamePasswordAuthenticationFilter之前
http.addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
```

**⚠️ 陷阱2: 密码明文存储**

错误示例:
```java
// 错误: 密码明文存储
user.setPassword(password);
userRepository.save(user);
```

正确示例:
```java
// 正确: 使用PasswordEncoder加密
@Autowired
private PasswordEncoder passwordEncoder;

user.setPassword(passwordEncoder.encode(password));
userRepository.save(user);
```

**⚠️ 陷阱3: 权限表达式错误**

错误示例:
```java
// 错误: 角色名称应该不包含ROLE_前缀
@PreAuthorize("hasRole('ROLE_ADMIN')")
public void deleteUser(Long id) { }
```

正确示例:
```java
// 正确: hasRole会自动添加ROLE_前缀
@PreAuthorize("hasRole('ADMIN')")
public void deleteUser(Long id) { }

// 或使用hasAuthority
@PreAuthorize("hasAuthority('ROLE_ADMIN')")
public void deleteUser(Long id) { }
```

**⚠️ 陷阱4: SecurityContext传播问题**

错误示例:
```java
// 错误: 异步线程无法获取SecurityContext
@Async
public void processAsync() {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    // auth可能为null
}
```

正确示例:
```java
// 正确: 配置SecurityContext传播策略
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {
    
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setTaskDecorator(new SecurityContextPropagatingTaskDecorator());
        executor.initialize();
        return executor;
    }
}

// 或在方法中手动传递
@Async
public void processAsync(Authentication authentication) {
    SecurityContextHolder.getContext().setAuthentication(authentication);
    // 处理业务逻辑
}
```

**⚠️ 陷阱5: 忘记清理SecurityContext**

错误示例:
```java
// 错误: 手动设置后未清理
SecurityContextHolder.getContext().setAuthentication(authentication);
// 执行业务逻辑
// 忘记清理,可能导致线程池复用时的安全问题
```

正确示例:
```java
// 正确: 使用try-finally确保清理
try {
    SecurityContextHolder.getContext().setAuthentication(authentication);
    // 执行业务逻辑
} finally {
    SecurityContextHolder.clearContext();
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
    key-alias: tomcat
```

**2. 配置安全响应头**
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .headers(headers -> headers
            .contentSecurityPolicy(csp -> csp
                .policyDirectives("default-src 'self'")
            )
            .frameOptions(frame -> frame.deny())
            .xssProtection(xss -> xss.block(true))
            .contentTypeOptions(Customizer.withDefaults())
        );
    return http.build();
}
```

**3. 限制登录尝试**
```java
@Component
public class LoginAttemptService {

    private final int MAX_ATTEMPT = 5;
    private LoadingCache<String, Integer> attemptsCache;

    public LoginAttemptService() {
        attemptsCache = CacheBuilder.newBuilder()
            .expireAfterWrite(1, TimeUnit.DAYS)
            .build(new CacheLoader<String, Integer>() {
                public Integer load(String key) {
                    return 0;
                }
            });
    }

    public void loginSucceeded(String key) {
        attemptsCache.invalidate(key);
    }

    public void loginFailed(String key) {
        int attempts = attemptsCache.getUnchecked(key);
        attempts++;
        attemptsCache.put(key, attempts);
    }

    public boolean isBlocked(String key) {
        try {
            return attemptsCache.get(key) >= MAX_ATTEMPT;
        } catch (ExecutionException e) {
            return false;
        }
    }
}
```

**4. 审计日志**
```java
@Aspect
@Component
public class SecurityAuditAspect {

    @Autowired
    private AuditLogRepository auditLogRepository;

    @AfterReturning("@annotation(org.springframework.security.access.prepost.PreAuthorize)")
    public void logSecurityAccess(JoinPoint joinPoint) {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        String username = auth != null ? auth.getName() : "anonymous";
        String method = joinPoint.getSignature().toShortString();
        
        AuditLog log = new AuditLog();
        log.setUsername(username);
        log.setAction(method);
        log.setTimestamp(LocalDateTime.now());
        log.setSuccess(true);
        
        auditLogRepository.save(log);
    }
}
```

## ❓ 常见问题

### Q1: Spring Security 6与5的主要区别是什么?

A: 主要变化包括:
1. **配置方式**: 使用Lambda DSL替代链式调用
2. **授权管理**: 使用`AuthorizationManager`替代`AccessDecisionManager`
3. **方法安全**: `@EnableMethodSecurity`替代`@EnableGlobalMethodSecurity`
4. **WebSecurityConfigurerAdapter已废弃**: 直接使用`SecurityFilterChain` Bean
5. **默认行为**: 默认启用CSRF保护和安全响应头

### Q2: 如何在Spring Security中实现多租户?

A: 实现方案:
```java
@Component
public class TenantFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {
        String tenantId = request.getHeader("X-Tenant-ID");
        TenantContext.setCurrentTenant(tenantId);
        try {
            filterChain.doFilter(request, response);
        } finally {
            TenantContext.clear();
        }
    }
}

@PreAuthorize("@tenantSecurity.hasAccessToTenant(#tenantId)")
public Data getData(String tenantId) {
    return dataRepository.findByTenantId(tenantId);
}
```

### Q3: 如何处理JWT令牌刷新?

A: 实现刷新令牌机制:
```java
@Service
public class TokenService {

    public TokenResponse refreshToken(String refreshToken) {
        if (!tokenProvider.validateToken(refreshToken)) {
            throw new InvalidTokenException("刷新令牌无效");
        }
        
        String username = tokenProvider.getUsernameFromToken(refreshToken);
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        
        String newAccessToken = tokenProvider.generateAccessToken(userDetails);
        String newRefreshToken = tokenProvider.generateRefreshToken(userDetails);
        
        return new TokenResponse(newAccessToken, newRefreshToken);
    }
}
```

### Q4: 如何实现细粒度的数据权限控制?

A: 使用@PostFilter或自定义切面:
```java
@Service
public class DataService {

    @PostFilter("filterObject.departmentId == authentication.principal.departmentId or hasRole('ADMIN')")
    public List<Data> getAllData() {
        return dataRepository.findAll();
    }
}

// 或使用自定义切面
@Aspect
@Component
public class DataPermissionAspect {

    @Around("@annotation(DataPermission)")
    public Object checkDataPermission(ProceedingJoinPoint joinPoint) throws Throwable {
        Authentication auth = SecurityContextHolder.getContext().getAuthentication();
        // 检查数据权限逻辑
        return joinPoint.proceed();
    }
}
```

### Q5: 如何集成第三方OAuth2登录(如GitHub、Google)?

A: 配置OAuth2客户端:
```yaml
# application.yml
spring:
  security:
    oauth2:
      client:
        registration:
          github:
            client-id: your-client-id
            client-secret: your-client-secret
            scope: read:user,user:email
          google:
            client-id: your-client-id
            client-secret: your-client-secret
            scope: profile,email
```

```java
@Configuration
@EnableWebSecurity
public class OAuth2LoginConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .anyRequest().authenticated()
            )
            .oauth2Login(oauth2 -> oauth2
                .loginPage("/login")
                .defaultSuccessUrl("/dashboard")
                .userInfoEndpoint(userInfo -> userInfo
                    .userService(customOAuth2UserService())
                )
            );
        return http.build();
    }

    @Bean
    public OAuth2UserService<OAuth2UserRequest, OAuth2User> customOAuth2UserService() {
        return new CustomOAuth2UserService();
    }
}
```

## 🔗 相关资源

### 官方资源
- **官方文档**: https://docs.spring.io/spring-security/reference/
- **GitHub仓库**: https://github.com/spring-projects/spring-security
- **Spring Security OAuth2**: https://spring.io/projects/spring-authorization-server
- **示例项目**: https://github.com/spring-projects/spring-security-samples

### 推荐文章
- Spring Security架构深度解析
- OAuth2.0协议详解
- JWT最佳实践指南
- 微服务安全架构设计

### 视频教程
- Spring Security官方教程系列
- OAuth2与JWT实战课程
- 企业级权限管理系统开发

### 相关技术
- **Spring Boot**: 简化Spring Security配置
- **OAuth2**: 授权框架标准
- **JWT**: JSON Web Token令牌标准
- **LDAP**: 企业目录服务集成
- **Keycloak**: 开源身份和访问管理解决方案

## 📝 学习检查清单

### 基础知识
- [ ] 理解认证(Authentication)和授权(Authorization)的区别
- [ ] 掌握Spring Security的核心架构
- [ ] 了解安全过滤器链的工作原理
- [ ] 理解SecurityContext和Authentication对象

### 核心功能
- [ ] 能够配置基于表单的登录认证
- [ ] 能够配置基于角色的URL访问控制
- [ ] 掌握UserDetailsService的实现
- [ ] 理解PasswordEncoder的使用

### 进阶特性
- [ ] 掌握方法级安全注解(@PreAuthorize等)
- [ ] 理解并使用SpEL权限表达式
- [ ] 能够实现自定义认证过滤器
- [ ] 掌握JWT令牌的生成和验证

### OAuth2与JWT
- [ ] 理解OAuth2的四种授权模式
- [ ] 能够配置OAuth2授权服务器
- [ ] 能够配置OAuth2资源服务器
- [ ] 掌握JWT的结构和使用场景

### 实战能力
- [ ] 能够实现完整的用户认证授权系统
- [ ] 能够实现基于数据库的动态权限控制
- [ ] 能够实现记住我功能
- [ ] 能够实现登录失败次数限制
- [ ] 能够实现审计日志功能

### 安全最佳实践
- [ ] 了解常见的Web安全威胁(XSS、CSRF等)
- [ ] 掌握密码加密存储的最佳实践
- [ ] 了解会话管理的安全配置
- [ ] 掌握HTTPS的配置和使用

### 性能优化
- [ ] 了解Spring Security的性能影响
- [ ] 掌握缓存策略优化认证性能
- [ ] 了解无状态认证的优势

---

**学习建议**:
1. 先掌握基础的认证授权概念
2. 通过实战项目理解过滤器链的工作原理
3. 重点理解方法级安全和权限表达式
4. 深入学习OAuth2和JWT的实现细节
5. 关注安全最佳实践和常见陷阱
6. 持续关注Spring Security的版本更新

**下一步学习**:
- 学习Shiro安全框架进行对比
- 深入学习OAuth2.0协议规范
- 学习Keycloak等IAM解决方案
- 研究微服务架构下的统一认证授权方案

---

**文档版本**: v1.0  
**最后更新**: 2024-01-04  
**维护者**: @author erik.zhou  
**反馈**: 如有问题或建议,请提交Issue
