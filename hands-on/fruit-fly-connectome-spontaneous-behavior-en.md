# I Ran a 139,000-Neuron Fruit Fly Brain on a Mac mini. With No Input at All, Does It Move on Its Own?

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/fruit-fly-connectome-spontaneous-behavior-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/fruit-fly-connectome-spontaneous-behavior-en?utm_source=github&utm_medium=referral)**

In March 2026, Eon Systems published "uploading a fruit fly": a spiking network built from the FlyWire whole-brain connectome, wired to a physics-simulated fly body that forages, grooms and escapes in a virtual world. The demo is striking, but the key layer — how neuron activity becomes movement — was not open-sourced.

I wanted to answer a plainer question: **once this fly brain is replicated, what behavior does it generate by itself?** So I assembled the open-source parts they used on a single Mac mini, tested it once with sensory input and once with none.

The short version:

- **With input, the connectome alone turns sensation into usable motor signals.** Sugar on the legs: the fly turns and starts feeding 0.66 s in. A black ball looming at the compound eyes: the giant fiber fires at 1.89 s and the body backs away.
- **With no input, the original model is completely silent.** It is a passive responder; it does not move on its own.
- After adding two pieces of basic physiology — membrane noise and spike-frequency adaptation — it does switch **spontaneously** between resting, walking forward, walking backward, grooming and startle. But version 1 keeps time like a metronome. Version 2 changes three things, and only then does the rhythm turn irregular, with bouts of varied length, closer to a living animal.
- Every one of these "behaviors" passes through a low-dimensional interface I wrote by hand (descending neurons → actions). **This is not a motor-neuron-level simulation.** Keep that in mind when reading the numbers below.

## Background: what exactly got "uploaded"

The stack I used has four parts. The first two are open-source projects Eon's write-up relies on, flybody is a flight body I added, and the last one Eon did not release:

| Component | Role | How I used it |
|---|---|---|
| FlyWire v783 + Shiu 2024 LIF model (Eon fly-brain) | Whole-brain spiking network: 138,639 neurons, ~5 million synapses | Unmodified, Brian2 C++ backend |
| NeuroMechFly v2 (flygym) | Fly body in MuJoCo + CPG gait | Walking, turning, backing up |
| flybody (Janelia / DeepMind) | A second body, flight-oriented | Flight footage; skipped in this article |
| Descending neurons → motor commands | Translate brain output into leg movement | **Not released — I wrote my own** |

FlyWire covers the brain only, not the ventral nerve cord (the fly's spinal cord), so there is a built-in gap between brain and legs. The last signal the brain produces is the firing of descending neurons (DNs). Everyone replicating this has to fill in the path from DNs to joint torques themselves.

...

---

**[👉 Continue reading: I Ran a 139,000-Neuron Fruit Fly Brain on a Mac mini. With No Input at All, Does It Move on Its Own?](https://tools.cooconsbit.com/en/articles/fruit-fly-connectome-spontaneous-behavior-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
