# 86 分钟：Rust 供应链攻击把 yank 机制变成了诱饵

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/arrayref-supply-chain-attack?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/arrayref-supply-chain-attack?utm_source=github&utm_medium=referral)**

> "We do not believe the author of arrayref to be acting maliciously, but their computer or credentials are likely compromised."
> —— Rust Security Response Team, 2026-08-20

---

2026 年 8 月 20 日 07:15 UTC，Rust 安全响应团队收到一份报告：`proc-macro1` 这个 crate 是恶意的。团队核实后发现，它的构建脚本会在编译期下载并执行远程载荷。

顺着这条线查下去，事情变大了：十年来第一次，`arrayref` 加了一个依赖——就是这个 `proc-macro1`。

`arrayref` 是那种你可能从没直接写进 `Cargo.toml`、但几乎一定在你依赖树里的 crate。crates.io 上累计 2.45 亿次下载，最近 90 天 5370 万次，403 个 crate 直接依赖它。它躺在 `blake3`、`winit`、`tiny-skia` 下面，也躺在 Solana 和以太坊工具链的大片区域下面。

恶意版本在线 86 分钟。但这 86 分钟里发生的事，值得每个用 cargo 的人认真读一遍——因为攻击者用的手法，把生态里一个**本来用来保护你的机制**，改造成了诱饵。

## 一、精确的时间线

官方公告给出的数据很干净，直接抄下来：

| crate | 恶意版本 | 发布时间 (UTC) | 删除时间 (UTC) | 在线时长 |
|---|---|---|---|---|
| `arrayref` | 0.3.10 | 2026-08-20 07:15:00 | 08:41:40 | 86 分钟 |
| `internment` | 0.8.7 | 2026-08-20 07:34:07 | 09:04:11 | 90 分钟 |
| `append-only-vec` | 0.1.9 | 2026-08-20 07:37:49 | 09:25:24 | 107 分钟 |

外加一批被整体删除的攻击者 crate（任意版本）：`proc-macro1`、`proc-macro-en`、`aovine`、`arone`、`aronenao`、`tinymember`。

三个被投毒的 crate 属于同一个作者。Rust 团队的判断是：作者本人没有恶意，是电脑或凭据被攻破了。账号已被锁定作为预防措施。

这里有个容易被忽略的细节：**攻击者的基础设施早就位了**。`arone` 和 `aronenao` 这两个带恶意构建脚本的 crate，最终发布时间是 2026-08-18——比 `arrayref` 被推送早两天。这不是临时起意，是准备好之后等一个凭据到手。

## 二、真正新颖的地方：yank 成了诱饵

如果只看"往流行 crate 里塞恶意依赖"，这是老套路。这次值得单独拿出来讲的是投递方式。

先说 `yank` 是什么。在 crates.io 上，yank 一个版本意味着"这个版本还能下载（老的 lockfile 不会崩），但不要再用它了"。它是维护者的正常工具，用来标记有 bug 或有安全问题的发布。当你的依赖里有被 yank 的版本时，cargo 会提示你：

```
warning: consider updating to a version that is not yanked
```

攻击者拿到账号之后做了什么？**把 0.3.5 到 0.3.9 全部 yank 掉**。

于是生态里出现了这样一个局面：你项目里锁着 `arrayref` 0.3.9，跑一次 cargo，工具链亲口告诉你——你用的版本被作者废弃了，建议升级到一个没被 yank 的版本。而当时唯一没被 yank 的版本，是 07:15 刚发布的 0.3.10。

这就是全部的社会工程量。不需要钓鱼邮件，不需要伪造文档，攻击者只是**把包管理器的安全提示接管过来，让它替自己说话**。一个每天都在提醒你"该升级了"的机制，被反向利用成了"请升级到我的后门"。

顺带一提，那个 typosquat 依赖 `proc-macro1` 是在冒充 dtolnay 的 `proc-macro2`——发布账号叫 `dtolney`，和真实维护者 `dtolnay` 只差一个字母。在依赖树里扫一眼，几乎不可能看出来。

...

---

**[👉 继续阅读全文：86 分钟：Rust 供应链攻击把 yank 机制变成了诱饵](https://tools.cooconsbit.com/zh/articles/arrayref-supply-chain-attack?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
