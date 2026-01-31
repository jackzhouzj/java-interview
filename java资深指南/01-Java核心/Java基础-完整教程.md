# Java基础 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [JDK版本特性对比](#jdk版本特性对比)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)

## 📚 技术概述
- **版本**: JDK 21 (LTS)
- **官方文档**: https://docs.oracle.com/en/java/javase/21/
- **学习难度**: ⭐⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐⭐ (1-5星)
- **前置知识**: 计算机基础、编程基础概念
- **文档来源**: OpenJDK官方文档 + Context7查询
- **更新时间**: 2024-12-31

## 🎯 学习目标
- [ ] 掌握Java基础语法和面向对象编程
- [ ] 理解Java泛型、反射、注解机制
- [ ] 熟练使用JDK 8+新特性（Lambda、Stream、Optional）
- [ ] 掌握Java集合框架和并发编程基础
- [ ] 理解Java异常处理机制
- [ ] 掌握Java I/O和NIO

## 📖 基础概念

### 1.1 什么是Java

Java是一种面向对象的编程语言，由Sun Microsystems（现Oracle）于1995年发布。Java具有"一次编写，到处运行"（Write Once, Run Anywhere）的特性。

**核心特点**：
- **面向对象**: 一切皆对象，支持封装、继承、多态
- **平台无关**: 通过JVM实现跨平台
- **自动内存管理**: 垃圾回收机制自动管理内存
- **安全性**: 内置安全机制，防止恶意代码
- **多线程**: 内置多线程支持
- **丰富的类库**: 提供大量标准类库

### 1.2 Java程序执行流程

```
源代码(.java) → 编译器(javac) → 字节码(.class) → JVM → 机器码 → 执行
```


### 1.3 Java开发环境搭建

**安装JDK**：
1. 下载JDK：访问Oracle官网或使用OpenJDK
2. 配置环境变量：
   - `JAVA_HOME`: JDK安装路径
   - `PATH`: 添加 `%JAVA_HOME%/bin`
3. 验证安装：`java -version`

**第一个Java程序**：

```java
/**
 * Hello World示例
 * @author erik.zhou
 */
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

编译运行：
```bash
javac HelloWorld.java
java HelloWorld
```

## 🔥 核心特性 (重点)

### 2.1 数据类型

**基本数据类型（8种）**：

| 类型 | 字节 | 范围 | 默认值 |
|------|------|------|--------|
| byte | 1 | -128 ~ 127 | 0 |
| short | 2 | -32768 ~ 32767 | 0 |
| int | 4 | -2^31 ~ 2^31-1 | 0 |
| long | 8 | -2^63 ~ 2^63-1 | 0L |
| float | 4 | IEEE 754 | 0.0f |
| double | 8 | IEEE 754 | 0.0d |
| char | 2 | 0 ~ 65535 | '\u0000' |
| boolean | 1 | true/false | false |

**引用数据类型**：
- 类（Class）
- 接口（Interface）
- 数组（Array）


### 2.2 面向对象编程 🔥

**三大特性**：

#### 封装（Encapsulation）
```java
/**
 * 用户实体类 - 演示封装
 * @author erik.zhou
 */
public class User {
    // 私有属性
    private String username;
    private String password;
    
    // 公共getter/setter方法
    public String getUsername() {
        return username;
    }
    
    public void setUsername(String username) {
        this.username = username;
    }
    
    public String getPassword() {
        return password;
    }
    
    public void setPassword(String password) {
        // 可以在setter中添加验证逻辑
        if (password != null && password.length() >= 6) {
            this.password = password;
        } else {
            throw new IllegalArgumentException("密码长度至少6位");
        }
    }
}
```

#### 继承（Inheritance）
```java
/**
 * 动物基类
 * @author erik.zhou
 */
public class Animal {
    protected String name;
    
    public void eat() {
        System.out.println(name + " is eating");
    }
}

/**
 * 狗类 - 继承动物类
 * @author erik.zhou
 */
public class Dog extends Animal {
    @Override
    public void eat() {
        System.out.println(name + " is eating bones");
    }
    
    public void bark() {
        System.out.println(name + " is barking");
    }
}
```


#### 多态（Polymorphism）
```java
/**
 * 多态示例
 * @author erik.zhou
 */
public class PolymorphismDemo {
    public static void main(String[] args) {
        // 父类引用指向子类对象
        Animal animal1 = new Dog();
        Animal animal2 = new Cat();
        
        // 调用同一方法，表现不同行为
        animal1.eat(); // 输出: Dog is eating bones
        animal2.eat(); // 输出: Cat is eating fish
    }
}
```

### 2.3 泛型（Generics）🔥 ⚠️ 难点

泛型是JDK 5引入的特性，提供编译时类型安全检查。

**泛型类**：
```java
/**
 * 泛型类示例
 * @author erik.zhou
 */
