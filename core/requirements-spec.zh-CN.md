---
name: requirements-spec
description: 开发需求规范检查。检查代码完整性、API复用、依赖管理、修改范围、API验证、编译成功、命名约定等是否符合规范。当用户提到"代码规范"、"开发规范"、"需求规范"、"代码质量"时使用。
trigger: always_on
---

# 开发需求规范 v1.0

AI 辅助开发的编码规则和要求，生成代码时严格遵循这些规则。

适用范围：Java/Spring Boot 项目
最后更新：2026-03-11

---

## 规则清单

| 规则 | 名称 | 强制级别 | 优先级 |
|------|------|----------|--------|
| 规则 1 | 生成完整可运行代码 | 【强制】 | CRITICAL |
| 规则 2 | 复用现有代码和API | 【强制】 | HIGH |
| 规则 3 | 最小化新增依赖 | 【强制】 | HIGH |
| 规则 4 | 从之前的错误中学习 | 【强制】 | HIGH |
| 规则 5 | 仅修改请求的内容 | 【强制】 | HIGH |
| 规则 6 | 验证所有API是否存在 | 【强制】 | CRITICAL |
| 规则 7 | 第一次就完全修复错误 | 【强制】 | HIGH |
| 规则 8 | 保持注释与代码一致 | 【建议】 | MEDIUM |
| 规则 9 | 功能优先于完美 | 【建议】 | MEDIUM |
| 规则 10 | 确保代码成功编译 | 【强制】 | CRITICAL |
| 规则 11 | 严格遵循提供的示例 | 【建议】 | MEDIUM |
| 规则 12 | 尊重项目命名约定 | 【建议】 | MEDIUM |
| 规则 13 | 只使用真实存在的库 | 【强制】 | CRITICAL |

---

## [规则 1] 生成完整可运行代码（优先级：CRITICAL）【强制】

生成完整、可执行的代码，不使用占位符。

### 要求
- 代码必须可立即执行，不允许使用占位符
- 包含所有必要的 import、依赖和配置
- 提供完整实现，不是部分代码片段
- 不允许有不完整的函数或缺失的逻辑
- 不允许 TODO 注释或"稍后实现"的说明

### 后果
- 占位或部分代码导致运行时失败和返工
- 缺少 import/配置会造成构建中断和部署延迟

### 示例

❌ **错误示例**：
```java
public class UserService {
    
    public UserDTO processUser(Long userId) {
        // TODO: implement this
        return null;
    }
}
```

✅ **正确示例**：
```java
import org.springframework.stereotype.Service;
import org.springframework.beans.factory.annotation.Autowired;

@Service
public class UserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public UserDTO processUser(Long userId) {
        User user = userRepository.findById(userId)
            .orElseThrow(() -> new BusinessException("用户不存在"));
        
        UserDTO dto = new UserDTO();
        dto.setId(user.getId());
        dto.setName(user.getName().trim());
        dto.setCreateTime(LocalDateTime.now());
        return dto;
    }
}
```

### 检查清单
- [ ] 代码无 TODO 注释或占位符
- [ ] 所有必要的 import 语句已添加
- [ ] 所有方法都有完整实现
- [ ] 依赖注入正确配置

---

## [规则 2] 复用现有代码和API（优先级：HIGH）【强制】

创建新接口前必须先复用现有接口和 API。

### 要求
- 创建新接口前检查现有代码库
- 不要重复发明已有功能
- 引用并使用当前的 API 端点
- 保持与现有接口的兼容性
- 首先搜索类似的实现

### 示例

❌ **错误示例**：
```java
// 创建新的验证方法
public class OrderService {
    
    // 自己实现验证逻辑
    private boolean validateUser(User user) {
        if (user == null) return false;
        if (StringUtils.isEmpty(user.getName())) return false;
        // ... 重复实现
        return true;
    }
}
```

✅ **正确示例**：
```java
import com.example.common.service.UserValidationService;

public class OrderService {
    
    @Autowired
    private UserValidationService userValidationService;
    
    public void createOrder(OrderRequest request) {
        // 使用现有的验证服务
        userValidationService.validateUser(request.getUserId());
        // ...
    }
}
```

### 检查清单
- [ ] 已搜索现有代码库中的类似实现
- [ ] 使用现有的工具类和服务
- [ ] 未重复实现已有功能
- [ ] 与现有接口保持兼容

---

## [规则 3] 最小化新增依赖（优先级：HIGH）【强制】

优先使用项目现有依赖。

### 要求
- 优先使用项目现有依赖
- 添加新依赖需要充分理由
- 添加前检查 pom.xml/build.gradle
- 优先使用 JDK 内置库而非外部包
- 当存在简单的原生解决方案时，避免使用重量级依赖

### 示例

