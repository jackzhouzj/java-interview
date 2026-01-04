# FastJSON 完整教程

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
- **版本**: 2.0.x (FastJSON 2)
- **官方文档**: https://github.com/alibaba/fastjson2
- **GitHub**: https://github.com/alibaba/fastjson
- **学习难度**: ⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐ (1-5星)
- **前置知识**: Java基础
- **文档来源**: Context7 - alibaba/fastjson
- **最后更新**: 2024-01-04

### 什么是 FastJSON

FastJSON 是阿里巴巴开源的一个高性能 JSON 处理库，专为 Java 服务端和 Android 客户端设计。它提供简洁高效的 API，用于 Java 对象与 JSON 字符串之间的相互转换。

### 核心优势

1. **性能极致**: 序列化和反序列化速度业界领先
2. **API 简洁**: 静态方法调用，使用简单
3. **功能丰富**: 支持 JSONPath、流式处理等高级特性
4. **内存占用低**: 优化的内存使用策略

### ⚠️ 安全警告

FastJSON 历史上存在多个安全漏洞，建议：
- 使用最新版本（2.0.x）
- 禁用 AutoType 功能
- 生产环境优先考虑 Jackson

## 🎯 学习目标
- [ ] 掌握 FastJSON 的基本序列化和反序列化操作
- [ ] 理解 Feature 配置的使用
- [ ] 掌握 JSONPath 查询语法
- [ ] 能够使用流式 API 处理大文件
- [ ] 了解性能优化技巧
- [ ] 理解安全风险和防范措施

## 📖 基础概念

### 1.1 核心类

| 类名 | 说明 | 主要方法 |
|------|------|---------|
| JSON | 核心入口类 | toJSONString()、parseObject() |
| JSONObject | JSON 对象 | get()、put()、getJSONObject() |
| JSONArray | JSON 数组 | get()、add()、getJSONObject() |
| JSONPath | JSON 路径查询 | eval()、extract()、set() |
| JSONReader | 流式读取 | readObject()、startArray() |
| JSONWriter | 流式写入 | writeValue()、startArray() |

### 1.2 应用场景

- **RESTful API**: 请求和响应的 JSON 处理
- **配置文件**: 读取和写入 JSON 配置
- **缓存序列化**: Redis 等缓存的数据序列化
- **日志记录**: 结构化日志输出
- **数据交换**: 系统间数据传输

## 🔥 核心特性 (重点)

### 2.1 基本序列化和反序列化 🔥

#### 2.1.1 环境搭建

```xml
<!-- Maven 依赖 - FastJSON 2 (推荐) -->
<dependency>
    <groupId>com.alibaba.fastjson2</groupId>
    <artifactId>fastjson2</artifactId>
    <version>2.0.45</version>
</dependency>

<!-- 或者 FastJSON 1.x (不推荐，存在安全漏洞) -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.83</version>
</dependency>
```

#### 2.1.2 基本使用

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;

/**
 * FastJSON 基本使用示例
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
        
        // Getter 和 Setter
        public String getName() { return name; }
        public void setName(String name) { this.name = name; }
        
        public int getAge() { return age; }
        public void setAge(int age) { this.age = age; }
        
        public String getEmail() { return email; }
        public void setEmail(String email) { this.email = email; }
    }
    
    public static void main(String[] args) {
        // ========== 序列化：Java 对象 -> JSON ==========
        Person person = new Person();
        person.setName("张三");
        person.setAge(25);
        person.setEmail("zhangsan@example.com");
        
        // 对象转 JSON 字符串
        String jsonString = JSON.toJSONString(person);
        System.out.println(jsonString);
        // 输出: {"age":25,"email":"zhangsan@example.com","name":"张三"}
        
        // ========== 反序列化：JSON -> Java 对象 ==========
        String json = "{\"name\":\"李四\",\"age\":30,\"email\":\"lisi@example.com\"}";
        
        // JSON 字符串转对象
        Person deserializedPerson = JSON.parseObject(json, Person.class);
        System.out.println(deserializedPerson.getName());  // 李四
        
        // JSON 字符串转 JSONObject
        JSONObject jsonObject = JSON.parseObject(json);
        String name = jsonObject.getString("name");
        int age = jsonObject.getIntValue("age");
    }
}
```

### 2.2 集合处理

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.TypeReference;
import java.util.List;
import java.util.Map;

/**
 * 集合处理示例
 * @author erik.zhou
 */
