# Model card: Luma adult chest X-ray ensemble

[← Back to README](../README.md)

| | |
|---|---|
| **Model** | Ensemble of three MobileViT-XS classifiers (three training seeds), exported to ONNX |
| **Training approach** | Supervised training plus representation distillation from a frozen RAD-DINO teacher (a chest X-ray foundation model). The teacher is used only during training and is not part of the exported model. |
| **Output** | A probability-like score (shown as 0–100) for **radiographic air-space opacity**, used as a proxy for pneumonia-pattern findings |
| **Aggregation** | The three models' logits are averaged, then one temperature fitted on a held-out calibration set is applied |
| **Input** | Adult, frontal (PA) chest radiograph; grayscale 224 × 224 after percentile windowing |
| **Version** | Locked release, with SHA-256 identities verified before any ONNX session opens |
| **Owner** | Amani Hassani, ESPRIT-Tech internship (2026) |

## Intended use

- Academic research and demonstration of external validation, calibration and safe ML deployment practices.
- **Out of scope:** clinical diagnosis, triage, treatment decisions, patient communication, paediatric images, AP or
  lateral projections, and images with burnt-in identifiers.

## Data

| Role | Dataset | Size | Notes |
|---|---|---|---|
| Training, validation, calibration | RSNA Pneumonia Detection (adult, NIH-derived) | 16,512 training images, plus separate validation, calibration (2,540) and development-test sets | Grouped by released identifiers |
| Configuration selection | CheXpert | 184,325 report-labelled frontal images | Used only to choose between candidate configurations |
| Final external test | VinBigData (VinDr-CXR competition set) | 14,250 PA images | Untouched until the model, weights, preprocessing and aggregation were fixed |

**Positive label on the external test:** at least two of three radiologists marked consolidation, infiltration or lung
opacity. **Negative:** no reader marked any of them. The 750 images marked by exactly one reader were excluded.

## Performance (external, VinBigData PA, n = 14,250)

| Measure | Estimate | 95% CI |
|---|---|---|
| AUROC | 0.854 | 0.839–0.869 |
| Brier score | 0.049 | — |
| Brier skill score | 0.112 | 0.082–0.142 |
| ECE (equal-width) | 0.061 | — |

## Runtime (CPU, 4 logical cores, ONNX Runtime 1.29)

| Model files | Median latency | p95 latency | Memory increase |
|---|---|---|---|
| 22.7 MiB (three models) | 96 ms | 118 ms | 41.6 MiB |

## Known limitations and risks

- **Proxy target.** The model detects an opacity pattern, not confirmed pneumonia.
- **Shortcut risk.** The learned features retain projection (AP/PA) information and some sensitivity to image borders,
  so the model must not be used outside the PA scope.
- **Teacher overlap.** RAD-DINO's pretraining data includes the NIH source of the RSNA images, and exact overlap was not
  measured.
- **Unmeasured subgroups.** Performance across age, sex, devices and sites was not established.
- **Retrospective only.** No prospective or clinical-utility evaluation has been done.

## Why an earlier model was withheld

A Phase I paediatric pneumonia model reached 0.994 AUROC internally but 0.757 on 26,683 external RSNA images (specificity
0.37 at its pre-set threshold). It was never released. That failure is the reason this model was selected and locked
before any external testing. The Phase I study itself is being prepared for publication.
