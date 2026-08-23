# Claude Didn't Write That Printer Driver: 14KB of Glue, a Linux Container, and HP's Own Binary

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/claude-code-printer-driver-14kb-glue-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/claude-code-printer-driver-14kb-glue-en?utm_source=github&utm_medium=referral)**

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

...

---

**[👉 Continue reading: Claude Didn't Write That Printer Driver: 14KB of Glue, a Linux Container, and HP's Own Binary](https://tools.cooconsbit.com/en/articles/claude-code-printer-driver-14kb-glue-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
