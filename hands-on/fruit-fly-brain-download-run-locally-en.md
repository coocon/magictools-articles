# How to Download the Fruit Fly Brain and Run It: 139,000 Neurons in 33 Seconds on a Mac mini

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-download-run-locally-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-download-run-locally-en?utm_source=github&utm_medium=referral)**

Most people searching for a "fruit fly brain download" have just seen the headline that scientists uploaded a fly's brain into a computer, and want to try it themselves. Good news: you can, and it's easier than you'd think. Bad news: follow the README's one-line switch to the public data and the official example crashes.

I went through it from scratch on a Mac mini and timed every step. The short version:

- **The "fruit fly brain" is two files**: a list of 138,639 neurons (3.3 MB) and a connection table with 15,091,983 rows (101 MB). Each row says who connects to whom, with how many synapses, and whether the connection excites or inhibits.
- **An ordinary computer is enough**: cloning took 57 s, installing dependencies 21 s, and the official sugar-taste example (30 trials × 1 s of brain time) ran in **33 s**.
- **Switching to the public v783 data as the README says gives a `KeyError`**, because the example's neuron IDs come from an older release. The fix is small; a working version is below.
- Once it runs, you can do experiments that are hard on a real fly: switch off one neuron at a time and watch what the whole brain does.

## Background: what you're actually downloading

In 2024 the FlyWire Consortium published the first complete wiring diagram (connectome) of an adult fruit fly brain in Nature. In the same issue, Shiu et al. published a whole-brain model built on it: every neuron is simplified to a leaky integrate-and-fire (LIF) unit, connection strength is simply the synapse count, and it runs in the Brian2 simulator.

The model's official repository, [philshiu/Drosophila_brain_model](https://github.com/philshiu/Drosophila_brain_model), ships the public data alongside the code, so the most direct way to "download the fruit fly brain" is to clone it:

| File | Size | Contents |
|---|---|---|
| `Completeness_783.csv` | 3.3 MB | FlyWire IDs of 138,639 neurons |
| `Connectivity_783.parquet` | 101 MB | 15,091,983 connections: pre ID, post ID, synapse count, excitatory/inhibitory |
| `model.py` / `utils.py` | 14 KB | The model and helpers for reading results |
| Older v630 data | 90 MB | The release the paper used; the example still points at it |

The whole clone is 380 MB, half of it git history. For richer data (cell types, neurotransmitters, 3D morphology), use FlyWire's Codex (codex.flywire.ai), the official data portal. **The licence is CC BY-NC 4.0: attribution required, no commercial use.**

## What's in the table

Before running anything, I pulled the table apart. A few numbers stand out:

...

---

**[👉 Continue reading: How to Download the Fruit Fly Brain and Run It: 139,000 Neurons in 33 Seconds on a Mac mini](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-download-run-locally-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
