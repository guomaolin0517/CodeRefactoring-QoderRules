---
name: java-rule
description: "java代码规范规则"
---
# Java 统一代码规范（Rule v2.0.0）

## 版本核心信息（供 Qoder Skill 解析校验）

- 规则版本：2.0.0
- 适配 Skill 版本：java-code-check >= v2.0.0
- 生效时间：2026-03-18
- 变更类型：主版本（融合空指针安全、日志规范、安全漏洞防护、异常处理、命名规范、Spring 最佳实践六大专项检查规则）
- 技术栈适配：Spring Boot 2.7+、Spring Cloud、MyBatis 3.5+、SLF4J 1.7+
- 基准规范：《阿里巴巴 Java 开发手册（黄山版）》+ OWASP Top 10

- 违规等级定义：
    - **严重（FAIL）**：直接导致生产故障或安全漏洞（如 SQL 注入、空指针、反序列化漏洞），Skill 强制阻断提交；
    - **高危（WARN）**：可能引发性能/安全问题（如日志脱敏缺失、事务配置不当），Skill 提示并记录；
    - **规范（INFO）**：格式/命名问题（如小驼峰错误），Skill 自动修复。

## 规则描述

适配《阿里巴巴 Java 开发手册（黄山版）》+ 生产环境落地要求 + OWASP Top 10 安全规范，约束 Java 代码的命名、注释、日志、异常、集合、SQL、空值处理、安全防护、Spring 框架使用等维度，核心目标：

1. 规避生产环境常见故障（空指针、SQL 注入、日志泄露、反序列化漏洞）；
2. 统一团队代码风格，降低维护成本；
3. 支持 Qoder Skill 自动校验/修复，提升研发效率；
4. 符合代码评审标准，无需人工逐条检查；
5. 覆盖 OWASP Top 10 中与 Java 后端相关的核心安全风险。

---

## 核心校验规则

### 1. 命名规范（规范级）

#### 1.1 通用规则

校验标准：禁止使用拼音/拼音缩写/无意义命名（如 `yonghuService` / `yhService` / `a1`）；

修复方式：Skill 给出语义化命名建议，人工确认后修改。

#### 1.2 类名规范（大驼峰 UpperCamelCase）

| 判定标准 | 状态 |
|---------|------|
| 类名不以大写字母开头 | **FAIL** |
| 类名包含下划线或连字符 | **FAIL** |
| 类名使用不规范缩写（如 `Ctrl` -> `Controller`、`Svc` -> `Service`、`Mgr` -> `Manager`） | **WARN** |
| 抽象类未以 `Abstract` 或 `Base` 开头 | **WARN** |
| 测试类未以 `Test` 结尾 | **WARN** |
| 工具类未以 `Utils` 或 `Util` 结尾 | **WARN** |

#### 1.3 方法名规范（小驼峰 lowerCamelCase）

| 判定标准 | 状态 |
|---------|------|
| 方法名以大写字母开头 | **FAIL** |
| 方法名包含下划线（测试方法除外） | **FAIL** |
| 获取方法未用 `get` 前缀 | **WARN** |
| 设置方法未用 `set` 前缀 | **WARN** |
| 判断方法未用 `is`/`has`/`can`/`should` 前缀 | **WARN** |
| 方法名超过 40 字符 | **WARN** |

#### 1.4 变量名规范（小驼峰 lowerCamelCase）

| 判定标准 | 状态 |
|---------|------|
| 字段名以大写字母开头 | **FAIL** |
| 字段名使用下划线分隔（Java Bean 属性，非数据库映射字段） | **WARN** |
| 存在单字符变量名（循环变量 `i/j/k` 及 lambda 参数除外） | **WARN** |
| 布尔类型字段未以 `is`/`has`/`can` 开头 | **WARN** |
| 集合类型变量未使用复数或 `List`/`Map`/`Set` 后缀 | **WARN** |

#### 1.5 常量名规范（全大写 UPPER_SNAKE_CASE）

