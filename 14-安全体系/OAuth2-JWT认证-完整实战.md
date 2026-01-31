# OAuth2.0 + JWT认证-完整实战

> @author erik.zhou  
> 难度: ⭐⭐⭐⭐  
> 技术栈: Spring Security + OAuth2 + JWT + Redis

## 📋 目录

- [OAuth2.0核心概念](#oauth20核心概念)
- [JWT令牌详解](#jwt令牌详解)
- [Spring Security集成](#spring-security集成)
- [完整实战案例](#完整实战案例)
- [最佳实践](#最佳实践)

---

## 🎯 OAuth2.0核心概念

### 什么是OAuth2.0

OAuth2.0是一个授权框架，允许第三方应用在用户授权的情况下访问用户资源，而无需获取用户的密码。

### 四种授权模式

```
1. 授权码模式 (Authorization Code)
   - 最安全、最常用
   - 适用于有后端的Web应用
   
2. 简化模式 (Implicit)
   - 适用于纯前端应用
   - 不推荐使用（安全性低）
   
3. 密码模式 (Password)
   - 用户直接提供用户名密码
   - 适用于可信任的第一方应用
   
4. 客户端模式 (Client Credentials)
   - 客户端以自己的名义访问资源
   - 适用于服务间调用
```

### OAuth2.0角色

```
┌─────────────────────────────────────────────────────────┐
│                    OAuth2.0 角色关系                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  Resource Owner (资源所有者)                              │
│         │                                                │
│         │ 1. 授权请求                                     │
│         ▼                                                │
│  Authorization Server (授权服务器)                        │
│         │                                                │
│         │ 2. 返回授权码                                   │
│         │ 3. 用授权码换取Token                            │
│         ▼                                                │
│  Client (客户端)                                          │
│         │                                                │
│         │ 4. 携带Token访问资源                            │
│         ▼                                                │
│  Resource Server (资源服务器)                             │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 🔐 JWT令牌详解

### JWT结构

```
JWT由三部分组成，用.分隔：

Header.Payload.Signature

示例：
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### JWT组成部分

```java
/**
 * JWT组成部分
 * @author erik.zhou
 */

// 1. Header（头部）
{
  "alg": "HS256",  // 签名算法
  "typ": "JWT"     // 令牌类型
}

// 2. Payload（载荷）
{
  "sub": "1234567890",           // 主题（用户ID）
  "name": "John Doe",            // 用户名
  "iat": 1516239022,             // 签发时间
  "exp": 1516242622,             // 过期时间
  "roles": ["ROLE_USER"]         // 自定义字段
}

// 3. Signature（签名）
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

### JWT工具类

```java
/**
 * JWT工具类
 * @author erik.zhou
 */
@Component
public class JwtTokenUtil {
    
    @Value("${jwt.secret}")
    private String secret;
    
    @Value("${jwt.expiration}")
    private Long expiration;
    
    /**
     * 生成Token
     */
    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("username", userDetails.getUsername());
        claims.put("roles", userDetails.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.toList()));
        
        return createToken(claims, userDetails.getUsername());
    }
    
    /**
     * 创建Token
     */
    private String createToken(Map<String, Object> claims, String subject) {
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + expiration * 1000);
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(subject)
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }
    
    /**
     * 从Token中获取用户名
     */
    public String getUsernameFromToken(String token) {
        return getClaimFromToken(token, Claims::getSubject);
    }
    
    /**
     * 从Token中获取过期时间
     */
    public Date getExpirationDateFromToken(String token) {
        return getClaimFromToken(token, Claims::getExpiration);
    }
    
    /**
     * 从Token中获取声明
     */
    public <T> T getClaimFromToken(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = getAllClaimsFromToken(token);
        return claimsResolver.apply(claims);
    }
    
    /**
     * 获取所有声明
     */
    private Claims getAllClaimsFromToken(String token) {
        return Jwts.parser()
            .setSigningKey(secret)
            .parseClaimsJws(token)
            .getBody();
    }
    
    /**
     * 检查Token是否过期
     */
    private Boolean isTokenExpired(String token) {
        final Date expiration = getExpirationDateFromToken(token);
        return expiration.before(new Date());
    }
    
    /**
     * 验证Token
     */
    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = getUsernameFromToken(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
    
    /**
     * 刷新Token
     */
    public String refreshToken(String token) {
        final Claims claims = getAllClaimsFromToken(token);
        claims.setIssuedAt(new Date());
        
        Date expiryDate = new Date(System.currentTimeMillis() + expiration * 1000);
        claims.setExpiration(expiryDate);
        
        return Jwts.builder()
            .setClaims(claims)
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }
}
```

---

## 🔧 Spring Security集成

### 1. 依赖配置

```xml
<!-- pom.xml -->
<dependencies>
    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    
    <!-- JWT -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt</artifactId>
        <version>0.9.1</version>
    </dependency>
    
    <!-- Redis -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>
</dependencies>
```

### 2. Security配置

```java
/**
 * Spring Security配置
 * @author erik.zhou
 */
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Autowired
    private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    
    @Autowired
    private JwtRequestFilter jwtRequestFilter;
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    /**
     * 密码编码器
     */
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    /**
     * 认证管理器
     */
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
    
    /**
     * 配置认证
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService)
            .passwordEncoder(passwordEncoder());
    }
    
    /**
     * 配置HTTP安全
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            // 禁用CSRF（使用JWT不需要CSRF）
            .csrf().disable()
            
            // 配置异常处理
            .exceptionHandling()
                .authenticationEntryPoint(jwtAuthenticationEntryPoint)
            .and()
            
            // 配置Session管理（无状态）
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            
            // 配置授权规则
            .authorizeRequests()
                // 允许匿名访问的接口
                .antMatchers("/api/auth/**").permitAll()
                .antMatchers("/api/public/**").permitAll()
                .antMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                // 其他接口需要认证
                .anyRequest().authenticated();
        
        // 添加JWT过滤器
        http.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);
    }
}
```

### 3. JWT认证过滤器

```java
/**
 * JWT认证过滤器
 * @author erik.zhou
 */
@Component
public class JwtRequestFilter extends OncePerRequestFilter {
    
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                   HttpServletResponse response, 
                                   FilterChain chain)
            throws ServletException, IOException {
        
        // 1. 从请求头获取Token
        final String requestTokenHeader = request.getHeader("Authorization");
        
        String username = null;
        String jwtToken = null;
        
        // 2. 解析Token
        if (requestTokenHeader != null && requestTokenHeader.startsWith("Bearer ")) {
            jwtToken = requestTokenHeader.substring(7);
            try {
                username = jwtTokenUtil.getUsernameFromToken(jwtToken);
            } catch (IllegalArgumentException e) {
                logger.error("无法获取JWT Token");
            } catch (ExpiredJwtException e) {
                logger.error("JWT Token已过期");
            }
        }
        
        // 3. 验证Token
        if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            
            // 检查Token是否在黑名单中
            if (isTokenBlacklisted(jwtToken)) {
                logger.warn("Token已被注销");
                chain.doFilter(request, response);
                return;
            }
            
            UserDetails userDetails = userDetailsService.loadUserByUsername(username);
            
            // 验证Token有效性
            if (jwtTokenUtil.validateToken(jwtToken, userDetails)) {
                UsernamePasswordAuthenticationToken authentication = 
                    new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                
                authentication.setDetails(
                    new WebAuthenticationDetailsSource().buildDetails(request));
                
                // 设置认证信息到Security上下文
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        }
        
        chain.doFilter(request, response);
    }
    
    /**
     * 检查Token是否在黑名单中
     */
    private boolean isTokenBlacklisted(String token) {
        String key = "token:blacklist:" + token;
        return Boolean.TRUE.equals(redisTemplate.hasKey(key));
    }
}
```

### 4. 认证入口点

```java
/**
 * JWT认证入口点
 * 处理认证失败的情况
 * @author erik.zhou
 */
@Component
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {
    
    @Override
    public void commence(HttpServletRequest request,
                        HttpServletResponse response,
                        AuthenticationException authException) 
            throws IOException {
        
        response.setContentType("application/json;charset=UTF-8");
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        
        Map<String, Object> result = new HashMap<>();
        result.put("code", 401);
        result.put("message", "未授权，请先登录");
        result.put("timestamp", System.currentTimeMillis());
        
        response.getWriter().write(JSON.toJSONString(result));
    }
}
```

---

## 💻 完整实战案例

### 1. 用户认证服务

```java
/**
 * 认证服务
 * @author erik.zhou
 */
@Service
public class AuthService {
    
    @Autowired
    private AuthenticationManager authenticationManager;
    
    @Autowired
    private JwtTokenUtil jwtTokenUtil;
    
    @Autowired
    private UserDetailsService userDetailsService;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 登录
     */
    public LoginResponse login(LoginRequest request) {
        // 1. 认证
        try {
            Authentication authentication = authenticationManager.authenticate(
                new UsernamePasswordAuthenticationToken(
                    request.getUsername(),
                    request.getPassword()
                )
            );
            
            SecurityContextHolder.getContext().setAuthentication(authentication);
            
        } catch (BadCredentialsException e) {
            throw new BusinessException("用户名或密码错误");
        }
        
        // 2. 生成Token
        UserDetails userDetails = userDetailsService.loadUserByUsername(request.getUsername());
        String accessToken = jwtTokenUtil.generateToken(userDetails);
        String refreshToken = generateRefreshToken(userDetails);
        
        // 3. 保存Token到Redis
        saveTokenToRedis(userDetails.getUsername(), accessToken, refreshToken);
        
        // 4. 返回结果
        LoginResponse response = new LoginResponse();
        response.setAccessToken(accessToken);
        response.setRefreshToken(refreshToken);
        response.setTokenType("Bearer");
        response.setExpiresIn(3600L);
        
        return response;
    }
    
    /**
     * 登出
     */
    public void logout(String token) {
        // 1. 将Token加入黑名单
        String key = "token:blacklist:" + token;
        redisTemplate.opsForValue().set(key, "1", 24, TimeUnit.HOURS);
        
        // 2. 删除Redis中的Token
        String username = jwtTokenUtil.getUsernameFromToken(token);
        String tokenKey = "token:user:" + username;
        redisTemplate.delete(tokenKey);
    }
    
    /**
     * 刷新Token
     */
    public LoginResponse refreshToken(String refreshToken) {
        // 1. 验证RefreshToken
        if (!validateRefreshToken(refreshToken)) {
            throw new BusinessException("RefreshToken无效");
        }
        
        // 2. 生成新的AccessToken
        String username = jwtTokenUtil.getUsernameFromToken(refreshToken);
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        String newAccessToken = jwtTokenUtil.generateToken(userDetails);
        
        // 3. 更新Redis
        updateTokenInRedis(username, newAccessToken);
        
        // 4. 返回结果
        LoginResponse response = new LoginResponse();
        response.setAccessToken(newAccessToken);
        response.setRefreshToken(refreshToken);
        response.setTokenType("Bearer");
        response.setExpiresIn(3600L);
        
        return response;
    }
    
    /**
     * 生成RefreshToken
     */
    private String generateRefreshToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("type", "refresh");
        
        Date now = new Date();
        Date expiryDate = new Date(now.getTime() + 7 * 24 * 3600 * 1000);  // 7天
        
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(now)
            .setExpiration(expiryDate)
            .signWith(SignatureAlgorithm.HS512, "refresh-secret")
            .compact();
    }
    
    /**
     * 保存Token到Redis
     */
    private void saveTokenToRedis(String username, String accessToken, String refreshToken) {
        String key = "token:user:" + username;
        
        Map<String, String> tokenMap = new HashMap<>();
        tokenMap.put("accessToken", accessToken);
        tokenMap.put("refreshToken", refreshToken);
        
        redisTemplate.opsForHash().putAll(key, tokenMap);
        redisTemplate.expire(key, 7, TimeUnit.DAYS);
    }
    
    /**
     * 更新Redis中的Token
     */
    private void updateTokenInRedis(String username, String newAccessToken) {
        String key = "token:user:" + username;
        redisTemplate.opsForHash().put(key, "accessToken", newAccessToken);
    }
    
    /**
     * 验证RefreshToken
     */
    private boolean validateRefreshToken(String refreshToken) {
        try {
            Jwts.parser()
                .setSigningKey("refresh-secret")
                .parseClaimsJws(refreshToken);
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}
```

### 2. 认证控制器

```java
/**
 * 认证控制器
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/auth")
public class AuthController {
    
    @Autowired
    private AuthService authService;
    
    /**
     * 登录
     */
    @PostMapping("/login")
    public Result<LoginResponse> login(@RequestBody @Valid LoginRequest request) {
        LoginResponse response = authService.login(request);
        return Result.success(response);
    }
    
    /**
     * 登出
     */
    @PostMapping("/logout")
    public Result<Void> logout(@RequestHeader("Authorization") String authorization) {
        String token = authorization.substring(7);
        authService.logout(token);
        return Result.success();
    }
    
    /**
     * 刷新Token
     */
    @PostMapping("/refresh")
    public Result<LoginResponse> refreshToken(@RequestBody RefreshTokenRequest request) {
        LoginResponse response = authService.refreshToken(request.getRefreshToken());
        return Result.success(response);
    }
    
    /**
     * 获取当前用户信息
     */
    @GetMapping("/me")
    public Result<UserInfo> getCurrentUser() {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        String username = authentication.getName();
        
        // 查询用户信息
        UserInfo userInfo = userService.getUserInfo(username);
        
        return Result.success(userInfo);
    }
}
```

### 3. 权限控制

```java
/**
 * 权限控制示例
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    /**
     * 需要ADMIN角色
     */
    @PreAuthorize("hasRole('ADMIN')")
    @GetMapping
    public Result<List<User>> getAllUsers() {
        List<User> users = userService.getAllUsers();
        return Result.success(users);
    }
    
    /**
     * 需要USER或ADMIN角色
     */
    @PreAuthorize("hasAnyRole('USER', 'ADMIN')")
    @GetMapping("/{id}")
    public Result<User> getUser(@PathVariable Long id) {
        User user = userService.getUser(id);
        return Result.success(user);
    }
    
    /**
     * 自定义权限表达式
     */
    @PreAuthorize("@permissionService.hasPermission(#id)")
    @PutMapping("/{id}")
    public Result<Void> updateUser(@PathVariable Long id, @RequestBody User user) {
        userService.updateUser(id, user);
        return Result.success();
    }
}

/**
 * 权限服务
 * @author erik.zhou
 */
@Service("permissionService")
public class PermissionService {
    
    /**
     * 检查是否有权限
     */
    public boolean hasPermission(Long userId) {
        Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
        String currentUsername = authentication.getName();
        
        // 检查是否是用户本人或管理员
        User currentUser = userService.getUserByUsername(currentUsername);
        
        return currentUser.getId().equals(userId) || 
               authentication.getAuthorities().stream()
                   .anyMatch(a -> a.getAuthority().equals("ROLE_ADMIN"));
    }
}
```

---

## 🎓 最佳实践

### 1. Token存储策略

```java
/**
 * Token存储最佳实践
 * @author erik.zhou
 */

// ✅ 推荐：使用Redis存储Token
// 优点：支持分布式、可以主动失效、可以统计在线用户
public class RedisTokenStore {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    /**
     * 保存Token
     */
    public void saveToken(String username, String token) {
        String key = "token:user:" + username;
        redisTemplate.opsForValue().set(key, token, 24, TimeUnit.HOURS);
    }
    
    /**
     * 获取Token
     */
    public String getToken(String username) {
        String key = "token:user:" + username;
        return (String) redisTemplate.opsForValue().get(key);
    }
    
    /**
     * 删除Token
     */
    public void deleteToken(String username) {
        String key = "token:user:" + username;
        redisTemplate.delete(key);
    }
}

// ❌ 不推荐：纯JWT（无状态）
// 缺点：无法主动失效、无法踢人下线
```

### 2. Token刷新策略

```java
/**
 * Token刷新策略
 * @author erik.zhou
 */

// ✅ 推荐：双Token机制（AccessToken + RefreshToken）
// AccessToken: 短期有效（1小时）
// RefreshToken: 长期有效（7天）

public class TokenRefreshStrategy {
    
    /**
     * 刷新Token
     */
    public LoginResponse refreshToken(String refreshToken) {
        // 1. 验证RefreshToken
        if (!isValidRefreshToken(refreshToken)) {
            throw new BusinessException("RefreshToken无效");
        }
        
        // 2. 生成新的AccessToken
        String username = getUsernameFromToken(refreshToken);
        String newAccessToken = generateAccessToken(username);
        
        // 3. 返回新Token
        return new LoginResponse(newAccessToken, refreshToken);
    }
}
```

### 3. 安全加固

```java
/**
 * 安全加固最佳实践
 * @author erik.zhou
 */
public class SecurityEnhancement {
    
    /**
     * 1. Token加密传输
     */
    // 使用HTTPS传输Token
    // 不要在URL中传递Token
    
    /**
     * 2. Token黑名单
     */
    public void addToBlacklist(String token) {
        String key = "token:blacklist:" + token;
        // 设置过期时间为Token的剩余有效期
        long ttl = getTokenTTL(token);
        redisTemplate.opsForValue().set(key, "1", ttl, TimeUnit.SECONDS);
    }
    
    /**
     * 3. 限制登录尝试次数
     */
    public void checkLoginAttempts(String username) {
        String key = "login:attempts:" + username;
        Integer attempts = (Integer) redisTemplate.opsForValue().get(key);
        
        if (attempts != null && attempts >= 5) {
            throw new BusinessException("登录尝试次数过多，请30分钟后再试");
        }
    }
    
    /**
     * 4. IP白名单
     */
    public boolean isIpAllowed(String ip) {
        Set<Object> whitelist = redisTemplate.opsForSet().members("ip:whitelist");
        return whitelist != null && whitelist.contains(ip);
    }
    
    /**
     * 5. 设备指纹
     */
    public String generateDeviceFingerprint(HttpServletRequest request) {
        String userAgent = request.getHeader("User-Agent");
        String ip = getClientIp(request);
        
        return DigestUtils.md5Hex(userAgent + ip);
    }
}
```

### 4. 性能优化

```java
/**
 * 性能优化最佳实践
 * @author erik.zhou
 */
public class PerformanceOptimization {
    
    /**
     * 1. 缓存UserDetails
     */
    @Cacheable(value = "userDetails", key = "#username")
    public UserDetails loadUserByUsername(String username) {
        // 从数据库加载用户信息
        return userRepository.findByUsername(username);
    }
    
    /**
     * 2. 使用Redis Pipeline批量操作
     */
    public void batchCheckTokens(List<String> tokens) {
        List<Object> results = redisTemplate.executePipelined(
            (RedisCallback<Object>) connection -> {
                tokens.forEach(token -> {
                    String key = "token:blacklist:" + token;
                    connection.exists(key.getBytes());
                });
                return null;
            }
        );
    }
    
    /**
     * 3. 异步记录日志
     */
    @Async
    public void logLoginEvent(String username, String ip) {
        LoginLog log = new LoginLog();
        log.setUsername(username);
        log.setIp(ip);
        log.setLoginTime(LocalDateTime.now());
        
        loginLogRepository.save(log);
    }
}
```

---

## 📝 总结

### 核心要点

1. **OAuth2.0** - 理解四种授权模式，选择合适的模式
2. **JWT** - 理解JWT结构，正确使用JWT
3. **Spring Security** - 掌握Spring Security配置和使用
4. **Token管理** - 使用Redis管理Token，支持主动失效
5. **安全加固** - Token黑名单、限流、IP白名单等

### 安全建议

1. **使用HTTPS** - 保证Token传输安全
2. **Token加密** - 使用强加密算法
3. **定期刷新** - 使用双Token机制
4. **黑名单机制** - 支持Token主动失效
5. **限流防刷** - 防止暴力破解

---

**作者**: erik.zhou  
**最后更新**: 2024-01-04
