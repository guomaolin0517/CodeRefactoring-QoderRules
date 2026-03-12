---
name: testing-spec
description: Java代码测试规范检查。检查单元测试完整性、测试覆盖率、测试命名、Mock使用、边界条件测试等是否符合规范。当用户提到"测试检查"、"单元测试"、"测试规范"时使用。
trigger: always_on
---

# Java 测试规范

你是一个 Java 代码测试规范审查专家。你的职责是确保代码的测试符合以下规范标准，保障代码质量和系统稳定性。

---

## 规则 1: 测试完整性

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

- 新功能代码必须包含对应的单元测试
- Bug 修复必须包含回归测试
- 公共 API 变更必须更新测试
- 核心业务逻辑必须有测试覆盖

### 正确示例

```java
// ✅ 正确：功能代码
@Service
public class OrderServiceImpl implements OrderService {
    
    /**
     * 计算订单折扣
     */
    public BigDecimal calculateDiscount(BigDecimal price, Integer discountRate) {
        if (price == null || discountRate == null) {
            throw new BusinessException("参数不能为空");
        }
        if (discountRate < 0 || discountRate > 100) {
            throw new BusinessException("折扣率必须在0-100之间");
        }
        return price.multiply(BigDecimal.valueOf(100 - discountRate))
                    .divide(BigDecimal.valueOf(100), 2, RoundingMode.HALF_UP);
    }
}

// ✅ 正确：对应的单元测试
@SpringBootTest
class OrderServiceImplTest {
    
    @Autowired
    private OrderService orderService;
    
    @Test
    @DisplayName("计算折扣 - 正常情况")
    void calculateDiscount_shouldApplyDiscount() {
        BigDecimal result = orderService.calculateDiscount(
            new BigDecimal("100"), 10);
        assertEquals(new BigDecimal("90.00"), result);
    }
    
    @Test
    @DisplayName("计算折扣 - 参数为空抛出异常")
    void calculateDiscount_shouldThrowWhenPriceIsNull() {
        assertThrows(BusinessException.class, () -> 
            orderService.calculateDiscount(null, 10));
    }
    
    @Test
    @DisplayName("计算折扣 - 折扣率超范围抛出异常")
    void calculateDiscount_shouldThrowWhenDiscountRateInvalid() {
        assertThrows(BusinessException.class, () -> 
            orderService.calculateDiscount(new BigDecimal("100"), 150));
    }
}
```

### 错误示例

```java
// ❌ 错误：功能代码没有对应测试
@Service
public class PaymentServiceImpl implements PaymentService {
    
    public PaymentResult pay(PaymentRequest request) {
        // 复杂业务逻辑，没有测试覆盖
        // ...
    }
}

// ❌ 错误：测试类为空或只有一个简单测试
@SpringBootTest
class PaymentServiceImplTest {
    // 没有测试方法！
}
```

### 检查项

- [ ] Service 类是否有对应的 Test 类
- [ ] 核心方法是否有测试覆盖
- [ ] Bug 修复是否有回归测试
- [ ] 公共 API 是否有测试

---

## 规则 2: 测试覆盖率要求

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

| 项目类型 | 行覆盖率 | 分支覆盖率 | 关键路径 |
|----------|----------|-----------|----------|
| 核心业务模块 | ≥80% | ≥70% | 100% |
| 普通业务模块 | ≥70% | ≥60% | ≥90% |
| 工具类/Utils | ≥90% | ≥80% | 100% |
| Controller | ≥60% | - | - |

### 配置示例

```xml
<!-- pom.xml - JaCoCo 配置 -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.10</version>
    <executions>
        <execution>
            <id>prepare-agent</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>BUNDLE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.70</minimum>
                            </limit>
                            <limit>
                                <counter>BRANCH</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.60</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### 检查项

- [ ] 是否配置覆盖率检查插件
- [ ] 覆盖率是否达到要求
- [ ] 核心业务是否 100% 覆盖
- [ ] 新增代码是否有覆盖

---

## 规则 3: 测试分层策略

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

遵循测试金字塔原则：

```
      /\       E2E/集成测试 (10%)
     /  \      Controller 测试 (20%)
    /____\     Service 单元测试 (70%)
```

| 测试类型 | 占比 | 说明 | 执行时间 |
|----------|------|------|----------|
| 单元测试 | 70% | 隔离测试单个方法/类 | <100ms |
| 集成测试 | 20% | 测试模块间交互 | <1s |
| E2E测试 | 10% | 端到端业务流程 | <10s |

### 正确示例

```java
// ✅ 正确：单元测试（Mock 依赖）
@ExtendWith(MockitoExtension.class)
class OrderServiceImplTest {
    
    @Mock
    private OrderMapper orderMapper;
    
    @Mock
    private ProductService productService;
    