| 判定标准 | 状态 |
|---------|------|
| `static final` 字段未全大写下划线分隔 | **FAIL** |
| 枚举值未全大写下划线分隔 | **FAIL** |
| 硬编码魔法值未提取为常量 | **WARN** |

#### 1.6 包名规范（全小写 lowercase）

| 判定标准 | 状态 |
|---------|------|
| 包名包含大写字母 | **FAIL** |
| 包名包含下划线或连字符 | **WARN** |
| 包名使用数字开头的段 | **WARN** |

#### 1.7 分层命名后缀一致性

| 分层 | 标准后缀 |
|------|---------|
| 控制层 | `Controller` |
| 服务接口 | `Service` |
| 服务实现 | `ServiceImpl` |
| 数据层 | `Dao` / `Mapper` |
| 实体 | 无特定后缀或 `Entity` |
| DTO | `DTO` |
| VO | `VO` / `Vo` |
| 查询对象 | `Query` / `Param` |

判定标准：类不在对应层级或无正确后缀 -> **WARN**

#### 1.8 接口命名

校验标准：接口名应使用前缀 `I` 或语义化命名（如 `IUserService` / `UserApi`）；

判定标准：接口与实现类命名不一致或无法区分 -> **WARN**（提示修复，不自动改）。

---

### 2. 注释规范（规范级）

#### 2.1 类注释（强制）

校验标准：类顶部必须包含 Javadoc 注释，含 `@author` / `@date` / `@description`；

修复方式：Skill 自动补全注释模板，人工补充业务描述；

正确示例：

```java
/**
 * 用户业务层服务类
 * @author gml
 * @date 2026-03-18
 * @description 处理用户登录、查询、修改等核心业务逻辑
 */
public class UserService { ... }
```

#### 2.2 方法注释（强制）

校验标准：公共方法必须包含 `@param` / `@return` / `@throws`（有异常时），参数需标注"必传/可选"；

修复方式：Skill 自动解析方法参数/返回值/异常，生成注释模板；

正确示例：

```java
/**
 * 根据用户ID查询用户信息
 * @param userId 用户ID（必传，>=1）
 * @return com.company.entity.User 用户实体
 * @throws BusinessException 1001-用户不存在，1002-参数非法
 */
public User getUserById(Long userId) throws BusinessException { ... }
```

#### 2.3 行注释（推荐）

校验标准：行注释需在代码后空 2 格，`//` 后空 1 格，禁止无意义注释（如 `// 循环`）；

修复方式：Skill 自动调整空格，提示删除无意义注释。

---

### 3. 日志规范（高危级）

#### 3.1 日志框架统一（强制）

| 判定标准 | 状态 |
|---------|------|
| 使用 `System.out.println` / `System.err.println` | **FAIL** |
| 直接使用 `java.util.logging.Logger` | **WARN**（应统一使用 SLF4J） |
| 直接使用 `org.apache.log4j.Logger` | **WARN**（应使用 SLF4J 门面） |
| 使用 `org.slf4j.Logger` + `LoggerFactory` 或 `@Slf4j` | **PASS** |

修复方式：Skill 自动替换 `System.out.println` 为 `logger.info`，`e.printStackTrace()` 为 `log.error("描述", e)`。

#### 3.2 Logger 声明规范（强制）

| 判定标准 | 状态 |
|---------|------|
| Logger 未声明为 `private static final` | **WARN** |
| Logger 变量名不是 `log` 或 `logger` | **WARN** |
| `LoggerFactory.getLogger()` 参数不是当前类 | **FAIL** |
| 使用 Lombok `@Slf4j` | **PASS** |

正确示例：

```java
private static final Logger logger = LoggerFactory.getLogger(UserService.class);
logger.info("用户登录成功，userId: {}", userId);
```

#### 3.3 日志级别使用规范（强制）

| 级别 | 适用场景 |
|------|---------|
| ERROR | 系统异常、严重错误、需要人工介入 |
| WARN | 可恢复的异常、潜在问题、业务规则违反 |
| INFO | 重要业务流程节点、系统启停、配置加载 |
| DEBUG | 开发调试信息、方法入参出参 |
| TRACE | 极细粒度追踪信息 |

