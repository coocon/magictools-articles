---
title: "Claude 没有写那个打印机驱动：14KB 胶水代码、一个 Linux 容器，和 HP 自己的二进制"
slug: claude-code-printer-driver-14kb-glue
summary: "一条「Claude 给只支持 Windows 的 HP 打印机写了 macOS 驱动」的推文拿到 262 万阅读。把仓库拉下来一看：Shell 6497 字节、Python 7638 字节、Dockerfile 550 字节，没有一行 C，真正做编码的是 HP 官方 Linux 驱动里的 rastertospl 二进制，跑在 Mac 上的 Linux 容器里。HN 上一群人指出了这件事。但这篇文章想说的不是拆台——四小时会话里真正有含金量的部分（读打印机吐出的错误页、绕过 CUPS 沙箱、libusb 直写），以及那个决定性转折点是人提出来的、不是 Claude 想到的，才是 AI 干脏活能力边界的准确刻度。"
category: claude
tags: [Claude Code, 逆向工程, 打印机驱动, CUPS, macOS, AI 边界, 开源, HN 讨论]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: claude-code-printer-driver-14kb-glue-en
---

# Claude 没有写那个打印机驱动：14KB 胶水代码、一个 Linux 容器，和 HP 自己的二进制

8 月 17 日，19 岁的开发者 Kuber Mehta 发了一条推：

> "just Claude writing a MacOS driver for my obscure HP printer built only for Windows support"

两天时间，**262 万阅读、14805 个赞**。作者当时只有两千多粉丝，这是一次彻底的自然爆发。Hacker News 上开了两个帖子（151 分和 105 分），媒体也跟进了，wccftech 的标题更进一步，写成了 "Wrote A Driver For macOS From Scratch"。

我把仓库拉下来看了一眼。代码构成是这样的：

| 语言 | 字节数 |
|------|--------|
| Shell | 6,497 |
| Python | 7,638 |
| Dockerfile | 550 |

**总共约 14KB，没有一行 C，没有编译型 filter，没有内核代码。**

而 README 里作者自己写着：

> "This repo contains only glue code. It does **not** redistribute HP's driver. `install.sh` downloads the Unified Linux Driver from HP at install time."

真正把页面数据编码成打印机能懂的格式的，是 **HP 官方 Linux 驱动里的 `rastertospl` 二进制**——被放进 Mac 上的一个 Linux 容器里跑。

所以标题不成立。但如果文章到这里就结束，那才是真的看漏了：这四个小时里发生的事情，比「写了个驱动」这个假命题有意思得多，而且它精确地画出了今天 AI 干脏活的能力边界在哪。

---

## 一、这台打印机为什么在 macOS 上是块砖

HP Laser 1008a 属于 HP Laser 1000 系列。关键背景：**这是 HP 2017 年收购三星打印机业务之后的换标三星机型**，血统不是 LaserJet。

它带来三个后果：

1. **说的是 SPL/QPDL 语言**（三星那一系的页面描述语言），不是 PCL，也不是 PostScript。README 里那行表格写得很直白：Generic PCL / PostScript → "Printer speaks neither, so the CUPS backend hangs 'offline'"。
2. **是 host-based 打印机**——机器里没有解释器，光栅化得在主机端做完再传过去。俗称「傻瓜打印机」。
3. **HP 从来没出过 macOS 驱动**，Linux 侧的官方方案是三星血统的 ULD（Unified Linux Driver），最新一版发布于 2020 年。

开源社区那边也没有现成答案。这一点很重要，我逐个核过：

| 项目 | 支持 1008a 吗 |
|------|--------------|
| **hplip** | 不支持。Launchpad 上的官方答复：「The HP Laser 100 series is not supported by HPLIP」——这个系列是三星血统，走 ULD 不走 hplip |
| **foo2zjs** | 协议就不对。它支持的 HP LaserJet 1000/1005/1018/1020 是老的 ZjStream 系，和「HP Laser」（没有 Jet）1008a 是两码事 |
| **splix** | 名义上支持，实测失败。splix 2.0.2 通过 PR #9 加了 "HP Laser 10x" 支持，但**该 PR 原文自己写着 "HP are not tested"**；而且它面向的是 HP Laser 103-108 那条线，1008a 属于另一条产品线，SPL3 帧格式有差异 |

