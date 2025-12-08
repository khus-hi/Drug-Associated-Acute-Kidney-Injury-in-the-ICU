flowchart TD

A[MIMIC-IV v3.1 <br> patients/admissions/labs/ICU] --> B[ICU Cohort <br> cohort_icu_v2]
B --> C[Baseline Creatinine <br> baseline_scr_v2]

C --> D1[SCr Time Series <br> scr_timeseries_v2]
C --> D2[UO Hourly <br> uo_hourly_v2]

D1 --> E1[SCr AKI Events <br> aki_scr_events_v2]
D2 --> E2[UO AKI Events <br> aki_uo_events_v2]

E1 --> F[Full KDIGO AKI <br> aki_full_kdigo_v2]
E2 --> F

F --> G1[Drug Exposures <br> vanco / aminogly / nsaid / ace_arb]
F --> G2[Confounders <br> sepsis / vaso / vent / CK / loop]

G1 --> H[final_ade_model_v2]
G2 --> H

H --> I[Temporal Feature Engineering <br> 24h labs/vitals/UO/SOFA]
I --> J[ade_feature_enriched_v1]

J --> K[Machine Learning <br> CatBoost + SHAP]