| 判定标准 | 状态 |
|---------|------|
| 正常业务逻辑中使用 `log.error()` | **WARN**（应为 INFO 或 DEBUG） |
| 异常处理中使用 `log.info()` | **WARN**（应为 ERROR 或 WARN） |
| 循环体内使用 `log.info()` 或以上级别 | **WARN**（可能产生大量日志） |
| 生产代码中大量 `log.debug()` 无条件输出 | **WARN** |

#### 3.4 日志参数使用占位符（强制）

| 判定标准 | 状态 |
|---------|------|
| `log.info("user=" + user + ", id=" + id)` 字符串拼接 | **FAIL**（应使用 `log.info("user={}, id={}", user, id)`） |
| `log.debug("data: " + JSON.toJSONString(obj))` 无条件序列化 | **FAIL**（应先 `isDebugEnabled()` 或使用占位符） |

#### 3.5 异常日志规范（强制）

| 判定标准 | 状态 |
|---------|------|
| `log.error(e.getMessage())` 未传入异常对象 | **FAIL**（应 `log.error("描述", e)`） |
| `log.error("error", e.getMessage())` 将 message 当参数 | **FAIL**（应 `log.error("error", e)`） |
| `e.printStackTrace()` | **FAIL**（应替换为 `log.error("描述", e)`） |

#### 3.6 日志脱敏（强制）

校验标准：禁止打印敏感信息（手机号/身份证/密码/银行卡号/token），必须脱敏；

敏感关键字扫描：`password`、`token`、`secret`、`key`、`credential`、`idCard`、`phone`

| 判定标准 | 状态 |
|---------|------|
| 日志中输出密码、token、密钥等 | **FAIL** |
| 日志中输出完整手机号、身份证号、银行卡号 | **WARN**（应脱敏） |
| 日志中输出完整 SQL（含参数值）可能泄露业务数据 | **WARN** |

修复方式：Skill 自动识别敏感字段，生成脱敏代码；

正确示例：

```java
String maskedPhone = user.getPhone().replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");
logger.info("用户手机号：{}", maskedPhone);
```

---

### 4. 异常处理规范（严重级）

#### 4.1 禁止吞掉异常（强制）

| 判定标准 | 状态 |
|---------|------|
| catch 块完全为空 | **FAIL** |
| catch 块仅有 `e.printStackTrace()` | **FAIL**（应替换为日志框架） |
| catch 块仅有注释无实际处理 | **FAIL** |

#### 4.2 禁止笼统捕获 Exception/Throwable（强制）

| 判定标准 | 状态 |
|---------|------|
| 业务代码中 `catch(Exception e)` | **WARN**（应捕获具体异常） |
| `catch(Throwable t)` | **FAIL**（极少数场景才需要） |
| 全局异常处理器中 `catch(Exception e)` | **PASS**（允许兜底） |

#### 4.3 finally 块禁止 return（强制）

校验标准：finally 块中包含 return 语句，会覆盖 try/catch 中的返回值和异常；

判定标准：finally 块中存在 `return` -> **FAIL**

#### 4.4 异常信息完整性（强制）

| 判定标准 | 状态 |
|---------|------|
| `throw new XxxException(e.getMessage())` 未传入原始异常 | **FAIL**（应 `new XxxException(msg, e)`） |
| `log.error("xxx")` 未传入异常对象 | **WARN**（应 `log.error("xxx", e)`） |
| 重新抛出异常时丢弃 cause | **FAIL** |

#### 4.5 自定义异常规范（强制）

| 判定标准 | 状态 |
|---------|------|
| 自定义异常未继承 `RuntimeException` 或 `Exception` | **FAIL** |
| 自定义异常类名未以 `Exception` 结尾 | **WARN** |
| 自定义异常无带 message 参数的构造方法 | **WARN** |
| 自定义异常无带 cause 参数的构造方法 | **WARN** |
| 使用 `RuntimeException` 传递业务信息（应自定义 `BusinessException` 含错误码+错误信息） | **FAIL** |

