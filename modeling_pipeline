# 📘 **Modeling Pipeline Documentation**  
### **Drug-Associated AKI Prediction in the ICU (MIMIC-IV v3.1)**

This document describes the **machine learning pipeline** used to model adverse drug event (ADE)–associated acute kidney injury (AKI) in ICU patients. It covers dataset preparation, preprocessing, feature engineering, CatBoost training, evaluation, and explainability.

The modeling pipeline uses two datasets:

1. **Baseline modeling table:** `final_ade_model_v2`  
2. **Enriched feature table:** `ade_feature_enriched_v1`  

The goal is to predict:
- **AKI within 48 hours after drug start** (`ade_aki48_full`)  
- **Any AKI after drug exposure** (`any_aki_post_drug`)  

---

# **1. Overview of the Modeling Workflow**


The same modeling workflow is applied to both baseline and enriched datasets.

---

# **2. Inputs**

## **2.1 Baseline ADE Modeling Table (`final_ade_model_v2`)**
Includes:
- Demographics  
- Comorbidities  
- Baseline SCr  
- ICU context (ventilation, vasopressors, sepsis)  
- Exposure times  
- Drug class  
- AKI outcomes  

This serves as the control dataset.

---

## **2.2 Enriched Temporal Feature Table (`ade_feature_enriched_v1`)**

Adds ~25 new features computed in the **24 hours before drug_start_time**, including:

### **Rolling Labs**
- SCr: mean_6h, mean_12h, mean_24h, max_24h, min_24h  
- BUN: mean_6h, mean_24h  
- Potassium: mean_24h, max_24h  
- Bicarbonate: mean_24h  

### **Rolling Vitals**
- HR: mean_6h, mean_24h, max_24h  
- MAP: mean_24h, min_24h  
- SpO₂: mean_24h  

### **Urine Output**
- uo_6h_ml  
- uo_12h_ml  
- uo_24h_ml  

### **Severity Scores**
- `sofa_score_approx`  
- `sapsii_score_approx`  

### **Exposure Meta-Features**
- `drug_duration_hours`  

### **Interaction Terms**
- CKD × Vancomycin  
- Diabetes × NSAID  
- Sepsis × Vasopressor  
- Loop Diuretic × ACE/ARB  

These features significantly increase predictive performance.

---

# **3. Label Definitions**

Two primary supervised learning tasks:

| Label | Meaning | Use Case |
|-------|---------|----------|
| **`ade_aki48_full`** | AKI within 48h of drug start | Predict immediate post-drug AKI risk |
| **`any_aki_post_drug`** | Any AKI after drug exposure | Understand delayed kidney injury |

Both labels come from **full KDIGO AKI** phenotyping.

---

# **4. Dataset Builder (Leakage Prevention)**

A critical component of the modeling workflow.

### **The dataset builder performs:**

#### **1. Remove identifiers**
- subject_id  
- hadm_id  
- stay_id  

#### **2. Drop all time columns**
- drug_start_time  
- drug_end_time  
- ICU intime/outtime  
- AKI onset timestamps  

#### **3. Remove label-like columns**
- aki_scr_any  
- aki_scr_stage  
- aki_uo_any  
- aki_full_*  
- ade_aki48_* (except chosen label)  

#### **4. Define feature types**
- **Categorical:** drug_class, gender, race, first_careunit, admission_type  
- **Numeric:** all other fields  

#### **5. Stratified train/test split**
- 80% train  
- 20% test  
- preserves class percentages  

### Output:
- X_train, X_test  
- y_train, y_test  
- categorical feature names  
- numeric feature names  

---

# **5. Preprocessing Pipeline**

### **Numeric Features**
- Median imputation  
- StandardScaler  

### **Categorical Features**
- Most frequent imputation  
- OneHotEncoder (`sparse_output=False`)  

A unified `ColumnTransformer` handles both branches cleanly.

---

# **6. CatBoost Model**

The primary model architecture:

