---
name: security-spec
description: Java代码安全规范检查。检查SQL注入防护、XSS防护、敏感数据保护、认证授权、接口安全、文件上传安全等是否符合安全规范。当用户提到"安全检查"、"漏洞检查"、"SQL注入"、"XSS"时使用。
trigger: always_on
---

# Java 安全规范

你是一个 Java 代码安全规范审查专家。你的职责是确保代码符合以下安全规范标准，防止常见安全漏洞，保护系统和用户数据安全。

---

## 规则 1: SQL 注入防护

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

**禁止**直接拼接 SQL 语句，必须使用参数化查询或 ORM 框架。

### 正确示例

```java
// ✅ 正确：使用 MyBatis 参数化查询
@Mapper
public interface UserMapper {
    
    @Select("SELECT * FROM user WHERE id = #{userId}")
    UserEntity selectById(@Param("userId") Long userId);
    
    @Select("SELECT * FROM user WHERE username = #{username} AND status = #{status}")
    List<UserEntity> selectByCondition(@Param("username") String username, 
                                       @Param("status") Integer status);
}

// ✅ 正确：MyBatis XML 参数化
<select id="selectByName" resultType="UserEntity">
    SELECT * FROM user WHERE username = #{username}
</select>

// ✅ 正确：JPA 参数化查询
@Repository
public interface UserRepository extends JpaRepository<UserEntity, Long> {
    
    @Query("SELECT u FROM UserEntity u WHERE u.username = :username")
    UserEntity findByUsername(@Param("username") String username);
}

// ✅ 正确：JdbcTemplate 参数化
public UserEntity findById(Long userId) {
    String sql = "SELECT * FROM user WHERE id = ?";
    return jdbcTemplate.queryForObject(sql, new Object[]{userId}, userRowMapper);
}
```

### 错误示例

```java
// ❌ 错误：字符串拼接 SQL
public List<UserEntity> findByName(String name) {
    String sql = "SELECT * FROM user WHERE username = '" + name + "'";
    return jdbcTemplate.query(sql, userRowMapper);
}

// ❌ 错误：MyBatis 使用 ${} 拼接
<select id="selectByName" resultType="UserEntity">
    SELECT * FROM user WHERE username = '${username}'  <!-- 危险！ -->
</select>

// ❌ 错误：动态拼接 ORDER BY
public List<UserEntity> findAll(String orderBy) {
    String sql = "SELECT * FROM user ORDER BY " + orderBy;  // SQL注入风险
    return jdbcTemplate.query(sql, userRowMapper);
}
```

### 检查项

- [ ] 是否存在字符串拼接 SQL
- [ ] MyBatis 是否使用 `#{}` 而非 `${}`
- [ ] 动态 SQL 是否安全处理
- [ ] ORDER BY 字段是否白名单校验
- [ ] LIKE 查询是否正确转义

---

## 规则 2: XSS 攻击防护

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

- 所有用户输入必须转义后再输出
- 富文本内容需过滤危险标签
- 设置 Content-Type 和 CSP 响应头

### 正确示例

```java
// ✅ 正确：使用 HTML 转义工具
import org.apache.commons.text.StringEscapeUtils;

public String safeOutput(String userInput) {
    return StringEscapeUtils.escapeHtml4(userInput);
}

// ✅ 正确：使用 OWASP Java Encoder
import org.owasp.encoder.Encode;

public String safeOutput(String userInput) {
    return Encode.forHtml(userInput);
}

// ✅ 正确：富文本过滤危险标签
import org.jsoup.Jsoup;
import org.jsoup.safety.Safelist;

public String sanitizeHtml(String html) {
    // 只允许基本格式标签
    return Jsoup.clean(html, Safelist.basic());
}

// ✅ 正确：设置安全响应头
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.headers()
            .contentTypeOptions()      // X-Content-Type-Options: nosniff
            .and()
            .xssProtection()           // X-XSS-Protection: 1; mode=block
            .and()
            .contentSecurityPolicy("default-src 'self'");  // CSP
    }
}
```

### 错误示例

```java
// ❌ 错误：直接输出用户输入
@GetMapping("/user/{id}")
public String getUserInfo(@PathVariable Long id, Model model) {
    UserVO user = userService.getById(id);
    model.addAttribute("username", user.getUsername());  // 未转义
    return "user";
}

// ❌ 错误：直接存储未过滤的 HTML
public void saveArticle(ArticleDTO dto) {
    ArticleEntity entity = new ArticleEntity();
    entity.setContent(dto.getContent());  // 未过滤危险标签
    articleMapper.insert(entity);
}
```