public class Box<T> {
    private T content;
    
    public void set(T content) {
        this.content = content;
    }
    
    public T get() {
        return content;
    }
}

// 使用泛型类
Box<String> stringBox = new Box<>();
stringBox.set("Hello");
String value = stringBox.get(); // 无需强制类型转换
```

**泛型方法**：
```java
/**
 * 泛型方法示例
 * @author erik.zhou
 */
public class GenericMethod {
    public static <T> void printArray(T[] array) {
        for (T element : array) {
            System.out.print(element + " ");
        }
        System.out.println();
    }
    
    public static void main(String[] args) {
        Integer[] intArray = {1, 2, 3, 4, 5};
        String[] strArray = {"A", "B", "C"};
        
        printArray(intArray);
        printArray(strArray);
    }
}
```


**泛型通配符** ⚠️ 难点：
```java
/**
 * 泛型通配符示例
 * @author erik.zhou
 */
public class WildcardDemo {
    // 上界通配符 <? extends T>
    public static double sumOfList(List<? extends Number> list) {
        double sum = 0.0;
        for (Number num : list) {
            sum += num.doubleValue();
        }
        return sum;
    }
    
    // 下界通配符 <? super T>
    public static void addNumbers(List<? super Integer> list) {
        for (int i = 1; i <= 10; i++) {
            list.add(i);
        }
    }
    
    // 无界通配符 <?>
    public static void printList(List<?> list) {
        for (Object obj : list) {
            System.out.print(obj + " ");
        }
        System.out.println();
    }
}
```

**泛型擦除** ⚠️ 难点：
- Java泛型是编译时特性，运行时会被擦除
- 泛型信息在运行时不可用
- 不能创建泛型数组：`new T[]` 是非法的

### 2.4 反射（Reflection）🔥 ⚠️ 难点

反射允许程序在运行时检查和操作类、方法、字段等。

**获取Class对象**：
```java
/**
 * 反射基础示例
 * @author erik.zhou
 */
public class ReflectionDemo {
    public static void main(String[] args) throws Exception {
        // 方式1: 通过类名
        Class<?> clazz1 = User.class;
        
        // 方式2: 通过对象
        User user = new User();
        Class<?> clazz2 = user.getClass();
        
        // 方式3: 通过全限定名
        Class<?> clazz3 = Class.forName("com.example.User");
    }
}
```


**反射操作示例**：
```java
/**
 * 反射操作类、方法、字段
 * @author erik.zhou
 */
public class ReflectionOperations {
    public static void main(String[] args) throws Exception {
        Class<?> clazz = User.class;
        
        // 创建实例
        User user = (User) clazz.getDeclaredConstructor().newInstance();
        
        // 获取并操作字段
        Field usernameField = clazz.getDeclaredField("username");
        usernameField.setAccessible(true); // 访问私有字段
        usernameField.set(user, "admin");
        
        // 获取并调用方法
        Method setPasswordMethod = clazz.getMethod("setPassword", String.class);
        setPasswordMethod.invoke(user, "password123");
        
        // 获取所有方法
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            System.out.println("方法名: " + method.getName());
        }
    }
}
```

**反射的应用场景**：
- 框架开发（Spring IoC容器）
- 动态代理
- 注解处理
- ORM框架（MyBatis、Hibernate）

### 2.5 注解（Annotation）🔥 ⚠️ 难点

注解是JDK 5引入的元数据机制，用于为代码添加说明信息。

**内置注解**：
```java
/**
 * 内置注解示例
 * @author erik.zhou
 */
public class AnnotationDemo {
    @Override // 标记重写方法
    public String toString() {
        return "AnnotationDemo";
    }
    
    @Deprecated // 标记过时方法
    public void oldMethod() {
        System.out.println("This method is deprecated");
    }
    
    @SuppressWarnings("unchecked") // 抑制警告
    public void uncheckedMethod() {
        List list = new ArrayList();
        list.add("item");
    }
}
```


**自定义注解**：
```java
import java.lang.annotation.*;

/**
 * 自定义注解示例
 * @author erik.zhou
 */
@Target(ElementType.METHOD) // 注解作用目标
@Retention(RetentionPolicy.RUNTIME) // 注解保留策略
@Documented // 包含在JavaDoc中
public @interface MyAnnotation {
    String value() default ""; // 注解属性
    int priority() default 0;
}

/**
 * 使用自定义注解
 * @author erik.zhou
 */
public class MyService {
    @MyAnnotation(value = "重要方法", priority = 1)
    public void importantMethod() {
        System.out.println("执行重要方法");
    }
}

/**
 * 注解处理器
 * @author erik.zhou
 */
