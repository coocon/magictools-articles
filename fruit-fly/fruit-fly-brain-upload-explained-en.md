# Was a Fruit Fly Brain Really Uploaded to a Computer? Eon's Demo, Taken Apart Layer by Layer

> 📍 Originally published at [MagicTools](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-upload-explained-en?utm_source=github&utm_medium=referral). This mirror only carries a preview — **[read the full article →](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-upload-explained-en?utm_source=github&utm_medium=referral)**

On March 8, 2026, San Francisco startup Eon Systems posted: "We've uploaded a fruit fly." In the video, a virtual fly follows invisible taste cues toward slices of banana, stops to groom when dust settles on it, carries on, and starts eating. For many people it was the first time they'd heard that an animal's brain could be copied into a computer.

Is that what happened? I rebuilt the system on a Mac mini from the same open-source components and read Eon's two posts and the three Nature papers behind them line by line. The short version:

- **What was copied is a wiring diagram**: which of the 139,255 neurons in one female fly's brain connect to which, and through how many synapses. That part is solid, published in Nature in 2024, and the data is public.
- **Most of what makes it move doesn't come from that diagram.** Walking, grooming and feeding movements are produced by controllers that already existed in the body model, and the mapping between brain and body was chosen by hand. Eon says so in its own technical write-up.
- **The widely shared "91% behavior accuracy" is a misreading.** It comes from a 2024 paper in which the model made 164 testable predictions about feeding and grooming circuits, 91% of which matched experiments. Those are circuit-level predictions, not a behavior score for the virtual fly.

Let's take it apart.

## An "upload" in four layers

Eon's announcement said it used just four things: the graph of connections, weights set by synapse counts, a map of excitatory and inhibitory neurons, and a leaky integrate-and-fire (LIF) neuron model. The technical post is more complete: this was an **integration** of several already published pieces.

| Layer | What it is | Who built it | Openness |
|---|---|---|---|
| ① Wiring diagram | Connectome of 139,000 neurons and 54.5 million synapses | FlyWire Consortium, Nature 2024 | Data public, CC BY-NC 4.0 |
| ② Neuron model | Every neuron as an LIF unit, whole brain run in Brian2 | Shiu et al., Nature 2024 (preprint 2023) | Code MIT |
| ③ Body | A fly body modelled from an X-ray microtomography scan, 87 independent joints, running in MuJoCo | NeuroMechFly v2, Wang-Chen et al. 2024 | Apache-2.0 |
| ④ Brain–body interface | Translates a few descending neurons' firing into turning, walking, grooming and feeding commands | Eon | No public code that I could find |

A visual model (Lappalainen et al. 2024, flyvis) also turns the compound-eye image into visual-neuron activity that feeds the brain model — but Eon itself describes this part as "somewhat 'decorative'" for now, with little effect on behavior.

...

---

**[👉 Continue reading: Was a Fruit Fly Brain Really Uploaded to a Computer? Eon's Demo, Taken Apart Layer by Layer](https://tools.cooconsbit.com/en/articles/fruit-fly-brain-upload-explained-en?utm_source=github&utm_medium=referral)**

More articles: [tools.cooconsbit.com/articles](https://tools.cooconsbit.com/en/articles?utm_source=github&utm_medium=referral)
