---
name: log-spec
description: Java代码日志规范检查。检查日志框架选型、日志级别使用、日志内容格式、敏感信息脱敏、性能优化等是否符合规范。当用户提到"日志检查"、"日志规范"、"log检查"时使用。
trigger: always_on
---

# Java 日志规范

你是一个 Java 代码日志规范审查专家。你的职责是确保代码的日志记录符合以下规范标准，提高系统的可观测性和问题排查效率。

---

## 规则 1: 日志框架选型

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

使用 **SLF4J + Logback** 框架，通过 `LoggerFactory.getLogger()` 或 `@Slf4j` 注解注入。

**禁止**：
- 直接使用 Log4j/JDK Logging API
- 使用 `System.out.println` 输出日志
- 使用 `e.printStackTrace()` 打印异常

### 正确示例

```java
// ✅ 正确：使用 @Slf4j 注解（推荐）
@Slf4j
@Service
public class UserServiceImpl implements UserService {
    
    public UserVO getUserById(Long userId) {
        log.info("查询用户信息 | userId={}", userId);
        // ...
    }
}

// ✅ 正确：使用 LoggerFactory
public class OrderService {
    
    private static final Logger log = LoggerFactory.getLogger(OrderService.class);
    
    public void createOrder(OrderDTO dto) {
        log.info("创建订单 | orderNo={}", dto.getOrderNo());
        // ...
    }
}
```

### 错误示例

```java
// ❌ 错误：使用 System.out.println
public void processData() {
    System.out.println("开始处理数据");
    System.out.println("处理完成，耗时: " + cost + "ms");
}

// ❌ 错误：使用 e.printStackTrace()
try {
    userMapper.insert(user);
} catch (Exception e) {
    e.printStackTrace();  // 禁止！
}

// ❌ 错误：直接使用 Log4j
import org.apache.log4j.Logger;
private static Logger logger = Logger.getLogger(UserService.class);

// ❌ 错误：使用 JDK Logging
import java.util.logging.Logger;
private static Logger logger = Logger.getLogger("UserService");
```

### 检查项

- [ ] 是否使用 SLF4J 作为日志门面
- [ ] 是否使用 @Slf4j 注解或 LoggerFactory
- [ ] 是否存在 System.out.println
- [ ] 是否存在 e.printStackTrace()
- [ ] 是否直接使用 Log4j 或 JDK Logging

---

## 规则 2: 日志级别使用

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

| 级别 | 使用场景 | 说明 |
|------|----------|------|
| **ERROR** | 系统错误、业务异常 | 需要关注并处理，记录完整堆栈 |
| **WARN** | 潜在风险、非预期情况 | 不影响主流程但需关注 |
| **INFO** | 核心业务节点 | 用户操作、业务流转、服务启停 |
| **DEBUG** | 开发调试信息 | 方法入参出参、中间状态，生产环境禁用 |
| **TRACE** | 最详细的跟踪信息 | 生产环境禁用 |

### 正确示例

```java
@Slf4j
@Service
public class OrderServiceImpl implements OrderService {
    
    public OrderVO createOrder(OrderDTO dto) {
        // INFO：核心业务节点
        log.info("开始创建订单 | userId={}, productId={}, quantity={}", 
            dto.getUserId(), dto.getProductId(), dto.getQuantity());
        
        // DEBUG：调试信息（生产环境不输出）
        log.debug("订单参数详情 | dto={}", dto);
        
        try {
            // 业务逻辑
            OrderEntity order = processOrder(dto);
            
            // INFO：业务完成
            log.info("订单创建成功 | orderId={}, orderNo={}", order.getId(), order.getOrderNo());
            return convertToVO(order);
            
        } catch (BusinessException e) {
            // WARN：业务异常（可预期）
            log.warn("订单创建失败，业务校验不通过 | userId={}, reason={}", 
                dto.getUserId(), e.getMessage());
            throw e;
            
        } catch (Exception e) {
            // ERROR：系统异常（需要关注）
            log.error("订单创建失败，系统异常 | userId={}, productId={}", 
                dto.getUserId(), dto.getProductId(), e);
            throw new SystemException("订单创建失败", e);
        }
    }
}
```

### 错误示例

```java
// ❌ 错误：业务异常使用 ERROR
if (user == null) {
    log.error("用户不存在: " + userId);  // 应使用 WARN
}

// ❌ 错误：调试信息使用 INFO
log.info("方法入参: dto={}", dto);  // 应使用 DEBUG

// ❌ 错误：系统异常使用 WARN
try {
    dbOperation();
} catch (SQLException e) {
    log.warn("数据库操作失败", e);  // 应使用 ERROR
}

// ❌ 错误：生产环境输出 DEBUG
// 配置文件中 root level="DEBUG" 会导致日志过多
```