### 检查项

- [ ] 用户输入输出前是否转义
- [ ] 富文本是否过滤危险标签
- [ ] 是否设置 X-XSS-Protection 响应头
- [ ] 是否设置 Content-Security-Policy
- [ ] JSON 响应 Content-Type 是否正确

---

## 规则 3: 敏感数据保护

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

- 密码使用 BCrypt/Argon2 单向哈希，加盐
- 敏感数据（身份证、银行卡）加密存储
- 使用 HTTPS 传输敏感数据
- 不在日志、URL、错误消息中暴露敏感数据

### 正确示例

```java
// ✅ 正确：密码哈希
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

@Service
public class UserServiceImpl implements UserService {
    
    private final BCryptPasswordEncoder passwordEncoder = new BCryptPasswordEncoder();
    
    /**
     * 用户注册 - 密码哈希存储
     */
    public void register(UserDTO dto) {
        UserEntity user = new UserEntity();
        user.setUsername(dto.getUsername());
        // 密码哈希，不存储明文
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        userMapper.insert(user);
    }
    
    /**
     * 用户登录 - 密码验证
     */
    public boolean verifyPassword(String rawPassword, String encodedPassword) {
        return passwordEncoder.matches(rawPassword, encodedPassword);
    }
}

// ✅ 正确：敏感数据加密存储
import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;

@Component
public class EncryptUtil {
    
    @Value("${encrypt.key}")
    private String encryptKey;
    
    /**
     * AES 加密
     */
    public String encrypt(String data) {
        try {
            SecretKeySpec key = new SecretKeySpec(encryptKey.getBytes(), "AES");
            Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
            cipher.init(Cipher.ENCRYPT_MODE, key);
            byte[] encrypted = cipher.doFinal(data.getBytes());
            return Base64.getEncoder().encodeToString(encrypted);
        } catch (Exception e) {
            throw new SystemException("加密失败", e);
        }
    }
    
    /**
     * AES 解密
     */
    public String decrypt(String encryptedData) {
        // 解密逻辑
    }
}

// ✅ 正确：存储加密的身份证号
public void saveUser(UserDTO dto) {
    UserEntity user = new UserEntity();
    user.setIdCard(encryptUtil.encrypt(dto.getIdCard()));  // 加密存储
    userMapper.insert(user);
}
```

### 错误示例

```java
// ❌ 错误：明文存储密码
user.setPassword(dto.getPassword());

// ❌ 错误：使用 MD5/SHA1 存储密码（已不安全）
user.setPassword(DigestUtils.md5Hex(dto.getPassword()));

// ❌ 错误：明文存储身份证
user.setIdCard(dto.getIdCard());

// ❌ 错误：日志中打印敏感数据
log.info("用户注册 | password={}", password);

// ❌ 错误：URL 中传递敏感数据
// GET /api/user?idCard=110101199001011234
```

### 检查项

- [ ] 密码是否使用 BCrypt/Argon2 哈希
- [ ] 是否使用 MD5/SHA1 存储密码（应禁止）
- [ ] 身份证/银行卡是否加密存储
- [ ] 日志是否打印敏感数据
- [ ] URL 是否传递敏感参数

---

## 规则 4: 认证与授权

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

- 使用成熟的认证框架（Spring Security、Shiro）
- 授权检查在服务端，不依赖客户端
- 遵循最小权限原则
- 敏感操作需二次验证

### 正确示例

```java
// ✅ 正确：服务端权限校验
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    @DeleteMapping("/{userId}")
    @PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
    public ReturnData<Void> deleteUser(@PathVariable Long userId) {
        userService.delete(userId);
        return ReturnData.success();
    }
    
    @PutMapping("/{userId}")
    public ReturnData<Void> updateUser(@PathVariable Long userId, @RequestBody UserDTO dto) {
        // 服务端校验权限
        Long currentUserId = SecurityContextHolder.getContext().getAuthentication().getPrincipal().getId();
        if (!userId.equals(currentUserId) && !isAdmin()) {
            throw new PermissionDeniedException("无权修改他人信息");
        }
        userService.update(userId, dto);
        return ReturnData.success();
    }
}

// ✅ 正确：接口权限配置
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
            .antMatchers("/api/public/**").permitAll()
            .antMatchers("/api/admin/**").hasRole("ADMIN")
            .antMatchers("/api/**").authenticated()
            .and()
            .sessionManagement()
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
    }
}
```

