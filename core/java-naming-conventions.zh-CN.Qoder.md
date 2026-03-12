---
name: java-naming-conventions
description: Java代码命名规范检查。检查包名、类名、方法名、变量名、常量名、文件名等是否符合命名规范。当用户提到"命名检查"、"命名规范"、"变量命名"时使用。
trigger: always_on
---

# Java 命名规范

你是一个 Java 代码命名规范审查专家。你的职责是确保代码命名符合以下规范标准，提高代码可读性和一致性。

---

## 规则 1: 包名命名规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

采用全小写字母，遵循反向域名格式 `com.公司.模块.分层`，单词间直接连接。

**禁止**：下划线、大写字母、中文

### 正确示例

```java
// ✅ 正确：全小写，反向域名格式
package com.ctjsoft.user.controller;
package com.ctjsoft.order.service;
package com.ctjsoft.element.dao;
package com.ctjsoft.common.util;
```

### 错误示例

```java
// ❌ 错误：包含大写字母
package com.ctjsoft.User.controller;

// ❌ 错误：包含下划线
package com.ctjsoft.user_manager.service;

// ❌ 错误：包含中文
package com.ctjsoft.用户.controller;
```

### 检查项

- [ ] 包名是否全小写
- [ ] 是否遵循反向域名格式
- [ ] 是否包含下划线
- [ ] 是否包含大写字母
- [ ] 是否按分层组织（controller/service/dao等）

---

## 规则 2: 类/接口命名规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 类名：大驼峰命名法（PascalCase）
- 后缀明确区分类型：`Controller`/`Service`/`ServiceImpl`/`DAO`/`Mapper`/`DTO`/`VO`/`Entity`/`Exception`

### 标准后缀规范

| 类型 | 后缀 | 示例 |
|------|------|------|
| 控制器 | Controller | UserController |
| 服务接口 | Service | UserService |
| 服务实现 | ServiceImpl | UserServiceImpl |
| 数据访问 | Mapper / DAO | UserMapper / UserDAO |
| 数据传输对象 | DTO | UserDTO |
| 视图对象 | VO | UserVO |
| 实体类 | Entity / DO | UserEntity / UserDO |
| 查询对象 | Query | UserQuery |
| 业务对象 | BO | OrderBO |
| 异常类 | Exception | BusinessException |
| 工具类 | Util / Utils | StringUtil |
| 常量类 | Constants | ElementConstants |

### 正确示例

```java
// ✅ 正确：大驼峰，后缀明确
public class UserController { }
public interface UserService { }
public class UserServiceImpl implements UserService { }
public class UserDTO { }
public class UserVO { }
public class UserEntity { }
public class BusinessException extends RuntimeException { }
```

### 错误示例

```java
// ❌ 错误：小写开头
public class userController { }

// ❌ 错误：缺少后缀，角色不明确
public class User { }

// ❌ 错误：使用缩写
public class UserCtrl { }
public class UserSvc { }

// ❌ 错误：使用下划线
public class User_Controller { }

// ❌ 错误：后缀不规范
public class UserModel { }  // 应明确是 DTO/VO/Entity
```

### 检查项

- [ ] 类名是否使用大驼峰命名
- [ ] Controller 类是否以 Controller 结尾
- [ ] Service 接口是否以 Service 结尾
- [ ] Service 实现类是否以 ServiceImpl 结尾
- [ ] Mapper/DAO 是否以 Mapper 或 DAO 结尾
- [ ] DTO/VO/Entity 是否有明确后缀
- [ ] 是否使用完整单词而非缩写
- [ ] 是否存在下划线

---

## 规则 3: 方法命名规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

小驼峰命名法（camelCase），动词开头，明确表达功能意图。

### 方法命名模式

