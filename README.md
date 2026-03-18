# CodeRefactoring-QoderRules

代码重构规则库 - 为 Qoder Skill 提供标准化的代码检查和重构规则

## 项目结构

```
CodeRefactoring-QoderRules/
├── README.md                     # 项目说明文档
├── java-rule/                    # Java 代码规范
│   └── v1.0.0/
│       └── java-rule.zh-CN.md     # Java 代码规范 v1.0.0（基于阿里 Java 开发手册）
|
|
├── sql-rule/                     # SQL 数据库规范
│   └── v1.0.0/
│       └── sql-conventions.zh-CN.md  # SQL 编程规范 v1.0.0
|
|
├── web-rule/                     # Web 前端规范
│   └── v1.0.0/
│       └── vue3-rule.zh-CN.md          # Vue 3 代码规范 v1.0.0

```

## 文件说明

| 路径 | 描述 |
|------|------|
| `java-rule/v1.0.0/java-rule.zh-CN.md` | Java 代码规范，包含命名、注释、日志、异常、集合、SQL 等规则 |
| `sql-rule/v1.0.0/sql-conventions.zh-CN.md` | SQL 编程规范，包含数据库设计、SQL 编写、索引优化等约定 |
| `web-rule/v1.0.0/vue3-rule.zh-CN.md` | Vue 3 前端代码规范，包含组件开发、状态管理等规则 |


## 使用说明

1. 查看各技术栈的规范文档
2. 在 `.qoder/` 目录下配置智能体和技能
3. Qoder Skill 根据规范文件自动进行代码检查和重构

---

**维护者**: guomaolin0517  
**更新时间**: 2026-03-11