### 错误示例

```java
// ❌ 错误：仅前端隐藏按钮，后端无校验
@DeleteMapping("/{userId}")
public ReturnData<Void> deleteUser(@PathVariable Long userId) {
    // 没有权限校验！任何人都能删除
    userService.delete(userId);
    return ReturnData.success();
}

// ❌ 错误：信任客户端传递的角色
@PostMapping("/update")
public ReturnData<Void> update(@RequestParam String role, @RequestBody UserDTO dto) {
    if ("admin".equals(role)) {  // 客户端可伪造
        // 执行管理员操作
    }
}
```

### 检查项

- [ ] 敏感接口是否有权限校验
- [ ] 权限校验是否在服务端
- [ ] 是否使用 Spring Security/Shiro
- [ ] 是否存在越权访问风险
- [ ] 删除/修改操作是否校验数据归属

---

## 规则 5: 接口安全防护

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

| 防护类型 | 要求 |
|----------|------|
| 防 CSRF | 非 GET 请求校验 CSRF Token |
| 防重放 | 关键接口添加时间戳 + 签名 |
| 接口限流 | 核心接口限制请求频率 |
| 请求大小 | 限制请求体大小 |

### 正确示例

```java
// ✅ 正确：CSRF 防护配置
@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf()
            .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse())
            .ignoringAntMatchers("/api/public/**");  // 公开接口可忽略
    }
}

// ✅ 正确：接口限流
import com.google.common.util.concurrent.RateLimiter;

@RestController
public class LoginController {
    
    // 每秒最多5个请求
    private final RateLimiter rateLimiter = RateLimiter.create(5.0);
    
    @PostMapping("/api/login")
    public ReturnData<TokenVO> login(@RequestBody LoginDTO dto) {
        if (!rateLimiter.tryAcquire()) {
            throw new BusinessException("请求过于频繁，请稍后再试");
        }
        return ReturnData.success(authService.login(dto));
    }
}

// ✅ 正确：使用注解限流（Sentinel/Resilience4j）
@SentinelResource(value = "login", blockHandler = "handleBlock")
@PostMapping("/api/login")
public ReturnData<TokenVO> login(@RequestBody LoginDTO dto) {
    return ReturnData.success(authService.login(dto));
}

// ✅ 正确：防重放攻击
@PostMapping("/api/pay")
public ReturnData<Void> pay(@RequestBody PayRequest request) {
    // 校验时间戳（5分钟内有效）
    long now = System.currentTimeMillis();
    if (Math.abs(now - request.getTimestamp()) > 5 * 60 * 1000) {
        throw new BusinessException("请求已过期");
    }
    
    // 校验签名
    String expectedSign = SignUtil.sign(request, secretKey);
    if (!expectedSign.equals(request.getSign())) {
        throw new BusinessException("签名校验失败");
    }
    
    // 校验请求唯一性（防重放）
    String requestId = request.getRequestId();
    if (redisTemplate.hasKey("pay:" + requestId)) {
        throw new BusinessException("请勿重复提交");
    }
    redisTemplate.opsForValue().set("pay:" + requestId, "1", 5, TimeUnit.MINUTES);
    
    return payService.pay(request);
}
```

### 错误示例

```java
// ❌ 错误：未启用 CSRF 防护
http.csrf().disable();  // 除非是纯 API 服务

// ❌ 错误：关键接口无限流
@PostMapping("/api/login")
public ReturnData<TokenVO> login(@RequestBody LoginDTO dto) {
    // 无限流，可被暴力破解
    return ReturnData.success(authService.login(dto));
}

// ❌ 错误：支付接口无防重放
@PostMapping("/api/pay")
public ReturnData<Void> pay(@RequestBody PayRequest request) {
    // 可被重放攻击，导致重复支付
    return payService.pay(request);
}
```

### 检查项

- [ ] 是否启用 CSRF 防护（Web应用）
- [ ] 登录接口是否限流
- [ ] 支付/下单接口是否防重放
- [ ] 是否限制请求体大小
- [ ] 是否设置请求超时

