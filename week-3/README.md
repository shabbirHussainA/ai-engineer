# Machine Learning Fundamentals — Customer Churn Classification

A hands-on supervised machine learning experiment comparing three classification models on the same customer churn dataset.

This project is part of my transition from production Full Stack/MERN engineering toward AI Engineering.

After building Python foundations and working with data using NumPy/Pandas, Week 3 focused on understanding the machine learning workflow itself:

**data → features/target → preprocessing → training → inference → thresholding → evaluation → model comparison**

The objective was not to maximize a leaderboard metric. It was to understand how different supervised learning models behave under the same data and preprocessing conditions.

---

## Problem

Customer churn prediction is a binary classification problem:

> Given information about a customer, can a model estimate whether that customer is likely to churn?

For this experiment, the target variable is:

```text
churn = 0 → customer does not churn
churn = 1 → customer churns
```

The experiment uses five input features:

| Feature | Type |
|---|---|
| `age` | Numerical |
| `monthly_spend` | Numerical |
| `support_calls` | Numerical |
| `tenure_months` | Numerical |
| `contract_type` | Categorical |

`customer_id` is generated in the dataset but is intentionally excluded from the model features.

---

## Dataset

The notebook generates a synthetic dataset containing **1,000 customer records**.

The generated features include:

```text
customer_id
age
monthly_spend
support_calls
tenure_months
contract_type
churn
```

Three contract types are generated:

```text
Monthly
Yearly
Two-year
```

The churn target is generated probabilistically rather than assigned randomly without structure.

Higher churn probability is associated with conditions including:

- Monthly contracts
- More than five support calls
- Tenure below twelve months
- Monthly spending above 200

Randomness is still included so that these conditions do not deterministically define the target.

The generated target distribution is:

```text
Non-churn: 700
Churn:     300
```

This creates a moderately imbalanced binary classification problem.

---

## Missing Data

Missing values are intentionally introduced into:

```text
age
monthly_spend
contract_type
```

This makes preprocessing part of the training workflow rather than assuming perfectly clean input data.

Numerical missing values are handled using:

```python
SimpleImputer(strategy="median")
```

Categorical missing values are handled using:

```python
SimpleImputer(strategy="most_frequent")
```

---

## Machine Learning Pipeline

The experiment follows this workflow:

```text
Synthetic Customer Data
        │
        ▼
Feature / Target Separation
        │
        ▼
Stratified Train/Test Split
        │
        ▼
ColumnTransformer
        │
        ├──────── Numerical Features
        │              │
        │              ├── Median Imputation
        │              └── StandardScaler
        │
        └──────── Categorical Features
                       │
                       ├── Most-Frequent Imputation
                       └── One-Hot Encoding
        │
        ▼
Three Model Pipelines
        │
        ├── Logistic Regression
        ├── Decision Tree
        └── Random Forest
        │
        ▼
predict_proba()
        │
        ▼
Decision Threshold = 0.30
        │
        ▼
Predictions
        │
        ▼
Evaluation
        │
        ├── Confusion Matrix
        ├── Accuracy
        ├── Precision
        ├── Recall
        └── F1 Score
```

---

## Train/Test Split

The dataset is divided into training and test sets using:

```python
train_test_split(
    X,
    y,
    stratify=y,
    test_size=0.2,
    random_state=42
)
```

The test set therefore contains **200 observations**.

`stratify=y` is used to preserve the churn class distribution across the split.

A fixed `random_state` makes the split reproducible.

---

## Preprocessing

Numerical and categorical features require different preprocessing strategies.

### Numerical Pipeline

The numerical features are:

```python
[
    "age",
    "monthly_spend",
    "support_calls",
    "tenure_months"
]
```

They pass through:

```text
Missing Value
      ↓
Median Imputation
      ↓
StandardScaler
```

Implemented using:

```python
numerical_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="median")),
    ("scaler", StandardScaler())
])
```

### Categorical Pipeline

The categorical feature is:

```text
contract_type
```

It passes through:

```text
Missing Value
      ↓
Most-Frequent Imputation
      ↓
One-Hot Encoding
```

Implemented using:

```python
categorical_pipeline = Pipeline([
    ("imputer", SimpleImputer(strategy="most_frequent")),
    ("onehot", OneHotEncoder())
])
```

### Combined Preprocessor

Both pipelines are combined using a `ColumnTransformer`.

This allows the same preprocessing workflow to be attached directly to each model pipeline.

---

## Models Compared

Three supervised classification algorithms are trained on the same dataset.

