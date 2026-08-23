# Your AI Video's Subtitles Drift More the Longer It Runs: Stop Fixing the Aligner — the Bug Is 'Generate the Whole Audio First'

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/tts-subtitle-drift-sentence-level-fix-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/tts-subtitle-drift-sentence-level-fix-en?utm_source=github&utm_medium=referral)**

## Symptom

I was automating explainer videos for a tinnitus-education project: script → TTS narration → stock-footage edit → burned-in subtitles, on a MoneyPrinterTurbo (MPT) pipeline with MiniMax speech-02-turbo for TTS.

In the first render, subtitles matched the voice perfectly for about 30 seconds, lagged visibly after a minute, and were seconds off by the end. The generated `.srt` was worse than it looked on screen: **a run of entries at the tail all had `00:00:00` timestamps** — alignment hadn't just slipped, it had snapped mid-file.

## What the Original Pipeline Does

The default approach in MPT-style tools is a three-step relay:

1. Send the **entire script in one shot** to TTS, producing one multi-minute audio file;
2. Run whisper over that audio to **transcribe it back into text** with word-level timestamps;
3. **Fuzzy-match** the transcription against the original script (MPT's `correct()`), assigning each script sentence a time range, and emit the srt.

The logic seems sound: TTS doesn't return timestamps, so use speech recognition to "recover" them.

## Why It Snapped

The script said "4000 Hz"; whisper transcribed the Chinese narration as the spelled-out form ("four thousand hertz"). The script had em dashes and quote marks; the transcription didn't. Filler words and pause punctuation disagreed too. Fuzzy matching depends on two strings looking alike, and **systematic differences in numerals and punctuation crater the similarity score**. Once the matcher fails to find a good-enough candidate for one sentence, it loses its anchor — and everything after that point unravels, falling back to the `00:00:00` default.

The natural first move is to patch the matcher: normalize numerals to one form, strip punctuation before comparing, loosen the threshold. That version stopped snapping. The slow drift remained — because drift and snapping were never the same problem.

...

---

**[👉 Continue reading: Your AI Video's Subtitles Drift More the Longer It Runs: Stop Fixing the Aligner — the Bug Is 'Generate the Whole Audio First'](https://tools.cooconsbit.com/en/articles/tts-subtitle-drift-sentence-level-fix-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