---

## 规则 6: 文件上传安全

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 验证文件类型（MIME + 扩展名 + 文件头）
- 限制文件大小
- 存储在非 Web 目录或对象存储
- 随机重命名文件

### 正确示例

```java
// ✅ 正确：安全的文件上传
@RestController
public class FileController {
    
    /** 允许的文件类型 */
    private static final Set<String> ALLOWED_TYPES = Set.of(
        "image/jpeg", "image/png", "image/gif", "application/pdf"
    );
    
    /** 允许的扩展名 */
    private static final Set<String> ALLOWED_EXTENSIONS = Set.of(
        ".jpg", ".jpeg", ".png", ".gif", ".pdf"
    );
    
    /** 最大文件大小：10MB */
    private static final long MAX_FILE_SIZE = 10 * 1024 * 1024;
    
    @PostMapping("/api/upload")
    public ReturnData<String> upload(@RequestParam("file") MultipartFile file) {
        // 1. 校验文件大小
        if (file.getSize() > MAX_FILE_SIZE) {
            throw new BusinessException("文件大小不能超过10MB");
        }
        
        // 2. 校验 MIME 类型
        String contentType = file.getContentType();
        if (!ALLOWED_TYPES.contains(contentType)) {
            throw new BusinessException("不支持的文件类型");
        }
        
        // 3. 校验扩展名
        String originalName = file.getOriginalFilename();
        String extension = getExtension(originalName).toLowerCase();
        if (!ALLOWED_EXTENSIONS.contains(extension)) {
            throw new BusinessException("不支持的文件扩展名");
        }
        
        // 4. 校验文件头（防止伪造扩展名）
        if (!validateFileHeader(file, extension)) {
            throw new BusinessException("文件内容与扩展名不匹配");
        }
        
        // 5. 生成随机文件名
        String newFileName = UUID.randomUUID().toString() + extension;
        
        // 6. 存储到非 Web 目录
        String savePath = "/data/uploads/" + newFileName;
        file.transferTo(new File(savePath));
        
        return ReturnData.success(newFileName);
    }
    
    /**
     * 校验文件头
     */
    private boolean validateFileHeader(MultipartFile file, String extension) {
        try {
            byte[] header = new byte[8];
            file.getInputStream().read(header);
            
            // JPEG: FF D8 FF
            if (".jpg".equals(extension) || ".jpeg".equals(extension)) {
                return header[0] == (byte) 0xFF && header[1] == (byte) 0xD8;
            }
            // PNG: 89 50 4E 47
            if (".png".equals(extension)) {
                return header[0] == (byte) 0x89 && header[1] == (byte) 0x50;
            }
            // PDF: 25 50 44 46 (%PDF)
            if (".pdf".equals(extension)) {
                return header[0] == 0x25 && header[1] == 0x50;
            }
            return true;
        } catch (IOException e) {
            return false;
        }
    }
}
```

### 错误示例

```java
// ❌ 错误：未校验文件类型
@PostMapping("/api/upload")
public ReturnData<String> upload(@RequestParam("file") MultipartFile file) {
    String savePath = "/uploads/" + file.getOriginalFilename();  // 使用原文件名
    file.transferTo(new File(savePath));  // 未校验
    return ReturnData.success(savePath);
}

// ❌ 错误：仅校验扩展名
String extension = getExtension(file.getOriginalFilename());
if (!".jpg".equals(extension)) {  // 攻击者可伪造扩展名
    throw new BusinessException("只支持JPG");
}

// ❌ 错误：存储在 Web 可访问目录
String savePath = "/webapp/uploads/" + fileName;  // 可直接访问执行
```

### 检查项

- [ ] 是否校验文件 MIME 类型
- [ ] 是否校验文件扩展名
- [ ] 是否校验文件头（魔数）
- [ ] 是否限制文件大小
- [ ] 是否使用随机文件名
- [ ] 是否存储在非 Web 目录

---

## 规则 7: 安全配置管理

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

- **禁止**在代码中硬编码密钥、密码、Token
- 使用环境变量或配置中心
- 不提交敏感配置到版本控制
- 生产环境关闭调试模式

### 正确示例