public class CollectionExample {
    
    public static void main(String[] args) {
        // ========== List 序列化和反序列化 ==========
        List<Person> personList = Arrays.asList(
            new Person("张三", 25, "zhangsan@example.com"),
            new Person("李四", 30, "lisi@example.com")
        );
        
        // List 转 JSON
        String listJson = JSON.toJSONString(personList);
        
        // JSON 转 List（使用 TypeReference）
        List<Person> deserializedList = JSON.parseObject(
            listJson,
            new TypeReference<List<Person>>() {}
        );
        
        // ========== Map 序列化和反序列化 ==========
        Map<String, Person> personMap = new HashMap<>();
        personMap.put("user1", new Person("张三", 25, "zhangsan@example.com"));
        personMap.put("user2", new Person("李四", 30, "lisi@example.com"));
        
        // Map 转 JSON
        String mapJson = JSON.toJSONString(personMap);
        
        // JSON 转 Map
        Map<String, Person> deserializedMap = JSON.parseObject(
            mapJson,
            new TypeReference<Map<String, Person>>() {}
        );
    }
}
```

### 2.3 Feature 配置 (⚠️ 难点)

Feature 用于控制序列化和反序列化的行为。

#### 2.3.1 SerializerFeature（序列化特性）

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.serializer.SerializerFeature;

/**
 * 序列化特性示例
 * @author erik.zhou
 */
public class SerializerFeatureExample {
    
    public static void main(String[] args) {
        Person person = new Person("张三", 25, null);
        
        // 1. 美化输出（格式化 JSON）
        String prettyJson = JSON.toJSONString(person, SerializerFeature.PrettyFormat);
        
        // 2. 输出 null 值字段
        String withNull = JSON.toJSONString(person, SerializerFeature.WriteMapNullValue);
        
        // 3. 日期格式化
        String dateFormat = JSON.toJSONStringWithDateFormat(
            person,
            "yyyy-MM-dd HH:mm:ss"
        );
        
        // 4. 禁用循环引用检测
        String noCircular = JSON.toJSONString(
            person,
            SerializerFeature.DisableCircularReferenceDetect
        );
        
        // 5. 组合多个特性
        String combined = JSON.toJSONString(
            person,
            SerializerFeature.PrettyFormat,
            SerializerFeature.WriteMapNullValue,
            SerializerFeature.DisableCircularReferenceDetect
        );
    }
}
```

#### 2.3.2 Feature（反序列化特性）

```java
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.alibaba.fastjson.parser.Feature;

/**
 * 反序列化特性示例
 * @author erik.zhou
 */
public class ParserFeatureExample {
    
    public static void main(String[] args) {
        // 1. 允许不带引号的字段名
        String relaxedJson = "{name: 'John', age: 30}";
        JSONObject obj = JSON.parseObject(
            relaxedJson,
            Feature.AllowUnQuotedFieldNames,
            Feature.AllowSingleQuotes
        );
        
        // 2. 允许注释
        String jsonWithComments = "{\n" +
            "  // 这是注释\n" +
            "  \"name\": \"Alice\"\n" +
            "}";
        JSONObject withComments = JSON.parseObject(
            jsonWithComments,
            Feature.AllowComment
        );
        
        // 3. 使用 BigDecimal（避免精度丢失）
        String numJson = "{\"price\": 99.99}";
        JSONObject precise = JSON.parseObject(numJson, Feature.UseBigDecimal);
        
        // 4. 忽略未知字段
        String extraFields = "{\"name\":\"Bob\",\"age\":25,\"unknownField\":\"value\"}";
        Person person = JSON.parseObject(
            extraFields,
            Person.class,
            Feature.IgnoreNotMatch
        );
        
        // 5. 字符串字段初始化为空字符串（而非 null）
        Person emptyStrings = JSON.parseObject(
            "{\"name\":\"Alice\"}",
            Person.class,
            Feature.InitStringFieldAsEmpty
        );
    }
}
```