#### 4.6 Controller 层统一异常处理（强制）

| 判定标准 | 状态 |
|---------|------|
| 缺少 `@ControllerAdvice` 或 `@RestControllerAdvice` 全局异常处理器 | **WARN** |
| Controller 方法内部 try-catch 并手动构建错误响应 | **WARN**（应由全局处理器统一处理） |
| 全局异常处理器未覆盖常见异常（`MethodArgumentNotValidException`、`BindException`） | **WARN** |

#### 4.7 事务中的异常处理（强制）

| 判定标准 | 状态 |
|---------|------|
| `@Transactional` 方法中 catch 异常后未重新抛出（导致事务不回滚） | **FAIL** |
| `@Transactional` 未指定 `rollbackFor = Exception.class` | **WARN** |

---

### 5. 空指针安全（严重级）

#### 5.1 方法返回值未判空直接链式调用

| 判定标准 | 状态 |
|---------|------|
| `obj.getXxx().getYyy()` 链式调用无中间判空 | **WARN** |
| `map.get(key).toString()` | **FAIL** |
| `list.get(index).getXxx()` | **WARN** |
| `repository.findById(id).getXxx()` | **FAIL** |

#### 5.2 集合操作未判空

| 判定标准 | 状态 |
|---------|------|
| 对可能为 null 的集合直接 `for-each` 遍历 | **FAIL** |
| 对可能为 null 的集合调用 `.size()` / `.isEmpty()` | **WARN** |
| 方法返回集合时返回 null 而非空集合 | **FAIL**（应返回 `Collections.emptyList()`） |

#### 5.3 equals 调用安全

| 判定标准 | 状态 |
|---------|------|
| 变量可能为 null 时 `variable.equals("constant")` | **WARN**（应 `"constant".equals(variable)`） |
| 两个可能为 null 的对象使用 `==` 比较（非基本类型） | **WARN**（应使用 `Objects.equals()`） |

#### 5.4 自动拆箱 NPE 风险

| 判定标准 | 状态 |
|---------|------|
| `Integer`/`Long`/`Boolean` 等包装类型直接参与算术运算，未判空 | **FAIL** |
| 方法返回包装类型，调用方以基本类型接收且未判空 | **WARN** |
| 三目表达式中混用包装类型和基本类型 | **WARN** |

#### 5.5 String 操作安全

| 判定标准 | 状态 |
|---------|------|
| 对可能为 null 的 String 直接调用 `.length()` / `.trim()` / `.substring()` | **WARN** |
| 未使用 `StringUtils.isBlank()` / `StringUtils.isEmpty()` 进行判空 | **WARN** |
| 字符串拼接中变量可能为 null | **WARN** |

#### 5.6 外部数据源未判空

| 判定标准 | 状态 |
|---------|------|
| `@RequestParam` 参数无 `required=false` 且无默认值，但代码中未判空 | **WARN** |
| 数据库查询结果（`selectOne`/`selectById`）未判空直接使用 | **FAIL** |
| Feign 调用返回值未判空 | **FAIL** |
| `HttpServletRequest.getParameter()` 返回值未判空 | **WARN** |

#### 5.7 Optional 使用规范

| 判定标准 | 状态 |
|---------|------|
| `Optional.get()` 未先调用 `isPresent()` 检查 | **FAIL** |
| Optional 作为方法参数类型 | **WARN**（不推荐） |
| Optional 作为字段类型 | **WARN**（不推荐） |
| 应使用 `orElse()` / `orElseGet()` / `orElseThrow()` 替代 `isPresent()+get()` | **WARN** |

#### 5.8 空值判断方式（强制）

校验标准：禁止 `obj == null`，优先使用 `Objects.isNull(obj)` / `Objects.nonNull(obj)`；

修复方式：Skill 自动替换。

正确示例：

```java
if (Objects.isNull(user)) {
    throw new BusinessException(1001, "用户不存在");
}
```

---

### 6. 集合使用规范（规范级）

#### 6.1 集合初始化（强制）

