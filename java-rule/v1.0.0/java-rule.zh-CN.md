# Java代码规范（Rule v1.0.0）

## 版本核心信息（供Qoder Skill解析校验）

- 规则版本：1.0.0

- 适配Skill版本：java-code-check ≥ v1.0.0

- 生效时间：2026-03-17

- 变更类型：主版本（基于阿里Java手册黄山版，覆盖生产级约束）

- 技术栈适配：Spring Boot 2.7+、MyBatis 3.5+、SLF4J 1.7+

- 违规等级定义：
        

    - 严重：直接导致生产故障（如SQL注入、空指针），Skill强制阻断提交；

    - 高危：可能引发性能/安全问题（如日志脱敏缺失），Skill提示并记录；

    - 规范：格式/命名问题（如小驼峰错误），Skill自动修复。

## 规则描述（description）

适配《阿里巴巴Java开发手册（黄山版）》+ 生产环境落地要求，约束Java代码的命名、注释、日志、异常、集合、SQL、空值处理等维度，核心目标：

1. 规避生产环境常见故障（空指针、SQL注入、日志泄露）；

2. 统一团队代码风格，降低维护成本；

3. 支持Qoder Skill自动校验/修复，提升研发效率；

4. 符合代码评审标准，无需人工逐条检查。

## 核心校验规则（产品研发落地级，含校验标准+修复方式+示例）

### 1. 命名规范（规范级，Skill可自动修复/提示）

#### 1.1 通用规则

校验标准：禁止使用拼音/拼音缩写/无意义命名（如yonghuService/yhService/a1）；

修复方式：Skill给出语义化命名建议，人工确认后修改。

#### 1.2 细分命名规则

|类型|命名格式|违规示例|正确示例|Skill修复能力|
|---|---|---|---|---|
|类名|大驼峰（CamelCase）|userService/User_service|UserService|自动修复（首字母大写+去下划线）|
|方法/变量|小驼峰（camelCase）|GetUserInfo/USER_ID|getUserInfo/userId|自动修复（首字母小写+去下划线）|
|常量|全大写+下划线|maxRetryTimes/Max_Retry|MAX_RETRY_TIMES|自动修复（全大写+加下划线）|
|包名|全小写+点分隔|com.Company.User|com.company.user|自动修复（全小写）|
|接口名|前缀I/语义化|UserService（接口）|IUserService/UserApi|提示修复（不自动改，避免语义错误）|
### 2. 注释规范（规范级，Skill可自动补全/校验）

#### 2.1 类注释（强制）

校验标准：类顶部必须包含Javadoc注释，含@author/@date/@description；

修复方式：Skill自动补全注释模板，人工补充业务描述；

正确示例：

```java
/**
 * 用户业务层服务类
 * @author gml
 * @date 2026-03-17
 * @description 处理用户登录、查询、修改等核心业务逻辑
 */
public class UserService { ... }
```

#### 2.2 方法注释（强制）

校验标准：公共方法必须包含@param/@return/@throws（有异常时），参数需标注“必传/可选”；

修复方式：Skill自动解析方法参数/返回值/异常，生成注释模板；

正确示例：

```java
/**
 * 根据用户ID查询用户信息
 * @param userId 用户ID（必传，≥1）
 * @return com.company.entity.User 用户实体
 * @throws BusinessException 1001-用户不存在，1002-参数非法
 */
public User getUserById(Long userId) throws BusinessException { ... }
```

#### 2.3 行注释（推荐）

校验标准：行注释需在代码后空2格，//后空1格，禁止无意义注释（如// 循环）；

修复方式：Skill自动调整空格，提示删除无意义注释。

### 3. 日志规范（高危级，Skill可自动修复/提示）

#### 3.1 日志框架（强制）

校验标准：必须使用SLF4J + Logback，禁止System.out.println/e.printStackTrace()；

修复方式：Skill自动替换System.out.println为logger.info，e.printStackTrace()为logger.error并补全堆栈；

正确示例：

```java
private static final Logger logger = LoggerFactory.getLogger(UserService.class);
logger.info("用户登录成功，userId: {}", userId); // 禁止直接拼接字符串
```

#### 3.2 日志级别（强制）

校验标准：

- DEBUG：调试信息（仅开发/测试环境，禁止生产开启）；

- INFO：核心业务流程（如“用户下单成功，orderId: 123”）；

- WARN：非致命异常（如“参数可选但为空，使用默认值”）；

- ERROR：致命异常（必须记录堆栈，如“数据库连接失败”）；