所以这不是一个「翻翻 GitHub 就有」的问题。这台在亚马逊上正常售卖的打印机（HN 用户 1970-01-01 专门吐槽过 "obscure" 这个形容：「Obscure? Sir, the printer is sold on Amazon.」），在 Mac 上确实是块砖。

## 二、四个小时里真正发生了什么

作者把完整的 Claude Code 会话记录（209KB Markdown）公开了，这是这件事最有价值的部分——它让整个过程可复核。

起点是一句极其随意的 prompt：

> "hey can you set up the drivers for HP Laser 1008a on this mac please, it's connected"

然后是一条长长的失败链：

1. **Generic PCL PPD** → 队列卡死在 "offline-report connecting-to-device"
2. 搜索确认是三星血统的 SPL 机型，试 **splix-macos（ML-2010 PPD）** → **打印机固件被之前灌进去的 PCL 垃圾数据搞挂了**，`STATUS:BUSY`，需要用户物理断电重启
3. 查 USB 描述符，发现接口的 `bInterfaceProtocol = 4`，即 **IPP-over-USB**——这解释了 macOS 自带 `usb` backend 为什么一直误判 offline
4. 绕行方案：用 pyusb/libusb **直写 USB bulk endpoint**。先撞上 macOS 内核驱动占用（EACCES），确认**只有 root 能 detach**
5. 首次直写 QPDL：**出纸了，但内容是乱码，而且连吐 4-5 张**。用户的原话是 "it printed... something LOL back to back 4-5 times"，还附了照片
6. 试 raw PWG/URF 直写 → 被打印机静默吞掉；试 IPP-over-USB 通道 → HTTP 通了但 `/ipp/print` 返回 404
7. 试 `foo2qpdl -z0` → 打印机自己打出一张错误页：**"SPL ERROR - Please use the proper driver"**；试 1200x600 分辨率 → 又打出一张：**"SPL ERROR Illegal Resolution ... ERROR CODE: 11-1113"**

第 7 步这里有一句话，我认为是整份记录里最漂亮的技术观察，来自 Claude：

> "the printer's error page is basically telling us how to fix it each time."

**打印机每次都在用打印出来的纸告诉你哪儿错了。** 把厂商固件的错误报告当成调试通道用——这是很老派、很对的工程直觉。

## 三、转折点不是 Claude 想到的

走到这一步，方向仍然是错的：还在试图用开源实现凑出一个能被打印机接受的 SPL3 编码器。splix 的 600dpi A4 波段宽度表都调对了（608 字节），输出还是「页面原点处的条纹光栅，反复出纸」。

**决定性的一步来自用户。** 他建议：别再逆向了。

Claude 的回应，原文：

> "Oh, that's a much better instinct than reimplementing — **we don't need to reverse-engineer or clean-room anything. The actual SPL3 codec is freely downloadable.**"

于是从 ftp.hp.com 下载 ULD 压缩包，打开一看——里面有 **aarch64 版本的 `rastertospl`**，能在 Apple Silicon 上的 Linux 容器里原生跑。

第一张干净的测试页就是这么打出来的。

我想把这一段单独拎出来，因为它是全文最有信息量的地方：

**在「怎么试」上，Claude 表现得很强。** 它读懂了 USB 描述符、认出了 IPP-over-USB 协议号、诊断出 backend 误判的根因、想到了 libusb 直写、发现了只有 root 能 detach 内核驱动、把打印机的错误页当成反馈信号来迭代。这些都是需要真本事的活，而且它连续做对了。

**但在「要不要试」上，它一头扎进了七步弯路。** 因为初始 prompt 说的是「装驱动」，它就一路朝着「造一个能用的编码器」走，从没停下来问一句：这个编码器是不是根本不需要造，HP 官网上就摆着？