❌ **错误示例**：
```java
// 引入重量级依赖处理日期
import org.joda.time.DateTime;
import org.joda.time.format.DateTimeFormat;

public String formatDate(Date date) {
    DateTime dateTime = new DateTime(date);
    return dateTime.toString("yyyy-MM-dd");
}
```

✅ **正确示例**：
```java
// 使用 JDK 8+ 内置 API
import java.time.LocalDate;
import java.time.format.DateTimeFormatter;

public String formatDate(LocalDate date) {
    return date.format(DateTimeFormatter.ISO_LOCAL_DATE);
}
```

### 检查清单
- [ ] 已检查 pom.xml 中的现有依赖
- [ ] 优先使用 JDK 内置 API
- [ ] 新依赖有充分的添加理由
- [ ] 避免引入重量级依赖

---

## [规则 4] 从之前的错误中学习（优先级：HIGH）【强制】

不要重复对话历史中的错误。

### 要求
- 实现前参考对话历史
- 避免犯同样的错误两次
- 在所有代码中一致地应用修正
- 记住用户反馈和偏好

### 检查清单
- [ ] 已参考之前的错误记录
- [ ] 未重复已知的错误模式
- [ ] 修正已一致应用到所有代码
- [ ] 已考虑用户的反馈和偏好

---

## [规则 5] 仅修改请求的内容（优先级：HIGH）【强制】

只修改明确要求的内容，不要未经授权重构。

### 要求
- 未经许可不要重构可工作的代码
- 除非被要求改进，否则保持原有逻辑
- 严格遵守具体任务要求
- 最小化修改范围
- 保留现有代码结构

### 后果
- 非请求的重构引入回归并拖慢评审
- 修改范围膨胀增加成本且影响进度

### 示例（用户只要求修改按钮文本）

❌ **错误示例**：
```java
// 重构整个Controller架构
// 切换到新的响应格式
// 添加性能优化
@RestController
public class UserController {
    // 大规模重写...
}
```

✅ **正确示例**：
```java
// 只修改要求的内容
@GetMapping("/user/{id}")
public Result<UserVO> getUser(@PathVariable Long id) {
    // 只修改返回消息文本
    return Result.success(userService.getById(id), "获取成功");
}
```

### 检查清单
- [ ] 修改范围限于用户请求的内容
- [ ] 未进行未经授权的重构
- [ ] 保留了现有代码结构
- [ ] 修改最小化

---

## [规则 6] 验证所有API是否存在（优先级：CRITICAL）【强制】

不要发明不存在的 API 或方法。

### 要求
- 使用前验证 API 是否存在
- 查阅文档确认正确的 API 签名
- 只使用已确认可用的 API
- 不要编造虚构的库方法
- 测试 API 与项目版本的兼容性

### 后果
- 使用不存在或不兼容的 API 会导致崩溃与支持负担
- 错误签名可能造成数据悄然损坏

### 示例

❌ **错误示例**：
```java
// 使用不存在的方法
List<String> list = new ArrayList<>();
list.removeAt(0);  // ArrayList 没有 removeAt 方法
```

✅ **正确示例**：
```java
// 使用标准方法
List<String> list = new ArrayList<>();
list.remove(0);  // 正确的标准方法
```

### 检查清单
- [ ] 所有 API 调用已验证存在
- [ ] API 签名与文档一致
- [ ] API 版本与项目兼容
- [ ] 未使用虚构的方法

---

## [规则 7] 第一次就完全修复错误（优先级：HIGH）【强制】

解决根本原因，而不是症状。

### 要求
- 提交前测试解决方案
- 提供可工作的修复，而不是迭代尝试
- 解决根本原因，而不仅仅是症状
- 确保代码在交付前能编译和运行

### 检查清单
- [ ] 已分析错误的根本原因
- [ ] 修复方案已测试验证
- [ ] 代码可编译和运行
- [ ] 不是临时的补丁方案

---

## [规则 8] 保持注释与代码一致（优先级：MEDIUM）【建议】

确保注释与实际实现匹配。

### 要求
- 代码变更时更新注释
- 提供准确、真实的文档
- 不要有误导性或过时的注释
- 注释应反映实际行为

### 示例

❌ **错误示例**：
```java
/**
 * 获取用户列表
 */
public void deleteUser(Long id) {
    // 注释与方法功能不匹配
    userRepository.deleteById(id);
}
```

✅ **正确示例**：
```java
/**
 * 删除指定用户
 * @param id 用户ID
 */
public void deleteUser(Long id) {
    userRepository.deleteById(id);
}
```

### 检查清单
- [ ] 注释与代码功能一致
- [ ] 方法注释准确描述功能
- [ ] 无过时或误导性注释
- [ ] 代码变更时同步更新注释