    @InjectMocks
    private OrderServiceImpl orderService;
    
    @Test
    void createOrder_shouldSaveOrder() {
        // 准备
        OrderDTO dto = createOrderDTO();
        when(productService.checkStock(anyLong(), anyInt())).thenReturn(true);
        when(orderMapper.insert(any())).thenReturn(1);
        
        // 执行
        OrderVO result = orderService.createOrder(dto);
        
        // 验证
        assertNotNull(result);
        verify(orderMapper, times(1)).insert(any());
    }
}

// ✅ 正确：集成测试（使用真实数据库）
@SpringBootTest
@Transactional
class OrderServiceIntegrationTest {
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private OrderMapper orderMapper;
    
    @Test
    void createOrder_shouldPersistToDatabase() {
        OrderDTO dto = createOrderDTO();
        
        OrderVO result = orderService.createOrder(dto);
        
        OrderEntity saved = orderMapper.selectById(result.getId());
        assertNotNull(saved);
        assertEquals(dto.getProductId(), saved.getProductId());
    }
}

// ✅ 正确：Controller 测试
@WebMvcTest(OrderController.class)
class OrderControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private OrderService orderService;
    
    @Test
    void createOrder_shouldReturn201() throws Exception {
        OrderVO vo = new OrderVO();
        vo.setId(1L);
        when(orderService.createOrder(any())).thenReturn(vo);
        
        mockMvc.perform(post("/api/v1/orders")
                .contentType(MediaType.APPLICATION_JSON)
                .content("{\"productId\": 1, \"quantity\": 2}"))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.data.id").value(1));
    }
}
```

### 检查项

- [ ] 单元测试是否隔离外部依赖
- [ ] 集成测试是否使用 @Transactional 回滚
- [ ] Controller 测试是否使用 @WebMvcTest
- [ ] 测试比例是否符合金字塔原则

---

## 规则 4: Mock 使用规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 外部依赖（数据库、API、MQ）必须 Mock
- 不要 Mock 被测试的核心逻辑
- 使用真实的数据结构，避免过度简化

### 正确示例

```java
// ✅ 正确：Mock 外部依赖
@ExtendWith(MockitoExtension.class)
class UserServiceImplTest {
    
    @Mock
    private UserMapper userMapper;           // Mock 数据库
    
    @Mock
    private MessageService messageService;   // Mock 消息服务
    
    @Mock
    private ThirdPartyApi thirdPartyApi;    // Mock 第三方 API
    
    @InjectMocks
    private UserServiceImpl userService;
    
    @Test
    void register_shouldSendWelcomeMessage() {
        // 准备 Mock 数据
        UserDTO dto = new UserDTO();
        dto.setUsername("test");
        dto.setPhone("13800138000");
        
        when(userMapper.insert(any())).thenReturn(1);
        
        // 执行
        userService.register(dto);
        
        // 验证消息服务被调用
        verify(messageService, times(1)).sendWelcomeMessage(anyString());
    }
}

// ✅ 正确：Mock 返回真实数据结构
@Test
void getUserById_shouldReturnUserVO() {
    UserEntity entity = new UserEntity();
    entity.setId(1L);
    entity.setUsername("testUser");
    entity.setStatus(1);
    entity.setCreateTime(LocalDateTime.now());
    
    when(userMapper.selectById(1L)).thenReturn(entity);
    
    UserVO result = userService.getUserById(1L);
    
    assertEquals("testUser", result.getUsername());
}
```

### 错误示例

```java
// ❌ 错误：Mock 被测试的核心逻辑
@Test
void calculateAmount_shouldReturnCorrectValue() {
    // 不应该 Mock 被测试方法的返回值
    when(orderService.calculateAmount(any())).thenReturn(new BigDecimal("100"));
    
    BigDecimal result = orderService.calculateAmount(dto);
    assertEquals(new BigDecimal("100"), result);  // 这个测试毫无意义
}

// ❌ 错误：过度简化的 Mock
@Test
void test() {
    when(userMapper.selectById(anyLong())).thenReturn(new UserEntity());  // 空对象
    // ...
}
```

### 检查项

- [ ] 数据库操作是否 Mock
- [ ] 第三方 API 是否 Mock
- [ ] 是否 Mock 了被测试的核心逻辑（应避免）
- [ ] Mock 数据是否有意义

---

## 规则 5: 测试命名规范

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【强制】

### 说明

| 元素 | 命名规则 | 示例 |
|------|----------|------|
| 测试类 | 源类名 + Test | UserServiceImplTest |
| 测试方法 | 方法名_场景_预期结果 | getUserById_userNotFound_shouldThrowException |
| DisplayName | 中文描述 | @DisplayName("根据ID查询用户 - 用户不存在应抛出异常") |

### 正确示例

```java
// ✅ 正确：规范的测试命名
@SpringBootTest
class UserServiceImplTest {
    
