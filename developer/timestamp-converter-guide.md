---
title: "时间戳转换工具完全指南：Unix 时间戳与日期时间互转"
slug: "timestamp-converter-guide"
category: developer
tags:
  - 时间戳
  - Unix时间
  - 开发者工具
  - 日期时间
summary: "Unix 时间戳是后端开发中处理时间的基础概念。本文详解时间戳的定义、秒级与毫秒级的区别、时区陷阱与 2038 年问题，并提供 JavaScript、Python、Go 等语言的代码示例，帮助开发者正确处理时间相关逻辑。"
coverImage: ""
status: published
scheduledAt: ""
---

## 什么是 Unix 时间戳？

**Unix 时间戳**（Unix Timestamp，也称 POSIX 时间、Epoch 时间）是从 **1970 年 1 月 1 日 00:00:00 UTC** 到某一时刻的**总秒数**。

这个起始时间点被称为 **Unix 纪元**（Unix Epoch），是由早期 Unix 系统开发者选定的任意基准点，后来成为全球通用标准。

```
1970-01-01 00:00:00 UTC  →  时间戳 0
2026-03-18 10:00:00 UTC  →  时间戳 1742292000
```

时间戳的优势在于：
- **语言无关**：任何编程语言都能处理一个整数
- **时区无关**：时间戳始终基于 UTC，不受时区影响
- **便于计算**：两个时间戳相减直接得到秒数差值
- **存储高效**：4~8 字节即可表示任意时间点

---

## 毫秒时间戳 vs 秒时间戳

这是开发中最容易踩的坑之一：**不同平台使用不同的时间单位**。

| 平台/语言 | 默认单位 | 典型值（2026年） |
|----------|---------|--------------|
| JavaScript / Node.js | **毫秒** | `1742292000000` |
| Unix 系统命令 (`date +%s`) | 秒 | `1742292000` |
| Python `time.time()` | 秒（浮点） | `1742292000.123` |
| Go `time.Now().Unix()` | 秒 | `1742292000` |
| Java `System.currentTimeMillis()` | **毫秒** | `1742292000000` |
| MySQL `UNIX_TIMESTAMP()` | 秒 | `1742292000` |
| Redis `TIME` 命令 | 秒 | `1742292000` |

**快速判断方法**：2026 年的时间戳，秒级约为 **17 亿**（10位数），毫秒级约为 **17000 亿**（13位数）。当你看到一个 13 位的"时间戳"，它一定是毫秒。

互换公式：
```javascript
// 秒 → 毫秒
const ms = seconds * 1000;

// 毫秒 → 秒（取整）
const s = Math.floor(milliseconds / 1000);
```

---

## 在线工具核心功能

访问 [MagicTools 时间戳工具](/tools/timestamp)，支持以下操作：

### 1. 实时当前时间戳

工具主页实时显示当前时间戳（秒级和毫秒级同时展示），每秒自动刷新，方便调试时对比。

### 2. 时间戳 → 日期时间

输入任意 Unix 时间戳（支持秒和毫秒自动识别），选择目标时区，工具输出对应的格式化日期时间：

```
输入：1742292000
时区：Asia/Shanghai (UTC+8)
输出：2026-03-18 18:00:00
```

### 3. 日期时间 → 时间戳

选择时区，填写日期和时间，工具输出对应的 Unix 时间戳：

```
输入：2026-03-18 18:00:00
时区：Asia/Shanghai (UTC+8)
输出：1742292000（秒）/ 1742292000000（毫秒）
```

### 4. 时间差计算

输入两个时间戳，工具计算差值并以多种单位展示：

```
开始：1742292000
结束：1742378400
差值：86400 秒 = 1440 分钟 = 24 小时 = 1 天
```

---

## 多语言代码示例

### JavaScript / Node.js

```javascript
// 获取当前时间戳
Date.now()                           // 毫秒：1742292000000
Math.floor(Date.now() / 1000)        // 秒：1742292000

// 时间戳 → 日期对象
const date = new Date(1742292000 * 1000);
console.log(date.toISOString());     // 2026-03-18T10:00:00.000Z
console.log(date.toLocaleString('zh-CN', {
  timeZone: 'Asia/Shanghai'
}));                                 // 2026/3/18 18:00:00

// 日期字符串 → 时间戳
const ts = new Date('2026-03-18T10:00:00Z').getTime() / 1000;
console.log(ts);                     // 1742292000
```

### Python

```python
import time
from datetime import datetime, timezone, timedelta

# 获取当前时间戳
int(time.time())                     # 秒：1742292000
time.time_ns() // 1_000_000          # 毫秒：1742292000000

# 时间戳 → datetime（UTC）
dt_utc = datetime.fromtimestamp(1742292000, tz=timezone.utc)
print(dt_utc)                        # 2026-03-18 10:00:00+00:00

# 时间戳 → datetime（北京时间 UTC+8）
tz_shanghai = timezone(timedelta(hours=8))
dt_sh = datetime.fromtimestamp(1742292000, tz=tz_shanghai)
print(dt_sh)                         # 2026-03-18 18:00:00+08:00

# datetime → 时间戳
ts = dt_utc.timestamp()
print(int(ts))                       # 1742292000
```