这个失败模式非常典型。**模型极其擅长在你划定的方向里穷举，但不擅长质疑方向本身。** 那句「别逆向了」值多少钱？它抵掉了前面所有的努力，并且是这个项目唯一真正解决问题的决策。

Matthew Garrett（mjg59，内核和固件领域的知名工程师）在 HN 上给了一个我觉得最准确的表述：

> "My experience is that they're better than me at a lot of the process... so having some skills that are pretty much 'This smells wrong' helps a lot"

流程上它比你强，**「这里味道不对」的嗅觉还得你自己有**。

## 四、最终方案长什么样，以及它绕过了什么

最后落地的架构是这样的：

```
任意 App 按 Cmd-P
  → CUPS 队列（用 HP 官方 PPD）
  → CUPS raster
  → CUPS 自带的 socket backend（socket://127.0.0.1:9108）
  → root LaunchDaemon（Python 脚本，跑在沙箱外）
  → HP rastertospl（在 colima Linux 容器里，输出真正的 SPL3）
  → libusb 直写打印机的 USB bulk endpoint
```

这里面确实有真东西。macOS 的 CUPS 有强制沙箱，**禁止 filter 和 backend 调用容器或直接操作 USB**。绕过的方法是：不写 filter，改用 CUPS 自带的 socket backend 把数据发到 localhost 一个端口，端口那头是一个跑在沙箱外的 root 守护进程，由它去调容器转码、再用 libusb 写 USB。

等于**自建了一个 backend 替代品**，同时绕开了 macOS 自带 `usb` backend 的 offline 误判和 CUPS 的沙箱限制。这个设计是这个项目里最像工程的部分。

代价也很实在：

- **必须常驻一个 colima Linux VM**（重启后第一次打印最多等约 1 分钟，等 VM 起来）
- **一个 root LaunchDaemon 在执行用户目录 `~/.hp1008` 下的代码**——HN 用户 Tiberium 专门点了这条："It also requires a root launcher that runs code from the user ~/.hp1008 dir, so security is weakened."
- 换个 USB PID 就得手改 `direct_write.py`
- 只在 macOS 26 / Apple Silicon 上验证过

效果方面：600dpi、A4、干净的单页文本测试页，转码加写入约 1 秒。作者提供了照片。**但要注意，全部实物证据都是作者单方提供的，我没有找到任何第三方复现报告。**

## 五、HN 怎么拆的，以及作者怎么回应的

两个帖子下面，质疑相当集中：

> **ssdspoimdsjvv**：「If I understand correctly, it just wraps an existing Linux driver in a container. You can hardly call that writing a driver.」

> **Tiberium**（在两个帖子里都发了）：「Unfortunately this is a very misleading post... Claude didn't write a driver. It basically used HP's existing proprietary driver in a Linux VM on macOS, and just bridged that to macOS.」

> **asveikau**：「It didn't write a driver. It used a linux driver. You don't need an LLM.」

> **3129476** 还翻出了前人工作：「Claude did not write any macOS driver. It uses the HP Linux driver inside docker. Here is prior art from 2017...」——2017 年就有人写过「在 Docker 容器里跑 Linux 打印驱动」的教程。另一位 HN 用户 oneplane 指出已经有现成产品在做同一件事（printervention.app，在浏览器 WebUSB 里跑 Linux VM 驱动打印机）。

也有中间派。mariuolo 的评价大概是最公道的一句：「But it's not completely native, it's a wrapper around the original linux driver... **Still better than nothing.**」happyPersonR 甚至说他更喜欢这个方案："Makes it so if there's an issue I don't get a kernel panic."

**作者的应对值得一提。** 他没有硬扛：

- 发帖后补测了 splix 2.0.2，更新 README，commit 信息写着 "Correct SpliX claim + apply review fixes"
- 措辞从「写驱动」往回收（HN 用户 feintruled 注意到了这一点："I note the claim on this page is walked back from the original 'writing the driver'"）
- 在 README 里公开求助：「If you can pin down the exact byte-level difference between HP's rastertospl output and SpliX's for this printer... SpliX could likely be patched and colima dropped entirely.」
- 仓库唯一那个 issue（#1 "SpliX support"），正是 splix PR #9 的作者 ValdikSS 开的

