# Jackson 完整教程

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
- **版本**: 2.16.x
- **官方文档**: https://github.com/FasterXML/jackson
- **GitHub**: https://github.com/FasterXML/jackson-databind
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、泛型、反射
- **文档来源**: Context7 - fasterxml/jackson-databind
- **最后更新**: 2024-01-04

### 什么是 Jackson

Jackson 是一个高性能的 Java JSON 处理库，用于将 Java 对象序列化为 JSON 字符串，以及将 JSON 字符串反序列化为 Java 对象。它是目前 Java 生态中最流行的 JSON 处理库，被广泛应用于 Spring Boot、RESTful API 等场景。

### 核心优势

1. **性能优异**: 序列化和反序列化速度快
2. **功能强大**: 支持复杂对象、泛型、多态等
3. **注解丰富**: 提供大量注解控制序列化行为
4. **Spring 集成**: Spring Boot 默认使用 Jackson
5. **社区活跃**: 持续更新，文档完善

## 🎯 学习目标
- [ ] 掌握 Jackson 的基本序列化和反序列化操作
- [ ] 理解 ObjectMapper 的配置和使用
- [ ] 掌握常用注解的使用方法
- [ ] 能够处理复杂的 JSON 结构
- [ ] 掌握自定义序列化器和反序列化器
- [ ] 了解 Jackson 在 Spring Boot 中的集成
- [ ] 掌握性能优化技巧

## 📖 基础概念

### 1.1 Jackson 核心模块

Jackson 由三个核心模块组成：

| 模块 | 说明 | 依赖 |
|------|------|------|
| jackson-core | 核心包，定义流式 API | 必需 |
| jackson-annotations | 注解包，提供标准注解 | 推荐 |
| jackson-databind | 数据绑定包，提供 ObjectMapper | 推荐 |

### 1.2 核心类

- **ObjectMapper**: 核心类，用于序列化和反序列化
- **JsonNode**: JSON 树模型，用于操作 JSON 结构
- **JsonParser**: JSON 解析器，流式读取
- **JsonGenerator**: JSON 生成器，流式写入

### 1.3 应用场景

- **RESTful API**: 请求和响应的 JSON 处理
- **配置文件**: 读取和写入 JSON 配置
- **数据存储**: 将对象持久化为 JSON
- **消息队列**: JSON 格式的消息传递
- **日志记录**: 结构化日志输出

## 🔥 核心特性 (重点)

### 2.1 基本序列化和反序列化 🔥

#### 2.1.1 环境搭建

```xml
<!-- Maven 依赖 -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.16.1</version>
</dependency>
```

#### 2.1.2 基本使用

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import java.io.File;
import java.io.IOException;

/**
 * Jackson 基本使用示例
 * @author erik.zhou
 */
public class BasicExample {
    
    /**
     * 定义 POJO 类
     */
    public static class Person {
        public String name;
        public int age;
        public String email;
        
        // 无参构造函数（反序列化必需）
        public Person() {}
        
        public Person(String name, int age, String email) {
            this.name = name;
            this.age = age;
            this.email = email;
        }
    }
    
    public static void main(String[] args) throws IOException {
        // 创建 ObjectMapper 实例（建议复用）
        ObjectMapper mapper = new ObjectMapper();
        
        // ========== 序列化：Java 对象 -> JSON ==========
        Person person = new Person("John Doe", 30, "john@example.com");
        
        // 1. 转换为 JSON 字符串
        String jsonString = mapper.writeValueAsString(person);
        System.out.println(jsonString);
        // 输出: {"name":"John Doe","age":30,"email":"john@example.com"}
        
        // 2. 转换为字节数组
        byte[] jsonBytes = mapper.writeValueAsBytes(person);
        
        // 3. 写入文件
        mapper.writeValue(new File("person.json"), person);
        
        // ========== 反序列化：JSON -> Java 对象 ==========
        String json = "{\"name\":\"Jane Smith\",\"age\":25,\"email\":\"jane@example.com\"}";
        
        // 1. 从字符串反序列化
        Person deserializedPerson = mapper.readValue(json, Person.class);
        System.out.println(deserializedPerson.name);  // Jane Smith
        
        // 2. 从文件反序列化
        Person fromFile = mapper.readValue(new File("person.json"), Person.class);
        
        // 3. 从 URL 反序列化
        Person fromUrl = mapper.readValue(
            new java.net.URL("http://api.example.com/person/1"),
            Person.class
        );
    }
}
```

### 2.2 常用注解 🔥

#### 2.2.1 @JsonProperty - 属性重命名

```java
import com.fasterxml.jackson.annotation.JsonProperty;

