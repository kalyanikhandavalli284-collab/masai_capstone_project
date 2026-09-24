# Analytics Pipeline

## Objective

This module is a single Titanic analytics-to-modeling workflow. The first notebook loads and cleans the dataset and performs the required EDA; the second notebook continues from the saved cleaned CSV and performs classification, imbalance handling, Random Forest tuning, regression, and pipeline persistence.

## Files

- `01_eda.ipynb` — data loading, profiling, cleaning, EDA, correlations, data-story charts, and standardization check.
- `02_modelling.ipynb` — classification, imbalance comparison, tuning, regression, model comparison, and saved pipeline validation.
- `titanic.csv` — offline copy of the initially loaded Titanic dataset.
- `clean_df.csv` — cleaned dataset consumed by the modeling notebook.
- `best_titanic_pipeline.pkl` — saved fitted preprocessing + Random Forest pipeline.

## Data Flow

```text
sns.load_dataset("titanic")
        |
        v
titanic.csv  (offline fallback)
        |
        v
Profiling + missing-value handling
        |
        v
clean_df.csv
        |
        +--> EDA / charts / correlation
        |
        v
Stratified train/test split
        |
        v
ColumnTransformer
  - StandardScaler
  - OneHotEncoder
        |
        +--> Logistic Regression
        +--> Decision Tree
        +--> Random Forest
        |
        +--> class_weight comparison
        +--> SMOTE comparison
        +--> GridSearchCV
        |
        +--> Linear Regression (fare)
        |
        v
best_titanic_pipeline.pkl
```

## EDA Results Captured in the Supplied Notebook

The raw Titanic dataset has **891 rows and 15 columns**.

Measured missing-value percentages include:

| Column | Missing |
|---|---:|
| `age` | 19.8653% |
| `embarked` | 0.2245% |
| `deck` | 77.2166% |

The notebook then:

- drops `class`, `who`, `adult_male`, `embark_town`, `alone`, and `alive`;
- median-imputes `age`;
- drops rows with missing `embarked`;
- drops `deck`.

The resulting cleaned dataset has **889 rows and 8 columns**.

### Outliers

Using the IQR rule:

- `age`: 65 outliers.
- `fare`: 114 outliers.

The EDA notebook reports:

- Fare mean: **32.10**
- Fare median: **14.4542**
- Fare mode: **8.05**

The ordering mean > median > mode is used in the notebook to describe fare as right-skewed.

### Survival Breakdowns

The executed notebook reports survival rates of:

| Breakdown | Reported survival rate |
|---|---:|
| Male | 32.06% |
| Female | 67.94% |
| Pclass 1 | 62.62% |
| Pclass 2 | 47.28% |
| Pclass 3 | 24.24% |

The combined `pclass` + `sex` table is also printed in the notebook.

### Correlation

The required six-column matrix is used:

```text
survived, pclass, age, sibsp, parch, fare
```

The two largest absolute off-diagonal correlations reported are:

- `pclass` ↔ `fare`: approximately **-0.5482**
- `sibsp` ↔ `parch`: approximately **0.4145**

### Standardization Check

The EDA notebook standardizes `age` and `fare` using `StandardScaler`. Its recorded output shows approximately:

```text
Age:  mean 0.00, std 1.00
Fare: mean 0.00, std 1.00
```

## Modeling

The modeling notebook reads `clean_df.csv`, removes the single record with `fare == 512.3292`, and then performs a stratified 80/20 split.

The recorded split is:

- Training: 708 rows
- Test: 178 rows
- Training target: 439 non-survivors / 269 survivors
- Test target: 110 non-survivors / 68 survivors

Preprocessing is implemented with a `ColumnTransformer`:

- numeric: `age`, `fare` → `StandardScaler`
- categorical: `sex`, `embarked` → `OneHotEncoder`
- remaining columns pass through

The transformation is placed inside model pipelines, so fitting occurs on the training split rather than on the complete dataset.

## Classifier Results

The executed notebook reports the following test metrics:

| Metric | Logistic Regression | Decision Tree | Random Forest |
|---|---:|---:|---:|
| Accuracy | 0.7697 | 0.7416 | 0.7978 |
| Precision | 0.7547 | 0.6667 | 0.7857 |
| Recall | 0.5882 | 0.6471 | 0.6471 |
| F1 | 0.6612 | 0.6567 | 0.7097 |

