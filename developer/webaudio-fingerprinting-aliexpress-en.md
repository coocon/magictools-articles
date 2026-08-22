---
title: "The Sound of Zero Gain: A WebAudio Fingerprinting Investigation, and an Overstated Conclusion"
slug: webaudio-fingerprinting-aliexpress-en
summary: "Someone noticed their multipoint Bluetooth headphones stopped switching back to their phone whenever an AliExpress page was open. The cause: two hidden AudioContexts with gain set to zero but still wired to the system audio destination, so the browser kept genuinely processing audio and pinned the Bluetooth path open. The investigation is worth learning from. The conclusion that traveled with it — that WebAudio fingerprinting is the next big threat — was shot down by Firefox's fingerprinting protection lead."
category: developer
tags: [WebAudio, browser-fingerprinting, privacy, frontend-debugging, Firefox, anti-tracking]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: webaudio-fingerprinting-aliexpress
---

# The Sound of Zero Gain: A WebAudio Fingerprinting Investigation, and an Overstated Conclusion

> "Given enough victims, your attack code is going to change something that makes someone notice."
> — Tom Ritter, Firefox fingerprinting protection tech lead

---

This started as an ordinary annoyance.

A developer's Bluetooth headphones support multipoint, connected to a PC and a phone simultaneously, with the PC taking priority and the phone playing whenever the PC is silent. The setup worked reliably — until he opened an AliExpress page in Firefox or Chrome.

A few seconds after the homepage loaded, audio from the phone would stop. Closing the tab fixed it immediately.

And: **muting the tab didn't help, muting Firefox didn't help, muting Windows didn't help.** There was no visible video, music, or media element on the page.

Strange enough to investigate. What turned up is considerably more interesting than "another e-commerce site tracks you."

## 1. The investigation — this part is worth copying

The obvious first guess is an autoplaying product video or ad. So he checked the usual suspects:

- `<audio>` and `<video>` elements
- `HTMLMediaElement.play()` calls
- active Media Session metadata
- media requests
- embedded frames containing media

**Nothing.** No media elements, no playback calls, `navigator.mediaSession.playbackState` stayed `none`.

Then a key clue got caught: **the problem didn't start immediately — it appeared after the page sat idle for several seconds.** A delay means this isn't a load-time resource; it's something a running script does deliberately.

So the investigation pivoted from "find the media element" to "instrument the Web Audio API." The technique is simple — wrap the `AudioContext` constructor before the page loads:

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

Plus a wrapper on `AudioNode.prototype.connect()` to see whether anything reached the context's destination.

**That found it immediately: two hidden audio contexts.**

During an idle capture of the homepage, the page created two `AudioContext` objects. Both entered `running` state. Both connected nodes to `AudioContext.destination`. Meanwhile: zero audio or video elements, zero `play()` calls, no active Media Session, no audible sound.

The constructor stack traces pointed at two scripts:

```
https://assets.aliexpress-media.com/g/AWSC/uab/1.140.0/collina.js
https://assets.aliexpress-media.com/g/AWSC/fireyejs/1.231.67/fireyejs.js
```

Both live under an `AWSC` directory and appear to be part of Alibaba's browser security and anti-abuse tooling. The first context came from `collina.js`, the second from `fireyejs.js`.

This "wrap the constructor, wrap connect" pattern generalizes. Any time you suspect a page is quietly using some Web API that DevTools panels don't surface, the same approach works — **instrument the API entry point and dump the stack**. Far faster than reading obfuscated bundles line by line.

## 2. The audio graph, and where the side effect comes from

The scripts are heavily obfuscated, but enough names and operations survive to reconstruct the graph. Both build something like:

```
Sawtooth oscillator
  → AnalyserNode
  → ScriptProcessorNode
  → GainNode (gain set to zero)
  → AudioContext.destination
```

The logic is clear: the oscillator generates a known waveform, it passes through the browser's audio implementation, and the analyser reads back frequency data. The same input waveform produces subtly different results on different browsers, CPUs, and audio stacks — that's the basis of WebAudio fingerprinting.

**Gain is set to zero, so the user hears nothing.**

But here's the crux: **the graph is still connected to `AudioContext.destination`.**

Connecting to the destination forces the browser to actively process the graph — even though the final volume is zero. As far as the system is concerned, this page is performing live audio processing.

That explains the Bluetooth behavior. The system audio path stays occupied, Firefox or Windows keeps the Bluetooth audio route active, and the multipoint headphones can't cleanly hand back to the phone.

It also explains why muting failed: **there's no media element for the tab mute control to stop.** Browser tab muting is built around `HTMLMediaElement`, and there is no media element here. This is a fundamentally different thing from an autoplaying video.

## 3. Then the conclusion got reversed

The post hit around 725 points on Hacker News, and a conclusion grew alongside it in transit: **WebAudio fingerprinting is a stealthier, harder-to-block successor to Canvas fingerprinting, and it's the next thing to defend against.**

That conclusion is wrong, and the person who refuted it carries weight.

Tom Ritter, tech lead for Firefox's fingerprinting protection, published an evaluation on August 20 that opens bluntly:

> Browser fingerprinting is a far-too-pervasive method of tracking users across the web, but **at least for WebAudio specifically, it's not very effective**. Firefox has largely eliminated this fingerprinting vector.

The data is solid. Firefox made WebAudio output approximately constant in **version 118** — three years ago — as part of its first round of fingerprinting protection features. Telemetry shows:

- **99.24% of users** land on one of **three** values
- 0.76% had the collection point fail (value of zero)
- The long tail is 23 other values across 48 users

A signal that sorts users into three buckets is nearly worthless as a fingerprint.

So why three and not one? Ritter's answer is itself interesting: **the differences are CPU-level.** Audio processing is float-heavy math, and instruction set differences leak into results.

- One value from all x86 CPUs, plus x64 CPUs lacking fused multiply-add
- One value from x64 with FMA
- One value from CPUs with the NEON instruction set (ARM)

Firefox is still collapsing them: Bug 2036977 merges x64-with-FMA3 into the x86/x64-without-FMA3 bucket, and Bug 2040494 is filed to collapse the remainder into the NEON bucket — deprioritized behind larger wins like **sanitizing the WebGL renderer and vendor strings**, which is the fatter entropy source today.

Ritter also notes Chrome's WebAudio code was made approximately constant many years ago, and Brave and Safari likely have defenses too — though they probably still leak CPU architecture.

So the accurate statement is: **WebAudio fingerprinting is largely dead on privacy-focused browsers.** It still works against most of the web's users — because most users aren't on those browsers — but it is emphatically not "the next hole to plug." It's a hole that was mostly plugged three years ago.

## 4. So what's the real problem

If the fingerprint itself is weak, what's left worth caring about?

Two things, both more important than the fingerprint.

**First, unobservability.** A web page can keep your audio hardware active, producing real user-perceptible side effects — headphones that won't hand back, an audio chip that never enters a low-power state, higher Bluetooth power draw — while **the browser UI says nothing at all**. No speaker icon on the tab, because that icon only tracks media elements.

Someone on HN asked the obvious question: why does Firefox let a page play audio at volume zero without a tab indicator? That's a legitimate product gap. Others called for browsers to show a status-bar warning when a site is actively circumventing privacy protections.

**Second, the motive may be less sinister — and it doesn't change the conclusion.** A well-argued HN counterpoint: this may not be spyware so much as absurd optimization. Audio devices sleep to save power and take a second or two to wake, so keeping *something* playing — even silence — guarantees product videos start instantly.

A different read from Lobsters: it isn't trying to listen to anything; it's forcing the audio subsystem to genuinely process the generated signal, to squeeze more per-device entropy out of side channels.

Both explanations are plausible, and **both land on the same problem**: whether the motive is optimization or collection, the user pays a real cost and gets neither notice nor choice. Intent doesn't change the side effect.

## 5. Practical steps

### Check whether your own site does this

The third-party SDKs you embed — risk control, anti-bot, ads, analytics — may already do it. Open DevTools and search the Sources panel globally for:

```
AudioContext
OfflineAudioContext
createOscillator
createAnalyser
```

Or more directly, inject this probe before page scripts run (a browser extension or a breakpoint on first statement makes it reliable):

```js
// Detect every AudioContext created on the page and where it came from
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

// And see who connects to the destination
const originalConnect = AudioNode.prototype.connect;
AudioNode.prototype.connect = function (dest, ...rest) {
  if (dest instanceof AudioDestinationNode) {
    console.warn("[audio-probe] node connected to destination", new Error().stack);
  }
  return originalConnect.call(this, dest, ...rest);
};
```

If you ship a product that goes through privacy compliance review, this is worth auditing proactively — you may have inherited third-party fingerprinting without knowing, and the liability is yours.

### If you just want your headphones to work

The original author's fix was blocking the two scripts with uBlock Origin:

```
||assets.aliexpress-media.com/g/AWSC/uab/*/collina.js
||assets.aliexpress-media.com/g/AWSC/fireyejs/*/fireyejs.js
```

**But there's a cost worth stating.** Italian outlet ilsoftware points out these scripts are part of Alibaba's risk-assessment systems, so permanently blocking them isn't consequence-free — the site may respond with extra CAPTCHAs, verification requests, or trouble during login and payment.

A more pragmatic approach is a per-site toggle rather than a global block, or simply a separate browser profile for shopping sites.

### If you build anti-tracking tools

Ritter's evaluation effectively hands you a priority list: don't spend effort on WebAudio, that battle has converged. **WebGL renderer and vendor strings** are the higher-entropy target worth working on now.

---

To close with Ritter's line, which is the best summary of the whole episode. He says he doesn't fully agree with "given enough eyeballs, all bugs are shallow," and offers a truer version:

**Given enough victims, your attack code is going to change something that makes someone notice.**

The person who noticed this time was an ordinary user who just wanted his headphones to switch back to his phone.

---

**Sources**

- [AliExpress webpage keeping multipoint Bluetooth headphones active with WebAudio fingerprinting — laserphile (original finding)](https://blog.laserphile.com/2026/08/aliexpress-webpage-keeping-multipoint.html)
- [webaudio fingerprinting on alibaba — Tom Ritter / ritter.vg (Firefox fingerprinting protection evaluation)](https://ritter.vg/blog-webaudio_alibaba.html)
- [Hacker News discussion](https://news.ycombinator.com/item?id=49372583)
- [Lobsters discussion](https://lobste.rs/s/b0olmy/aliexpress_keeps_multipoint_bluetooth)
- [AliExpress usa WebAudio per il fingerprinting e disturba il Bluetooth? — ilsoftware.it](https://www.ilsoftware.it/aliexpress-audio-script-browser-problemi-cuffie-bluetooth/)
