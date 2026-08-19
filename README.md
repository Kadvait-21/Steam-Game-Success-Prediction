# Steam Game Success Prediction

> **Can we predict whether a Steam game will be successful before it launches?**

This project builds a machine-learning pipeline to predict **Steam game success from pre-launch information** such as price, release timing, platform support, genres, categories, language support, achievements, developers, and publishers.

Instead of defining success using a single metric, the project constructs a **multi-dimensional success score** combining player reception, market reach, engagement, and recency. The score is then converted into three interpretable classes: **Low, Medium, and High**.

The completed pipeline covers **data cleaning → target construction → exploratory analysis → feature engineering → Random Forest modeling → hyperparameter tuning → final evaluation → feature-importance analysis**.

---

##  Project Highlights

| Metric | Result |
|---|---:|
| Original games | ~111,452 |
| Final modeling dataset | **89,238 games** |
| Prediction classes | **Low / Medium / High** |
| Final model | **Tuned Random Forest Classifier** |
| Test Accuracy | **86.34%** |
| Test F1-Score | **86.34%** |
| Best CV Weighted F1 | **86.34%** |
| Validation Accuracy | **86.36%** |

### Final Test Performance

| Class | Precision | Recall | F1-Score |
|---|---:|---:|---:|
| High | 0.88 | 0.83 | 0.85 |
| Low | 0.90 | 0.86 | 0.88 |
| Medium | 0.84 | 0.89 | 0.86 |
| **Overall** | **0.86** | **0.86** | **0.86** |

---

##  Objective

The goal is to answer:

> **Can a game's success category be predicted using only information available before or at release?**

This makes the project more realistic than models that directly use post-launch reviews, owners, playtime, or concurrent users as predictors.

The completed system performs **three-class classification**:

```text
Pre-launch Game Information
          ↓
Feature Engineering
          ↓
Random Forest Classifier
          ↓
┌─────────┬─────────┬─────────┐
│   Low   │ Medium  │  High   │
└─────────┴─────────┴─────────┘
```

---

#  Dataset

The project uses a Steam games dataset containing approximately **111K game records** and 39 raw columns.

After cleaning and preprocessing, **89,238 records** were used for the modeling pipeline.

The raw dataset contains information such as:

- Game name
- Release date
- Estimated owners
- Price
- DLC count
- Supported languages
- Platforms
- Achievements
- Developers
- Publishers
- Categories
- Genres
- Tags
- Reviews
- Playtime
- Peak CCU

Not all of these fields are used as prediction features.

---

#  Defining Game Success

"Success" is inherently multi-dimensional. A game can have:

- high reach but poor reviews,
- strong reviews but a small audience,
- high engagement but limited commercial reach.

To capture these dimensions, the project defines:

```text
success_score =
    0.40 × quality
  + 0.30 × reach
  + 0.20 × engagement
  + 0.10 × recency
```

### Components

| Component | Weight | Definition |
|---|---:|---|
| **Quality** | 40% | Positive review ratio |
| **Reach** | 30% | Normalized estimated owners |
| **Engagement** | 20% | Normalized average playtime |
| **Recency** | 10% | Normalized release year |

The resulting score ranges from approximately **0.032 to 0.761**.

---

#  Success Categories

Because the score distribution is skewed, fixed thresholds such as `0.33` and `0.66` produced an undesirable class imbalance.

The project therefore uses percentile-informed thresholds:

```text
Low       : success_score < 0.25
Medium    : 0.25 ≤ success_score < 0.43
High      : success_score ≥ 0.43
```

Approximate class distribution:

| Class | Games | Share |
|---|---:|---:|
| Low | 24,682 | ~27.7% |
| Medium | 41,394 | ~46.4% |
| High | 23,162 | ~26.0% |

This provides a much more practical classification target.

---

#  Preventing Target Leakage

One of the central design decisions was separating **how success is measured** from **what the model is allowed to know before launch**.

### Used to define the target

- Positive reviews
- Negative reviews
- Estimated owners
- Average playtime

### Used as prediction inputs

Only pre-launch or release-time information such as:

- Price
- Required age
- DLC count
- Achievements
- Release year
- Developers
- Publishers
- Genres
- Categories
- Tags
- Supported languages
- Windows / Mac / Linux support
- Website/support availability

Post-launch variables such as review counts, user scores, peak CCU, recommendations, and playtime are **not used as predictive features**.

This allows the model to answer the intended question:

> **Given what we know before launch, which success category is this game likely to fall into?**

---

#  Data Preparation

## 1. Raw Data Issue