public class AnnotationProcessor {
    public static void process(Class<?> clazz) {
        Method[] methods = clazz.getDeclaredMethods();
        for (Method method : methods) {
            if (method.isAnnotationPresent(MyAnnotation.class)) {
                MyAnnotation annotation = method.getAnnotation(MyAnnotation.class);
                System.out.println("方法: " + method.getName());
                System.out.println("描述: " + annotation.value());
                System.out.println("优先级: " + annotation.priority());
            }
        }
    }
}
```

**注解的元注解**：
- `@Target`: 指定注解可以用在哪些地方（类、方法、字段等）
- `@Retention`: 指定注解的保留策略（SOURCE、CLASS、RUNTIME）
- `@Documented`: 注解是否包含在JavaDoc中
- `@Inherited`: 注解是否可以被继承


## 🚀 JDK版本特性对比

### JDK 8 (2014年) 🔥 重点

**Lambda表达式**：
```java
/**
 * Lambda表达式示例
 * @author erik.zhou
 */
public class LambdaDemo {
    public static void main(String[] args) {
        // 传统方式
        List<String> list = Arrays.asList("a", "b", "c");
        list.forEach(new Consumer<String>() {
            @Override
            public void accept(String s) {
                System.out.println(s);
            }
        });
        
        // Lambda方式
        list.forEach(s -> System.out.println(s));
        
        // 方法引用
        list.forEach(System.out::println);
    }
}
```

**Stream API** 🔥：
```java
/**
 * Stream API示例
 * @author erik.zhou
 */
public class StreamDemo {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // 过滤、映射、求和
        int sum = numbers.stream()
            .filter(n -> n % 2 == 0)  // 过滤偶数
            .map(n -> n * n)           // 平方
            .reduce(0, Integer::sum);  // 求和
        
        System.out.println("偶数平方和: " + sum);
        
        // 分组
        Map<Boolean, List<Integer>> partitioned = numbers.stream()
            .collect(Collectors.partitioningBy(n -> n % 2 == 0));
        
        System.out.println("偶数: " + partitioned.get(true));
        System.out.println("奇数: " + partitioned.get(false));
    }
}
```

**Optional类** 🔥：
```java
/**
 * Optional示例 - 避免空指针异常
 * @author erik.zhou
 */
public class OptionalDemo {
    public static void main(String[] args) {
        // 创建Optional
        Optional<String> optional = Optional.of("Hello");
        Optional<String> empty = Optional.empty();
        
        // 判断是否有值
        if (optional.isPresent()) {
            System.out.println(optional.get());
        }
        
        // 使用orElse提供默认值
        String value = empty.orElse("Default Value");
        
        // 使用orElseGet（延迟计算）
        String value2 = empty.orElseGet(() -> "Computed Default");
        
        // 使用ifPresent
        optional.ifPresent(System.out::println);
        
        // 链式调用
        String result = Optional.ofNullable(getUserName())
            .map(String::toUpperCase)
            .filter(name -> name.length() > 3)
            .orElse("UNKNOWN");
    }
    
    private static String getUserName() {
        return "admin";
    }
}
```


**函数式接口** 🔥：
```java
/**
 * 函数式接口示例
 * @author erik.zhou
 */
@FunctionalInterface
public interface Calculator {
    int calculate(int a, int b);
}

public class FunctionalInterfaceDemo {
    public static void main(String[] args) {
        // 使用Lambda实现函数式接口
        Calculator add = (a, b) -> a + b;
        Calculator multiply = (a, b) -> a * b;
        
        System.out.println("加法: " + add.calculate(5, 3));
        System.out.println("乘法: " + multiply.calculate(5, 3));
        
        // 常用函数式接口
        Predicate<Integer> isEven = n -> n % 2 == 0;
        Function<String, Integer> strLength = String::length;
        Consumer<String> printer = System.out::println;
        Supplier<Double> randomSupplier = Math::random;
    }
}
```

**接口默认方法和静态方法**：
```java
/**
 * 接口默认方法示例
 * @author erik.zhou
 */
public interface Vehicle {
    // 抽象方法
    void start();
    
    // 默认方法
    default void stop() {
        System.out.println("Vehicle stopped");
    }
    
    // 静态方法
    static void checkMaintenance() {
        System.out.println("Checking maintenance");
    }
}
```

**新的日期时间API**：
```java
import java.time.*;

/**
 * 新日期时间API示例
 * @author erik.zhou
 */
