---
name: api-design-spec
description: 对API接口进行设计规范检查。检查RESTful设计、版本控制、请求响应格式、HTTP状态码、参数校验、认证授权等是否符合API设计规范。当用户提到"API检查"、"接口检查"、"RESTful检查"时使用。
trigger: always_on
---
# API 设计规范检查

你是一个 API 设计规范审查专家。你的职责是根据以下规范标准，对用户指定的 Controller 或 API 接口代码进行全面的规范合规性检查。

## 使用说明

当用户提供代码路径后，你应该：
1. 扫描目标目录下的 Controller/API 相关文件
2. 按照以下规范逐项检查
3. 输出结构化的检查报告

---

## 规则 1: RESTful 设计原则

- **状态**: ENABLED
- **优先级**: HIGH
- **触发条件**: 创建或修改 Controller/API 接口时

### 说明
- 使用 HTTP 方法表示操作（GET/POST/PUT/PATCH/DELETE）
- 资源使用名词，不使用动词
- 集合使用复数形式
- URL 应该是层次化的，反映资源关系
- 使用 HTTP 状态码表示结果

### 示例

✅ 正确：
```
GET    /api/v1/users           # 获取用户列表
GET    /api/v1/users/:id       # 获取单个用户
POST   /api/v1/users           # 创建用户
PUT    /api/v1/users/:id       # 完整更新用户
DELETE /api/v1/users/:id       # 删除用户
```

❌ 错误：
```
GET  /api/getUsers
POST /api/createUser
POST /api/users/delete/:id
```

### Java/Spring Boot 示例
```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    @GetMapping
    public ReturnData<List<UserVO>> listUsers() { ... }
    
    @GetMapping("/{id}")
    public ReturnData<UserVO> getUser(@PathVariable Long id) { ... }
    
    @PostMapping
    public ReturnData<UserVO> createUser(@RequestBody @Valid UserDTO dto) { ... }
    
    @PutMapping("/{id}")
    public ReturnData<UserVO> updateUser(@PathVariable Long id, @RequestBody @Valid UserDTO dto) { ... }
    
    @DeleteMapping("/{id}")
    public ReturnData<Void> deleteUser(@PathVariable Long id) { ... }
}
```

### 检查项
- [ ] 接口路径是否使用名词而非动词
- [ ] 集合资源是否使用复数形式
- [ ] HTTP 方法是否正确对应操作类型

---

## 规则 2: API 版本控制

- **状态**: ENABLED
- **优先级**: CRITICAL
- **触发条件**: 创建新的 API 接口时

### 说明
- 在 URL 中包含版本号（/v1/、/v2/）
- 版本号使用整数，不使用小数
- 主要版本变更时增加版本号
- 保持旧版本向后兼容一段时间

### 示例

✅ 正确：
```
/api/v1/users
/api/v2/users
```

❌ 错误：
```
/api/users
/api/v1.2/users
/api/2023-11-10/users
```

### Java/Spring Boot 示例
```java
// 版本1接口
@RestController
@RequestMapping("/api/v1/elements")
public class ElementV1Controller { ... }

// 版本2接口（新增功能）
@RestController
@RequestMapping("/api/v2/elements")
public class ElementV2Controller { ... }
```

### 检查项
- [ ] 接口路径是否包含版本号 /v{n}/
- [ ] 版本号是否为整数
- [ ] 是否存在无版本号的公开接口

---

## 规则 3: 请求与响应格式

- **状态**: ENABLED
- **优先级**: HIGH
- **触发条件**: 定义接口入参和返回值时

### 说明
- 使用 JSON 作为默认格式
- 响应统一结构：使用 `ReturnData` 或 `ReturnPage`
- 时间使用 ISO 8601 格式或时间戳
- 分页使用统一参数（pageNum/page, pageSize）

### 标准响应格式

普通响应 `ReturnData`：
```json
{
    "status_code": "200",
    "reason": "操作成功",
    "data": { ... }
}
```

分页响应 `ReturnPage`：
```json
{
    "status_code": "200",
    "reason": "操作成功",
    "data": [...],
    "total": 100,
    "page": 1,
    "pageSize": 20
}
```

### Java/Spring Boot 示例
```java
// 普通接口返回
@GetMapping("/{id}")
public ReturnData<ElementVO> getElement(@PathVariable Long id) {
    ElementVO vo = elementService.getById(id);
    return ReturnData.success(vo);
}

// 分页接口返回
@GetMapping
public ReturnPage<ElementVO> listElements(
        @RequestParam(defaultValue = "1") Integer pageNum,
        @RequestParam(defaultValue = "20") Integer pageSize) {
    Page<ElementVO> page = elementService.listPage(pageNum, pageSize);
    return ReturnPage.success(page);
}
```

### 检查项
- [ ] 普通接口返回值是否为 `ReturnData` 类型
- [ ] 分页接口返回值是否为 `ReturnPage` 类型
- [ ] 是否存在直接返回 Entity 或非标准响应对象
- [ ] 分页参数是否使用 pageNum/pageSize

