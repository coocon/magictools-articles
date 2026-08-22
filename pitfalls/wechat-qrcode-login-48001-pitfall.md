---
title: "公众号做微信扫码登录：踩了 48001 才知道要服务号，附订阅号也能用的验证码登录方案"
slug: wechat-qrcode-login-48001-pitfall
category: pitfalls
tags: [微信登录, 公众号, 48001, 带参数二维码, NextAuth, 扫码登录, 服务号, 复盘]
summary: "公众号带参数二维码做扫码登录，上线即遭 48001——排查穿过 IP 白名单、token 缓存、后台权限页、业务域名四个假嫌疑后，官方 rid 诊断揭晓：该接口只给「认证的非个人主体服务号」，认证企业公众号也不行且类型不可互转。本文复盘 rid/quota 诊断三板斧，并给出订阅号可用的替代方案：固定二维码 + 6 位验证码 + NextAuth database session 手工铸造。"
status: published
locale: zh
source: authored
---

## 背景：不想为一个登录按钮再花一份认证钱

网站已经有 GitHub / Google OAuth，想补微信登录。微信生态里"正经"的网站扫码登录走**开放平台「网站应用」**，需要单独的开发者认证；而我们手里已经有一个**认证过的公众号**，于是决定走公众号路线省下这笔钱——顺带还有个副产品：扫码登录的用户自动成为公众号粉丝。

公众号路线的经典玩法是**带参数二维码**：

1. 服务端调 `cgi-bin/qrcode/create` 生成带 `scene` 参数的临时二维码
2. 网页展示，用户扫码关注（或已关注直接扫）
3. 微信把 `subscribe` / `SCAN` 事件推到你配置的回调 URL，事件里带 openid 和 scene
4. 服务端把 scene 对应的登录票据标记为已确认，前端轮询到结果，登录完成

我们把整条链路实现完，本地 18 项断言全绿（验签正反例、票据状态机、防重放、会话铸造），部署上线，配好 IP 白名单和服务器配置——然后第一次真实出码就炸了。

## 事故现场：48001 api unauthorized

```
[wechat-qrcode] Error: [wechat-mp] 创建二维码失败: 48001 api unauthorized
rid: 6a89846c-6a4ab866-02c7a905
```

48001 的官方解释是"api 功能未授权"。接下来的排查穿过了四个假嫌疑，每一个都值得记下来，因为搜索引擎上关于 48001 的答案几乎全是这四个方向——而它们在我们的场景里全部不成立。

### 假嫌疑一：IP 白名单 / 凭据错误

不成立的证据很直接：**access_token 取到了**。错误发生在 `qrcode/create` 这一步而不是 `cgi-bin/token`。如果是白名单或 AppSecret 问题，会在取 token 时报 40164 / 40125，根本走不到创建二维码。

### 假嫌疑二：token 缓存陷阱

我们把 access_token 缓存在数据库单行表里（token 全局唯一 + 每日获取限次，多机部署必须共享缓存）。这带来一个真实风险：**如果最初配错过凭据，缓存里可能躺着"别人家"的 token**，改对凭据后接口用的还是旧缓存。

清掉缓存、强制用当前凭据取全新 token——还是 48001。排除。

（这个坑值得单独记：任何 token 缓存表，在换凭据后都要手动清一次。）

### 假嫌疑三：后台「接口权限」页显示"已获得"

公众号后台的接口权限列表里，「生成带参数的二维码」赫然显示可用状态。这是整个排查中最有迷惑性的一环——**后台页面的展示与 API 实际权限可以不一致**，不要把它当权威依据。

### 假嫌疑四：业务域名没配置

设置了业务域名之后用全新 token 重测：依旧 48001。这符合原理——业务域名、JS 接口安全域名、网页授权域名影响的是**网页侧能力**（JSSDK、OAuth 网页授权），`qrcode/create` 是纯服务端接口，和域名配置没有任何关系。

## 诊断三板斧：让微信自己告诉你答案

比逐个猜嫌疑高效得多的，是微信自带的两个诊断接口：

**① `openapi/rid/get` —— 拿着报错 rid 查请求详情**

每个微信 API 错误都带一个 `rid`。把它交给这个接口，能拿回那次失败请求的完整现场：

```bash
curl "https://api.weixin.qq.com/cgi-bin/openapi/rid/get?access_token=$TK" \
  -d '{"rid":"6a89846c-6a4ab866-02c7a905"}'
# 返回 invoke_time / cost_in_ms / request_url / response_body / client_ip
```

**② `openapi/quota/get` —— 查某接口是否对本账号开放**

```bash
curl "https://api.weixin.qq.com/cgi-bin/openapi/quota/get?access_token=$TK" \
  -d '{"cgi_path":"/cgi-bin/qrcode/create"}'
# 我们的返回：{"errcode":76022,"errmsg":"could not use this cgi_path, no permission"}
```

**76022 就是权威判决**：不管后台页面显示什么，这个账号对这个接口就是没有权限。

最终通过 rid 走微信官方支持渠道拿到了完整结论（原文）：

> 该 API 为创建带参数的二维码 ticket 接口，调用方账号类型为**已认证的企业主体公众号**。但该接口明确要求调用账号必须为"**已认证的非个人主体服务号**"，而"公众号"类型账号无权调用。尽管当前账号已认证且为企业主体，但因账号类型不符，导致权限被拒绝。**公众号与服务号类型不可相互转换。**

