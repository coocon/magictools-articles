# Claude 没有写那个打印机驱动：14KB 胶水代码、一个 Linux 容器，和 HP 自己的二进制

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/claude-code-printer-driver-14kb-glue?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/claude-code-printer-driver-14kb-glue?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：Claude 没有写那个打印机驱动：14KB 胶水代码、一个 Linux 容器，和 HP 自己的二进制](https://tools.cooconsbit.com/zh/articles/claude-code-printer-driver-14kb-glue?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
