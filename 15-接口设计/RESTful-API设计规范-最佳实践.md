# RESTful API设计规范-最佳实践

> @author erik.zhou  
> 难度: ⭐⭐⭐  
> 重要性: ⭐⭐⭐⭐⭐

## 📋 目录

- [REST核心原则](#rest核心原则)
- [URL设计规范](#url设计规范)
- [HTTP方法使用](#http方法使用)
- [状态码规范](#状态码规范)
- [请求响应设计](#请求响应设计)
- [版本管理](#版本管理)
- [最佳实践](#最佳实践)

---

## 🎯 REST核心原则

### 什么是REST

REST (Representational State Transfer) 是一种软件架构风格，用于设计网络应用程序的接口。

### REST六大约束

```
1. 客户端-服务器 (Client-Server)
   - 关注点分离
   - 客户端和服务器独立演化
   
2. 无状态 (Stateless)
   - 每个请求包含所有必要信息
   - 服务器不保存客户端状态
   
3. 可缓存 (Cacheable)
   - 响应可以被缓存
   - 提高性能和可扩展性
   
4. 统一接口 (Uniform Interface)
   - 资源标识
   - 通过表述操作资源
   - 自描述消息
   - 超媒体驱动
   
5. 分层系统 (Layered System)
   - 客户端无法直接知道连接的是服务器还是中间层
   
6. 按需代码 (Code on Demand) - 可选
   - 服务器可以返回可执行代码
```

---

## 🔗 URL设计规范

### 1. 基本规范

```
✅ 正确的URL设计：

# 使用名词复数表示资源集合
GET    /api/users              # 获取用户列表
GET    /api/users/123          # 获取ID为123的用户
POST   /api/users              # 创建用户
PUT    /api/users/123          # 更新ID为123的用户
DELETE /api/users/123          # 删除ID为123的用户

# 使用嵌套表示资源关系
GET    /api/users/123/orders   # 获取用户123的订单列表
GET    /api/users/123/orders/456  # 获取用户123的订单456

# 使用查询参数进行过滤、排序、分页
GET    /api/users?status=active&page=1&size=10
GET    /api/users?sort=createTime,desc
GET    /api/products?category=electronics&minPrice=100

❌ 错误的URL设计：

# 不要使用动词
GET    /api/getUsers           # 错误
POST   /api/createUser         # 错误
PUT    /api/updateUser/123     # 错误
DELETE /api/deleteUser/123     # 错误

# 不要使用单数
GET    /api/user               # 错误
GET    /api/user/123           # 错误

# 不要在URL中包含动作
GET    /api/users/123/delete   # 错误
POST   /api/users/123/activate # 错误
```

### 2. URL命名规范

```java
/**
 * URL命名规范
 * @author erik.zhou
 */
public class UrlNamingConvention {
    
    // ✅ 正确：使用小写字母和连字符
    String url1 = "/api/user-profiles";
    String url2 = "/api/order-items";
    
    // ❌ 错误：使用驼峰命名
    String url3 = "/api/userProfiles";
    String url4 = "/api/orderItems";
    
    // ❌ 错误：使用下划线
    String url5 = "/api/user_profiles";
    String url6 = "/api/order_items";
    
    // ✅ 正确：版本号放在URL中
    String url7 = "/api/v1/users";
    String url8 = "/api/v2/users";
    
    // ✅ 正确：使用查询参数
    String url9 = "/api/users?page=1&size=10&sort=createTime,desc";
    
    // ❌ 错误：在URL中包含文件扩展名
    String url10 = "/api/users.json";
    String url11 = "/api/users.xml";
}
```

### 3. 资源层级设计

```
✅ 推荐的资源层级（不超过3层）：

/api/users                          # 一级资源
/api/users/123/orders               # 二级资源
/api/users/123/orders/456/items     # 三级资源

❌ 不推荐（层级过深）：

/api/users/123/orders/456/items/789/details  # 四级资源（过深）

解决方案：
# 使用独立的资源端点
/api/order-items/789
/api/order-item-details/789
```

---

## 🔧 HTTP方法使用

### 1. 标准HTTP方法

```java
/**
 * HTTP方法使用规范
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    /**
     * GET - 获取资源（幂等、安全）
     */
    @GetMapping
    public Result<PageResult<User>> getUsers(
            @RequestParam(defaultValue = "1") int page,
            @RequestParam(defaultValue = "10") int size) {
        PageResult<User> users = userService.getUsers(page, size);
        return Result.success(users);
    }
    
    @GetMapping("/{id}")
    public Result<User> getUser(@PathVariable Long id) {
        User user = userService.getUser(id);
        return Result.success(user);
    }
    
    /**
     * POST - 创建资源（非幂等）
     */
    @PostMapping
    public Result<User> createUser(@RequestBody @Valid UserCreateRequest request) {
        User user = userService.createUser(request);
        return Result.success(user);
    }
    
    /**
     * PUT - 完整更新资源（幂等）
     */
    @PutMapping("/{id}")
    public Result<User> updateUser(
            @PathVariable Long id,
            @RequestBody @Valid UserUpdateRequest request) {
        User user = userService.updateUser(id, request);
        return Result.success(user);
    }
    
    /**
     * PATCH - 部分更新资源（幂等）
     */
    @PatchMapping("/{id}")
    public Result<User> patchUser(
            @PathVariable Long id,
            @RequestBody Map<String, Object> updates) {
        User user = userService.patchUser(id, updates);
        return Result.success(user);
    }
    
    /**
     * DELETE - 删除资源（幂等）
     */
    @DeleteMapping("/{id}")
    public Result<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return Result.success();
    }
}
```

### 2. HTTP方法特性

```
┌─────────┬──────────┬──────────┬─────────────────────┐
│ 方法     │ 幂等性    │ 安全性    │ 说明                 │
├─────────┼──────────┼──────────┼─────────────────────┤
│ GET     │ 是       │ 是       │ 获取资源             │
│ POST    │ 否       │ 否       │ 创建资源             │
│ PUT     │ 是       │ 否       │ 完整更新资源         │
│ PATCH   │ 是       │ 否       │ 部分更新资源         │
│ DELETE  │ 是       │ 否       │ 删除资源             │
│ HEAD    │ 是       │ 是       │ 获取资源元数据       │
│ OPTIONS │ 是       │ 是       │ 获取资源支持的方法   │
└─────────┴──────────┴──────────┴─────────────────────┘

幂等性：多次执行结果相同
安全性：不会修改资源状态
```

---

## 📊 状态码规范

### 1. 常用状态码

```java
/**
 * HTTP状态码使用规范
 * @author erik.zhou
 */
public class HttpStatusExample {
    
    /**
     * 2xx 成功
     */
    // 200 OK - 请求成功
    @GetMapping("/{id}")
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        User user = userService.getUser(id);
        return ResponseEntity.ok(user);
    }
    
    // 201 Created - 资源创建成功
    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        User created = userService.createUser(user);
        URI location = URI.create("/api/users/" + created.getId());
        return ResponseEntity.created(location).body(created);
    }
    
    // 204 No Content - 请求成功但无返回内容
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.deleteUser(id);
        return ResponseEntity.noContent().build();
    }
    
    /**
     * 4xx 客户端错误
     */
    // 400 Bad Request - 请求参数错误
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationException(
            MethodArgumentNotValidException ex) {
        ErrorResponse error = new ErrorResponse(400, "请求参数错误");
        return ResponseEntity.badRequest().body(error);
    }
    
    // 401 Unauthorized - 未认证
    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<ErrorResponse> handleAuthenticationException(
            AuthenticationException ex) {
        ErrorResponse error = new ErrorResponse(401, "未认证，请先登录");
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(error);
    }
    
    // 403 Forbidden - 无权限
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDeniedException(
            AccessDeniedException ex) {
        ErrorResponse error = new ErrorResponse(403, "无权限访问");
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }
    
    // 404 Not Found - 资源不存在
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFoundException(
            ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(404, "资源不存在");
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    // 409 Conflict - 资源冲突
    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ErrorResponse> handleConflictException(
            DuplicateResourceException ex) {
        ErrorResponse error = new ErrorResponse(409, "资源已存在");
        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }
    
    // 422 Unprocessable Entity - 业务逻辑错误
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ErrorResponse> handleBusinessException(
            BusinessException ex) {
        ErrorResponse error = new ErrorResponse(422, ex.getMessage());
        return ResponseEntity.status(422).body(error);
    }
    
    // 429 Too Many Requests - 请求过多
    @ExceptionHandler(RateLimitException.class)
    public ResponseEntity<ErrorResponse> handleRateLimitException(
            RateLimitException ex) {
        ErrorResponse error = new ErrorResponse(429, "请求过于频繁");
        return ResponseEntity.status(429).body(error);
    }
    
    /**
     * 5xx 服务器错误
     */
    // 500 Internal Server Error - 服务器内部错误
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception ex) {
        log.error("服务器内部错误", ex);
        ErrorResponse error = new ErrorResponse(500, "服务器内部错误");
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
    
    // 503 Service Unavailable - 服务不可用
    @ExceptionHandler(ServiceUnavailableException.class)
    public ResponseEntity<ErrorResponse> handleServiceUnavailableException(
            ServiceUnavailableException ex) {
        ErrorResponse error = new ErrorResponse(503, "服务暂时不可用");
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE).body(error);
    }
}
```

### 2. 状态码选择指南

```
选择状态码的原则：

1. 成功响应
   - 200: 通用成功响应
   - 201: 创建资源成功
   - 204: 删除成功（无返回内容）

2. 客户端错误
   - 400: 请求参数错误
   - 401: 未认证
   - 403: 无权限
   - 404: 资源不存在
   - 409: 资源冲突
   - 422: 业务逻辑错误
   - 429: 请求过多

3. 服务器错误
   - 500: 服务器内部错误
   - 503: 服务不可用
```

---

## 📦 请求响应设计

### 1. 统一响应格式

```java
/**
 * 统一响应格式
 * @author erik.zhou
 */
@Data
public class Result<T> {
    
    /**
     * 响应码
     */
    private Integer code;
    
    /**
     * 响应消息
     */
    private String message;
    
    /**
     * 响应数据
     */
    private T data;
    
    /**
     * 时间戳
     */
    private Long timestamp;
    
    /**
     * 请求ID（用于追踪）
     */
    private String requestId;
    
    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMessage("success");
        result.setData(data);
        result.setTimestamp(System.currentTimeMillis());
        result.setRequestId(MDC.get("requestId"));
        return result;
    }
    
    public static <T> Result<T> error(Integer code, String message) {
        Result<T> result = new Result<>();
        result.setCode(code);
        result.setMessage(message);
        result.setTimestamp(System.currentTimeMillis());
        result.setRequestId(MDC.get("requestId"));
        return result;
    }
}

/**
 * 分页响应格式
 * @author erik.zhou
 */
@Data
public class PageResult<T> {
    
    /**
     * 数据列表
     */
    private List<T> list;
    
    /**
     * 总记录数
     */
    private Long total;
    
    /**
     * 当前页码
     */
    private Integer page;
    
    /**
     * 每页大小
     */
    private Integer size;
    
    /**
     * 总页数
     */
    private Integer pages;
    
    public PageResult(List<T> list, Long total, Integer page, Integer size) {
        this.list = list;
        this.total = total;
        this.page = page;
        this.size = size;
        this.pages = (int) Math.ceil((double) total / size);
    }
}

/**
 * 错误响应格式
 * @author erik.zhou
 */
@Data
public class ErrorResponse {
    
    /**
     * 错误码
     */
    private Integer code;
    
    /**
     * 错误消息
     */
    private String message;
    
    /**
     * 错误详情
     */
    private List<FieldError> errors;
    
    /**
     * 时间戳
     */
    private Long timestamp;
    
    /**
     * 请求路径
     */
    private String path;
    
    /**
     * 请求ID
     */
    private String requestId;
    
    @Data
    public static class FieldError {
        private String field;
        private String message;
    }
}
```

### 2. 请求参数设计

```java
/**
 * 请求参数设计规范
 * @author erik.zhou
 */

// ✅ 正确：使用DTO接收参数
@Data
public class UserCreateRequest {
    
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20, message = "用户名长度必须在3-20之间")
    private String username;
    
    @NotBlank(message = "密码不能为空")
    @Size(min = 6, max = 20, message = "密码长度必须在6-20之间")
    private String password;
    
    @Email(message = "邮箱格式不正确")
    private String email;
    
    @Pattern(regexp = "^1[3-9]\\d{9}$", message = "手机号格式不正确")
    private String phone;
}

// ✅ 正确：查询参数使用Query对象
@Data
public class UserQueryRequest {
    
    @Min(value = 1, message = "页码必须大于0")
    private Integer page = 1;
    
    @Min(value = 1, message = "每页大小必须大于0")
    @Max(value = 100, message = "每页大小不能超过100")
    private Integer size = 10;
    
    private String keyword;
    
    private Integer status;
    
    private String sort = "createTime,desc";
}

// ❌ 错误：使用Map接收参数
@PostMapping
public Result<User> createUser(@RequestBody Map<String, Object> params) {
    // 无法进行参数校验
    // 无法生成API文档
    return null;
}
```

### 3. 响应数据设计

```java
/**
 * 响应数据设计规范
 * @author erik.zhou
 */

// ✅ 正确：使用VO返回数据
@Data
public class UserVO {
    
    private Long id;
    
    private String username;
    
    private String email;
    
    private String phone;
    
    private Integer status;
    
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime createTime;
    
    // 不返回敏感信息（如密码）
    // 不返回数据库字段（如isDeleted）
}

// ✅ 正确：使用@JsonIgnore隐藏字段
@Data
public class User {
    
    private Long id;
    
    private String username;
    
    @JsonIgnore  // 不返回密码
    private String password;
    
    private String email;
}

// ✅ 正确：使用@JsonProperty重命名字段
@Data
public class UserVO {
    
    @JsonProperty("user_id")
    private Long id;
    
    @JsonProperty("user_name")
    private String username;
}
```

---

## 🔄 版本管理

### 1. URL版本

```java
/**
 * URL版本管理
 * @author erik.zhou
 */

// ✅ 推荐：在URL中包含版本号
@RestController
@RequestMapping("/api/v1/users")
public class UserV1Controller {
    
    @GetMapping("/{id}")
    public Result<UserV1VO> getUser(@PathVariable Long id) {
        return Result.success(userService.getUserV1(id));
    }
}

@RestController
@RequestMapping("/api/v2/users")
public class UserV2Controller {
    
    @GetMapping("/{id}")
    public Result<UserV2VO> getUser(@PathVariable Long id) {
        return Result.success(userService.getUserV2(id));
    }
}
```

### 2. Header版本

```java
/**
 * Header版本管理
 * @author erik.zhou
 */

// 使用自定义Header指定版本
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    @GetMapping(value = "/{id}", headers = "API-Version=1")
    public Result<UserV1VO> getUserV1(@PathVariable Long id) {
        return Result.success(userService.getUserV1(id));
    }
    
    @GetMapping(value = "/{id}", headers = "API-Version=2")
    public Result<UserV2VO> getUserV2(@PathVariable Long id) {
        return Result.success(userService.getUserV2(id));
    }
}
```

### 3. 版本兼容策略

```java
/**
 * 版本兼容策略
 * @author erik.zhou
 */
public class VersionCompatibility {
    
    /**
     * 1. 向后兼容
     * 新版本兼容旧版本的请求
     */
    @GetMapping("/{id}")
    public Result<UserVO> getUser(
            @PathVariable Long id,
            @RequestHeader(value = "API-Version", defaultValue = "1") String version) {
        
        if ("2".equals(version)) {
            return Result.success(userService.getUserV2(id));
        }
        return Result.success(userService.getUserV1(id));
    }
    
    /**
     * 2. 废弃通知
     * 在响应头中添加废弃警告
     */
    @GetMapping("/old-endpoint")
    public ResponseEntity<Result<User>> oldEndpoint() {
        Result<User> result = Result.success(userService.getUser());
        
        return ResponseEntity.ok()
            .header("Warning", "299 - \"Deprecated API, use /api/v2/users instead\"")
            .header("Sunset", "2024-12-31")  // 废弃日期
            .body(result);
    }
}
```

---

## 🎓 最佳实践

### 1. HATEOAS（超媒体）

```java
/**
 * HATEOAS最佳实践
 * @author erik.zhou
 */
@Data
public class UserVO {
    
    private Long id;
    private String username;
    private String email;
    
    // 添加相关链接
    private Map<String, String> links;
    
    public void addLink(String rel, String href) {
        if (links == null) {
            links = new HashMap<>();
        }
        links.put(rel, href);
    }
}

@GetMapping("/{id}")
public Result<UserVO> getUser(@PathVariable Long id) {
    UserVO user = userService.getUser(id);
    
    // 添加相关链接
    user.addLink("self", "/api/users/" + id);
    user.addLink("orders", "/api/users/" + id + "/orders");
    user.addLink("update", "/api/users/" + id);
    user.addLink("delete", "/api/users/" + id);
    
    return Result.success(user);
}
```

### 2. 过滤、排序、分页

```java
/**
 * 过滤、排序、分页最佳实践
 * @author erik.zhou
 */
@GetMapping
public Result<PageResult<User>> getUsers(
        // 过滤
        @RequestParam(required = false) String keyword,
        @RequestParam(required = false) Integer status,
        @RequestParam(required = false) String email,
        
        // 排序
        @RequestParam(defaultValue = "createTime,desc") String sort,
        
        // 分页
        @RequestParam(defaultValue = "1") Integer page,
        @RequestParam(defaultValue = "10") Integer size) {
    
    // 构建查询条件
    UserQuery query = UserQuery.builder()
        .keyword(keyword)
        .status(status)
        .email(email)
        .sort(sort)
        .page(page)
        .size(size)
        .build();
    
    PageResult<User> result = userService.getUsers(query);
    
    return Result.success(result);
}
```

### 3. 批量操作

```java
/**
 * 批量操作最佳实践
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/users")
public class UserBatchController {
    
    /**
     * 批量创建
     */
    @PostMapping("/batch")
    public Result<List<User>> batchCreate(@RequestBody List<UserCreateRequest> requests) {
        List<User> users = userService.batchCreate(requests);
        return Result.success(users);
    }
    
    /**
     * 批量更新
     */
    @PutMapping("/batch")
    public Result<Void> batchUpdate(@RequestBody List<UserUpdateRequest> requests) {
        userService.batchUpdate(requests);
        return Result.success();
    }
    
    /**
     * 批量删除
     */
    @DeleteMapping("/batch")
    public Result<Void> batchDelete(@RequestBody List<Long> ids) {
        userService.batchDelete(ids);
        return Result.success();
    }
}
```

### 4. 异步操作

```java
/**
 * 异步操作最佳实践
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/tasks")
public class AsyncTaskController {
    
    /**
     * 提交异步任务
     * 返回202 Accepted和任务ID
     */
    @PostMapping("/export")
    public ResponseEntity<Result<String>> exportData(@RequestBody ExportRequest request) {
        // 创建异步任务
        String taskId = taskService.createExportTask(request);
        
        Result<String> result = Result.success(taskId);
        
        return ResponseEntity
            .status(HttpStatus.ACCEPTED)
            .header("Location", "/api/tasks/" + taskId)
            .body(result);
    }
    
    /**
     * 查询任务状态
     */
    @GetMapping("/{taskId}")
    public Result<TaskStatus> getTaskStatus(@PathVariable String taskId) {
        TaskStatus status = taskService.getTaskStatus(taskId);
        return Result.success(status);
    }
    
    /**
     * 获取任务结果
     */
    @GetMapping("/{taskId}/result")
    public ResponseEntity<Resource> getTaskResult(@PathVariable String taskId) {
        Resource resource = taskService.getTaskResult(taskId);
        
        return ResponseEntity.ok()
            .header(HttpHeaders.CONTENT_DISPOSITION, 
                "attachment; filename=\"export.xlsx\"")
            .body(resource);
    }
}
```

---

## 📝 总结

### 核心要点

1. **URL设计** - 使用名词复数，层级不超过3层
2. **HTTP方法** - 正确使用GET/POST/PUT/PATCH/DELETE
3. **状态码** - 使用标准HTTP状态码
4. **统一格式** - 请求响应使用统一格式
5. **版本管理** - 使用URL版本或Header版本
6. **文档完善** - 使用Swagger生成API文档

### 设计原则

1. **简单明了** - API设计要简单易懂
2. **一致性** - 保持API风格一致
3. **向后兼容** - 新版本兼容旧版本
4. **安全性** - 注意安全防护
5. **性能** - 考虑性能优化

---

**作者**: erik.zhou  
**最后更新**: 2024-01-04
