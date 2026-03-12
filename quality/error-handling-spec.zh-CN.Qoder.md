---
name: error-handling-spec
description: Java代码错误处理规范检查。检查异常分类、自定义异常、日志记录、全局异常处理、错误码标准化等是否符合规范。当用户提到"异常处理"、"错误处理"、"Exception检查"时使用。
trigger: always_on
---

# Java 错误处理规范

你是一个 Java 代码错误处理规范审查专家。你的职责是确保代码的异常处理符合以下规范标准，提高系统的健壮性和可维护性。

---

## 规则清单

| 规则 | 名称 | 强制级别 | 优先级 |
|------|------|----------|--------|
| 规则 1 | 异常分类体系 | 【强制】 | CRITICAL |
| 规则 2 | 自定义业务异常 | 【强制】 | HIGH |
| 规则 3 | 异常日志记录 | 【强制】 | HIGH |
| 规则 4 | 全局异常处理器 | 【强制】 | CRITICAL |
| 规则 5 | Try-Catch 最佳实践 | 【强制】 | HIGH |
| 规则 6 | 错误码标准化 | 【强制】 | HIGH |
| 规则 7 | 错误恢复策略 | 【推荐】 | MEDIUM |
| 规则 8 | 用户友好的错误提示 | 【强制】 | MEDIUM |

---

## 规则 1: 异常分类体系

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

建立清晰的异常分类体系：

| 异常类型 | 说明 | 处理方式 | HTTP状态码 |
|----------|------|----------|-----------|
| 业务异常 | 验证失败、业务规则违反 | 用户可恢复，返回友好提示 | 400/422 |
| 系统异常 | 数据库连接失败、I/O错误 | 需运维介入，记录详细日志 | 500 |
| 第三方异常 | 外部API调用失败 | 重试+降级处理 | 502/503 |
| 认证异常 | 未登录、Token过期 | 引导重新登录 | 401 |
| 授权异常 | 无权限访问 | 提示权限不足 | 403 |

### 正确示例

```java
// ✅ 正确：定义异常基类
public abstract class BaseException extends RuntimeException {
    
    /** 错误码 */
    private final String code;
    
    /** 错误消息 */
    private final String message;
    
    public BaseException(String code, String message) {
        super(message);
        this.code = code;
        this.message = message;
    }
    
    // getter...
}

// ✅ 正确：业务异常
public class BusinessException extends BaseException {
    
    public BusinessException(String code, String message) {
        super(code, message);
    }
    
    public BusinessException(String message) {
        super("BUSINESS_ERROR", message);
    }
}

// ✅ 正确：系统异常
public class SystemException extends BaseException {
    
    /** 原始异常 */
    private final Throwable cause;
    
    public SystemException(String message, Throwable cause) {
        super("SYSTEM_ERROR", message);
        this.cause = cause;
    }
}

// ✅ 正确：第三方服务异常
public class ExternalServiceException extends BaseException {
    
    /** 服务名称 */
    private final String serviceName;
    
    public ExternalServiceException(String serviceName, String message) {
        super("EXTERNAL_SERVICE_ERROR", message);
        this.serviceName = serviceName;
    }
}
```

### 错误示例

```java
// ❌ 错误：使用通用 RuntimeException
throw new RuntimeException("用户不存在");

// ❌ 错误：异常类型不明确
throw new Exception("操作失败");

// ❌ 错误：没有错误码
public class MyException extends RuntimeException {
    public MyException(String message) {
        super(message);
    }
}
```

### 检查项

- [ ] 是否定义了异常基类
- [ ] 是否区分业务异常和系统异常
- [ ] 异常是否包含错误码
- [ ] 是否存在直接使用 RuntimeException/Exception 的情况

---

## 规则 2: 自定义业务异常

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

为不同业务领域创建专用异常类，包含错误代码、元数据、原始错误。

### 正确示例

