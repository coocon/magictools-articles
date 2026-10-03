# What Is MaleCNS? The First Whole-CNS Wiring Diagram of a Male Fruit Fly, and How It Differs from FlyWire

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/malecns-male-fly-connectome-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/malecns-male-fly-connectome-en?utm_source=github&utm_medium=referral)**

If you've looked at the recent wave of "fruit fly brain plays a game" projects, you'll notice that nearly everything built after September names the same dataset: **MaleCNS**.

It stands for Male Central Nervous System: the wiring diagram of a **male** fruit fly's **entire central nervous system**. The short version:

- **It's the first fly wiring diagram with the brain and the ventral nerve cord together.** The ventral nerve cord is the fly's spinal cord, home of the leg and wing motor neurons. FlyWire covers only the brain.
- **Scale**: 166,700 neurons and 11,710 neuron types, all proofread and annotated.
- **Timeline**: v0.9 in October 2025, v1.0 on June 8, 2026, and the paper in Cell in September 2026.
- **Licence: CC BY 4.0, commercial use allowed with attribution.** FlyWire is CC BY-NC 4.0, no commercial use.
- **It's an entirely separate effort from FlyWire**: a fly of the other sex, a different microscope, a different AI segmentation.
- **Male and female fly brains are mostly the same.** The paper finds sex differences in only a small set of cell types, concentrated in higher brain centres.

## MaleCNS vs. FlyWire

MaleCNS is a collaboration between Janelia's FlyEM team, the University of Cambridge's Department of Zoology, the MRC Laboratory of Molecular Biology and Google Research. The numbers below come from its paper (Cell 2026 and the bioRxiv preprint) and the two original FlyWire papers:

| | FlyWire (FAFB) | MaleCNS |
|---|---|---|
| Fly | One female | One male |
| Coverage | Brain (incl. optic lobes) | Brain + optic lobes + **ventral nerve cord** |
| Microscope | Serial-section transmission EM | Enhanced focused ion beam SEM (eFIB-SEM) |
| Cutting | 7,062 sections, 35–40 nm each | "Hot-knife" cut into 20 µm slabs, then milled and imaged layer by layer |
| Voxel | 4 × 4 × 40 nm | 8 × 8 × 8 nm, equally fine in all three directions |
| Imaging | 2 custom microscopes, ~16 months | 7 microscopes in parallel, 13 months, 160 teravoxels |
| Segmentation | Princeton's boundary-detecting convolutional nets | Google's flood-filling networks (FFN) |
| Neurons | 139,255 | 166,700 |
| Cell types | 8,453 | 11,710 |
| Proofreading | ~33 person-years: research community, professionals, citizen scientists | ~44 person-years: 29 expert proofreaders over 3 years |
| Synapses | ~130 million (54.5 million between proofread neurons) | 46 million presynapses connected to 312 million postsynaptic sites |
| Licence | CC BY-NC 4.0 (non-commercial) | CC BY 4.0 (commercial use allowed) |

...

---

**[👉 Continue reading: What Is MaleCNS? The First Whole-CNS Wiring Diagram of a Male Fruit Fly, and How It Differs from FlyWire](https://tools.cooconsbit.com/en/articles/malecns-male-fly-connectome-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
