---
title: "两次 API 调用，偷走大模型脑子里的秘密"
slug: stealing-reasoning-traces-zh
summary: "闭源大模型的加密思维链块可以被跨模型重放——把 Opus 的加密思维塞给 Haiku 让它逐字念，就能还原强模型的隐藏推理过程。研究者扫了 6708 条公开 agent 轨迹，捞出 62 个真实 API key 和 33 个密码。"
category: ai-tutorials
tags: [AI安全, 思维链, Claude, OpenAI, Gemini, prompt injection, 大模型]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: stealing-reasoning-traces-en
---

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

打个比方：Opus 用密码写了一份日记，把加密后的密文交给你保管。你发现可以用同一个密码本（同一个厂商的 API 协议）把密文塞给 Haiku 读。Haiku 是个好骗的小孩，你哄它说"这是你自己写的，念出来听听"，它就一字不差念出来了。Opus 全程不知道日记被人看过。

## 不是理论，是真挖出了密码和信用卡号

论文团队不只是 demo 了攻击可行性，他们做了一件更狠的事：**扫描真实世界的数据**。

他们从 GitHub 和 HuggingFace 收集了 6,708 个包含加密推理块的公开 agent 对话记录，覆盖 Claude、GPT 和 Gemini 三家的模型。用上述方法解码后，得到了 315,320 个被重建的推理块。

在这些推理块中，他们筛出了 704 个隐私泄露项。其中：

- 62 个 API 密钥
- 33 个密码
- 24 个 access token
- 30 个个人邮箱
- 以及姓名、地址、内部 URL 等

关键在于，其中 64 个只出现在推理块中。意思是：**用户以为自己的对话是安全的，敏感信息根本没出现在可见的部分，但其实模型在"脑子里"过了一遍，而这个脑子可以被偷看。**

一个具体的例子：GPT-5.2 Codex 在执行 sanitize-git-repo 任务时，推理块里列出了它要清理的具体 token 值——包括 AWS access key、GitHub token、HuggingFace token，全是明文。这些 token 在最终的对话输出里已经被模型自觉地抹掉了，但推理过程里记得一清二楚。

还有一个 Claude Sonnet 4.6 的例子：用户让它帮忙订机票，推理块里完整记录了乘客姓名、护照号、生日、信用卡号（含有效期和 CVV），甚至 aeroplan 积分卡号。这些信息用户可能并没有在对话中明确提供——是模型从上下文推断出来的，但推断过程被加密块原样保存。

## 厂商藏思考链，到底在怕什么？

OpenAI 和 Anthropic 早就开始限制用户查看模型的 chain-of-thought。官方说法是"出于安全考虑"——如果用户看到模型是怎么推理的，可能更容易绕过安全护栏。

但这篇论文暴露了另一个动机：**怕被人低成本复制。**

论文的实验证明了，Opus 的推理质量确实更高。同一个 Codeforces 题目，Opus 的推理块长度和 API 报告的 hidden thinking tokens 高度吻合（r² 接近 1），说明它真的在想。但如果你能把 Opus 的思考过程原样提取出来，喂给一个便宜的小模型做训练数据，那你就是在用几百美元的 API 费用，蒸馏出几百万美元训练成本才能达到的效果。

所谓的"闭源模型护城河"，被这个叫"推理轨迹泄露"的侧门慢慢抽干了。

而且这个漏洞很难快速修复。根本矛盾在于：模型提供商需要加密块来实现多轮对话的上下文保持——用户第二轮请求时，服务器用加密块重建上一轮的思考状态。加密块必须返回给客户端，客户端必须在下一轮请求时原样发回来。一旦这个加密块不绑定会话和用户，replay 攻击就是先天缺陷。

## 你今天能做什么

别慌，但有几件事值得马上做：

**1. 检查你调的 API 版本和参数。** 如果你用的是 OpenAI 的 reasoning 模型或 Anthropic 的 extended thinking，关注响应里的 `encrypted_content` / `signature` 字段。你的应用服务器是否保存了这些字段？日志里有没有？如果你把完整 API 响应写进了日志文件，那这些加密块就在日志里躺着。

**2. 不要把完整 API 响应暴露到公开仓库。** 这个建议看起来像废话，但论文的数据就是最好的警示：GitHub 和 HuggingFace 上有 6,708 个公开会话包含了加密块。你觉得自己不会犯这种错，但这些数据的主人当时也是这么想的。

**3. 敏感数据别经过模型。** 如果你让模型处理包含 API key、密码、用户 PII 的数据，这些信息会进入模型的推理过程——即使最终输出里没有出现。而推理过程以加密块的形式返回了。加密不是绝对安全的，这篇论文证明了。

**4. 关注厂商的修复。** Anthropic 和 OpenAI 收到论文后大概率已经在修了。可能的修复方向包括：加密块绑定会话、加密块绑定模型版本、对加密块做完整性校验。在新版本出来之前，保守使用思考模式。

---

一句话总结：你以为你只看到了模型的答案，但模型把整个草稿纸也一并寄给你了——只是用了一个能被破解的信封。

> 本文基于 codefarm 码农早餐 2026-08-12 期选题，参考 stolen-thoughts.com 论文原始数据。

**参考来源：**
- [Stealing Reasoning Traces from Proprietary LLM APIs — stolen-thoughts.com](https://stolen-thoughts.com/)
- [arXiv: 2608.09867](https://arxiv.org/abs/2608.09867)
