# Gson 完整教程

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
- **版本**: 2.10.x
- **官方文档**: https://github.com/google/gson
- **GitHub**: https://github.com/google/gson
- **学习难度**: ⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、泛型
- **文档来源**: Context7 - google/gson
- **最后更新**: 2024-01-04

### 什么是 Gson

Gson 是 Google 开发的一个 Java 库，用于将 Java 对象转换为 JSON 表示形式，也可以将 JSON 字符串转换为等效的 Java 对象。Gson 可以处理任意 Java 对象，包括你没有源代码的预先存在的对象。

### 核心优势

1. **API 简洁**: 使用简单，学习成本低
2. **功能完善**: 支持泛型、自定义序列化等
3. **Google 出品**: 稳定可靠，社区活跃
4. **灵活配置**: 提供 GsonBuilder 进行灵活配置
5. **类型安全**: 强类型支持，编译时检查

## 🎯 学习目标
- [ ] 掌握 Gson 的基本序列化和反序列化操作
- [ ] 理解 GsonBuilder 的配置和使用
- [ ] 掌握泛型集合的处理
- [ ] 能够自定义 TypeAdapter
- [ ] 了解 InstanceCreator 的使用
- [ ] 掌握注解的使用方法
- [ ] 理解性能优化技巧

## 📖 基础概念

### 1.1 核心类

| 类名 | 说明 | 主要方法 |
|------|------|---------|
| Gson | 核心类 | toJson()、fromJson() |
| GsonBuilder | 构建器 | create()、setPrettyPrinting() |
| TypeAdapter | 类型适配器 | write()、read() |
| JsonElement | JSON 元素 | getAsString()、getAsInt() |
| JsonObject | JSON 对象 | get()、add() |
| JsonArray | JSON 数组 | get()、add() |

### 1.2 应用场景

- **RESTful API**: 请求和响应的 JSON 处理
- **配置文件**: 读取和写入 JSON 配置
- **数据存储**: 将对象持久化为 JSON
- **Android 开发**: Android 应用中的 JSON 处理
- **数据交换**: 系统间数据传输

## 🔥 核心特性 (重点)

### 2.1 基本序列化和反序列化 🔥

#### 2.1.1 环境搭建

```xml
<!-- Maven 依赖 -->
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10.1</version>
</dependency>
```

#### 2.1.2 基本使用

```java
import com.google.gson.Gson;

/**
 * Gson 基本使用示例
 * @author erik.zhou
 */
public class BasicExample {
    
    /**
     * 定义 POJO 类
     */
    public static class Person {
        private String name;
        private int age;
        private String email;
        
        public Person(String name, int age, String email) {
            this.name = name;
            this.age = age;
            this.email = email;
        }
        
        // Getter 和 Setter
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        
        public int getAge() { return age; }
        public void setAge(int age) { this.age = age; }
        
        public String getEmail() { return email; }
        public void setEmail(String email) { this.email = email; }
    }
    
    public static void main(String[] args) {
        // 创建 Gson 实例
        Gson gson = new Gson();
        
        // ========== 序列化：Java 对象 -> JSON ==========
        Person person = new Person("张三", 25, "zhangsan@example.com");
        
        // 对象转 JSON 字符串
        String json = gson.toJson(person);
        System.out.println(json);
        // 输出: {"name":"张三","age":25,"email":"zhangsan@example.com"}
        
        // ========== 反序列化：JSON -> Java 对象 ==========
        String jsonString = "{\"name\":\"李四\",\"age\":30,\"email\":\"lisi@example.com\"}";
        
        // JSON 字符串转对象
        Person deserializedPerson = gson.fromJson(jsonString, Person.class);
        System.out.println(deserializedPerson.getName());  // 李四
    }
}
```

### 2.2 泛型集合处理