| 类型 | 前缀 | 示例 |
|------|------|------|
| 查询单个 | get / find | getUserById, findByCode |
| 查询列表 | list / find | listUsers, findAllByStatus |
| 新增 | save / insert / add / create | saveUser, insertOrder, createAccount |
| 更新 | update / modify | updateUserName, modifyOrderStatus |
| 删除 | delete / remove | deleteUser, removeOrder |
| 校验 | validate / check / verify | validateParam, checkPermission |
| 判断 | is / has / can | isValid, hasPermission, canAccess |
| 转换 | convert / to / parse | convertToDTO, toVO, parseDate |
| 计算 | calculate / compute | calculateAmount, computeScore |
| 初始化 | init / initialize | initConfig, initializeCache |

### 正确示例

```java
// ✅ 正确：小驼峰，动词开头，语义明确
public UserVO getUserById(Long userId) { }
public List<UserVO> listUsersByStatus(Integer status) { }
public void saveUser(UserDTO userDTO) { }
public void updateUserName(Long userId, String newName) { }
public void deleteUserById(Long userId) { }
public boolean validateUserParam(UserDTO userDTO) { }
public boolean isUserExist(Long userId) { }
public UserVO convertToVO(UserEntity entity) { }
```

### 错误示例

```java
// ❌ 错误：大写开头
public UserVO GetUserById(Long userId) { }

// ❌ 错误：无意义命名
public void doSomething() { }
public void process() { }
public void handle() { }

// ❌ 错误：名词开头
public UserVO userQuery(Long userId) { }

// ❌ 错误：包含下划线
public void get_user_by_id(Long userId) { }

// ❌ 错误：缩写不清晰
public UserVO getUsrById(Long id) { }
```

### 检查项

- [ ] 方法名是否使用小驼峰命名
- [ ] 是否以动词开头
- [ ] 命名是否语义明确
- [ ] 是否存在无意义命名（doSomething, process等）
- [ ] 是否使用完整单词
- [ ] 是否包含下划线

---

## 规则 4: 变量命名规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 成员变量/局部变量/参数：小驼峰命名法
- **禁止**单字母变量（`i`/`j`/`k` 仅用于循环）
- 避免歧义，使用完整语义（用 `userName` 而非 `name`）

### 变量命名后缀规范

| 类型 | 后缀 | 示例 |
|------|------|------|
| ID | Id | userId, orderId |
| 编号 | No / Code | orderNo, elementCode |
| 名称 | Name | userName, elementName |
| 状态 | Status | orderStatus, userStatus |
| 类型 | Type | elementType, payType |
| 时间 | Time / Date | createTime, updateDate |
| 数量 | Count / Num | orderCount, pageNum |
| 金额 | Amount / Price | totalAmount, unitPrice |
| 列表 | List / s(复数) | userList, orders |
| 布尔 | is / has / can | isDeleted, hasPermission |

### 正确示例

```java
// ✅ 正确：小驼峰，语义明确
private Long userId;
private String userName;
private Integer orderStatus;
private LocalDateTime createTime;
private BigDecimal totalAmount;
private List<OrderVO> orderList;
private Boolean isDeleted;

// ✅ 正确：循环变量可用单字母
for (int i = 0; i < 10; i++) { }
```

### 错误示例

```java
// ❌ 错误：单字母变量
private String n;  // 应为 name
private Long u;    // 应为 userId

// ❌ 错误：语义不明
private String name;   // 应为 userName / elementName
private Integer status; // 应为 orderStatus / userStatus

// ❌ 错误：拼音命名
private String xingming;  // 应为 userName
private Integer zhuangtai; // 应为 status

// ❌ 错误：下划线命名
private String user_name;

// ❌ 错误：大写开头
private String UserName;
```

### 检查项

- [ ] 变量名是否使用小驼峰命名
- [ ] 是否存在单字母变量（循环变量除外）
- [ ] 是否存在语义不明的命名
- [ ] 是否存在拼音命名
- [ ] 是否包含下划线
- [ ] ID/状态/时间等是否使用标准后缀

---

