---
name: sql-conventions
description: 数据库设计与SQL编写规范检查。检查表命名、字段命名、字段类型、索引设计、SQL编写、性能优化等是否符合规范。当用户提到"SQL检查"、"数据库规范"、"表设计"、"索引检查"时使用。
trigger: always_on
---

# 数据库设计与 SQL 编写规范

你是一个数据库设计与 SQL 编写规范审查专家。你的职责是确保数据库设计和 SQL 语句符合以下规范标准，保障数据库性能、一致性和可维护性。

---

## 规则 1: 表名命名规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 所有命名使用**小写字母**，单词间**下划线 `_`** 分隔
- **禁止**大写、特殊字符（`@`、`#`）、中文、拼音
- **禁止**使用数据库关键字（如 `order`、`user`、`table`）

### 正确示例

```sql
-- ✅ 正确：小写 + 下划线分隔
CREATE TABLE pt_element_value (
    id NUMBER(19) PRIMARY KEY,
    element_code VARCHAR2(50) NOT NULL,
    element_name VARCHAR2(100) NOT NULL
);

-- ✅ 正确：有意义的表名
CREATE TABLE sys_user_info (...);
CREATE TABLE biz_order_main (...);
CREATE TABLE cfg_system_param (...);
```

### 错误示例

```sql
-- ❌ 错误：使用大写
CREATE TABLE PT_ELEMENT_VALUE (...);

-- ❌ 错误：使用驼峰命名
CREATE TABLE ptElementValue (...);

-- ❌ 错误：使用关键字
CREATE TABLE order (...);  -- order 是关键字
CREATE TABLE user (...);   -- user 是关键字

-- ❌ 错误：使用中文或拼音
CREATE TABLE 用户表 (...);
CREATE TABLE yonghu_biao (...);

-- ❌ 错误：包含特殊字符
CREATE TABLE user@info (...);
```

### 检查项

- [ ] 表名是否全小写
- [ ] 单词间是否使用下划线分隔
- [ ] 是否使用了数据库关键字
- [ ] 是否包含中文或拼音
- [ ] 是否包含特殊字符

---

## 规则 2: 字段命名规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 字段名：小写字母 + 下划线分隔，见名知意
- 主键字段：统一命名 `id` 或 `表名简称_id`
- **禁止**模糊命名（`col1`、`data`、`info`）
- **禁止**使用外键约束

### 通用字段规范（所有表必含）

| 字段名 | 类型 | 说明 |
|--------|------|------|
| `id` | `NUMBER(19)` | 主键，雪花算法生成 |
| `rid` | `VARCHAR2(40)` | 数据唯一记录号，全库唯一 |
| `create_time` | `VARCHAR2(17)` | 创建时间，格式 yyyyMMddHHmmssSSS |
| `update_time` | `VARCHAR2(17)` | 更新时间，格式 yyyyMMddHHmmssSSS |
| `create_by` | `VARCHAR2(50)` | 创建人 |
| `update_by` | `VARCHAR2(50)` | 更新人 |

### 正确示例

```sql
-- ✅ 正确：规范的字段命名
CREATE TABLE pt_element (
    id NUMBER(19) PRIMARY KEY,
    rid VARCHAR2(40) NOT NULL,
    element_code VARCHAR2(50) NOT NULL,
    element_name VARCHAR2(100) NOT NULL,
    element_type NUMBER(3) DEFAULT 0,        -- 元素类型：0-普通 1-特殊
    parent_id NUMBER(19),
    sort_no NUMBER(5) DEFAULT 0,
    is_deleted NUMBER(1) DEFAULT 0,          -- 是否删除：0-否 1-是
    create_time VARCHAR2(17) NOT NULL,
    update_time VARCHAR2(17) NOT NULL,
    create_by VARCHAR2(50),
    update_by VARCHAR2(50)
);

-- ✅ 正确：字段注释说明枚举值
COMMENT ON COLUMN pt_element.element_type IS '元素类型：0-普通 1-特殊 2-系统';
COMMENT ON COLUMN pt_element.is_deleted IS '是否删除：0-否 1-是';
```

### 错误示例

```sql
-- ❌ 错误：模糊命名
CREATE TABLE pt_element (
    col1 NUMBER(19),           -- col1 是什么？
    data VARCHAR2(100),        -- data 是什么？
    info VARCHAR2(200),        -- info 是什么？
    type NUMBER(3)             -- type 太模糊，应为 element_type
);

-- ❌ 错误：使用大写或驼峰
CREATE TABLE pt_element (
    ElementCode VARCHAR2(50),  -- 应为 element_code
    ELEMENT_NAME VARCHAR2(100) -- 应为 element_name
);

-- ❌ 错误：使用外键
CREATE TABLE pt_element_value (
    element_id NUMBER(19) REFERENCES pt_element(id)  -- 禁止外键
);
```