```java
// ✅ 正确：参数校验异常
public class ValidationException extends BusinessException {
    
    /** 校验失败的字段 */
    private final String field;
    
    /** 校验失败的值 */
    private final Object value;
    
    public ValidationException(String field, Object value, String message) {
        super("VALIDATION_ERROR", message);
        this.field = field;
        this.value = value;
    }
}

// ✅ 正确：认证异常
public class AuthenticationException extends BusinessException {
    
    public AuthenticationException() {
        super("AUTH_FAILED", "认证失败，请重新登录");
    }
    
    public AuthenticationException(String message) {
        super("AUTH_FAILED", message);
    }
}

// ✅ 正确：资源不存在异常
public class ResourceNotFoundException extends BusinessException {
    
    /** 资源类型 */
    private final String resourceType;
    
    /** 资源ID */
    private final String resourceId;
    
    public ResourceNotFoundException(String resourceType, String resourceId) {
        super("NOT_FOUND", resourceType + " 不存在: " + resourceId);
        this.resourceType = resourceType;
        this.resourceId = resourceId;
    }
}

// ✅ 正确：权限不足异常
public class PermissionDeniedException extends BusinessException {
    
    /** 需要的权限 */
    private final String requiredPermission;
    
    public PermissionDeniedException(String requiredPermission) {
        super("PERMISSION_DENIED", "权限不足，需要权限: " + requiredPermission);
        this.requiredPermission = requiredPermission;
    }
}

// ✅ 正确：使用自定义异常
public UserVO getUserById(Long userId) {
    UserEntity user = userMapper.selectById(userId);
    if (user == null) {
        throw new ResourceNotFoundException("User", String.valueOf(userId));
    }
    return convertToVO(user);
}
```

### 检查项

- [ ] 是否为不同业务场景定义专用异常
- [ ] 异常是否包含上下文信息（字段、ID等）
- [ ] 异常消息是否清晰明确
- [ ] 是否正确继承异常基类

---

## 规则 3: 异常日志记录

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

标准化日志记录规则：

| 异常类型 | 日志级别 | 记录内容 |
|----------|----------|----------|
| 业务异常 | WARN | 用户ID、操作、失败原因 |
| 系统异常 | ERROR | 完整堆栈、请求参数、上下文 |
| 关键异常 | ERROR + 告警 | 同上 + 触发告警通知 |

**禁止**记录敏感数据：密码、Token、身份证号、银行卡号

### 正确示例

```java
// ✅ 正确：业务异常日志
@Slf4j
@Service
public class UserServiceImpl implements UserService {
    
    public void updateUser(Long userId, UserDTO dto) {
        UserEntity user = userMapper.selectById(userId);
        if (user == null) {
            // 业务异常：WARN 级别
            log.warn("更新用户失败，用户不存在 | userId={}", userId);
            throw new ResourceNotFoundException("User", String.valueOf(userId));
        }
        // ...
    }
}

// ✅ 正确：系统异常日志
public void saveUser(UserDTO dto) {
    try {
        userMapper.insert(convertToEntity(dto));
    } catch (DataAccessException e) {
        // 系统异常：ERROR 级别，记录完整堆栈
        log.error("保存用户失败 | userName={}", dto.getUserName(), e);
        throw new SystemException("保存用户数据失败", e);
    }
}

// ✅ 正确：第三方服务异常日志
public PaymentResult callPaymentService(PaymentRequest request) {
    try {
        return paymentClient.pay(request);
    } catch (FeignException e) {
        // 第三方异常：ERROR 级别
        log.error("支付服务调用失败 | orderId={}, amount={}", 
            request.getOrderId(), request.getAmount(), e);
        throw new ExternalServiceException("PaymentService", "支付服务暂不可用");
    }
}
```

### 错误示例

```java
// ❌ 错误：记录敏感信息
log.info("用户登录 | password={}", password);
log.info("支付请求 | cardNo={}", cardNo);

// ❌ 错误：日志级别不当
log.error("用户名已存在");  // 业务异常不应使用 ERROR

// ❌ 错误：没有记录异常堆栈
try {
    userMapper.insert(user);
} catch (Exception e) {
    log.error("保存失败: " + e.getMessage());  // 缺少堆栈
    throw e;
}

// ❌ 错误：日志信息不完整
log.error("操作失败");  // 没有上下文
```

### 检查项

- [ ] 业务异常是否使用 WARN 级别
- [ ] 系统异常是否使用 ERROR 级别
- [ ] 是否记录完整的异常堆栈
- [ ] 是否包含请求上下文（用户ID、操作等）
- [ ] 是否存在敏感信息泄露

---

## 规则 4: 全局异常处理器

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