一个 19 岁开发者，一条推爆到 262 万阅读，然后老老实实回收措辞、补做实验、开源全部会话记录供人核查——这个处理方式比那条推本身更值得夸。

他在 HN 上自己总结方法论时说的也很实在：「there's not really a hack for it, you just work with it, convey thoughts decently and try out things with educated guesses until it sticks」。

## 六、我的解读

**第一，「AI 写了个 X」这类标题，先做一次层级折算。** 通用做法是三步：拉仓库看代码构成（语言分布和字节数最诚实）、找 README 里的依赖声明、搜一遍这个领域有没有现成的开源/厂商实现。这次三步都指向同一个答案，五分钟就能查完。作者本人的 README 其实第一句就写了 "only glue code"——**夸张发生在推文标题和媒体转载里，不在代码仓库里**。

**第二，AI 在这类任务里省掉的是「试」的成本，不是「想」的成本。** 拆开看，四小时里 Claude 做对的事有个共同点：都是在明确方向下的高强度试错和信息检索——查 USB 描述符、认协议号、写 libusb 直写、读错误页迭代、绕沙箱设计。这些活人类也会做，只是要花几天而不是几小时。而它没做对的那一件——「这个轮子根本不用造」——恰恰是最省事的那个念头。**方向性的怀疑，目前仍然是人的活。**

**第三，别急着照着装。** 这个方案要常驻一台 Linux VM，要一个 root 守护进程去执行你 home 目录下的脚本，只在一种硬件和一个 macOS 版本上验证过，且没有任何第三方复现。它是一份很好的技术记录，不是一个可以推荐给别人的产品。作者自己都在 README 里请人帮忙把 colima 干掉。

**第四，真正可迁移的是那句「打印机的错误页在告诉你怎么修」。** 硬件和厂商固件通常会吐出比你想象中多得多的诊断信息——错误页、状态码、描述符、日志。把这些接进 AI 的反馈回路，比让它凭空「逆向」有效得多。HN 那条讨论串里几个成功案例（有人用 Claude + ILSpy + Wireshark 逆向高尔夫球车电控，有人 5 小时搞定 Xbox 手柄适配，有人做 epaper 的嵌入式 Rust 驱动）都是同一个模式：**给它一个能反复读到真实反馈的闭环，而不是给它一个纯猜的任务。**

那条推的标题是错的。但底下那 209KB 的会话记录，是我最近读到的关于「AI 到底能替你干哪部分脏活」的最好材料——恰恰因为它把七步弯路和那个人类给出的转折点都原样留在了里面。

---

**参考链接**

- [原推 @kuberwastaken（262 万阅读）](https://x.com/kuberwastaken/status/2089377982536388964)
- [完整 Claude Code 会话记录（209KB）](https://cdn.kuber.studio/chat/hp-laser-1008a-driver)
- [GitHub: Kuberwastaken/hp-laser-1008a-macos](https://github.com/Kuberwastaken/hp-laser-1008a-macos)
- [HN 讨论帖一（151 分）](https://news.ycombinator.com/item?id=49344643)
- [HN 讨论帖二（105 分）](https://news.ycombinator.com/item?id=49352806)
- [splix PR #9（原文自述 "HP are not tested"）](https://github.com/OpenPrinting/splix/pull/9)
- [hplip 官方答复：不支持 HP Laser 100 系列](https://answers.launchpad.net/hplip/+question/694005)
- [HP Laser 1000 系列官方驱动页](https://support.hp.com/in-en/drivers/hp-laser-1000-printer-series/model/2101513671)
- [Pierov：HP Laser 107a on Linux（前置参考）](https://www.pierov.org/2023/07/25/hp-laser-107a-linux/)
- 站内相关：[Claude Code 完全指南](/articles/claude-code-guide) · [「Codex 提速 232 倍」的真相](/articles/codex-autoresearch-gpu-kernel-232x) · [「关灯」软件工厂跑了四个月](/articles/lights-off-software-factory-postmortem)