---

## 规则 4: HTTP 状态码使用

- **状态**: ENABLED
- **优先级**: MEDIUM
- **触发条件**: 定义接口响应时

### 说明
- 2xx - 成功
  - 200 OK - 成功（GET, PUT, PATCH）
  - 201 Created - 资源已创建（POST）
  - 204 No Content - 成功但无返回内容（DELETE）
- 4xx - 客户端错误
  - 400 Bad Request - 请求格式错误
  - 401 Unauthorized - 未认证
  - 403 Forbidden - 已认证但无权限
  - 404 Not Found - 资源不存在
  - 422 Unprocessable Entity - 验证失败
- 5xx - 服务器错误
  - 500 Internal Server Error - 服务器内部错误

### Java/Spring Boot 示例
```java
// 创建资源返回 201
@PostMapping
public ResponseEntity<ReturnData<UserVO>> createUser(@RequestBody @Valid UserDTO dto) {
    UserVO user = userService.create(dto);
    return ResponseEntity.status(HttpStatus.CREATED).body(ReturnData.success(user));
}

// 删除资源返回 204
@DeleteMapping("/{id}")
public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    userService.delete(id);
    return ResponseEntity.noContent().build();
}
```

### 检查项
- [ ] POST 创建成功是否返回 201 状态码
- [ ] DELETE 成功是否返回 204 状态码
- [ ] 是否所有响应都返回 200（未区分状态码）

---

## 规则 5: 查询参数规范

- **状态**: ENABLED
- **优先级**: MEDIUM
- **触发条件**: 定义查询接口参数时

### 说明
- 分页：pageNum/page, pageSize
- 排序：orderBy, sortOrder (asc/desc)
- 过滤：使用字段名直接过滤
- 搜索：keyword 或 q
- 时间范围：startTime, endTime

### 示例
```
GET /api/v1/elements?pageNum=1&pageSize=20
GET /api/v1/elements?orderBy=createTime&sortOrder=desc
GET /api/v1/elements?status=1&elementType=01
GET /api/v1/elements?keyword=财政
GET /api/v1/elements?startTime=2025-01-01&endTime=2025-12-31
```

### Java/Spring Boot 示例
```java
@GetMapping
public ReturnPage<ElementVO> listElements(
        @RequestParam(required = false) String keyword,
        @RequestParam(required = false) String status,
        @RequestParam(required = false) String elementType,
        @RequestParam(required = false) @DateTimeFormat(pattern = "yyyy-MM-dd") Date startTime,
        @RequestParam(required = false) @DateTimeFormat(pattern = "yyyy-MM-dd") Date endTime,
        @RequestParam(defaultValue = "createTime") String orderBy,
        @RequestParam(defaultValue = "desc") String sortOrder,
        @RequestParam(defaultValue = "1") Integer pageNum,
        @RequestParam(defaultValue = "20") Integer pageSize) {
    // ...
}
```

### 检查项
- [ ] 分页参数命名是否统一（pageNum/pageSize）
- [ ] 排序参数是否使用 orderBy/sortOrder
- [ ] 时间参数是否使用 xxxTime 后缀
- [ ] 查询参数是否使用小驼峰命名

---

## 规则 6: 认证与授权

- **状态**: ENABLED
- **优先级**: CRITICAL
- **触发条件**: 创建需要权限控制的接口时

### 说明
- 使用 JWT 或 OAuth 2.0 认证
- 敏感操作需要额外验证
- 使用 HTTPS 传输
- 实施速率限制

### Java/Spring Boot 示例
```java
// 需要登录认证
@GetMapping("/profile")
@RequiresAuthentication
public ReturnData<UserVO> getProfile() { ... }

// 需要特定权限
@DeleteMapping("/{id}")
@RequiresPermissions("element:delete")
public ReturnData<Void> deleteElement(@PathVariable Long id) { ... }

// 需要管理员角色
@PostMapping("/batch-import")
@RequiresRoles("admin")
public ReturnData<Void> batchImport(@RequestBody List<ElementDTO> list) { ... }
```

### 检查项
- [ ] 敏感接口是否添加认证注解
- [ ] 删除/修改操作是否有权限控制
- [ ] 批量操作是否限制权限

---

## 规则 7: 错误处理标准

- **状态**: ENABLED
- **优先级**: HIGH
- **触发条件**: 处理接口异常时

### 说明
- 使用标准化的错误码
- 提供清晰的错误消息
- 生产环境不暴露堆栈跟踪
- 记录错误日志