实施统一的全局异常处理，确保：
- 所有异常返回统一格式
- 区分异常类型返回不同 HTTP 状态码
- 生产环境不暴露堆栈信息
- 记录异常日志

### 正确示例

```java
// ✅ 正确：全局异常处理器
@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    /**
     * 处理业务异常
     */
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ReturnData<Void>> handleBusinessException(BusinessException e) {
        log.warn("业务异常 | code={}, message={}", e.getCode(), e.getMessage());
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(ReturnData.fail(e.getCode(), e.getMessage()));
    }
    
    /**
     * 处理资源不存在异常
     */
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ReturnData<Void>> handleNotFoundException(ResourceNotFoundException e) {
        log.warn("资源不存在 | type={}, id={}", e.getResourceType(), e.getResourceId());
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(ReturnData.fail(e.getCode(), e.getMessage()));
    }
    
    /**
     * 处理认证异常
     */
    @ExceptionHandler(AuthenticationException.class)
    public ResponseEntity<ReturnData<Void>> handleAuthException(AuthenticationException e) {
        log.warn("认证失败 | message={}", e.getMessage());
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
            .body(ReturnData.fail(e.getCode(), e.getMessage()));
    }
    
    /**
     * 处理权限异常
     */
    @ExceptionHandler(PermissionDeniedException.class)
    public ResponseEntity<ReturnData<Void>> handlePermissionException(PermissionDeniedException e) {
        log.warn("权限不足 | requiredPermission={}", e.getRequiredPermission());
        return ResponseEntity.status(HttpStatus.FORBIDDEN)
            .body(ReturnData.fail(e.getCode(), e.getMessage()));
    }
    
    /**
     * 处理参数校验异常
     */
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ReturnData<Void>> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .collect(Collectors.joining(", "));
        log.warn("参数校验失败 | message={}", message);
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
            .body(ReturnData.fail("VALIDATION_ERROR", message));
    }
    
    /**
     * 处理系统异常
     */
    @ExceptionHandler(SystemException.class)
    public ResponseEntity<ReturnData<Void>> handleSystemException(SystemException e) {
        log.error("系统异常", e);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(ReturnData.fail("SYSTEM_ERROR", "系统繁忙，请稍后重试"));
    }
    
    /**
     * 处理未知异常
     */
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ReturnData<Void>> handleException(Exception e) {
        log.error("未知异常", e);
        // 生产环境不暴露详细信息
        String message = "系统繁忙，请稍后重试";
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(ReturnData.fail("SYSTEM_ERROR", message));
    }
}
```

### 检查项

- [ ] 是否有全局异常处理器（@RestControllerAdvice）
- [ ] 是否区分不同异常类型
- [ ] 是否返回正确的 HTTP 状态码
- [ ] 系统异常是否隐藏详细信息
- [ ] 是否记录异常日志

---

## 规则 5: Try-Catch 最佳实践

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 只捕获可以处理的异常
- 禁止空 catch 块（静默忽略）
- 在合适的层级捕获（Service 层）
- 使用 try-with-resources 关闭资源

### 正确示例

```java
// ✅ 正确：捕获并处理
public void saveUser(UserDTO dto) {
    try {
        userMapper.insert(convertToEntity(dto));
    } catch (DuplicateKeyException e) {
        log.warn("用户已存在 | userName={}", dto.getUserName());
        throw new BusinessException("USER_EXISTS", "用户名已存在");
    } catch (DataAccessException e) {
        log.error("保存用户失败", e);
        throw new SystemException("保存用户数据失败", e);
    }
}

// ✅ 正确：使用 try-with-resources
public String readFile(String filePath) {
    try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
        return reader.lines().collect(Collectors.joining("\n"));
    } catch (IOException e) {
        log.error("读取文件失败 | filePath={}", filePath, e);
        throw new SystemException("文件读取失败", e);
    }
}

// ✅ 正确：多资源关闭
public void processData(String inputPath, String outputPath) {
    try (
        InputStream is = new FileInputStream(inputPath);
        OutputStream os = new FileOutputStream(outputPath)
    ) {
        // 处理逻辑
    } catch (IOException e) {
        log.error("数据处理失败", e);
        throw new SystemException("数据处理失败", e);
    }
}
```

### 错误示例