### 2.4 JSONPath 查询 🔥

JSONPath 是一种强大的 JSON 查询语言，类似于 XPath。

```java
import com.alibaba.fastjson.JSONPath;

/**
 * JSONPath 查询示例
 * @author erik.zhou
 */
public class JSONPathExample {
    
    public static void main(String[] args) {
        String json = "{\n" +
            "  \"store\": {\n" +
            "    \"book\": [\n" +
            "      {\"category\": \"reference\", \"author\": \"Nigel Rees\", \"title\": \"Sayings of the Century\", \"price\": 8.95},\n" +
            "      {\"category\": \"fiction\", \"author\": \"Evelyn Waugh\", \"title\": \"Sword of Honour\", \"price\": 12.99}\n" +
            "    ],\n" +
            "    \"bicycle\": {\"color\": \"red\", \"price\": 19.95}\n" +
            "  }\n" +
            "}";
        
        // 1. 读取值
        Object result = JSONPath.eval(json, "$.store.book[0].title");
        // 结果: "Sayings of the Century"
        
        // 2. 读取数组
        Object books = JSONPath.eval(json, "$.store.book");
        
        // 3. 过滤查询（价格小于 10 的书）
        Object cheapBooks = JSONPath.eval(json, "$.store.book[price < 10]");
        
        // 4. 计算大小
        int size = JSONPath.size(json, "$.store.book");
        // 结果: 2
        
        // 5. 判断是否包含
        boolean contains = JSONPath.contains(json, "$.store.bicycle");
        // 结果: true
        
        // 6. 修改值
        JSONPath.set(json, "$.store.bicycle.price", 25.00);
        
        // 7. 获取所有作者
        Object authors = JSONPath.eval(json, "$.store.book[*].author");
    }
}
```

### 2.5 流式处理大文件 (⚠️ 难点)

对于大文件，使用流式 API 可以避免内存溢出。

```java
import com.alibaba.fastjson.JSONReader;
import com.alibaba.fastjson.JSONWriter;
import java.io.*;

/**
 * 流式处理大文件示例
 * @author erik.zhou
 */
public class StreamingExample {
    
    /**
     * 使用 JSONWriter 写入大文件
     */
    public void writeLargeFile() {
        try (JSONWriter writer = new JSONWriter(new FileWriter("output.json"))) {
            writer.startArray();
            
            // 写入 100 万条数据
            for (int i = 0; i < 1000000; i++) {
                Person person = new Person("User" + i, 20 + (i % 50), "user" + i + "@example.com");
                writer.writeValue(person);
            }
            
            writer.endArray();
            writer.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 使用 JSONReader 读取大文件
     */
    public void readLargeFile() {
        try (JSONReader reader = new JSONReader(new FileReader("input.json"))) {
            reader.startArray();
            
            while (reader.hasNext()) {
                Person person = reader.readObject(Person.class);
                // 处理每个对象，不会将整个文件加载到内存
                processPerson(person);
            }
            
            reader.endArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    /**
     * 流式处理复杂 JSON
     */
    public void processComplexJson() {
        try (JSONReader reader = new JSONReader(new FileReader("data.json"))) {
            reader.startObject();
            
            while (reader.hasNext()) {
                String key = reader.readString();
                
                if ("users".equals(key)) {
                    reader.startArray();
                    while (reader.hasNext()) {
                        Person person = reader.readObject(Person.class);
                        processPerson(person);
                    }
                    reader.endArray();
                } else {
                    reader.readObject();  // 跳过未知字段
                }
            }
            
            reader.endObject();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    
    private void processPerson(Person person) {
        // 处理逻辑
        System.out.println("Processing: " + person.getName());
    }
}
```

## 💻 实战应用