```java
import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;
import java.lang.reflect.Type;
import java.util.List;
import java.util.Map;

/**
 * 泛型集合处理示例
 * @author erik.zhou
 */
public class GenericExample {
    
    public static void main(String[] args) {
        Gson gson = new Gson();
        
        // ========== List 序列化和反序列化 ==========
        List<Person> personList = Arrays.asList(
            new Person("张三", 25, "zhangsan@example.com"),
            new Person("李四", 30, "lisi@example.com")
        );
        
        // List 转 JSON
        String listJson = gson.toJson(personList);
        
        // JSON 转 List（使用 TypeToken）
        Type listType = new TypeToken<List<Person>>(){}.getType();
        List<Person> deserializedList = gson.fromJson(listJson, listType);
        
        // ========== Map 序列化和反序列化 ==========
        Map<String, Person> personMap = new HashMap<>();
        personMap.put("user1", new Person("张三", 25, "zhangsan@example.com"));
        personMap.put("user2", new Person("李四", 30, "lisi@example.com"));
        
        // Map 转 JSON
        String mapJson = gson.toJson(personMap);
        
        // JSON 转 Map
        Type mapType = new TypeToken<Map<String, Person>>(){}.getType();
        Map<String, Person> deserializedMap = gson.fromJson(mapJson, mapType);
    }
}
```

### 2.3 GsonBuilder 配置 (⚠️ 难点)

GsonBuilder 提供了丰富的配置选项，用于自定义 Gson 的行为。

```java
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.FieldNamingPolicy;

/**
 * GsonBuilder 配置示例
 * @author erik.zhou
 */
public class GsonBuilderExample {
    
    public static Gson createGson() {
        return new GsonBuilder()
            // 1. 美化输出（格式化 JSON）
            .setPrettyPrinting()
            
            // 2. 序列化 null 值
            .serializeNulls()
            
            // 3. 日期格式化
            .setDateFormat("yyyy-MM-dd HH:mm:ss")
            
            // 4. 字段命名策略（驼峰转下划线）
            .setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES)
            
            // 5. 禁用 HTML 转义
            .disableHtmlEscaping()
            
            // 6. 排除没有 @Expose 注解的字段
            .excludeFieldsWithoutExposeAnnotation()
            
            // 7. 设置版本控制
            .setVersion(1.0)
            
            // 8. 禁用内部类序列化
            .disableInnerClassSerialization()
            
            // 9. 生成不可执行的 JSON（安全性）
            .generateNonExecutableJson()
            
            // 10. 宽松模式（允许非标准 JSON）
            .setLenient()
            
            .create();
    }
}
```

### 2.4 注解使用

```java
import com.google.gson.annotations.*;

/**
 * Gson 注解示例
 * @author erik.zhou
 */
public class AnnotationExample {
    
    /**
     * @SerializedName 注解
     */
    public static class User {
        
        // 指定序列化名称
        @SerializedName("userName")
        private String name;
        
        // 支持多个别名（反序列化时）
        @SerializedName(value = "userAge", alternate = {"age", "user_age"})
        private int age;
        
        // Getter 和 Setter
    }
    
    /**
     * @Expose 注解
     */
    public static class Account {
        
        @Expose  // 参与序列化和反序列化
        private String username;
        
        @Expose(serialize = true, deserialize = false)  // 只序列化
        private String token;
        
        @Expose(serialize = false, deserialize = true)  // 只反序列化
        private String password;
        
        private String internalId;  // 不参与（需配合 excludeFieldsWithoutExposeAnnotation）
        
        // Getter 和 Setter
    }
    
    /**
     * @Since 和 @Until 注解（版本控制）
     */
    public static class Product {
        
        private String name;
        
        @Since(1.0)  // 从版本 1.0 开始支持
        private double price;
        
        @Since(2.0)  // 从版本 2.0 开始支持
        private String description;
        
        @Until(1.5)  // 到版本 1.5 为止支持
        private String oldField;
        
        // Getter 和 Setter
    }
}
```

### 2.5 自定义 TypeAdapter (⚠️ 难点)

TypeAdapter 提供了对序列化和反序列化过程的完全控制。

