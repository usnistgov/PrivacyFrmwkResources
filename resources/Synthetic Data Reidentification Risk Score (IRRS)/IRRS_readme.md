
# IRRS – Synthetic Data Reidentification Risk Score

*A heuristic framework for estimating reidentification risk in AI-generated synthetic data.*

---

## Abstract

AI-generated synthetic data are often proposed as a way to:

- reduce privacy risks
- enable data sharing in regulated domains
- support research and model development

However, recent research shows that synthetic data do not automatically eliminate reidentification risk, particularly under:

- membership inference
- attribute inference
- singling-out attacks

This document introduces the **IRRS – Synthetic Data Reidentification Risk Score**, a heuristic 0–100 metric that estimates residual reidentification risk without access to the real dataset, using only high-level information about the synthetic data generation process.

IRRS is conceptually aligned with:

- NIST Privacy Framework
- NIST AI Risk Management Framework

but is not a normative implementation and does not replace formal audits or mathematical guarantees.

---

## Goal

Provide a metric that is:

- transparent
- interpretable
- reproducible
- open source

to support:

- privacy engineering
- impact assessments
- AI governance
- education and applied research

---

## Risk dimensions considered

IRRS considers three major classes of reidentification risk.

### 1. Membership inference

Key question: *Was this individual included in the training dataset?*

Demonstrated in:
- Shokri et al. (2017)
- Yeom et al. (2018)

### 2. Attribute inference

Key question: *Can sensitive attributes be inferred even if not explicitly disclosed?*

Demonstrated in:
- Fredrikson et al. (2015)

### 3. Singling-out

Related to the GDPR notion of identifiability (Recital 26):

> Is it possible to isolate an individual record in the synthetic dataset?

Associated with linkage attacks:
- Sweeney (2000)
- Narayanan & Shmatikov (2008)

---

## Variables used in the model

IRRS uses only **metadata about the generation process**:

- does not access the real dataset
- does not process personal data
- is consistent with privacy by design

Variables include:

- sensitivity of training data
- model complexity and memorization capacity
- number of quasi-identifiers
- proportion of rare records
- synthetic dataset size
- synthesis mode
- privacy-enhancing techniques applied

---

## Normalization of variables

### Sensitivity of training data

| Category | Normalized value |
|----------|------------------|
| low | 0.2 |
| moderate | 0.6 |
| high | 1.0 |

### Model complexity / memorization capacity

| Model type | Value |
|------------|-------|
| GAN, VAE, diffusion, autoregressive | 1.0 |
| statistical, simple noise | 0.2 |
| other | 0.5 |

### Quasi-identifiers

```
Q_QI = min(1, n_QI / 10)
```

### Rare records

```
R_rare = proportion of rare or unique records (0–1)
```

### Synthetic dataset size

```
T_N = 1 − log10(N) / 6
```

---

## Privacy techniques and heuristic weights

| Technique | Effectiveness weight |
|----------|----------------------|
| differential_privacy | 0.6 |
| aggregation_only | 0.5 |
| k_anonymity | 0.4 |
| l_diversity | 0.3 |
| t_closeness | 0.3 |
| generalization | 0.25 |
| suppression | 0.2 |
| noise_addition | 0.2 |

Combined using:

```
P_priv = 1 − ∏ (1 − e_t)
```

where `e_t` is each technique's effectiveness weight.

---

## Partial risk scores

Partial risks (0–100):

- R_mem — membership inference
- R_attr — attribute inference
- R_single — singling-out

Example (membership inference):

```
r_mem =
  0.35 S_sens +
  0.20 C_model +
  0.15 Q_QI +
  0.15 R_rare +
  0.10 M_mode −
  0.25 P_priv
```

Values are truncated to [0,1] and scaled to 0–100.

---

## Global IRRS score

```
IRRS = 0.4 R_mem + 0.3 R_attr + 0.3 R_single
```

Qualitative categories:

| Range | Category |
|-------|----------|
| 0–29 | Low |
| 30–59 | Moderate |
| 60–100 | High |

---

## Limitations

- does not compute epsilon for Differential Privacy
- does not access the original dataset
- depends on declared parameters
- cannot detect individual leakage cases
- heuristic weights are empirically motivated

---

## Selected references

- Carlini et al., 2019 — The Secret Sharer: Measuring Unintended Neural Network Memorization
- Carlini et al., 2021 — Extracting Training Data from Large Language Models
- Dwork et al., 2006 — Calibrating Noise to Sensitivity in Private Data Analysis
- Fredrikson et al., 2015 — Model Inversion Attacks that Exploit Confidence Information
- Jordon et al., 2022 — Synthetic Data — A Review
- Narayanan & Shmatikov, 2008 — Robust De-anonymization of Large Sparse Datasets
- Shokri et al., 2017 — Membership Inference Attacks Against Machine Learning Models
- Sweeney, 2000 — Simple Demographics Often Identify People Uniquely
