# 7 月 28 日，MCP 的成人礼：协议砍掉状态，生态打响第一场官司

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/mcp-stateless-spec-first-lawsuit?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/mcp-stateless-spec-first-lawsuit?utm_source=github&utm_medium=referral)**

2026 年 7 月 28 日，MCP（Model Context Protocol）生态在同一天迎来两件事。

第一件在规范层：[MCP 2026-07-28 版规范](https://modelcontextprotocol.io/specification/2026-07-28/changelog)正式发布，官方维护者称之为「协议诞生以来最大的一次修订」。核心变更一句话：**MCP 不再有状态了。** initialize 握手没了，session ID 没了，协议从有状态的双向连接改成了无状态的请求/响应。

第二件在商业层：MCP 安全网关创企 Runlayer [起诉 Rippling](https://techcrunch.com/2026/07/28/mcp-startup-runlayer-accuses-rippling-of-stealing-its-product-idea/)，指控这家 HR 科技公司以「准客户」身份试用近一年、拿到产品 roadmap 和源码之后，转身自建了同类 MCP 网关产品。诉由三项：商业机密盗用、不正当竞争、违约。这是 MCP 生态的第一场商业诉讼。

一个协议在同一天完成两件事：向运维现实低头，和证明自己值得打官司。这两件事哪件单拎出来都不小，但放在一起读才是完整的故事——**一项技术从实验品变成基础设施的成人礼，恰好被压缩进了 24 小时。**

先把两件事分别拆开。

## 规范这边：砍掉的是什么，为什么

新规范由 5 月 21 日锁定的 Release Candidate 演化而来，中间留了 10 周给各语言 SDK 维护者做验证。主要变更集中在几个 SEP（规范增强提案）上：

**SEP-2575 砍掉了 initialize 握手。** 旧协议里客户端和服务器要先握手交换版本和能力，才能开始干活；新协议里这些信息随每个请求走——协议版本和客户端能力放进 `_meta` 字段（`io.modelcontextprotocol/protocolVersion`、`io.modelcontextprotocol/clientCapabilities`），另有 `MCP-Protocol-Version`、`Mcp-Method`、`Mcp-Name` 三个 HTTP header 随请求携带。想了解服务器能力？新增的 `server/discover` 端点随时可查，不用握手。

**SEP-2567 砍掉了 Mcp-Session-Id。** 协议层的会话概念整个移除。工具调用之间确实需要跨请求状态的，改用服务器显式签发的 handle，当普通参数传回来。

**MRTR（Multi Round-Trip Requests）接替了服务器主动请求。** 旧协议里工具执行到一半要用户确认，得靠一条常开的双向流回传；新设计里服务器直接返回 `resultType: "input_required"` 带上问题，客户端拿到用户输入后重新发起调用。没有长连接，一样能对话。

...

---

**[👉 继续阅读全文：7 月 28 日，MCP 的成人礼：协议砍掉状态，生态打响第一场官司](https://tools.cooconsbit.com/zh/articles/mcp-stateless-spec-first-lawsuit?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
