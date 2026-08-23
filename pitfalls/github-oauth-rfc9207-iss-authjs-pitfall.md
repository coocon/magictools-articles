# GitHub OAuth 登录突然全挂：一次 RFC 9207 灰度撞上 Auth.js 占位 issuer 的排查实录

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/github-oauth-rfc9207-iss-authjs-pitfall?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/github-oauth-rfc9207-iss-authjs-pitfall?utm_source=github&utm_medium=referral)**

某天中午点 GitHub 登录，浏览器转了一圈回到首页，URL 挂着 `?error=Configuration`。两台生产服务器（境外 + 国内）**同时**挂，而且——最要命的巧合——当天上午刚部署了数据库读写分离。

换你是第一反应，大概率也是：「今天的部署搞坏了登录」。

这篇文章复盘这次事故的完整排查链：怎么排除部署嫌疑、真正的根因是什么、一行修复背后的验证方法，以及几条能带走的通用经验。

## 现象：error=Configuration，日志里一条陌生的报错

前端能看到的只有 `/?error=Configuration`，信息量为零。真正有价值的在服务端日志：

```
[auth][error] CallbackRouteError: Read more at https://errors.authjs.dev#callbackrouteerror
[auth][cause]: OperationProcessingError: unexpected "iss" (issuer) response parameter value
[auth][details]: {
  "expected": "https://authjs.dev"
}
```

两个关键信息：

1. 报错发生在 **OAuth 回调阶段**，是 `iss`（issuer）参数校验不过；
2. 期望值 `https://authjs.dev` 明显不是一个真实的 issuer——这是 Auth.js 的**占位符**。

日志里还伴生一些 `MissingCSRF` 报错，后来确认是用户在错误页反复重试产生的噪音，不是线索。排查时**先抓最早、最具体的那条错误**，别被伴生噪音带偏。

## 第一嫌疑人：当天的部署（以及怎么无罪释放它）

当天上午刚上线数据库读写分离（Prisma 读路由扩展 + 只读副本），登录中午挂掉——时间上严丝合缝。但「时间吻合」只是相关性，定罪需要证据链。三步排除：

**第一步：diff 依赖 lock 文件。** 这次部署动了什么？`git diff` 看 `package-lock.json`，auth 相关依赖（`next-auth`、`@auth/core`、`@auth/prisma-adapter`、底层的 `oauth4webapi`）**零变化**，只新增了一个 Prisma 读写分离扩展。登录链路的代码和依赖都没动。

**第二步：查数据库里最后一次成功登录。** Session 表显示最后一次成功登录是 6 天前。也就是说，这 6 天里根本没人登录过——今天的失败是灰度命中后的**第一次尝试**，而不是「部署前能登、部署后不能登」。所谓的时间吻合，只是采样稀疏造成的错觉。

**第三步：看故障面。** 两台机器跑同一份代码、同时挂。如果是某台机器的环境问题（比如副本延迟、网络），不会两台同步表现一致。同一份代码 + 外部变更，才能解释「同时全挂」。

...

---

**[👉 继续阅读全文：GitHub OAuth 登录突然全挂：一次 RFC 9207 灰度撞上 Auth.js 占位 issuer 的排查实录](https://tools.cooconsbit.com/zh/articles/github-oauth-rfc9207-iss-authjs-pitfall?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