### 1. Logistic Regression

```python
LogisticRegression()
```

This provides a linear classification baseline.

---

### 2. Decision Tree

```python
DecisionTreeClassifier(
    max_depth=5,
    min_samples_split=20,
    class_weight="balanced",
    random_state=42
)
```

The experiment constrains tree depth and minimum split size rather than allowing an unrestricted decision tree.

`class_weight="balanced"` is used because the generated target contains more non-churn than churn observations.

---

### 3. Random Forest

```python
RandomForestClassifier(
    max_depth=5,
    min_samples_split=20,
    class_weight="balanced",
    random_state=42
)
```

This provides an ensemble-tree comparison against both logistic regression and the individual decision tree.

---

## Why Use Pipelines?

Each model is wrapped together with the same preprocessing stage:

```text
Raw Features
     ↓
Preprocessor
     ↓
Model
```

For example:

```python
logistic_pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("model", LogisticRegression())
])
```

The same pattern is used for the Decision Tree and Random Forest models.

This keeps preprocessing and model execution inside a single training/inference workflow rather than manually transforming the dataset separately for every model.

---

## Probability-Based Predictions

Instead of using only:

```python
model.predict()
```

the notebook obtains churn probabilities using:

```python
model.predict_proba(X_test)[:, 1]
```

for each classifier.

A custom decision threshold is then applied:

```python
threshold = 0.3
```

with:

```python
prediction = (probability >= threshold).astype(int)
```

This means a customer is classified as churn when the estimated churn probability is at least **30%**.

The experiment therefore separates:

```text
Model Probability
        ↓
Decision Threshold
        ↓
Final Classification
```

rather than treating the model's default class prediction as the only possible decision rule.

---

## Evaluation

The models are evaluated using:

- Confusion matrix
- Accuracy
- Precision
- Recall
- F1 score

A reusable evaluation function calculates the metrics for each model:

```python
def evaluate_model(name, y_test, y_pred):
    cm = confusion_matrix(y_test, y_pred)

    print(cm)
    print(classification_report(y_test, y_pred))

    return {
        "Model": name,
        "Accuracy": accuracy_score(y_test, y_pred),
        "Precision": precision_score(y_test, y_pred),
        "Recall": recall_score(y_test, y_pred),
        "F1 Score": f1_score(y_test, y_pred)
    }
```

---

## Results

Using the **0.30 classification threshold**, the notebook produced:

| Model | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|
| Logistic Regression | 0.625 | 0.412 | 0.583 | 0.483 |
| Decision Tree | 0.540 | 0.379 | 0.833 | 0.521 |
| Random Forest | 0.505 | 0.371 | **0.933** | **0.531** |

These results demonstrate why selecting a model using only accuracy can be misleading.

### Logistic Regression

Produced the highest overall accuracy:

```text
Accuracy: 62.5%
Recall:   58.3%
F1:       48.3%
```

Its confusion matrix was:

```text
[[90 50]
 [25 35]]
```

It generated fewer false positives than the tree-based models, but missed more actual churn cases.

---

### Decision Tree

Produced:

```text
Accuracy: 54.0%
Recall:   83.3%
F1:       52.1%
```

Confusion matrix:

```text
[[58 82]
 [10 50]]
```

Compared with Logistic Regression, the model identified substantially more churn cases, but at the cost of more false positives.

---

### Random Forest

Produced:

```text
Accuracy: 50.5%
Recall:   93.3%
F1:       53.1%
```

Confusion matrix:

```text
[[45 95]
 [ 4 56]]
```

The Random Forest identified **56 of the 60 churn cases** in the test set, giving it the highest churn recall of the three models.

However, it also classified **95 non-churn customers as churn**.

That tradeoff matters.

---

## Accuracy Is Not the Whole Decision

The experiment produced an important model-selection tradeoff.

If the objective were simply:

> Maximize overall classification accuracy

then Logistic Regression performed best in this experiment.

But if the business objective were:

> Avoid missing customers who are actually going to churn

then recall becomes more important.

Under the selected threshold:

```text
Logistic Regression → Recall 58.3%
Decision Tree       → Recall 83.3%
Random Forest       → Recall 93.3%
```

The Random Forest detects considerably more positive churn cases, but generates many more false positives.

Therefore, the notebook does **not** establish one universally "best" model.

The appropriate choice depends on the cost of:

```text
False Negative
Customer likely to churn
but model says non-churn

vs.

False Positive
Customer unlikely to churn
but model says churn
```

