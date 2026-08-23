# 蜗牛在你点开的瞬间原地调头：一帧的时钟误差，被取模回绕放大成了最大误差

> 📍 本文首发于 [MagicTools 码农早餐](https://tools.cooconsbit.com/zh/articles/swiftui-timelineview-phase-wraparound-flip?utm_source=github&utm_medium=referral)。镜像仓库仅收录预览，**[点此阅读全文 →](https://tools.cooconsbit.com/zh/articles/swiftui-timelineview-phase-wraparound-flip?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 继续阅读全文：蜗牛在你点开的瞬间原地调头：一帧的时钟误差，被取模回绕放大成了最大误差](https://tools.cooconsbit.com/zh/articles/swiftui-timelineview-phase-wraparound-flip?utm_source=github&utm_medium=referral)**

更多文章：[tools.cooconsbit.com/articles](https://tools.cooconsbit.com/zh/articles?utm_source=github&utm_medium=referral)