public class DateTimeDemo {
    public static void main(String[] args) {
        // LocalDate - 日期
        LocalDate today = LocalDate.now();
        LocalDate birthday = LocalDate.of(1990, Month.JANUARY, 1);
        
        // LocalTime - 时间
        LocalTime now = LocalTime.now();
        LocalTime specificTime = LocalTime.of(14, 30, 0);
        
        // LocalDateTime - 日期时间
        LocalDateTime dateTime = LocalDateTime.now();
        
        // ZonedDateTime - 带时区的日期时间
        ZonedDateTime zonedDateTime = ZonedDateTime.now(ZoneId.of("Asia/Shanghai"));
        
        // 日期计算
        LocalDate nextWeek = today.plusWeeks(1);
        LocalDate lastMonth = today.minusMonths(1);
        
        // 格式化
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        String formatted = dateTime.format(formatter);
    }
}
```


### JDK 9 (2017年)

**模块系统（Jigsaw）**：
```java
// module-info.java
module com.example.myapp {
    requires java.sql;
    exports com.example.myapp.api;
}
```

**集合工厂方法**：
```java
/**
 * 集合工厂方法示例
 * @author erik.zhou
 */
public class CollectionFactoryDemo {
    public static void main(String[] args) {
        // 不可变List
        List<String> list = List.of("A", "B", "C");
        
        // 不可变Set
        Set<Integer> set = Set.of(1, 2, 3);
        
        // 不可变Map
        Map<String, Integer> map = Map.of(
            "one", 1,
            "two", 2,
            "three", 3
        );
    }
}
```

**Stream API增强**：
```java
/**
 * Stream API增强示例
 * @author erik.zhou
 */
public class StreamEnhancementDemo {
    public static void main(String[] args) {
        // takeWhile - 从头开始取元素直到条件不满足
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6);
        List<Integer> result1 = numbers.stream()
            .takeWhile(n -> n < 4)
            .collect(Collectors.toList()); // [1, 2, 3]
        
        // dropWhile - 从头开始丢弃元素直到条件不满足
        List<Integer> result2 = numbers.stream()
            .dropWhile(n -> n < 4)
            .collect(Collectors.toList()); // [4, 5, 6]
        
        // ofNullable - 创建可能为null的Stream
        Stream<String> stream = Stream.ofNullable(getNullableValue());
    }
    
    private static String getNullableValue() {
        return null;
    }
}
```

### JDK 10 (2018年)

**局部变量类型推断（var）**：
```java
/**
 * var关键字示例
 * @author erik.zhou
 */
public class VarDemo {
    public static void main(String[] args) {
        // 使用var声明局部变量
        var message = "Hello"; // 推断为String
        var number = 100;      // 推断为int
        var list = new ArrayList<String>(); // 推断为ArrayList<String>
        
        // 在循环中使用
        var numbers = List.of(1, 2, 3, 4, 5);
        for (var num : numbers) {
            System.out.println(num);
        }
        
        // 注意：var只能用于局部变量，不能用于字段、方法参数、返回类型
    }
}
```


### JDK 11 (2018年 - LTS)

**字符串增强**：
```java
/**
 * 字符串增强示例
 * @author erik.zhou
 */
public class StringEnhancementDemo {
    public static void main(String[] args) {
        String str = "  Hello World  ";
        
        // isBlank() - 判断是否为空白字符串
        System.out.println("   ".isBlank()); // true
        
        // strip() - 去除首尾空白（支持Unicode）
        System.out.println(str.strip());
        
        // lines() - 按行分割字符串
        String multiline = "Line1\nLine2\nLine3";
        multiline.lines().forEach(System.out::println);
        
        // repeat() - 重复字符串
        System.out.println("Java".repeat(3)); // JavaJavaJava
    }
}
```

**HTTP Client API**：
```java
import java.net.http.*;
import java.net.URI;

/**
 * HTTP Client示例
 * @author erik.zhou
 */
public class HttpClientDemo {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newHttpClient();
        
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create("https://api.example.com/data"))
            .GET()
            .build();
        
        HttpResponse<String> response = client.send(
            request,
            HttpResponse.BodyHandlers.ofString()
        );
        
        System.out.println("状态码: " + response.statusCode());
        System.out.println("响应体: " + response.body());
    }
}
```

### JDK 14 (2020年)

**Switch表达式（正式版）**：
```java
/**
 * Switch表达式示例
 * @author erik.zhou
 */
public class SwitchExpressionDemo {
    public static void main(String[] args) {
        // 传统switch语句
        int day = 3;
        String dayName;
        switch (day) {
            case 1:
                dayName = "Monday";
                break;
            case 2:
                dayName = "Tuesday";
                break;
            default:
                dayName = "Unknown";
                break;
        }
        
        // 新的switch表达式
        String dayName2 = switch (day) {
            case 1 -> "Monday";
            case 2 -> "Tuesday";
            case 3 -> "Wednesday";
            default -> "Unknown";
        };
        
        // 支持多个case
        String type = switch (day) {
            case 1, 2, 3, 4, 5 -> "Weekday";
            case 6, 7 -> "Weekend";
            default -> "Invalid";
        };
    }
}
```


**Record类（预览）**：
```java
/**
 * Record类示例 - 不可变数据类
 * @author erik.zhou
 */
public record Point(int x, int y) {
    // 自动生成：
    // - 构造器
    // - getter方法（x(), y()）
    // - equals()、hashCode()、toString()
}