校验标准：集合初始化时必须指定初始容量（如 `new ArrayList<>(10)`），避免扩容性能损耗；

修复方式：Skill 自动补充初始容量（默认按预估大小）。

#### 6.2 集合判空（强制）

校验标准：判空必须使用 `CollectionUtils.isEmpty()` / `CollectionUtils.isNotEmpty()`（避免 NPE），禁止 `list == null || list.size() == 0`；

修复方式：Skill 自动替换为工具类方法。

#### 6.3 遍历方式（推荐）

校验标准：遍历集合优先使用增强 for / stream，禁止使用普通 for + 下标（除非需要索引）；

修复方式：Skill 提示优化遍历方式，人工调整。

---

### 7. SQL 规范（严重级）

#### 7.1 SQL 防注入（强制）

| 判定标准 | 状态 |
|---------|------|
| MyBatis XML 中使用 `${}` 替代 `#{}` | **FAIL**（`${}` 不做预编译） |
| 代码中使用 `"SELECT ... WHERE id = " + id` 拼接 SQL | **FAIL** |
| 使用 `Statement` 而非 `PreparedStatement` | **FAIL** |
| MyBatis `<if>` 条件中拼接用户输入 | **WARN** |

安全写法：
- MyBatis 使用 `#{param}` 占位符
- JDBC 使用 `PreparedStatement` + `setXxx()`
- 动态 SQL 使用 `<where>` `<foreach>` 标签

违规示例（禁止）：

```java
String sql = "SELECT * FROM user WHERE id = " + userId;  // 注入风险
```

正确示例：

```xml
<!-- MyBatis映射文件 -->
<select id="getUserById" resultType="com.company.entity.User">
    SELECT * FROM user WHERE id = #{userId}
</select>
```

#### 7.2 SQL 命名（规范级）

校验标准：表名小写+下划线，前缀区分模块（如 `t_user` / `t_order`）；字段名小写+下划线（如 `user_id` / `order_no`）；

修复方式：Skill 提示命名不规范，人工调整。

---

### 8. 安全漏洞防护（严重级）

#### 8.1 XSS 防护

| 判定标准 | 状态 |
|---------|------|
| Controller 直接将用户输入原样返回页面 | **WARN** |
| 未对用户输入进行 HTML 转义处理 | **WARN** |
| `@ResponseBody` 返回含有用户输入的 HTML 内容 | **WARN** |
| 使用 `HttpServletResponse.getWriter().write()` 输出用户数据 | **FAIL** |

#### 8.2 路径遍历防护

| 判定标准 | 状态 |
|---------|------|
| 文件操作路径直接使用用户输入 `new File(userInput)` | **FAIL** |
| 未对文件路径做 `../` 过滤 | **FAIL** |
| 文件下载接口未校验路径范围 | **FAIL** |

#### 8.3 硬编码敏感信息

| 判定标准 | 状态 |
|---------|------|
| 代码中硬编码密码 `password = "xxx"` | **FAIL** |
| 代码中硬编码 API Key / Token | **FAIL** |
| 代码中硬编码数据库连接串 | **FAIL** |
| 配置文件中明文存储密码（无加密方案） | **WARN** |

扫描关键字：`password`、`passwd`、`secret`、`apiKey`、`token`、`accessKey`、`jdbc:` 后跟明文密码

#### 8.4 不安全的反序列化

| 判定标准 | 状态 |
|---------|------|
| 使用 `ObjectInputStream.readObject()` 反序列化不可信数据 | **FAIL** |
| 使用 `XMLDecoder` 解析不可信 XML | **FAIL** |
| 使用 `JSON.parseObject(input, Feature.SupportNonPublicField)` | **WARN** |
| Fastjson 未关闭 `autoType` | **FAIL** |

#### 8.5 接口权限控制

| 判定标准 | 状态 |
|---------|------|
| 管理类接口缺少权限注解（`@PreAuthorize` / `@RequiresPermissions` 或自定义注解） | **WARN** |
| 数据操作接口（增删改）无用户身份校验 | **WARN** |
| 批量操作接口无数量限制 | **WARN** |

