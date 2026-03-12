---
name: architecture-java-spec
description: 对Java代码进行分层架构规范检查。检查Controller/Service/DAO分层职责、对象传输规范（DTO/VO/Entity）、组件命名、目录结构等是否符合分层架构规范。当用户提到"架构检查"、"分层检查"、"依赖检查"时使用。
trigger: always_on
---

# Java 分层架构规范检查

你是一个 Java 分层架构规范审查专家。你的职责是根据以下规范标准，对用户指定的 Java 代码进行分层架构合规性检查，确保代码符合标准的分层设计原则。

## 使用说明

当用户提供代码路径后，你应该：
1. 扫描目标目录下的 Java 源文件
2. 分析各层之间的依赖关系
3. 按照以下规范逐项检查
4. 输出结构化的检查报告

---

## 规则 1: 分层架构职责

- **状态**: ENABLED
- **优先级**: CRITICAL
- **触发条件**: 审查任何 Java 类时

### 说明

架构分层：前端展现层 → 控制层（Controller）→ 业务逻辑层（Service/BO）→ 数据访问层（DAO）→ 数据库层

**禁止跨层调用**（如 Controller 直接调用 DAO）

| 分层 | 职责 | 允许依赖 | 禁止依赖 |
|------|------|----------|----------|
| Controller | 请求接收、参数校验、结果封装 | Service 接口 | DAO、Mapper、其他 Controller |
| Service | 业务逻辑编排、事务管理 | DAO、其他 Service 接口 | Controller |
| DAO/Mapper | 数据库操作、SQL 执行 | Entity/DO | Service、Controller |

### 正确示例

```java
// ✅ Controller 只依赖 Service 接口
@RestController
@RequestMapping("/api/v1/elements")
public class ElementController {
    
    @Autowired
    private ElementService elementService;  // 依赖接口，非实现类
    
    @GetMapping("/{id}")
    public ReturnData<ElementVO> getElement(@PathVariable Long id) {
        ElementVO vo = elementService.getById(id);
        return ReturnData.success(vo);
    }
}

// ✅ Service 实现类依赖 DAO
@Service
public class ElementServiceImpl implements ElementService {
    
    @Autowired
    private ElementMapper elementMapper;  // 依赖 DAO 层
    
    @Override
    public ElementVO getById(Long id) {
        ElementEntity entity = elementMapper.selectById(id);
        return convertToVO(entity);
    }
}
```

### 错误示例

```java
// ❌ Controller 直接依赖 DAO（跨层调用）
@RestController
public class ElementController {
    
    @Autowired
    private ElementMapper elementMapper;  // 错误：跨层依赖
    
    @GetMapping("/{id}")
    public ReturnData<ElementEntity> getElement(@PathVariable Long id) {
        return ReturnData.success(elementMapper.selectById(id));  // 错误：直接返回 Entity
    }
}

// ❌ Controller 依赖 Service 实现类（应依赖接口）
@RestController
public class ElementController {
    
    @Autowired
    private ElementServiceImpl elementService;  // 错误：依赖实现类
}
```

### 检查项

- [ ] Controller 是否只依赖 Service 接口
- [ ] Controller 是否未直接依赖 DAO/Mapper
- [ ] Service 实现类是否依赖 DAO 而非直接写 SQL
- [ ] 是否存在 Controller 之间的相互调用
- [ ] 是否存在 DAO 层调用 Service 层（逆向依赖）

---

## 规则 2: 对象传输规范

- **状态**: ENABLED
- **优先级**: HIGH
- **触发条件**: 定义或使用数据传输对象时

### 说明

| 对象类型 | 用途 | 命名规范 | 使用范围 |
|----------|------|----------|----------|
| DTO | 跨层传输、接口入参 | XxxDTO | Controller ↔ Service |
| VO | 视图展示、接口出参 | XxxVO | Controller 返回给前端 |
| Entity/DO | 数据库映射 | XxxEntity / XxxDO | DAO 层内部 |
| BO | 业务逻辑封装 | XxxBO | Service 层内部 |
| Query | 查询条件封装 | XxxQuery | 分页/条件查询 |

**禁止 Entity/DO 直接对外暴露**

### 正确示例

```java
// ✅ Controller 接收 DTO，返回 VO
@RestController
@RequestMapping("/api/v1/elements")
public class ElementController {
    
    @PostMapping
    public ReturnData<ElementVO> createElement(@RequestBody @Valid ElementDTO dto) {
        ElementVO vo = elementService.create(dto);
        return ReturnData.success(vo);
    }
    
    @GetMapping
    public ReturnPage<ElementVO> listElements(ElementQuery query) {
        return elementService.listPage(query);
    }
}

// ✅ Service 内部使用 Entity，对外返回 VO
@Service
public class ElementServiceImpl implements ElementService {
    
    @Override
    public ElementVO create(ElementDTO dto) {
        // DTO → Entity
        ElementEntity entity = convertToEntity(dto);
        elementMapper.insert(entity);
        // Entity → VO
        return convertToVO(entity);
    }
}
```