---

## [规则 9] 功能优先于完美（优先级：MEDIUM）【建议】

优先让代码运行起来。

### 要求
- 可工作的代码 > 完美的代码
- 功能优先于优化
- 先交付 MVP 再做增强
- 先让它运行，然后再改进

### 检查清单
- [ ] 代码功能完整可运行
- [ ] 优先实现核心功能
- [ ] 避免过度优化
- [ ] 后续可迭代改进

---

## [规则 10] 确保代码成功编译（优先级：CRITICAL）【强制】

所有代码必须能编译/运行且无错误。

### 要求
- 交付前检查语法
- 解决所有编译错误
- 测试构建过程
- 验证 import 和依赖是否正确

### 后果
- 构建失败会阻塞交付流水线
- 运行时错误会降低用户体验与信任

### 检查清单
- [ ] 代码语法正确
- [ ] 无编译错误
- [ ] import 语句完整且正确
- [ ] 依赖配置正确

---

## [规则 11] 严格遵循提供的示例（优先级：MEDIUM）【建议】

当给出示例时，精确匹配它们。

### 要求
- 匹配示例的代码风格
- 使用相同的模式和结构
- 与提供的样本保持一致
- 复制结构和命名约定

### 检查清单
- [ ] 代码风格与示例一致
- [ ] 使用相同的设计模式
- [ ] 结构与示例匹配
- [ ] 命名约定一致

---

## [规则 12] 尊重项目命名约定（优先级：MEDIUM）【建议】

保留现有的命名模式。

### 要求
- 未经许可不要重命名变量/函数
- 遵循项目的命名风格
- 与代码库保持一致
- 匹配现有模式（camelCase、PascalCase 等）

### 检查清单
- [ ] 遵循项目命名风格
- [ ] 未擅自重命名现有代码
- [ ] 命名与代码库一致
- [ ] 类名用 PascalCase，方法名用 camelCase

---

## [规则 13] 只使用真实存在的库（优先级：CRITICAL）【强制】

只导入实际存在的库。

### 要求
- 导入前验证包是否存在
- 不要使用虚构或编造的包
- 检查 Maven Central/私有仓库
- 确认库版本兼容性

### 后果
- 导入虚构或错误版本的库会导致构建失败
- 未审查的包可能带来安全风险

### 示例

❌ **错误示例**：
```java
// 不存在的库
import com.example.magichelper.SuperMagicLib;

public class MyService {
    public void process() {
        SuperMagicLib.doMagic();  // 虚构的库
    }
}
```

✅ **正确示例**：
```java
// 真实存在的库
import org.apache.commons.lang3.StringUtils;

public class MyService {
    public void process(String input) {
        if (StringUtils.isNotBlank(input)) {
            // 使用真实的 Apache Commons 库
        }
    }
}
```

### 检查清单
- [ ] 所有导入的库真实存在
- [ ] 已在 Maven Central 验证
- [ ] 库版本与项目兼容
- [ ] 未使用虚构的包

---

## 输出报告格式

检查完成后，按以下格式输出报告：

```markdown
## 开发需求规范检查报告

### 检查概要
- 检查文件数：X
- 发现问题数：Y
- 严重程度分布：CRITICAL(n) / HIGH(n) / MEDIUM(n)

### 问题清单

#### [CRITICAL] 规则 X 违规
- 文件：`path/to/file.java`
- 位置：第 N 行
- 问题：具体问题描述
- 建议：修复建议

#### [HIGH] 规则 X 违规
- 文件：`path/to/file.java`
- 位置：第 N 行
- 问题：具体问题描述
- 建议：修复建议

### 合规项
- [x] 规则 X：符合要求
- [x] 规则 X：符合要求

### 改进建议
1. 具体改进建议
2. 具体改进建议
```

---

## 项目类型配置

不同项目类型推荐启用的规则集合：

### Web 应用（Spring Boot）
- **启用**：规则 1, 2, 3, 5, 6, 7, 10, 11, 12, 13
- **可选**：规则 8, 9
- **说明**：Web 应用需要完整性、API 准确性、快速修复，以及编译/运行保证

### 微服务项目
- **启用**：规则 1, 2, 3, 5, 6, 7, 10, 12, 13
- **可选**：规则 8, 9, 11
- **说明**：微服务更强调可靠性、最小依赖、稳定运行于多环境

### SDK/工具库
- **启用**：规则 1, 2, 3, 6, 7, 10, 12, 13
- **可选**：规则 5, 8, 9, 11
- **说明**：库强调 API 正确性、版本纪律、完整性与高质量错误处理

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-03-11 | 基于用户反馈数据的初始版本 |
| v1.1 | 2026-03-12 | 适配 Java/Spring Boot，添加检查清单 |