public class RecordDemo {
    public static void main(String[] args) {
        Point point = new Point(10, 20);
        System.out.println(point.x()); // 10
        System.out.println(point.y()); // 20
        System.out.println(point);     // Point[x=10, y=20]
    }
}
```

### JDK 15 (2020年)

**文本块（正式版）**：
```java
/**
 * 文本块示例
 * @author erik.zhou
 */
public class TextBlockDemo {
    public static void main(String[] args) {
        // 传统方式
        String json1 = "{\n" +
                      "  \"name\": \"John\",\n" +
                      "  \"age\": 30\n" +
                      "}";
        
        // 文本块方式
        String json2 = """
            {
              "name": "John",
              "age": 30
            }
            """;
        
        // SQL示例
        String sql = """
            SELECT id, name, email
            FROM users
            WHERE status = 'ACTIVE'
            ORDER BY created_at DESC
            """;
    }
}
```

### JDK 17 (2021年 - LTS) 🔥

**密封类（Sealed Classes）**：
```java
/**
 * 密封类示例 - 限制继承
 * @author erik.zhou
 */
public sealed class Shape
    permits Circle, Rectangle, Triangle {
    // 只有Circle、Rectangle、Triangle可以继承Shape
}

public final class Circle extends Shape {
    private final double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
}

public final class Rectangle extends Shape {
    private final double width;
    private final double height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
}

public non-sealed class Triangle extends Shape {
    // non-sealed允许进一步继承
}
```


**模式匹配（Pattern Matching）**：
```java
/**
 * 模式匹配示例
 * @author erik.zhou
 */
public class PatternMatchingDemo {
    public static void main(String[] args) {
        Object obj = "Hello";
        
        // 传统方式
        if (obj instanceof String) {
            String str = (String) obj;
            System.out.println(str.length());
        }
        
        // 模式匹配方式
        if (obj instanceof String str) {
            System.out.println(str.length());
        }
        
        // 在switch中使用模式匹配
        String result = switch (obj) {
            case String s -> "字符串: " + s;
            case Integer i -> "整数: " + i;
            case null -> "null值";
            default -> "其他类型";
        };
    }
}
```

### JDK 21 (2023年 - LTS) 🔥

**虚拟线程（Virtual Threads）**：
```java
/**
 * 虚拟线程示例
 * @author erik.zhou
 */
public class VirtualThreadDemo {
    public static void main(String[] args) throws Exception {
        // 创建虚拟线程
        Thread vThread = Thread.ofVirtual().start(() -> {
            System.out.println("虚拟线程执行");
        });
        vThread.join();
        
        // 使用ExecutorService创建虚拟线程池
        try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < 10000; i++) {
                executor.submit(() -> {
                    // 处理任务
                    return "Result";
                });
            }
        }
    }
}
```

**Record模式匹配**：
```java
/**
 * Record模式匹配示例
 * @author erik.zhou
 */
public record Point(int x, int y) {}

public class RecordPatternDemo {
    public static void printPoint(Object obj) {
        if (obj instanceof Point(int x, int y)) {
            System.out.println("x = " + x + ", y = " + y);
        }
    }
    
    public static void main(String[] args) {
        Point point = new Point(10, 20);
        printPoint(point); // x = 10, y = 20
    }
}
```


## 💻 实战应用

### 3.1 集合框架 🔥

**List接口**：
```java
import java.util.*;

/**
 * List集合示例
 * @author erik.zhou
 */
public class ListDemo {
    public static void main(String[] args) {
        // ArrayList - 基于数组，查询快，增删慢
        List<String> arrayList = new ArrayList<>(10); // 指定初始容量
        arrayList.add("A");
        arrayList.add("B");
        arrayList.add("C");
        
        // LinkedList - 基于链表，增删快，查询慢
        List<String> linkedList = new LinkedList<>();
        linkedList.add("X");
        linkedList.add("Y");
        linkedList.add("Z");
        
        // 遍历方式
        // 1. for循环
        for (int i = 0; i < arrayList.size(); i++) {
            System.out.println(arrayList.get(i));
        }
        
        // 2. 增强for循环
        for (String item : arrayList) {
            System.out.println(item);
        }
        
        // 3. Iterator
        Iterator<String> iterator = arrayList.iterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
        
        // 4. forEach + Lambda
        arrayList.forEach(System.out::println);
    }
}
```

**Set接口**：
```java
import java.util.*;

/**
 * Set集合示例
 * @author erik.zhou
 */