### 检查项

- [ ] 字段名是否全小写下划线分隔
- [ ] 是否存在模糊命名（col1、data等）
- [ ] 是否包含通用字段（id、rid、create_time等）
- [ ] 枚举字段是否有注释说明
- [ ] 是否使用了外键约束

---

## 规则 3: 字段类型规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

优先使用占用空间小的类型，避免资源浪费。

| 数据类型 | 推荐类型 | 说明 |
|----------|----------|------|
| 主键 | `NUMBER(19)` | 雪花算法ID |
| 整型 | `NUMBER(n)` | 根据范围选择精度 |
| 金额 | `NUMBER(12,2)` | 精确到分，**禁止** FLOAT/DOUBLE |
| 字符串 | `VARCHAR2(n)` | 按需设置长度 |
| 时间 | `VARCHAR2(17)` | yyyyMMddHHmmssSSS |
| 布尔 | `NUMBER(1)` | 0-否 1-是 |
| 枚举 | `NUMBER(3)` | 通过注释说明含义 |

### 常用字段长度规范

| 字段类型 | 推荐长度 | 示例 |
|----------|----------|------|
| 用户名 | `VARCHAR2(32)` | username |
| 手机号 | `CHAR(11)` | phone |
| 身份证 | `CHAR(18)` | id_card |
| 邮箱 | `VARCHAR2(64)` | email |
| URL | `VARCHAR2(500)` | url |
| 备注 | `VARCHAR2(500)` | remark |
| 编码 | `VARCHAR2(50)` | xxx_code |
| 名称 | `VARCHAR2(100)` | xxx_name |

### 正确示例

```sql
-- ✅ 正确：合适的字段类型
CREATE TABLE biz_order (
    id NUMBER(19) PRIMARY KEY,
    order_no VARCHAR2(32) NOT NULL,          -- 订单号
    user_id NUMBER(19) NOT NULL,             -- 用户ID
    total_amount NUMBER(12,2) NOT NULL,      -- 订单金额，精确到分
    order_status NUMBER(3) DEFAULT 0,        -- 订单状态
    pay_time VARCHAR2(17),                   -- 支付时间
    is_deleted NUMBER(1) DEFAULT 0,          -- 是否删除
    create_time VARCHAR2(17) NOT NULL
);

-- ✅ 正确：非空约束和默认值
ALTER TABLE biz_order MODIFY order_status NUMBER(3) DEFAULT 0 NOT NULL;
```

### 错误示例

```sql
-- ❌ 错误：金额使用浮点数
CREATE TABLE biz_order (
    total_amount BINARY_DOUBLE  -- 会丢失精度！应用 NUMBER(12,2)
);

-- ❌ 错误：时间使用 TIMESTAMP
CREATE TABLE biz_order (
    create_time TIMESTAMP  -- 应用 VARCHAR2(17)
);

-- ❌ 错误：字段长度过大
CREATE TABLE sys_user (
    username VARCHAR2(500),  -- 用户名不需要500，32足够
    phone VARCHAR2(50)       -- 手机号固定11位，用 CHAR(11)
);

-- ❌ 错误：布尔值使用字符串
CREATE TABLE sys_user (
    is_deleted VARCHAR2(10)  -- 应用 NUMBER(1)
);
```

### 检查项

- [ ] 金额字段是否使用 NUMBER(n,2)
- [ ] 时间字段是否使用 VARCHAR2(17)
- [ ] 字段长度是否合理
- [ ] 布尔字段是否使用 NUMBER(1)
- [ ] 必填字段是否有 NOT NULL 约束
- [ ] 是否设置合理的默认值

---

## 规则 4: 索引设计规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- 主键自动创建聚簇索引
- 高频查询字段创建索引
- 联合索引遵循"最左前缀原则"
- 单表索引不超过 5 个
- **禁止**在低基数字段创建索引
- **禁止**索引字段使用函数

### 正确示例

```sql
-- ✅ 正确：高频查询字段创建索引
CREATE INDEX idx_order_user_id ON biz_order(user_id);
CREATE INDEX idx_order_order_no ON biz_order(order_no);

-- ✅ 正确：联合索引，高频字段在左侧
CREATE INDEX idx_order_user_status_time 
    ON biz_order(user_id, order_status, create_time);

-- ✅ 正确：唯一索引
CREATE UNIQUE INDEX uk_element_code ON pt_element(element_code);

-- ✅ 正确：分页查询配合排序字段
CREATE INDEX idx_order_create_time ON biz_order(create_time DESC);
```

### 错误示例