### 3.1 Spring Boot 集成

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.http.converter.HttpMessageConverter;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;
import com.alibaba.fastjson.support.spring.FastJsonHttpMessageConverter;
import com.alibaba.fastjson.support.config.FastJsonConfig;
import com.alibaba.fastjson.serializer.SerializerFeature;
import java.util.List;

/**
 * FastJSON 配置类
 * @author erik.zhou
 */
@Configuration
public class FastJsonConfiguration implements WebMvcConfigurer {
    
    @Override
    public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
        // 创建 FastJSON 消息转换器
        FastJsonHttpMessageConverter converter = new FastJsonHttpMessageConverter();
        
        // 创建配置对象
        FastJsonConfig config = new FastJsonConfig();
        
        // 设置序列化特性
        config.setSerializerFeatures(
            SerializerFeature.PrettyFormat,           // 格式化输出
            SerializerFeature.WriteMapNullValue,      // 输出 null 值
            SerializerFeature.DisableCircularReferenceDetect  // 禁用循环引用
        );
        
        // 设置日期格式
        config.setDateFormat("yyyy-MM-dd HH:mm:ss");
        
        // 应用配置
        converter.setFastJsonConfig(config);
        
        // 添加到转换器列表
        converters.add(0, converter);
    }
}
```

### 3.2 Controller 中使用

```java
import org.springframework.web.bind.annotation.*;

/**
 * RESTful API 示例
 * @author erik.zhou
 */
@RestController
@RequestMapping("/api/users")
public class UserController {
    
    /**
     * 自动序列化返回值
     */
    @GetMapping("/{id}")
    public Person getUser(@PathVariable Long id) {
        // Spring 自动使用 FastJSON 序列化返回值
        return userService.findById(id);
    }
    
    /**
     * 自动反序列化请求体
     */
    @PostMapping
    public Person createUser(@RequestBody Person person) {
        // Spring 自动使用 FastJSON 反序列化请求体
        return userService.save(person);
    }
    
    /**
     * 手动使用 FastJSON
     */
    @PostMapping("/manual")
    public String manualProcess(@RequestBody String jsonString) {
        // 手动反序列化
        Person person = JSON.parseObject(jsonString, Person.class);
        
        // 业务处理
        person = userService.save(person);
        
        // 手动序列化
        return JSON.toJSONString(person);
    }
}
```

### 3.3 注解使用

```java
import com.alibaba.fastjson.annotation.*;
import java.util.Date;

/**
 * FastJSON 注解示例
 * @author erik.zhou
 */
public class AnnotationExample {
    
    /**
     * @JSONField 注解
     */
    public static class User {
        
        // 指定序列化名称
        @JSONField(name = "userName")
        private String name;
        
        // 指定序列化顺序
        @JSONField(ordinal = 1)
        private int age;
        
        // 日期格式化
        @JSONField(format = "yyyy-MM-dd HH:mm:ss")
        private Date createTime;
        
        // 不序列化该字段
        @JSONField(serialize = false)
        private String password;
        
        // 不反序列化该字段
        @JSONField(deserialize = false)
        private String token;
        
        // Getter 和 Setter
    }
    
    /**
     * @JSONType 注解（类级别）
     */
    @JSONType(
        orders = {"name", "age", "email"},  // 字段顺序
        ignores = {"password"}               // 忽略字段
    )
    public static class Person {
        private String name;
        private int age;
        private String email;
        private String password;
        
        // Getter 和 Setter
    }
}
```

### 3.4 自定义序列化器

```java
import com.alibaba.fastjson.serializer.JSONSerializer;
import com.alibaba.fastjson.serializer.ObjectSerializer;
import com.alibaba.fastjson.serializer.SerializeWriter;
import java.io.IOException;
import java.lang.reflect.Type;

/**
 * 自定义序列化器示例
 * @author erik.zhou
 */
public class CustomSerializerExample {
    
    /**
     * 自定义金额序列化器
     */
    public static class MoneySerializer implements ObjectSerializer {
        
