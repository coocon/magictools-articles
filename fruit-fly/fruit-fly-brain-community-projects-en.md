# What People Built with the Fruit Fly Brain: Mario, Minecraft, DOOM and More — 9 Projects Taken Apart, 2 Retested

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-community-projects-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-community-projects-en?utm_source=github&utm_medium=referral)**

After Eon Systems announced in March 2026 that it had "uploaded a fruit fly", the fly brain became a new toy for developers. In September alone, GitHub saw a string of projects: a fly brain playing DOOM, Mario and Flappy Bird, running a Minecraft NPC, living on a HarmonyOS phone and on a Mac desktop — and one pitched for stock trading signals.

The headlines escalate quickly. I read all nine READMEs and the key code, and asked three questions of each:

1. **How many neurons?** The whole brain (139k or 166k), a small piece extracted from it, or a few dozen built by hand?
2. **Who actually decides the actions?** The connectome itself, a trained readout layer, a state machine, or hand-tuned weights?
3. **Are there controls?** If you cut a pathway, does the behavior change?

Then I retested the two that run on a Mac. The short version:

- **The most honest projects in this batch are the most popular ones.** DOOMFLY (417 stars) states in its opening paragraph that learned survival has not been demonstrated; DesktopFly (1,063 stars) has a whole "What's real" section.
- **Loading the whole brain is not the same as the whole brain making decisions.** Several projects load all 100k+ neurons, but what actually picks the action is a trained readout layer or a state machine.
- **A 21-neuron circuit with hand-tuned weights can clear a Mario level — but it's fragile.** Scale every weight to 0.8× and the fly won't take a step; scale to 1.3× and it dies halfway.

## The nine projects

Star counts as of 2026-10-03.

| Project | What it does | Data | Neurons | What decides the action |
|---|---|---|---|---|
| [DOOMFLY](https://github.com/nftechie/doomfly) (417★) | Fly brain plays DOOM | MaleCNS v1.0 | All 166,700 | Fixed neuron-to-button mapping, plus experimental dopamine plasticity |
| [DesktopFly](https://github.com/DenisSergeevitch/desktop-fly) (1,063★) | 3D fly pet on the macOS desktop | FlyWire + MaleCNS | Extracted 668 + 1,045 | Subcircuit LIF plus modelled body and states |
| [fly-flappy](https://github.com/ns2250225/fly-flappy) (28★) | Fly brain plays Flappy Bird | MaleCNS v1.0 | All 166,700 | Trained logistic-regression readout plus safety rules |
| [FlyCraft](https://github.com/Yi-111-a/FlyCraft) | Combat NPC in Minecraft | MaleCNS | Subgraph of ~8,000 nodes | Subgraph dynamics → fight / flee / wander |
| [CyberFly for HarmonyOS](https://github.com/zhuhaozoo/FlyBrain-HarmonyOS) | 3D fly ecosystem on a HarmonyOS phone | MaleCNS v1.0 | All 165,122 | State machine first, connectome modulates six channels |
| [Fly Mario](https://github.com/FuChen1649/fly-mario) | Fly circuit auto-clears a Mario-style level | Only cell names and IDs borrowed | **21** | Hand-tuned LIF microcircuit |
| [@chnak/fly](https://github.com/chnak/fly) | TypeScript training library; README pitches trading signals | MaleCNS | Not clearly stated | Logistic-regression readout |
| ["A Day in the Life of a Fly"](https://github.com/SlimeBoyOwO/LingChat/pull/825) (LingChat PR) | 3D fly life mini-game in Rust | FlyWire v783 | All 139,255 | LIF plus reward/punishment plasticity and a sensorimotor mapping |
| [fly-brain (Rojas)](https://github.com/erojasoficial-byte/fly-brain) (65★) | Two identical connectomes "grow individuality" | FlyWire v783 | All 138,639 | LIF plus Hebbian plasticity in a NeuroMechFly body |

...

---

**[👉 Continue reading: What People Built with the Fruit Fly Brain: Mario, Minecraft, DOOM and More — 9 Projects Taken Apart, 2 Retested](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-community-projects-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