```java
// ❌ 错误：空 catch 块
try {
    userMapper.insert(user);
} catch (Exception e) {
    // 什么都不做 - 静默忽略
}

// ❌ 错误：捕获后只打印日志不处理
try {
    userMapper.insert(user);
} catch (Exception e) {
    e.printStackTrace();  // 不抛出也不处理
}

// ❌ 错误：捕获范围过大
try {
    // 大段代码
} catch (Exception e) {
    throw new RuntimeException(e);
}

// ❌ 错误：资源未关闭
public void readFile(String path) {
    BufferedReader reader = new BufferedReader(new FileReader(path));
    // 使用 reader
    // 没有关闭！
}
```

### 检查项

- [ ] 是否存在空 catch 块
- [ ] 捕获后是否正确处理或重新抛出
- [ ] 捕获范围是否过大（catch Exception）
- [ ] 资源是否使用 try-with-resources 关闭
- [ ] 是否记录异常日志

---

## 规则 6: 错误码标准化

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

定义统一的错误码体系：
- 格式：`领域_操作_原因`
- 文档化所有错误码
- 客户端可据错误码采取特定操作

### 正确示例

```java
// ✅ 正确：错误码枚举
public enum ErrorCode {
    
    // ========== 通用错误 ==========
    SUCCESS("200", "操作成功"),
    SYSTEM_ERROR("500", "系统繁忙，请稍后重试"),
    PARAM_ERROR("400", "参数错误"),
    
    // ========== 认证相关 1001-1099 ==========
    AUTH_FAILED("1001", "认证失败"),
    AUTH_TOKEN_EXPIRED("1002", "登录已过期，请重新登录"),
    AUTH_TOKEN_INVALID("1003", "无效的Token"),
    
    // ========== 权限相关 1101-1199 ==========
    PERMISSION_DENIED("1101", "权限不足"),
    ROLE_NOT_FOUND("1102", "角色不存在"),
    
    // ========== 用户相关 2001-2099 ==========
    USER_NOT_FOUND("2001", "用户不存在"),
    USER_EXISTS("2002", "用户已存在"),
    USER_DISABLED("2003", "用户已被禁用"),
    
    // ========== 元素相关 3001-3099 ==========
    ELEMENT_NOT_FOUND("3001", "元素不存在"),
    ELEMENT_CODE_EXISTS("3002", "元素编码已存在"),
    ELEMENT_IN_USE("3003", "元素正在使用中，无法删除"),
    
    // ========== 数据校验 4001-4099 ==========
    VALIDATION_REQUIRED("4001", "必填字段不能为空"),
    VALIDATION_FORMAT("4002", "数据格式错误"),
    VALIDATION_LENGTH("4003", "数据长度超限"),
    
    // ========== 第三方服务 5001-5099 ==========
    EXTERNAL_SERVICE_ERROR("5001", "外部服务调用失败"),
    EXTERNAL_TIMEOUT("5002", "外部服务超时");
    
    private final String code;
    private final String message;
    
    ErrorCode(String code, String message) {
        this.code = code;
        this.message = message;
    }
    
    // getter...
}

// ✅ 正确：使用错误码
public class BusinessException extends BaseException {
    
    public BusinessException(ErrorCode errorCode) {
        super(errorCode.getCode(), errorCode.getMessage());
    }
    
    public BusinessException(ErrorCode errorCode, String detailMessage) {
        super(errorCode.getCode(), detailMessage);
    }
}

// ✅ 正确：抛出异常时使用错误码
if (user == null) {
    throw new BusinessException(ErrorCode.USER_NOT_FOUND);
}

if (elementMapper.existsByCode(dto.getElementCode())) {
    throw new BusinessException(ErrorCode.ELEMENT_CODE_EXISTS, 
        "元素编码已存在: " + dto.getElementCode());
}
```

### 检查项

- [ ] 是否定义了错误码枚举
- [ ] 错误码是否有明确的分类
- [ ] 异常是否使用错误码
- [ ] 错误码是否有对应的描述信息

---

## 规则 7: 错误恢复策略

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【推荐】

### 说明

实施优雅降级和重试机制：
- 外部服务失败：重试 + 指数退避
- 非关键功能失败：降级到默认值
- 数据库连接失败：重连 + 断路器

### 正确示例