The notebook also produces confusion matrices and a combined ROC plot.

### Important acceptance-criteria gap

The assignment asks for the full metric suite including **AUC** in the side-by-side model comparison table. The notebook calculates AUC for plotting, but its final printed comparison table contains only accuracy, precision, recall, and F1. The README therefore does not claim that the final table fully satisfies the AUC requirement.

## Imbalance Comparison

The notebook reports a survived proportion of **38.04%** and compares:

- baseline Logistic Regression;
- Logistic Regression with `class_weight="balanced"`;
- Logistic Regression with SMOTE.

Test results recorded in the notebook:

| Metric | Baseline | Weighted | SMOTE |
|---|---:|---:|---:|
| Accuracy | 0.7697 | 0.7528 | 0.7640 |
| Precision | 0.7547 | 0.6765 | 0.7097 |
| Recall | 0.5882 | 0.6765 | 0.6471 |
| F1 | 0.6612 | 0.6765 | 0.6769 |

SMOTE is implemented in an imbalanced-learn pipeline after the preprocessing step, so the oversampling is performed within the training pipeline rather than directly on the complete dataset.

## Random Forest Tuning

The grid search varies:

```text
max_depth:    [4, 6, 8, 10, None]
n_estimators: [50, 100, 150]
```

The recorded best parameters are:

```text
max_depth = 10
n_estimators = 50
```

The recorded cross-validation F1 score is approximately **0.7758**.

The final OOB-enabled Random Forest reports an OOB score of approximately **0.8305**.

## Regression Side Task

The regression pipeline predicts `fare` from the other available features (excluding `fare` and `survived`) using linear regression with preprocessing.

Recorded metrics:

| Metric | Value |
|---|---:|
| MAE | 19.6006 |
| RMSE | 30.3000 |
| R² | 0.4336 |
| Adjusted R² | 0.4033 |

The notebook's residual-plot interpretation identifies a widening residual spread as predicted fare increases and describes this as heteroscedasticity.

## Saved Pipeline

The notebook saves the complete preprocessing + Random Forest estimator as:

```text
best_titanic_pipeline.pkl
```

It then reloads the object with `joblib.load()` and predicts on raw, unpreprocessed test rows. This satisfies the important design requirement that the persisted artifact contains preprocessing together with the estimator.

## How to Run

Run the notebooks in order:

```bash
jupyter notebook analytics/01_eda.ipynb
jupyter notebook analytics/02_modelling.ipynb
```

The second notebook expects `clean_df.csv` to exist in the same working directory.

## Current Implementation Notes

The supplied code should be reviewed before final submission against the exact rubric:

1. **AUC is calculated but is not included in the final printed comparison table.**
2. The EDA chart interpretations are present as notebook strings, but some interpretations contain inaccurate statements or wording that should be corrected before submission. For example, a survival-rate chart is described as a percentage split of all survivors rather than the plotted survival rate by sex.
3. The assignment asks for a Decision Tree rendered using `plot_tree`; the supplied modeling notebook does not currently contain a `plot_tree` call.
4. The final modeling notebook does not print a 3–5 sentence written deployment recommendation tied to the final metric values.
5. The assignment asks the GridSearchCV search to cover `max_features` as well as `n_estimators` and `max_depth`; the current grid omits `max_features`.
6. The final tuned-model statistics are generated manually rather than directly taking the estimator returned by `GridSearchCV`; the notebook hard-codes `max_depth=10` and `n_estimators=50` after observing the search result.
7. The EDA stage's missing-value handling does not explicitly apply the percentage threshold rationale in written form for each affected column, although the measured percentages are printed.
8. The notebook removes the £512.3292 fare observation during modeling after EDA. This is a documented code choice, but the final README should keep that decision visible because it changes the modeling dataset.

## Design Summary

The implementation is intentionally split into two ordered notebooks: EDA creates the reusable cleaned CSV, and modeling consumes that same cleaned file. Pipelines and a `ColumnTransformer` are used to keep train/test preprocessing separated and to make the saved model usable on raw feature rows.