The original CSV contained a column alignment problem around:

```text
DiscountDLC count
```

which should represent two separate fields:

```text
Discount
DLC count
```

The column structure was corrected before analysis.

## 2. Removing Unreliable Records

Games with:

```text
Estimated owners = 0 - 0
```

were removed because these records included playtests, removed/delisted games, and records with unusable ownership information.

Approximately **22K records** were removed at this stage.

## 3. Data Conversion

The pipeline converts:

- Release date → release year
- Owner ranges → numerical midpoint estimates
- Price → numerical value

Example:

```text
20,000 - 50,000 → 35,000
```

## 4. Missing Values

Categorical fields were handled using `"Unknown"` where appropriate, while critical records such as invalid release years were removed.

## 5. Additional Features

The project creates engineered features including:

- `Has_English`
- `Language_Count`
- `Language_Category`
- `Platform_Count`
- `Platform_Support`
- `Price_Range`
- `Release_Period`
- Website/support indicators

---

#  Exploratory Data Analysis

The project examined relationships between game success and:

- Genres
- Categories
- Price
- Platform availability
- Language localization
- Release period
- Numeric features

## Major EDA Findings

###  Genre

Strategy, RPG, and Multiplayer-oriented games showed relatively strong high-success rates.

More importantly, **genre combinations appear more informative than isolated genres**.

---

###  Price

Higher-priced games showed a larger proportion of high-success titles.

The `$10–$30` range showed a particularly strong association with high-success games.

However, this is **correlation, not proof that higher pricing causes success**.

---

###  Platform Support

Games supporting:

```text
Windows + Mac + Linux
```

showed the highest high-success proportion in the EDA.

This suggests broader platform availability may be associated with greater market reach.

---

###  Language Localization

Games supporting more languages showed substantially higher high-success proportions.

Games supporting **10+ languages** had a high-success rate of approximately **41%** in the EDA.

This supports the idea that localization can be an important indicator of market reach.

---

###  Market Saturation

Success rates decline considerably across newer release periods.

The analysis found approximately:

```text
2006–2013 → average success ~0.45–0.47
2021–2024 → average success ~0.27–0.30
```

This suggests that increasing competition and market saturation are important temporal factors.

---

#  Feature Engineering

The final modeling pipeline transforms the original metadata into machine-learning features.

### Numeric Features

- Price
- Required age
- DLC count
- Achievements
- Release year
- Language count
- Platform count
- Other engineered numeric variables

### Encoded Categorical Features

Multi-category information was transformed into numerical representations suitable for the Random Forest model.

The pipeline handles:

- Genres
- Categories
- Developers
- Publishers
- Platform indicators
- Support indicators
- Language-related features
- Release-period information

High-cardinality developer and publisher information is particularly important in the final model.

---

#  Model Development

Several modeling considerations were explored, with the final predictive model being a **Random Forest Classifier**.

## Baseline

A baseline Random Forest was first trained using default parameters.

The baseline achieved approximately:

```text
Validation Accuracy: 85.75%
Validation F1:       85.75%
```

## Hyperparameter Optimization

The Random Forest was optimized using:

```text
RandomizedSearchCV
3-fold cross-validation
20 parameter configurations
F1-weighted scoring
```

### Final Hyperparameters

```text
n_estimators      = 300
max_depth         = 40
min_samples_split = 20
min_samples_leaf  = 1
max_features      = sqrt
bootstrap         = False
class_weight      = None
```

The best cross-validation weighted F1-score was:

```text
0.8634
```

---

#  Final Results

The tuned model was evaluated on a held-out test set.

### Final Test Performance

```text
Accuracy : 86.34%
F1-Score : 86.34%
```

### Classification Report

| Class | Precision | Recall | F1 |
|---|---:|---:|---:|
| High | 0.88 | 0.83 | 0.85 |
| Low | 0.90 | 0.86 | 0.88 |
| Medium | 0.84 | 0.89 | 0.86 |
| **Weighted Avg.** | **0.86** | **0.86** | **0.86** |

The model performs particularly well on the **Low** class, while the High class has slightly lower recall, indicating that some genuinely high-success games are classified into another category.

---

#  Feature Importance

Random Forest feature importance reveals an important result:

| Feature Group | Importance |
|---|---:|
| **Developer / Publisher** | **67.17%** |
| Numeric | 12.55% |
| Genres | 5.53% |
| Categories | 5.29% |
| Platforms | 0.53% |

### Key Finding

> **Developer and publisher information dominates the model's predictive signal.**

