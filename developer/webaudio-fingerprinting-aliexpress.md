---
title: "零增益的声音：一次 WebAudio 指纹排查，和一个被夸大的结论"
slug: webaudio-fingerprinting-aliexpress
summary: "有人发现打开速卖通页面后，蓝牙多点耳机就切不回手机了。排查下去是两个隐藏的 AudioContext——增益设成 0，但仍连着系统音频输出，于是浏览器一直在真实处理音频，把蓝牙通道摁住了。这个排查过程值得学。但顺着它传出去的结论「WebAudio 指纹是下一个大威胁」，被 Firefox 指纹防护负责人当场否掉了。"
category: developer
tags: [WebAudio, 浏览器指纹, 隐私, 前端调试, Firefox, 反追踪]
coverImage: ""
status: published
locale: zh
source: authored
translationSlug: webaudio-fingerprinting-aliexpress-en
---

# 零增益的声音：一次 WebAudio 指纹排查，和一个被夸大的结论

> "Given enough victims, your attack code is going to change something that makes someone notice."
> —— Tom Ritter, Firefox 指纹防护技术负责人

---

这件事是从一个纯粹的日常烦躁开始的。

一位开发者的蓝牙耳机支持多点连接，同时连着电脑和手机，电脑音频优先，手机在电脑不出声时能正常播放。这套组合一直工作得很好——直到他在 Firefox 或 Chrome 里打开速卖通的页面。

页面加载几秒后，手机上的音乐停了。关掉那个标签页，立刻恢复。

而且：**静音标签页没用，静音 Firefox 没用，静音 Windows 也没用**。页面上没有任何可见的视频、音乐或者媒体元素。

这个现象足够怪，值得挖一下。挖出来的东西，比"又一家电商在追踪你"有意思得多。

## 一、排查过程：这部分值得抄下来

第一反应都是自动播放的产品视频或广告。所以先查常规嫌疑人：

- `<audio>` 和 `<video>` 元素
- `HTMLMediaElement.play()` 调用
- 活动的 Media Session 元数据
- 媒体资源请求
- 内嵌 iframe 里的媒体

**全部没有。** 没有音视频元素，没有播放调用，`navigator.mediaSession.playbackState` 一直是 `none`。

这里有个关键线索被抓住了：**问题不是立刻出现的，而是页面空闲几秒之后才出现。** 延迟意味着这不是加载时的资源，是运行中某段脚本主动做的事。

于是排查方向从"找媒体元素"转向了"监听 Web Audio API"。手法很简单——在页面加载前包装 `AudioContext` 构造函数：

```js
const OriginalAudioContext = window.AudioContext;

window.AudioContext = class extends OriginalAudioContext {
  constructor(...args) {
    super(...args);
    console.log("AudioContext created", {
      state: this.state,
      stack: new Error().stack,
    });
  }
};
```

同时包装 `AudioNode.prototype.connect()`，看有没有东西连到了音频上下文的 destination。

**这一下就抓到了：两个隐藏的 AudioContext。**

在一次纯空闲的首页抓取里，页面创建了两个 `AudioContext` 对象，两个都进入 `running` 状态，两个都把节点连到了 `AudioContext.destination`。与此同时：零个音视频元素、零次 `play()` 调用、没有活动的 Media Session、没有任何声音。

构造函数的调用栈指向两个脚本：

```
https://assets.aliexpress-media.com/g/AWSC/uab/1.140.0/collina.js
https://assets.aliexpress-media.com/g/AWSC/fireyejs/1.231.67/fireyejs.js
```

两个都在 `AWSC` 目录下，属于阿里的浏览器安全与反滥用工具链。第一个上下文由 `collina.js` 创建，第二个来自 `fireyejs.js`。

这套「包装构造函数 + 包装 connect」的方法是通用的。任何时候你怀疑页面在偷偷用某个 Web API 而 DevTools 面板里看不到，这个思路都成立——**在 API 入口打桩，把调用栈打出来**，比逐个读混淆代码快得多。

## 二、音频图长什么样，以及副作用是怎么来的

脚本混淆得很厉害，但足够的符号和操作残留下来，能还原出音频图的结构。两个脚本构建的图大致是：