```java
// ✅ 正确：从配置文件读取
@Value("${jwt.secret}")
private String jwtSecret;

@Value("${database.password}")
private String dbPassword;

// ✅ 正确：application.yml（不提交敏感值）
spring:
  datasource:
    password: ${DB_PASSWORD}  # 从环境变量读取

jwt:
  secret: ${JWT_SECRET}

// ✅ 正确：.gitignore
application-local.yml
application-prod.yml
*.key
*.pem

// ✅ 正确：生产环境配置
spring:
  profiles:
    active: prod
  
# 生产环境关闭调试
logging:
  level:
    root: INFO
    
# 关闭 Swagger
springfox:
  documentation:
    enabled: false
```

### 错误示例

```java
// ❌ 错误：硬编码密钥
private static final String JWT_SECRET = "my-secret-key-12345";
private static final String DB_PASSWORD = "root123456";
private static final String API_KEY = "sk-abcdef123456";

// ❌ 错误：提交敏感配置到 Git
// application-prod.yml（已提交）
spring:
  datasource:
    password: realPassword123  # 泄露！

// ❌ 错误：生产环境开启调试
spring:
  profiles:
    active: prod
logging:
  level:
    root: DEBUG  # 生产环境不应开启
```

### 检查项

- [ ] 是否存在硬编码的密钥/密码
- [ ] 敏感配置是否使用环境变量
- [ ] .gitignore 是否排除敏感文件
- [ ] 生产环境是否关闭 DEBUG
- [ ] 生产环境是否关闭 Swagger

---

## 规则 8: 会话管理安全

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 登录后重新生成 Session ID
- 设置会话超时
- Cookie 设置 HttpOnly、Secure、SameSite
- 注销时销毁会话

### 正确示例

```java
// ✅ 正确：安全的 Session 配置
@Configuration
public class SessionConfig extends WebSecurityConfigurerAdapter {
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.sessionManagement()
            .sessionFixation().newSession()  // 登录后生成新Session
            .maximumSessions(1)              // 单点登录
            .expiredUrl("/login?expired");
    }
}

// ✅ 正确：Cookie 安全配置
@Bean
public CookieSerializer cookieSerializer() {
    DefaultCookieSerializer serializer = new DefaultCookieSerializer();
    serializer.setCookieName("SESSIONID");
    serializer.setUseHttpOnlyCookie(true);   // 防 XSS 读取
    serializer.setUseSecureCookie(true);     // 仅 HTTPS
    serializer.setSameSite("Strict");        // 防 CSRF
    serializer.setCookieMaxAge(30 * 60);     // 30分钟
    return serializer;
}

// ✅ 正确：安全注销
@PostMapping("/api/logout")
public ReturnData<Void> logout(HttpServletRequest request) {
    HttpSession session = request.getSession(false);
    if (session != null) {
        session.invalidate();  // 销毁会话
    }
    SecurityContextHolder.clearContext();
    return ReturnData.success();
}
```

### 检查项

- [ ] 登录后是否重新生成 Session ID
- [ ] 是否设置会话超时时间
- [ ] Cookie 是否设置 HttpOnly
- [ ] Cookie 是否设置 Secure（HTTPS）
- [ ] 注销是否销毁会话

---

## 输出报告格式

检查完成后，请按以下格式输出报告：

```markdown
# Java 安全规范检查报告

## 检查概览
- 检查路径：{path}
- 检查文件数：{count}
- 通过项：{pass_count}
- 警告项：{warn_count}
- 不通过项：{fail_count}

## 详细结果

### 1. SQL 注入防护
| 文件 | 检查项 | 状态 | 说明 |
|------|--------|------|------|
| UserMapper.xml | 参数化查询 | PASS | 使用 #{} |
| OrderDao.java | SQL拼接 | FAIL | 存在字符串拼接 |

### 2. 敏感数据保护
| 文件 | 检查项 | 状态 | 说明 |
|------|--------|------|------|
| UserService | 密码哈希 | PASS | 使用 BCrypt |
| ConfigFile | 硬编码密钥 | FAIL | 发现硬编码 API_KEY |

## 安全风险等级
- 高危：{high_count} 个
- 中危：{medium_count} 个
- 低危：{low_count} 个

## 修复建议
1. [高危] OrderDao.java 存在 SQL 注入风险，请改用参数化查询
2. [高危] 发现硬编码密钥 API_KEY，请移至配置文件
```

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-03-12 | 初始版本，包含 8 条安全规范规则，适配 Java/Spring Boot |
