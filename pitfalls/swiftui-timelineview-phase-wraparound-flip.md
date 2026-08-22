---
title: "蜗牛在你点开的瞬间原地调头：一帧的时钟误差，被取模回绕放大成了最大误差"
slug: swiftui-timelineview-phase-wraparound-flip
category: pitfalls
locale: zh
tags: [SwiftUI, TimelineView, 动画, iOS, Bug 复盘, 交叉淡化]
summary: "SwiftUI 里用 TimelineView 驱动一张吉祥物场景卡，两个毛病：第一次点开音色时蜗牛原地镜像翻转一下，停止播放时背景硬切。前者查了两轮——第一轮以为是 timeline 暂停导致时间源错位，改完仍在；真正的根因是相位取模时那句 `raw < 0 ? raw + 1 : raw`，它把「timeline.date 比 Date() 早了一帧」这个几十毫秒的误差，回绕放大成了 0.9999 的相位，而朝向恰好是相位在 0 处的不连续函数。这篇复盘讲清楚为什么在 TimelineView 里不能用 .transition、怎么把动画状态写成时间的纯函数、以及交叉淡化为什么要设计成「只有淡入没有淡出」。"
status: published
source: authored
translationSlug: swiftui-timelineview-phase-wraparound-flip-en
---

## 现象

耳鸣舒缓 App 的声音页顶部有一张常驻场景卡：吉祥物蜗牛 Coco 站在平地上，用户点开雨声，背景换成雨景、Coco 开始极慢地来回爬（一个来回 90 秒）；停止播放，背景换回平地。整张卡片包在 `TimelineView(.animation(minimumInterval: 1/20))` 里，每秒重算 20 次。

两个毛病，用户的原话是：

1. **第一次**点自然声的时候，蜗牛会原地左右调头一下。第二次、第三次都不会。
2. 打开有渐入，关闭太生硬，背景是硬切的。

第二个是设计问题，第一个是真 bug——而且它查了两轮才见底。

## 为什么不能用 SwiftUI 自带的过渡

先交代前提，否则后面的代码看着像在造轮子。

这张卡最初是想用 `.transition(.opacity)` 加 `withAnimation` 做背景切换的，实际表现是**渐变时有时无，第一次几乎必然没有**。原因是 SwiftUI 的过渡依赖「状态变更发生在一个带动画的事务（transaction）里」，而 `TimelineView` 每 50ms 就要重算一次 body，那次重算会把事务吞掉——过渡还没开始就被下一帧的无动画事务覆盖了。

结论：**在 TimelineView 内部，不要依赖 SwiftUI 的隐式/显式动画**。时间轴反正每帧都把 `t` 递给你，那就自己用 `t` 算进度。

## 骨架：把动画状态写成时间的纯函数

于是场景卡里的每一样会动的东西，都是一个纯值语义的小状态机，对外只暴露 `f(t) -> 值`：

```swift
/// 爬行进度时钟：走-停-走之间保留相位（停在哪就留在哪，恢复时从原地继续）
struct WalkClock: Equatable {
    private(set) var frozenPhase: Double = 0   // 停住时的相位（0..1，三角波一个来回）
    private(set) var origin: Double?           // 在走时的相位原点（nil = 停住）

    func phase(at t: Double, period: Double) -> Double { ... }
    mutating func start(at t: Double, period: Double) {
        guard origin == nil else { return }
        origin = t - frozenPhase * period      // 从冻结的相位续走
    }
}
```

同一套写法还有两个：`AmplitudeRamp`（蠕动幅度的 smoothstep 渐起渐停）和 `SceneFade`（场景交叉淡化的进度）。它们全是 struct、无副作用、不碰 SwiftUI，所以能直接进单元测试——这是意外收获，动画逻辑通常是最难测的那部分。

渲染层只负责把 `t` 喂进去：

