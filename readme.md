# Late Refill Risk Prediction  
## Data-AI-Hackathon-Task1-ChallengeA

### Team Name
**DataVision Innovators**

### Team Members

| Name | Email ID |
|------|----------|
| Nouran Khallaf | N.Khallaf@leeds.ac.uk |
| Rushikesh Kothawade | rrtq0904@leeds.ac.uk |
| Baishali Patra | pbcb0449@leeds.ac.uk |

### Team Description
DataVision Innovators is a collaborative team of three members who worked together with strong coordination, ownership, and shared commitment. Nouran Khallaf led the team with clarity and initiative, taking responsibility for guiding the project and supporting all members throughout the process. Rushikesh Kothawade and Baishali Patra contributed actively as team members, engaging in data analysis, implementation, and collaborative problem-solving to achieve the project objectives effectively.

### Project Overview
This repository contains the implementation for **Challenge A – Late Refill Risk Prediction**, developed as part of the **Data & AI Hackathon**. The goal of this challenge is to analyse patient medication refill behaviour and build a predictive model that identifies individuals who may be at risk of late refills. Early identification of such patterns can support better patient adherence and enable proactive healthcare interventions.

### Report Description: `late_refill_risk.ipynb`
The `late_refill_risk.ipynb` notebook provides a structured, end-to-end workflow for analysing medication refill data and predicting late refill behaviour. It includes data preprocessing, exploratory data analysis, feature engineering, and model development. The notebook is organised to support smooth experimentation and evaluation within Google Colab, enabling clear visualisation of trends and performance metrics. The overall aim is to build a reliable model that can help healthcare providers identify at-risk patients and improve medication adherence outcomes.

### Data and Outputs
The repository also includes the project data and generated outputs:

- **Processed data** contains the **ready-to-use datasets with engineered features**, prepared for direct modelling and evaluation.
- **Output figures and results** include generated visualisations, model outputs, and performance summaries from the experiments.

### Repository Structure

```text
├── late_refill_risk.ipynb     # Main notebook for analysis and modelling
├── README.md                  # Project documentation
└── data/                      # Project datasets
    └── processed/             # Ready-to-use processed data with features
└── outputs/                   # Output figures, model results, and experiment artefacts
```


### Validation results

We evaluated the late-refill model using a **time-based split with patient holdout**, which is the appropriate setup for this task because it reduces temporal leakage and ensures that the same patient does not appear in both training and validation.

The validation split date was **2009-07-30**, with **72,890 validation rows** and a **late-refill prevalence of 0.7746**.

Core validation metrics were:

- **PR-AUC:** 0.8430
- **AUROC:** 0.6357
- **Brier score:** 0.3841

The model output is the **predicted probability of late refill**, and the displayed risk score is that probability multiplied by 100.

During feature alignment, the model expected **115 raw columns**. To match the trained schema, **85 missing columns were added as NaN** and **1 extra column was dropped**.

---

## Interpretation of the results

These results suggest a model that is **more useful for ranking high-risk refill events than for clean binary separation**.

This pattern is visible in the metrics:

- **PR-AUC is strong (0.8253)**
- **AUROC is modest (0.5950)**

Because late refill is very common in the validation set, **PR-AUC is the more informative metric** in this setting. In practical terms, the model is better at concentrating late-refill cases toward the higher-risk end than at sharply distinguishing every late case from every non-late case.

A second important finding comes from the calibration and decile analyses: the model appears to be **systematically underpredicting absolute risk**. Across deciles, the observed late-refill rate remains high, while the mean predicted risk is lower, especially in the lower and middle deciles.

This means the model has **useful ranking signal**, but its probabilities should **not yet be treated as well-calibrated clinical or operational risk estimates**.

---

## What the plots show

The **ROC curve** is consistent with the AUROC of about **0.595**, indicating that discrimination is present but limited.

The **precision-recall curve** is much stronger, with **AP = 0.8253**, which is expected in this high-prevalence setting and suggests that the model can still be useful for prioritisation.

The **risk-distribution plot** shows substantial overlap between the predicted scores for the observed on-time and late-refill classes. That overlap explains the modest ROC performance.

At the same time, the **predicted-vs-observed decile chart** shows that higher deciles generally correspond to higher observed late-refill rates, meaning the model is still capturing some useful ordering of risk.

The **lift plot** looks weak, with only a small increase in late-refill rate from lower to higher predicted-risk deciles. This indicates that the current model does not yet create a sharply separated top-risk segment. In a deployment setting, that would limit its usefulness if the goal were to target a small subset of patients for intervention.

---

## Feature interpretation

The SHAP plots suggest that the model relies most on **refill-sequence** and **time-position** features, including:

- `num__chain_length`
- `num__patient_fill_index`
- `num__service_year`
- `num__fraction_of_chain_elapsed`
- `num__DAYS_SUPLY_NUM`

The local SHAP waterfall plots show that **short days supply**, **higher prior patient payment**, **greater fraction of chain elapsed**, and **prior gap features** can push an individual prediction upward.

Globally, `num__chain_length` is the strongest named driver, followed by `num__patient_fill_index` and `num__service_year`.

One caution is that the SHAP summary also shows a very large combined contribution from the **“other features”** bucket. This means that the model signal is still spread across many variables rather than being dominated by a small stable set of predictors. That makes interpretation harder and may reflect a noisy, high-dimensional feature space.

---