/**
 * 属性重命名示例
 * @author erik.zhou
 */
public class User {
    
    /**
     * 将 Java 属性 name 映射为 JSON 的 userName
     */
    @JsonProperty("userName")
    private String name;
    
    private int age;
    
    // Getter 和 Setter
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public int getAge() { return age; }
    public void setAge(int age) { this.age = age; }
}

// 序列化结果: {"userName":"张三","age":25}
```

#### 2.2.2 @JsonIgnore - 忽略属性

```java
import com.fasterxml.jackson.annotation.JsonIgnore;

/**
 * 忽略属性示例
 * @author erik.zhou
 */
public class Account {
    
    private String username;
    
    /**
     * 密码字段不参与序列化和反序列化
     */
    @JsonIgnore
    private String password;
    
    private String email;
    
    // Getter 和 Setter
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    
    public String getPassword() { return password; }
    public void setPassword(String password) { this.password = password; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}

// 序列化结果: {"username":"admin","email":"admin@example.com"}
// password 字段被忽略
```

#### 2.2.3 @JsonIgnoreProperties - 批量忽略属性

```java
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

/**
 * 批量忽略属性示例
 * @author erik.zhou
 */
@JsonIgnoreProperties({"internalId", "tempData"})
public class Product {
    
    private String name;
    private double price;
    private String internalId;  // 被忽略
    private String tempData;    // 被忽略
    
    // Getter 和 Setter
}

// 序列化结果: {"name":"商品A","price":99.99}
```

#### 2.2.4 @JsonAlias - 多个属性名映射

```java
import com.fasterxml.jackson.annotation.JsonAlias;
import com.fasterxml.jackson.annotation.JsonProperty;

/**
 * 多属性名映射示例
 * @author erik.zhou
 */
public class UserInfo {
    
    /**
     * 反序列化时，user_name、userName、name 都可以映射到 username
     * 序列化时，使用 username
     */
    @JsonAlias({"user_name", "userName", "name"})
    @JsonProperty("username")
    private String username;
    
    // Getter 和 Setter
}

// 以下 JSON 都可以正确反序列化：
// {"user_name":"张三"}
// {"userName":"张三"}
// {"name":"张三"}
// {"username":"张三"}
```

#### 2.2.5 @JsonCreator - 自定义构造函数

```java
import com.fasterxml.jackson.annotation.JsonCreator;
import com.fasterxml.jackson.annotation.JsonProperty;

/**
 * 自定义构造函数示例（不可变对象）
 * @author erik.zhou
 */
public class ImmutablePerson {
    
    public final String name;
    public final int age;
    
    /**
     * 使用 @JsonCreator 指定反序列化使用的构造函数
     */
    @JsonCreator
    private ImmutablePerson(
            @JsonProperty("name") String name,
            @JsonProperty("age") int age) {
        this.name = name;
        this.age = age;
    }
    
    // 只提供 Getter，不提供 Setter（不可变对象）
    public String getName() { return name; }
    public int getAge() { return age; }
}
```

#### 2.2.6 @JsonFormat - 日期格式化

```java
import com.fasterxml.jackson.annotation.JsonFormat;
import java.util.Date;

/**
 * 日期格式化示例
 * @author erik.zhou
 */
public class Order {
    
    private String orderNo;
    