```swift
let phase = clock.phase(at: t, period: Self.walkPeriod)
let tri = phase < 0.5 ? phase * 2 : 2 - phase * 2   // 三角波往返 0→1→0
let facingRight = phase < 0.5
...
.scaleEffect(x: facingRight ? 1 : -1, y: 1)         // 朝向靠镜像
.offset(x: 16 + tri * travel, y: -10)
```

注意 `facingRight` 这一行：**朝向是相位在 0.5 和 0 两处的不连续函数**。相位差之毫厘，镜像翻转是全量的。这个性质待会儿是关键。

## 第一轮误判：暂停的时间轴

第一轮定位到的是 `TimelineView` 的 `paused` 参数。当时写的是 `paused: !animated`——没在播放时暂停时间轴省电。问题是 **`timeline.date` 在 paused 期间是冻结的**，恢复后从冻结点继续；而 `clock.start(at:)` 用的是 `Date()`，真实墙钟。

用户进页面停留了 40 秒再点雨声，两个时间源就差了 40 秒：`origin` 落在 `timeline.date` 的未来 40 秒处，相位算出来是负的，回绕成 0.55 左右——蜗牛朝左、还在中间，然后一路走到相位 1 再翻回来。这解释了「先左移、再掉头」。

改法是 `paused: reduceMotion`，让时间轴在非「减少动态效果」时一直跑，两个时间源对齐。写进了项目的 lessons，收工。

**然后用户回来说：还是调头。**

## 真正的根因：取模把 ε 放大成最大误差

上一轮把误差从 40 秒压到了一帧，但没压到零，而这个 bug 不需要 40 秒——它只需要误差是负的。

`origin = Date()` 取的是用户点击那一刻的墙钟。状态一变，SwiftUI 立刻重算 body，而这次重算拿到的 `timeline.date` 是**上一帧**的时间戳，必然早于 `Date()`。所以第一帧上 `t - origin` 是个小负数，几十毫秒。

然后它撞上了这一行：

```swift
let raw = ((t - origin) / period).truncatingRemainder(dividingBy: 1)
return raw < 0 ? raw + 1 : raw    // ← 这里
```

`+1` 是为了把负相位规范化到 `0..1`，本身没错。但它把 `-0.0005` 变成了 **`0.9995`**——一个数值上距离 0 最远的相位。取模回绕的性质就是这样：**在环上，0 的负邻域就是 1 的正邻域**，ε 的误差被翻译成 1-ε 的坐标。

对位置而言无所谓：`tri = 2 - 2 × 0.9995 ≈ 0.001`，横向差了 0.3 个像素，看不出来。但对朝向而言是灾难：`facingRight = phase < 0.5` 判成 `false`，`scaleEffect(x: -1)` —— **整只蜗牛镜像翻转**。下一帧 `t` 追上 `origin`，相位回到 0.0001，又翻回来。

一次 50ms 的全身镜像，眼睛看得清清楚楚，就是「原地调头一下」。

为什么只有第一次？因为 `origin = t - frozenPhase * period`。冷启动时 `frozenPhase == 0`，`origin` 恰好等于「现在」，是它能取到的最大值，一帧的负误差刚好把相位推过 0 这个边界。第二次点击时 `frozenPhase` 已经是 0.02、0.3……，`origin` 被推到过去几秒甚至几十秒，一帧的偏差穿不过去。

**这个 bug 的存活条件是「起点恰好在环的接缝上」**，而冷启动必然把起点放在接缝上。

## 实测：让 App 自己把两个时钟打出来

上面这套推断是自洽的，但「`timeline.date` 一定滞后于 `Date()`」这句话读文档读不出来——SwiftUI 从没承诺过时间轴给的是哪个时刻，它完全**可以**给你下一帧的呈现时间。真是那样的话 `elapsed` 就永远为正，本文的根因当场作废。所以别推断，直接量。

在 `TimelineView` 的 body 里同时取两个时钟，并把新旧两套公式各算一遍：