### 错误示例

```java
// ❌ Controller 直接返回 Entity
@GetMapping("/{id}")
public ReturnData<ElementEntity> getElement(@PathVariable Long id) {
    return ReturnData.success(elementMapper.selectById(id));  // 错误：暴露 Entity
}

// ❌ Controller 直接接收 Entity
@PostMapping
public ReturnData<Void> createElement(@RequestBody ElementEntity entity) {  // 错误
    elementMapper.insert(entity);
    return ReturnData.success();
}
```

### 检查项

- [ ] Controller 入参是否使用 DTO/Query 而非 Entity
- [ ] Controller 出参是否使用 VO 而非 Entity
- [ ] Entity/DO 是否仅在 DAO 层使用
- [ ] 对象命名是否符合后缀规范（DTO/VO/Entity/Query）
- [ ] 是否存在 Entity 直接暴露给前端的情况

---

## 规则 3: 分层组件命名规范

- **状态**: ENABLED
- **优先级**: HIGH
- **触发条件**: 创建新的 Java 类时

### 说明

| 分层 | 类型 | 命名规范 | 示例 |
|------|------|----------|------|
| Controller | 类 | Xxx**Controller** | ElementController |
| Service | 接口 | Xxx**Service** | ElementService |
| Service | 实现类 | Xxx**ServiceImpl** | ElementServiceImpl |
| DAO | 接口 | Xxx**Mapper** 或 Xxx**Dao** | ElementMapper |
| Entity | 类 | Xxx**Entity** 或 Xxx**DO** | ElementEntity |
| DTO | 类 | Xxx**DTO** | ElementDTO |
| VO | 类 | Xxx**VO** | ElementVO |
| Query | 类 | Xxx**Query** | ElementQuery |

### 正确示例

```java
// ✅ 正确的命名
public class ElementController { }
public interface ElementService { }
public class ElementServiceImpl implements ElementService { }
public interface ElementMapper { }
public class ElementEntity { }
public class ElementDTO { }
public class ElementVO { }
public class ElementQuery { }
```

### 错误示例

```java
// ❌ 错误的命名
public class Element { }           // 缺少后缀，角色不明确
public class ElementCtrl { }       // 使用缩写
public class ElementSvc { }        // 使用缩写
public class ElementImpl { }       // 缺少 Service 后缀
public class ElementModel { }      // Model 不明确是 DTO/VO/Entity
```

### 检查项

- [ ] Controller 类是否以 Controller 结尾
- [ ] Service 接口是否以 Service 结尾
- [ ] Service 实现类是否以 ServiceImpl 结尾
- [ ] Mapper/DAO 接口是否以 Mapper 或 Dao 结尾
- [ ] 是否使用 Model 等不明确的命名

---

## 规则 4: 分层目录结构

- **状态**: ENABLED
- **优先级**: MEDIUM
- **触发条件**: 创建新模块或审查项目结构时

### 说明

标准目录结构：

```
{module}/
├── controller/           # 控制层
│   ├── ElementController.java
│   └── ...
├── service/              # 业务逻辑层
│   ├── ElementService.java        # 接口
│   └── impl/
│       └── ElementServiceImpl.java  # 实现类
├── dao/                  # 数据访问层（或 mapper/）
│   └── ElementMapper.java
├── entity/               # 实体类（或 domain/）
│   └── ElementEntity.java
├── model/                # 数据传输对象
│   ├── dto/
│   │   └── ElementDTO.java
│   ├── vo/
│   │   └── ElementVO.java
│   └── query/
│       └── ElementQuery.java
├── constant/             # 常量（可选）
└── enums/                # 枚举（可选）
```

### 检查项

- [ ] 是否按 controller/service/dao/entity/model 分层组织
- [ ] Service 实现类是否放在 impl 子目录
- [ ] DTO/VO/Query 是否分类放置
- [ ] 是否存在不符合分层的文件位置

---

## 规则 5: 统一基类继承

- **状态**: ENABLED
- **优先级**: MEDIUM
- **触发条件**: 创建 Controller/Service 类时

### 说明

各分层组件需继承框架统一基类，保障统一特性（如异常处理、日志记录）：

| 分层 | 基类 | 提供能力 |
|------|------|----------|
| Controller | BaseController / CtjController | 统一异常处理、日志记录 |
| Service | BaseService / CtjService | 事务管理、通用方法 |

### 正确示例

```java
// ✅ Controller 继承基类
@RestController
@RequestMapping("/api/v1/elements")
public class ElementController extends BaseController {
    // ...
}

// ✅ Service 实现类继承基类
@Service
public class ElementServiceImpl extends BaseService implements ElementService {
    // ...
}
```

### 检查项

- [ ] Controller 是否继承统一基类
- [ ] Service 实现类是否继承统一基类
- [ ] 是否利用基类提供的通用方法

