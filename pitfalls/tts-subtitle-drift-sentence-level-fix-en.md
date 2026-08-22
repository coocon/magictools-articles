---
title: "Your AI Video's Subtitles Drift More the Longer It Runs: Stop Fixing the Aligner — the Bug Is 'Generate the Whole Audio First'"
slug: tts-subtitle-drift-sentence-level-fix-en
category: pitfalls
locale: en
tags: [TTS, subtitle alignment, AI video, whisper, bug postmortem]
summary: "Building explainer videos with TTS + auto subtitles: the first 30 seconds were perfectly synced, then the subtitles drifted further and further behind, and the tail of the .srt degraded to 00:00:00 timestamps. The first instinct — patch the alignment (normalize numbers, loosen similarity thresholds) — treats symptoms. This postmortem shows the real cause: any 'whole-clip TTS + whisper transcription + post-hoc matching' pipeline drifts by construction, because it hands timing — which the generation step could produce directly — to a lossy reconstruction chain. The fix: sentence-level TTS with sample-accurate concatenation, where each subtitle's timestamp is the running sum of real wav sample counts. Sentence boundaries reset the error, so drift is physically impossible. Includes splitting rules, caching design, and the limits of the approach."
status: published
source: authored
translationSlug: tts-subtitle-drift-sentence-level-fix
---

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

## The Real Root Cause: Timing Travels a Lossy Path

Step back and look at the architecture. The script is the single source of truth; the audio is generated from it, and so are the subtitles. But **the subtitle timestamps aren't taken from the generation step — they're reconstructed via audio → transcription → matching.** Every link in that chain has error:

- whisper's word-level timestamps jitter by tens to hundreds of milliseconds;
- the transcription and the script are two different representations, reconciled probabilistically;
- nothing ever resets the error — it **accumulates monotonically** along the timeline.

"Worse the further you go" isn't a bug; it's the guaranteed behavior of this architecture. Fixing the matching algorithm is patching a pipe that leaks by design.

## The Fix: Make Timestamps a Byproduct of Generation

Switch to **sentence-level TTS with sample-accurate concatenation**:

1. **Split** the script into subtitle-sized sentences. For Chinese: split on `。！？；` and em dashes, and re-split sentences over 24 characters at a comma within the 8–24 character window (so no subtitle line overflows the screen). For English: split on sentence-ending punctuation, and re-split sentences over 14 words at a comma.
2. **TTS each sentence separately**, producing one small wav per sentence.
3. **Concatenate at the sample level**, inserting a fixed 200 ms of silence between sentences.
4. **Timestamp = running sample count.** Sentence N's start time is the sum of the real sample counts of sentences 1..N-1 (plus gaps), divided by the sample rate.

The key property: **subtitle line = TTS unit = calibration unit**. Each sentence's duration comes from its own wav's actual sample count — not estimated, not recognized, not matched. Even if one sentence's duration surprises you, the error is confined to that sentence: the next one starts from real data again. **Sentence boundaries are natural calibration points, so error physically cannot accumulate across them.** The entire transcribe-and-match chain is simply deleted.

A free engineering bonus: cache each sentence's TTS result on disk, keyed by a hash of its text. Resuming an interrupted run costs nothing, and editing one sentence in the script re-generates only that sentence — everything else hits the cache.

## Where This Applies (and Where It Doesn't)

- **Prosody**: per-sentence synthesis loses some cross-sentence intonation flow, and the inter-sentence pause is a fixed 200 ms rather than a natural one. Entirely acceptable for narration; for dialogue or dramatic reading, group sentences into paragraphs and tune the gap.
- **Request count**: one video goes from 1 TTS call to a few dozen. The hash cache makes retries nearly free, but if your TTS bills per call with no caching, do the math.
- **Not applicable**: if you didn't generate the audio — subtitling a human recording, say — recognition-based alignment is your only option. This article's premise is that **the audio and the subtitles share a source**.

## The Portable Lesson

When two artifacts must stay strictly in sync, don't generate them independently and reconcile afterward — **make the synchronization information a byproduct of generation**. "Build it all, then line things up" has no error-reset points and must drift at scale. "Carry the timing out of the generator" has no alignment step left to fail.

This isn't just about subtitles. Code and docs, schemas and type definitions, configs and deploy manifests — wherever multiple artifacts derive from one source of truth, prefer derivation that carries consistency with it, over producing outputs and then chasing them with diffs, matchers, and audits.