#### 8.6 敏感数据暴露

| 判定标准 | 状态 |
|---------|------|
| 接口返回中包含密码字段（即使为 null） | **FAIL** |
| 错误响应中返回堆栈信息（`stackTrace`） | **FAIL** |
| 接口返回中暴露内部 ID、数据库表结构等 | **WARN** |
| 日志中记录完整请求/响应体含敏感字段 | **WARN** |

#### 8.7 文件上传安全

| 判定标准 | 状态 |
|---------|------|
| 未校验上传文件类型（MIME / 后缀） | **FAIL** |
| 未限制上传文件大小 | **WARN** |
| 上传后文件名直接使用原始文件名 | **FAIL**（应重命名） |
| 上传目录位于 Web 可访问路径 | **WARN** |

---

### 9. Spring Boot/Cloud 最佳实践（高危级）

#### 9.1 依赖注入方式

推荐方式：构造器注入（Spring 官方推荐）

| 判定标准 | 状态 |
|---------|------|
| 使用 `@Autowired` 字段注入（非测试类） | **WARN**（建议构造器注入） |
| `@Autowired(required=false)` 但代码中未判空 | **FAIL** |
| 同一个类中 `@Autowired` 字段超过 7 个 | **WARN**（类职责可能过重） |
| 使用 `@Resource` 按名称注入但名称与 Bean 不匹配 | **WARN** |

#### 9.2 @Transactional 使用规范

| 判定标准 | 状态 |
|---------|------|
| `@Transactional` 加在 `private` 方法上 | **FAIL**（事务不生效） |
| `@Transactional` 方法被同类其他方法直接调用 | **FAIL**（事务不生效，需通过代理） |
| `@Transactional` 未指定 `rollbackFor` | **WARN**（默认只回滚 RuntimeException） |
| 只读查询方法使用 `@Transactional` 且无 `readOnly=true` | **WARN** |
| `@Transactional` 方法中包含 RPC/HTTP 外部调用 | **WARN**（可能长事务） |

#### 9.3 Controller 层规范

| 判定标准 | 状态 |
|---------|------|
| Controller 中包含业务逻辑代码（超过 15 行非调用代码） | **WARN**（应下沉到 Service） |
| Controller 方法参数超过 5 个且未封装为对象 | **WARN** |
| Controller 未使用 `@RestController` 或 `@Controller` 注解 | **FAIL** |
| `@RequestMapping` 未指定 HTTP 方法 | **WARN** |

#### 9.4 配置管理规范

| 判定标准 | 状态 |
|---------|------|
| 硬编码配置值未外部化到 `application.yml` | **WARN** |
| 使用 `@Value` 注入静态字段 | **FAIL**（不生效） |
| 配置类未使用 `@ConfigurationProperties` 而是大量 `@Value` | **WARN** |
| 多环境配置缺少 `application-{profile}.yml` | **WARN** |

#### 9.5 Bean 定义规范

| 判定标准 | 状态 |
|---------|------|
| `@Component` / `@Service` 等注解加在接口上而非实现类 | **FAIL** |
| 存在多个同类型 Bean 未使用 `@Qualifier` 或 `@Primary` 区分 | **WARN** |
| `@Configuration` 类中 `@Bean` 方法间直接调用（非代理模式） | **WARN** |
| `@Scope("prototype")` 的 Bean 被 Singleton Bean 注入 | **FAIL**（每次获取到同一个实例） |

#### 9.6 异步与线程池

| 判定标准 | 状态 |
|---------|------|
| 使用 `new Thread()` 手动创建线程 | **FAIL**（应使用线程池） |
| `@Async` 方法在同类中被调用 | **FAIL**（不生效） |
| `@Async` 未配置自定义线程池 | **WARN**（使用默认 SimpleAsyncTaskExecutor 不复用线程） |
| `CompletableFuture` 未指定 Executor | **WARN** |

#### 9.7 Feign 客户端规范（Spring Cloud）

