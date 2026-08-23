# C2PA 内容凭证是什么？如何验证一张图片是不是 AI 生成的

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/c2pa-content-credentials-guide?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/c2pa-content-credentials-guide?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：C2PA 内容凭证是什么？如何验证一张图片是不是 AI 生成的](https://tools.cooconsbit.com/zh/articles/c2pa-content-credentials-guide?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