public class SetDemo {
    public static void main(String[] args) {
        // HashSet - 无序，基于HashMap
        Set<String> hashSet = new HashSet<>();
        hashSet.add("Apple");
        hashSet.add("Banana");
        hashSet.add("Apple"); // 重复元素不会被添加
        
        // LinkedHashSet - 保持插入顺序
        Set<String> linkedHashSet = new LinkedHashSet<>();
        linkedHashSet.add("One");
        linkedHashSet.add("Two");
        linkedHashSet.add("Three");
        
        // TreeSet - 自动排序
        Set<Integer> treeSet = new TreeSet<>();
        treeSet.add(5);
        treeSet.add(2);
        treeSet.add(8);
        treeSet.add(1);
        System.out.println(treeSet); // [1, 2, 5, 8]
    }
}
```


**Map接口**：
```java
import java.util.*;

/**
 * Map集合示例
 * @author erik.zhou
 */
public class MapDemo {
    public static void main(String[] args) {
        // HashMap - 无序，允许null键和null值
        Map<String, Integer> hashMap = new HashMap<>(16); // 指定初始容量
        hashMap.put("Alice", 25);
        hashMap.put("Bob", 30);
        hashMap.put("Charlie", 35);
        
        // LinkedHashMap - 保持插入顺序
        Map<String, Integer> linkedHashMap = new LinkedHashMap<>();
        linkedHashMap.put("One", 1);
        linkedHashMap.put("Two", 2);
        linkedHashMap.put("Three", 3);
        
        // TreeMap - 按键排序
        Map<String, Integer> treeMap = new TreeMap<>();
        treeMap.put("C", 3);
        treeMap.put("A", 1);
        treeMap.put("B", 2);
        System.out.println(treeMap); // {A=1, B=2, C=3}
        
        // 遍历Map
        // 1. 遍历键值对
        for (Map.Entry<String, Integer> entry : hashMap.entrySet()) {
            System.out.println(entry.getKey() + " = " + entry.getValue());
        }
        
        // 2. 遍历键
        for (String key : hashMap.keySet()) {
            System.out.println(key + " = " + hashMap.get(key));
        }
        
        // 3. 遍历值
        for (Integer value : hashMap.values()) {
            System.out.println(value);
        }
        
        // 4. forEach + Lambda
        hashMap.forEach((key, value) -> 
            System.out.println(key + " = " + value)
        );
        
        // JDK 8新增方法
        hashMap.putIfAbsent("David", 40);
        hashMap.computeIfAbsent("Eve", k -> 45);
        hashMap.merge("Alice", 5, Integer::sum); // Alice的值变为30
    }
}
```

### 3.2 异常处理 🔥

**异常体系**：
```
Throwable
├── Error (系统错误，不应捕获)
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception (程序异常)
    ├── RuntimeException (运行时异常，非受检异常)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   └── IllegalArgumentException
    └── IOException (受检异常，必须处理)
        ├── FileNotFoundException
        └── SQLException
```


**异常处理示例**：
```java
import java.io.*;

/**
 * 异常处理示例
 * @author erik.zhou
 */
public class ExceptionDemo {
    public static void main(String[] args) {
        // try-catch-finally
        try {
            int result = divide(10, 0);
            System.out.println(result);
        } catch (ArithmeticException e) {
            System.err.println("除数不能为0: " + e.getMessage());
        } finally {
            System.out.println("finally块总是执行");
        }
        
        // try-with-resources (JDK 7+)
        try (BufferedReader reader = new BufferedReader(
                new FileReader("file.txt"))) {
            String line = reader.readLine();
            System.out.println(line);
        } catch (IOException e) {
            System.err.println("文件读取失败: " + e.getMessage());
        }
        
        // 多重catch
        try {
            processData();
        } catch (FileNotFoundException e) {
            System.err.println("文件未找到");
        } catch (IOException e) {
            System.err.println("IO异常");
        } catch (Exception e) {
            System.err.println("其他异常");
        }
    }
    
    private static int divide(int a, int b) {
        if (b == 0) {
            throw new IllegalArgumentException("除数不能为0");
        }
        return a / b;
    }
    
    private static void processData() throws IOException {
        // 方法声明抛出异常
        throw new IOException("数据处理失败");
    }
}
```

**自定义异常**：
```java
/**
 * 自定义业务异常
 * @author erik.zhou
 */
public class BusinessException extends RuntimeException {
    private final int errorCode;
    