```java
import com.google.gson.*;
import com.google.gson.stream.JsonReader;
import com.google.gson.stream.JsonWriter;
import java.io.IOException;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

/**
 * 自定义 TypeAdapter 示例
 * @author erik.zhou
 */
public class CustomTypeAdapterExample {
    
    /**
     * 自定义 LocalDateTime 适配器
     */
    public static class LocalDateTimeAdapter extends TypeAdapter<LocalDateTime> {
        
        private static final DateTimeFormatter FORMATTER = 
            DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        
        @Override
        public void write(JsonWriter out, LocalDateTime value) throws IOException {
            if (value == null) {
                out.nullValue();
            } else {
                out.value(value.format(FORMATTER));
            }
        }
        
        @Override
        public LocalDateTime read(JsonReader in) throws IOException {
            if (in.peek() == com.google.gson.stream.JsonToken.NULL) {
                in.nextNull();
                return null;
            }
            String dateString = in.nextString();
            return LocalDateTime.parse(dateString, FORMATTER);
        }
    }
    
    /**
     * 使用自定义适配器
     */
    public static void main(String[] args) {
        Gson gson = new GsonBuilder()
            .registerTypeAdapter(LocalDateTime.class, new LocalDateTimeAdapter())
            .create();
        
        Event event = new Event();
        event.setName("会议");
        event.setEventTime(LocalDateTime.now());
        
        // 序列化
        String json = gson.toJson(event);
        System.out.println(json);
        
        // 反序列化
        Event deserializedEvent = gson.fromJson(json, Event.class);
    }
    
    public static class Event {
        private String name;
        private LocalDateTime eventTime;
        
        // Getter 和 Setter
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        
        public LocalDateTime getEventTime() { return eventTime; }
        public void setEventTime(LocalDateTime eventTime) { this.eventTime = eventTime; }
    }
}
```

### 2.6 InstanceCreator

当类没有无参构造函数时，可以使用 InstanceCreator 创建实例。

```java
import com.google.gson.*;
import java.lang.reflect.Type;

/**
 * InstanceCreator 示例
 * @author erik.zhou
 */
public class InstanceCreatorExample {
    
    /**
     * 不可变对象（没有无参构造函数）
     */
    public static class ImmutablePerson {
        private final String name;
        private final int age;
        
        public ImmutablePerson(String name, int age) {
            this.name = name;
            this.age = age;
        }
        
        public String getName() { return name; }
        public int getAge() { return age; }
    }
    
    /**
     * 自定义 InstanceCreator
     */
    public static class ImmutablePersonCreator implements InstanceCreator<ImmutablePerson> {
        @Override
        public ImmutablePerson createInstance(Type type) {
            // 返回默认实例
            return new ImmutablePerson("Unknown", 0);
        }
    }
    
    public static void main(String[] args) {
        Gson gson = new GsonBuilder()
            .registerTypeAdapter(ImmutablePerson.class, new ImmutablePersonCreator())
            .create();
        
        String json = "{\"name\":\"张三\",\"age\":25}";
        ImmutablePerson person = gson.fromJson(json, ImmutablePerson.class);
        System.out.println(person.getName());  // 张三
    }
}
```

### 2.7 JsonElement 树模型

```java
import com.google.gson.*;

/**
 * JsonElement 树模型示例
 * @author erik.zhou
 */
public class JsonElementExample {
    
    public static void main(String[] args) {
        Gson gson = new Gson();
        
        // ========== 解析 JSON 为树模型 ==========
        String json = "{\"name\":\"张三\",\"age\":25,\"hobbies\":[\"读书\",\"运动\"]}";
        JsonElement element = gson.fromJson(json, JsonElement.class);
        
        // 转换为 JsonObject
        JsonObject jsonObject = element.getAsJsonObject();
        
        // 访问属性
        String name = jsonObject.get("name").getAsString();  // 张三
        int age = jsonObject.get("age").getAsInt();          // 25
        
        // 访问数组
        JsonArray hobbies = jsonObject.getAsJsonArray("hobbies");
        for (JsonElement hobby : hobbies) {
            System.out.println(hobby.getAsString());
        }
        
        // ========== 构建 JSON 树 ==========
        JsonObject personObject = new JsonObject();
        personObject.addProperty("name", "李四");
        personObject.addProperty("age", 30);
        
        JsonArray hobbiesArray = new JsonArray();
        hobbiesArray.add("游泳");
        hobbiesArray.add("旅游");
        personObject.add("hobbies", hobbiesArray);
        
        // 转换为 JSON 字符串
        String result = gson.toJson(personObject);
        System.out.println(result);
    }
}
```

