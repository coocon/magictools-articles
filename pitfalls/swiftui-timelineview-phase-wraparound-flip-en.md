# The Snail Spins Around the Instant You Tap: One Frame of Clock Skew, Amplified by a Modulo Into the Worst Possible Error

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/swiftui-timelineview-phase-wraparound-flip-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/swiftui-timelineview-phase-wraparound-flip-en?utm_source=github&utm_medium=referral)**

## The Symptoms

The sounds tab of a tinnitus-relief app has a permanent scene card at the top: Coco the snail stands on flat ground; tap the rain sound and the backdrop becomes a rainy scene while Coco crawls back and forth extremely slowly (90 seconds per round trip); stop playback and the ground comes back. The whole card lives inside `TimelineView(.animation(minimumInterval: 1/20))`, so its body is recomputed 20 times a second.

Two defects, in the user's own words:

1. The **first** time you tap a nature sound, the snail turns around in place. It never happens on the second or third tap.
2. Opening fades in, but closing is abrupt — the background hard-cuts.

The second one is a design problem. The first one is a real bug, and it took two rounds to bottom out.

## Why SwiftUI's Built-in Transitions Don't Work Here

Some context first, or the code below looks like reinventing a wheel.

The card originally used `.transition(.opacity)` with `withAnimation` for backdrop changes. In practice the fade **fired intermittently, and essentially never on the first switch**. SwiftUI transitions depend on the state change happening inside an animated transaction, and `TimelineView` recomputes the body every 50ms — that recomputation swallows the transaction, overwriting the pending transition with the next frame's non-animated one before it ever starts.

The conclusion: **inside a TimelineView, do not rely on SwiftUI's implicit or explicit animation**. The timeline hands you `t` every frame anyway, so compute the progress from `t` yourself.

## The Skeleton: Animation State as a Pure Function of Time

So every moving part on the card became a small value-semantics state machine that exposes nothing but `f(t) -> value`:

```swift
/// Crawl clock: phase survives walk-stop-walk (stop where you are, resume in place)
struct WalkClock: Equatable {
    private(set) var frozenPhase: Double = 0   // phase while stopped (0..1, one triangle round trip)
    private(set) var origin: Double?           // phase origin while moving (nil = stopped)

    func phase(at t: Double, period: Double) -> Double { ... }
    mutating func start(at t: Double, period: Double) {
        guard origin == nil else { return }
        origin = t - frozenPhase * period      // resume from the frozen phase
    }
}
```

Two more follow the same shape: `AmplitudeRamp` (smoothstep ramp for the wiggle amplitude) and `SceneFade` (crossfade progress). All of them are structs, side-effect free, and touch no SwiftUI, so they drop straight into unit tests — an accidental win, since animation logic is normally the least testable part of a view.

...

---

**[👉 Continue reading: The Snail Spins Around the Instant You Tap: One Frame of Clock Skew, Amplified by a Modulo Into the Worst Possible Error](https://tools.cooconsbit.com/en/articles/swiftui-timelineview-phase-wraparound-flip-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
