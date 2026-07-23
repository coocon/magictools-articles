---
title: "JSON vs XML vs YAML：三大数据格式深度对比与选型指南"
slug: "json-xml-yaml-comparison"
category: developer
tags:
  - json
  - xml
  - yaml
  - 数据格式
  - API设计
summary: "JSON、XML、YAML 是开发中最常见的三种数据格式，各有适用场景。本文通过深度对比语法、可读性、生态工具和典型使用场景，帮你做出最合适的技术选型决策。"
coverImage: ""
status: published
scheduledAt: ""
---

在现代软件开发中，数据的序列化与传输无处不在：前后端交互、配置文件管理、服务间通信……每一个环节都需要选择合适的数据格式。选错了，轻则可读性差、维护困难，重则引发安全漏洞或兼容性问题。

JSON、XML 和 YAML 是当今最主流的三种数据格式。它们各自有鲜明的设计哲学和适用场景，了解它们的本质区别，才能在项目中做出正确的选型。

## 三格式横向对比

| 维度 | JSON | XML | YAML |
|------|------|-----|------|
| 发布年份 | 2001 | 1998 | 2001 |
| 可读性 | 良好 | 较差（标签冗余） | 最佳 |
| 体积 | 小 | 大（标签开销大） | 小 |
| 注释支持 | 不支持 | 支持 | 支持 |
| 类型系统 | 基础（string/number/bool/null/array/object） | 无原生类型（需 Schema） | 丰富（含日期、多行字符串等） |
| Schema 验证 | JSON Schema | XSD（XML Schema） | 无标准，依赖工具 |
| 流行生态 | REST API、NoSQL、前端 | SOAP、企业系统、SVG | K8s、Docker、CI/CD |
| 解析难度 | 简单 | 复杂 | 中等（坑多） |
| 流式解析 | 支持（streaming JSON） | 支持（SAX） | 不适合 |

---

## JSON：互联网时代的通用语言

JSON（JavaScript Object Notation）由 Douglas Crockford 在 2001 年提出，脱胎于 JavaScript 对象字面量语法。它的设计哲学极其简洁：只保留最核心的数据类型，去掉一切多余的东西。

```json
{
  "user": {
    "id": 1024,
    "name": "张三",
    "roles": ["admin", "editor"],
    "active": true,
    "metadata": null
  }
}
```

**JSON 的优势：**

- **生态最成熟**：几乎所有编程语言都内置 JSON 解析，`JSON.parse()` / `json.loads()` 随手可用
- **体积相对较小**：相比 XML，无冗余的闭合标签
- **与 JavaScript 天然契合**：前端项目零成本使用
- **REST API 事实标准**：GitHub、Twitter、Stripe 等所有主流 API 均使用 JSON

**JSON 的局限：**

- **不支持注释**：配置文件场景体验较差（JSONC/JSON5 弥补了这一点）
- **不支持多行字符串**：长文本需要手动拼接 `\n`
- **类型有限**：没有日期类型，日期只能用字符串表示（如 ISO 8601）
- **严格语法**：末尾逗号、单引号均会导致解析错误

---

## XML：企业级系统的老兵

XML（eXtensible Markup Language）于 1998 年由 W3C 发布，设计目标是"人机均可读的结构化文档"。它的设计思想来自 SGML，强调文档的结构化表达和可扩展性。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<user id="1024">
  <name>张三</name>
  <roles>
    <role>admin</role>
    <role>editor</role>
  </roles>
  <active>true</active>
  <!-- 元数据为空 -->
  <metadata/>
</user>
```

**XML 的优势：**

- **Schema 验证体系完整**：XSD（XML Schema Definition）提供强类型验证
- **命名空间（Namespace）**：避免多来源数据的字段冲突
- **注释支持**：`<!-- 这是注释 -->`
- **XSLT 转换**：可直接转换为 HTML、PDF 等格式
- **成熟的流式解析**：SAX 解析器可处理超大文件

**XML 的适用场景：**

- **SOAP Web Services**：金融、政务系统的 B2B 集成
- **配置文件**：Maven（pom.xml）、Android 布局、Spring 配置
- **文档格式**：Microsoft Office（.docx 本质是 ZIP+XML）、SVG 矢量图
- **RSS/Atom 订阅源**

**XML 的缺点：**

- **极度冗余**：同样的数据，XML 体积通常是 JSON 的 2~3 倍
- **解析复杂**：DOM/SAX/StAX 等多种解析方式，学习成本高
- **新项目很少选择**：现代 REST API 生态基本已被 JSON 取代

---

## YAML：人类友好，但陷阱暗藏

YAML（YAML Ain't Markup Language，递归缩写）于 2001 年发布，核心设计理念是"对人类最友好的序列化格式"。它被广泛用于配置文件，尤其是云原生领域。

```yaml
# 这是注释
user:
  id: 1024
  name: 张三
  roles:
    - admin
    - editor
  active: true
  metadata: ~   # null 的另一种写法
  bio: |
    这是一段
    多行文本
    完美保留格式