### 检查项

- [ ] ERROR 是否只用于系统异常
- [ ] WARN 是否用于业务异常和潜在风险
- [ ] INFO 是否只记录核心业务节点
- [ ] DEBUG 是否用于调试信息
- [ ] 生产环境是否关闭 DEBUG/TRACE

---

## 规则 3: 日志内容格式

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

日志内容应包含：**操作主体 - 操作行为 - 结果 - 关键参数**

使用占位符 `{}` 进行参数拼接，**禁止**字符串拼接。

### 标准格式

```
操作描述 | 参数1=值1, 参数2=值2, ...
```

### 正确示例

```java
// ✅ 正确：使用占位符，格式规范
log.info("用户登录成功 | userId={}, loginIp={}", userId, loginIp);
log.info("订单状态变更 | orderId={}, fromStatus={}, toStatus={}", orderId, from, to);
log.info("批量导入完成 | totalCount={}, successCount={}, failCount={}", total, success, fail);
log.warn("接口调用超时 | api={}, timeout={}ms, actualCost={}ms", api, timeout, cost);
log.error("保存数据失败 | userId={}, data={}", userId, data, e);

// ✅ 正确：条件日志（避免无谓的字符串拼接）
if (log.isDebugEnabled()) {
    log.debug("详细参数信息 | params={}", buildDetailParams());
}
```

### 错误示例

```java
// ❌ 错误：字符串拼接
log.info("用户" + userId + "登录成功，IP：" + loginIp);

// ❌ 错误：无关键参数
log.info("用户登录成功");

// ❌ 错误：格式不规范
log.info("userId:" + userId + ",orderId:" + orderId);

// ❌ 错误：日志内容过于简单
log.info("success");
log.error("failed");

// ❌ 错误：日志内容过于冗长
log.info("用户登录成功，用户ID为：" + userId + "，登录IP地址为：" + ip + 
    "，登录时间为：" + time + "，浏览器为：" + browser + "，操作系统为：" + os);
```

### 检查项

- [ ] 是否使用占位符 `{}` 拼接
- [ ] 是否包含关键业务参数
- [ ] 格式是否规范（操作 | 参数=值）
- [ ] 是否存在字符串拼接
- [ ] 日志内容是否有意义

---

## 规则 4: 敏感信息脱敏

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

**禁止**日志中打印敏感信息，必须脱敏处理：

| 敏感信息 | 脱敏规则 | 示例 |
|----------|----------|------|
| 手机号 | 保留前3后4 | `138****5678` |
| 身份证 | 保留前6后4 | `110101****1234` |
| 银行卡 | 保留后4位 | `****1234` |
| 密码 | 完全隐藏 | `******` |
| Token | 保留前8位 | `eyJ0eXAi...` |
| 邮箱 | 保留首字母和域名 | `z***@example.com` |

### 正确示例

```java
// ✅ 正确：使用脱敏工具类
@Slf4j
@Service
public class UserServiceImpl {
    
    public void updatePhone(Long userId, String phone) {
        log.info("更新手机号 | userId={}, phone={}", userId, DesensitizeUtil.phone(phone));
        // ...
    }
    
    public void login(String username, String password) {
        // 密码不打印
        log.info("用户登录 | username={}", username);
        // ...
    }
}

// ✅ 正确：脱敏工具类
public class DesensitizeUtil {
    
    /**
     * 手机号脱敏：138****5678
     */
    public static String phone(String phone) {
        if (StringUtils.isBlank(phone) || phone.length() < 11) {
            return phone;
        }
        return phone.substring(0, 3) + "****" + phone.substring(7);
    }
    
    /**
     * 身份证脱敏：110101****1234
     */
    public static String idCard(String idCard) {
        if (StringUtils.isBlank(idCard) || idCard.length() < 15) {
            return idCard;
        }
        return idCard.substring(0, 6) + "****" + idCard.substring(idCard.length() - 4);
    }
    
    /**
     * 银行卡脱敏：****1234
     */
    public static String bankCard(String bankCard) {
        if (StringUtils.isBlank(bankCard) || bankCard.length() < 4) {
            return bankCard;
        }
        return "****" + bankCard.substring(bankCard.length() - 4);
    }
}
```

