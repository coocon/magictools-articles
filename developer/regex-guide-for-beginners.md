# 正则表达式入门完全指南：10 个实用例子从零学会

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/regex-guide-for-beginners?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/regex-guide-for-beginners?utm_source=github&utm_medium=referral)**

第一次看到正则表达式时，很多开发者的第一反应是："这是什么乱码？"

```
^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[a-zA-Z\d@$!%*?&]{8,}$
```

但当你真正理解正则之后，你会发现：曾经需要写 30 行代码的文本处理，现在一行搞定。表单验证、日志分析、代码重构、数据提取——正则表达式无处不在。

本文用最直观的方式带你入门，并给出 10 个直接可以复制使用的实战正则。

---

## 基础语法快速上手

### 字符类

| 语法 | 含义 | 示例 |
|------|------|------|
| `.` | 任意单个字符（除换行符） | `a.c` 匹配 `abc`、`a1c` |
| `\d` | 数字，等同于 `[0-9]` | `\d\d` 匹配 `42` |
| `\D` | 非数字 | `\D+` 匹配 `abc` |
| `\w` | 字母/数字/下划线，等同于 `[a-zA-Z0-9_]` | `\w+` 匹配 `hello_world` |
| `\W` | 非单词字符 | `\W` 匹配空格、标点 |
| `\s` | 空白字符（空格、Tab、换行） | `\s+` 匹配连续空格 |
| `[abc]` | 字符集合，匹配 a、b 或 c | `[aeiou]` 匹配元音 |
| `[^abc]` | 否定字符集，不匹配 a、b、c | `[^0-9]` 匹配非数字 |
| `[a-z]` | 字符范围，匹配 a 到 z | `[A-Za-z]` 匹配所有字母 |

### 量词

| 语法 | 含义 | 示例 |
|------|------|------|
| `*` | 0 个或多个 | `ab*c` 匹配 `ac`、`abc`、`abbc` |
| `+` | 1 个或多个 | `ab+c` 匹配 `abc`、`abbc`，不匹配 `ac` |
| `?` | 0 个或 1 个（可选） | `colou?r` 匹配 `color` 和 `colour` |
| `{n}` | 恰好 n 个 | `\d{4}` 匹配 4 位数字 |
| `{n,m}` | n 到 m 个 | `\d{6,8}` 匹配 6~8 位数字 |
| `{n,}` | 至少 n 个 | `\d{3,}` 匹配 3 位及以上数字 |

### 锚点

| 语法 | 含义 |
|------|------|
| `^` | 字符串开头 |
| `$` | 字符串结尾 |
| `\b` | 单词边界（单词与非单词的交界处） |

### 分组与捕获

| 语法 | 含义 |
|------|------|
| `(...)` | 捕获组，可通过 `$1`、`\1` 反向引用 |
| `(?:...)` | 非捕获组，仅分组不捕获 |
| `(?P<name>...)` | 命名捕获组（Python/PCRE 语法） |
| `(?<name>...)` | 命名捕获组（JavaScript ES2018+ 语法） |

### 其他

| 语法 | 含义 |
|------|------|
| `a\|b` | 或：匹配 a 或 b |
| `\` | 转义字符，`\.` 匹配字面量点号 |
| `(?=...)` | 正向前瞻（后面必须跟着...） |
| `(?!...)` | 负向前瞻（后面不能跟着...） |

---

## 10 个实用正则表达式

### 1. 验证邮箱地址

```regex
^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$
```

**说明**：本地部分允许字母、数字和 `._%+-`，`@` 后为域名，顶级域名至少 2 个字母。

```javascript
// JavaScript 示例
const emailRegex = /^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$/;
emailRegex.test('user@example.com');    // true
emailRegex.test('invalid.email');       // false
emailRegex.test('a@b.c');              // false（TLD 太短）
```

...

---

**[👉 继续阅读全文：正则表达式入门完全指南：10 个实用例子从零学会](https://tools.cooconsbit.com/zh/articles/regex-guide-for-beginners?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