```swift
enum FlipProbe {
    static func tick(t: Double, walkOrigin: Double?, period: Double) {
        guard let o = walkOrigin else { return }
        let now = Date().timeIntervalSinceReferenceDate
        let elapsed = t - o
        let old = WalkClock.legacyPhase(elapsed: elapsed, period: period)  // raw < 0 ? raw + 1 : raw
        let new = max(0, elapsed) / period
        print(String(format: "skew=%+.5f elapsed=%+.5f | old=%.5f %@ | new=%.5f %@",
                     t - now, elapsed,
                     old, old < 0.5 ? "→R" : "←L",
                     new, new < 0.5 ? "→R" : "←L"))
    }
}
```

模拟器里跑一次，第 4 秒由代码自动开一次雨声（避免手点带来的时序偶然）：

```
base #01..#15  skew(t-now) = -0.00001 … -0.02717      ← 15 帧全为负
armed  walkOrigin=808935265.34566
#01  skew=-0.01543  elapsed=-0.01517 | old=0.99983 ←L | new=0.00000 →R
#02  skew=-0.02723  elapsed=+0.03483 | old=0.00039 →R | new=0.00039 →R
#03  skew=-0.01057  elapsed=+0.08483 | old=0.00094 →R | new=0.00094 →R
```

三件事一次问清楚：

- **`timeline.date` 恒滞后于墙钟**，常态 0.5ms，偶尔 15–27ms，十五帧无一超前。它给的是刚渲染完那帧的时间，不是未来的呈现时间。
- **状态变更触发的那次 body 求值，`elapsed` 是负的**（-0.015s），和推断一致。
- **旧公式在那一帧算出 `0.99983`，判定朝左**；新公式算出 `0`，朝右。下一次求值（`#02`）就恢复了。

`#01` 到 `#02` 之间墙钟走了 62ms（`0.03483 + 0.02723`），意味着镜像状态在屏幕上活了约四个 60Hz 刷新周期——不是理论上的一瞬，是真的看得见。把旧公式装回去真正驱动画面，录屏逐帧翻出来：