## 规则 5: 常量命名规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 常量：全大写字母，单词间下划线分隔
- 存储在 `Constants` 类中
- **禁止**魔法值直接出现在代码中

### 正确示例

```java
// ✅ 正确：常量定义
public class ElementConstants {
    
    /** 元素状态：启用 */
    public static final Integer STATUS_ENABLED = 1;
    
    /** 元素状态：停用 */
    public static final Integer STATUS_DISABLED = 2;
    
    /** 默认分页大小 */
    public static final Integer DEFAULT_PAGE_SIZE = 20;
    
    /** 最大批量操作数量 */
    public static final Integer MAX_BATCH_SIZE = 1000;
    
    /** 默认字符集 */
    public static final String DEFAULT_CHARSET = "UTF-8";
}

// ✅ 正确：使用常量
if (element.getStatus().equals(ElementConstants.STATUS_ENABLED)) {
    // ...
}
```

### 错误示例

```java
// ❌ 错误：小写或驼峰
public static final int maxRetryCount = 3;
public static final String defaultCharset = "UTF-8";

// ❌ 错误：魔法值
if (status == 1) {  // 1 是什么意思？
    // ...
}

if (pageSize > 100) {  // 100 是什么意思？
    // ...
}

// ❌ 错误：常量散落在各处
public class UserService {
    private static final int MAX_SIZE = 100;  // 应放在 Constants 类中
}
```

### 检查项

- [ ] 常量是否全大写下划线分隔
- [ ] 是否存在魔法值
- [ ] 常量是否集中在 Constants 类中
- [ ] 常量是否有注释说明含义

---

## 规则 6: 枚举命名规范

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【强制】

### 说明

- 枚举类名：大驼峰命名，以 `Enum` 结尾（可选）
- 枚举值：全大写，单词间下划线分隔

### 正确示例

```java
// ✅ 正确：枚举定义
public enum OrderStatusEnum {
    
    /** 待支付 */
    WAIT_PAY(1, "待支付"),
    
    /** 已支付 */
    PAID(2, "已支付"),
    
    /** 已发货 */
    SHIPPED(3, "已发货"),
    
    /** 已完成 */
    COMPLETED(4, "已完成"),
    
    /** 已取消 */
    CANCELLED(5, "已取消");
    
    private final Integer code;
    private final String desc;
    
    OrderStatusEnum(Integer code, String desc) {
        this.code = code;
        this.desc = desc;
    }
    
    // getter...
}
```

### 错误示例

```java
// ❌ 错误：枚举值使用驼峰
public enum OrderStatus {
    WaitPay,    // 应为 WAIT_PAY
    Paid,       // 应为 PAID
    Shipped     // 应为 SHIPPED
}

// ❌ 错误：枚举值使用小写
public enum OrderStatus {
    wait_pay,
    paid,
    shipped
}
```

### 检查项

- [ ] 枚举类名是否使用大驼峰
- [ ] 枚举值是否全大写下划线分隔
- [ ] 枚举是否有 code 和 desc 属性
- [ ] 枚举值是否有注释说明

---

## 规则 7: 文件命名规范

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【强制】

### 说明

| 类型 | 命名规则 | 示例 |
|------|----------|------|
| Java 文件 | 与类名完全一致（含大小写） | `UserService.java` |
| Mapper XML | 与 Mapper 接口同名 | `UserMapper.xml` |
| 配置文件 | 全小写，单词间中划线分隔 | `application-dev.yml` |
| 资源文件 | 全小写，单词间中划线分隔 | `log-config.xml` |

### 正确示例

```
// ✅ 正确：文件命名
UserController.java
UserService.java
UserServiceImpl.java
UserMapper.java
UserMapper.xml
application.yml
application-dev.yml
application-prod.yml
logback-spring.xml
```

### 错误示例

