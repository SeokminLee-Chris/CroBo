# Behind the Scenes

Things that didn't make it into the paper — but shaped it.

---

## Masking Ratio

### The Short Version

We tried ~25 different architectures before landing on CroBo. By the time we had something worth submitting, there was no time left for a proper masking ratio ablation — even though it's a core hyperparameter.

A reviewer caught this during rebuttal and asked us to run it. We did, and **95% turned out to be surprisingly strong**.

### What Happened

The ablation results came in during the rebuttal period, and 95% clearly outperformed the default 90% we had been using. But we were up against the camera-ready deadline, and the main paper couldn't be substantially revised at that stage. So we made a pragmatic call:

- Report the 95% result **in the ablation table** (100-epoch training)
- Keep the **default setting at 90%** to avoid touching the main results

It's not ideal, but it's honest — and that's why this note exists.

### Ablation Table (Franka Kitchen, 100 Epochs)

| Masking Ratio | Knob1 On | Light On | S.Door Open | L.Door Open | Micro Open |
|:---:|:---:|:---:|:---:|:---:|:---:|
| 75% | 41.4 | 70.6 | 94.0 | 22.6 | 35.0 |
| 90% | 57.6 | 81.6 | 98.6 | 36.8 | 50.4 |
| **95%** | **59.0** | **86.6** | **99.4** | **41.2** | **58.0** |
| 97.5% | 52.8 | 79.6 | 99.6 | 38.8 | 55.2 |

95% hits a sweet spot — 97.5% starts to hurt performance, likely because too few visible patches remain to provide meaningful spatial context for reconstruction.

---

## Training Cost

### VRAM Usage & 1 Epoch Time

Training was done on **2× NVIDIA H200 GPUs** with a batch size of **1536**.

| Backbone | VRAM per GPU | 1 Epoch Time |
|:---:|:---:|:---:|
| ViT-S | 14,430 MiB | ~22 min |
| ViT-B | 20,856 MiB | ~22 min |
| ViT-L | 41,910 MiB | ~22 min |

### Why H200?

VRAM usage alone doesn't tell the full story. With a batch size of 1536, the real bottleneck is **data loading**, not compute — which makes epoch time nearly identical across all backbone sizes.

To saturate the data pipeline efficiently, we rely on **distributed training across 2 GPUs**. The H200's NVLink interconnect makes inter-GPU communication significantly faster than alternatives, which is critical at this batch size. We tested on H100s and observed a **~2.5× slowdown**, despite similar compute specs — the difference comes down to interconnect bandwidth and how aggressively it can feed the data pipeline.
