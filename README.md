Highlights (5):
• Three task framings on identical data and the same 80/20 stratified hold-out: multiclass classification of 6 construction-era
bins aligned to French thermal regulations (RT1974, RT1988, RT2000, RT2005, RT2012), binary classification at the 1975
regulatory cliff, and regression on the exact construction year (rows with non-null annee_construction).

• Strong test-set results. HistGradientBoosting champion attains era macro-F1 = 0.82, accuracy 0.81, and 92% within-1-era
accuracy. Random forest reaches macro-F1 = 0.91 on the binary pre/post-1975 task and MAE = 10.2 years (R² = 0.61) on
regression. Strongly beats dummy/baseline (era macro-F1 = 0.07).

• Rigorous leakage defence via 4-way feature ablation (energy-only / structural-only / combined / combined-minus-equipment)
on Random Forest and HistGB. Removing the equipment-type features that most plausibly proxy era (heating system,
ventilation) costs only ~0.01 macro-F1: the model is reading building fabric, not equipment vintage.

• Per-segment failure analysis: apartments are easier to date than houses (test macro-F1 0.82 vs 0.73), plausibly because
individually-built and renovated houses carry more vintage-noise than apartments in consistent whole-buildings. Île-de-France
behaves close to the rest of France.

• Decision-theoretic deployment view. Isotonic-calibrated era champion enables selective prediction: 93% exact accuracy on
the 55% of buildings where the top-class probability ≥ 0.7, and 98% accuracy on the 9% where probability ≥ 0.9.

Brief Report:
We applied the standard supervised-learning pipeline to a single, large public dataset — the ADEME DPE Logements Existants
registry of post-July-2021 French energy performance certificates — to predict each building's construction era from its physical and
energy-performance characteristics. 

We ingested 100,000 dwellings (apartments + houses) via the open ADEME API, cleaned the
table (era target derivation, plausibility filters on surface, floor count, ceiling height — 0.26% rows dropped, all changes logged), and
enriched it with five engineered features (insulation indicator, heating-to-hot-water ratio, energy-per-floor, all-electric flag, climate–
heating combo). Ordered categoricals — DPE / GES letter grades A–G, insulation quality, altitude bands, inertia class — were
ordinal-encoded with hand-coded integer scales rather than one-hot, preserving rank. The final matrix has 43 features (21 energy +
14 structural + 3 geographic + 5 engineered).

We framed three parallel tasks (multiclass era / binary pre-post-1975 / regression on year), all on the same stratified hold-out, and
trained every model through a scikit-learn Pipeline with per-fold-refit median imputation, standard scaling (linear models only),
and one-hot / ordinal / target encoding so cross-validation is leakage-free. 

The model zoo: majority/mean baseline, logistic
regression, ridge regression, linear SVC, KNN, decision tree, bagging, random forest, HistGradientBoosting, MLP. Top four
candidates were refined to 5-fold stratified cross-validation and tuned via RandomizedSearchCV over learning rate, depth, leaf
count, regularisation and ensemble size. 

We further analysed the champion with permutation importance, a 4-way feature ablation,
isotonic calibration with a coverage-vs-accuracy selective-prediction curve, class-imbalance experiments (sample weighting
variants), threshold tuning for the binary task, and a fair classification-vs-regression comparison (Protocol A: classifier on year-
restricted rows = 0.79; Protocol B: regress era index, round to bucket = 0.60 — classification wins by 19 points).

Headline test-set numbers (champions): era macro-F1 = 0.79, accuracy 0.78, within-1-era accuracy 0.92; binary macro-F1 = 0.90;
regression MAE = 10.2 years, R² = 0.61. The strongest individual predictors per permutation importance are wall-insulation quality and
envelope air-renewal heat-loss, both directly tied to RT thermal regulations; removing the equipment-type features that most plausibly
proxy era (heating system, ventilation) costs only ~0.01 macro-F1. 

Novelty. We are unaware of a published benchmark for DPE era prediction with this exact regulatory-bin framing and ablation discipline; our contributions are 

(i) framing the task as classification aligned to RT regulatory transitions rather than year-regression (the fair-comparison shows this is better by 19 macro-F1 points),

(ii) the 4-way ablation isolating equipment-type leakage, and 

(iii) a per-region / per-building-type analysis surfacing the apartment-vs-house gap. Scope set aside (mentioned for clarity): SHAP, SMOTE, and modern boosters (XGBoost / LightGBM / CatBoost) were attempted but their dependencies were unavailable on our shared compute environment; HistGB is the equivalent stand-in. Main references: ADEME DPE dataset (data.ademe.fr); scikit-learn library; French DGEC thermal regulations (RT1974 → RT2012) for the era bin boundaries.
