---
title: "Claude Didn't Write That Printer Driver: 14KB of Glue, a Linux Container, and HP's Own Binary"
slug: claude-code-printer-driver-14kb-glue-en
summary: "A tweet claiming Claude wrote a macOS driver for a Windows-only HP printer pulled 2.62 million views. Pull the repo and you find 6,497 bytes of shell, 7,638 of Python, 550 of Dockerfile — not one line of C. The actual encoding is done by rastertospl, a binary from HP's official Linux driver, running in a Linux container on the Mac. Hacker News caught this. But debunking isn't the point: the genuinely valuable parts of that four-hour session (reading the printer's own error pages, escaping the CUPS sandbox, writing USB directly) and the fact that the decisive turn came from the human, not from Claude, are a precise measurement of where AI's grunt-work ability currently ends."
category: claude
tags: [Claude Code, reverse engineering, printer drivers, CUPS, macOS, AI limits, open source, Hacker News]
coverImage: ""
status: published
locale: en
source: authored
translationSlug: claude-code-printer-driver-14kb-glue
---

# Claude Didn't Write That Printer Driver: 14KB of Glue, a Linux Container, and HP's Own Binary

On August 17, a 19-year-old developer named Kuber Mehta tweeted:

> "just Claude writing a MacOS driver for my obscure HP printer built only for Windows support"

Within two days: **2.62 million views, 14,805 likes.** He had a couple thousand followers at the time — this was a clean organic breakout. Hacker News ran two threads (151 and 105 points), press picked it up, and wccftech escalated the headline to "Wrote A Driver For macOS From Scratch."

I pulled the repo. Here's what's in it:

| Language | Bytes |
|----------|-------|
| Shell | 6,497 |
| Python | 7,638 |
| Dockerfile | 550 |

**About 14KB total. No C, no compiled filter, no kernel code.**

And the README, written by the author himself, says:

> "This repo contains only glue code. It does **not** redistribute HP's driver. `install.sh` downloads the Unified Linux Driver from HP at install time."

The thing that actually encodes page data into something the printer understands is **`rastertospl`, a binary from HP's official Linux driver** — placed inside a Linux container running on the Mac.

So the headline doesn't hold. But stopping there would miss the real story: what happened during those four hours is far more interesting than the false claim, and it draws a remarkably precise line around what AI can and can't do on this kind of dirty work.

---

## 1. Why This Printer Is a Brick on macOS

The HP Laser 1008a belongs to the HP Laser 1000 series. The key background: **this is a rebadged Samsung, from HP's 2017 acquisition of Samsung's printer business.** Its lineage is not LaserJet.

Three consequences:

1. **It speaks SPL/QPDL** (the Samsung-family page description language), not PCL, not PostScript. The README table is blunt about Generic PCL / PostScript: "Printer speaks neither, so the CUPS backend hangs 'offline'."
2. **It's a host-based printer** — there's no interpreter in the box, so rasterization has to finish on the host before anything gets sent.
3. **HP never shipped a macOS driver.** The official Linux path is the Samsung-derived ULD (Unified Linux Driver), last released in 2020.

The open-source world doesn't have a ready answer either. This matters, so I checked each one:

| Project | Supports the 1008a? |
|---------|---------------------|
| **hplip** | No. Official answer on Launchpad: "The HP Laser 100 series is not supported by HPLIP" — this series is Samsung-derived and goes through ULD, not hplip |
| **foo2zjs** | Wrong protocol entirely. The HP LaserJet 1000/1005/1018/1020 models it supports are the older ZjStream family — a different thing from "HP Laser" (no Jet) 1008a |
| **splix** | Nominally supported, fails in practice. splix 2.0.2 added "HP Laser 10x" support via PR #9, but **that PR states outright that "HP are not tested"** — and it targets the HP Laser 103–108 line, while the 1008a is a different product line with SPL3 frame format differences |

So this isn't a "just search GitHub" problem. This printer — sold on Amazon, as HN user 1970-01-01 pointedly noted about the word "obscure" ("Obscure? Sir, the printer is sold on Amazon.") — really is a brick on a Mac.

## 2. What Actually Happened in Those Four Hours

The author published the full Claude Code session transcript (209KB of Markdown), which is the most valuable artifact here — it makes the whole process auditable.

It starts with a maximally casual prompt:

> "hey can you set up the drivers for HP Laser 1008a on this mac please, it's connected"

What follows is a long chain of failures:

1. **Generic PCL PPD** → queue hangs at "offline-report connecting-to-device"
2. Searching confirms it's a Samsung-lineage SPL device; trying **splix-macos (ML-2010 PPD)** → **the printer's firmware locks up** from the PCL garbage already pushed into it, `STATUS:BUSY`, requiring the user to physically power-cycle it
3. Reading the USB descriptors reveals `bInterfaceProtocol = 4` on the interface — **IPP-over-USB** — which explains why macOS's stock `usb` backend keeps misreporting offline
4. Workaround: use pyusb/libusb to **write the USB bulk endpoint directly.** First it hits the macOS kernel driver holding the interface (EACCES), establishing that **only root can detach it**
5. First direct QPDL write: **paper comes out, but it's garbage — and it prints 4–5 sheets back to back.** The user's own words: "it printed... something LOL back to back 4-5 times", with a photo
6. Raw PWG/URF writes → silently swallowed by the printer. The IPP-over-USB channel → HTTP connects but `/ipp/print` returns 404
7. `foo2qpdl -z0` → the printer prints its own error page: **"SPL ERROR - Please use the proper driver"**. A 1200x600 attempt → another one: **"SPL ERROR Illegal Resolution ... ERROR CODE: 11-1113"**

At step 7 there's a line from Claude that I think is the sharpest technical observation in the whole transcript:

> "the printer's error page is basically telling us how to fix it each time."

**The printer is using printed paper to tell you what's wrong, every time.** Treating vendor firmware's error reports as a debugging channel is old-fashioned, and exactly right.

## 3. The Turning Point Didn't Come From Claude

Even at that stage the direction was wrong: still trying to assemble an SPL3 encoder from open-source pieces that the printer would accept. splix's 600dpi A4 band-width table was correctly configured (608 bytes) and the output was still "striped raster at the page origin, repeated sheets."

**The decisive step came from the user.** He suggested: stop reverse-engineering.

Claude's response, verbatim:

> "Oh, that's a much better instinct than reimplementing — **we don't need to reverse-engineer or clean-room anything. The actual SPL3 codec is freely downloadable.**"

Download the ULD tarball from ftp.hp.com, open it, and there it is: an **aarch64 build of `rastertospl`** that runs natively in a Linux container on Apple Silicon.

That's how the first clean test page got printed.

I want to isolate this section, because it's the most informative part of the story:

**On *how* to try things, Claude was strong.** It read USB descriptors, recognized the IPP-over-USB protocol number, diagnosed the root cause of the backend misdetection, arrived at direct libusb writes, worked out that only root can detach the kernel driver, and used the printer's error pages as an iteration signal. All of that takes real skill, and it got them right in sequence.

**On *whether* to try things, it went seven steps down the wrong road.** Because the opening prompt said "set up the drivers," it marched toward "build a working encoder" and never once stopped to ask whether that encoder needed building at all, given it was sitting on HP's website.

This failure mode is extremely typical. **Models are excellent at exhaustive search inside a direction you've set, and poor at questioning the direction itself.** What was "stop reverse-engineering" worth? It cancelled every preceding effort and was the only decision in the project that actually solved the problem.

Matthew Garrett (mjg59, a well-known kernel and firmware engineer) gave what I think is the most accurate formulation, on HN:

> "My experience is that they're better than me at a lot of the process... so having some skills that are pretty much 'This smells wrong' helps a lot"

Better than you at the process. **The "this smells wrong" instinct is still yours to supply.**

## 4. What Shipped, and What It Routes Around

The final architecture:

```
Any app hits Cmd-P
  → CUPS queue (using HP's official PPD)
  → CUPS raster
  → CUPS's stock socket backend (socket://127.0.0.1:9108)
  → root LaunchDaemon (Python script, outside the sandbox)
  → HP rastertospl (in a colima Linux container, emitting real SPL3)
  → libusb writes the printer's USB bulk endpoint directly
```

There is real engineering in here. macOS's CUPS enforces a mandatory sandbox that **forbids filters and backends from invoking containers or touching USB directly.** The way around it: don't write a filter. Use CUPS's built-in socket backend to send data to a localhost port, and put a root daemon on the other end of that port — outside the sandbox — that calls the container to transcode and then writes USB via libusb.

That amounts to **building a backend replacement**, simultaneously routing around macOS's `usb` backend offline misdetection and the CUPS sandbox. It's the most engineering-shaped part of the project.

The costs are real too:

- **A colima Linux VM has to stay resident** (after a reboot, the first print can wait up to a minute for the VM)
- **A root LaunchDaemon executes code from `~/.hp1008` in the user's home directory** — HN user Tiberium called this out specifically: "It also requires a root launcher that runs code from the user ~/.hp1008 dir, so security is weakened."
- A different USB PID means hand-editing `direct_write.py`
- Verified only on macOS 26 / Apple Silicon

Results: 600dpi, A4, a clean single-page text test print, roughly one second to transcode and write. The author supplied photos. **Note that all physical evidence comes from the author alone — I found no third-party reproduction.**

## 5. How HN Took It Apart, and How the Author Responded

Across both threads, the criticism converged:

> **ssdspoimdsjvv**: "If I understand correctly, it just wraps an existing Linux driver in a container. You can hardly call that writing a driver."

> **Tiberium** (who posted in both threads): "Unfortunately this is a very misleading post... Claude didn't write a driver. It basically used HP's existing proprietary driver in a Linux VM on macOS, and just bridged that to macOS."