```
// ❌ 错误：大小写不一致
userController.java  // 类名是 UserController

// ❌ 错误：配置文件用下划线
application_dev.yml  // 应为 application-dev.yml

// ❌ 错误：Mapper XML 名称不匹配
UserDAO.xml  // 接口是 UserMapper，应为 UserMapper.xml
```

### 检查项

- [ ] Java 文件名是否与类名一致
- [ ] Mapper XML 是否与 Mapper 接口同名
- [ ] 配置文件是否使用中划线分隔
- [ ] 是否存在大小写不一致

---

## 规则 8: 工具类命名规范

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【强制】

### 说明

- 工具类命名以 `Util` 或 `Utils` 结尾
- 构造方法私有，禁止实例化
- 功能单一，避免万能工具类

### 正确示例

```java
// ✅ 正确：工具类定义
public final class StringUtil {
    
    // 私有构造方法，禁止实例化
    private StringUtil() {
        throw new UnsupportedOperationException("工具类不允许实例化");
    }
    
    /**
     * 判断字符串是否为空
     */
    public static boolean isEmpty(String str) {
        return str == null || str.trim().isEmpty();
    }
    
    /**
     * 判断字符串是否不为空
     */
    public static boolean isNotEmpty(String str) {
        return !isEmpty(str);
    }
}
```

### 错误示例

```java
// ❌ 错误：没有私有构造方法
public class StringUtil {
    public static boolean isEmpty(String str) { }
}

// ❌ 错误：命名不规范
public class StringHelper { }  // 应为 StringUtil
public class StringTool { }    // 应为 StringUtil

// ❌ 错误：万能工具类
public class CommonUtil {
    public static String formatDate() { }
    public static BigDecimal calculateAmount() { }
    public static void sendEmail() { }
    // 功能太杂，应拆分为 DateUtil, AmountUtil, EmailUtil
}
```

### 检查项

- [ ] 工具类是否以 Util/Utils 结尾
- [ ] 是否有私有构造方法
- [ ] 类是否声明为 final
- [ ] 功能是否单一

---

## 规则清单

| 规则 | 名称 | 强制级别 | 优先级 |
|------|------|----------|--------|
| 规则 1 | 包名命名规范 | 【强制】 | HIGH |
| 规则 2 | 类/接口命名规范 | 【强制】 | HIGH |
| 规则 3 | 方法命名规范 | 【强制】 | HIGH |
| 规则 4 | 变量命名规范 | 【强制】 | HIGH |
| 规则 5 | 常量命名规范 | 【强制】 | HIGH |
| 规则 6 | 枚举命名规范 | 【强制】 | MEDIUM |
| 规则 7 | 文件命名规范 | 【强制】 | MEDIUM |
| 规则 8 | 工具类命名规范 | 【强制】 | MEDIUM |

---

## 输出报告格式

检查完成后，请按以下格式输出报告：

```markdown
# Java 命名规范检查报告

## 检查概览
- 检查路径：{path}
- 检查文件数：{count}
- 通过项：{pass_count}
- 警告项：{warn_count}
- 不通过项：{fail_count}

## 详细结果

### 1. 包名规范
| 包名 | 状态 | 说明 |
|------|------|------|
| com.ctjsoft.user.controller | PASS | 符合规范 |
| com.ctjsoft.User.service | FAIL | 包含大写字母 |

### 2. 类名规范
| 类名 | 状态 | 说明 |
|------|------|------|
| UserController | PASS | 符合规范 |
| userService | FAIL | 应使用大驼峰命名 |

### 3. 方法名规范
| 类名 | 方法名 | 状态 | 说明 |
|------|--------|------|------|
| UserService | getUserById | PASS | 符合规范 |
| OrderService | DoSomething | FAIL | 应使用小驼峰，且语义不明 |

## 修复建议
1. [FAIL] userService 类名应改为 UserService
2. [FAIL] DoSomething 方法应改为具体语义的方法名
```

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-03-12 | 初始版本，包含 8 条命名规范规则 |