    /**
     * 指定日期格式和时区
     */
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date createTime;
    
    // Getter 和 Setter
}

// 序列化结果: {"orderNo":"20240104001","createTime":"2024-01-04 10:30:00"}
```

### 2.3 ObjectMapper 配置 (⚠️ 难点)

ObjectMapper 提供了丰富的配置选项，用于控制序列化和反序列化的行为。

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.databind.SerializationFeature;
import com.fasterxml.jackson.databind.DeserializationFeature;
import com.fasterxml.jackson.annotation.JsonInclude;

/**
 * ObjectMapper 配置示例
 * @author erik.zhou
 */
public class ObjectMapperConfig {
    
    public static ObjectMapper createMapper() {
        ObjectMapper mapper = new ObjectMapper();
        
        // ========== 序列化配置 ==========
        
        // 1. 格式化输出（美化 JSON）
        mapper.enable(SerializationFeature.INDENT_OUTPUT);
        
        // 2. 忽略空值属性
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        
        // 3. 忽略空集合
        mapper.setSerializationInclusion(JsonInclude.Include.NON_EMPTY);
        
        // 4. 日期格式化为时间戳
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        
        // ========== 反序列化配置 ==========
        
        // 1. 忽略未知属性（JSON 中有但 Java 类中没有的属性）
        mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        
        // 2. 允许空字符串转换为 null
        mapper.enable(DeserializationFeature.ACCEPT_EMPTY_STRING_AS_NULL_OBJECT);
        
        // 3. 允许单引号
        mapper.configure(com.fasterxml.jackson.core.JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
        
        // 4. 允许不带引号的字段名
        mapper.configure(com.fasterxml.jackson.core.JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
        
        return mapper;
    }
}
```

### 2.4 处理泛型集合

```java
import com.fasterxml.jackson.core.type.TypeReference;
import java.util.List;
import java.util.Map;

/**
 * 泛型集合处理示例
 * @author erik.zhou
 */
public class GenericExample {
    
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        
        // ========== List 序列化和反序列化 ==========
        List<Person> personList = Arrays.asList(
            new Person("张三", 25, "zhangsan@example.com"),
            new Person("李四", 30, "lisi@example.com")
        );
        
        // 序列化
        String json = mapper.writeValueAsString(personList);
        
        // 反序列化（使用 TypeReference）
        List<Person> deserializedList = mapper.readValue(
            json,
            new TypeReference<List<Person>>() {}
        );
        
        // ========== Map 序列化和反序列化 ==========
        Map<String, Person> personMap = new HashMap<>();
        personMap.put("user1", new Person("张三", 25, "zhangsan@example.com"));
        personMap.put("user2", new Person("李四", 30, "lisi@example.com"));
        
        // 序列化
        String mapJson = mapper.writeValueAsString(personMap);
        
        // 反序列化
        Map<String, Person> deserializedMap = mapper.readValue(
            mapJson,
            new TypeReference<Map<String, Person>>() {}
        );
    }
}
```

### 2.5 JsonNode 树模型

```java
import com.fasterxml.jackson.databind.JsonNode;
import com.fasterxml.jackson.databind.node.ObjectNode;
import com.fasterxml.jackson.databind.node.ArrayNode;

/**
 * JsonNode 树模型示例
 * @author erik.zhou
 */
public class JsonNodeExample {
    
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        
        // ========== 读取 JSON 为树模型 ==========
        String json = "{\"name\":\"张三\",\"age\":25,\"hobbies\":[\"读书\",\"运动\"]}";
        JsonNode rootNode = mapper.readTree(json);
        
        // 访问属性
        String name = rootNode.get("name").asText();  // 张三
        int age = rootNode.get("age").asInt();        // 25
        
        // 访问数组
        JsonNode hobbiesNode = rootNode.get("hobbies");
        if (hobbiesNode.isArray()) {
            for (JsonNode hobby : hobbiesNode) {
                System.out.println(hobby.asText());
            }
        }
        
        // ========== 构建 JSON 树 ==========
        ObjectNode personNode = mapper.createObjectNode();
        personNode.put("name", "李四");
        personNode.put("age", 30);
        
        ArrayNode hobbiesArray = mapper.createArrayNode();
        hobbiesArray.add("游泳");
        hobbiesArray.add("旅游");
        personNode.set("hobbies", hobbiesArray);
        
        // 转换为 JSON 字符串
        String result = mapper.writeValueAsString(personNode);
        System.out.println(result);
    }
}
```

## 💻 实战应用

### 3.1 Spring Boot 集成

Spring Boot 默认使用 Jackson 作为 JSON 处理库。

#### 3.1.1 自定义 ObjectMapper

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.json.Jackson2ObjectMapperBuilder;
import com.fasterxml.jackson.databind.ObjectMapper;

/**
 * Jackson 配置类
 * @author erik.zhou
 */
@Configuration
public class JacksonConfig {
    
