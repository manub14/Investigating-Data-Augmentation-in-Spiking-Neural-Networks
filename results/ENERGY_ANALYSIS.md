# Estimated inference energy on CIFAR-100 (Spike-driven Transformer)

This document reports an **operation-level inference energy estimate** from the SNN augmentation experiments, not measurements of actual electricity consumption or power on neuromorphic hardware. Values are reproduced from **Table 5.7 of Manu Singh Bist, *Augmenting Spiking Neural Networks: Pre- and Post-Encoding Strategies for Robust and Energy-Efficient Learning*, MSc thesis, University of Hildesheim (2026)**. The accompanying model and augmentation implementations are in this repository.

## Main finding

On SDT/CIFAR-100, post-encoding PatchDropout improved top-1 test accuracy from **78.40% to 78.93%** (+0.53 percentage points) and reduced estimated inference energy from **572.69 µJ to 568.24 µJ per image** (**0.78% lower**). Other methods, including TimeMix and PAPMix, also reduced the estimated inference energy while exceeding baseline accuracy. The results show that the accuracy and estimated energy effects should be considered together, and that energy savings are method-dependent.

## Model and estimation protocol

- **Model and dataset:** Spike-driven Transformer (SDT) on CIFAR-100; model dimension D = 512, two encoder blocks, four simulation timesteps (T = 4), pooling configuration `0011`, and 64 tokens, as described in the thesis.
- **Aggregation:** Values reported in the thesis are means over five runs.
- **Technology assumption:** 45 nm CMOS operation-level energy model used by Yao et al. (2023, *Spike-driven Transformer*, NeurIPS).
- **Per-operation costs:** multiply-accumulate (MAC) = **4.6 pJ** and accumulate-only (AC) = **0.9 pJ**.
- **Computation model:** operation counts are weighted by layer activity/firing rates and the relevant MAC or AC energy constant. The input/patch-splitting convolution is accounted for at MAC cost, while subsequent spike-driven operations are accounted for at AC cost in the thesis model.
- **Components:** SPS (spike patch splitting and associated convolutions), SSA (spike self-attention) and MLP. The classification head is excluded in this comparison; the thesis reports that its contribution stays below 0.08 µJ.
- **Metric:** estimated per-image inference energy, shown in microjoules (µJ), and top-1 CIFAR-100 classification accuracy (%). Percentage change is computed as `100 × (method energy − 572.69) / 572.69`.
- **Caveat:** numerical component totals and the overall total are transcribed as printed in the thesis, so small discrepancies may occur from rounding or the source accounting.

## Full reported results

Lower energy is preferable; positive energy change means **higher** estimated energy consumption than the baseline. `Input` indicates augmentation before the first spike-generating LIF stage, while `Post` indicates augmentation after it. The baseline is the usual crop/flip training recipe without an extra named augmentation.

| Augmentation | Placement | SPS (µJ) | SSA (µJ) | MLP (µJ) | Total (µJ) | Energy change | Accuracy (%) |
|:--|:--|--:|--:|--:|--:|--:|--:|
| Standard baseline | Reference | 345.29 | 92.18 | 135.17 | 572.69 | reference | 78.40 |
| No augmentation | Reference | 249.52 | 81.23 | 119.57 | 450.37 | -21.36% | 68.39 |
| PatchShuffle | Input | 380.54 | 96.37 | 138.62 | 615.57 | +7.49% | 78.78 |
| Frequency Encoding | Post | 374.66 | 93.77 | 137.91 | 606.39 | +5.88% | 78.40 |
| RSM | Post | 379.85 | 82.99 | 120.38 | 583.28 | +1.85% | 76.13 |
| PatchMix | Input | 354.39 | 92.16 | 132.92 | 579.53 | +1.19% | 79.29 |
| PatchDropout | Input | 339.30 | 95.25 | 142.80 | 577.41 | +0.82% | 78.67 |
| CenterPatch MinLift | Post | 344.46 | 91.84 | 134.83 | 571.19 | -0.26% | 79.15 |
| Spatial Hole Filling (SHF) | Post | 343.78 | 91.59 | 134.51 | 569.92 | -0.48% | 79.06 |
| TimeMask | Post | 360.16 | 86.99 | 136.72 | 583.93 | +1.96% | 79.56 |
| TimeShuffle | Post | 344.59 | 95.25 | 137.30 | 577.19 | +0.79% | 79.45 |
| Temporal Jitter | Post | 346.82 | 92.12 | 135.33 | 574.33 | +0.29% | 78.90 |
| Local TimeShuffle | Post | 346.31 | 91.67 | 134.25 | 572.28 | -0.07% | 79.15 |
| TimeMix | Post | 327.68 | 86.23 | 134.67 | 548.64 | -4.20% | 78.69 |
| Binary Spike CutMix | Post | 351.43 | 94.17 | 136.22 | 581.88 | +1.60% | 79.38 |
| Spatiotemporal Hole Filling (STHF) | Post | 343.27 | 91.82 | 134.87 | 570.01 | -0.47% | 79.17 |
| PAPMix | Post | 327.42 | 85.97 | 133.98 | 547.43 | -4.41% | 78.80 |
| IPMix | Input | 366.80 | 99.59 | 151.00 | 617.46 | +7.82% | 74.43 |
| LayerMix | Input | 342.36 | 88.63 | 129.75 | 560.80 | -2.08% | 72.64 |
| PatchMix (post-encoding) | Post | 352.65 | 92.44 | 133.82 | 578.97 | +1.10% | 78.99 |
| PatchShuffle (post-encoding) | Post | 350.81 | 89.85 | 130.95 | 571.67 | -0.18% | 78.44 |
| PatchDropout (post-encoding) | Post | 345.15 | 89.81 | 133.22 | 568.24 | -0.78% | 78.93 |

## Reproducibility status

Code currently present in the repository:

- [`Spike-Driven-Transformer/firing_num.py`](../Spike-Driven-Transformer/firing_num.py) collects firing/nonzero activity statistics.
- [`Spike-Driven-Transformer/plot_spike_ratio.py`](../Spike-Driven-Transformer/plot_spike_ratio.py) reads firing-rate log blocks, generates spike-ratio CSV summaries and plots.

**Important:** those two scripts do **not** by themselves constitute a complete end-to-end reproduction of the energy totals in this table. Fully reproducible estimates would additionally require the trained checkpoints or representative logs, the per-layer operation counts, an explicit MAC/AC aggregation script, and the evaluation configuration. These should be deposited and linked if available. Do not describe the numbers as directly measured hardware energy.

## References

1. Manu Singh Bist. *Augmenting Spiking Neural Networks: Pre- and Post-Encoding Strategies for Robust and Energy-Efficient Learning*. MSc thesis, University of Hildesheim, 2026. Energy ablation: Section 5.6; Table 5.7 (printed page 83).
2. M. Yao et al. *Spike-driven Transformer*. NeurIPS, 2023. Basis for the operation-level energy model.

## Supplementary material

The [existing supplementary methodology and results PDF](../ESANN_SNN_Supplementary_Methodology_with_Results.pdf) covers augmentation methods and classification outcomes. This Markdown document separately provides the energy analysis, which is not currently part of that PDF.
