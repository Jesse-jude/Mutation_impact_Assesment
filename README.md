# 🧬 HelixMind — Mutation Impact Predictor

> **Prototype Learning Core v0.1** — A 1D CNN that learns to detect biologically significant point mutations from DNA sequence context.

This is not a rule-based simulator. It's a **learned biological signal detector** — trained to recognize sequence patterns that correlate with functional impact, the same way a real mutation effect predictor would work at scale.

Built as the first learning module in the HelixMind pipeline: a system for predicting microbial mutation behaviour and antimicrobial resistance.

---

## What It Does

Takes a DNA sequence + a point mutation and outputs a probability that the mutation disrupts protein function.

```
Input:  DNA sequence (50 bases) + mutation position + new base
Model:  1D CNN → GlobalMaxPooling → Dense → Sigmoid
Output: Impact probability  e.g. 0.87 → HIGH FUNCTIONAL IMPACT 🔴
```

---

## Model Architecture

```
DNA Sequence Input  (50, 4)  ← one-hot encoded
    └─ Conv1D(32, kernel=3) + BatchNorm + ReLU
    └─ Conv1D(64, kernel=3) + BatchNorm + ReLU
    └─ GlobalMaxPooling1D
    └─ Dense(32) + Dropout(0.3)
         ↕  Concatenate
Mutation Position  (1) ──┐
New Base Identity  (1) ──┘
    └─ Dense(8) → Dense(16) → Dense(1, sigmoid)
```

Two-branch design: the CNN reads full sequence context, while a second branch encodes *where* the mutation is and *what* the new base is. They merge before the final classification head.

---

## Biological Logic

Training data is synthetic but encodes real biological reasoning:

| Condition | Label |
|---|---|
| Mutation falls inside a functional motif (`ATGCG`, `GCATG`, `TTAGG`) | High Impact (1) |
| Mutation falls outside all motifs | Low Impact (0) |
| ~10% random label flip | Biological ambiguity noise |

The model learns to associate sequence context around the mutation site with functional consequence — it doesn't just memorise positions.

---

## Results

Trained on 5,000 synthetic samples, 80/10/10 split.

| Metric | Score |
|---|---|
| Test ROC-AUC | ~0.92 |
| Accuracy | ~88% |
| Training time | ~3 min on Colab CPU |

---

## Quickstart

**Run in Google Colab — no setup needed.**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/)

1. Upload `HelixMind_Mutation_Predictor.ipynb` to Colab
2. Runtime → Run All
3. Edit the interactive cell at the bottom to test your own sequences

---

## Project Structure

```
helixmind-mutation-predictor/
├── HelixMind_Mutation_Predictor.ipynb   # Main notebook (data → train → eval → demo)
├── helixmind_demo.html                  # Standalone browser demo (no Python needed)
├── helixmind_mutation_core_v0.keras     # Saved model (generated after training)
└── README.md
```

---

## Live Demo

Open `helixmind_demo.html` in any browser. Paste a 50-base DNA sequence, pick a mutation position and new base, and get an instant prediction with a sequence map highlighting the mutated position.

The demo runs the same biological signal logic client-side — no server required.

---

## Using the Predictor

```python
# After running all notebook cells:

result = predict_mutation_impact(
    sequence='AAAAAAAAAAAAAAAAAAAATGCGAAAAAAAAAAAAAAAAAAAAAAAAAAA'[:50],
    mutation_position=20,
    new_base='C'
)

# Output:
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#   🧬 HelixMind Mutation Impact Assessment
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
#   Mutation:     A→C at position 20
#   Impact Prob:  0.8731
#   Prediction:   HIGH IMPACT 🔴
#   Confidence:   87.3%
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**Important — always build test sequences programmatically to avoid off-by-one errors:**

```python
motif = 'ATGCG'
pos = 20
sequence = 'A' * pos + motif + 'A' * (SEQ_LEN - pos - len(motif))  # guaranteed 50 bases
```

---

## Roadmap

This is v0.1 of the HelixMind learning core. Planned expansions:

- [ ] Swap synthetic data for real ClinVar / UniProt variant datasets
- [ ] Sliding window input for full-length gene sequences
- [ ] Resistance phenotype prediction head (AMR focus)
- [ ] FastAPI wrapper → HelixMind backend module
- [ ] Streamlit UI for wet-lab researcher use

---

## Stack

- Python 3.10+
- TensorFlow / Keras
- NumPy, scikit-learn, matplotlib, seaborn
- Google Colab (free tier compatible)

---



---

*Built with the goal of merging ML engineering and computational biology into something that actually matters.*