```
Sawtooth oscillator（锯齿波振荡器）
  → AnalyserNode（分析节点）
  → ScriptProcessorNode
  → GainNode（增益设为 0）
  → AudioContext.destination
```

逻辑很清楚：振荡器产生一个已知波形，让它穿过浏览器的音频实现，再用分析节点测量出来的频率数据。同一个输入波形，在不同的浏览器、不同的 CPU、不同的音频栈上算出来的结果会有细微差异——这就是 WebAudio 指纹的原理。

**增益设成 0，所以用户听不见任何东西。**

但关键在这里：**图仍然连接到了 `AudioContext.destination`。**

连到 destination 意味着浏览器必须真实地处理这个音频图——即使最终音量是零。对系统来说，这个页面正在进行实时音频处理。

这就解释了那个蓝牙的怪现象。系统音频通道被持续占用着，Firefox 或 Windows 因此把蓝牙音频路径保持在活跃状态，多点耳机就没法干净地切回手机。

也解释了为什么静音没用：**没有媒体元素可供标签页的静音控件去停止。** 浏览器的 tab 静音功能是围绕 `HTMLMediaElement` 建的，而这里根本没有 media element。这跟一个自动播放的视频完全是两码事。

## 三、然后，结论被反转了

这个帖子在 Hacker News 拿了 725 分，传播过程中自然长出了一个结论：**WebAudio 指纹是比 Canvas 指纹更难拦、更隐蔽的下一代追踪手段，该重点防了。**

这个结论是错的，而且否掉它的人份量很足。

Firefox 指纹防护的技术负责人 Tom Ritter 在 8 月 20 日发了一篇评估，开门见山：

> 浏览器指纹确实是一种被滥用得过分的跨站追踪手段，但**具体到 WebAudio，它的效果很差**。Firefox 已经基本消除了这个指纹向量。

数据很硬。Firefox 从 **118 版本**（三年前）就把 WebAudio 的计算结果常量化了，这是第一轮指纹防护特性的一部分。遥测数据显示：

- **99.24% 的用户**落在**三个**取值之一
- 0.76% 的用户数据采集失败（值为 0）
- 长尾是 23 个其他取值，共 48 个用户

一个只能把用户分成三桶的信号，作为指纹几乎没有价值。

那为什么是三个而不是一个？Ritter 给了答案，而且这个答案本身挺有意思：**三个桶的差异来自 CPU 架构层面**——音频处理是密集的浮点运算，硬件指令集的差异会渗透到结果里。

- 一个值来自所有 x86 CPU，以及不支持 FMA（fused multiply-add）指令的 x64
- 一个值来自支持 FMA 的 x64
- 一个值来自带 NEON 指令集的 ARM

Firefox 还在继续收敛：Bug 2036977 已经把 x64（含 FMA3）合并进 x86/x64（无 FMA3）那个桶；Bug 2040494 计划把剩下那个也合并进 NEON 桶，只是优先级排在更大的改进后面——比如**净化 WebGL 的 renderer 和 vendor 字符串**，那才是当下更肥的熵源。

Ritter 也提到，Chrome 的 WebAudio 代码大概很多年前就做了类似的常量化处理，Brave 和 Safari 也应该有防御，不过可能仍然泄露 CPU 架构。

所以准确的说法是：**WebAudio 指纹在隐私优先的浏览器上已经基本失效了。** 它对全网大多数用户仍然有效——因为大多数用户不在那些浏览器上——但它绝不是"下一个该堵的口子"，它是一个三年前就被堵得差不多的口子。

## 四、那真正的问题是什么

如果指纹本身没什么用，这件事还剩下什么值得关心的？

剩下两件，而且都比指纹本身更重要。

**第一，不可观测性。** 一个网页可以让你的音频硬件保持活跃状态，产生真实的、可被用户感知的副作用（耳机切不回去、笔记本音频芯片不进省电状态、蓝牙功耗上升），而**浏览器 UI 完全不提示**。标签页上没有小喇叭图标，因为那个图标只跟踪 media element。

HN 讨论里有人问得很直接：为什么 Firefox 允许一个页面用 0 音量播放音频而不显示标签页指示器？这是个合理的产品缺口。有人呼吁浏览器在检测到站点主动绕过隐私保护时，在状态栏给出警告。

