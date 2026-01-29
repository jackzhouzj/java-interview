# 单元测试 JUnit 5 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [JUnit 5基础](#junit-5基础)
- [断言和假设](#断言和假设)
- [参数化测试](#参数化测试)
- [Mock测试](#mock测试)
- [Spring Boot测试](#spring-boot测试)
- [实战案例](#实战案例)
- [最佳实践](#最佳实践)

## 📚 技术概述
- **版本**: JUnit 5.9+
- **学习难度**: ⭐⭐⭐ (3星)
- **重要程度**: ⭐⭐⭐⭐⭐ (5星)
- **前置知识**: Java基础、Spring Boot
- **更新时间**: 2024-01-04
- **作者**: erik.zhou

## 🎯 学习目标
- [ ] 掌握JUnit 5基本用法
- [ ] 理解测试生命周期
- [ ] 掌握各种断言方法
- [ ] 能够编写参数化测试
- [ ] 掌握Mock测试技术
- [ ] 能够进行Spring Boot集成测试

## 📖 JUnit 5基础

### 1.1 基本测试示例

```java
/**
 * 计算器测试类
 * @author erik.zhou
 */
public class CalculatorTest {
    
    private Calculator calculator;
    
    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }
    
    @Test
    @DisplayName("测试加法")
    void testAdd() {
        int result = calculator.add(2, 3);
        assertEquals(5, result, "2 + 3 应该等于 5");
    }
    
    @Test
    @DisplayName("测试减法")
    void testSubtract() {
        int result = calculator.subtract(5, 3);
        assertEquals(2, result);
    }
    
    @Test
    @DisplayName("测试除法")
    void testDivide() {
        double result = calculator.divide(10, 2);
        assertEquals(5.0, result, 0.001);
    }
    
    @Test
    @DisplayName("测试除以零抛出异常")
    void testDivideByZero() {
        assertThrows(ArithmeticException.class, () -> {
            calculator.divide(10, 0);
        });
    }
    
    @AfterEach
    void tearDown() {
        calculator = null;
    }
}
```

### 1.2 测试生命周期

```java
/**
 * 测试生命周期示例
 * @author erik.zhou
 */
public class LifecycleTest {
    
    @BeforeAll
    static void initAll() {
        System.out.println("在所有测试之前执行一次");
    }
    
    @BeforeEach
    void init() {
        System.out.println("在每个测试之前执行");
    }
    
    @Test
    void test1() {
        System.out.println("执行测试1");
    }
    
    @Test
    void test2() {
        System.out.println("执行测试2");
    }
    
    @AfterEach
    void tearDown() {
        System.out.println("在每个测试之后执行");
    }
    
    @AfterAll
    static void tearDownAll() {
        System.out.println("在所有测试之后执行一次");
    }
}
```

## 🔥 断言和假设

### 2.1 常用断言

```java
/**
 * 断言示例
 * @author erik.zhou
 */
public class AssertionsTest {
    
    @Test
    @DisplayName("基本断言")
    void testBasicAssertions() {
        // 相等断言
        assertEquals(2, 1 + 1);
        assertNotEquals(3, 1 + 1);
        
        // 布尔断言
        assertTrue(true);
        assertFalse(false);
        
        // null断言
        assertNull(null);
        assertNotNull("not null");
        
        // 数组断言
        int[] expected = {1, 2, 3};
        int[] actual = {1, 2, 3};
        assertArrayEquals(expected, actual);
    }
    
    @Test
    @DisplayName("异常断言")
    void testExceptionAssertions() {
        Exception exception = assertThrows(
            IllegalArgumentException.class,
            () -> {
                throw new IllegalArgumentException("参数错误");
            }
        );
        
        assertEquals("参数错误", exception.getMessage());
    }
    
    @Test
    @DisplayName("超时断言")
    void testTimeoutAssertions() {
        assertTimeout(Duration.ofSeconds(2), () -> {
            Thread.sleep(1000);
        });
    }
    
    @Test
    @DisplayName("组合断言")
    void testGroupedAssertions() {
        User user = new User("张三", 25);
        
        assertAll("用户信息",
            () -> assertEquals("张三", user.getName()),
            () -> assertEquals(25, user.getAge()),
            () -> assertNotNull(user.getId())
        );
    }
}
```

### 2.2 假设(Assumptions)

```java
/**
 * 假设示例
 * @author erik.zhou
 */
public class AssumptionsTest {
    
    @Test
    @DisplayName("假设条件满足才执行")
    void testOnlyOnDev() {
        assumeTrue("DEV".equals(System.getenv("ENV")));
        // 只有在DEV环境才执行
        System.out.println("在DEV环境执行");
    }
    
    @Test
    @DisplayName("假设条件不满足才执行")
    void testNotOnProd() {
        assumeFalse("PROD".equals(System.getenv("ENV")));
        // 只有不在PROD环境才执行
        System.out.println("不在PROD环境执行");
    }
    
    @Test
    @DisplayName("条件假设")
    void testAssumingThat() {
        assumingThat("CI".equals(System.getenv("ENV")),
            () -> {
                // 只有在CI环境才执行这部分
                System.out.println("在CI环境执行");
            }
        );
        
        // 这部分总是执行
        System.out.println("总是执行");
    }
}
```

## 🔥 参数化测试

### 3.1 基本参数化测试

```java
/**
 * 参数化测试示例
 * @author erik.zhou
 */
public class ParameterizedTests {
    
    @ParameterizedTest
    @ValueSource(ints = {1, 2, 3, 4, 5})
    @DisplayName("测试正数")
    void testPositiveNumbers(int number) {
        assertTrue(number > 0);
    }
    
    @ParameterizedTest
    @ValueSource(strings = {"", "  ", "\t", "\n"})
    @DisplayName("测试空白字符串")
    void testBlankStrings(String input) {
        assertTrue(input.isBlank());
    }
    
    @ParameterizedTest
    @NullSource
    @EmptySource
    @ValueSource(strings = {" ", "\t", "\n"})
    @DisplayName("测试null和空白")
    void testNullEmptyAndBlank(String input) {
        assertTrue(input == null || input.isBlank());
    }
}
```

### 3.2 CSV参数化测试

```java
/**
 * CSV参数化测试
 * @author erik.zhou
 */
public class CsvParameterizedTests {
    
    @ParameterizedTest
    @CsvSource({
        "1, 1, 2",
        "2, 3, 5",
        "10, 20, 30"
    })
    @DisplayName("测试加法 - CSV")
    void testAddWithCsv(int a, int b, int expected) {
        Calculator calculator = new Calculator();
        assertEquals(expected, calculator.add(a, b));
    }
    
    @ParameterizedTest
    @CsvFileSource(resources = "/test-data.csv", numLinesToSkip = 1)
    @DisplayName("测试加法 - CSV文件")
    void testAddWithCsvFile(int a, int b, int expected) {
        Calculator calculator = new Calculator();
        assertEquals(expected, calculator.add(a, b));
    }
}
```

### 3.3 方法参数化测试

```java
/**
 * 方法参数化测试
 * @author erik.zhou
 */
public class MethodSourceTests {
    
    @ParameterizedTest
    @MethodSource("provideStringsForIsBlank")
    @DisplayName("测试字符串是否为空")
    void testIsBlank(String input, boolean expected) {
        assertEquals(expected, input == null || input.isBlank());
    }
    
    static Stream<Arguments> provideStringsForIsBlank() {
        return Stream.of(
            Arguments.of(null, true),
            Arguments.of("", true),
            Arguments.of("  ", true),
            Arguments.of("not blank", false)
        );
    }
    
    @ParameterizedTest
    @MethodSource("provideUsers")
    @DisplayName("测试用户验证")
    void testUserValidation(User user, boolean expected) {
        assertEquals(expected, user.isValid());
    }
    
    static Stream<User> provideUsers() {
        return Stream.of(
            new User("张三", 25),
            new User("李四", 30),
            new User("", 20)
        );
    }
}
```

---

**作者**: erik.zhou  
**最后更新**: 2024-01-04

## 🔥 Mock测试

### 4.1 Mockito基础

```java
/**
 * Mockito基础示例
 * @author erik.zhou
 */
@ExtendWith(MockitoExtension.class)
public class MockitoBasicTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    @DisplayName("测试查询用户")
    void testGetUser() {
        // 准备测试数据
        User mockUser = new User(1L, "张三", 25);
        
        // 设置Mock行为
        when(userRepository.findById(1L)).thenReturn(Optional.of(mockUser));
        
        // 执行测试
        User user = userService.getUser(1L);
        
        // 验证结果
        assertNotNull(user);
        assertEquals("张三", user.getName());
        assertEquals(25, user.getAge());
        
        // 验证方法调用
        verify(userRepository, times(1)).findById(1L);
    }
    
    @Test
    @DisplayName("测试保存用户")
    void testSaveUser() {
        User user = new User(null, "李四", 30);
        User savedUser = new User(2L, "李四", 30);
        
        when(userRepository.save(any(User.class))).thenReturn(savedUser);
        
        User result = userService.saveUser(user);
        
        assertNotNull(result.getId());
        assertEquals(2L, result.getId());
        
        verify(userRepository).save(argThat(u -> 
            "李四".equals(u.getName()) && u.getAge() == 30
        ));
    }
    
    @Test
    @DisplayName("测试删除用户")
    void testDeleteUser() {
        doNothing().when(userRepository).deleteById(1L);
        
        userService.deleteUser(1L);
        
        verify(userRepository, times(1)).deleteById(1L);
    }
    
    @Test
    @DisplayName("测试异常情况")
    void testException() {
        when(userRepository.findById(999L))
            .thenThrow(new RuntimeException("用户不存在"));
        
        assertThrows(RuntimeException.class, () -> {
            userService.getUser(999L);
        });
    }
}
```

### 4.2 Spy和Stub

```java
/**
 * Spy和Stub示例
 * @author erik.zhou
 */
@ExtendWith(MockitoExtension.class)
public class SpyAndStubTest {
    
    @Test
    @DisplayName("使用Spy部分Mock")
    void testSpy() {
        List<String> list = new ArrayList<>();
        List<String> spyList = spy(list);
        
        // 真实方法调用
        spyList.add("one");
        spyList.add("two");
        
        // Mock特定方法
        when(spyList.size()).thenReturn(100);
        
        assertEquals(100, spyList.size());
        assertEquals("one", spyList.get(0));
    }
    
    @Test
    @DisplayName("使用Stub")
    void testStub() {
        UserService userService = mock(UserService.class);
        
        // Stub方法
        when(userService.getUser(anyLong())).thenAnswer(invocation -> {
            Long id = invocation.getArgument(0);
            return new User(id, "User" + id, 20);
        });
        
        User user = userService.getUser(1L);
        assertEquals("User1", user.getName());
    }
}
```

### 4.3 参数捕获

```java
/**
 * 参数捕获示例
 * @author erik.zhou
 */
@ExtendWith(MockitoExtension.class)
public class ArgumentCaptorTest {
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    @DisplayName("捕获方法参数")
    void testArgumentCaptor() {
        User user = new User(1L, "张三", 25);
        user.setEmail("zhangsan@example.com");
        
        userService.registerUser(user);
        
        // 捕获参数
        ArgumentCaptor<Email> emailCaptor = ArgumentCaptor.forClass(Email.class);
        verify(emailService).sendEmail(emailCaptor.capture());
        
        // 验证捕获的参数
        Email capturedEmail = emailCaptor.getValue();
        assertEquals("zhangsan@example.com", capturedEmail.getTo());
        assertEquals("欢迎注册", capturedEmail.getSubject());
    }
}
```

## 🔥 Spring Boot测试

### 5.1 Controller测试

```java
/**
 * Controller测试示例
 * @author erik.zhou
 */
@WebMvcTest(UserController.class)
public class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    @DisplayName("测试获取用户接口")
    void testGetUser() throws Exception {
        User mockUser = new User(1L, "张三", 25);
        when(userService.getUser(1L)).thenReturn(mockUser);
        
        mockMvc.perform(get("/api/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("张三"))
            .andExpect(jsonPath("$.age").value(25))
            .andDo(print());
        
        verify(userService, times(1)).getUser(1L);
    }
    
    @Test
    @DisplayName("测试创建用户接口")
    void testCreateUser() throws Exception {
        UserDTO userDTO = new UserDTO("李四", 30);
        User savedUser = new User(2L, "李四", 30);
        
        when(userService.createUser(any(UserDTO.class))).thenReturn(savedUser);
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\":\"李四\",\"age\":30}"))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(2))
            .andExpect(jsonPath("$.name").value("李四"))
            .andDo(print());
    }
    
    @Test
    @DisplayName("测试参数校验")
    void testValidation() throws Exception {
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"name\":\"\",\"age\":-1}"))
            .andExpect(status().isBadRequest())
            .andDo(print());
    }
}
```

### 5.2 Service测试

```java
/**
 * Service测试示例
 * @author erik.zhou
 */
@SpringBootTest
@Transactional
public class UserServiceTest {
    
    @Autowired
    private UserService userService;
    
    @Autowired
    private UserRepository userRepository;
    
    @Test
    @DisplayName("测试创建用户")
    void testCreateUser() {
        UserDTO userDTO = new UserDTO("王五", 28);
        
        User user = userService.createUser(userDTO);
        
        assertNotNull(user.getId());
        assertEquals("王五", user.getName());
        assertEquals(28, user.getAge());
        
        // 验证数据库
        Optional<User> saved = userRepository.findById(user.getId());
        assertTrue(saved.isPresent());
        assertEquals("王五", saved.get().getName());
    }
    
    @Test
    @DisplayName("测试更新用户")
    void testUpdateUser() {
        // 先创建用户
        User user = new User(null, "赵六", 35);
        user = userRepository.save(user);
        
        // 更新用户
        UserDTO updateDTO = new UserDTO("赵六(已更新)", 36);
        User updated = userService.updateUser(user.getId(), updateDTO);
        
        assertEquals("赵六(已更新)", updated.getName());
        assertEquals(36, updated.getAge());
    }
    
    @Test
    @DisplayName("测试删除用户")
    void testDeleteUser() {
        User user = new User(null, "孙七", 40);
        user = userRepository.save(user);
        
        userService.deleteUser(user.getId());
        
        Optional<User> deleted = userRepository.findById(user.getId());
        assertFalse(deleted.isPresent());
    }
}
```

### 5.3 Repository测试

```java
/**
 * Repository测试示例
 * @author erik.zhou
 */
@DataJpaTest
public class UserRepositoryTest {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    @DisplayName("测试根据名称查询")
    void testFindByName() {
        User user = new User(null, "周八", 45);
        entityManager.persist(user);
        entityManager.flush();
        
        List<User> found = userRepository.findByName("周八");
        
        assertFalse(found.isEmpty());
        assertEquals("周八", found.get(0).getName());
    }
    
    @Test
    @DisplayName("测试自定义查询")
    void testCustomQuery() {
        User user1 = new User(null, "吴九", 20);
        User user2 = new User(null, "郑十", 30);
        entityManager.persist(user1);
        entityManager.persist(user2);
        entityManager.flush();
        
        List<User> users = userRepository.findByAgeGreaterThan(25);
        
        assertEquals(1, users.size());
        assertEquals("郑十", users.get(0).getName());
    }
}
```

## ✨ 最佳实践

### 6.1 测试命名规范

```java
/**
 * 测试命名最佳实践
 * @author erik.zhou
 */
public class NamingBestPracticeTest {
    
    // 方式1: should_ExpectedBehavior_When_StateUnderTest
    @Test
    void should_ReturnUser_When_UserExists() {
        // ...
    }
    
    // 方式2: given_Preconditions_When_StateUnderTest_Then_ExpectedBehavior
    @Test
    void given_UserExists_When_GetUser_Then_ReturnUser() {
        // ...
    }
    
    // 方式3: 使用DisplayName
    @Test
    @DisplayName("当用户存在时，应该返回用户信息")
    void testGetExistingUser() {
        // ...
    }
}
```

### 6.2 测试数据准备

```java
/**
 * 测试数据准备最佳实践
 * @author erik.zhou
 */
public class TestDataBestPracticeTest {
    
    // 使用Builder模式
    @Test
    void testWithBuilder() {
        User user = User.builder()
            .name("测试用户")
            .age(25)
            .email("test@example.com")
            .build();
        
        // 测试逻辑
    }
    
    // 使用测试数据工厂
    @Test
    void testWithFactory() {
        User user = TestDataFactory.createUser();
        // 测试逻辑
    }
    
    // 使用@BeforeEach准备通用数据
    private User testUser;
    
    @BeforeEach
    void prepareTestData() {
        testUser = new User(1L, "测试用户", 25);
    }
}

/**
 * 测试数据工厂
 * @author erik.zhou
 */
class TestDataFactory {
    
    public static User createUser() {
        return new User(1L, "测试用户", 25);
    }
    
    public static User createUser(String name, int age) {
        return new User(null, name, age);
    }
}
```

---

**作者**: erik.zhou
