```mermaid
flowchart TD

A[MIMIC-IV v3.1 Tables<br>patients / admissions / labevents / chartevents / outputevents / prescriptions / derived]
    --> B[1. ICU Cohort<br>cohort_icu_v2]

B --> C[2. Baseline SCr<br>baseline_scr_v2]

C --> D[ KDIGO AKI Phenotyping ]

D --> E1[scr_timeseries_v2]
D --> E2[uo_hourly_v2]
D --> E3[weight_durations]

E1 --> F1[aki_scr_events_v2]
E2 --> F2[aki_uo_events_v2]

F1 --> G1[aki_labels_v2]
F2 --> G2[aki_uo_labels_v2]

G1 --> H[aki_full_kdigo_v2]
G2 --> H

H --> I1[vanco_exposure_v2]
H --> I2[aminogly_exposure_v2]
H --> I3[nsaid_exposure_v2]
H --> I4[ace_arb_exposure_v2]
H --> I5[loop_exposure_v2]
H --> I6[confounders_v2]

I1 --> J[final_ade_model_v2]
I2 --> J
I3 --> J
I4 --> J
I5 --> J
I6 --> J

J --> K[ade_feature_enriched_v1<br>(24h labs/vitals/UO/SOFA/SAPS-II)]