```sql
-- ❌ 错误：在低基数字段创建索引
CREATE INDEX idx_order_is_deleted ON biz_order(is_deleted);  -- 只有0和1
CREATE INDEX idx_order_status ON biz_order(order_status);    -- 只有几个值

-- ❌ 错误：过多索引
-- 单表超过5个索引会影响写入性能

-- ❌ 错误：索引字段使用函数
SELECT * FROM biz_order WHERE TRUNC(create_time) = '20240501';  -- 索引失效

-- ❌ 错误：联合索引顺序不当
-- 查询条件：WHERE order_status = 1 AND user_id = 100
CREATE INDEX idx_wrong ON biz_order(order_status, user_id);  -- 应该 user_id 在前
```

### 检查项

- [ ] 高频查询字段是否有索引
- [ ] 单表索引是否超过5个
- [ ] 联合索引顺序是否正确
- [ ] 是否在低基数字段创建索引
- [ ] WHERE 条件是否对索引字段使用函数

---

## 规则 5: SQL 编写规范

- **状态**: ENABLED
- **优先级**: HIGH
- **强制级别**: 【强制】

### 说明

- SQL 关键字**大写**，表名/字段名**小写**
- **禁止** `SELECT *`
- **禁止**字符串拼接 SQL（防注入）
- 分页查询必须包含 `ORDER BY`
- 批量操作单次不超过 1000 条

### 正确示例

```sql
-- ✅ 正确：关键字大写，字段明确
SELECT id, element_code, element_name, create_time
FROM pt_element
WHERE is_deleted = 0
  AND element_type = 1
ORDER BY create_time DESC;

-- ✅ 正确：分页查询带排序
SELECT id, order_no, total_amount, create_time
FROM biz_order
WHERE user_id = #{userId}
  AND order_status = #{status}
ORDER BY create_time DESC
OFFSET #{offset} ROWS FETCH NEXT #{pageSize} ROWS ONLY;

-- ✅ 正确：参数化查询（MyBatis）
<select id="selectByCondition" resultType="OrderEntity">
    SELECT id, order_no, user_id, total_amount, order_status, create_time
    FROM biz_order
    WHERE user_id = #{userId}
    <if test="status != null">
        AND order_status = #{status}
    </if>
    ORDER BY create_time DESC
</select>

-- ✅ 正确：批量插入控制数量
<insert id="batchInsert">
    INSERT INTO pt_element_value (id, element_id, value_code, value_name, create_time)
    <foreach collection="list" item="item" separator=" UNION ALL ">
        SELECT #{item.id}, #{item.elementId}, #{item.valueCode}, 
               #{item.valueName}, #{item.createTime}
        FROM DUAL
    </foreach>
</insert>
```

### 错误示例

```sql
-- ❌ 错误：使用 SELECT *
SELECT * FROM pt_element WHERE id = 1;

-- ❌ 错误：关键字小写（不规范）
select id, name from pt_element where id = 1;

-- ❌ 错误：字符串拼接（SQL注入风险）
<select id="selectByName">
    SELECT * FROM pt_element WHERE element_name = '${name}'
</select>

-- ❌ 错误：分页无排序
SELECT id, order_no FROM biz_order
OFFSET 0 ROWS FETCH NEXT 10 ROWS ONLY;  -- 缺少 ORDER BY

-- ❌ 错误：笛卡尔积
SELECT * FROM pt_element e, pt_element_value v;  -- 缺少 ON 条件

-- ❌ 错误：UPDATE/DELETE 无 WHERE
UPDATE biz_order SET order_status = 1;  -- 全表更新！
DELETE FROM biz_order;  -- 全表删除！
```

### 检查项

- [ ] SQL 关键字是否大写
- [ ] 是否使用 SELECT *
- [ ] 是否使用参数化查询（#{}）
- [ ] 分页查询是否有 ORDER BY
- [ ] UPDATE/DELETE 是否有 WHERE 条件
- [ ] 是否存在笛卡尔积

---

## 规则 6: 性能优化规范

- **状态**: ENABLED
- **优先级**: MEDIUM
- **强制级别**: 【强制】

### 说明

- 避免长事务，减少锁竞争
- 高频查询小表可适度冗余字段
- **禁止** TRUNCATE（数据不可恢复）
- 批量操作分批次执行

### 正确示例

```sql
-- ✅ 正确：分批删除
<delete id="batchDelete">
    DELETE FROM pt_element_value 
    WHERE id IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</delete>

-- ✅ 正确：使用 EXISTS 代替 IN（大数据量）
SELECT id, element_code, element_name
FROM pt_element e
WHERE EXISTS (
    SELECT 1 FROM pt_element_value v 
    WHERE v.element_id = e.id AND v.is_deleted = 0
);

-- ✅ 正确：避免 SELECT COUNT(*)
SELECT COUNT(1) FROM pt_element WHERE is_deleted = 0;

-- ✅ 正确：覆盖索引
CREATE INDEX idx_element_code_name ON pt_element(element_code, element_name);
-- 查询只需要 element_code 和 element_name 时，不需要回表
SELECT element_code, element_name FROM pt_element WHERE element_code = 'E001';
```