        @Override
        public void write(JSONSerializer serializer, Object object, Object fieldName,
                          Type fieldType, int features) throws IOException {
            SerializeWriter out = serializer.out;
            
            if (object == null) {
                out.writeNull();
                return;
            }
            
            BigDecimal value = (BigDecimal) object;
            // 保留两位小数
            String formatted = value.setScale(2, BigDecimal.ROUND_HALF_UP).toString();
            out.writeString(formatted);
        }
    }
    
    /**
     * 使用自定义序列化器
     */
    public static class Product {
        private String name;
        
        @JSONField(serializeUsing = MoneySerializer.class)
        private BigDecimal price;
        
        // Getter 和 Setter
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
     * 1. 禁用循环引用检测（提升性能）
     */
    public String optimized1(Object obj) {
        return JSON.toJSONString(obj, SerializerFeature.DisableCircularReferenceDetect);
    }
    
    /**
     * 2. 使用 SerializeWriter 复用（减少对象创建）
     */
    public String optimized2(Object obj) {
        SerializeWriter out = new SerializeWriter();
        try {
            JSON.writeJSONString(out, obj);
            return out.toString();
        } finally {
            out.close();
        }
    }
    
    /**
     * 3. 使用流式 API 处理大数据
     */
    public void optimized3() {
        // 见 2.5 流式处理大文件
    }
    
    /**
     * 4. 指定字段序列化（减少数据量）
     */
    public String optimized4(Object obj) {
        return JSON.toJSONString(obj, new String[]{"name", "age"});
    }
}
```

### 4.2 安全性考虑 (⚠️ 重要)

```java
import com.alibaba.fastjson.parser.ParserConfig;

/**
 * 安全性最佳实践
 * @author erik.zhou
 */
public class SecurityBestPractices {
    
    /**
     * 1. 禁用 AutoType（防止反序列化漏洞）
     */
    public static void disableAutoType() {
        // 全局禁用 AutoType
        ParserConfig.getGlobalInstance().setAutoTypeSupport(false);
    }
    
    /**
     * 2. 使用白名单（如果必须使用 AutoType）
     */
    public static void useWhitelist() {
        ParserConfig.getGlobalInstance().addAccept("com.example.model.");
    }
    
    /**
     * 3. 升级到 FastJSON 2（更安全）
     */
    // 推荐使用 FastJSON 2.x 版本
    
    /**
     * 4. 输入验证
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
        
        return JSON.parseObject(json, Person.class);
    }
}
```

### 4.3 常见陷阱

#### ⚠️ 陷阱 1: AutoType 安全漏洞

```java
// ❌ 危险：启用 AutoType
ParserConfig.getGlobalInstance().setAutoTypeSupport(true);

// ✅ 安全：禁用 AutoType
ParserConfig.getGlobalInstance().setAutoTypeSupport(false);

// ✅ 或者使用白名单
ParserConfig.getGlobalInstance().addAccept("com.example.model.");
```

#### ⚠️ 陷阱 2: 循环引用

```java
/**
 * 处理循环引用
 * @author erik.zhou
 */
public class CircularReferenceExample {
    
    // ❌ 错误：循环引用会导致 $ref
    public static class Parent {
        private String name;
        private Child child;
    }
    
    public static class Child {
        private String name;
        private Parent parent;  // 循环引用
    }
    
    // 序列化结果: {"name":"父节点","child":{"name":"子节点","parent":{"$ref":".."}}}
    
    // ✅ 正确：禁用循环引用检测
    public String serialize(Parent parent) {
        return JSON.toJSONString(parent, SerializerFeature.DisableCircularReferenceDetect);
    }
}
```

#### ⚠️ 陷阱 3: 日期格式不一致

```java
/**
 * 日期处理最佳实践
 * @author erik.zhou
 */
public class DateHandling {
    
    /**
     * 统一日期格式
     */
    public String serializeWithDateFormat(Object obj) {
        return JSON.toJSONStringWithDateFormat(obj, "yyyy-MM-dd HH:mm:ss");
    }
    