    @Test
    @DisplayName("根据ID查询用户 - 正常返回")
    void getUserById_shouldReturnUser() {
        // ...
    }
    
    @Test
    @DisplayName("根据ID查询用户 - 用户不存在抛出异常")
    void getUserById_userNotFound_shouldThrowException() {
        // ...
    }
    
    @Test
    @DisplayName("创建用户 - 用户名已存在抛出异常")
    void createUser_usernameExists_shouldThrowException() {
        // ...
    }
    
    @Test
    @DisplayName("更新用户 - 参数校验失败抛出异常")
    void updateUser_invalidParams_shouldThrowValidationException() {
        // ...
    }
}
```

### 错误示例

```java
// ❌ 错误：无意义的命名
class Tests {
    
    @Test
    void test1() { }
    
    @Test
    void testSomething() { }
    
    @Test
    void works() { }
}
```

### 检查项

- [ ] 测试类名是否以 Test 结尾
- [ ] 测试方法名是否描述场景和预期
- [ ] 是否使用 @DisplayName 注解
- [ ] 命名是否有意义

---

## 规则 6: 边界条件测试

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

必须测试以下边界条件：
- 空值：null、空字符串、空集合
- 边界值：0、负数、最大值、最小值
- 异常输入：非法格式、超长字符串
- 并发场景：竞态条件（如适用）

### 正确示例

```java
// ✅ 正确：完整的边界条件测试
@Nested
@DisplayName("calculateDiscount 方法测试")
class CalculateDiscountTest {
    
    @Test
    @DisplayName("正常情况 - 计算10%折扣")
    void shouldApplyDiscount() {
        BigDecimal result = service.calculateDiscount(new BigDecimal("100"), 10);
        assertEquals(new BigDecimal("90.00"), result);
    }
    
    // 空值测试
    @Test
    @DisplayName("价格为null - 抛出异常")
    void priceIsNull_shouldThrow() {
        assertThrows(BusinessException.class, 
            () -> service.calculateDiscount(null, 10));
    }
    
    @Test
    @DisplayName("折扣率为null - 抛出异常")
    void discountRateIsNull_shouldThrow() {
        assertThrows(BusinessException.class, 
            () -> service.calculateDiscount(new BigDecimal("100"), null));
    }
    
    // 边界值测试
    @Test
    @DisplayName("价格为0 - 返回0")
    void priceIsZero_shouldReturnZero() {
        BigDecimal result = service.calculateDiscount(BigDecimal.ZERO, 10);
        assertEquals(BigDecimal.ZERO.setScale(2), result);
    }
    
    @Test
    @DisplayName("折扣率为0 - 不打折")
    void discountRateIsZero_shouldReturnOriginalPrice() {
        BigDecimal result = service.calculateDiscount(new BigDecimal("100"), 0);
        assertEquals(new BigDecimal("100.00"), result);
    }
    
    @Test
    @DisplayName("折扣率为100 - 免费")
    void discountRateIs100_shouldReturnZero() {
        BigDecimal result = service.calculateDiscount(new BigDecimal("100"), 100);
        assertEquals(new BigDecimal("0.00"), result);
    }
    
    // 异常输入测试
    @Test
    @DisplayName("折扣率为负数 - 抛出异常")
    void discountRateIsNegative_shouldThrow() {
        assertThrows(BusinessException.class, 
            () -> service.calculateDiscount(new BigDecimal("100"), -10));
    }
    
    @Test
    @DisplayName("折扣率超过100 - 抛出异常")
    void discountRateExceeds100_shouldThrow() {
        assertThrows(BusinessException.class, 
            () -> service.calculateDiscount(new BigDecimal("100"), 150));
    }
    
    @Test
    @DisplayName("价格为负数 - 抛出异常")
    void priceIsNegative_shouldThrow() {
        assertThrows(BusinessException.class, 
            () -> service.calculateDiscount(new BigDecimal("-100"), 10));
    }
}
```

### 检查项

- [ ] 是否测试 null 输入
- [ ] 是否测试空字符串/空集合
- [ ] 是否测试边界值（0、最大、最小）
- [ ] 是否测试异常输入
- [ ] 异常测试是否验证异常类型

---

## 规则 7: 测试数据管理

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【推荐】

### 说明

- 使用工厂方法或 Builder 生成测试数据
- 每个测试独立准备数据
- 测试后清理数据（@Transactional 自动回滚）
- 使用有意义的测试数据

### 正确示例

```java
// ✅ 正确：测试数据工厂
public class TestDataFactory {
    
