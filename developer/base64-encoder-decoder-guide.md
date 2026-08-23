# Base64 编码解码完全指南：原理、使用场景与在线工具

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/base64-encoder-decoder-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/base64-encoder-decoder-guide?utm_source=github&utm_medium=referral)**

## 什么是 Base64？

Base64 是一种将任意二进制数据转换为纯 ASCII 字符的编码方式。它使用 64 个可打印字符（A-Z、a-z、0-9、+、/）来表示二进制数据，并用 `=` 作为填充符。

**为什么叫 Base64？** 因为编码字符集恰好包含 64 个字符，每 6 位二进制数据对应一个字符（2⁶ = 64）。

Base64 的核心用途是：**在只能处理文本的场景中安全传输二进制数据**。注意，Base64 是编码（Encoding），不是加密（Encryption）——任何人都可以轻松解码，它不提供任何安全保护。

---

## 常见使用场景

### 1. 在 JSON/XML 中嵌入图片（Data URI）

在 HTML 或 CSS 中，可以用 Data URI 直接嵌入图片，避免额外的 HTTP 请求：

```html
<img src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..." />
```

```css
.icon {
  background-image: url('data:image/svg+xml;base64,PHN2ZyB4bWxucz0...');
}
```

小图标（< 5KB）特别适合这种方式，可减少页面加载时的 HTTP 请求数。

### 2. HTTP Basic Authentication 头部

HTTP 基本认证将用户名和密码以 Base64 编码后放入请求头：

```
Authorization: Basic dXNlcm5hbWU6cGFzc3dvcmQ=
```

其中 `dXNlcm5hbWU6cGFzc3dvcmQ=` 是 `username:password` 的 Base64 编码。**这不安全**——任何人都能解码。生产环境必须配合 HTTPS 使用。

### 3. 邮件附件编码（MIME）

SMTP 协议最初只支持 ASCII 文本。MIME 标准引入 Base64 编码，使邮件可以携带二进制附件（图片、PDF、Office 文件等）。你在邮件客户端看到的附件，在协议层面都是 Base64 字符串。

### 4. JWT Token 的 Header/Payload 部分

JSON Web Token（JWT）由三部分组成，前两部分均为 Base64URL 编码：

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIn0.SflKxwRJSMeKKF2QT4fwpMeJf36P
```

将第一部分解码：`{"alg":"HS256","typ":"JWT"}`

### 5. CSS 中嵌入小图片

将小图标内联进 CSS 文件，减少 HTTP 请求：

```css
.logo {
  background: url('data:image/png;base64,iVBORw0KGgoAAAA...') no-repeat;
}
```

...

---

**[👉 继续阅读全文：Base64 编码解码完全指南：原理、使用场景与在线工具](https://tools.cooconsbit.com/zh/articles/base64-encoder-decoder-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
