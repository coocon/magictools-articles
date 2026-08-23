# 零增益的声音：一次 WebAudio 指纹排查，和一个被夸大的结论

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/webaudio-fingerprinting-aliexpress?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/webaudio-fingerprinting-aliexpress?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：零增益的声音：一次 WebAudio 指纹排查，和一个被夸大的结论](https://tools.cooconsbit.com/zh/articles/webaudio-fingerprinting-aliexpress?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