> **asveikau**: "It didn't write a driver. It used a linux driver. You don't need an LLM."

> **3129476** dug up prior art: "Claude did not write any macOS driver. It uses the HP Linux driver inside docker. Here is prior art from 2017..." — someone wrote a "run Linux print drivers in a Docker container" tutorial back in 2017. HN user oneplane pointed to an existing product doing the same thing (printervention.app, running a Linux VM in the browser over WebUSB).

There were moderates too. mariuolo's line is probably the fairest: "But it's not completely native, it's a wrapper around the original linux driver... **Still better than nothing.**" happyPersonR actually preferred this design: "Makes it so if there's an issue I don't get a kernel panic."

**The author's response deserves credit.** He didn't dig in:

- After posting, he tested splix 2.0.2 and updated the README, with a commit message reading "Correct SpliX claim + apply review fixes"
- He walked back the wording from "writing the driver" (HN user feintruled noticed: "I note the claim on this page is walked back from the original 'writing the driver'")
- He put a request for help in the README: "If you can pin down the exact byte-level difference between HP's rastertospl output and SpliX's for this printer... SpliX could likely be patched and colima dropped entirely."
- The repo's only issue (#1, "SpliX support") was opened by ValdikSS — the author of splix PR #9

A 19-year-old whose tweet hit 2.62 million views, who then walked back the claim, ran the missing experiments, and open-sourced the full session for anyone to audit. That's a better story than the tweet was.

His own methodology summary on HN is refreshingly unglamorous: "there's not really a hack for it, you just work with it, convey thoughts decently and try out things with educated guesses until it sticks."

## 6. What I Take From This

**First, run a layer-check on any "AI wrote an X" headline.** Three steps: pull the repo and look at the language breakdown and byte counts (the most honest signal there is), find the dependency declarations in the README, and search whether the field already has a vendor or open-source implementation. All three pointed the same way here, and it took five minutes. The author's own README opens with "only glue code" — **the exaggeration lives in the tweet and the press coverage, not in the repository.**

**Second, what AI saved here was the cost of *trying*, not the cost of *thinking*.** Everything Claude got right over those four hours has a common shape: high-intensity trial and information retrieval inside a set direction — reading descriptors, recognizing protocol numbers, writing direct libusb, iterating on error pages, designing around the sandbox. Humans do all of that too; it just takes days instead of hours. And the one thing it got wrong — "this wheel doesn't need inventing" — was the single cheapest thought available. **Directional skepticism is still a human job.**

**Third, don't rush to install this.** It requires a resident Linux VM and a root daemon executing scripts from your home directory, has been verified on exactly one hardware and OS combination, and has no third-party reproduction. It's an excellent technical record, not a product you'd recommend to someone. The author is himself asking for help eliminating colima.

**Fourth, the genuinely portable lesson is "the printer's error page is telling you how to fix it."** Hardware and vendor firmware emit far more diagnostic information than people expect — error pages, status codes, descriptors, logs. Wiring those into the AI's feedback loop beats asking it to "reverse-engineer" from nothing. The success stories in that HN thread (someone reverse-engineering golf cart electronics with Claude + ILSpy + Wireshark, someone adapting an Xbox controller in five hours, someone writing an embedded Rust epaper driver) all share the pattern: **give it a closed loop where it can repeatedly read real feedback, not a task where it has to guess.**

The tweet's headline was wrong. But the 209KB transcript underneath it is the best material I've read lately on which parts of the dirty work AI can actually take off your hands — precisely because it left the seven wrong turns and the human's course correction in the record, unedited.

---

**References**

- [Original tweet, @kuberwastaken (2.62M views)](https://x.com/kuberwastaken/status/2089377982536388964)
- [Full Claude Code session transcript (209KB)](https://cdn.kuber.studio/chat/hp-laser-1008a-driver)
- [GitHub: Kuberwastaken/hp-laser-1008a-macos](https://github.com/Kuberwastaken/hp-laser-1008a-macos)
- [HN thread one (151 points)](https://news.ycombinator.com/item?id=49344643)
- [HN thread two (105 points)](https://news.ycombinator.com/item?id=49352806)
- [splix PR #9 (states "HP are not tested")](https://github.com/OpenPrinting/splix/pull/9)
- [hplip official answer: HP Laser 100 series unsupported](https://answers.launchpad.net/hplip/+question/694005)
- [HP Laser 1000 series official driver page](https://support.hp.com/in-en/drivers/hp-laser-1000-printer-series/model/2101513671)
- [Pierov: HP Laser 107a on Linux (prior reference)](https://www.pierov.org/2023/07/25/hp-laser-107a-linux/)
- Related on this site: [The Complete Guide to Claude Code](/articles/claude-code-guide-en) · [The truth about "Codex made it 232x faster"](/articles/codex-autoresearch-gpu-kernel-232x) · [Four months inside a "lights-off" software factory](/articles/lights-off-software-factory-postmortem-en)