    public BusinessException(int errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
    
    public int getErrorCode() {
        return errorCode;
    }
}

/**
 * 使用自定义异常
 * @author erik.zhou
 */
public class UserService {
    public void createUser(String username) {
        if (username == null || username.isEmpty()) {
            throw new BusinessException(400, "用户名不能为空");
        }
        // 创建用户逻辑
    }
}
```


### 3.3 I/O操作

**文件操作**：
```java
import java.io.*;
import java.nio.file.*;

/**
 * 文件操作示例
 * @author erik.zhou
 */
public class FileDemo {
    public static void main(String[] args) throws IOException {
        // 传统I/O方式
        File file = new File("test.txt");
        
        // 写入文件
        try (FileWriter writer = new FileWriter(file)) {
            writer.write("Hello, World!");
        }
        
        // 读取文件
        try (BufferedReader reader = new BufferedReader(
                new FileReader(file))) {
            String line;
            while ((line = reader.readLine()) != null) {
                System.out.println(line);
            }
        }
        
        // NIO.2方式 (JDK 7+)
        Path path = Paths.get("test.txt");
        
        // 写入文件
        Files.write(path, "Hello, NIO!".getBytes());
        
        // 读取文件
        List<String> lines = Files.readAllLines(path);
        lines.forEach(System.out::println);
        
        // 复制文件
        Path source = Paths.get("source.txt");
        Path target = Paths.get("target.txt");
        Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
        
        // 删除文件
        Files.deleteIfExists(path);
    }
}
```

## ✨ 最佳实践

### 4.1 编码规范

**命名规范**：
```java
/**
 * 命名规范示例
 * @author erik.zhou
 */
public class NamingConvention {
    // 常量：全大写 + 下划线
    public static final int MAX_RETRY_COUNT = 3;
    public static final String DEFAULT_USERNAME = "admin";
    
    // 类名：PascalCase
    public class UserService {}
    
    // 方法名/变量名：camelCase
    private String userName;
    
    public String getUserName() {
        return userName;
    }
    
    public void setUserName(String userName) {
        this.userName = userName;
    }
}
```


**字符串操作** ⚠️ 注意：
```java
/**
 * 字符串最佳实践
 * @author erik.zhou
 */
public class StringBestPractice {
    public static void main(String[] args) {
        // ❌ 错误：使用 == 比较字符串
        String str1 = "hello";
        String str2 = "hello";
        if (str1 == str2) { // 可能为true（字符串池），但不可靠
            System.out.println("相等");
        }
        
        // ✅ 正确：使用equals()比较字符串
        if ("hello".equals(str1)) { // 常量在前，避免空指针
            System.out.println("相等");
        }
        
        // ❌ 错误：循环中拼接字符串
        String result = "";
        for (int i = 0; i < 1000; i++) {
            result += i; // 每次都创建新对象，性能差
        }
        
        // ✅ 正确：使用StringBuilder
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < 1000; i++) {
            sb.append(i);
        }
        String result2 = sb.toString();
    }
}
```

**集合操作** ⚠️ 注意：
```java
import java.util.*;

/**
 * 集合最佳实践
 * @author erik.zhou
 */
public class CollectionBestPractice {
    public static void main(String[] args) {
        // ✅ 正确：初始化时指定容量
        List<String> list = new ArrayList<>(100);
        Map<String, Integer> map = new HashMap<>(16);
        
        // ❌ 错误：遍历时删除元素
        List<Integer> numbers = new ArrayList<>(Arrays.asList(1, 2, 3, 4, 5));
        for (Integer num : numbers) {
            if (num % 2 == 0) {
                // numbers.remove(num); // 抛出ConcurrentModificationException
            }
        }
        
        // ✅ 正确：使用Iterator删除
        Iterator<Integer> iterator = numbers.iterator();
        while (iterator.hasNext()) {
            Integer num = iterator.next();
            if (num % 2 == 0) {
                iterator.remove();
            }
        }
        
        // ✅ 更好：使用removeIf()
        numbers.removeIf(num -> num % 2 == 0);
        
        // ✅ 正确：返回空集合而不是null
        List<String> emptyList = Collections.emptyList();
        Map<String, Integer> emptyMap = Collections.emptyMap();
    }
}
```


### 4.2 性能优化

**避免不必要的对象创建**：
```java
/**
 * 对象创建优化
 * @author erik.zhou
 */
public class ObjectCreationOptimization {
    // ❌ 错误：每次调用都创建新对象
    public static String getConstant() {
        return new String("CONSTANT"); // 不必要的对象创建
    }
    
    // ✅ 正确：使用字符串字面量
    public static String getConstant2() {
        return "CONSTANT"; // 使用字符串池
    }
    
    // ❌ 错误：自动装箱在循环中
    public static long sum() {
        Long sum = 0L;
        for (long i = 0; i < 1000000; i++) {
            sum += i; // 每次都装箱/拆箱
        }
        return sum;
    }
    
    // ✅ 正确：使用基本类型
    public static long sum2() {
        long sum = 0L;
        for (long i = 0; i < 1000000; i++) {
            sum += i;
        }
        return sum;
    }
}
```

**使用合适的数据结构**：
```java
import java.util.*;

/**
 * 数据结构选择
 * @author erik.zhou
 */
public class DataStructureChoice {
    // 场景1：频繁随机访问 → ArrayList
    public void randomAccess() {
        List<String> list = new ArrayList<>();
        String item = list.get(100); // O(1)
    }
    
