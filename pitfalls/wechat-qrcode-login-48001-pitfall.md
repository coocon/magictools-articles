# 公众号做微信扫码登录：踩了 48001 才知道要服务号，附订阅号也能用的验证码登录方案

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/wechat-qrcode-login-48001-pitfall?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/wechat-qrcode-login-48001-pitfall?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：公众号做微信扫码登录：踩了 48001 才知道要服务号，附订阅号也能用的验证码登录方案](https://tools.cooconsbit.com/zh/articles/wechat-qrcode-login-48001-pitfall?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