    /**
     * 使用注解指定格式
     */
    public static class Event {
        @JSONField(format = "yyyy-MM-dd HH:mm:ss")
        private Date eventTime;
    }
}
```

### 4.4 异常处理

```java
import com.alibaba.fastjson.JSONException;

/**
 * 异常处理示例
 * @author erik.zhou
 */
public class ExceptionHandling {
    
    public Person parseUser(String json) {
        try {
            return JSON.parseObject(json, Person.class);
        } catch (JSONException e) {
            // JSON 解析异常
            log.error("JSON 解析失败: {}", json, e);
            throw new BusinessException("数据格式错误");
        } catch (Exception e) {
            // 其他异常
            log.error("处理失败", e);
            throw new BusinessException("系统错误");
        }
    }
}
```

## ❓ 常见问题

### Q1: FastJSON、Jackson、Gson 如何选择？

**A**:
- **Jackson**: 功能最强大，Spring Boot 默认，推荐使用
- **FastJSON**: 性能最快，但安全漏洞多，不推荐生产环境使用
- **Gson**: Google 出品，API 简单，适合简单场景

### Q2: FastJSON 1.x 和 2.x 有什么区别？

**A**:
- **FastJSON 2.x**: 重写版本，性能更好，安全性更高，推荐使用
- **FastJSON 1.x**: 老版本，存在多个安全漏洞，不推荐使用

### Q3: 如何处理 null 值？

**A**:
```java
// 输出 null 值
String json = JSON.toJSONString(obj, SerializerFeature.WriteMapNullValue);

// 忽略 null 值（默认行为）
String json = JSON.toJSONString(obj);
```

### Q4: 如何美化 JSON 输出？

**A**:
```java
String prettyJson = JSON.toJSONString(obj, SerializerFeature.PrettyFormat);
```

### Q5: 如何处理枚举类型？

**A**:
```java
public enum Status {
    ACTIVE(1, "激活"),
    INACTIVE(0, "未激活");
    
    private final int code;
    private final String desc;
    
    Status(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }
    
    @JSONField
    public int getCode() {
        return code;
    }
}

// 序列化时使用 code
String json = JSON.toJSONString(Status.ACTIVE);  // 输出: 1
```

### Q6: 如何处理大数字精度问题？

**A**:
```java
// 使用 BigDecimal
String json = JSON.parseObject(jsonString, Feature.UseBigDecimal);
```

### Q7: FastJSON 有哪些安全漏洞？

**A**:
- **AutoType 反序列化漏洞**: 可导致远程代码执行
- **解决方案**: 
  1. 升级到最新版本
  2. 禁用 AutoType
  3. 使用白名单
  4. 考虑迁移到 Jackson

## 🔗 相关资源

### 官方资源
- **GitHub (FastJSON 2)**: https://github.com/alibaba/fastjson2
- **GitHub (FastJSON 1)**: https://github.com/alibaba/fastjson
- **Wiki**: https://github.com/alibaba/fastjson/wiki

### 推荐文章
- 《FastJSON 官方文档》
- 《FastJSON 安全漏洞分析》
- 《FastJSON 性能优化实践》

### 相关技术
- **Jackson**: 推荐的 JSON 处理库
- **Gson**: Google 的 JSON 处理库
- **JSON-B**: Java EE 标准 JSON 绑定 API

## 📝 学习检查清单

- [ ] 理解 FastJSON 的核心类和 API
- [ ] 掌握基本的序列化和反序列化操作
- [ ] 掌握 Feature 配置的使用
- [ ] 熟练使用 JSONPath 查询
- [ ] 能够使用流式 API 处理大文件
- [ ] 了解常用注解的使用
- [ ] 掌握 Spring Boot 集成方式
- [ ] 理解性能优化技巧
- [ ] 了解安全风险和防范措施
- [ ] 能够处理常见的异常情况
- [ ] 理解 FastJSON 与其他库的区别

---

**@author erik.zhou**  
**文档来源**: Context7 - alibaba/fastjson  
**最后更新**: 2024-01-04
