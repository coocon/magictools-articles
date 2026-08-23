# The Sound of Zero Gain: A WebAudio Fingerprinting Investigation, and an Overstated Conclusion

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/webaudio-fingerprinting-aliexpress-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/webaudio-fingerprinting-aliexpress-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: The Sound of Zero Gain: A WebAudio Fingerprinting Investigation, and an Overstated Conclusion](https://tools.cooconsbit.com/en/articles/webaudio-fingerprinting-aliexpress-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
