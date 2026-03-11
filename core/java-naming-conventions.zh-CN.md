---
trigger: automatic
---

# 命名约定规范 v1.0
# ============================================
# AI 辅助代码生成的命名标准
# 通过将 [ENABLED] 更改为 [DISABLED] 来启用/禁用约定，规则或者约束默认ENABLED，除非特别标注DISABLED
#
# 使用方法：
# 1. 将此文件放在项目根目录
# 2. 根据您的语言/框架启用/禁用约定
# 3. 在 AI 对话中使用 @naming-conventions.zh-CN.txt 引用
# 4. AI 将只遵循 ENABLED 的约定
# 5.本规范适用于所有 Java 后端开发项目，无关业务领域与行业属性，是保障代码一致性、可维护性、安全性的基础准则
# 最后更新：2026-03-11
# ============================================

# ============================================
# 摘要 - 规则概述
# ============================================
✅ [规则 1] 包名命名
✅ [规则 2] 类 / 接口命名
✅ [规则 3] 方法命名
✅ [规则 4] 变量 / 常量命名
✅ [规则 5] 文件命名
# ============================================
# 版本历史
# ============================================
# v1.0 (2026-03-11) - java命名规范，包含 5 条规则
# ============================================

## [规则 1] 包名命名

> **【强制】** 采用全小写字母，遵循反向域名格式 `com.ctjsoft.模块.分层`，单词间直接连接，**禁止**下划线、大写字母或中文。

| 示例 |
|------|
| `com.ctjsoft.user.controller` |
| `com.ctjsoft.order.service` |

## [规则 2] 类 / 接口命名

> **【强制】** 类名：大驼峰命名法（PascalCase），后缀明确区分类型（`Controller`/`Service`/`BO`/`DAO`/`DTO`/`DO`/`JOB`/`Exception`）。

| 类型 | 示例 |
|------|------|
| 控制器 | `UserController` |
| 服务实现 | `OrderServiceImpl` |
| 数据传输对象 | `UserDTO` |
| 异常类 | `BusinessException` |

> **【强制】** 接口名：大驼峰命名法，可前缀 `I`（可选，团队统一即可）或直接语义命名。

| 示例 |
|------|
| `UserService` |
| `IOrderRepository` |

> **【强制】** 实现类命名：接口名 + `Impl` 后缀

| 示例 |
|------|
| `UserServiceImpl` 实现 `UserService` |

## [规则 3] 方法命名

> **【强制】** 小驼峰命名法（camelCase），动词开头，明确表达功能意图。

| 类型 | 命名示例 |
|------|----------|
| 查询类 | `getUserById`、`listOrders` |
| 新增类 | `saveUser`、`insertOrder` |
| 更新类 | `updateUserName`、`modifyOrderStatus` |
| 删除类 | `deleteUser`、`removeOrder` |
| 校验类 | `validateParam`、`checkDataPermission` |

> **【禁止】** 无意义命名（`aaa`、`test123`）或模糊命名（`doSomething`）。

## [规则 4] 变量 / 常量命名

> **【强制】** 成员变量 / 参数：小驼峰命名法，**禁止**单字母（`i`/`j` 仅用于循环变量），避免歧义（用 `userName` 而非 `name`）。

> **【强制】** 常量：全大写字母，单词间下划线分隔，存储在 `Constants` 类中，**禁止**魔法值直接出现在代码中。

```java
public static final int MAX_RETRY_COUNT = 3;
public static final String DEFAULT_CHARSET = "UTF-8";
```

> **【强制】** 枚举类：大驼峰命名，枚举值全大写 + 下划线分隔。

```java
enum OrderStatus { 
    ORDER_PENDING, 
    ORDER_PAID, 
    ORDER_CANCELLED 
}
```

## [规则 5] 文件命名

| 类型 | 命名规则 | 示例 |
|------|----------|------|
| Java 文件 | 与类名完全一致（含大小写） | `UserService.java` |
| Mapper 文件 | 与 DAO 接口同名 + `Mapper.xml` 后缀，路径与 DAO 接口一致 | `UserDAO` 对应 `UserMapper.xml` |
| 配置文件 | 全小写，单词间中划线分隔 | `application-dev.yml`、`logback-spring.xml` |

---

## [规则 6] 工具类规范

> **【强制】** 工具类为单例或静态方法，构造方法私有，**禁止**实例化。

```java
private ToolClass() {}
```

> **【强制】** 工具类命名以 `Util` 结尾（如 `StringUtil`、`DateUtil`），功能单一（避免万能工具类）。

> **【强制】** 工具类依赖的资源（流、连接）需及时释放，优先使用 **try-with-resources** 语法。