    // 场景2：频繁插入删除 → LinkedList
    public void frequentInsertDelete() {
        List<String> list = new LinkedList<>();
        list.add(0, "item"); // O(1)
    }
    
    // 场景3：需要去重 → HashSet
    public void uniqueElements() {
        Set<String> set = new HashSet<>();
        set.add("item");
    }
    
    // 场景4：需要排序 → TreeSet
    public void sortedElements() {
        Set<Integer> set = new TreeSet<>();
        set.add(5);
        set.add(2);
        set.add(8); // 自动排序
    }
}
```

## ❓ 常见问题

### Q1: == 和 equals() 的区别？
**A**: 
- `==` 比较的是引用地址（对于基本类型比较值）
- `equals()` 比较的是对象内容（需要正确重写）

```java
String s1 = new String("hello");
String s2 = new String("hello");
System.out.println(s1 == s2);        // false（不同对象）
System.out.println(s1.equals(s2));   // true（内容相同）
```


### Q2: String、StringBuilder、StringBuffer的区别？
**A**:
- `String`: 不可变，线程安全，适合少量字符串操作
- `StringBuilder`: 可变，非线程安全，适合单线程大量字符串操作
- `StringBuffer`: 可变，线程安全（synchronized），适合多线程字符串操作

### Q3: ArrayList和LinkedList的区别？
**A**:
- `ArrayList`: 基于数组，随机访问快O(1)，插入删除慢O(n)
- `LinkedList`: 基于链表，插入删除快O(1)，随机访问慢O(n)

### Q4: HashMap的工作原理？
**A**:
- 基于哈希表实现
- 通过key的hashCode()计算hash值，确定存储位置
- 发生hash冲突时，使用链表或红树（JDK 8+）存储
- JDK 8+：链表长度>8且数组长度>64时，转为红黑树

### Q5: 什么是泛型擦除？
**A**:
- Java泛型是编译时特性，运行时会被擦除
- 泛型信息在字节码中不存在
- 无法在运行时获取泛型类型参数
- 不能创建泛型数组

### Q6: 如何避免空指针异常？
**A**:
```java
// 1. 使用Optional
Optional<String> optional = Optional.ofNullable(getValue());
String result = optional.orElse("default");

// 2. 常量在前
if ("ACTIVE".equals(status)) { // 而不是 status.equals("ACTIVE")
    // ...
}

// 3. 使用Objects工具类
Objects.requireNonNull(obj, "对象不能为null");

// 4. 提前判断
if (obj != null) {
    obj.doSomething();
}
```

## 🔗 相关资源

### 官方文档
- [Java SE Documentation](https://docs.oracle.com/en/java/javase/)
- [OpenJDK](https://openjdk.org/)
- [Java Language Specification](https://docs.oracle.com/javase/specs/)

### 推荐书籍
- 《Java核心技术 卷I》- Cay S. Horstmann
- 《Effective Java》- Joshua Bloch
- 《Java编程思想》- Bruce Eckel

### 在线资源
- [Oracle Java Tutorials](https://docs.oracle.com/javase/tutorial/)
- [Baeldung](https://www.baeldung.com/)
- [Java Code Geeks](https://www.javacodegeeks.com/)


## 📝 学习检查清单

### 基础知识
- [ ] 理解Java程序执行流程
- [ ] 掌握8种基本数据类型
- [ ] 理解面向对象三大特性（封装、继承、多态）
- [ ] 掌握访问修饰符（public、private、protected、default）

### 核心特性
- [ ] 掌握泛型的使用和原理
- [ ] 理解反射机制及应用场景
- [ ] 掌握注解的定义和使用
- [ ] 理解接口和抽象类的区别

### JDK 8+特性
- [ ] 熟练使用Lambda表达式
- [ ] 掌握Stream API进行集合操作
- [ ] 理解Optional避免空指针
- [ ] 掌握函数式接口
- [ ] 理解接口默认方法和静态方法
- [ ] 掌握新的日期时间API

### 集合框架
- [ ] 理解List、Set、Map的区别和使用场景
- [ ] 掌握ArrayList、LinkedList的区别
- [ ] 理解HashMap的工作原理
- [ ] 掌握集合的遍历和操作方法

### 异常处理
- [ ] 理解异常体系结构
- [ ] 掌握try-catch-finally的使用
- [ ] 理解受检异常和非受检异常
- [ ] 掌握自定义异常

### I/O操作
- [ ] 掌握文件读写操作
- [ ] 理解字节流和字符流
- [ ] 掌握NIO.2的使用

### 最佳实践
- [ ] 遵循命名规范
- [ ] 正确使用equals()和hashCode()
- [ ] 避免常见的性能陷阱
- [ ] 掌握代码优化技巧

---

**@author erik.zhou**
**文档版本**: 1.0
**最后更新**: 2024-12-31
**文档来源**: OpenJDK官方文档 + Context7查询
