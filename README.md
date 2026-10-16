# Stroke Risk Prediction with Supervised Machine Learning

Supervised machine learning project for **stroke-risk prediction** developed in **KNIME Analytics Platform**, with a strong focus on **imbalanced classification**, **cross-validation**, and **sensitivity-driven threshold optimization**.

The project compares several classification algorithms and evaluates how decision-threshold selection can improve the identification of stroke cases while controlling the number of false positives.

---

## Objective

Stroke prediction is a strongly imbalanced classification problem: stroke cases represent only a small fraction of the available observations.

The objective of the project is therefore not simply to maximize overall accuracy, but to build a classifier that is able to identify as many positive stroke cases as possible.

For this reason, particular attention is given to:

- class imbalance
- sensitivity / recall
- false negatives
- probability-based classification
- decision-threshold optimization
- unbiased final evaluation on an independent test set

---

## Dataset

The project uses the **Stroke Prediction Dataset**, containing demographic, clinical, and behavioral information about patients.

The provided dataset contains **5,110 observations and 12 attributes**.

The target variable is:

```text
stroke
```

where:

- `1` = patient experienced a stroke
- `0` = no stroke

The original dataset is highly imbalanced, with approximately **5% positive stroke cases**.

### Main Variables

- `gender`
- `age`
- `hypertension`
- `heart_disease`
- `ever_married`
- `work_type`
- `Residence_type`
- `avg_glucose_level`
- `bmi`
- `smoking_status`
- `stroke`

The `id` variable is treated as a non-informative identifier and removed during preprocessing.

---

## Workflow

The complete analysis was developed as a **KNIME workflow**.

The pipeline is organized into the following main stages:

1. Data import
2. Preprocessing
3. Missing-value imputation
4. Categorical encoding
5. Dataset partitioning
6. Class-imbalance handling
7. Model training with cross-validation
8. Model comparison
9. Validation and dynamic threshold search
10. Final evaluation on the hold-out test set

---

## Preprocessing

The preprocessing pipeline includes:

- removal of the single observation with `gender = Other`
- binary encoding of categorical variables
- one-hot encoding of multi-level categorical variables
- removal of non-informative attributes
- analysis of redundant features
- stratified data partitioning

The dataset is divided into:

- **60% Training**
- **30% Validation**
- **10% Test**

A fixed random seed is used to make the partitioning procedure reproducible.

---

## Missing Data

Missingness is concentrated in the `bmi` variable.

Instead of removing incomplete observations, BMI values are imputed using **Multiple Imputation by Chained Equations (MICE)** through an R Snippet inside the KNIME workflow.

The imputation uses **Predictive Mean Matching (PMM)** with:

- 5 imputations
- 10 iterations

The target variable `stroke` is explicitly excluded from the imputation predictors to avoid information leakage.

---

## Class Imbalance

The original dataset contains approximately **95% non-stroke and 5% stroke observations**.

The training strategy combines two approaches:

### Initial Undersampling

The majority class is reduced before training, bringing the class distribution to approximately:

```text
90% No Stroke
10% Stroke
```

### SMOTE inside Cross-Validation

During model training, **SMOTE** is applied inside the cross-validation procedure through a Weka `FilteredClassifier`.

Configuration:

- Nearest neighbors: `K = 5`
- Oversampling percentage: `100%`

Applying oversampling only inside the training folds prevents information leakage into validation observations.

---

## Competitive Models

Seven supervised classification algorithms are compared:

- Logistic Regression
- Naive Bayes
- Decision Tree 
- Random Forest
- AdaBoost
- LogitBoost
- Multilayer Perceptron 

Models are evaluated through **10-fold cross-validation**.

Random Forest is configured with **500 trees**, while the remaining models use baseline Weka configurations.

---

## Cross-Validation Results

