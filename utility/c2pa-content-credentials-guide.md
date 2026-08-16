---
title: "C2PA 内容凭证是什么？如何验证一张图片是不是 AI 生成的"
slug: "c2pa-content-credentials-guide"
category: utility
locale: zh
source: authored
tags:
  - C2PA
  - Content Credentials
  - AI 图片
  - 数字签名
  - 内容溯源
  - 在线工具
summary: "OpenAI、Anthropic、Adobe、Google 和相机厂商都在给内容加同一种'出生证明'——C2PA 内容凭证。本文讲清它的结构（manifest、断言、数字签名）、如何在浏览器里免费验证一张图的凭证、验证结果怎么读（validation_state 与 failures 的真实关系，用一个我们踩过的坑说明）、以及它的边界：截图和社交平台转发为什么会让凭证消失。"
coverImage: ""
status: published
scheduledAt: ""
---

AI 生成的图片越来越难用肉眼分辨，行业给出的答案不是"更强的检测器"，而是反过来：**让内容自带可验证的出处记录**。这套开放标准叫 C2PA（Coalition for Content Provenance and Authenticity，内容来源与真实性联盟），由 Adobe、Microsoft、Intel、BBC 等发起，Google、OpenAI 相继加入；落到产品里的名字是 **Content Credentials（内容凭证）**。

