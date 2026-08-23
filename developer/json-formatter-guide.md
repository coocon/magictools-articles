# JSON 格式化工具使用教程：格式化、压缩与验证技巧

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/json-formatter-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/json-formatter-guide?utm_source=github&utm_medium=referral)**

## 什么是 JSON？

JSON（JavaScript Object Notation）是一种轻量级的数据交换格式，由 Douglas Crockford 在 2001 年提出并推广。它基于 JavaScript 对象字面量语法，但已成为语言无关的通用标准（RFC 8259）。

**为什么 JSON 成为现代 API 的标准格式？**

- **可读性好**：比 XML 更简洁，人类可直接阅读和理解
- **解析简单**：几乎所有编程语言都有原生或高效的 JSON 库
- **体积小**：比 XML 省去大量冗余标签
- **类型丰富**：支持字符串、数字、布尔值、数组、对象、null

```json
{
  "name": "张三",
  "age": 28,
  "skills": ["Python", "TypeScript", "Go"],
  "active": true,
  "address": null
}
```

---

## 三种核心功能详解

### 1. 格式化（Pretty Print）

将压缩或混乱的 JSON 展开为层次清晰、缩进整齐的格式，便于人类阅读。

**压缩前：**
```json
{"name":"张三","age":28,"skills":["Python","TypeScript"],"address":{"city":"北京","district":"朝阳"}}
```

**格式化后（2 空格缩进）：**
```json
{
  "name": "张三",
  "age": 28,
  "skills": [
    "Python",
    "TypeScript"
  ],
  "address": {
    "city": "北京",
    "district": "朝阳"
  }
}
```

在线工具支持选择缩进方式：2 空格、4 空格或 Tab，根据团队规范选择即可。

### 2. 压缩（Minify）

移除所有不必要的空白字符（空格、换行、缩进），将 JSON 压缩为单行，减小传输体积。

- **适用场景**：API 响应体、嵌入 HTML 的配置数据、CDN 缓存的静态 JSON 文件
- **体积节省**：复杂 JSON 通常可减少 20%~40% 的字节数
- **注意**：压缩后不适合人工阅读，调试时请先格式化

### 3. 验证（Validate）

检测 JSON 的语法是否合法，并定位错误所在行号和列号。

当 JSON 存在语法错误时，工具会高亮显示错误位置，例如：

```
Error: Unexpected token '}' at line 5, column 3
Expected ',' or ']' after array element
```

---

## 常见 JSON 语法错误及修复

开发中最频繁出现的 JSON 错误：

### ❌ 属性名必须用双引号

...

---

**[👉 继续阅读全文：JSON 格式化工具使用教程：格式化、压缩与验证技巧](https://tools.cooconsbit.com/zh/articles/json-formatter-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
