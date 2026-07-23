---
title: "JWT Authentication Explained: How It Works & Security Best Practices"
slug: "jwt-authentication-guide"
category: developer
tags:
  - JWT
  - authentication
  - API security
  - tokens
  - web security
summary: "JWT (JSON Web Token) is the backbone of modern stateless authentication — this guide explains the token structure, standard claims, the authentication flow, storage strategies, and the critical security pitfalls that cause real-world breaches."
coverImage: ""
status: published
scheduledAt: ""
---

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

---

## Standard Claims Reference

| Claim | Full Name | Purpose |
|---|---|---|
| `sub` | Subject | Unique identifier for the user/entity |
| `iss` | Issuer | Who created and signed the token |
| `aud` | Audience | Who the token is intended for |
| `exp` | Expiration Time | Unix timestamp after which token is invalid |
| `nbf` | Not Before | Unix timestamp before which token is invalid |
| `iat` | Issued At | Unix timestamp when token was created |
| `jti` | JWT ID | Unique identifier for this specific token (enables revocation) |

Always validate `exp`. Always validate `iss` and `aud` if you have multiple services. Use `jti` if you need the ability to revoke individual tokens.

---

## JWT vs Session Tokens

| Dimension | JWT (Stateless) | Session Token (Stateful) |
|---|---|---|
| Server storage | Nothing — token is self-contained | Session data stored in DB or Redis |
| Scalability | Excellent — any server validates without shared state | Requires shared session store for multi-server deployments |
| Revocation | Difficult — token valid until expiry | Trivial — delete session record |
| Token size | Larger (full payload in every request) | Tiny (just an ID) |
| Server-side data | Payload reflects state at issuance, not current state | Always reflects current state |
| Suitable for | APIs, microservices, mobile apps | Traditional web apps with server-side sessions |

The "stateless" benefit is real in microservice architectures where adding a shared session store creates a dependency. For a single-server web application, session tokens are often simpler and safer because revocation is trivial.

---

## Authentication Flow

```
1. LOGIN REQUEST
   Client → POST /auth/login
           { "email": "alice@example.com", "password": "..." }

2. CREDENTIAL VALIDATION
   Server → Verify password hash against database
           If valid, generate JWT:
           payload = { sub: "user123", exp: now + 1h, iss: "myapp.com" }
           token = sign(payload, SECRET_KEY)

3. TOKEN DELIVERY
   Server → Client
           Set-Cookie: token=<jwt>; HttpOnly; Secure; SameSite=Strict
           (or) Response body: { "token": "<jwt>" }

4. AUTHENTICATED REQUEST
   Client → GET /api/user/profile
           Authorization: Bearer <jwt>
           (or Cookie header, if using cookie storage)

5. TOKEN VALIDATION (no database required)
   Server → Decode header + payload
           Verify signature: HMACSHA256(header.payload, SECRET_KEY) matches
           Check exp claim: current time < exp
           Check iss/aud if configured
           If all pass → request is authenticated
```

---

## Token Storage: The Security Trade-Off

Where you store the JWT on the client side determines your attack surface.

### localStorage / sessionStorage
```javascript
// Storing in localStorage
localStorage.setItem('token', jwt);

// Retrieving for requests
const token = localStorage.getItem('token');
fetch('/api/data', {
  headers: { Authorization: `Bearer ${token}` }
});
```

**Risk:** JavaScript on the page can read `localStorage`. If your site has an XSS vulnerability — even in a third-party script — an attacker can steal the token. XSS attacks are common; localStorage tokens make them devastating.

### HttpOnly Cookie
```http
Set-Cookie: token=<jwt>; HttpOnly; Secure; SameSite=Strict; Path=/
```

- `HttpOnly` — JavaScript cannot read this cookie. XSS cannot steal it.
- `Secure` — only sent over HTTPS
- `SameSite=Strict` — only sent with same-site requests, blocking CSRF

**Risk:** Cookies are automatically sent with requests, which historically enabled CSRF (Cross-Site Request Forgery). `SameSite=Strict` eliminates CSRF for modern browsers. For older browser support, add a CSRF token as well.

**Recommendation:** HttpOnly cookie with `Secure` + `SameSite=Strict` is the most secure storage option for browser-based applications. Use localStorage only when cookies are genuinely not feasible (some native app scenarios).

---

## Security Pitfalls

### 1. Never Store Sensitive Data in the Payload

The payload is readable by anyone. Do not store:
- Passwords or password hashes
- Full credit card numbers
- Social Security Numbers
- Internal system architecture details

Only store what the receiving service needs to authorize the request: user ID, roles, plan tier.

### 2. The `alg: none` Vulnerability

Some early JWT libraries accepted `{ "alg": "none" }` in the header, disabling signature verification entirely. An attacker could craft any payload they wanted and it would be "valid."