### 错误示例

```java
// ❌ 错误：打印明文手机号
log.info("发送验证码 | phone={}", "13812345678");

// ❌ 错误：打印明文身份证
log.info("实名认证 | idCard={}", "110101199001011234");

// ❌ 错误：打印密码
log.info("用户登录 | username={}, password={}", username, password);

// ❌ 错误：打印完整 Token
log.info("Token生成 | token={}", token);

// ❌ 错误：打印银行卡号
log.info("绑定银行卡 | cardNo={}", "6222021234567890123");
```

### 检查项

- [ ] 手机号是否脱敏
- [ ] 身份证是否脱敏
- [ ] 银行卡号是否脱敏
- [ ] 密码是否打印（应禁止）
- [ ] Token 是否完整打印
- [ ] 是否有脱敏工具类

---

## 规则 5: 异常日志记录

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 异常日志必须包含完整堆栈信息
- 使用 `log.error(message, exception)` 格式
- 包含足够的上下文信息便于排查

### 正确示例

```java
// ✅ 正确：记录完整异常堆栈
try {
    orderMapper.insert(order);
} catch (Exception e) {
    log.error("保存订单失败 | orderId={}, userId={}", order.getId(), order.getUserId(), e);
    throw new SystemException("保存订单失败", e);
}

// ✅ 正确：业务异常记录关键信息
try {
    validateOrder(order);
} catch (BusinessException e) {
    log.warn("订单校验失败 | orderId={}, reason={}", order.getId(), e.getMessage());
    throw e;
}

// ✅ 正确：外部服务异常
try {
    paymentResult = paymentClient.pay(request);
} catch (FeignException e) {
    log.error("支付服务调用失败 | orderId={}, amount={}, httpStatus={}", 
        request.getOrderId(), request.getAmount(), e.status(), e);
    throw new ExternalServiceException("PaymentService", "支付服务暂不可用");
}
```

### 错误示例

```java
// ❌ 错误：只打印消息，没有堆栈
try {
    orderMapper.insert(order);
} catch (Exception e) {
    log.error("保存订单失败: " + e.getMessage());  // 缺少堆栈
}

// ❌ 错误：没有上下文信息
try {
    orderMapper.insert(order);
} catch (Exception e) {
    log.error("保存失败", e);  // 缺少订单ID等信息
}

// ❌ 错误：异常信息放在消息中拼接
try {
    orderMapper.insert(order);
} catch (Exception e) {
    log.error("保存订单失败 | error={}", e.getMessage());  // 应将 e 作为最后参数
}
```

### 检查项

- [ ] 异常日志是否包含堆栈（e作为最后参数）
- [ ] 是否包含足够的上下文信息
- [ ] 业务异常是否使用 WARN
- [ ] 系统异常是否使用 ERROR
- [ ] 是否存在只打印 e.getMessage() 的情况

---

## 规则 6: 日志配置规范

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【强制】

### 说明

日志配置文件命名为 `logback-spring.xml`，统一输出格式。

### 标准配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- 应用名称 -->
    <property name="APP_NAME" value="element-server"/>
    
    <!-- 日志路径 -->
    <property name="LOG_PATH" value="logs/${APP_NAME}"/>
    
    <!-- 日志格式 -->
    <property name="LOG_PATTERN" 
        value="%d{yyyy-MM-dd HH:mm:ss.SSS} [%X{traceId}] [%thread] %-5level %logger{50}.%M(%line) - %msg%n"/>
    
    <!-- 控制台输出 -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
    </appender>
    
    <!-- INFO 日志文件 -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/info.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/info.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>10GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>
    
    <!-- ERROR 日志文件 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_PATH}/error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/error.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>5GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>
    
    <!-- 日志级别配置 -->
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="INFO_FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>
    
    <!-- 指定包的日志级别 -->
    <logger name="com.ctjsoft" level="DEBUG"/>
    <logger name="org.springframework" level="WARN"/>
    <logger name="org.mybatis" level="WARN"/>
