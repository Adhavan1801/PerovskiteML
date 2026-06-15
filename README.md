<div align="center">

# PerovskiteML

### Predicting Perovskite Material Properties from Chemical Formulas using Machine Learning

*CBFV · Magpie Featurization · XGBoost · LightGBM · Extra Trees · Gradient Boosting*

<br/>

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=flat-square&logo=python&logoColor=white)](https://python.org)
[![scikit-learn](https://img.shields.io/badge/scikit--learn-1.x-F7931E?style=flat-square&logo=scikit-learn&logoColor=white)](https://scikit-learn.org)
[![XGBoost](https://img.shields.io/badge/XGBoost-Regressor-189CE5?style=flat-square)](https://xgboost.readthedocs.io)
[![LightGBM](https://img.shields.io/badge/LightGBM-Regressor-2E8B57?style=flat-square)](https://lightgbm.readthedocs.io)
[![License](https://img.shields.io/badge/License-MIT-22C55E?style=flat-square)](LICENSE)

<br/>

| Model | R² Score | RMSE |
|:-----:|:--------:|:----:|
| **Extra Trees (Tuned)** | **0.7763** | **28.17** |
| XGBoost | 0.7783 | 30.19 |
| LightGBM | 0.7676 | — |
| Gradient Boosting | 0.7543 | 30.21 |

</div>

---

## About the Project

Perovskite materials are a structurally diverse family of compounds with applications spanning solar cells, superconductors, and piezoelectrics. A key property of interest is the **Debye temperature** — a measure of lattice vibration stiffness that correlates with phonon dynamics, thermal conductivity, and phase stability.

Experimentally measuring material properties is expensive and slow. This project builds a **machine learning pipeline** that predicts the Debye temperature of perovskite oxides and halides directly from their **chemical formula** — with no crystal structure or DFT computation required.

**Dataset:** 588 double perovskite compounds (A₂BB'X₆ structure) with Debye temperatures ranging from 24.09 K to 380.39 K.

---

## Pipeline

```
Chemical Formula (string)
        │
        ▼
  CBFV / Magpie Featurization
  (273 element-property statistics)
        │
        ▼
  Train / Validation / Test Split (80/10/10)
        │
        ├──► XGBoost Regressor
        ├──► LightGBM Regressor
        ├──► Extra Trees Regressor    ◄── Best RMSE
        ├──► Gradient Boosting Regressor
        └──► Random Forest Regressor
                │
                ▼
        Bootstrapped Evaluation
        (R², Adjusted R², RMSE)
```

---

## Featurization — CBFV / Magpie

Features are generated using [CBFV](https://github.com/kaaiian/CBFV) with the **Magpie** element-property set, which encodes each element's properties (atomic number, electronegativity, melting point, valence electrons, covalent radius, etc.) and computes statistical aggregates (sum, mean, range, mode, maximum, minimum) over all elements in the formula.

This results in **273 numerical features** per compound — all derived from the formula string alone, with no structural input.

### Feature statistics generated per property
| Aggregation | Description |
|-------------|-------------|
| `sum_` | Sum across constituent elements |
| `mean_` | Weighted mean |
| `range_` | Max − Min |
| `mode_` | Most frequent value |
| `max_` / `min_` | Extremes |

---

## Models and Results

### Benchmark — 42 models screened (LazyPredict)
All models were first evaluated via LazyPredict to identify top candidates. Ensemble tree models dominated the leaderboard.

### Fine-tuned models

| Model | R² | RMSE | Notes |
|-------|----|------|-------|
| **Extra Trees (tuned)** | **0.776** | **28.17** | Best RMSE — selected as final model |
| XGBoost (bootstrapped) | 0.778 | 30.19 | Strong R², slightly higher RMSE |
| LightGBM | 0.768 | — | Fast training, competitive accuracy |
| Gradient Boosting | 0.754 | 30.21 | Slower, lower R² |

### Why Extra Trees?
Extra Trees introduces additional randomness in split selection compared to Random Forest — this reduces variance on small tabular datasets and often outperforms XGBoost on RMSE when the sample count is limited (588 samples here).

---

## Repository Structure

```
PerovskiteML/
├── data/
│   └── perovskites.xlsx                  # 588 perovskite compounds dataset (formula + target)
├── docs/
│   └── project_report.pdf                # Detailed project report / paper
├── notebooks/
│   ├── 01_eda.ipynb                      # Exploratory Data Analysis
│   ├── 02_featurization_benchmark.ipynb  # CBFV feature generation & model screening
│   ├── 03_xgboost.ipynb                  # XGBoost Regressor
│   ├── 04_lightgbm.ipynb                 # LightGBM Regressor
│   ├── 05_extra_trees.ipynb              # Extra Trees Regressor (best model)
│   ├── 06_gradient_boosting.ipynb        # Gradient Boosting Regressor
│   └── 07_full_pipeline.ipynb            # Full end-to-end pipeline
├── results/
│   └── model_comparison.csv              # Model comparison results (R² and RMSE)
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Dataset

| Property | Value |
|----------|-------|
| **Compounds** | 588 double perovskites (A₂BB'X₆) |
| **Target** | Debye Temperature (K) |
| **Target range** | 24.09 K — 380.39 K |
| **Target std** | ±58.18 K |
| **Formula examples** | ReBTeGeO6, GeZnAl2O6, TcCaMnBeO6 |
| **Anion sets** | Oxide (O6), Halide (Cl6, Br6) |

---

## Evaluation Metrics

| Metric | Description |
|--------|-------------|
| **R²** | Fraction of variance explained — higher is better |
| **Adjusted R²** | R² penalised for number of features |
| **RMSE** | Root Mean Squared Error in Kelvin — lower is better |

---

## Setup

```bash
# Clone the repository
git clone https://github.com/Adhavan1801/PerovskiteML.git
cd PerovskiteML

# Install the required dependencies
pip install -r requirements.txt
```

Open any notebook in `notebooks/` in Jupyter or Google Colab to run the pipeline.

---

## Key Takeaways

1. **Formula-only features are sufficient** — Magpie featurization encodes enough chemical information to predict Debye temperature at R² ≈ 0.78 without crystal structure data.
2. **Extra Trees outperforms on RMSE** — The additional split randomness reduces overfitting on the 588-sample dataset.
3. **Ensemble methods dominate** — Out of 42 models benchmarked via LazyPredict, all top performers were tree-based ensembles.
4. **Scalable to larger datasets** — The pipeline generalises to any material family expressible as a chemical formula.

---

<div align="center">
  Built by <a href="https://github.com/Adhavan1801">Adhavan U S</a>
</div>
