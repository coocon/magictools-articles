# 密码安全完全指南：从创建到管理的最佳实践

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/password-security-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/password-security-guide?utm_source=github&utm_medium=referral)**

2023 年，NordPass 发布的年度报告显示，全球最常用密码榜首依然是 `123456`，使用人数超过 400 万。同年，Have I Been Pwned 数据库收录的泄露账号已突破 120 亿条。密码安全不是"专业人士才需要关心"的问题——每一个有网络账号的人，都是潜在的目标。

本文不谈枯燥的理论，只讲实用的方法：为什么你的密码不安全，以及如何系统性地改变这一现状。

## 常见密码攻击方式

了解攻击才能有效防御。以下是黑客最常用的五种密码攻击手段：

### 1. 暴力破解（Brute Force）

攻击者用程序枚举所有可能的字符组合，直到找到正确密码。现代 GPU 每秒可以尝试数十亿次组合。

**防护**：使用足够长的密码。密码长度每增加一位，破解时间呈指数级增长。

### 2. 字典攻击（Dictionary Attack）

不枚举所有组合，而是优先尝试常见词汇、常见密码列表（`rockyou.txt` 包含 1400 万个真实泄露密码）。

**防护**：避免使用完整的单词、姓名、生日。`Password@2024` 虽然"看起来复杂"，但在字典攻击面前形同虚设。

### 3. 彩虹表攻击（Rainbow Table）

针对数据库泄露场景。如果网站只存储密码的 MD5/SHA1 哈希值，攻击者可以用预计算的哈希表反查明文。

**防护**：这一点你无法直接控制，但可以选择知名大平台（使用 bcrypt、Argon2 等加盐哈希）。你能做的是：每个网站用唯一密码，即便一处泄露，其他账号不受影响。

### 4. 钓鱼攻击（Phishing）

伪造银行、邮件、社交平台登录页面，诱骗用户输入真实密码。这是目前最高效的攻击手段，技术再好的密码也无法防范。

**防护**：开启双因素认证（2FA）；使用密码管理器（自动填充功能不会在钓鱼网站上工作，因为域名不匹配）。

### 5. 撞库攻击（Credential Stuffing）

利用已泄露的账号密码，批量尝试登录其他网站。由于大量用户在多个网站使用相同密码，成功率极高。

**防护**：**每个网站使用唯一密码**，这是最重要的一条原则。

---

## 密码强度与破解时间

| 密码类型 | 示例 | 理论破解时间（现代GPU） |
|---------|------|----------------------|
| 6位纯数字 | `123456` | < 1秒 |
| 8位纯小写字母 | `password` | < 1分钟 |
| 8位大小写+数字 | `Pass1234` | 约 1小时 |
| 12位大小写+数字+符号 | `X#7kP!m2qR@v` | 约 3000年 |
| 16位随机字符 | `tK9@mV#2pQ!xL&nR` | 数亿年 |
| 4个随机单词（Passphrase）| `correct-horse-battery-staple` | 数百年（且好记）|

...

---

**[👉 继续阅读全文：密码安全完全指南：从创建到管理的最佳实践](https://tools.cooconsbit.com/zh/articles/password-security-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
