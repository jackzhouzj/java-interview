# Spring MVC 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [相关资源](#相关资源)
- [学习检查清单](#学习检查清单)

## 📚 技术概述
- **版本**: Spring MVC 6.1.x (Spring Framework 6.1.x)
- **官方文档**: https://docs.spring.io/spring-framework/reference/web/webmvc.html
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: 
  - Spring Framework核心知识
  - Java Servlet基础
  - HTTP协议
  - RESTful API设计
- **文档来源**: Context7 + Spring官方文档
- **更新时间**: 2024-12-31
- **作者**: @author erik.zhou

## 🎯 学习目标
- [ ] 理解Spring MVC的架构和请求处理流程
- [ ] 掌握DispatcherServlet的工作原理
- [ ] 理解HandlerMapping和HandlerAdapter
- [ ] 掌握请求参数绑定和数据验证
- [ ] 理解拦截器的使用
- [ ] 掌握异常处理机制
- [ ] 理解参数解析器和返回值处理器

## 📖 基础概念

### 1.1 什么是Spring MVC

Spring MVC是Spring Framework的Web模块，基于MVC（Model-View-Controller）设计模式构建。它采用前端控制器模式，通过中央Servlet（DispatcherServlet）提供请求处理的共享算法，而实际工作由可配置的委托组件执行。

**MVC模式**：
- **Model（模型）**: 封装应用程序数据和业务逻辑
- **View（视图）**: 负责渲染模型数据，生成HTML输出
- **Controller（控制器）**: 处理用户请求，调用模型，选择视图

### 1.2 核心组件

- **DispatcherServlet**: 前端控制器，负责请求分发
- **HandlerMapping**: 处理器映射，将请求映射到处理器
- **HandlerAdapter**: 处理器适配器，执行处理器
- **ViewResolver**: 视图解析器，解析视图名称
- **HandlerInterceptor**: 拦截器，在处理器执行前后进行拦截
- **HandlerExceptionResolver**: 异常解析器，处理异常

### 1.3 请求处理流程

```
客户端请求
    ↓
DispatcherServlet（前端控制器）
    ↓
HandlerMapping（查找处理器）
    ↓
HandlerAdapter（执行处理器）
    ↓
Controller（处理业务逻辑）
    ↓
ModelAndView（返回模型和视图）
    ↓
ViewResolver（解析视图）
    ↓
View（渲染视图）
    ↓
响应客户端
```


## 🔥 核心特性

### 2.1 DispatcherServlet 🔥 (⚠️ 难点)

DispatcherServlet是Spring MVC的核心，作为前端控制器处理所有HTTP请求。

**配置DispatcherServlet（Spring Boot自动配置）**：

```java
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

**传统web.xml配置**：

```xml
<web-app>
    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/app-context.xml</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>
</web-app>
```

### 2.2 Controller和请求映射 🔥

```java
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    // GET请求 - 获取所有用户
    @GetMapping
    public List<User> getAllUsers() {
        return userService.findAll();
    }
    
    // GET请求 - 根据ID获取用户
    @GetMapping("/{id}")
    public User getUserById(@PathVariable Long id) {
        return userService.findById(id);
    }
    
    // POST请求 - 创建用户
    @PostMapping
    public User createUser(@RequestBody @Valid User user) {
        return userService.save(user);
    }
    
    // PUT请求 - 更新用户
    @PutMapping("/{id}")
    public User updateUser(@PathVariable Long id, @RequestBody @Valid User user) {
        return userService.update(id, user);
    }
    
    // DELETE请求 - 删除用户
    @DeleteMapping("/{id}")
    @ResponseStatus(HttpStatus.NO_CONTENT)
    public void deleteUser(@PathVariable Long id) {
        userService.delete(id);
    }
    
    // 查询参数
    @GetMapping("/search")
    public List<User> searchUsers(
        @RequestParam(required = false) String name,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) {
        return userService.search(name, page, size);
    }
}
```

### 2.3 参数绑定和数据验证

**路径变量**：

```java
@GetMapping("/users/{id}/orders/{orderId}")
public Order getOrder(@PathVariable Long id, @PathVariable Long orderId) {
    return orderService.findByUserAndOrder(id, orderId);
}
```

**请求参数**：

```java
@GetMapping("/users")
public List<User> getUsers(
    @RequestParam(required = false) String name,
    @RequestParam(defaultValue = "0") int page) {
    return userService.findByName(name, page);
}
```

**请求体**：

```java
@PostMapping("/users")
public User createUser(@RequestBody @Valid UserDTO userDTO) {
    return userService.create(userDTO);
}
```

**数据验证**：

```java
public class UserDTO {
    
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 50, message = "用户名长度必须在3-50之间")
    private String username;
    
    @NotBlank(message = "邮箱不能为空")
    @Email(message = "邮箱格式不正确")
    private String email;
    
    @Min(value = 18, message = "年龄必须大于等于18")
    @Max(value = 100, message = "年龄必须小于等于100")
    private Integer age;
    
    // Getter和Setter
}
```

### 2.4 拦截器 (⚠️ 难点)

**创建拦截器**：

```java
@Component
public class LoggingInterceptor implements HandlerInterceptor {
    
