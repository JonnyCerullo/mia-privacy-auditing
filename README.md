[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/JonnyCerullo/mia-privacy-auditing/blob/main/MIA_Project.ipynb)

# Membership Inference Attacks on ML Models

### Privacy Auditing and GDPR Implications

A privacy-auditing pipeline for image classifiers. Two membership-inference
attack strategies of increasing sophistication are implemented and evaluated
against five model configurations spanning the spectrum from "no defense" to
"differentially private training", and the empirical results are mapped to
specific provisions of the General Data Protection Regulation (GDPR).

---

## Author

**Jonathan Ted Benson Cerullo Uyi**
`jonathan.cerullouyi@studio.unibo.it` — Matricola: 0001164850
MSc in Artificial Intelligence — University of Bologna
Course: *Ethics in Artificial Intelligence* (A.Y. 2025/2026)

---

## Overview

Trained ML models can leak information about their training data. The most
studied form of this leakage is the **Membership Inference Attack (MIA)**: an
adversary with black-box query access to a model attempts to determine whether
a given record was part of its training set. A successful MIA does not
necessarily recover the record's contents, but it tells the attacker that the
record exists in the training corpus — which is enough to violate
confidentiality in many real-world contexts (medical imaging, biometric
recognition, demographic profiling).

This project pursues two parallel goals:

- **A technical contribution.** Build a reproducible audit pipeline that
  quantifies MIA vulnerability across realistic defense configurations, and
  measures the privacy/utility trade-off of Differentially Private SGD
  (DP-SGD) as a formal defense.
- **A normative contribution.** Take the empirical numbers and use them to
  interrogate what the GDPR's risk-based standard for "appropriate technical
  measures" (Art.~32) should mean in concrete terms for ML systems trained on
  personal data.

---

## Methodology

**Attacks**
- Threshold-based attack (Yeom et al., 2018): per-sample loss as the
  membership signal.
- Shadow-model attack (Shokri et al., 2017): a meta-classifier trained on the
  outputs of multiple shadow models, applied to the target model's outputs.

Both attacks are evaluated with privacy-aware metrics — AUC, TPR@1%FPR,
TPR@0.1%FPR — rather than aggregate accuracy alone.

**Defenses**
- Informal: L2 weight decay, dropout, early stopping (no formal guarantee).
- Formal: DP-SGD via [Opacus](https://opacus.ai), swept over privacy budgets
  ε ∈ {1, 5, 10} at δ = 10⁻⁵.

**Experimental design**

CIFAR-10 (60,000 32×32 colour images, 10 classes), partitioned into six
mutually disjoint pools — target/target-test, shadow-train/shadow-test,
regularisation validation, and DP training pool — with runtime assertions
enforcing both pairwise disjointness and the inclusion of the target
members in the DP training pool.

A small CNN (~1.05M parameters) is used throughout. Runs are seeded
(`SEED = 42`) across NumPy, PyTorch CPU/CUDA, and the cuDNN backend
(`cudnn.deterministic = True`).

---

## GDPR Mapping

The empirical findings are mapped to four GDPR articles:

- **Art. 5(1)(f)** — Integrity and confidentiality. MIA on an undefended
  model is a confidentiality breach mediated by the model itself.
- **Art. 5(1)(c)** — Data minimisation. DP-SGD operates as an
  *algorithmic* data-minimisation mechanism.
- **Art. 6** — Lawfulness, with emphasis on the necessity/proportionality
  test for legitimate-interest processing.
- **Art. 17** — Right to erasure: under DP-SGD, each record's "memory" in
  the trained weights is mathematically bounded.
- **Art. 32** — Security of processing: empirical privacy auditing
  operationalises "appropriate to the risk".

---

## Repository contents

- `MIA_Project.ipynb` — main Jupyter notebook with the full pipeline
  (data loading, model architecture, attacks, defenses, results, figures,
  GDPR analysis).
- Final report (LaTeX)
- Presentation slides

---

## How to reproduce

The notebook is designed to run end-to-end on Google Colab with a GPU
runtime (T4 or better).

1. Open `MIA_Project.ipynb` in Colab.
2. Run all cells from top to bottom.
3. The CIFAR-10 dataset is downloaded automatically from the Hugging Face
   Hub (no manual setup required).
4. Total runtime is approximately 1 hour on a Colab T4 GPU.

Outputs are written to `results/results.json` and `figures/*.pdf` inside
the Colab session; the latest figures are also archived under `imgs/` in
this repository for inclusion in the final report.

---

## References

- Shokri, R., Stronati, M., Song, C., & Shmatikov, V. (2017).
  *Membership Inference Attacks Against Machine Learning Models.*
  IEEE Symposium on Security and Privacy.
- Yeom, S., Giacomelli, I., Fredrikson, M., & Jha, S. (2018).
  *Privacy Risk in Machine Learning: Analyzing the Connection to
  Overfitting.* IEEE Computer Security Foundations Symposium.
- Carlini, N., Chien, S., Nasr, M., Song, S., Terzis, A., & Tramèr, F.
  (2022). *Membership Inference Attacks From First Principles.*
  IEEE Symposium on Security and Privacy.
- Abadi, M., Chu, A., Goodfellow, I., McMahan, H. B., Mironov, I.,
  Talwar, K., & Zhang, L. (2016). *Deep Learning with Differential
  Privacy.* ACM Conference on Computer and Communications Security.
- Tramèr, F., & Boneh, D. (2021). *Differentially Private Learning Needs
  Better Features (or Much More Data).* ICLR.
- Regulation (EU) 2016/679 of the European Parliament and of the Council
  (GDPR). *Official Journal of the European Union*, L 119/1, 2016.

---

## Notes

This work was developed for the *Ethics in AI* exam at the University of
Bologna. It is an academic privacy-auditing exercise and is not intended
as a production privacy-auditing tool.
