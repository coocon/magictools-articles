---
title: "JWT 身份认证完全指南：原理、使用与安全最佳实践"
slug: "jwt-authentication-guide"
category: developer
tags:
  - jwt
  - 身份认证
  - API安全
  - Token
summary: "JWT（JSON Web Token）是现代 API 身份认证的主流方案。本文深度解析 JWT 的三段式结构、签名验证原理、与 Session 的对比，以及存储位置选择、过期刷新机制、算法混淆漏洞等关键安全实践。"
coverImage: ""
status: published
scheduledAt: ""
---

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

---

## JWT 完整使用流程

```
1. 用户登录（POST /auth/login，发送账号密码）
         ↓
2. 服务端验证凭据，生成 JWT
   - Header: { alg: "HS256", typ: "JWT" }
   - Payload: { sub: "user_123", role: "admin", exp: now+7days }
   - 用 secret 签名
         ↓
3. 服务端返回 JWT 给客户端
         ↓
4. 客户端存储 JWT（LocalStorage 或 Cookie）
         ↓
5. 客户端每次请求携带 JWT
   Authorization: Bearer <token>
         ↓
6. 服务端验证 JWT
   - 检查签名是否合法
   - 检查 exp 是否过期
   - 从 Payload 提取用户信息，处理业务逻辑
```

服务端生成 JWT 的代码示例（Node.js）：

```javascript
import jwt from 'jsonwebtoken';

// 生成 JWT
function generateToken(userId, role) {
  return jwt.sign(
    {
      sub: userId,
      role: role,
    },
    process.env.JWT_SECRET,
    {
      expiresIn: '7d',    // 7 天过期
      issuer: 'myapp.com',
    }
  );
}

// 验证 JWT（中间件）
function verifyToken(req, res, next) {
  const authHeader = req.headers.authorization;
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Missing token' });
  }

  const token = authHeader.slice(7);
  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET);
    req.user = payload;
    next();
  } catch (err) {
    return res.status(401).json({ error: 'Invalid or expired token' });
  }
}
```

---

## Token 存储位置的安全分析

这是 JWT 中最有争议的话题之一：

### localStorage / sessionStorage

```javascript
// 存储
localStorage.setItem('token', jwt);

// 使用
const token = localStorage.getItem('token');
fetch('/api/data', {
  headers: { Authorization: `Bearer ${token}` }
});
```

**优点**：简单直接，可跨子域访问
**缺点**：存在 XSS 攻击风险——如果页面有 XSS 漏洞，攻击者可以通过注入脚本读取 localStorage 中的 Token

### HttpOnly Cookie（推荐）

```javascript
// 服务端设置 Cookie（Node.js/Express）
res.cookie('access_token', jwt, {
  httpOnly: true,   // JavaScript 无法读取，防 XSS
  secure: true,     // 仅 HTTPS 传输
  sameSite: 'strict', // 防 CSRF
  maxAge: 7 * 24 * 60 * 60 * 1000, // 7 天
});
```

**优点**：JavaScript 无法读取（防 XSS），浏览器自动携带
**缺点**：需要处理 CSRF（用 `sameSite: 'strict'` 或 CSRF Token 防护）；不适合移动端 App

**结论**：Web 应用优先使用 HttpOnly Cookie；移动端 App 或第三方 API 客户端使用 localStorage（同时严格防范 XSS）。

---

## 安全注意事项

### 1. 设置合理的过期时间

```javascript
// Access Token：短生命周期（15分钟~1小时）
jwt.sign(payload, secret, { expiresIn: '15m' });

// Refresh Token：较长生命周期（7~30天），存储在服务端
jwt.sign({ sub: userId }, refreshSecret, { expiresIn: '30d' });
```

### 2. Refresh Token 机制

Access Token 设置短过期时间，用 Refresh Token 换取新的 Access Token，在安全与用户体验之间取得平衡：

```
Access Token 过期（返回 401）
         ↓
客户端用 Refresh Token 请求 /auth/refresh
         ↓
服务端验证 Refresh Token（查数据库，确认未被吊销）
         ↓
返回新的 Access Token（+ 可选：轮换 Refresh Token）
```

### 3. 使用 HTTPS

JWT 在 Header 中明文传输，没有 HTTPS 就是裸奔。

### 4. 警惕算法混淆攻击（`alg: none` 漏洞）

早期部分 JWT 库存在严重漏洞：攻击者将 Header 中的 `alg` 修改为 `none`，使库跳过签名验证。

```json
// 攻击者篡改的 Header
{ "alg": "none", "typ": "JWT" }
```

**防护**：使用最新版本的 JWT 库；在验证时**显式指定允许的算法**：

```javascript
// ✅ 显式指定算法，拒绝其他算法
jwt.verify(token, secret, { algorithms: ['HS256'] });
```

### 5. HS256 vs RS256 的选择

| | HS256 | RS256 |
|--|-------|-------|
| 算法 | HMAC-SHA256（对称） | RSA-SHA256（非对称） |
| 签名密钥 | 单一 secret | 私钥签名 / 公钥验证 |
| 验证方式 | 需要持有 secret | 只需公钥，公钥可公开 |
| 适合场景 | 单服务、内部系统 | 微服务、第三方验证（如 Auth0） |

---

## FAQ

**Q：JWT 能注销吗？**

这是 JWT 的天然局限。由于 JWT 是无状态的，服务端默认不维护 Token 状态，Token 在 `exp` 到期前始终有效。实现注销有两种方案：

1. **Token 黑名单**：注销时将 Token 的 `jti`（JWT ID）存入 Redis，验证时检查黑名单。缺点：引入了状态，增加了 Redis 查询，部分抵消了 JWT 的无状态优势。
2. **短过期 + Refresh Token 轮换**：Access Token 15 分钟过期，注销时吊销 Refresh Token（服务端删除），用户最多在 15 分钟内仍可访问，但无法续期。这是业界最常用的折中方案。

**Q：JWT 和 OAuth 是什么关系？**

OAuth 2.0 是一套**授权框架**（Authorization Framework），定义了客户端如何获取访问权限的流程（授权码流程、客户端凭据流程等）。JWT 是一种**Token 格式**。两者是不同维度的概念，可以叠加使用：OAuth 2.0 颁发的 Access Token 可以是 JWT 格式（如 Auth0、Google 的实现），也可以是随机字符串（如 GitHub 的 Personal Access Token）。简单说：**OAuth 定义"怎么拿 Token"，JWT 定义"Token 长什么样"**。

**Q：HS256 和 RS256 怎么选？**

如果你是**单体应用或内部服务**，选 HS256：实现简单，性能高，管理一个 secret 即可。

如果你的架构是**微服务**，或者需要让**第三方服务验证你的 Token**（如合作伙伴的系统），选 RS256：认证服务持有私钥签名，其他服务只需公钥即可验证，不需要共享私密信息，更安全。Auth0、Google、Apple 等 IdP（身份提供商）均使用 RS256/ES256。

---

## 总结

JWT 本质上是一种将用户状态"嵌入 Token"的设计——它以牺牲"实时吊销"能力为代价，换取无状态验证带来的水平扩展能力。

三条核心原则记住就够：

1. **Payload 不放敏感数据**（Base64 不是加密）
2. **Access Token 短过期 + Refresh Token 机制**（平衡安全与体验）
3. **Web 端用 HttpOnly Cookie 存储**（防 XSS 的第一道防线）

在合适的场景选择 JWT，在需要实时吊销的场景（如政务、金融）考虑 Session + Redis 的有状态方案——没有银弹，只有适合的工具。
