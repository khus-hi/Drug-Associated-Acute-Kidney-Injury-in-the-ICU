# 📊 **Bigquery Pipeline**

The flowchart above illustrates the **end-to-end BigQuery data pipeline** used to construct the analytic datasets for the Drug-Associated AKI study. The pipeline moves from raw MIMIC-IV data to fully engineered features used in machine-learning models.

The diagram flows **top to bottom**, showing how each dataset feeds into the next.

---

## **1. MIMIC-IV v3.1 Data Sources**
The pipeline begins with core hospital, ICU, and derived clinical tables from MIMIC-IV v3.1, including:
- patient demographics  
- hospital admissions  
- laboratory values  
- vitals and outputs  
- prescription records  
- derived severity scores (SOFA, SAPS-II, sepsis, ventilation, vasoactive agents)

These are the **raw EHR inputs** for all downstream processing.

---

## **2. ICU Cohort Construction**
A clean cohort is created by:
- selecting adult patients (age ≥18)  
- keeping the **first ICU stay** per patient  
- joining demographic and admission info  

This forms the **base population** from which AKI and ADE events are derived.

---

## **3. Baseline Serum Creatinine**
For each patient, baseline kidney function is computed using:
1. the lowest SCr within 7 days before ICU admission, or  
2. the first SCr during hospitalization  

If none exist, the value is marked as missing. Baseline SCr is required for KDIGO AKI classification.

---

## **4. KDIGO AKI Phenotyping**

### **4.1 SCr-Based AKI**
- `scr_timeseries_v2`: extracts all creatinine measurements  
- `aki_scr_events_v2`: applies KDIGO criteria  
  (≥0.3 mg/dL rise in 48h, ≥1.5× baseline, ≥2× baseline, etc.)  
- `aki_labels_v2`: produces serum creatinine–based AKI stage + onset  

### **4.2 Urine Output AKI**
- `uo_hourly_v2`: aggregates hourly urine outputs  
- `weight_durations`: provides patient weight for mL/kg/hr calculations  
- `aki_uo_events_v2`: detects oliguria/anuria per KDIGO thresholds  
- `aki_uo_labels_v2`: produces UO-based AKI stage + onset  

### **4.3 Full KDIGO AKI **
Serum creatinine and urine output labels are merged to generate:
- unified AKI stage  
- unified AKI onset  
- indicators for SCr-only, UO-only, or combined AKI  

This dataset represents the **final AKI phenotype**.

---

## **5. Drug Exposure Extraction**
Exposure windows are identified for:
- Vancomycin  
- Aminoglycosides  
- NSAIDs  
- ACE inhibitors / ARBs  
- Loop diuretics  

Each exposure includes:
- start and end timestamps  
- mapping to the corresponding ICU stay  

These drug episodes form the basis of the AKI-onset relative timing (ADE definitions).

---

## **6. Confounders**
Clinical conditions that modify AKI risk are extracted:
- sepsis (Sepsis-3)  
- mechanical ventilation  
- vasopressor use  
- rhabdomyolysis (CK ≥5000)  
- loop diuretic therapy  

These ensure the modeling dataset properly adjusts for illness severity and competing risks.

---

## **7. ADE Modeling Table**
This is the main dataset for analysis and baseline modeling.  
Each row represents a **drug exposure episode** with:
- demographics and comorbidities  
- baseline SCr  
- confounders  
- full KDIGO AKI outcomes  
- 48-hour drug-associated AKI labels (`ade_aki48_full`)  

This table is used for:
- descriptive statistics  
- baseline ML models  
- exposure-level analysis  

---

## **8. Enriched Feature Engineering**
A secondary modeling table adds **24-hour pre-drug temporal features**, including:
- rolling labs (SCr, BUN, K, bicarbonate)  
- rolling vitals (HR, MAP, SpO₂)  
- urine output totals (6h, 12h, 24h)  
- SOFA and SAPS-II severity scores  
- drug exposure duration  
- interaction terms (e.g., CKD × vancomycin)  

These features significantly improve predictive performance, especially recall.

---

## **9. Machine Learning Models**
The enriched dataset feeds into CatBoost-based ML models predicting:
- **AKI within 48 hours** of drug initiation  
- **Any AKI** following drug exposure  

Models are interpreted using SHAP to identify key risk factors.

---

## 📌 **Summary**
This pipeline converts raw MIMIC-IV EHR data into a structured, validated, and feature-rich dataset for studying **drug-associated acute kidney injury** in ICU patients. Each block in the diagram corresponds to a reproducible BigQuery table, forming a clear, auditable data lineage from raw EHR inputs to final ML predictions.

---



```mermaid
flowchart TD

A["MIMIC-IV v3.1 Tables: patients, admissions, labevents, chartevents, outputevents, prescriptions, derived"]
    --> B["ICU Cohort (cohort_icu_v2)"]

B --> C["Baseline SCr (baseline_scr_v2)"]

C --> D["KDIGO AKI Phenotyping"]

D --> E1["scr_timeseries_v2"]
D --> E2["uo_hourly_v2"]
D --> E3["weight_durations"]

E1 --> F1["aki_scr_events_v2"]
E2 --> F2["aki_uo_events_v2"]

F1 --> G1["aki_labels_v2"]
F2 --> G2["aki_uo_labels_v2"]

G1 --> H["aki_full_kdigo_v2"]
G2 --> H

H --> I1["vanco_exposure_v2"]
H --> I2["aminogly_exposure_v2"]
H --> I3["nsaid_exposure_v2"]
H --> I4["ace_arb_exposure_v2"]
H --> I5["loop_exposure_v2"]
H --> I6["confounders_v2"]

I1 --> J["final_ade_model_v2"]
I2 --> J
I3 --> J
I4 --> J
I5 --> J
I6 --> J

J --> K["ade_feature_enriched_v1 (24h labs, vitals, UO, SOFA, SAPS-II)"]
