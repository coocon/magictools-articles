---
title: "86 分钟：Rust 供应链攻击把 yank 机制变成了诱饵"
slug: arrayref-supply-chain-attack
summary: "2026 年 8 月 20 日，arrayref 被投毒。真正值得研究的不是恶意代码本身，而是攻击者把 cargo 的 yank 提示变成了社会工程武器——先把好版本全部标记为废弃，再让 cargo 亲口建议你升级到那个恶意版本。整条链路只在线 86 分钟，却踩中了 Rust 依赖模型里最难防的一环：build.rs 在编译期就是任意代码执行。"
category: developer
tags: [Rust, cargo, 供应链安全, 依赖管理, build.rs, 软件供应链]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: arrayref-supply-chain-attack-en
---

# 86 分钟：Rust 供应链攻击把 yank 机制变成了诱饵

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

Rust 团队的处置里有一步很聪明：他们不只是删掉了恶意版本，还**把被恶意 yank 的版本重新 unyank 了**。否则清理完之后，所有人会发现自己锁着的版本仍然显示"已废弃"，那个诱饵会一直挂在那里。

## 三、为什么常规依赖审计看不见它

三个被投毒 crate 的库代码本身是干净的。你去读 `arrayref` 0.3.10 的 `src/`，读不出任何问题。

恶意逻辑全部在 `proc-macro1` 的 `build.rs` 里。

这是 Rust 依赖模型里最硬的一块结构性问题：**`build.rs` 是编译期的任意代码执行**。cargo 会自动编译并运行它，不需要任何确认，不需要任何权限声明。也就是说——

**你不需要运行你的程序。`cargo build` 本身就够了。**

甚至不需要 `cargo build`。`cargo check`、IDE 后台的 rust-analyzer、CI 里的一次依赖预热，任何触发构建图求值的动作都可能执行到它。

这次的构建脚本做了这么几件事：从 base64 里重建攻击者的服务器地址（所以静态扫描抓不到明文域名）、通过 TLS 拉取第二阶段的二进制、**且不做证书校验**、然后在编译过程中直接运行它。

对比一下你平时的防御手段，会发现大部分都不在这条路径上：

- **代码审计**：库代码是干净的，审计通过。
- **`cargo audit` / RustSec**：靠的是已知漏洞数据库。攻击窗口是 86 分钟，公告出来的时候你早就中招或者早就没事了。
- **锁文件**：`Cargo.lock` 确实能锁住版本——但那个 yank 提示存在的意义，就是劝你改锁文件。
- **CI 沙箱**：如果你的 CI runner 有 npm/cargo 凭据、云凭据或 SSH key，编译期 RCE 就是凭据泄露。

真正能挡住的只有一件事：**在拉取新依赖版本的那一刻，有人或有工具去看了一眼这个版本新增了什么依赖。** `arrayref` 十年零依赖，突然多出一个——这个信号足够刺眼，前提是有人在看。

## 四、你现在该做什么

先说清楚版本关系，因为网上已经有不少二手报道把这里写反了：

**0.3.10 是恶意版本。最后一个安全版本是 0.3.9。方向是往回锁，不是往前升。**

三个 crate 的安全版本分别是：

- `arrayref` → **0.3.9**
- `internment` → **0.8.6**
- `append-only-vec` → **0.1.8**

如果你看到任何建议你"升级到 0.3.8 或更高"的说法，那句话在 8 月 20 日那天会把人直接送进恶意版本。现在恶意版本已经从 crates.io 删除了，装不到了，但请确认你手上的信息是对的。

### 检查本机是否真的拉过

官方给了一条直接查缓存的命令。这比查 `Cargo.lock` 更可靠——它反映的是"这台机器实际下载过什么"：

```bash
find ~/.cargo/registry/cache -type f \( \
  -name 'append-only-vec-0.1.9.crate' -o \
  -name 'arrayref-0.3.10.crate' -o \
  -name 'internment-0.8.7.crate' -o \
  -name 'proc-macro1-*.crate' -o \
  -name 'proc-macro-en-*.crate' -o \
  -name 'aovine-*.crate' -o \
  -name 'arone-*.crate' -o \
  -name 'aronenao-*.crate' -o \
  -name 'tinymember-*.crate' \
\) -print
```

没有输出 = 这台机器没下载过，可以放心。

### 顺便查一下 lockfile

```bash
grep -A2 'name = "arrayref"' Cargo.lock
grep -n 'proc-macro1' Cargo.lock
```

出现 `0.3.10` 或者任何 `proc-macro1` 条目，都意味着载荷在这台机器上执行过。