```

**YAML 的优势：**

- **最高的人类可读性**：无引号、无括号、无逗号，清爽简洁
- **支持注释**：`#` 开头
- **多行字符串**：`|`（保留换行）和 `>`（折叠换行）
- **支持引用与锚点**：避免重复配置（`&anchor` / `*alias`）
- **YAML 是 JSON 的超集**：合法的 JSON 同时也是合法的 YAML（1.2 规范）

**YAML 常见陷阱（重点警示）：**

### 陷阱1：Norway Problem（挪威问题）

```yaml
# YAML 1.1 中，这些都会被解析为布尔值！
country: NO   # → false（挪威的国家代码！）
enabled: yes  # → true
toggle: on    # → true
flag: off     # → false
```

YAML 1.2 规范修复了这个问题，但很多库仍使用 1.1 规范。**解决方案：字符串值加引号** `country: "NO"`

### 陷阱2：冒号后必须有空格

```yaml
# 错误
key:value    # 解析失败或作为字符串处理

# 正确
key: value
```

### 陷阱3：缩进必须一致（不能混用 Tab 和空格）

```yaml
# 错误（混用 Tab 和空格）
parent:
    child1: a   # 4空格
	child2: b   # Tab，解析错误
```

### 陷阱4：特殊值自动类型转换

```yaml
version: 1.0      # 解析为数字 1.0，而非字符串 "1.0"
port: 8080        # 数字
api_key: 123456   # 数字，可能导致问题
```

---

## 实际场景选型建议

| 场景 | 推荐格式 | 理由 |
|------|---------|------|
| REST API 请求/响应 | **JSON** | 标准、轻量、生态最好 |
| Kubernetes 配置 | **YAML** | 官方支持，可读性强 |
| Docker Compose | **YAML** | 标准格式 |
| GitHub Actions / GitLab CI | **YAML** | 行业标准 |
| SOAP 企业服务 | **XML** | 协议要求 |
| Maven/Android 构建配置 | **XML** | 生态约定 |
| Hugo/Jekyll 静态站点配置 | **YAML / TOML** | 前者更通用 |
| 需要注释的 JSON 配置 | **JSONC 或 JSON5** | 如 VS Code settings.json |
| 数据库导入导出 | **JSON / CSV** | 工具链支持好 |

---

## 新兴替代格式简介

**TOML（Tom's Obvious Minimal Language）**：Rust 生态的配置格式，语法比 YAML 更严格直观，无隐式类型转换，适合配置文件。Cargo.toml、pyproject.toml 均采用此格式。

**JSON5**：JSON 的超集，支持注释、末尾逗号、单引号、多行字符串，保持与 JSON 的最大兼容性。

**JSONC**：带注释的 JSON，VS Code 配置文件（`settings.json`、`tsconfig.json`）均采用此格式。

---

## FAQ

**Q：JSON 能完全替代 XML 吗？**

大多数场景可以，但并非全部。XML 在以下场景仍不可替代：命名空间（解决字段冲突）、XSLT 转换、SOAP 协议（协议层面强制 XML）、以及需要完整 Schema 验证的企业系统。如果你在做新项目且没有历史包袱，REST + JSON 是更好的选择。

**Q：YAML 真的是 JSON 的超集吗？**

从 YAML 1.2 规范起，理论上合法的 JSON 也是合法的 YAML，所以说"YAML 是 JSON 的超集"。但在实践中，很多 YAML 解析库仍基于 1.1 规范，存在细微差异，不应完全依赖这一特性。

**Q：哪种格式解析速度最快？**

从纯性能看：**JSON > YAML > XML**。JSON 结构简单、解析器高度优化；YAML 因为类型推断和锚点引用，解析复杂；XML 因 DOM 树构建开销大，在大文件场景下最慢（SAX 模式可缓解）。在实际业务中，解析速度很少是瓶颈，格式选型更应关注可维护性和生态支持。

**Q：配置文件该用 YAML 还是 TOML？**

如果你的项目是 Rust 生态或追求严格类型安全，选 TOML。如果你的配置文件需要与 Kubernetes/Docker/CI 工具链兼容，选 YAML。如果团队对 YAML 陷阱不熟悉且配置较复杂，TOML 是更安全的选择。

---

## 总结

三种格式各有其历史使命：**XML** 是企业级系统的可靠基石，**JSON** 是互联网 API 的通用货币，**YAML** 是云原生时代的配置王者。

选型的核心原则：**不要为了用新技术而换格式**。看你的使用场景、团队熟悉度和生态工具链支持，选择最适合的那个。遇到 YAML，多加一点小心——那些隐式类型转换的陷阱，每年都在给工程师们"惊喜"。