    @Bean
    public ObjectMapper objectMapper(Jackson2ObjectMapperBuilder builder) {
        ObjectMapper mapper = builder.createXmlMapper(false).build();
        
        // 忽略未知属性
        mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        
        // 忽略空值
        mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);
        
        // 日期格式化
        mapper.setDateFormat(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss"));
        
        return mapper;
    }
}
```

#### 3.1.2 Controller 中使用

```java
import org.springframework.web.bind.annotation.*;
import com.fasterxml.jackson.databind.ObjectMapper;

/**
 * RESTful API 示例
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final ObjectMapper objectMapper;
    
    public UserController(ObjectMapper objectMapper) {
        this.objectMapper = objectMapper;
    }
    
    /**
     * 自动序列化返回值
     */
    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        // Spring 自动使用 Jackson 序列化返回值
        return userService.findById(id);
    }
    
    /**
     * 自动反序列化请求体
     */
    @PostMapping
    public User createUser(@RequestBody User user) {
        // Spring 自动使用 Jackson 反序列化请求体
        return userService.save(user);
    }
    
    /**
     * 手动使用 ObjectMapper
     */
    @PostMapping("/manual")
    public String manualProcess(@RequestBody String jsonString) throws Exception {
        // 手动反序列化
        User user = objectMapper.readValue(jsonString, User.class);
        
        // 业务处理
        user = userService.save(user);
        
        // 手动序列化
        return objectMapper.writeValueAsString(user);
    }
}
```

### 3.2 自定义序列化器

```java
import com.fasterxml.jackson.core.JsonGenerator;
import com.fasterxml.jackson.databind.JsonSerializer;
import com.fasterxml.jackson.databind.SerializerProvider;
import com.fasterxml.jackson.databind.annotation.JsonSerialize;
import java.io.IOException;
import java.math.BigDecimal;

/**
 * 自定义金额序列化器
 * @author erik.zhou
 */
public class MoneySerializer extends JsonSerializer<BigDecimal> {
    
    @Override
    public void serialize(BigDecimal value, JsonGenerator gen, SerializerProvider serializers)
            throws IOException {
        if (value != null) {
            // 保留两位小数
            gen.writeString(value.setScale(2, BigDecimal.ROUND_HALF_UP).toString());
        }
    }
}

/**
 * 使用自定义序列化器
 */
public class Product {
    
    private String name;
    
    @JsonSerialize(using = MoneySerializer.class)
    private BigDecimal price;
    
    // Getter 和 Setter
}

// 序列化结果: {"name":"商品A","price":"99.99"}
```

### 3.3 自定义反序列化器

```java
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.databind.DeserializationContext;
import com.fasterxml.jackson.databind.JsonDeserializer;
import com.fasterxml.jackson.databind.annotation.JsonDeserialize;
import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * 自定义日期反序列化器
 * @author erik.zhou
 */
public class LocalDateTimeDeserializer extends JsonDeserializer<LocalDateTime> {
    
