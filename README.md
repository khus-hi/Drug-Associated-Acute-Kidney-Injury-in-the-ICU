# Drug-Associated-Acute-Kidney-Injury-in-the-ICU: A KDIGO-Based Phenotyping and Predictive Modeling Study Using MIMIC-IV

This project builds an end-to-end pipeline to study **drug-induced acute kidney injury (AKI)** in critically ill adults using **MIMIC-IV v3.1**. We identify exposures to nephrotoxic drugs (vancomycin, aminoglycosides, NSAIDs, ACE/ARBs), generate KDIGO-based AKI labels, and train machine-learning models to predict **48-hour AKI risk** following drug initiation.

The work includes reproducible BigQuery cohort construction, KDIGO phenotyping, enriched feature engineering, and CatBoost modeling with SHAP interpretation.

---

## 🚀 Project Highlights

- **Cohort of 65k ICU stays** built from MIMIC-IV v3.1  
- **Full KDIGO 2012 AKI phenotype** (SCr + urine output)
- **Drug-level ADE labels** for nephrotoxic medications  
- **Temporal feature engineering** (24h pre-drug window):
  - Rolling labs, vitals, urine output  
  - SOFA & SAPS-II scores  
  - Clinically meaningful interaction features  
- **CatBoost models** for:
  - `ade_aki48_full` — AKI within 48h of drug start  
  - `any_aki_post_drug` — any AKI after exposure  
- **Significant performance gain** with enriched features:
  - ROC-AUC ↑ from ~0.82 → ~0.85  
  - Recall nearly doubled while maintaining precision  
- **Model explainability** using SHAP feature importance  

---


---

## 📚 Overview of the Pipeline

### 1. **Cohort & AKI Phenotyping (BigQuery)**
- Build adult ICU cohort  
- Compute baseline serum creatinine  
- Generate KDIGO SCr + UO AKI labels  
- Merge into full KDIGO AKI table  

### 2. **Drug Exposure Extraction**
- Identify exposure windows for:
  - Vancomycin  
  - Aminoglycosides  
  - NSAIDs  
  - ACE/ARBs  
- Create drug-level ADE tables with AKI outcome labels  

### 3. **Enriched Features**
All features are computed in the **24 hours before drug_start_time**, including:
- Labs (SCr, BUN, K, bicarbonate)  
- Vitals (HR, MAP, SpO₂)  
- Urine output (6h, 12h, 24h windows)  
- SOFA & SAPS-II severity scores  
- Drug duration and interaction flags  

### 4. **Modeling**
- Train CatBoost using:
  - Baseline ADE table  
  - Enriched temporal feature table  
- Evaluate AUC, PR-AUC, F1, precision-recall  
- Interpret model using SHAP  

---

## 📊 Key Results (High-Level)

| Model | ROC-AUC | PR-AUC | Recall @0.50 | Notes |
|-------|---------|--------|----------------|-------|
| **Baseline** | ~0.82 | ~0.15 | ~0.37 | Static features only |
| **Enriched** | ~0.85 | ~0.19 | ~0.77 | Uses temporal + severity features |

**Takeaway:**  
Temporal labs/vitals + SOFA/SAPS-II nearly **double recall** while preserving precision — crucial for early nephrotoxin risk prediction.

---

## 🛠 Requirements

- Access to **MIMIC-IV v3.1** on BigQuery  
- Google Cloud project with BigQuery enabled  
- Python 3.9+  
- Packages:
  - pandas, numpy, scikit-learn  
  - catboost  
  - shap  
  - matplotlib / seaborn  

---

## ▶️ How to Run

### **1. Build BigQuery Tables**
Run the SQL scripts in `/sql/` in order:

1. `cohort_construction.sql`  
2. `aki_kdigo_phenotyping.sql`  
3. `drug_exposure_tables.sql`  
4. `final_ade_model_v2.sql`  
5. `ade_feature_enriched_v1.sql`

### **2. Export Tables**
From BigQuery UI or CLI:

- `final_ade_model_v2`  
- `ade_feature_enriched_v1`  

### **3. Train Models**
Run the notebooks:

- `/notebooks/ade_modeling_baseline.ipynb`  
- `/notebooks/ade_modeling_enriched.ipynb`  

These notebooks perform:

Preprocessing

* Train/validation/test splits

* CatBoost model training

* ROC/PR analysis

* SHAP-based interpretation