### 错误示例

```sql
-- ❌ 错误：使用 TRUNCATE
TRUNCATE TABLE pt_element;  -- 数据不可恢复，应用 DELETE

-- ❌ 错误：大数据量 IN 查询
SELECT * FROM pt_element 
WHERE id IN (SELECT element_id FROM pt_element_value);  -- 可能很慢

-- ❌ 错误：长事务
BEGIN;
-- 大量操作
-- ...长时间未提交
COMMIT;

-- ❌ 错误：一次性批量操作过多
INSERT INTO pt_element_value (...) 
VALUES (...), (...), ... -- 10000条一次插入
```

### 检查项

- [ ] 是否使用 TRUNCATE（应禁止）
- [ ] 批量操作是否分批（≤1000条）
- [ ] 大数据量是否使用 EXISTS 替代 IN
- [ ] 是否存在长事务
- [ ] 是否有不必要的全表扫描

---

## 规则 7: 安全规范

- **状态**: ENABLED
- **优先级**: CRITICAL
- **强制级别**: 【强制】

### 说明

- 敏感字段加密存储（密码、身份证、银行卡）
- 使用参数绑定，**禁止**字符串拼接
- 查询需关联数据权限

### 加密存储规范

| 敏感字段 | 加密方式 | 说明 |
|----------|----------|------|
| 密码 | BCrypt | 单向哈希，不可逆 |
| 身份证号 | AES | 对称加密，可解密 |
| 银行卡号 | AES | 对称加密，可解密 |
| 手机号 | AES/脱敏 | 按需选择 |

### 正确示例

```sql
-- ✅ 正确：参数化查询
<select id="selectById" resultType="UserEntity">
    SELECT id, username, phone, create_time
    FROM sys_user
    WHERE id = #{userId}
      AND is_deleted = 0
</select>

-- ✅ 正确：动态表名需白名单校验
<select id="selectByTable" resultType="map">
    SELECT * FROM ${tableName}
    WHERE is_deleted = 0
</select>
<!-- Java 代码中校验 tableName 在白名单内 -->

-- ✅ 正确：数据权限过滤
SELECT id, element_code, element_name
FROM pt_element
WHERE is_deleted = 0
  AND rg_code IN (
      SELECT rg_code FROM sys_user_data_scope WHERE user_id = #{currentUserId}
  );
```

### 错误示例

```sql
-- ❌ 错误：字符串拼接（SQL注入）
<select id="selectByName">
    SELECT * FROM sys_user WHERE username = '${username}'  -- 危险！
</select>

-- ❌ 错误：明文存储密码
INSERT INTO sys_user (username, password) VALUES ('admin', '123456');

-- ❌ 错误：无数据权限控制
SELECT * FROM biz_order;  -- 可查看所有订单，应加权限过滤
```

### 检查项

- [ ] 是否使用参数绑定（#{} 而非 ${}）
- [ ] 动态表名/字段名是否有白名单校验
- [ ] 敏感字段是否加密存储
- [ ] 查询是否有数据权限过滤

---

## 输出报告格式

检查完成后，请按以下格式输出报告：

```markdown
# 数据库规范检查报告

## 检查概览
- 检查路径：{path}
- 检查文件数：{count}
- 通过项：{pass_count}
- 警告项：{warn_count}
- 不通过项：{fail_count}

## 详细结果

### 1. 表命名规范
| 表名 | 检查项 | 状态 | 说明 |
|------|--------|------|------|
| pt_element | 命名规范 | PASS | 符合规范 |
| UserInfo | 命名规范 | FAIL | 应改为 user_info |

### 2. 字段规范
| 表名 | 字段名 | 状态 | 说明 |
|------|--------|------|------|
| pt_element | element_code | PASS | 符合规范 |
| biz_order | Data | FAIL | 命名模糊，应明确含义 |

### 3. SQL 编写
| 文件 | 检查项 | 状态 | 说明 |
|------|--------|------|------|
| UserMapper.xml | SELECT * | FAIL | 应明确字段列表 |
| OrderMapper.xml | 参数绑定 | PASS | 使用 #{} |

## 修复建议
1. [FAIL] UserInfo 表名应改为 user_info
2. [FAIL] UserMapper.xml 第15行 SELECT * 应改为具体字段
```

---

## 版本历史

| 版本 | 日期 | 说明 |
|------|------|------|
| v1.0 | 2026-03-12 | 初始版本，包含 7 条数据库规范规则 |