## 💻 实战应用

### 3.1 Spring Boot 集成

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.json.GsonHttpMessageConverter;
import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

/**
 * Gson 配置类
 * @author erik.zhou
 */
@Configuration
public class GsonConfig {
    
    @Bean
    public Gson gson() {
        return new GsonBuilder()
            .setPrettyPrinting()
            .serializeNulls()
            .setDateFormat("yyyy-MM-dd HH:mm:ss")
            .disableHtmlEscaping()
            .create();
    }
    
    @Bean
    public GsonHttpMessageConverter gsonHttpMessageConverter(Gson gson) {
        GsonHttpMessageConverter converter = new GsonHttpMessageConverter();
        converter.setGson(gson);
        return converter;
    }
}
```

### 3.2 Controller 中使用

```java
import org.springframework.web.bind.annotation.*;
import com.google.gson.Gson;

/**
 * RESTful API 示例
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    private final Gson gson;
    
    public UserController(Gson gson) {
        this.gson = gson;
    }
    
    /**
     * 自动序列化返回值
     */
    @GetMapping("/{id}")
    public Person getUser(@PathVariable Long id) {
        // Spring 自动使用 Gson 序列化返回值
        return userService.findById(id);
    }
    
    /**
     * 自动反序列化请求体
     */
    @PostMapping
    public Person createUser(@RequestBody Person person) {
        // Spring 自动使用 Gson 反序列化请求体
        return userService.save(person);
    }
    
    /**
     * 手动使用 Gson
     */
    @PostMapping("/manual")
    public String manualProcess(@RequestBody String jsonString) {
        // 手动反序列化
        Person person = gson.fromJson(jsonString, Person.class);
        
        // 业务处理
        person = userService.save(person);
        
        // 手动序列化
        return gson.toJson(person);
    }
}
```

### 3.3 排除策略

```java
import com.google.gson.*;

/**
 * 排除策略示例
 * @author erik.zhou
 */
public class ExclusionStrategyExample {
    
    /**
     * 自定义排除策略
     */
    public static class CustomExclusionStrategy implements ExclusionStrategy {
        
        @Override
        public boolean shouldSkipField(FieldAttributes f) {
            // 排除特定字段
            return f.getName().equals("password") || f.getName().equals("internalId");
        }
        
        @Override
        public boolean shouldSkipClass(Class<?> clazz) {
            // 排除特定类
            return clazz == InternalData.class;
        }
    }
    
    public static void main(String[] args) {
        Gson gson = new GsonBuilder()
            .setExclusionStrategies(new CustomExclusionStrategy())
            .create();
        
        User user = new User();
        user.setName("张三");
        user.setPassword("secret");
        user.setInternalId("ID123");
        
        String json = gson.toJson(user);
        // password 和 internalId 不会被序列化
        System.out.println(json);
    }
}
```

### 3.4 处理多态类型

```java
import com.google.gson.*;
import com.google.gson.reflect.TypeToken;
import java.lang.reflect.Type;

/**
 * 多态类型处理示例
 * @author erik.zhou
 */
public class PolymorphismExample {
    
    /**
     * 自定义运行时类型适配器
     */
    public static class RuntimeTypeAdapterFactory<T> implements TypeAdapterFactory {
        
        private final Class<?> baseType;
        private final String typeFieldName;
        private final Map<String, Class<?>> labelToSubtype = new LinkedHashMap<>();
        