    private static final DateTimeFormatter FORMATTER = 
        DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    
    @Override
    public LocalDateTime deserialize(JsonParser p, DeserializationContext ctxt)
            throws IOException {
        String dateString = p.getText();
        return LocalDateTime.parse(dateString, FORMATTER);
    }
}

/**
 * 使用自定义反序列化器
 */
public class Event {
    
    private String name;
    
    @JsonDeserialize(using = LocalDateTimeDeserializer.class)
    private LocalDateTime eventTime;
    
    // Getter 和 Setter
}
```

### 3.4 处理多态类型

```java
import com.fasterxml.jackson.annotation.JsonSubTypes;
import com.fasterxml.jackson.annotation.JsonTypeInfo;

/**
 * 多态类型处理示例
 * @author erik.zhou
 */
@JsonTypeInfo(
    use = JsonTypeInfo.Id.NAME,
    include = JsonTypeInfo.As.PROPERTY,
    property = "type"
)
@JsonSubTypes({
    @JsonSubTypes.Type(value = Dog.class, name = "dog"),
    @JsonSubTypes.Type(value = Cat.class, name = "cat")
})
public abstract class Animal {
    private String name;
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

public class Dog extends Animal {
    private String breed;
    
    public String getBreed() { return breed; }
    public void setBreed(String breed) { this.breed = breed; }
}

public class Cat extends Animal {
    private int lives;
    
    public int getLives() { return lives; }
    public void setLives(int lives) { this.lives = lives; }
}

// 使用示例
public class PolymorphismExample {
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        
        // 序列化
        Animal dog = new Dog();
        dog.setName("旺财");
        ((Dog) dog).setBreed("金毛");
        
        String json = mapper.writeValueAsString(dog);
        // 输出: {"type":"dog","name":"旺财","breed":"金毛"}
        
        // 反序列化（自动识别类型）
        Animal animal = mapper.readValue(json, Animal.class);
        if (animal instanceof Dog) {
            System.out.println("这是一只狗");
        }
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

#### 4.1.1 复用 ObjectMapper

```java
/**
 * ObjectMapper 性能优化
 * @author erik.zhou
 */
public class PerformanceOptimization {
    
    // ❌ 错误示例：每次都创建新的 ObjectMapper
    public String badExample(Object obj) throws Exception {
        ObjectMapper mapper = new ObjectMapper();  // 创建开销大
        return mapper.writeValueAsString(obj);
    }
    
    // ✅ 正确示例：复用 ObjectMapper（线程安全）
    private static final ObjectMapper MAPPER = new ObjectMapper();
    
    public String goodExample(Object obj) throws Exception {
        return MAPPER.writeValueAsString(obj);
    }
}
```

#### 4.1.2 使用 @JsonView 减少序列化字段

```java
import com.fasterxml.jackson.annotation.JsonView;

/**
 * JsonView 示例
 * @author erik.zhou
 */
public class ViewExample {
    
    // 定义视图
    public static class Views {
        public static class Public {}
        public static class Internal extends Public {}
    }
    
    public static class User {
        @JsonView(Views.Public.class)
        private String name;
        
        @JsonView(Views.Public.class)
        private int age;
        
        @JsonView(Views.Internal.class)
        private String password;
        
        @JsonView(Views.Internal.class)
        private String internalId;
        
        // Getter 和 Setter
    }
    
    public static void main(String[] args) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        User user = new User();
        user.setName("张三");
        user.setAge(25);
        user.setPassword("secret");
        user.setInternalId("ID123");
        
        // 使用 Public 视图（只序列化 name 和 age）
        String publicJson = mapper
            .writerWithView(Views.Public.class)
            .writeValueAsString(user);
        // 输出: {"name":"张三","age":25}
        
        // 使用 Internal 视图（序列化所有字段）
        String internalJson = mapper
            .writerWithView(Views.Internal.class)
            .writeValueAsString(user);
        // 输出: {"name":"张三","age":25,"password":"secret","internalId":"ID123"}
    }
}
```

### 4.2 常见陷阱

#### ⚠️ 陷阱 1: 忘记提供无参构造函数

```java
// ❌ 错误示例：没有无参构造函数
public class User {
    private String name;
    private int age;
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
    // 反序列化会失败！
}

// ✅ 正确示例：提供无参构造函数
public class User {
    private String name;
    private int age;
    
    public User() {}  // 必需
    
    public User(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

#### ⚠️ 陷阱 2: 循环引用导致栈溢出

```java
import com.fasterxml.jackson.annotation.JsonManagedReference;
import com.fasterxml.jackson.annotation.JsonBackReference;

/**
 * 处理循环引用
 * @author erik.zhou
 */
public class CircularReferenceExample {
    