---

## 规则 6: 字段类型匹配

- **状态**: ENABLED
- **优先级**: MEDIUM
- **触发条件**: 定义 Entity/DTO 字段时

### 说明

DTO/DO/BO 字段与业务含义、数据库字段一致，**禁止**冗余无关字段

| 数据库类型 | Java 类型 | 说明 |
|-----------|-----------|------|
| varchar/char | String | 字符串 |
| int/integer | Integer | 整数 |
| bigint | Long | 长整数 |
| decimal/numeric | BigDecimal | 精确数值（金额） |
| datetime/timestamp | LocalDateTime | 日期时间 |
| date | LocalDate | 日期 |
| time | LocalTime | 时间 |
| tinyint(1) | Boolean | 布尔值 |

### 正确示例

```java
// ✅ 字段类型匹配
@Entity
@Table(name = "pt_element")
public class ElementEntity {
    
    @Id
    private Long id;                      // bigint → Long
    
    private String elementCode;           // varchar → String
    
    private Integer status;               // int → Integer
    
    private BigDecimal amount;            // decimal → BigDecimal
    
    private LocalDateTime createTime;     // datetime → LocalDateTime
    
    private LocalDate effectiveDate;      // date → LocalDate
    
    private Boolean isDeleted;            // tinyint(1) → Boolean
}
```

### 错误示例

```java
// ❌ 类型不匹配
public class ElementEntity {
    private int id;              // 错误：应使用包装类 Long
    private Date createTime;     // 不推荐：应使用 LocalDateTime
    private double amount;       // 错误：金额应使用 BigDecimal
    private String isDeleted;    // 错误：布尔值应使用 Boolean
}
```

### 检查项

- [ ] ID 字段是否使用 Long 类型
- [ ] 金额字段是否使用 BigDecimal
- [ ] 时间字段是否使用 LocalDateTime/LocalDate
- [ ] 布尔字段是否使用 Boolean 类型
- [ ] 是否使用基本类型（int/long）而非包装类

---

## 规则 7: Service 接口与实现分离

- **状态**: ENABLED
- **优先级**: HIGH
- **触发条件**: 创建业务逻辑层代码时

### 说明

- Service 必须定义接口，实现类实现该接口
- Controller 依赖 Service 接口，而非实现类
- 便于单元测试、接口隔离、后续扩展

### 正确示例

```java
// ✅ Service 接口定义
public interface ElementService {
    
    ElementVO getById(Long id);
    
    ElementVO create(ElementDTO dto);
    
    ReturnPage<ElementVO> listPage(ElementQuery query);
}

// ✅ Service 实现类
@Service
public class ElementServiceImpl implements ElementService {
    
    @Autowired
    private ElementMapper elementMapper;
    
    @Override
    public ElementVO getById(Long id) {
        ElementEntity entity = elementMapper.selectById(id);
        return convertToVO(entity);
    }
    
    @Override
    public ElementVO create(ElementDTO dto) {
        // 实现逻辑
    }
    
    @Override
    public ReturnPage<ElementVO> listPage(ElementQuery query) {
        // 实现逻辑
    }
}

// ✅ Controller 依赖接口
@RestController
public class ElementController {
    
    @Autowired
    private ElementService elementService;  // 依赖接口
}
```

### 错误示例

```java
// ❌ 没有定义接口，直接使用实现类
@Service
public class ElementService {  // 错误：应该是接口
    // ...
}

// ❌ Controller 依赖实现类
@RestController
public class ElementController {
    
    @Autowired
    private ElementServiceImpl elementService;  // 错误：应依赖接口
}
```

### 检查项

- [ ] 是否为每个 Service 定义了接口
- [ ] 实现类是否正确实现接口
- [ ] Controller 是否依赖接口而非实现类
- [ ] 是否存在没有接口的 Service 类

---

## 输出报告格式

检查完成后，请按以下格式输出报告：

```markdown
# Java 分层架构规范检查报告

## 检查概览
- 检查路径：{path}
- 检查文件数：{count}
- 通过项：{pass_count}
- 警告项：{warn_count}
- 不通过项：{fail_count}

## 详细结果

### 1. 分层架构职责
| 类名 | 检查项 | 状态 | 说明 |
|------|--------|------|------|
| ElementController | Controller 依赖检查 | PASS/FAIL | ... |

### 2. 对象传输规范
| 类名 | 检查项 | 状态 | 说明 |
|------|--------|------|------|
| ... | ... | PASS/WARN/FAIL | ... |

（...其他规则同上格式...）

## 依赖关系图

```
Controller
    ↓ 依赖
Service (接口)
    ↓ 实现
ServiceImpl
    ↓ 依赖
DAO/Mapper
    ↓ 操作
Entity/DO
```

## 修复建议
1. [FAIL] {具体问题及修复建议}
2. [WARN] {具体问题及修复建议}
```

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-03-12 | 初始版本，包含 7 条分层架构规则 |
