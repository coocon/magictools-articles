# How the Fruit Fly Brain Map Was Made: 7,062 Slices, 21 Million Images, 3 Million Edits

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-map-how-made-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-map-how-made-en?utm_source=github&utm_medium=referral)**

In October 2024 the FlyWire Consortium published the first complete wiring diagram of an adult fruit fly brain in Nature: 139,255 neurons and 54.5 million synapses, with every neuron's partners and synapse counts mapped.

That map has since powered whole-brain simulations, been described as a "fruit fly brain upload", and been wired into Mario and Minecraft by hobbyists. Far fewer people explain how it was made. The answer: one fly, a diamond knife, two custom electron microscopes, a stack of AI, and many years of patience from hundreds of people.

This article goes step by step. The numbers come from the two original papers: Zheng et al. 2018 in Cell (the electron microscopy) and Dorkenwald et al. 2024 in Nature (the FlyWire reconstruction).

## The whole process at a glance

| Step | What happened | Key numbers |
|---|---|---|
| ① Choose a fly | One brain picked from several 7-day-old female flies | 1 fly |
| ② Slice | The brain cut into ultra-thin serial sections with a diamond knife | 7,062 sections, 35–40 nm each, about 3 weeks |
| ③ Image | Two custom high-throughput transmission electron microscopes image every section | 21 million images, 106 TB, about 16 months |
| ④ Align | Thousands of section images realigned in 3D | A neural network predicts the deformation between neighbouring sections |
| ⑤ Segment | AI traces every neurite in the images | A convolutional network decides, pixel by pixel, whether it's looking at a cell boundary |
| ⑥ Proofread | People fix the AI's mistakes one by one | 3,013,513 edits, about 33 person-years |
| ⑦ Synapses and labels | Synapses detected, neurotransmitters predicted, cell types annotated | 54.5 million synapses, 8,453 cell types |

## ①② One fly, 7,062 slices

Why electron microscopy? Because synapses are tiny. The contact between two neurons is tens of nanometres across, while light microscopy is limited by the wavelength of light to about 200 nm. To see every synapse you need electrons.

The catch: electrons can't pass through thick samples, so you can only image very thin sections. The whole brain has to be sliced.

Davi Bock's team at Janelia picked a 7-day-old female fly. Its brain tissue was soaked in heavy metals so cell membranes would show up under the electron beam, embedded in resin, checked with X-ray tomography, and then cut slice by slice with a diamond knife. **Each section is 35–40 nm thick — roughly two-thousandths the width of a human hair.** The fly brain is about 250 µm deep, so cutting all the way through took 7,062 sections and about three weeks. Three sections went onto each bar-coded metal grid, about 2,400 grids in all.

...

---

**[👉 Continue reading: How the Fruit Fly Brain Map Was Made: 7,062 Slices, 21 Million Images, 3 Million Edits](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-map-how-made-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