This suggests that historical developer/publisher reputation is highly informative when predicting Steam game success.

It also provides an important business insight: success is not determined solely by the intrinsic characteristics of a game. **Who develops and publishes the game matters substantially.**

This result should not be interpreted causally—the feature importance indicates predictive usefulness, not that a developer or publisher directly causes success.

---

#  Final Pipeline

```text
                 Steam Games Dataset
                         │
                         ▼
               Data Cleaning & Validation
                         │
                         ▼
              Remove Unreliable Records
                         │
                         ▼
               Success Score Construction
                         │
                         ▼
             Low / Medium / High Classes
                         │
                         ▼
                 Feature Engineering
                         │
             ┌───────────┴───────────┐
             ▼                       ▼
        Numeric Features       Categorical Features
             │                       │
             └───────────┬───────────┘
                         ▼
                  Train / Validation /
                     Test Split
                         │
                         ▼
                Baseline Random Forest
                         │
                         ▼
                 Randomized Search
                         │
                         ▼
                 Tuned Random Forest
                         │
                         ▼
                Final Test Evaluation
                         │
              ┌──────────┴──────────┐
              ▼                     ▼
        86.34% Accuracy      Feature Importance
```

---

#  Evaluation Strategy

The project uses a separate held-out test set for the final performance estimate.

The tuned model was selected using cross-validation and then evaluated independently on the test set.

Primary metrics:

- Accuracy
- Precision
- Recall
- F1-score
- Confusion matrix
- Feature importance

Weighted F1 was used during hyperparameter optimization because the three classes are not perfectly equal in size.

---

#  Key Insights

The completed analysis leads to several important conclusions:

1. **Steam game success is multi-dimensional.**
2. **Genre combinations matter more than isolated genres.**
3. **Localization breadth is strongly associated with success.**
4. **Broader platform availability correlates with stronger outcomes.**
5. **Pricing is associated with success, but the relationship is not causal.**
6. **Steam has become increasingly competitive over time.**
7. **Developer and publisher identity provide the strongest predictive signal in the final Random Forest.**
8. A relatively simple ensemble model can achieve **~86% classification accuracy** on this dataset.

---

#  Limitations

Despite the strong predictive performance, the result should not be interpreted as a guarantee of future game success.

### 1. Success Score Is Project-Defined

The `40/30/20/10` weighting is a modeling choice rather than an industry-standard definition of success.

### 2. Target Uses Post-Launch Outcomes

The target is constructed from information that becomes available after release. The model itself uses pre-launch information, but the target necessarily represents an eventual outcome.

### 3. Correlation ≠ Causation

For example:

```text
More languages → higher observed success
```

does not prove:

```text
Adding more languages → causes higher success
```

### 4. Developer/Publisher Dominance

The strong importance of developer/publisher features may reflect historical reputation, established studios, repeated franchises, or other correlated factors.

### 5. Temporal Generalization

Steam's competitive environment changes over time. A model trained on historical data may not maintain the same performance on future releases.

---

#  Technologies

```text
Python
├── pandas
├── NumPy
├── scikit-learn
├── matplotlib
└── seaborn
```

Core ML components:

```text
RandomForestClassifier
RandomizedSearchCV
classification_report
confusion_matrix
feature_importances_
```

---

#  Project Structure

```text
Steam-Game-Success-Prediction/
│
├── games.csv
├── success_prediction_clean(1).ipynb
└── README.md
```

---

#  How to Run

### 1. Clone the repository

```bash
git clone <repository-url>
cd Steam-Game-Success-Prediction
```

### 2. Install dependencies

```bash
pip install pandas numpy scikit-learn matplotlib seaborn
```

### 3. Add the dataset

Place:

```text
games.csv
```

in the project directory.

### 4. Open the notebook

```bash
jupyter notebook success_prediction_clean(1).ipynb
```

Run the notebook cells sequentially to reproduce the data preparation, analysis, feature engineering, model training, tuning, and evaluation pipeline.

---


### Final Result

> **Tuned Random Forest Classifier — 86.34% test accuracy / 86.34% F1-score**

---

##  Conclusion

This project demonstrates a complete end-to-end machine-learning workflow for predicting Steam game success from pre-launch information.

The final Random Forest model achieves **86.34% accuracy** on the held-out test set. More importantly, the project provides interpretable insights into the factors associated with game success, with **developer and publisher information emerging as the strongest predictive feature group**.

The project is **complete**: data preparation, EDA, feature engineering, model development, hyperparameter tuning, evaluation, and interpretation have all been performed.

---