| Model | Recall | Precision | F-Measure | AUC |
|---|---:|---:|---:|---:|
| Logistic Regression | 0.396 | 0.434 | 0.414 | **0.852** |
| LogitBoost | 0.255 | **0.521** | 0.342 | 0.848 |
| Random Forest | 0.215 | 0.427 | 0.286 | 0.825 |
| Naive Bayes | **0.631** | 0.325 | **0.429** | 0.823 |
| AdaBoost | 0.315 | 0.461 | 0.375 | 0.802 |
| J48 | 0.295 | 0.411 | 0.344 | 0.801 |
| Multilayer Perceptron | 0.376 | 0.392 | 0.384 | 0.769 |

Naive Bayes achieves the highest recall during cross-validation, while **Logistic Regression obtains the highest AUC** and provides a strong balance between discrimination, stability, and interpretability.

---

## Dynamic Threshold Optimization

Because the cost of a false negative is particularly important in stroke-risk prediction, the default probability threshold of `0.50` is not used automatically.

Instead, the KNIME validation workflow performs a **grid search over classification thresholds**.

For each threshold, the workflow:

1. extracts `P(stroke = Yes)`
2. applies the candidate threshold
3. generates binary predictions
4. computes the confusion matrix
5. collects Recall, Precision, and related performance measures

The threshold search is driven primarily by minimum Recall constraints:

- Recall ≥ 0.80
- Recall ≥ 0.85

This intentionally prioritizes sensitivity over specificity.

---

## Model Selection

Among the configurations satisfying the high-recall constraint, Logistic Regression provides the best trade-off between sensitivity and false positives.

On the validation set, one of the strongest configurations is:

```text
Model: Logistic Regression
Threshold: 0.14
Recall: 0.853
```

Logistic Regression is selected as the final model because of its:

- strong AUC
- stable performance
- high achievable sensitivity
- simplicity
- interpretability

---

## Final Test Results

The selected Logistic Regression model is evaluated on the independent **10% hold-out test set**.

Two operating thresholds are compared.

### Threshold = 0.14

| | Predicted Stroke | Predicted No Stroke |
|---|---:|---:|
| Actual Stroke | 22 | 3 |
| Actual No Stroke | 148 | 338 |

### Threshold = 0.17

| | Predicted Stroke | Predicted No Stroke |
|---|---:|---:|
| Actual Stroke | 22 | 3 |
| Actual No Stroke | 131 | 355 |

Increasing the threshold from **0.14 to 0.17** preserves the same number of correctly detected stroke cases while reducing false positives from **148 to 131**.

The final operating point is therefore:

```text
Threshold = 0.17
Recall = 0.88
```

This corresponds to correctly identifying **22 of the 25 stroke cases** in the independent test set.

---

## Key Takeaways

The project highlights several important aspects of machine learning on imbalanced healthcare data:

- overall accuracy alone can be misleading
- resampling must be performed without contaminating validation data
- probability thresholds should reflect the cost of classification errors
- a simpler model can outperform more complex alternatives on the most relevant metric
- sensitivity can be improved substantially by selecting an appropriate operating threshold

The final model prioritizes the reduction of **false negatives**, accepting additional false positives in exchange for higher stroke-case detection.

---

## Repository Structure

```text
stroke-risk-prediction/
├── data/
│   └── healthcare-dataset-stroke-data.csv
├── workflow/
│   └── Team30_Stroke_Risk_Prediction.knwf
├── docs/
│   └── Stroke_Risk_Prediction_Through_Supervised_Machine_Learning_Models-TEAM30.pdf
└── README.md
```

---

## Reproducibility

The KNIME workflow was developed using KNIME Analytics Platform with Weka 3.6 legacy nodes and an R Snippet for MICE imputation.

When importing the workflow on another machine, the **CSV Reader** may need to be pointed to:

```text
data/healthcare-dataset-stroke-data.csv
```

The original analysis logic, trained nodes, validation pipeline, threshold search, and final evaluation are preserved in the workflow.

---

## Report

The complete project report is available here:

[Read the full report](docs/Stroke_Risk_Prediction_Through_Supervised_Machine_Learning_Models-TEAM30.pdf)

---

## Authors

- Filippo Avon
- Irida Pajenga
- Carmelo Ferraro
- Lorenzo Luciano Gulizia

Master's Degree in Data Science  
University of Milano-Bicocca  
Academic Year 2025/2026