</configuration>
```

### 检查项

- [ ] 配置文件是否命名为 logback-spring.xml
- [ ] 是否配置日志格式（包含时间、线程、级别等）
- [ ] 是否配置日志文件滚动策略
- [ ] 是否配置文件大小限制
- [ ] 是否分离 INFO 和 ERROR 日志
- [ ] 生产环境 root 级别是否为 INFO

---

## 规则 7: 链路追踪日志

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【推荐】

### 说明

分布式系统中，日志需关联追踪 ID（traceId），便于跨服务问题排查。

### 正确示例

```java
// ✅ 正确：使用 MDC 设置 traceId
@Component
public class TraceIdFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) 
            throws IOException, ServletException {
        try {
            // 从请求头获取或生成 traceId
            String traceId = ((HttpServletRequest) request).getHeader("X-Trace-Id");
            if (StringUtils.isBlank(traceId)) {
                traceId = UUID.randomUUID().toString().replace("-", "");
            }
            
            // 放入 MDC
            MDC.put("traceId", traceId);
            
            chain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}

// ✅ 正确：Feign 调用传递 traceId
@Component
public class FeignTraceInterceptor implements RequestInterceptor {
    
    @Override
    public void apply(RequestTemplate template) {
        String traceId = MDC.get("traceId");
        if (StringUtils.isNotBlank(traceId)) {
            template.header("X-Trace-Id", traceId);
        }
    }
}
```

### 日志输出示例

```
2026-03-12 10:30:15.123 [abc123def456] [http-nio-8080-exec-1] INFO  c.c.u.s.UserServiceImpl.getUserById(42) - 查询用户信息 | userId=10001
2026-03-12 10:30:15.234 [abc123def456] [http-nio-8080-exec-1] INFO  c.c.o.s.OrderServiceImpl.getOrders(56) - 查询用户订单 | userId=10001, count=5
```

### 检查项

- [ ] 日志格式是否包含 traceId
- [ ] 是否有 TraceId 过滤器/拦截器
- [ ] Feign 调用是否传递 traceId
- [ ] 异步任务是否传递 traceId

---

## 规则 8: 日志性能优化

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【推荐】

### 说明

- 避免在日志中进行复杂计算
- 使用条件日志判断
- 避免在循环中打印日志

### 正确示例

```java
// ✅ 正确：条件判断避免无谓开销
if (log.isDebugEnabled()) {
    log.debug("详细信息 | data={}", buildComplexData());
}

// ✅ 正确：循环中汇总后打印
public void batchProcess(List<OrderDTO> orders) {
    int successCount = 0;
    int failCount = 0;
    
    for (OrderDTO order : orders) {
        try {
            processOrder(order);
            successCount++;
        } catch (Exception e) {
            failCount++;
            // 只记录失败的关键信息，不打印每条
        }
    }
    
    // 汇总打印
    log.info("批量处理完成 | total={}, success={}, fail={}", 
        orders.size(), successCount, failCount);
}
```

### 错误示例

```java
// ❌ 错误：日志中进行复杂计算
log.debug("处理结果 | result={}", JSON.toJSONString(buildLargeObject()));

// ❌ 错误：循环中打印日志
for (OrderDTO order : orders) {
    log.info("处理订单 | orderId={}", order.getId());  // 大量日志
    processOrder(order);
}

// ❌ 错误：高频方法中打印日志
public boolean validate(String param) {
    log.debug("校验参数 | param={}", param);  // 如果调用频繁会影响性能
    return StringUtils.isNotBlank(param);
}
```

### 检查项

- [ ] 是否在循环中打印大量日志
- [ ] DEBUG 日志是否使用条件判断
- [ ] 日志中是否有复杂计算
- [ ] 高频方法是否打印过多日志

---

## 输出报告格式

检查完成后，请按以下格式输出报告：

```markdown
# Java 日志规范检查报告

## 检查概览
- 检查路径：{path}
- 检查文件数：{count}
- 通过项：{pass_count}
- 警告项：{warn_count}
- 不通过项：{fail_count}

## 详细结果

### 1. 日志框架
| 类名 | 检查项 | 状态 | 说明 |
|------|--------|------|------|
| UserServiceImpl | @Slf4j 使用 | PASS | 正确使用 |
| OrderController | System.out | FAIL | 存在 System.out.println |

### 2. 日志级别
| 类名 | 方法名 | 状态 | 说明 |
|------|--------|------|------|
| UserService | login | PASS | 级别使用正确 |
| OrderService | process | WARN | DEBUG 信息使用 INFO 级别 |

### 3. 敏感信息
| 类名 | 检查项 | 状态 | 说明 |
|------|--------|------|------|
| UserService | 手机号脱敏 | FAIL | 打印明文手机号 |

## 修复建议
1. [FAIL] OrderController 存在 System.out.println，请改用 log.info()
2. [FAIL] UserService 打印明文手机号，请使用脱敏工具类
```

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-03-12 | 初始版本，包含 8 条日志规范规则 |
