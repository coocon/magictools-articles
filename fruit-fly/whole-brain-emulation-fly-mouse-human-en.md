# Fly, Mouse, Human: How Far Is Whole Brain Emulation? The Math, Starting from a Mac mini Benchmark

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/whole-brain-emulation-fly-mouse-human-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/whole-brain-emulation-fly-mouse-human-en?utm_source=github&utm_medium=referral)**

After "scientists uploaded a fruit fly's brain", the next question is almost always: **what about a human brain?**

This article answers it concretely. It takes a whole-brain fruit fly simulation we actually ran on a Mac mini as the baseline, scales the same model and algorithm up to a mouse and a human, and looks at which walls you hit. The short version:

- **Neuron counts differ by five to six orders of magnitude.** A fly brain has 139,255, a mouse 70.89 million (about 500×), a human 86.1 billion (about 620,000×).
- **Wall one is mapping.** The largest nervous system ever mapped completely is the fly's; for mammals, only one cubic millimetre of mouse cortex has been mapped — about 1/400 of a mouse brain.
- **Wall two is proofreading.** Proofreading the whole fly brain took about 33 person-years. Scaled linearly, a mouse would take about 17,000.
- **Wall three is compute and memory.** One second of fly brain takes 12.6 s on a Mac mini; extrapolated, a mouse takes about 41 hours and 1.4 TB of memory, a human about 5.7 years and 1.7 PB.
- **Wall four is the most fundamental: the wiring diagram isn't enough.** In our tests, the existing whole-fly-brain model fires no spikes at all without input; it has no learning, no memory and no neuromodulation.

## How big are these brains?

| Species | Neurons | Complete wiring diagram | Source |
|---|---|---|---|
| *C. elegans* (worm) | ~300 | Mapped 1986 | Dorkenwald 2024, citing White 1986 |
| Fruit fly larva | ~3,000 | Mapped 2023 | Same, citing Winding 2023 |
| Adult fruit fly (brain) | 139,255 | 2024, FlyWire | Dorkenwald et al. 2024, Nature |
| Adult fruit fly (brain + nerve cord) | 166,700 | 2026, MaleCNS | Berg et al. 2026, Cell |
| Mouse | 70.89 million | Only 1 mm³ of cortex | Herculano-Houzel et al. 2006, PNAS |
| Human | 86.1 billion | Not begun | Azevedo et al. 2009 |

"The human brain has 100 billion neurons" is a widely repeated figure. In 2009 Azevedo et al. actually counted, using the isotropic fractionator (turning brain tissue into a uniform suspension of nuclei and counting them), and got 86.1 ± 8.1 billion.

## Wall one: mapping

The first step in building a wiring diagram is imaging the whole block of tissue with an electron microscope at nanometre resolution. Here are the volumes imaged so far:

| Dataset | Imaged volume | Scale |
|---|---|---|
| Fly brain (FlyWire neuropil) | 0.0175 mm³ | 139,000 neurons |
| Fly whole CNS (MaleCNS) | 0.082 mm³ | 167,000 neurons; 7 microscopes for 13 months |
| Mouse visual cortex (MICrONS) | 1 mm³ | 200,000+ cells, ~0.5 billion synapses |
| Whole mouse brain | ~400 mm³ (brain mass 0.42 g) | 70.89 million neurons |
| Human brain | ~1.2 million mm³ | 86.1 billion neurons |

...

---

**[👉 Continue reading: Fly, Mouse, Human: How Far Is Whole Brain Emulation? The Math, Starting from a Mac mini Benchmark](https://tools.cooconsbit.com/en/articles/whole-brain-emulation-fly-mouse-human-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