### 如果真的中了

这不是"升级一下就完事"的性质。编译期执行意味着攻击者拿到了构建机器上的用户权限：

1. 检查是否有被投放的二进制文件；
2. 检查网络出站记录，公开分析里点名的地址是 `23.254.165.112`；
3. **轮换这台机器上的所有凭据**——crates.io token、npm token、SSH key、云凭据、CI secret。构建机上有什么就轮换什么。

RustSec 的公告编号是 RUSTSEC-2026-0260。截至公告发布，没有分配 CVE，且没有证据表明有实际的大规模利用。但"没有证据"和"没有发生"不是一回事，尤其考虑到窗口期正好覆盖欧洲和亚洲的工作时间。

## 五、归因：不是随机的机会主义

Wiz 的分析指出，这次攻击的基础设施与近期几起朝鲜（DPRK）相关的供应链攻击有显著重叠，包括针对 `Mastra` 和 `axios` 的行动。

这一点改变了事件的性质。如果是随机的脚本小子，你可以把它当成一次运气不好的意外。但如果是有组织、有持续投入、会提前两天布置基础设施、会研究 cargo 的 yank 语义并加以利用的团队——那么这次成功与否不重要，重要的是**这套手法会被复用**。

yank-as-lure 这个技巧，对 npm 的 deprecate、PyPI 的 yank 同样适用。任何一个"维护者可以标记旧版本为不推荐"的生态，都存在同构的攻击面。区别只在于那个生态的构建过程有没有像 `build.rs` 这样默认执行代码的入口——npm 的 `postinstall` 有，Python 的 `setup.py` 有。

## 六、这件事暴露的结构性问题

短期的补救都是有边界的。真正的问题是 cargo 的信任模型里有几个默认值站在攻击者一边：

**默认值一：build.rs 无声执行。** 没有权限模型，没有网络访问声明，没有"这个 crate 想在编译期联网，是否允许"的确认。Cargo 团队讨论沙箱化构建脚本很多年了，但至今没有落地方案。

**默认值二：新增依赖不产生任何摩擦。** 一个十年零依赖的 crate 突然新增依赖，`cargo update` 不会多问一句。你能拿到的唯一提示是 lockfile 的 diff，而大多数团队的 lockfile diff 在 code review 里是被折叠的。

**默认值三：yank 提示无条件信任维护者账号。** 这次被利用的就是这一条。工具链默认账号说的话代表维护者意志，而账号是可以被偷的。

在这些默认值改变之前，能做的事情不多，但有几条确实有效：

- **CI 里禁止自动 `cargo update`。** 依赖升级必须是一个有人看 diff 的 PR。
- **构建环境和凭据分离。** 编译不需要你的云凭据，就别让它拿得到。
- **给 lockfile diff 加审查权重。** 尤其盯"新增依赖"这一类变更，不只看版本号涨跌。
- **`cargo vendor` 或依赖镜像**，对高价值项目值得付出这个成本——把"上游随时可以变"变成"上游变了我得主动同意"。

最后一句给这次事件的评价：Rust 团队的响应速度是漂亮的，86 分钟从报告到删除，还想到了 unyank 那一步。但响应快是运气好的一部分——Nextron Systems 的研究团队恰好发现并报告了它。如果没有那份报告，`arrayref` 0.3.10 会在 crates.io 上挂多久？

那才是这件事真正应该让人不舒服的地方。

---

**参考来源**

- [Supply chain attack on arrayref — Rust Blog（官方公告）](https://blog.rust-lang.org/2026/08/20/supply-chain-attack-on-arrayref/)
- [Malware: arrayref 0.3.10 executes a remote payload at build time via typosquatted proc-macro1 — rustsec/advisory-db #3161](https://github.com/rustsec/advisory-db/issues/3161)
- [Malicious Rust Crate arrayref Runs a Build-Time Payload — SafeDep](https://safedep.io/arrayref-proc-macro1-rust-build-time-malware/)
- [Rust Supply Chain Attack on arrayref: Significant Overlap with DPRK Campaigns — Wiz](https://www.wiz.io/blog/rust-supply-chain-attack-on-arrayref-significant-overlap-with-dprk-campaigns)
- [Rust Crates arrayref & append-only-vec Compromised — Semgrep](https://semgrep.dev/blog/2026/rust-crates-arrayref-append-only-vec-compromised-proc-macro1/)
- [arrayref, internment, and append-only-vec Poisoned by the proc-macro1 Build-Time Dropper — StepSecurity](https://www.stepsecurity.io/blog/arrayref-rust-crate-supply-chain-attack)