**第二，动机可能没那么坏，但不影响结论。** HN 上有个反驳很值得看：这未必是间谍行为，可能是一种荒谬的优化——音频设备为了省电会休眠，唤醒要一两秒，所以持续播放点什么（哪怕是静音）能保证产品视频点开就响。

另一种解读来自 Lobsters：它不是想"听"什么，而是强迫音频子系统真实处理生成的信号，好从旁路里榨出更多的每设备熵。

这两种解释都说得通，而且**都指向同一个问题**：不管动机是优化还是采集，用户都为此付出了真实的代价，且没有得到任何提示或选择权。意图不改变副作用。

## 五、给开发者的实操

### 检查你自己的站点有没有这种东西

你引的第三方 SDK（风控、反爬、广告、统计）里可能就有。打开 DevTools，在 Sources 面板全局搜：

```
AudioContext
OfflineAudioContext
createOscillator
createAnalyser
```

或者更直接，把前面那段包装代码贴进 console 之前先跑一遍（要在页面脚本执行前注入才最可靠，可以用浏览器扩展或 `--auto-open-devtools-for-tabs` 配合断点）：

```js
// 检测页面上所有活跃的 AudioContext 及其创建来源
const OriginalAC = window.AudioContext || window.webkitAudioContext;
window.AudioContext = window.webkitAudioContext = class extends OriginalAC {
  constructor(...args) {
    super(...args);
    console.warn("[audio-probe] AudioContext created", {
      state: this.state,
      stack: new Error().stack,
    });
  }
};

// 顺便看谁连到了 destination
const originalConnect = AudioNode.prototype.connect;
AudioNode.prototype.connect = function (dest, ...rest) {
  if (dest instanceof AudioDestinationNode) {
    console.warn("[audio-probe] node connected to destination", new Error().stack);
  }
  return originalConnect.call(this, dest, ...rest);
};
```

如果你做的是需要通过合规审查的产品，这件事值得主动查一遍——你可能在不知情的情况下引入了第三方的指纹采集，而责任是你的。

### 如果你只是想让耳机正常工作

原作者的方案是用 uBlock Origin 拦掉那两个脚本：

```
||assets.aliexpress-media.com/g/AWSC/uab/*/collina.js
||assets.aliexpress-media.com/g/AWSC/fireyejs/*/fireyejs.js
```

**但这里有个必须说明的代价。** 意大利媒体 ilsoftware 指出：这些脚本是阿里风控系统的一部分，永久屏蔽不是没有后果的——站点可能用额外的验证码、身份核验来回应，甚至可能在登录或支付环节出问题。

所以更务实的做法是按站点开关，而不是全局屏蔽；或者干脆用一个独立的浏览器 profile 逛购物网站。

### 如果你在做反追踪工具

Ritter 的评估其实给了一份优先级清单：别把力气花在 WebAudio 上，那已经是收敛完的战场。**WebGL 的 renderer 和 vendor 字符串**才是当下熵值更高、更值得处理的目标。

---

最后回到 Ritter 那句话，它其实是这整件事最好的注脚。他说他不太同意"眼睛足够多，bug 就无所遁形"这句老话，更真实的版本是：

**受害者足够多，你的攻击代码总会改变某个东西，让某个人注意到。**

这次注意到的人，是一个只想让耳机正常切回手机的普通用户。

---

**参考来源**

- [AliExpress webpage keeping multipoint Bluetooth headphones active with WebAudio fingerprinting — laserphile（原始发现）](https://blog.laserphile.com/2026/08/aliexpress-webpage-keeping-multipoint.html)
- [webaudio fingerprinting on alibaba — Tom Ritter / ritter.vg（Firefox 指纹防护评估）](https://ritter.vg/blog-webaudio_alibaba.html)
- [Hacker News 讨论](https://news.ycombinator.com/item?id=49372583)
- [Lobsters 讨论](https://lobste.rs/s/b0olmy/aliexpress_keeps_multipoint_bluetooth)
- [AliExpress usa WebAudio per il fingerprinting e disturba il Bluetooth? — ilsoftware.it](https://www.ilsoftware.it/aliexpress-audio-script-browser-problemi-cuffie-bluetooth/)