划重点：**认证状态对、主体类型对，都救不了账号类型不对**。订阅号（日常语境里的"公众号"）和服务号是两个物种，且不可互转——想用带参数二维码，只能另起炉灶注册服务号。

## 替代方案：固定二维码 + 6 位验证码

不想为一个登录按钮再养一个服务号。翻权限矩阵时注意到一个关键事实：**通过服务器配置收发文本消息，不需要任何账号类型或认证门槛**。于是方案改成：

```
网页显示：公众号固定关注二维码 + 大号 6 位验证码
→ 用户扫码关注（已关注则直接进对话框）
→ 在公众号对话框发送这 6 位数
→ 服务端收到 text 消息，匹配登录票据 → 确认
→ 公众号立即回复"✅ 登录成功"
→ 网页轮询到确认，2 秒内自动进入登录态
```

交互从"扫码即登录"变成"扫码 + 发个码"，多了一步，但零费用、零新资质，且涨粉属性不变。几个实现要点：

### 票据用双标识，防枚举与防撞分离

- `scene`：32 位随机 hex，只给前端轮询用——6 位数字码只有一百万种组合，直接拿来轮询会被枚举
- `code`：6 位数字，只给用户在微信里发送用，生成时在活跃票据内查重，碰撞就重试

票据状态机严格单向：`pending → confirmed →（原子 updateMany 消费）→ consumed`，同一票据只能换一次会话，微信服务器的事件重试天然幂等。

### NextAuth database session 的手工铸造

这是全文最冷门的技巧。这套登录不是 OAuth，Auth.js 没有对应 provider，而 Credentials provider 又不支持 database session 策略。解法是**绕过 Auth.js 直接铸造会话**：按 openid 查/建 `User` + `Account` 行，插入一条与 PrismaAdapter 完全同构的 `Session` 行，然后手工种 cookie：

```ts
const session = await prisma.session.create({
  data: {
    sessionToken: randomUUID(),
    userId,
    expires: new Date(Date.now() + 30 * 24 * 3600 * 1000), // 与 auth 配置的 maxAge 一致
  },
});
// 生产（https）下 cookie 名必须是 __Secure-authjs.session-token
res.cookies.set(cookieName, session.sessionToken, {
  httpOnly: true, sameSite: "lax", secure: true, path: "/", maxAge: 30 * 24 * 3600,
});
```

之后 `auth()` / `useSession()` 完全无感知——我们用断言验证过：拿铸造的 cookie 请求 `/api/auth/session`，Auth.js 正常返回用户。

### 多机部署的两个坑

如果你的站点像我们一样按地域 DNS 分流到多台服务器：

1. **微信的事件推送只会到达国内节点**（微信服务器在国内，DNS 就近解析）。用户可能在海外节点的页面上出码——所以票据状态必须放共享数据库，不能放进程内存
2. **access_token 全局唯一**，重复获取会使旧 token 失效且每日限次。多机各自刷新会互相踢——缓存进共享数据库单行表，谁的缓存快过期谁刷新回写

### 别忘了被动回复

启用服务器配置后，公众号后台的自动回复会失效（消息处理被你的服务器接管）。关注欢迎语要在 `subscribe` 事件的被动回复里补上——顺便把"发送验证码即可登录"的引导写进去，形成闭环。

## 常见问题 FAQ

### 48001 一定是账号类型问题吗？

不一定。48001 泛指"接口未授权"，也可能是接口被冻结、权限集变更等。判定方法不变：`openapi/quota/get` 查该 cgi_path 对你的账号是否开放（76022 = 不开放），`openapi/rid/get` 查具体请求现场，比翻论坛猜快得多。

### 后台「接口权限」页为什么和实际不符？

我们没有得到微信侧的官方解释，只能确认现象：页面显示可用、API 实际返回 76022。工程上的结论是：**以 API 返回为准，页面展示只作参考**。

### 验证码方案安全吗？

关键设计：轮询凭据（32 位随机 hex）与用户输入的 6 位码分离，枚举面在轮询侧不存在；6 位码 5 分钟过期、活跃期内唯一、confirmed 后原子消费防重放；事件端点做 sha1 签名校验，伪造的回调直接 403。

### 订阅号收发消息真的不需要认证吗？

服务器配置（URL + Token 验签 + 明文/安全模式）所有公众号都能启用，text 消息的接收与被动回复没有账号类型门槛。有门槛的是**主动**能力：客服消息接口、模板消息、带参二维码这类才挑账号类型。

## 参考链接

- [微信官方文档：生成带参数的二维码](https://developers.weixin.qq.com/doc/offiaccount/Account_Management/Generating_a_Parametric_QR_Code.html)
- [微信官方文档：接收普通消息（服务器配置）](https://developers.weixin.qq.com/doc/offiaccount/Message_Management/Receiving_standard_messages.html)
- [微信官方文档：openapi 管理接口（rid 查询 / quota 查询）](https://developers.weixin.qq.com/doc/offiaccount/openApi/get_rid_info.html)
- [Auth.js Database Session 策略](https://authjs.dev/concepts/session-strategies)
- [同期实战：跨境网站页面从 2.7s 降到 0.8s 的读写分离改造](https://tools.cooconsbit.com/zh/articles/mysql-cross-region-read-replica-prisma)