        private RuntimeTypeAdapterFactory(Class<?> baseType, String typeFieldName) {
            this.baseType = baseType;
            this.typeFieldName = typeFieldName;
        }
        
        public static <T> RuntimeTypeAdapterFactory<T> of(Class<T> baseType, String typeFieldName) {
            return new RuntimeTypeAdapterFactory<>(baseType, typeFieldName);
        }
        
        public RuntimeTypeAdapterFactory<T> registerSubtype(Class<? extends T> type, String label) {
            labelToSubtype.put(label, type);
            return this;
        }
        
        @Override
        public <R> TypeAdapter<R> create(Gson gson, TypeToken<R> type) {
            // 实现类型适配逻辑
            // 这里简化处理，实际需要完整实现
            return null;
        }
    }
    
    /**
     * 使用示例
     */
    public static void main(String[] args) {
        RuntimeTypeAdapterFactory<Animal> animalAdapter = 
            RuntimeTypeAdapterFactory.of(Animal.class, "type")
                .registerSubtype(Dog.class, "dog")
                .registerSubtype(Cat.class, "cat");
        
        Gson gson = new GsonBuilder()
            .registerTypeAdapterFactory(animalAdapter)
            .create();
        
        Animal dog = new Dog();
        dog.setName("旺财");
        
        String json = gson.toJson(dog);
        System.out.println(json);
        
        Animal animal = gson.fromJson(json, Animal.class);
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

```java
/**
 * 性能优化建议
 * @author erik.zhou
 */
public class PerformanceOptimization {
    
    /**
     * 1. 复用 Gson 实例（线程安全）
     */
    private static final Gson GSON = new GsonBuilder().create();
    
    public String serialize(Object obj) {
        return GSON.toJson(obj);
    }
    
    /**
     * 2. 使用 TypeToken 缓存
     */
    private static final Type LIST_TYPE = new TypeToken<List<Person>>(){}.getType();
    
    public List<Person> deserializeList(String json) {
        return GSON.fromJson(json, LIST_TYPE);
    }
    
    /**
     * 3. 禁用不必要的特性
     */
    public static Gson createOptimizedGson() {
        return new GsonBuilder()
            .disableHtmlEscaping()  // 禁用 HTML 转义
            .create();
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

// ✅ 或者使用 InstanceCreator
```

#### ⚠️ 陷阱 2: 泛型擦除

```java
// ❌ 错误示例：泛型擦除
List<Person> list = gson.fromJson(json, List.class);  // 运行时错误

// ✅ 正确示例：使用 TypeToken
Type listType = new TypeToken<List<Person>>(){}.getType();
List<Person> list = gson.fromJson(json, listType);
```

#### ⚠️ 陷阱 3: 循环引用

```java
/**
 * 处理循环引用
 * @author erik.zhou
 */
public class CircularReferenceExample {
    
    // ❌ Gson 不支持循环引用，会抛出异常
    public static class Parent {
        private String name;
        private Child child;
    }
    
    public static class Child {
        private String name;
        private Parent parent;  // 循环引用
    }
    
    // ✅ 解决方案：使用 @Expose 或排除策略
    public static class ParentFixed {
        @Expose
        private String name;
        
        @Expose
        private ChildFixed child;
    }
    
    public static class ChildFixed {
        @Expose
        private String name;
        
        // 不添加 @Expose，不参与序列化
        private ParentFixed parent;
    }
}
```

### 4.3 异常处理

```java
import com.google.gson.JsonSyntaxException;

/**
 * 异常处理示例
 * @author erik.zhou
 */
public class ExceptionHandling {
    
    public Person parseUser(String json) {
        try {
            return gson.fromJson(json, Person.class);
        } catch (JsonSyntaxException e) {
            // JSON 语法错误
            log.error("JSON 格式错误: {}", json, e);
            throw new BusinessException("数据格式错误");
        } catch (Exception e) {
            // 其他异常
            log.error("解析失败", e);
            throw new BusinessException("系统错误");
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
    
    /**
     * 1. 输入验证
     */
    public Person parseWithValidation(String json) {
        // 验证 JSON 格式
        if (json == null || json.trim().isEmpty()) {
            throw new IllegalArgumentException("JSON 不能为空");
        }
        
        // 限制 JSON 长度
        if (json.length() > 1024 * 1024) {  // 1MB
            throw new IllegalArgumentException("JSON 过大");
        }
        
        return gson.fromJson(json, Person.class);
    }
    
    /**
     * 2. 敏感数据处理
     */
    public static class SensitiveData {
        private String name;
        
        @Expose(serialize = false)  // 不序列化
        private String password;
        
        @Expose(deserialize = false)  // 不反序列化
        private String token;
        
        // Getter 和 Setter
    }
}
```

## ❓ 常见问题

### Q1: Gson、Jackson、FastJSON 如何选择？

**A**:
- **Jackson**: 功能最强大，Spring Boot 默认，推荐使用
- **Gson**: API 简单，适合简单场景，Google 出品
- **FastJSON**: 性能最快，但安全漏洞多，不推荐

### Q2: 如何处理 null 值？

**A**:
```java
// 序列化 null 值
Gson gson = new GsonBuilder()
    .serializeNulls()
    .create();

// 忽略 null 值（默认行为）
Gson gson = new Gson();
```

### Q3: 如何美化 JSON 输出？

**A**:
```java
Gson gson = new GsonBuilder()
    .setPrettyPrinting()
    .create();
```

### Q4: 如何处理日期格式？

**A**:
```java
// 方式1: 全局配置
Gson gson = new GsonBuilder()
    .setDateFormat("yyyy-MM-dd HH:mm:ss")
    .create();

// 方式2: 自定义 TypeAdapter
// 见 2.5 自定义 TypeAdapter
```

### Q5: 如何处理枚举类型？

**A**:
```java
// Gson 默认使用枚举的 name() 方法
public enum Status {
    ACTIVE, INACTIVE
}

// 自定义枚举序列化
public enum Status {
    ACTIVE(1), INACTIVE(0);
    
    private final int code;
    
    Status(int code) {
        this.code = code;
    }
    
    public int getCode() {
        return code;
    }
}

// 需要自定义 TypeAdapter
```

### Q6: 如何处理内部类？

**A**:
```java
// ❌ 非静态内部类会失败
public class Outer {
    public class Inner {
        private String name;
    }
}

// ✅ 使用静态内部类
public class Outer {
    public static class Inner {
        private String name;
    }
}
```

### Q7: 如何处理字段命名策略？

**A**:
```java
Gson gson = new GsonBuilder()
    .setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES)
    .create();

// Java: userName -> JSON: user_name
```

## 🔗 相关资源

### 官方资源
- **GitHub**: https://github.com/google/gson
- **用户指南**: https://github.com/google/gson/blob/main/UserGuide.md
- **API 文档**: https://www.javadoc.io/doc/com.google.code.gson/gson

### 推荐文章
- 《Gson 用户指南》
- 《Gson 高级特性详解》
- 《Gson vs Jackson 性能对比》

### 相关技术
- **Jackson**: 推荐的 JSON 处理库
- **FastJSON**: 阿里巴巴的 JSON 处理库
- **JSON-B**: Java EE 标准 JSON 绑定 API

## 📝 学习检查清单

- [ ] 理解 Gson 的核心类和 API
- [ ] 掌握基本的序列化和反序列化操作
- [ ] 掌握 GsonBuilder 的配置和使用
- [ ] 能够处理泛型集合
- [ ] 掌握常用注解的使用
- [ ] 了解自定义 TypeAdapter 的实现
- [ ] 了解 InstanceCreator 的使用场景
- [ ] 掌握 JsonElement 树模型的使用
- [ ] 掌握 Spring Boot 集成方式
- [ ] 理解性能优化技巧
- [ ] 能够处理常见的异常情况
- [ ] 理解 Gson 与其他库的区别

---

**@author erik.zhou**  
**文档来源**: Context7 - google/gson  
**最后更新**: 2024-01-04
