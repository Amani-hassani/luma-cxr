# Evaluation

[← Back to README](../README.md)

## Principles

1. **Lock first, then test.** Weights, preprocessing, calibration and ensemble aggregation were fixed before the
   external dataset was scored.
2. **Select on a different dataset.** Configuration choice used CheXpert. It never used the external test set.
3. **Report calibration, not just ranking.** Brier score, Brier skill and ECE are reported alongside AUROC.
4. **Quantify uncertainty.** 95% confidence intervals come from 2,000 bootstrap samples.
5. **Say what is not shown.** Every result below is scoped to its dataset, projection and label definition.

## Phase I: why the first model was withheld

| Measure | Internal test | External (RSNA, n = 26,683) |
|---|---|---|
| AUROC | 0.994 | 0.757 |
| Sensitivity at pre-set threshold | 0.98 | 0.94 |
| Specificity at pre-set threshold | 0.93 | 0.37 |

The threshold and calibration were fixed on a separate calibration set before external testing. The collapse in
specificity meant the model would flag most healthy adults, so it was not released.

> Detailed Phase I methods and comparisons are part of a paper being prepared for publication and are not reproduced here.

## Phase II: the released model

**External cohort:** VinBigData PA images, n = 14,250. **Positive:** at least two of three readers marked consolidation,
infiltration or lung opacity. **Negative:** no reader marked any of them.

| Measure | Estimate | 95% CI |
|---|---|---|
| AUROC | 0.854 | 0.839–0.869 |
| Brier score | 0.049 | — |
| Brier skill score | 0.112 | 0.082–0.142 |
| ECE (equal-width) | 0.061 | — |
| AUROC difference vs single-seed DenseNet-121 reference | +0.031 | 0.019–0.044 |

**Release criteria:** complete predictions, positive Brier skill, and AUROC above the reference. All three were met.

### Caveats

- The comparison with the reference combines architecture and ensemble-size differences.
- A full external comparison with and without distillation was not run, so the contribution of distillation is not
  established.
- Excluding single-reader cases removes ambiguous images and may make the task easier.
- These are retrospective metrics. They say nothing about clinical usefulness at a chosen operating point.

## Export and runtime checks

| Check | Result | Project limit |
|---|---|---|
| Model files | 22.7 MiB | 150 MiB |
| Median CPU latency | 96 ms | 500 ms |
| p95 CPU latency | 118 ms | 1,000 ms |
| Memory increase | 41.6 MiB | 2,048 MiB |
| ONNX vs PyTorch max probability difference | 4.8 × 10⁻⁷ | — |
| Inference and export tests | 47 passing | — |
