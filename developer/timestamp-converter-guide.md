# 时间戳转换工具完全指南：Unix 时间戳与日期时间互转

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/timestamp-converter-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/timestamp-converter-guide?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：时间戳转换工具完全指南：Unix 时间戳与日期时间互转](https://tools.cooconsbit.com/zh/articles/timestamp-converter-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