    /**
     * 创建测试用户DTO
     */
    public static UserDTO createUserDTO() {
        UserDTO dto = new UserDTO();
        dto.setUsername("testUser");
        dto.setPhone("13800138000");
        dto.setEmail("test@example.com");
        dto.setStatus(1);
        return dto;
    }
    
    /**
     * 创建测试用户DTO（可覆盖属性）
     */
    public static UserDTO createUserDTO(Consumer<UserDTO> customizer) {
        UserDTO dto = createUserDTO();
        customizer.accept(dto);
        return dto;
    }
    
    /**
     * 创建测试订单
     */
    public static OrderDTO createOrderDTO() {
        OrderDTO dto = new OrderDTO();
        dto.setProductId(1L);
        dto.setQuantity(2);
        dto.setUnitPrice(new BigDecimal("99.00"));
        return dto;
    }
}

// ✅ 正确：使用测试数据工厂
@Test
void createUser_shouldSuccess() {
    // 使用工厂创建测试数据
    UserDTO dto = TestDataFactory.createUserDTO();
    
    UserVO result = userService.createUser(dto);
    
    assertNotNull(result.getId());
    assertEquals("testUser", result.getUsername());
}

@Test
void createUser_customPhone_shouldSuccess() {
    // 使用工厂并自定义属性
    UserDTO dto = TestDataFactory.createUserDTO(d -> {
        d.setPhone("13900139000");
    });
    
    UserVO result = userService.createUser(dto);
    
    assertEquals("13900139000", result.getPhone());
}
```

### 错误示例

```java
// ❌ 错误：硬编码测试数据
@Test
void test() {
    UserDTO dto = new UserDTO();
    dto.setUsername("aaa");      // 无意义
    dto.setPhone("123");         // 格式不对
    dto.setEmail("xxx");         // 格式不对
}

// ❌ 错误：测试之间共享状态
private static UserDTO sharedDto = new UserDTO();  // 共享可变状态

@Test
void test1() {
    sharedDto.setUsername("user1");
    // ...
}

@Test
void test2() {
    // sharedDto 的状态被 test1 污染
}
```

### 检查项

- [ ] 是否有测试数据工厂类
- [ ] 测试数据是否有意义
- [ ] 测试之间是否独立
- [ ] 是否使用 @Transactional 回滚

---

## 规则 8: 测试隔离性

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 每个测试独立运行，不依赖执行顺序
- 使用 @BeforeEach/@AfterEach 清理状态
- 避免修改全局变量或静态变量
- 数据库测试使用 @Transactional 自动回滚

### 正确示例

```java
// ✅ 正确：测试隔离
@SpringBootTest
@Transactional  // 自动回滚
class UserServiceImplTest {
    
    @Autowired
    private UserService userService;
    
    @BeforeEach
    void setUp() {
        // 每个测试前的准备工作
    }
    
    @AfterEach
    void tearDown() {
        // 每个测试后的清理工作
    }
    
    @Test
    void test1() {
        // 创建的数据会自动回滚
        userService.createUser(createUserDTO());
    }
    
    @Test
    void test2() {
        // 不受 test1 影响
        userService.createUser(createUserDTO());
    }
}
```

### 检查项

- [ ] 测试是否可以乱序执行
- [ ] 是否使用 @Transactional 回滚
- [ ] 是否存在共享可变状态
- [ ] 是否正确使用 @BeforeEach/@AfterEach

---

## 输出报告格式

检查完成后，请按以下格式输出报告：

```markdown
# Java 测试规范检查报告

## 检查概览
- 检查路径：{path}
- 源文件数：{source_count}
- 测试文件数：{test_count}
- 测试覆盖率：{coverage}%
- 通过项：{pass_count}
- 不通过项：{fail_count}

## 详细结果

### 1. 测试完整性
| 源文件 | 测试文件 | 状态 | 说明 |
|--------|----------|------|------|
| UserServiceImpl | UserServiceImplTest | PASS | 测试完整 |
| OrderServiceImpl | - | FAIL | 缺少测试类 |

### 2. 测试覆盖率
| 模块 | 行覆盖率 | 分支覆盖率 | 状态 |
|------|----------|-----------|------|
| service | 85% | 70% | PASS |
| controller | 55% | - | FAIL |

### 3. 测试命名
| 测试类 | 检查项 | 状态 | 说明 |
|--------|--------|------|------|
| UserServiceImplTest | 方法命名 | PASS | 符合规范 |
| OrderTest | 类命名 | FAIL | 应改为 OrderServiceImplTest |

## 修复建议
1. [FAIL] OrderServiceImpl 缺少测试类，请创建 OrderServiceImplTest
2. [FAIL] controller 模块覆盖率 55%，低于 60% 要求
```

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-03-12 | 初始版本，包含 8 条测试规范规则，适配 Java/JUnit5/Spring Boot Test |
