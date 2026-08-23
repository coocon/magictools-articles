# JSON vs XML vs YAML：三大数据格式深度对比与选型指南

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/json-xml-yaml-comparison?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/json-xml-yaml-comparison?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：JSON vs XML vs YAML：三大数据格式深度对比与选型指南](https://tools.cooconsbit.com/zh/articles/json-xml-yaml-comparison?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
