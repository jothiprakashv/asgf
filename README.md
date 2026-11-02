# AS-GF

**Admissible-State Generative Forecasting for Reliable Service and Business Operations**

This repository accompanies the manuscript:

> **Admissible-State Generative Forecasting for Reliable Service and Business Operations**

Repository: https://github.com/jothiprakashv/asgf

## Overview

Generative forecasters support capacity planning, demand anticipation and economic decision-making in service-oriented systems. Optimizing for accuracy alone, however, often yields trajectories that break basic operational properties: negative values, abrupt transitions, exaggerated responses to small input perturbations, or responses that contradict a policy signal. Such trajectories cannot be passed safely to downstream planning and control.

This work introduces **AS-GF (Admissible-State Generative Forecasting)**, a constraint-governed generative forecasting framework. Instead of treating feasibility as a post hoc check, AS-GF builds it into both training and inference through three components:

* **Admissible state space over state windows.** Static constraints (window length 1) and transition constraints (window length 2) are expressed uniformly as `g_i(ξ) ≤ 0`. The constraint classes are non-negativity, temporal inertia, bounded change and policy consistency.
* **Constraint-governed training objective.** A hinge penalty on the generated trajectory is added to the prediction loss, `L_total = L_pred + λ L_adm`, with λ selected on a validation split.
* **Sequential inference-time projection.** The raw trajectory is projected step by step, conditioned on the already-projected predecessor, so that transition constraints hold on the emitted forecast. Closed-form projections are used where they exist. Elsewhere the solver uses Dykstra-style alternating projection with a feasibility check and a fallback to the last admissible state.

AS-GF is evaluated against ARIMA, ETS, LSTM, a Transformer and an unconstrained generator that has the identical backbone (λ = 0, no projection), so the comparison isolates the effect of the constraints. The system-level metrics are the admissibility violation rate (AVR, reported on the raw generator output and on the emitted forecast), the mean violation magnitude (MVM) and the forecast stability index (FSI).

## Datasets

Experiments use three publicly available benchmarks:

| Dataset | Granularity | Horizon | Role |
|---|---|---|---|
| OECD Main Economic Indicators | monthly / quarterly / annual | H = 12 | long-term service planning |
| World Bank World Development Indicators | annual | H = 5 | strategic service provisioning |
| M5 Forecasting | daily | H = 28 | retail demand |

Splits are 70% training, 10% validation and 20% testing. The training-derived constraint bounds (κ, ρ) are estimated on the training split only and frozen for validation and test.

## Experimental Setup (summary)

| Item | Setting |
|---|---|
| Backbone | Conditional VAE with 2-layer GRU encoder and decoder, 128 hidden units, latent dimension 32 |
| Optimizer | Adam, learning rate 1e-3, batch size 128 |
| Training | up to 200 epochs, early stopping with patience 20 on validation loss |
| Admissibility weight | λ* = 1.0, selected on the validation split |
| Inertia bound κ | 99.5th percentile of training transition magnitude (per series) |
| Bounded-change bound ρ | 99th percentile of absolute training one-step change (per coordinate) |
| Seeds | 10 |
| Samples per forecast | 20 latent samples, predictive mean |

Full hyperparameters, closed-form projections, solver diagnostics, runtime and memory measurements, per-dataset results and statistical tests are given in the supplementary material of the manuscript.

## Code Availability

The source code supporting the implementation and evaluation of AS-GF will be publicly released through this repository upon acceptance of the manuscript.

The code release aims to support research transparency, reproducibility, and further development of constraint-aware generative forecasting methods.

## Data Availability

The experiments in this study use publicly available datasets:

* **OECD Main Economic Indicators:** https://doi.org/10.1787/data-00052-en
* **World Bank World Development Indicators:** https://databank.worldbank.org/source/world-development-indicators
* **M5 Forecasting (Accuracy):** https://doi.org/10.5281/zenodo.12636070

## Authors

| Name | Email | Country |
|---|---|---|
| Dr. Jothi Prakash V | jothiprakashv@gmail.com | India |
| Dr. Arul Antran Vijay S (submitting author, ORCID [0000-0002-5543-7547](https://orcid.org/0000-0002-5543-7547)) | arulantranvijay@gmail.com | India |
| Dr. Mahendhiran PD | mahendhiranpd@gmail.com | India |
| Dr. Sreeja Balachandran Nair Premakumari (corresponding author) | bp_sreeja@cb.amrita.edu | India |

## Contact

For questions regarding this work, please contact the corresponding author:

Dr. Sreeja Balachandran Nair Premakumari
Email: bp_sreeja@cb.amrita.edu

## Disclaimer

This repository is intended for scientific research and reproducibility purposes. AS-GF is a computational forecasting method. Its admissibility guarantee holds only with respect to the constraint set that is specified, and it is not a substitute for expert judgment in policy, planning or operational decisions.