修复方式：Skill提示级别使用错误，人工调整。

#### 3.3 日志脱敏（强制）

校验标准：禁止打印敏感信息（手机号/身份证/密码/银行卡号），必须脱敏；

修复方式：Skill自动识别敏感字段，生成脱敏代码（如手机号138****1234）；

正确示例：

```java
// 脱敏前（违规）：logger.info("用户手机号：{}", user.getPhone());
// 脱敏后（正确）：
String maskedPhone = user.getPhone().replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2");
logger.info("用户手机号：{}", maskedPhone);
```

### 4. 异常处理（严重级，Skill可提示修复）

#### 4.1 异常捕获（强制）

校验标准：禁止捕获Exception/Throwable大异常，需捕获具体异常（如NullPointerException/SQLException）；

修复方式：Skill提示捕获大异常，建议拆分具体异常。

#### 4.2 自定义异常（强制）

校验标准：业务异常需自定义BusinessException，包含错误码+错误信息，禁止使用RuntimeException传递业务信息；

修复方式：Skill提示替换RuntimeException为自定义异常，生成异常模板。

#### 4.3 空指针防护（强制）

校验标准：对null可能的场景必须判空（如返回值/参数/集合），优先使用Objects.isNull()/Objects.nonNull()；

修复方式：Skill自动补充判空代码，避免空指针；

正确示例：

```java
if (Objects.isNull(user)) {
    throw new BusinessException(1001, "用户不存在");
}
```

### 5. 集合使用（规范级，Skill可自动修复）

#### 5.1 集合初始化（强制）

校验标准：集合初始化时必须指定初始容量（如new ArrayList<>(10)），避免扩容性能损耗；

修复方式：Skill自动补充初始容量（默认按预估大小）。

#### 5.2 集合判空（强制）

校验标准：判空必须使用CollectionUtils.isEmpty()/CollectionUtils.isNotEmpty()（避免NPE），禁止list == null || list.size() == 0；

修复方式：Skill自动替换为工具类方法。

#### 5.3 遍历方式（推荐）

校验标准：遍历集合优先使用增强for/stream，禁止使用普通for+下标（除非需要索引）；

修复方式：Skill提示优化遍历方式，人工调整。

### 6. SQL规范（严重级，Skill可强制阻断）

#### 6.1 SQL防注入（强制）

校验标准：禁止字符串拼接SQL，必须使用MyBatis #{}/JPA参数绑定；

修复方式：Skill识别拼接SQL，提示替换为参数绑定，阻断提交；

违规示例（禁止）：

```java
String sql = "SELECT * FROM user WHERE id = " + userId; // 注入风险
```

正确示例：

```xml
<!-- MyBatis映射文件 -->
<select id="getUserById" resultType="com.company.entity.User">
    SELECT * FROM user WHERE id = #{userId}
</select>
```

#### 6.2 SQL命名（规范级）

校验标准：表名小写+下划线，前缀区分模块（如t_user/t_order）；字段名小写+下划线（如user_id/order_no）；

修复方式：Skill提示命名不规范，人工调整。

### 7. 其他核心规范（生产级）

#### 7.1 空值判断（强制）

校验标准：禁止obj == null，优先使用Objects.isNull(obj)；

修复方式：Skill自动替换为Objects.isNull()/Objects.nonNull()。

#### 7.2 魔法值（强制）

校验标准：禁止代码中出现魔法值（如if (status == 1)），需定义常量；

修复方式：Skill识别魔法值，提示定义常量（如public static final Integer STATUS_ENABLE = 1;）。

#### 7.3 static使用（禁止）

校验标准：禁止在业务类中定义static成员变量（除非常量），避免线程安全问题；

修复方式：Skill提示static变量风险，人工调整。

## 版本变更记录（按倒序）

- v1.0.0：基于阿里Java手册黄山版重构，新增生产级日志脱敏、SQL防注入、空指针防护规则，适配Spring Boot 2.7+；集合初始化、魔法值、static使用规则，优化注释模板；包含命名、注释、日志基础规则。

## 校验结果输出规范（供Qoder Skill参考）

1. 严重违规：列出问题+修复示例，Skill强制阻断代码提交（如SQL拼接、捕获大异常）；

2. 高危违规：列出问题+自动修复代码，需人工确认（如日志脱敏缺失）；

3. 规范违规：Skill自动修复，生成修复报告（如命名错误、空行不规范）；

4. 输出格式：按模块分类（命名/注释/日志），标注违规行数+修复方式。