| 判定标准 | 状态 |
|---------|------|
| Feign 接口无 `fallback` 或 `fallbackFactory` | **WARN**（无降级策略） |
| Feign 接口方法签名与服务端不一致 | **FAIL** |
| Feign 请求超时未配置 | **WARN** |

#### 9.8 循环依赖检测

| 判定标准 | 状态 |
|---------|------|
| 两个 Bean 互相注入（A->B、B->A） | **WARN** |
| `@Lazy` 用于解决循环依赖而非重构 | **WARN** |
| 出现三个及以上 Bean 的循环引用链 | **FAIL** |

---

### 10. 其他核心规范（生产级）

#### 10.1 魔法值（强制）

校验标准：禁止代码中出现魔法值（如 `if (status == 1)`），需定义常量；

修复方式：Skill 识别魔法值，提示定义常量（如 `public static final Integer STATUS_ENABLE = 1;`）。

#### 10.2 static 使用（禁止）

校验标准：禁止在业务类中定义 static 成员变量（除非常量），避免线程安全问题；

修复方式：Skill 提示 static 变量风险，人工调整。

---

## 校验结果输出规范（供 Qoder Skill 参考）

### 输出原则

1. **严重违规（FAIL）**：列出问题+修复示例，Skill 强制阻断代码提交（如 SQL 拼接、捕获大异常、反序列化漏洞）；
2. **高危违规（WARN）**：列出问题+自动修复代码，需人工确认（如日志脱敏缺失、事务配置不当）；
3. **规范违规（INFO）**：Skill 自动修复，生成修复报告（如命名错误、空行不规范）；
4. **输出格式**：按模块分类（命名/注释/日志/异常/空指针/安全/Spring），标注违规行数+修复方式。

### 报告模板

```
# Java 统一代码规范检查报告

## 检查概览
- 检查路径：{path}
- 检查文件数：{count}
- FAIL（严重）：{fail_count}
- WARN（高危）：{warn_count}
- INFO（规范）：{info_count}

## 详细结果

### 命名规范
| 文件 | 行号 | 名称 | 问题描述 | 状态 |
|------|------|------|---------|------|

### 日志规范
| 文件 | 行号 | 代码片段 | 问题描述 | 状态 |
|------|------|---------|---------|------|

### 异常处理
| 文件 | 行号 | 问题描述 | 状态 |
|------|------|---------|------|

### 空指针安全
| 文件 | 行号 | 风险类型 | 代码片段 | 状态 |
|------|------|---------|---------|------|

### SQL 规范
| 文件 | 行号 | 风险代码 | 状态 |
|------|------|---------|------|

### 安全漏洞
| 文件 | 行号 | 漏洞类型 | 风险代码 | 状态 |
|------|------|---------|---------|------|

### Spring 最佳实践
| 文件 | 行号 | 问题描述 | 状态 |
|------|------|---------|------|

## 修复建议
1. [FAIL] {具体问题及修复方案}
2. [WARN] {具体问题及修复方案}
3. [INFO] {具体问题及自动修复说明}
```

---

## 版本变更记录（按倒序）

- v2.0.0：融合六大专项检查规则（空指针安全、日志规范、安全漏洞防护、异常处理、命名规范、Spring 最佳实践），新增 OWASP Top 10 安全防护规则（XSS/路径遍历/硬编码/反序列化/文件上传）、Spring Boot/Cloud 最佳实践规则（依赖注入/事务/@Async/Feign/循环依赖）、扩展命名规范（分层后缀/布尔属性/集合变量）、扩展异常处理（吞异常/finally return/异常链/事务异常/统一异常处理器）、扩展日志规范（Logger 声明/占位符/异常日志）、扩展空指针安全（链式调用/拆箱/Optional/外部数据源）；统一违规等级为 FAIL/WARN/INFO 三级体系。
- v1.0.0：基于阿里 Java 手册黄山版重构，新增生产级日志脱敏、SQL 防注入、空指针防护规则，适配 Spring Boot 2.7+；集合初始化、魔法值、static 使用规则，优化注释模板；包含命名、注释、日志基础规则。
