# Data Science in Healthcare — Radiomics CAD for CT Colonography

> *Data Science in Healthcare (DSH) — Master's Degree in Biomedical Engineering, Politecnico di Torino, A.A. 2025/26*

## Project

Design and evaluation of a **radiomics-based Computer-Aided Diagnosis (CAD) pipeline** to distinguish advanced diverticular disease from colorectal cancer (CRC) using 3D CT Colonography (CTC). The pipeline covers the full ML workflow — from raw image statistics to classifier validation — in a realistic multi-center scenario, with explicit modeling of inter-observer variability.

**Clinical motivation:** In the geriatric population, advanced diverticular disease can closely mimic CRC on CT Colonography, and biopsy is the only definitive differentiator. A CAD system offering quantitative, reproducible radiomic analysis can support — not replace — clinical judgment, reducing unnecessary invasive procedures.

**Dataset:** 148 3D CTC volumes from two acquisition centers. Center A (construction set) provides cases with paired manual segmentations for robustness analysis; Center B (test set) serves as the independent external validation.

## Pipeline

```
CT image + manual ROI mask
        │
        ▼
Lab 1 ── CS/TS block split (Center A = CS, Center B = TS)
        │
        ▼
Lab 2 ── PyRadiomics feature extraction (shape, first-order, texture; binWidth=10 HU)
         Outlier detection (IQR at feature level + hierarchical clustering at patient level)
         ICC robustness assessment (Dataset 2: paired segmentations)
        │
        ▼
Lab 3 ── Feature selection: mRMR · QRA · EBR · Genetic Algorithm · Decision Tree
        │
        ▼
Lab 4 ── TRS/VS stratified split (80/20) — Classifier training & hyperparameter tuning
         Models: Naive Bayes · DT · Logistic Regression · Random Forest · SVM · KNN · MLP
         Threshold optimization (min sensitivity ≥ 95%)
        │
        ▼
Lab 5 ── External evaluation on Center B test set
        │
        ▼
Lab 6 ── Robustness analysis: agreement rate on dual-segmentation cases, DSC stratification
        │
        ▼
Lab 7-8 ── Strict ICC > 0.8 filtering + ensemble learning ("at least one" rule)
```

## Key Results

**Best single model: mRMR + Naive Bayes** — highest agreement rate (≈90%) against inter-operator variability, stable generalization across centers.

**Final system (mRMR + NB, test set):**
| Metric | Score |
|---|---|
| Sensitivity | 81.5% (22/27 cancers correctly identified) |
| Specificity | 93.3% (14/15 diverticular disease correctly identified) |

Domain shift (Center A → Center B) mainly reduced sensitivity; threshold recalibration required for deployment at new centers.

## Technical Highlights

- **Block sampling**: deliberate center-based CS/TS split to test cross-center generalizability and measure batch effect
- **ICC filtering**: features retained only if ICC > 0.7 (baseline) or ICC > 0.8 (improved version) — directly addresses inter-operator variability
- **Multi-algorithm feature selection**: filter (mRMR, QRA, EBR), wrapper (GA with LR + 5-fold CV), embedded (DT Gini importance)
- **Overfitting control**: models with BA gap > 0.15 (TRS vs. VS) or Specificity < 40% discarded
- **Ensemble learning**: "at least one" voting to minimize False Negatives — marginal BA gains at the cost of reduced agreement stability; single NB outperformed ensembles on robustness

## Repository Structure

```
DSH/
├── labs/
│   ├── lab1/             # CS/TS split, descriptive statistics on ROIs
│   ├── lab2/             # Feature extraction (PyRadiomics), outlier detection, ICC
│   ├── lab3/             # Feature selection (mRMR, QRA, EBR, GA, DT)
│   ├── lab4/             # TRS/VS split, classifier training, hyperparameter optimization
│   ├── lab5/             # External test set evaluation (Center B)
│   ├── lab6/             # Robustness: inter-operator agreement, DSC stratification
│   └── lab7-8/           # Improvements: ICC > 0.8, ensemble learning
├── presentations/
│   └── presentazione_DSH.pptx         # Full project presentation (15 slides)
├── reports/
│   ├── labs-summary.docx              # Full written summary of all labs
│   └── presentation-script-ENG.docx   # Oral presentation script (English)
├── DICE_SCORE.ipynb      # Dice Similarity Coefficient computation for segmentation pairs
├── Diagnosi.xlsx         # Reference standard: diagnostic labels per patient
├── TRS_IDs_80.csv        # Training set patient IDs
└── VS_IDs_20.csv         # Validation set patient IDs
```

> **Team:** Natali Sebastiano, Guadagno Chiara, Memoli Chiara, Ejupi Eduard, Di Bitonto Giuseppe
> **Tools:** Python (PyRadiomics, scikit-learn, pandas, matplotlib), Jupyter Notebooks
> **Note:** CT images and manual segmentation masks are not included (clinical data, not redistributable). Feature CSVs extracted from those images are provided.