现在它已经不是纸面标准：OpenAI 的 gpt-image 生成的图片带 C2PA 签名，Anthropic 自 2026 年 8 月起为 Claude 生成的图片等文件附加 C2PA 元数据（背景见《[Claude 隐形水印是什么](https://tools.cooconsbit.com/articles/claude-text-watermark-guide)》），Adobe Firefly、部分 Leica 和 Sony 相机也在拍摄端直接签名。你手里的任何一张图，都可以花十秒验证一下——用 [MagicTools 的 C2PA 验证器](https://tools.cooconsbit.com/tools/c2pa-verifier) 拖进去即可，验签在浏览器本地完成，图片不上传。

## C2PA 凭证长什么样

一份内容凭证的核心是嵌在文件里的 **manifest（清单）**，可以理解为一张随文件旅行的"出生证明 + 修改履历"，三个组成部分：

1. **断言（assertions）**：关于内容的声明。最重要的是 `c2pa.actions`——记录内容经历了什么动作：`c2pa.created`（创建）、`c2pa.edited`（编辑）、`c2pa.converted`（格式转换）等。AI 生成的内容会带上 `digitalSourceType` 为 `trainedAlgorithmicMedia` 的标记，这是判断"是否 AI 生成"的关键字段
2. **声明生成器（claim generator）**：谁写的这份清单——比如 `GPT-4o`、Adobe Photoshop、相机固件
3. **数字签名（signature）**：签发方（issuer）用证书对整份清单签名，绑定文件内容的哈希。文件被改动一个字节，签名校验就会失败

一个文件可以叠加多份 manifest：相机拍摄签一份，Photoshop 编辑后再签一份并引用前一份作为 ingredient（原料），形成完整的编辑链条。

## 验证结果怎么读

把图片交给验证器后，会得到三种基本状态：

**有凭证且校验通过**：能看到签发者（issuer）、声明生成器、动作列表、是否含 AI 生成标记。这是最有信息量的结果——比如 issuer 显示 "OpenAI OpCo, LLC"、actions 含 `c2pa.created`、带 AI 生成标记，基本可以确认这张图出自 OpenAI 的生成模型。

**有凭证但校验失败（Invalid）**：清单存在但签名对不上——文件在签名后被篡改过，或清单本身损坏。这时凭证里的信息**不可作为出处证据**。

**没有凭证**：注意，**这是完全正常的状态，不是警告**。世界上绝大多数图片本来就没有 C2PA 凭证——所有存量照片、大多数手机拍摄、未接入 C2PA 的生成工具的产物。没有凭证只说明"无法用这条途径验证出处"，不说明图片有问题，更不说明它是或不是 AI 生成的。

### 一个容易做错的判定细节：以 validation_state 为准

C2PA 验证库（c2pa-rs 系）的返回里同时有两个东西：汇总字段 `validation_state`（`Trusted` / `Valid` / `Invalid`）和明细数组 `validation_results` 里的 failures 列表。直觉上"failures 非空 = 校验失败"，**但这是错的**。

我们在开发 [C2PA 验证器](https://tools.cooconsbit.com/tools/c2pa-verifier) 时踩过这个坑：初版用 `failures.length > 0` 判红绿，结果 C2PA 官方的测试签名图（issuer 为 C2PA Test Signing Cert）被标成红色——它的签名完整性校验完全通过，`validation_state` 是 `Valid`，failures 里只是记录了"签名证书不在默认信任列表上"这一条。签名有效性和签发者是否受信是**两个独立维度**：

- `Trusted`：签名有效，且证书在已知信任列表上
- `Valid`：签名有效（内容未被篡改），但证书不在信任列表——自签证书、测试证书、小厂商都会落在这档
- `Invalid`：签名校验失败，内容不可信

判定红绿应该以 c2pa-rs 汇总出的 `validation_state` 为权威，failures 数组是给人看的明细。顺带一提，OpenAI 新发布的 Content Provenance API 用的也是同一套状态语义（`trusted` / `valid` / `invalid` / `not_present`），我们在《[OpenAI Content Provenance API 解读](https://tools.cooconsbit.com/articles/openai-content-provenance-api-guide)》里有详细对照。

## 边界：凭证会在哪些环节消失

C2PA 是元数据方案，这决定了它的脆弱面——**凭证跟着文件的元数据走，元数据没了凭证就没了**：

- **截图**：新生成的图片文件与原文件毫无关系，凭证必然丢失
- **社交平台转发**：微信、微博、X 等平台上传时普遍压缩图片并剥离元数据
- **格式转换与重新保存**：不保留元数据的转换工具会把清单丢掉
- **裁剪编辑后未重新签名**：内容变了，旧签名自然失效

所以验证时尽量拿**原始文件**。也正因为这个弱点，行业的完整方案是"C2PA 元数据 + 像素级水印"双轨并行：Google 的 SynthID 直接嵌在像素/音频波形里，能扛住部分转换——两条线互补，谁都不是银弹。

还有一条边界要说破：**"没有 AI 标记" ≠ "不是 AI 生成"**。未接入 C2PA 的生成工具产出的图片什么凭证都没有；而恶意方也可以生成后截图洗掉凭证。C2PA 的价值是给诚实的内容提供可验证的出处，而不是抓住所有造假——它回答"这张图能证明什么"，回答不了"这张图是什么"。

## 动手验证：三种工具

| 工具 | 特点 |
|------|------|
| [MagicTools C2PA 验证器](https://tools.cooconsbit.com/tools/c2pa-verifier) | 浏览器本地 WASM 验签，文件不上传；展示 issuer、声明生成器、动作、AI 标记、validation state 与失败明细 |
| [Content Credentials 官方验证页](https://contentcredentials.org/verify) | C2PA 官方，含信任列表校验和编辑链可视化 |
| [openai.com/verify](https://openai.com/verify/) | OpenAI 官方，同时查 C2PA 与 SynthID，但只识别 OpenAI 自家信号 |

日常快速查一张图用第一个（本地、快、免登录）；需要完整信任链裁决时用官方验证页交叉确认。

## 常见问题 FAQ

### 验证显示"未找到内容凭证"，说明图片有问题吗？

没有。绝大多数图片本来就不带 C2PA 凭证——所有历史照片、多数手机相机、未接入标准的工具产物都是如此。"无凭证"是中性结果，只说明无法用凭证这条途径验证出处。

### C2PA 凭证能被伪造吗？

清单内容可以随便写，但**签名伪造不了**：验证时校验的是签发方证书对文件哈希的数字签名。攻击者可以自签一份声称"来自某相机"的清单，但它的证书不在信任列表上，验证结果最多到 `Valid`（签名自洽）而到不了 `Trusted`（签发者受信）——这正是要区分这两档的原因。

### 为什么我用 AI 生成的图片验证不出凭证？

三个常见原因：使用的生成工具尚未接入 C2PA；图片经过了剥离元数据的环节（截图、社交平台压缩、格式转换）；下载渠道给的不是原始文件。换原始文件重试即可排除后两种。

### 浏览器里验证安全吗？图片会上传吗？

取决于工具实现。MagicTools 的验证器把 c2pa-rs 编译成 WebAssembly 在浏览器内运行，验签全程本地完成，无网络上传；敏感图片可以断网验证。使用其他在线工具前，建议确认其是否声明本地处理。

### C2PA 和 AI 图片检测器有什么区别？

方向相反。AI 检测器分析像素特征去**猜测**内容是否 AI 生成，有误判率；C2PA 是**验证**内容自带的密码学证据，结果是确定的——签名要么有效要么无效。但 C2PA 只对带凭证的内容有效，检测器对任何图片都能给出（不可靠的）判断。两者适合组合使用而非互相替代。

## 参考链接

- [C2PA 技术规范 — c2pa.org](https://c2pa.org/specifications/specifications/2.1/index.html)
- [Content Credentials 官方验证工具](https://contentcredentials.org/verify)
- [C2PA in ChatGPT Images — OpenAI](https://help.openai.com/en/articles/8912793-c2pa-in-chatgpt-images)
- [How Claude marks AI-generated content — Anthropic](https://support.claude.com/en/articles/16266773-how-claude-marks-ai-generated-content)
- [c2pa-rs：C2PA 的 Rust 参考实现](https://github.com/contentauth/c2pa-rs)