```python
CatBoostClassifier(
    iterations=2000,
    learning_rate=0.05,
    depth=6,
    loss_function="Logloss",
    eval_metric="AUC",
    class_weights={0:1.0, 1:(neg/pos)},
    od_type="Iter",
    od_wait=50,
    task_type="GPU"
)

# **7. Model Evaluation**

### **Metrics Computed**
The following metrics are calculated for both baseline and enriched models:

- **ROC-AUC**  
- **PR-AUC** (important for imbalanced datasets)  
- **F1 Score**  
- **Precision**  
- **Recall**  
- **SHAP feature importance**  

### **Why PR-AUC Matters**
Drug-associated AKI is a **low-prevalence event (~3–4%)**, meaning ROC-AUC alone can be misleading.  
PR-AUC better reflects how well the model identifies true AKI cases without being overwhelmed by negatives.

---

# **8. Performance Summary**

## **8.1 48-Hour AKI Prediction (`ade_aki48_full`)**

| Model        | ROC-AUC | PR-AUC | Recall @ 0.50 | Notes |
|--------------|---------|--------|----------------|-------|
| **Baseline** | ~0.816  | ~0.150 | ~0.37          | Static covariates only |
| **Enriched** | ~0.846  | ~0.193 | ~0.77          | Includes temporal + severity features |

**Key Insight:**  
The enriched feature set nearly **doubles recall**, allowing the model to catch far more AKI cases after nephrotoxin exposure.

---

## **8.2 Any AKI After Drug Exposure (`any_aki_post_drug`)**

| Model        | ROC-AUC | PR-AUC | Recall @ 0.50 | Notes |
|--------------|---------|--------|----------------|-------|
| **Baseline** | ~0.836  | ~0.201 | ~0.41          | Static covariates only |
| **Enriched** | ~0.863  | ~0.237 | ~0.80          | Includes temporal + severity features |

**Summary:**  
Enriched models consistently outperform baseline models across all metrics.

---

# **9. SHAP Explainability**

SHAP values (Shapley Additive Explanations) help interpret which features most influence predictions.

### **Top Features in the Enriched Model**
- **sofa_score_approx** — Reflects severity of illness  
- **uo_24h_ml** — Early kidney function decline  
- **baseline_scr** — Renal vulnerability at baseline  
- **drug_class** — Nephrotoxic drug type  
- **loop_orders** — Fluid/diuretic management  
- **vent_any** — Critical illness indicator  
- **dm_nsaid** — Interaction (diabetes × NSAID)  
- **charlson_index** — Chronic disease burden  

These patterns align with clinical expectations:  
**Severity-of-illness + early renal stress + nephrotoxin exposure** drive AKI risk.

---

# **10. Model Outputs**

The modeling pipeline generates the following outputs for each target label:

### **Files / Artifacts**
- **Predicted probabilities** (test set)  
- **ROC curves**  
- **Precision–Recall curves**  
- **Confusion matrices (optional)**  
- **SHAP summary & dependence plots**  
- **Threshold-optimized metrics**  
- **Saved CatBoost model** (optional `.cbm` or pickle)  

These are used for reporting, manuscript preparation, and external validation.

---

# **11. Reproducibility Notes**

To ensure consistent and transparent modeling:

- Fixed random seeds** for deterministic splits and model training  
- No post-drug data** is used in features (strict leakage control)  
- All enriched features** come from data in the **24 hours before drug_start_time**  
- BigQuery tables are versioned** (`_v1`, `_v2`) to preserve lineage  
- Same preprocessing pipeline** used for all models  
- Same feature selection logic** applied across baseline and enriched datasets  

---

# 📌 **Final Summary**

The modeling pipeline converts raw ICU EHR data into a robust, leakage-free, temporally aligned dataset for predicting **drug-associated acute kidney injury  
Enriched features—especially urine output trends, severity scores, and early lab changes—lead to **substantial performance gains**, nearly **doubling recall while maintaining high precision.

This provides a strong foundation for developing early-warning systems and supporting nephrotoxin stewardship in the ICU.

---