### Go

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // 获取当前时间戳
    fmt.Println(time.Now().Unix())        // 秒
    fmt.Println(time.Now().UnixMilli())   // 毫秒
    fmt.Println(time.Now().UnixNano())    // 纳秒

    // 时间戳 → time.Time
    t := time.Unix(1742292000, 0)
    fmt.Println(t.UTC())                  // 2026-03-18 10:00:00 +0000 UTC

    // 北京时间
    loc, _ := time.LoadLocation("Asia/Shanghai")
    fmt.Println(t.In(loc))               // 2026-03-18 18:00:00 +0800 CST

    // time.Time → 时间戳
    ts := time.Date(2026, 3, 18, 10, 0, 0, 0, time.UTC).Unix()
    fmt.Println(ts)                       // 1742292000
}
```

### Shell 命令

```bash
# 获取当前时间戳（Linux/macOS）
date +%s

# 时间戳 → 日期（Linux GNU date）
date -d @1742292000 "+%Y-%m-%d %H:%M:%S"

# 时间戳 → 日期（macOS BSD date）
date -r 1742292000 "+%Y-%m-%d %H:%M:%S"
```

---

## 常见陷阱与注意事项

### 陷阱一：时区混乱

这是最常见的 Bug 来源。时间戳本身是 UTC 无时区的，但将其转换为可读日期时，必须明确指定时区：

```javascript
// ❌ 危险：依赖服务器本地时区，不同环境结果不同
new Date(timestamp * 1000).toString()

// ✅ 安全：明确指定 UTC
new Date(timestamp * 1000).toISOString()

// ✅ 安全：明确指定目标时区
new Date(timestamp * 1000).toLocaleString('zh-CN', {
  timeZone: 'Asia/Shanghai'
})
```

**黄金法则**：数据库和 API 传输始终使用 UTC 时间戳或 ISO 8601 字符串（含时区标记），只在**最终展示给用户**时转换为本地时间。

### 陷阱二：2038 年问题

32 位有符号整数最大值是 `2,147,483,647`，对应 **2038 年 1 月 19 日 03:14:07 UTC**。使用 32 位整数存储时间戳的旧系统，在那之后将溢出归零（回到 1970 年）。

**现代系统应使用 64 位整数**存储时间戳。JavaScript 的 `Number` 类型为 64 位浮点数，可安全表示到公元 275760 年。MySQL 的 `TIMESTAMP` 类型（32位）有此限制，应改用 `DATETIME` 或 `BIGINT` 存储。

### 陷阱三：夏令时（DST）

部分国家实行夏令时，在夏令时切换时刻，同一个本地时间可能对应两个不同的 UTC 时间（或反过来）。始终用 UTC 时间戳存储，用 IANA 时区数据库（`Asia/Shanghai`、`America/New_York`）转换，可避免夏令时问题。

---

## 常见问题 FAQ

**Q：数据库存时间应该用时间戳还是日期字符串？**

A：推荐存 **Unix 时间戳（整数）或 `DATETIME`（UTC）**，不推荐存字符串。理由：① 时间戳索引和范围查询效率高；② 不受字符集影响；③ 语言和框架都能直接解析整数。如果用 MySQL，建议 `BIGINT` 存毫秒时间戳（兼容 JS），或 `DATETIME` 存 UTC 时间。避免 `TIMESTAMP` 类型（有 2038 问题，且受服务器时区影响）。

**Q：如何处理跨时区的时间比较？**

A：将所有时间转换为 UTC 时间戳后再比较。时间戳是单调递增的，`ts_a > ts_b` 就代表 A 时刻晚于 B 时刻，不受时区影响。永远不要直接比较两个未标注时区的本地时间字符串。

**Q：`1970-01-01` 之前的时间怎么表示？**

A：Unix 时间戳支持负数。`-1` 表示 1969-12-31 23:59:59 UTC。现代系统和编程语言均支持负数时间戳，可以表示 1970 年之前的历史时间。

**Q：为什么 `new Date()` 在不同时区的服务器上返回不同的字符串？**

A：`new Date().toString()` 使用的是**运行环境的本地时区**，服务器时区不同，输出自然不同。使用 `new Date().toISOString()` 始终返回 UTC 格式（`2026-03-18T10:00:00.000Z`），在任何时区都一致。

**Q：JavaScript 的 `Date.now()` 精度够用吗？**

A：`Date.now()` 精度为毫秒（ms），对大多数业务场景足够。需要更高精度（如性能测量）时，使用 `performance.now()`（微秒精度）。注意：出于安全考虑，浏览器会对 `performance.now()` 的精度做一定模糊处理。

---

## 小结

Unix 时间戳是后端开发的基础知识，正确使用能避免大量时间相关的 Bug：

- **单位区分**：13 位 = 毫秒（JS/Java），10 位 = 秒（Python/Go/MySQL）
- **时区原则**：存储用 UTC，展示时转本地时区
- **2038 警惕**：新系统避免 MySQL `TIMESTAMP`，改用 `DATETIME` 或 `BIGINT`
- **比较时间**：转换为时间戳后用数值比较，不要比较字符串

遇到时间戳转换需求，直接使用 [MagicTools 时间戳工具](/tools/timestamp) 快速完成，无需手算。
