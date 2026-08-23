# 两次 API 调用，偷走大模型脑子里的秘密

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/stealing-reasoning-traces-zh?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/stealing-reasoning-traces-zh?utm_source=github&utm_medium=referral)**

你刚写完一段代码，把报错信息丢给 Claude，让它帮你 debug。Claude 返回了修复方案，你满意地继续干活。

但你不知道的是，Claude 返回的响应里藏着一个加密块——那是模型的"思考过程"，从分析你的代码到推理出解决方案的每一步。理论上这个加密块只有服务器能解。但一项最新研究证明：**攻击者只需要两次 API 调用，就能把这个加密块解码成明文，看到模型脑子里在想什么。**

更吓人的是，研究团队扫描了 GitHub 和 HuggingFace 上 6,708 个公开的 AI agent 对话记录，从中挖出了 62 个 API 密钥、33 个密码、24 个 access token，以及大量邮箱地址和内部 URL。其中 **64 个敏感信息只出现在推理块里，在可见对话中完全没有。**

这就是论文《Stealing Reasoning Traces from Proprietary LLM APIs》的核心发现。作者来自 MATS Research、马普所和 ELLIS Institute Tübingen，阵容硬核，实验扎实。

## 攻击原理：用"弱模型"当翻译机

整个过程只需要两步，理解起来甚至不需要机器学习背景。

第一步，拿到加密块。当你调用 GPT-4o、Claude Opus 或 Gemini 的 API 时，如果模型启用了"思考模式"（thinking/reasoning），响应里会附带一个 `thinking` 字段。对 OpenAI 来说是 `encrypted_content`，对 Anthropic 是 `signature` 字段包裹的加密数据。这段数据有几万个字符，看起来是一堆乱码。

第二步，跨模型重放。关键漏洞在这里：**这些加密块和具体的模型、会话、用户无关**。你可以把 Claude Opus 4 产生的加密块，原封不动地塞进 Claude Haiku 的同一个字段里，Haiku 会接受它。

第三步，诱导弱模型"翻译"。Haiku 本身没有 Opus 的安全护栏强，加上你可以 jailbreak 它，让它以为这段加密块就是自己的思考过程，并请求它"把你刚才的思考逐字抄出来"。然后——它就真的抄出来了。

本质上是把 Opus 的脑子移植到 Haiku 里，然后审问 Haiku："刚才在想什么？" Haiku 一五一十说了，因为它觉得那就是自己想出来的。

...

---

**[👉 继续阅读全文：两次 API 调用，偷走大模型脑子里的秘密](https://tools.cooconsbit.com/zh/articles/stealing-reasoning-traces-zh?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