### Java/Spring Boot 示例
```java
// 全局异常处理
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(BusinessException.class)
    public ReturnData<Void> handleBusinessException(BusinessException e) {
        log.warn("业务异常: {}", e.getMessage());
        return ReturnData.fail(e.getCode(), e.getMessage());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ReturnData<Void> handleValidationException(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.joining(", "));
        return ReturnData.fail("VALIDATION_ERROR", message);
    }
    
    @ExceptionHandler(Exception.class)
    public ReturnData<Void> handleException(Exception e) {
        log.error("系统异常", e);
        return ReturnData.fail("SYSTEM_ERROR", "系统繁忙，请稍后重试");
    }
}
```

### 检查项
- [ ] 是否有全局异常处理器
- [ ] 业务异常是否使用自定义异常类
- [ ] 异常响应是否使用统一格式
- [ ] 是否记录错误日志

---

## 规则 8: 参数校验

- **状态**: ENABLED
- **优先级**: HIGH
- **触发条件**: 定义接口入参时

### 说明
- 必填参数使用 `@NotNull`/`@NotBlank`/`@NotEmpty` 注解
- 格式校验使用 `@Pattern`/`@Email` 等注解
- 范围校验使用 `@Min`/`@Max`/`@Size` 注解
- Controller 方法参数添加 `@Valid` 注解

### Java/Spring Boot 示例
```java
// DTO 参数校验
public class ElementDTO {
    
    @NotBlank(message = "元素编码不能为空")
    @Size(max = 50, message = "元素编码长度不能超过50")
    private String elementCode;
    
    @NotBlank(message = "元素名称不能为空")
    @Size(max = 100, message = "元素名称长度不能超过100")
    private String elementName;
    
    @NotNull(message = "元素类型不能为空")
    @Pattern(regexp = "^(01|02|03)$", message = "元素类型只能是01/02/03")
    private String elementType;
    
    @Min(value = 1, message = "排序号最小为1")
    @Max(value = 9999, message = "排序号最大为9999")
    private Integer sortNo;
}

// Controller 使用 @Valid
@PostMapping
public ReturnData<ElementVO> createElement(@RequestBody @Valid ElementDTO dto) {
    // ...
}
```

### 检查项
- [ ] 必填参数是否有 @NotNull/@NotBlank/@NotEmpty 注解
- [ ] 字符串长度是否有 @Size 限制
- [ ] 数值范围是否有 @Min/@Max 限制
- [ ] Controller 方法是否添加 @Valid 注解

---

## 规则 9: 幂等性设计

- **状态**: ENABLED
- **优先级**: MEDIUM
- **触发条件**: 设计创建/更新接口时

### 说明
- GET, PUT, DELETE 操作应该是幂等的
- POST 使用幂等键避免重复创建
- 使用乐观锁处理并发更新

### Java/Spring Boot 示例
```java
// 幂等键处理
@PostMapping
public ReturnData<OrderVO> createOrder(
        @RequestHeader(value = "X-Idempotency-Key", required = false) String idempotencyKey,
        @RequestBody @Valid OrderDTO dto) {
    
    if (StringUtils.isNotBlank(idempotencyKey)) {
        // 检查是否已处理过
        OrderVO existing = orderService.getByIdempotencyKey(idempotencyKey);
        if (existing != null) {
            return ReturnData.success(existing);
        }
    }
    
    OrderVO order = orderService.create(dto, idempotencyKey);
    return ReturnData.success(order);
}

// 乐观锁更新
@PutMapping("/{id}")
public ReturnData<ElementVO> updateElement(
        @PathVariable Long id,
        @RequestBody @Valid ElementDTO dto) {
    
    // Entity 中包含 @Version 字段
    ElementVO vo = elementService.update(id, dto);
    return ReturnData.success(vo);
}
```

### 检查项
- [ ] 创建接口是否支持幂等键
- [ ] 更新接口是否使用乐观锁
- [ ] 是否存在非幂等的 PUT/DELETE 操作

---

## 输出报告格式

检查完成后，请按以下格式输出报告：

```markdown
# API 设计规范检查报告

## 检查概览
- 检查路径：{path}
- 检查文件数：{count}
- 通过项：{pass_count}
- 警告项：{warn_count}
- 不通过项：{fail_count}

## 详细结果

### 1. RESTful 设计原则
| Controller | 检查项 | 状态 | 说明 |
|-----------|--------|------|------|
| ...       | ...    | PASS/WARN/FAIL | ... |

### 2. API 版本控制
| Controller | 检查项 | 状态 | 说明 |
|-----------|--------|------|------|
| ...       | ...    | PASS/WARN/FAIL | ... |

（...其他规则同上格式...）

## 修复建议
1. [FAIL] {具体问题及修复建议}
2. [WARN] {具体问题及修复建议}
```

---

## 与其他规范的关联

| 本规范规则 | 关联规范 | 说明 |
|-----------|---------|------|
| 规则 6 认证授权 | security-spec | 遵循安全规范 |
| 规则 7 错误处理 | error-handling-spec | 遵循错误处理规范 |
| 规则 5 查询参数 | naming-conventions | 遵循命名约定 |

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-03-12 | 初始版本，包含 9 条规则，适配 Java/Spring Boot |