    // ❌ 错误示例：循环引用
    public static class Parent {
        private String name;
        private Child child;
        // 序列化会导致栈溢出！
    }
    
    public static class Child {
        private String name;
        private Parent parent;  // 循环引用
    }
    
    // ✅ 正确示例：使用注解处理循环引用
    public static class ParentFixed {
        private String name;
        
        @JsonManagedReference  // 正向引用
        private ChildFixed child;
    }
    
    public static class ChildFixed {
        private String name;
        
        @JsonBackReference  // 反向引用（不序列化）
        private ParentFixed parent;
    }
}
```

#### ⚠️ 陷阱 3: 日期格式不一致

```java
import com.fasterxml.jackson.databind.ObjectMapper;
import com.fasterxml.jackson.datatype.jsr310.JavaTimeModule;
import java.time.LocalDateTime;

/**
 * 日期处理最佳实践
 * @author erik.zhou
 */
public class DateHandling {
    
    public static ObjectMapper createMapper() {
        ObjectMapper mapper = new ObjectMapper();
        
        // 注册 Java 8 日期时间模块
        mapper.registerModule(new JavaTimeModule());
        
        // 禁用时间戳格式
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        
        return mapper;
    }
}
```

### 4.3 异常处理

```java
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.exc.InvalidFormatException;
import com.fasterxml.jackson.databind.exc.UnrecognizedPropertyException;

/**
 * 异常处理示例
 * @author erik.zhou
 */
public class ExceptionHandling {
    
    public User parseUser(String json) {
        ObjectMapper mapper = new ObjectMapper();
        
        try {
            return mapper.readValue(json, User.class);
        } catch (UnrecognizedPropertyException e) {
            // JSON 中有未知属性
            log.error("JSON 包含未知属性: {}", e.getPropertyName(), e);
            throw new BusinessException("数据格式错误");
        } catch (InvalidFormatException e) {
            // 数据格式不匹配
            log.error("数据格式不匹配: {}", e.getValue(), e);
            throw new BusinessException("数据格式错误");
        } catch (JsonProcessingException e) {
            // JSON 解析异常
            log.error("JSON 解析失败", e);
            throw new BusinessException("数据解析失败");
        }
    }
}
```

### 4.4 安全性考虑

```java
/**
 * 安全性最佳实践
 * @author erik.zhou
 */
public class SecurityBestPractices {
    
    public static ObjectMapper createSecureMapper() {
        ObjectMapper mapper = new ObjectMapper();
        
        // 1. 禁用默认类型处理（防止反序列化漏洞）
        mapper.deactivateDefaultTyping();
        
        // 2. 限制反序列化的类
        mapper.activateDefaultTyping(
            mapper.getPolymorphicTypeValidator(),
            ObjectMapper.DefaultTyping.NON_FINAL,
            JsonTypeInfo.As.PROPERTY
        );
        
        // 3. 忽略未知属性（防止恶意数据）
        mapper.disable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
        
        return mapper;
    }
    
    /**
     * 敏感数据脱敏
     */
    public static class SensitiveData {
        private String name;
        
        @JsonIgnore  // 不序列化敏感信息
        private String password;
        
        @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)  // 只允许写入
        private String idCard;
        
