# Mojo 开源那天，它的作者已属于高通

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/mojo-open-source-qualcomm-acquires-modular?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/mojo-open-source-qualcomm-acquires-modular?utm_source=github&utm_medium=referral)**

> 素材来自 Modular 官方博客（ModCon 2026 公告、Mojo 开源声明、Modular × Qualcomm 技术博客）与 WIRED 2026 年 6 月 24 日的收购报道。引语均为原文。

---

2026 年 8 月 18 日，Modular 在 ModCon 上宣布：Mojo 1.0 完全开源，Apache 2.0 协议，编译器和全部工具链公开在 GitHub 上。

对一门以「取代 CUDA」为使命的语言来说，这是它能做的最彻底的一件事。

但那份公告的页脚写着一行小字：**Copyright © 2026 Modular Inc, A Qualcomm Company.**

六月，高通宣布以近 40 亿美元收购 Modular，交易在 ModCon 前完成交割。等到 Mojo 的编译器源码真正公开的那天，写它的人已经在高通的组织架构图里了。

Chris Lattner 用了四年时间，想以一种「结构性」的方式打破一家芯片公司对 AI 软件栈的垄断。

结果是另一家芯片公司把他买了下来。

下面是这件事里，我认为真正值得琢磨的 10 个点。

---

## 1. 40 亿美元买的不是编译器，是「不用重写」这四个字

> "We believe the future belongs to developer-friendly, horizontal platforms that can run across diverse compute environments and give customers real choice in how and where they deploy AI."
>
> —— Cristiano Amon，高通总裁兼 CEO
>
> 译：我们相信未来属于对开发者友好的横向平台，它能跨越各种计算环境运行，让客户在如何部署、在哪部署 AI 上拥有真正的选择权。

先看价格。高通将发行最多 1920 万股普通股，按公告前收盘价算接近 40 亿美元。而九个月前，Modular 刚以 16 亿美元估值融了 2.5 亿。九个月，2.5 倍。

团队规模是 2 位联合创始人加上约 150 名员工，全部加入高通。按人头摊，每人约 2600 万美元。

这个价格显然不是在给一个编译器估值。高通买的是一句承诺：客户换硬件的时候，代码不用重写。

Modular 技术博客里那句话说得最直白：每有一款新 AI 加速器上市，开发者都要问「我们的软件栈要重写多少」，而 Modular 的答案是——**一行都不用**。

**My take：** 高通买的是时间，不是技术。芯片公司自建一套能跟 CUDA 抗衡的软件栈，历史给出的答案是五到十年，而且大概率失败——AMD 的 ROCm 已经证明了这条路多难走。40 亿美元换掉这五年，在一个 AI 推理硬件三年一换代的市场里，是笔便宜账。真正贵的从来不是芯片，是让人愿意为你的芯片写代码。

---

## 2. 「结构性」这个词，最后反噬了提出它的人

> At the time, Lattner said he believed that he and Davis were tackling a software problem that had to be solved outside of a Big Tech environment because it was "structural." Ultimately, the structure of Qualcomm won out.
>
> —— WIRED，2026 年 6 月 24 日
>
> 译：当时 Lattner 说，他相信他和 Davis 要解决的软件问题必须在大公司之外解决，因为这是个「结构性」问题。最终，赢的是高通的结构。

这是整篇 WIRED 报道里最锋利的一句，也是整件事的题眼。

Lattner 的原始判断是：AI 软件栈的碎片化不是技术问题，是结构问题——每家芯片公司都有动机把软件绑死在自家硬件上，所以在任何一家大公司内部都解不开。要解，必须站在所有硬件之外。

这个判断本身没错。错的是，他低估了「站在所有硬件之外」这个位置的经济学。

**My take：** 中立的软件层有一个先天缺陷——它对谁都有用，因此谁都不肯为它付真正的钱。芯片厂愿意为「能让我的芯片卖出去的软件」付钱，不愿为「能让我竞争对手的芯片也卖出去的软件」付钱。想解决结构性问题，就需要结构性的资源；而在这个行业里，结构性资源只有芯片公司手里有。这不是 Modular 一家的困境，是所有中立基础设施创业公司的宿命。

---

## 3. 他从来没有真正离开过硬件公司

> "What makes this team truly exceptional is the complementary partnership between Chris and Tim. Chris is an N-of-1 human, in that he's bold, visionary, and technically uncompromising."
>
> —— Dave Munichiello，GV 管理合伙人、Modular 早期投资人
>
> 译：这个团队真正卓越之处在于 Chris 和 Tim 之间的互补。Chris 是那种世上只此一个的人——他大胆、有远见、在技术上从不妥协。

把 Lattner 的履历摊开看，会发现一条别的线索。

LLVM 起于伊利诺伊大学的硕士论文，2005 年被苹果买下时才真正工程化。Clang、Swift、Xcode 工具链，全部诞生在苹果内部，他在那里待了 12 年。MLIR 诞生在谷歌，服务于 TPU。中间他去了特斯拉负责 Autopilot 软件——那段只持续了不到五个月，后来这个位置由 Andrej Karpathy 接手。再之后是 SiFive，一家 RISC-V 芯片公司。

...

---

**[👉 继续阅读全文：Mojo 开源那天，它的作者已属于高通](https://tools.cooconsbit.com/zh/articles/mojo-open-source-qualcomm-acquires-modular?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