![连续八帧 60fps 录屏：中间四帧蜗牛壳在右、触角在左，整只镜像；前后各两帧正常](https://cdn.tools.cooconsbit.com/uploads/hermes/2026-08-20/1787242645000-df3dff4c.png)

中间四帧就是 66ms，和日志算出的 62ms 对得上。

顺带一个坑，差点让我把结论搞反：`xcrun simctl io recordVideo` 产出的 mov **时间戳不单调**，ffmpeg 的 `trim=start=` 按 pts 选帧会选到完全不同的位置。第一次找这几帧时我按时间裁了一段、发现全是正常帧，差点写成「渲染了但没上屏」。先 `-fps_mode cfr -r 60` 转成恒定帧率再裁，才对得上。**用录屏当证据前，先确认你裁到的真是你以为的那一秒。**

## 修复：一行夹紧

```swift
func phase(at t: Double, period: Double) -> Double {
    guard let origin else { return frozenPhase }
    // t 可能略早于 origin：origin 取自事件发生那一刻的墙钟，而状态一变，
    // SwiftUI 会先用上一帧（更早）的时间轴时间重算一次 body。不夹住的话
    // 负相位会回绕到 ≈1.0，朝向瞬间反向——就是刚点开时蜗牛原地调头那一下。
    let elapsed = max(0, t - origin)
    return (elapsed / period).truncatingRemainder(dividingBy: 1)
}
```

时钟只会向前，负的 elapsed 一定是时间源抖动，不是真实语义，夹到 0 即可。

值得一提的是，同一个文件里另外两个状态机（`AmplitudeRamp.value`、`SceneFade.progress`）**早就有 `p <= 0` 的保护**，因为它们的进度是单调 0→1、天然要处理「还没开始」的情况。只有 `WalkClock` 漏了，恰恰因为它是周期性的——写循环相位的时候，脑子里想的是「回绕」，不是「还没开始」。

## 另一半：不要淡出，只要淡入

第二个毛病顺带说清楚，因为它的解法也是反直觉的。

交叉淡化的自然写法是「旧的淡出 + 新的淡入」，但两层同时处于半透明时，中间会短暂露出底色，观感是一次**洗白**。正确的层序是：

```swift
ZStack {
    // 旧场景整幅垫底，不透明，只管继续动
    if let outgoing = fade.outgoing, !fade.isSettled(at: t) {
        sceneLayer(outgoing, t: t, width: geo.size.width)
    }
    // 新场景按时间算出的 p 叠加淡入，盖住为止
    sceneLayer(fade.current, t: t, width: geo.size.width).opacity(p)
}
```

**只有淡入，没有淡出。** 旧层始终 100% 不透明地垫在底下，直到被完全盖住才移出视图树。任意时刻画面的总不透明度都是 1，不可能露底。

有了这个层序，「关闭太生硬」就不是加一个淡出动画，而是**把关闭也走同一条淡入路径**：停止播放时，「平地」也是一个正常场景，它淡入盖住雨景就行了。原来的代码为了「停止要干脆」特意写了 `groundFade = 0` 硬切，删掉这个特例，开和关立刻对称了。

一个细节：淡出层在过渡期间**必须继续动**。雨还在下、海浪还在涌，只是逐渐被盖住——如果这时候把旧层的时间冻成 0，观感是「雨先停住、再消失」，比硬切还别扭。所以每一层按自己的场景独立决定要不要动：

```swift
let time = (!reduceMotion && (layerScene != .ground || isWalking)) ? t : 0
```

最终效果：

![修复后的场景卡：开与关都是新场景整层淡入盖住旧场景，旧层在底下继续下雨直到被盖住；蜗牛全程保持朝向，不再翻转（iOS 模拟器实录，原速未加速）](https://cdn.tools.cooconsbit.com/uploads/hermes/2026-08-20/1787242018000-f2632233.gif)

## 适用边界

- **只在 TimelineView / CADisplayLink 这类自带时钟的场景里这么写。** 普通视图该用 `withAnimation` 就用，别为了「纯函数」把简单动画复杂化。
- **代价是所有动画状态要自己实现缓动**。这里用的是 smoothstep（`p*p*(3-2*p)`），够用；要 spring 就得自己写弹簧解，不如回去用 SwiftUI 的动画系统。
- **`paused` 省的那点电，未必值得引入第二个时间源。** 真要暂停，就把所有 `Date()` 换成从 `timeline.date` 传下去的 `t`，别混用。

## 带走的教训

**在环形坐标系（相位、角度、时刻、ring buffer 下标）里，取模会把一个微小的负误差翻译成一个最大的正误差。** 线性坐标里 `-0.0005` 和 `0` 差 0.0005；环上它俩差 0.9995。所以凡是「量 + 取模」的组合，都要先问：这个量有没有可能因为时间源抖动、浮点误差、时钟回拨而略微为负？如果有，是夹紧还是回绕，必须是明确的决定，不能是顺手写的那句 `x < 0 ? x + 1 : x`。

推论是：**别让离散的判定直接骑在连续量的不连续点上**。`facingRight = phase < 0.5` 这种写法，把一个全量的视觉翻转绑在了一个数值边界上，边界附近任何抖动都会被放大成 100% 的视觉变化。要么在数值层面保证不抖（本文的做法），要么给判定加滞回，别裸着。

最后一条更通用：**同一个动画里出现两个时间源，就是等着出错**。第一轮我们把两个时间源的差从 40 秒压到 40 毫秒，感觉「基本对齐了」——但这个 bug 的触发条件是符号，不是量级。压小误差不等于修好，只有把误差的**符号**变得不可能，才算修好。