        // Getter 和 Setter
    }
}
```

## ❓ 常见问题

### Q1: Jackson 和 Gson、FastJSON 如何选择？

**A**:
- **Jackson**: 功能最强大，性能优异，Spring Boot 默认，推荐使用
- **Gson**: Google 出品，API 简单，适合简单场景
- **FastJSON**: 阿里出品，性能最快，但安全漏洞较多，不推荐

### Q2: 如何处理枚举类型？

**A**:
```java
import com.fasterxml.jackson.annotation.JsonValue;
import com.fasterxml.jackson.annotation.JsonCreator;

public enum Status {
    ACTIVE(1, "激活"),
    INACTIVE(0, "未激活");
    
    private final int code;
    private final String desc;
    
    Status(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }
    
    @JsonValue  // 序列化时使用 code
    public int getCode() {
        return code;
    }
    
    @JsonCreator  // 反序列化时根据 code 创建枚举
    public static Status fromCode(int code) {
        for (Status status : Status.values()) {
            if (status.code == code) {
                return status;
            }
        }
        throw new IllegalArgumentException("Invalid status code: " + code);
    }
}
```

### Q3: 如何处理 null 值？

**A**:
```java
// 全局配置
mapper.setSerializationInclusion(JsonInclude.Include.NON_NULL);

// 类级别配置
@JsonInclude(JsonInclude.Include.NON_NULL)
public class User {
    // ...
}

// 字段级别配置
public class User {
    @JsonInclude(JsonInclude.Include.NON_NULL)
    private String email;
}
```

### Q4: 如何美化 JSON 输出？

**A**:
```java
// 方式1: 全局配置
mapper.enable(SerializationFeature.INDENT_OUTPUT);

// 方式2: 单次使用
String prettyJson = mapper.writerWithDefaultPrettyPrinter()
    .writeValueAsString(obj);
```

### Q5: 如何处理大 JSON 文件？

**A**:
```java
import com.fasterxml.jackson.core.JsonParser;
import com.fasterxml.jackson.core.JsonToken;

/**
 * 流式处理大 JSON 文件
 * @author erik.zhou
 */
public class LargeJsonHandling {
    
    public void processLargeJson(String filePath) throws Exception {
        ObjectMapper mapper = new ObjectMapper();
        
        try (JsonParser parser = mapper.getFactory().createParser(new File(filePath))) {
            // 流式读取
            while (parser.nextToken() != JsonToken.END_ARRAY) {
                if (parser.currentToken() == JsonToken.START_OBJECT) {
                    User user = mapper.readValue(parser, User.class);
                    // 处理单个对象
                    processUser(user);
                }
            }
        }
    }
}
```

## 🔗 相关资源

### 官方资源
- **GitHub**: https://github.com/FasterXML/jackson
- **官方文档**: https://github.com/FasterXML/jackson-docs
- **Wiki**: https://github.com/FasterXML/jackson-databind/wiki

### 推荐文章
- 《Jackson 官方文档》
- 《Jackson 注解完全指南》
- 《Jackson 性能优化实践》

### 相关技术
- **Gson**: Google 的 JSON 处理库
- **FastJSON**: 阿里巴巴的 JSON 处理库
- **JSON-B**: Java EE 标准 JSON 绑定 API

## 📝 学习检查清单

- [ ] 理解 Jackson 的核心模块和架构
- [ ] 掌握基本的序列化和反序列化操作
- [ ] 掌握 ObjectMapper 的配置和使用
- [ ] 熟练使用常用注解（@JsonProperty、@JsonIgnore 等）
- [ ] 能够处理泛型集合和复杂对象
- [ ] 掌握 JsonNode 树模型的使用
- [ ] 了解自定义序列化器和反序列化器
- [ ] 掌握 Spring Boot 中的集成方式
- [ ] 理解性能优化的关键点
- [ ] 能够处理循环引用、日期格式等常见问题
- [ ] 了解安全性最佳实践
- [ ] 掌握异常处理机制

---

**@author erik.zhou**  
**文档来源**: Context7 - fasterxml/jackson-databind  
**最后更新**: 2024-01-04