That business tradeoff would need to be defined before selecting a production model.

---

## Why the 0.30 Threshold Matters

A classifier produces probabilities before those probabilities become binary decisions.

Conceptually:

```text
Predicted churn probability
            ↓
      Decision threshold
            ↓
       Churn / No churn
```

Lowering the threshold can classify more observations as positive.

In this experiment, the threshold is explicitly set to:

```python
0.30
```

rather than relying only on the conventional `0.50` cutoff.

The results in this README therefore apply specifically to the **0.30 threshold used in the notebook**.

No threshold-optimization experiment is currently performed, so the project does not claim that `0.30` is the optimal threshold.

---

## What This Experiment Demonstrates

The notebook provides hands-on evidence of working with:

- Supervised machine learning
- Binary classification
- Features and target variables
- Training vs inference
- Train/test splitting
- Stratified sampling
- Missing-value handling
- Numerical preprocessing
- Categorical preprocessing
- Feature scaling
- One-hot encoding
- `ColumnTransformer`
- scikit-learn `Pipeline`
- Logistic Regression
- Decision Trees
- Random Forests
- Class imbalance handling
- Probability prediction
- Classification thresholds
- Confusion matrices
- Accuracy
- Precision
- Recall
- F1 score
- Model comparison

---

## Limitations

This is a learning experiment rather than a production churn prediction system.

Important limitations include:

### Synthetic data

The dataset is generated programmatically.

The relationships between customer attributes and churn therefore come from the data-generation logic rather than observed customer behavior.

Results should not be interpreted as evidence of performance on a real churn dataset.

### Single train/test split

The current experiment uses one stratified train/test split.

It does not yet perform:

- Cross-validation
- Repeated experiments
- Confidence intervals

### No hyperparameter search

The project does not perform systematic hyperparameter optimization.

### Fixed classification threshold

All reported model comparisons use:

```text
threshold = 0.30
```

The notebook does not compare multiple thresholds or optimize the threshold against a defined business cost.

### Limited model set

The experiment compares:

- Logistic Regression
- Decision Tree
- Random Forest

Gradient boosting and other classifiers are not included in the current notebook.

### No production inference layer

The trained models are not currently:

- Persisted to disk
- Exposed through an API
- Deployed
- Monitored

Those concerns belong to later stages of the AI Engineering roadmap.

---

## What I Learned

The most important lesson from this experiment was that training a model is only one part of machine learning.

The complete workflow is closer to:

```text
Understand Data
      ↓
Define Features / Target
      ↓
Split Data Correctly
      ↓
Build Preprocessing
      ↓
Train Models
      ↓
Generate Probabilities
      ↓
Choose Decision Rule
      ↓
Evaluate Errors
      ↓
Compare Tradeoffs
```

The model with the highest accuracy is not automatically the model that best serves the actual objective.

A model decision requires understanding both the metric and the cost of the mistakes behind that metric.

---

## AI Engineering Journey

This experiment represents **Week 3 — Machine Learning Fundamentals** in my broader transition:

```text
Production Full Stack / MERN Engineering
                    ↓
          Python for AI/Backend
                    ↓
        Data & Math Foundations
                    ↓
      Machine Learning Fundamentals
                    ↓
            Deep Learning
                    ↓
           LLM Engineering
                    ↓
                 RAG
                    ↓
               Agents
                    ↓
       Production AI Engineering
```

The objective is not to abandon software engineering for model experimentation.

It is to progressively combine:

> **Software Engineering + Machine Learning + AI Systems + Production Engineering**

into one engineering skill set.

---

## Tech Stack

```text
Python
Pandas
NumPy
scikit-learn
Jupyter Notebook
```

---

## Running the Notebook

The experiment is contained in the Week 3 Jupyter notebook.

Install the required libraries:

```bash
pip install pandas numpy scikit-learn jupyter
```

Start Jupyter:

```bash
jupyter notebook
```

Then open the Week 3 notebook and execute the cells in order.

The notebook first generates:

```text
customer_churn.csv
```

and then uses that dataset for preprocessing, training, inference, and model evaluation.

---

## Next Step

The next stage is to move beyond baseline model training into more rigorous ML evaluation and model serving.

That includes concepts such as:

- Validation strategy
- Cross-validation
- Overfitting and underfitting
- More deliberate metric selection
- Data leakage
- Model persistence
- API-based inference

The goal is to move from:

> **"I can train ML models."**

toward:

> **"I can evaluate, serve, and reason about ML systems."**