```json
// Malicious header
{ "alg": "none", "typ": "JWT" }
// Payload with admin privileges
{ "sub": "attacker", "role": "admin", "exp": 9999999999 }
// No signature required
```

**Fix:** Always explicitly specify which algorithms are acceptable when validating tokens. Reject tokens with `alg: none` or unexpected algorithms.

```javascript
// Node.js example — specify allowed algorithms explicitly
jwt.verify(token, SECRET_KEY, { algorithms: ['HS256'] });
```

### 3. Short Expiry + Refresh Token Pattern

Long-lived access tokens (days or weeks) are dangerous — if stolen, they provide prolonged access with no recourse. Use short-lived access tokens (15–60 minutes) paired with refresh tokens.

```
Access Token:  expires in 15 minutes — sent with every API request
Refresh Token: expires in 30 days — stored securely, used only to get new access tokens
```

When the access token expires, the client uses the refresh token at a dedicated endpoint to receive a new access token. Refresh tokens can be rotated (invalidated and replaced on each use) and stored in the database for revocability.

### 4. Use RS256 for Multi-Service Architectures

HS256 (symmetric) uses the same secret to sign and verify. Every service that validates tokens needs the secret — if any service is compromised, the secret is exposed.

RS256 (asymmetric) uses a private key to sign and a public key to verify. The auth service keeps the private key secret. All other services only need the public key to verify tokens. A compromised downstream service does not expose the ability to forge tokens.

```
Auth Service:  signs with PRIVATE KEY  → only this service can create tokens
API Services:  verify with PUBLIC KEY  → can validate but cannot forge
```

---

## Refresh Token Pattern

```
1. Login → receive: { accessToken (15min), refreshToken (30 days) }
2. API request → send accessToken in Authorization header
3. accessToken expires → client catches 401 response
4. Client → POST /auth/refresh with refreshToken (in HttpOnly cookie)
5. Server → validates refreshToken, issues new accessToken (+ optionally rotates refreshToken)
6. Client → retries original request with new accessToken
```

Store refresh tokens in the database to enable revocation. When a user logs out, delete the refresh token server-side — all future refresh attempts fail, forcing re-authentication.

---

## JWT Revocation Strategies

The stateless nature of JWT creates the revocation problem: once issued, a token is valid until expiry. There is no built-in "logout" for a JWT.

| Strategy | Mechanism | Trade-off |
|---|---|---|
| Short expiry | Small time window of unauthorized access | User logs out more frequently |
| Token blocklist | Store revoked `jti` values in Redis | Requires storage, adds latency |
| Refresh token rotation | Detect refresh token reuse (signals theft) | Complex to implement correctly |
| Stateful JWT | Store all active tokens, validate on each request | Eliminates stateless benefit |

For most applications: short access token expiry (15–60 min) + refresh token stored in database. This balances the stateless efficiency of JWT with meaningful logout capability.

---

## FAQ

**Can I logout with JWT?**
Not with a pure stateless approach. On the client side, deleting the stored token prevents it from being sent, which effectively logs the user out from that device. But the token itself remains valid until expiry. For genuine logout (preventing use of an intercepted token), you need either a short expiry, a server-side blocklist keyed on the `jti` claim, or a stateful refresh token system.

**What is the difference between JWT and OAuth 2.0?**
OAuth 2.0 is an authorization framework — a protocol for delegating access. JWT is a token format. These are complementary, not competing. OAuth 2.0 often uses JWT as its token format (specifically as the access token or ID token in OpenID Connect). You can use JWT without OAuth 2.0 (custom API authentication), and OAuth 2.0 without JWT (opaque tokens). In practice, most OAuth 2.0 implementations in 2024 use JWT for access tokens.

**HS256 vs RS256 — which should I use?**
Use HS256 for single-service applications or when the same team controls all services that validate tokens — simpler key management, faster performance. Use RS256 when multiple independent services validate tokens, when the token issuer and token consumers are maintained by different teams, or when you publish a JWKS endpoint so external services can dynamically discover the public key. RS256 eliminates the need to share secrets across services, which is critical at scale.

---

## Conclusion

JWT is an elegant solution to a genuine problem: how do you authenticate API requests without a database lookup on every request? The stateless verification model scales horizontally without shared state and works naturally across microservices.

The security model is sound when implemented correctly. The pitfalls are well-documented and avoidable: validate algorithms explicitly, never trust the payload as confidential, use short expiry with refresh tokens, prefer HttpOnly cookies over localStorage, and reach for RS256 when operating across multiple services.

The single most important decision is where you store tokens on the client. Choose HttpOnly cookies, implement `SameSite=Strict`, and keep access token lifetimes short. Everything else is refinement.
