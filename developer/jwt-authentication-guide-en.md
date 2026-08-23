# JWT Authentication Explained: How It Works & Security Best Practices

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/jwt-authentication-guide-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/jwt-authentication-guide-en?utm_source=github&utm_medium=referral)**

## Introduction

JSON Web Tokens appear in almost every modern web application. They handle API authentication in mobile apps, secure single-page applications, and enable single sign-on across microservices. Yet they are also routinely misconfigured in ways that create serious vulnerabilities.

Understanding JWT is not just about knowing the format. It is about understanding the trust model: what the token proves, what it cannot prove, and where the guarantees break down. This guide covers the structure, the authentication flow, storage decisions, and the security pitfalls that trip up experienced developers.

---

## What Is a JWT?

A JWT is a compact, URL-safe string that encodes claims — statements about a subject (typically a user) — in a format that can be cryptographically verified. The key property: the server does not need to look up the token in a database to validate it. It verifies the signature using a secret key.

This makes JWT **stateless**: every piece of information needed to authenticate the user is inside the token itself.

A JWT looks like this:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1c2VyMTIzIiwiaXNzIjoibXlhcHAuY29tIiwiZXhwIjoxNzAwMDAwMDAwLCJpYXQiOjE2OTk5OTY0MDB9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

Three Base64URL-encoded parts separated by dots: `header.payload.signature`

---

## JWT Structure: Every Part Explained

### Header

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

The header declares the algorithm used to sign the token (`alg`) and the token type (`typ`). `HS256` is HMAC-SHA256 (symmetric). `RS256` is RSA-SHA256 (asymmetric). The algorithm choice has significant security implications — covered in the Security Pitfalls section.

### Payload (Claims)

```json
{
  "sub": "user123",
  "iss": "myapp.com",
  "aud": "api.myapp.com",
  "exp": 1700000000,
  "iat": 1699996400,
  "jti": "a1b2c3d4",
  "role": "admin",
  "plan": "pro"
}
```

The payload contains claims. Standard claims are defined by RFC 7519. Custom claims (`role`, `plan`) can be anything your application needs.

**Critical:** The payload is Base64URL-encoded, not encrypted. Anyone who obtains the token can decode and read the payload without knowing the secret. The signature only proves the payload has not been tampered with — it does not hide the contents.

### Signature

```
HMACSHA256(
  base64url(header) + "." + base64url(payload),
  secret
)
```

The signature is computed by hashing the header and payload together with the secret key. If any byte of the header or payload changes, the signature becomes invalid. This is the integrity guarantee.

...

---

**[👉 Continue reading: JWT Authentication Explained: How It Works & Security Best Practices](https://tools.cooconsbit.com/en/articles/jwt-authentication-guide-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