```java
// ✅ 正确：重试机制
@Slf4j
@Component
public class RetryHelper {
    
    /**
     * 带重试的执行
     * @param supplier 执行的操作
     * @param maxRetries 最大重试次数
     * @param delayMs 初始延迟毫秒数
     */
    public <T> T executeWithRetry(Supplier<T> supplier, int maxRetries, long delayMs) {
        for (int i = 0; i < maxRetries; i++) {
            try {
                return supplier.get();
            } catch (Exception e) {
                if (i == maxRetries - 1) {
                    throw e;
                }
                // 指数退避
                long delay = delayMs * (long) Math.pow(2, i);
                log.warn("操作失败，第{}次重试，等待{}ms", i + 1, delay, e);
                try {
                    Thread.sleep(delay);
                } catch (InterruptedException ie) {
                    Thread.currentThread().interrupt();
                    throw new SystemException("重试被中断", ie);
                }
            }
        }
        throw new SystemException("重试次数已用尽", null);
    }
}

// ✅ 正确：使用 Spring Retry
@Retryable(
    value = {ExternalServiceException.class},
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000, multiplier = 2)
)
public PaymentResult callPaymentService(PaymentRequest request) {
    return paymentClient.pay(request);
}

@Recover
public PaymentResult recoverPayment(ExternalServiceException e, PaymentRequest request) {
    log.error("支付服务调用失败，进入降级处理 | orderId={}", request.getOrderId());
    // 降级处理：记录待处理，后续人工处理
    return PaymentResult.pending("支付处理中，请稍后查看结果");
}

// ✅ 正确：降级处理
public List<RecommendVO> getRecommendations(Long userId) {
    try {
        return recommendationService.getPersonalized(userId);
    } catch (Exception e) {
        log.warn("个性化推荐服务失败，使用默认推荐 | userId={}", userId, e);
        // 降级到默认推荐
        return getDefaultRecommendations();
    }
}
```

### 检查项

- [ ] 外部服务调用是否有重试机制
- [ ] 是否使用指数退避
- [ ] 非关键功能是否有降级处理
- [ ] 是否设置最大重试次数

---

## 规则 8: 用户友好的错误提示

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【强制】

### 说明

- 向用户显示：简洁、友好、可操作的消息
- 向开发者记录：详细、技术性、包含堆栈
- 避免技术术语和内部实现细节

### 示例对比

| 场景 | ❌ 错误提示 | ✅ 正确提示 |
|------|-----------|-----------|
| 数据库错误 | `SQLException: Connection refused` | `系统繁忙，请稍后重试` |
| 空指针 | `NullPointerException at line 42` | `数据异常，请联系管理员` |
| 参数错误 | `Invalid parameter: userId is null` | `用户ID不能为空` |
| 格式错误 | `DateTimeParseException` | `日期格式错误，请输入 yyyy-MM-dd 格式` |
| 权限不足 | `AccessDeniedException` | `您没有该操作的权限` |

### 检查项

- [ ] 系统错误是否返回友好提示
- [ ] 错误提示是否避免技术术语
- [ ] 是否提供可操作的建议
- [ ] 参数校验是否说明正确格式

---

## 输出报告格式

检查完成后，请按以下格式输出报告：

```markdown
# Java 错误处理规范检查报告

## 检查概览
- 检查路径：{path}
- 检查文件数：{count}
- 通过项：{pass_count}
- 警告项：{warn_count}
- 不通过项：{fail_count}

## 详细结果

### 1. 异常分类
| 类名 | 检查项 | 状态 | 说明 |
|------|--------|------|------|
| BusinessException | 异常分类 | PASS | 正确继承基类 |

### 2. 全局异常处理
| 类名 | 检查项 | 状态 | 说明 |
|------|--------|------|------|
| GlobalExceptionHandler | 存在性检查 | PASS | 已定义 |

### 3. Try-Catch 使用
| 类名 | 方法名 | 状态 | 说明 |
|------|--------|------|------|
| UserServiceImpl | saveUser | FAIL | 存在空 catch 块 |

## 修复建议
1. [FAIL] UserServiceImpl.saveUser 存在空 catch 块，请处理或记录异常
2. [WARN] OrderService 未使用错误码，建议使用 ErrorCode 枚举
```

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-03-12 | 初始版本，包含 8 条错误处理规范规则，适配 Java/Spring Boot |