    private static final Logger logger = LoggerFactory.getLogger(LoggingInterceptor.class);
    
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) {
        logger.info("请求URL: {}", request.getRequestURI());
        logger.info("请求方法: {}", request.getMethod());
        return true;  // 返回true继续执行，false中断请求
    }
    
    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, 
                          Object handler, ModelAndView modelAndView) {
        logger.info("请求处理完成");
    }
    
    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, 
                               Object handler, Exception ex) {
        logger.info("视图渲染完成");
        if (ex != null) {
            logger.error("请求异常", ex);
        }
    }
}
```

**注册拦截器**：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Autowired
    private LoggingInterceptor loggingInterceptor;
    
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(loggingInterceptor)
            .addPathPatterns("/api/**")  // 拦截路径
            .excludePathPatterns("/api/public/**");  // 排除路径
    }
}
```

### 2.5 异常处理

**全局异常处理器**：

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ErrorResponse handleResourceNotFound(ResourceNotFoundException ex) {
        return new ErrorResponse(404, ex.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ErrorResponse handleValidationException(MethodArgumentNotValidException ex) {
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getFieldErrors().forEach(error -> 
            errors.put(error.getField(), error.getDefaultMessage())
        );
        return new ErrorResponse(400, "验证失败", errors);
    }
    
    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ErrorResponse handleException(Exception ex) {
        return new ErrorResponse(500, "系统错误");
    }
}
```

### 2.6 参数解析器 (⚠️ 难点)

**自定义参数解析器**：

```java
public class CurrentUserArgumentResolver implements HandlerMethodArgumentResolver {
    
    @Override
    public boolean supportsParameter(MethodParameter parameter) {
        return parameter.hasParameterAnnotation(CurrentUser.class) 
            && parameter.getParameterType().equals(User.class);
    }
    
    @Override
    public Object resolveArgument(MethodParameter parameter, 
                                 ModelAndViewContainer mavContainer,
                                 NativeWebRequest webRequest, 
                                 WebDataBinderFactory binderFactory) {
        // 从请求中获取当前用户
        HttpServletRequest request = webRequest.getNativeRequest(HttpServletRequest.class);
        String userId = request.getHeader("X-User-Id");
        return userService.findById(Long.parseLong(userId));
    }
}
```

**注册参数解析器**：

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(new CurrentUserArgumentResolver());
    }
}
```

**使用自定义参数**：

```java
@GetMapping("/profile")
public UserProfile getProfile(@CurrentUser User user) {
    return userService.getProfile(user);
}
```

## 💻 实战应用

### 3.1 构建RESTful API

```java
@RestController
@RequestMapping("/api/products")
public class ProductController {
    
    @Autowired
    private ProductService productService;
    
    @GetMapping
    public ResponseEntity<Page<Product>> getProducts(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size,
        @RequestParam(defaultValue = "id") String sortBy) {
        
        Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy));
        Page<Product> products = productService.findAll(pageable);
        return ResponseEntity.ok(products);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        Product product = productService.findById(id);
        return ResponseEntity.ok(product);
    }
    
    @PostMapping
    public ResponseEntity<Product> createProduct(@RequestBody @Valid ProductDTO productDTO) {
        Product product = productService.create(productDTO);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(product.getId())
            .toUri();
        return ResponseEntity.created(location).body(product);
    }
}
```

### 3.2 文件上传

```java
@PostMapping("/upload")
public ResponseEntity<String> uploadFile(@RequestParam("file") MultipartFile file) {
    if (file.isEmpty()) {
        return ResponseEntity.badRequest().body("文件不能为空");
    }
    
    try {
        String filename = fileService.store(file);
        return ResponseEntity.ok("文件上传成功: " + filename);
    } catch (IOException e) {
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body("文件上传失败");
    }
}
```

### 3.3 跨域配置

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

## ✨ 最佳实践

### 4.1 RESTful API设计规范

1. **使用名词而非动词**
   - ✅ GET /api/users
   - ❌ GET /api/getUsers

2. **使用复数形式**
   - ✅ /api/users
   - ❌ /api/user

3. **使用HTTP方法表示操作**
   - GET: 查询
   - POST: 创建
   - PUT: 更新（全量）
   - PATCH: 更新（部分）
   - DELETE: 删除

4. **使用HTTP状态码**
   - 200: 成功
   - 201: 创建成功
   - 204: 无内容
   - 400: 请求错误
   - 401: 未认证
   - 403: 无权限
   - 404: 未找到
   - 500: 服务器错误

### 4.2 统一响应格式

```java
public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;
    private long timestamp;
    
    public static <T> ApiResponse<T> success(T data) {
        ApiResponse<T> response = new ApiResponse<>();
        response.setCode(200);
        response.setMessage("成功");
        response.setData(data);
        response.setTimestamp(System.currentTimeMillis());
        return response;
    }
    
    public static <T> ApiResponse<T> error(int code, String message) {
        ApiResponse<T> response = new ApiResponse<>();
        response.setCode(code);
        response.setMessage(message);
        response.setTimestamp(System.currentTimeMillis());
        return response;
    }
}
```

## ❓ 常见问题

### Q1: @Controller和@RestController的区别？

**A**: 
- `@Controller`返回视图名称，需要配合`@ResponseBody`返回JSON
- `@RestController` = `@Controller` + `@ResponseBody`，直接返回JSON

### Q2: 如何处理跨域问题？

**A**: 
1. 使用`@CrossOrigin`注解
2. 实现`WebMvcConfigurer`的`addCorsMappings`方法
3. 使用Filter

### Q3: 拦截器和过滤器的区别？

**A**: 
- 过滤器（Filter）：Servlet规范，在DispatcherServlet之前执行
- 拦截器（Interceptor）：Spring MVC规范，在DispatcherServlet之后、Controller之前执行
- 拦截器可以访问Spring容器中的Bean

## 🔗 相关资源

- [Spring MVC官方文档](https://docs.spring.io/spring-framework/reference/web/webmvc.html)
- [RESTful API设计指南](https://restfulapi.net/)

## 📝 学习检查清单

- [ ] 理解Spring MVC架构
- [ ] 掌握DispatcherServlet工作原理
- [ ] 掌握Controller和请求映射
- [ ] 理解参数绑定和数据验证
- [ ] 掌握拦截器的使用
- [ ] 理解异常处理机制
- [ ] 掌握RESTful API设计

**预计学习时长**: 20-30小时
