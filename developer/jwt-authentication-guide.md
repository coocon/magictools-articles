# JWT 身份认证完全指南：原理、使用与安全最佳实践

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/jwt-authentication-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/jwt-authentication-guide?utm_source=github&utm_medium=referral)**

当你登录一个 Web 应用后，服务器如何知道你后续的每个请求都是"你"发出的？HTTP 是无状态协议，每次请求天然是独立的，没有"记忆"。解决这个问题有两种主流方案：**Session Token** 和 **JWT**。

JWT（JSON Web Token）近年来在 REST API 领域几乎成为标准选择，被 GitHub、Auth0、Google 等主流平台广泛采用。本文从原理到实践，系统讲解 JWT 的一切。

---

## 什么是 JWT？

JWT 是一种**紧凑、自包含的 Token 格式**，用于在各方之间安全地传输 JSON 格式的声明（Claims）信息。

"自包含"是 JWT 的核心特征：Token 本身携带了用户信息和验证信息，服务端无需查询数据库就能验证 Token 的合法性。这使得 JWT 天然适合无状态的分布式架构。

---

## JWT 结构详解

一个 JWT 由三部分组成，用 `.` 分隔：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IuW8oOS4iSIsImlhdCI6MTUxNjIzOTAyMn0
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### Part 1：Header（头部）

Base64URL 解码后：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

- `alg`：签名算法（HS256、RS256、ES256 等）
- `typ`：Token 类型，固定为 `JWT`

### Part 2：Payload（载荷）

Base64URL 解码后：

```json
{
  "sub": "1234567890",
  "name": "张三",
  "email": "zhangsan@example.com",
  "role": "admin",
  "iat": 1516239022,
  "exp": 1516325422
}
```

Payload 包含"声明"（Claims），分为三类：

**标准声明（Registered Claims）：**

| 字段 | 全称 | 含义 |
|------|------|------|
| `sub` | Subject | Token 主体（通常是用户 ID） |
| `iss` | Issuer | Token 签发者 |
| `aud` | Audience | Token 受众（目标服务） |
| `exp` | Expiration Time | 过期时间（Unix 时间戳） |
| `iat` | Issued At | 签发时间 |
| `nbf` | Not Before | 生效时间（之前无效） |

**自定义声明：** 可以加入任何业务需要的字段，如 `role`、`email`、`permissions` 等。

> ⚠️ **重要警告：Payload 是 Base64URL 编码，不是加密！任何人都可以解码查看内容。永远不要在 Payload 中存储密码、信用卡号等敏感数据。**

### Part 3：Signature（签名）

签名的计算方式：

```
HMACSHA256(
  base64UrlEncode(header) + "." + base64UrlEncode(payload),
  secret
)
```

以 HS256 为例，服务端用一个只有自己知道的 `secret` 对前两部分进行 HMAC-SHA256 哈希运算，生成签名。

**验证机制：** 服务端收到请求时，用同样的算法重新计算签名，与 Token 中的签名比对。只要 `secret` 不泄露，任何人篡改 Header 或 Payload 后，签名就会对不上——因为他们没有 `secret`，无法生成有效的签名。

---

## JWT vs Session Token 对比

| 维度 | JWT | Session Token |
|------|-----|--------------|
| 存储位置 | 客户端 | 服务端（数据库/Redis） |
| 服务端状态 | 无状态（Stateless） | 有状态（Stateful） |
| 扩展性 | 天然支持水平扩展 | 需要 Session 共享（Redis） |
| 注销实现 | 困难（需黑名单机制） | 简单（删除服务端 Session） |
| 性能 | 验证无需 DB 查询 | 每次需查询 Session 存储 |
| Token 大小 | 较大（通常 500B~2KB） | 很小（仅 Session ID） |
| 安全性 | 过期前无法吊销（默认） | 可立即吊销 |
| 适用场景 | 分布式、微服务、API | 传统 Web 应用 |

...

---

**[👉 继续阅读全文：JWT 身份认证完全指南：原理、使用与安全最佳实践](https://tools.cooconsbit.com/zh/articles/jwt-authentication-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
