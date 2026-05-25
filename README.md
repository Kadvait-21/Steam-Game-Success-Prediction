# Steam Games Success Prediction - Project Summary

## Project Overview
This project analyzes Steam games data to predict game success based on **pre-launch features only**. The goal is to build a model that can predict whether a game will be successful before it launches, using only information available at or before release.

---

## Dataset Information
- **Source**: `games.csv` - Steam games dataset
- **Original size**: ~111,452 games
- **After cleaning**: 89,239 games
- **Final dataset**: 89,117 games (after removing missing values)
- **Features**: 23 pre-launch features
- **Target variable**: Success score (0-1 scale) and 3-class balanced categories (Low/Medium/High)

---

## Work Completed

### Phase 1: Data Preparation ✅ (Sections 1-7)

#### 1. Data Loading & Initial Inspection
**Issue Discovered**: Column name misalignment
- The CSV had a formatting error where `DiscountDLC count` should have been two separate columns: `Discount` and `DLC count`
- This caused all subsequent column names to be shifted
- **Solution**: Manually corrected all column names with the proper list

#### 2. Data Cleaning
**Filtered Out Unreliable Data:**
- **Removed 22,091 games** with `0 - 0` estimated owners
- These included:
  - Outdated data (old ownership numbers not updated)
  - Playtests (temporary test versions)
  - Removed/delisted games

**Data Type Conversions:**
1. **Release Date → Release Year**
   - Converted release dates to datetime format
   - Extracted only the year for modeling
   - 122 games had invalid dates (removed later)

2. **Estimated Owners → Numeric**
   - Original format: range strings (e.g., "20,000 - 50,000")
   - Converted to numeric using midpoint of range
   - Example: "20,000 - 50,000" → 35,000

3. **Price → Numeric**
   - Converted price strings to float values
   - Handled free games (Price = 0)

**Created Support Binary Flags:**
- `has_website`: 1 if game has a website, 0 otherwise
- `has_support_url`: 1 if game has support URL, 0 otherwise
- `has_support_email`: 1 if game has support email, 0 otherwise

#### 3. Success Metric Creation
**Multi-Dimensional Success Score Formula:**
```
success_score = 0.4 × quality + 0.3 × reach + 0.2 × engagement + 0.1 × recency
```

**Components:**

| Dimension | Variable | Weight | Calculation | What It Measures |
|-----------|----------|--------|-------------|------------------|
| **Quality** | positive_ratio | 0.4 (40%) | Positive / (Positive + Negative + 1) | Player satisfaction and reception |
| **Reach** | estimated_owners_norm | 0.3 (30%) | MinMax normalized log(owners) | Commercial success and market impact |
| **Engagement** | playtime_norm | 0.2 (20%) | MinMax normalized avg playtime | Player retention and replayability |
| **Recency** | release_year_norm | 0.1 (10%) | MinMax normalized release year | Account for older games losing attention |

**Key Statistics:**
- Score range: 0.032 to 0.761
- Mean: 0.322
- Median: 0.359
- 25th percentile: 0.222
- 75th percentile: 0.432

#### 4. Target Variable Creation
**Success_Category_Balanced (3-Class Classification)**

Using **percentile-based thresholds**:
- **Low**: success_score < 0.25 (~30% of games)
- **Medium**: 0.25 ≤ success_score < 0.43 (~45% of games)
- **High**: success_score ≥ 0.43 (~25% of games)

**Why balanced thresholds?**
- The success score distribution is left-skewed
- Simple equal thresholds (0.33, 0.66) created severe imbalance
- Percentile-based thresholds provide better class distribution for machine learning

#### 5. Feature Selection
**Selected 23 Pre-Launch Features:**

**Target Variables (2):**
- `success_score` (continuous, 0-1 scale)
- `Success_Category_Balanced` (categorical: Low/Medium/High)

**Numeric Features (6):**
- `Price`, `Required age`, `DLC count`, `Achievements`, `Release year`, `Estimated owners numeric`

**Categorical Features (6):**
- `Developers`, `Publishers`, `Genres`, `Categories`, `Tags`, `Supported languages`

**Platform Support (3 binary):**
- `Windows`, `Mac`, `Linux`

**Support Features (3 binary):**
- `has_website`, `has_support_url`, `has_support_email`

**Identification (1):**
- `Name`

**Excluded Features (Post-Launch Metrics):**
- ❌ Review counts, User scores, Peak CCU, Recommendations, Playtime metrics

#### 6. Missing Value Handling
**Strategy:**
1. **Categorical/Text Columns** → Filled with `'Unknown'`
   - Tags (17.2% missing), Categories (1.3% missing)
   - Genres, Supported languages, Developers, Publishers

2. **Critical Features** → Dropped rows
   - Release year (122 rows) - Essential for success score
   - success_score (0 rows after year drop) - Target variable

**Final Clean Dataset:**
- 89,117 games
- 23 features
- 0 missing values

#### 7. Duplicate Detection
- Identified 1,439 duplicate game entries (same name, different releases)
- **Not removed** - may be legitimate (remasters, different editions)

---

### Phase 2: Exploratory Data Analysis ✅ (Sections 8-15)

#### 8. Genre Analysis
**Key Findings:**
- **34 unique individual genres** identified
- **Success-driving genres**: 
  - Strategy: 27% High success rate
  - RPG: 24% High success rate  
  - Multiplayer: 18% High success rate
- **Volume vs Success gap**: 
  - Indie: 70% of all games but doesn't guarantee success
  - Casual: 42% of games with mixed success
- **Consistent performers**: Action and Adventure maintain ~40% presence across all success levels

**Insight**: Genre combinations matter more than single genres - successful games often blend Action/Strategy/RPG elements.

#### 9. Categories Analysis
**Key Findings:**
- **Single-player dominance**: ~95% across all success categories (universal presence)
- **Steam integration features** show strong positive correlation:
  - Steam Achievements
  - Steam Cloud
  - Full Controller Support
- **Multiplayer/Co-op modes**: Slightly more prevalent in high-success games

**Insight**: Player experience features (achievements, cloud saves, controller support) drive success more than game mode alone.

#### 10. Price Analysis
**Key Findings:**
- **Median price progression**: $4.99 (Low/Medium) → $6.99 (High success)
- **Free games**: ~10% in medium/high groups, high visibility but limited monetized impact
- **Moderately priced ($5-$20)**: Balanced performance, best value-to-quality ratio
- **Premium titles ($20-$30+)**: Highest proportion of high-success games (≈40-45%)
- **Sweet spot**: Mid-to-premium pricing ($10-$30) correlates most strongly with high success

**Insight**: Price is a quality signal - well-produced, higher-priced titles tend to succeed when supported by quality and reputation.

#### 11. Platform Support Analysis
**Key Findings:**
- **All 3 platforms (Win+Mac+Linux)**: ~39% high-success rate (highest)
- **Windows-only**: ~23% high-success rate despite market dominance
- **Two platforms**: Balanced performance, boosts success without full multi-platform investment

**Insight**: Cross-platform availability increases visibility, player base, and overall engagement.

#### 12. Language Support Analysis
**Key Findings:**
- **English language support**: 26% high-success rate vs 24.6% for non-English (crucial factor)
- **6-10 languages or 10+ languages**: Highest high-success rates (≈41%)
- **Single-language games**: ~55% of total, lowest high-success proportion

**Insight**: Localization strongly correlates with wider player adoption - accessibility and international reach play major roles.

#### 13. Release Year Trends Analysis
**Key Findings:**
- **Pre-2015**: Highest proportion of high-success titles (~49%), less competition
- **2015-2020**: Sharp increase in releases, drop in high-success proportion (Steam Greenlight/indie boom)
- **2021-2024**: Highest share of low-success titles (~37%), intense competition
- **Success score decline**: Steady decline from ~0.45-0.47 (2006-2013) to ~0.27-0.30 (2021-2024)
- **Market saturation**: Peak releases around 2021-2024 coincide with lowest average success scores

**Insight**: As Steam's game volume increased dramatically post-2014, achieving success became significantly harder.

#### 14. Feature Correlation Analysis
**Key Findings:**
- **Strongest predictors**: Achievements (+0.15), Language_Count (+0.12), Mac (+0.08), has_website (+0.07)
- **Weak predictors**: Price (near 0), DLC count (+0.04), Required age (near 0)
- **Multicollinearity detected**: Support features and platform features are intercorrelated
- **Missing piece**: Numeric correlations are weak-to-moderate; categorical features likely drive most variance

**Insight**: No single numeric feature dominates success - it's multidimensional. Feature engineering and categorical encoding will be critical.

#### 15. EDA Summary
Comprehensive summary of all findings organized by factor type:
- **Content & Design Factors**: Genre combinations, Steam integration features
- **Market Reach Factors**: Language localization, cross-platform support
- **Business Strategy Factors**: Optimal pricing ($10-$30), free game challenges
- **Temporal Trends**: Market saturation effects, competition intensity
- **Feature Correlations**: Weak numeric correlations, categorical importance

---

## Key Design Decisions

### 1. Why Multi-Dimensional Success Score?
- **Single metric is insufficient**: A game with high sales but poor reviews isn't truly successful
- **Captures different aspects**: Commercial success (reach), quality (reviews), engagement (playtime)
- **Weighted appropriately**: Quality matters most (40%), then reach (30%), engagement (20%), recency (10%)
- **Normalized components**: All components scaled 0-1 for fair combination

### 2. Why Release Year Instead of Release Date?
- **Reduces noise**: Specific dates add unnecessary granularity
- **Captures trends**: Gaming market evolves by year, not by day
- **Better for modeling**: Year is ordinal and captures temporal trends
- **Accounts for recency**: Newer games have different dynamics than older ones

### 3. Why Percentile-Based Thresholds?
- **Data-driven**: Based on actual distribution, not arbitrary cutoffs
- **Balanced classes**: Better for machine learning (avoids extreme imbalance)
- **Meaningful categories**: 
  - Low = bottom 30% (struggling games)
  - Medium = middle 45% (average performers)
  - High = top 25% (successful games)

### 4. Why Include "Supported languages"?
- **Pre-launch feature**: Known before release
- **Strong predictor**: Games supporting English have significantly wider reach
- **Market insight**: Language support indicates target market size
- **EDA confirmed**: 10+ languages correlates with 41% high-success rate

---

## Critical Insights from EDA

### Success Predictors Ranked by Importance:

**🔴 High Importance:**
1. **Genre combinations** (especially Strategy/RPG/Multiplayer blends)
2. **Steam integration features** (Achievements, Cloud, Controller Support)
3. **Multiplayer/Co-op capabilities**
4. **Language localization breadth** (10+ languages optimal)

**🟡 Medium Importance:**
5. **Cross-platform support** (multi-platform advantage clear)
6. **Price positioning** ($10-$30 sweet spot)
7. **Achievement count** (platform integration signal)
8. **Release timing** (pre-2015 had advantages, current market saturated)

**🟢 Low Importance:**
9. **DLC count** (at launch - limited signal)
10. **Required age** (near-zero correlation)
11. **Individual support flags** (multicollinearity issues)

### Market Evolution Insights:

**The Steam Market Has Changed Dramatically:**
- **2006-2013**: Average success ~0.45-0.47, limited competition
- **2014-2017**: Greenlight era begins, success starts declining
- **2018-2020**: Indie boom, sharp increase in releases
- **2021-2024**: Market saturation peak, average success ~0.27-0.30

**What This Means for Modeling:**
- Release period should be considered as categorical feature
- Older games may have fundamentally different success patterns
- Market saturation is a real phenomenon affecting all games

---

## Data Quality Observations

### Strengths:
✅ Large sample size (89,117 games)
✅ Comprehensive feature set (23 pre-launch features)
✅ Balanced target variable (30%/45%/25% distribution)
✅ Multiple genres and categories per game (rich categorical data)
✅ Clean data after processing (0 missing values)

### Challenges:
⚠️ **Left-skewed distribution**: Most games have low success scores (reflects reality)
⚠️ **High cardinality**: 8,000+ developers, 6,000+ publishers
⚠️ **Multi-label features**: Games have multiple genres, categories, tags
⚠️ **Weak numeric correlations**: Categorical features drive most variance
⚠️ **Multicollinearity**: Support features and platform features intercorrelated
⚠️ **Temporal effects**: Success criteria changed over time

---

## Next Steps (Phase 3: Feature Engineering & Modeling)

### 1. Feature Engineering (Priority: HIGH)

**Categorical Encoding:**
- [x] Language features extracted (Has_English, Language_Count, Language_Category)
- [x] Platform support aggregated (Platform_Support composite)
- [x] Price ranges categorized (Price_Range bins)
- [x] Release periods created (Release_Period categorical)
- [ ] **TODO**: One-hot encode Genres (multi-label)
- [ ] **TODO**: One-hot encode Categories (multi-label)
- [ ] **TODO**: Handle Tags (very sparse, 17.2% missing)
- [ ] **TODO**: Encode Developers/Publishers (high cardinality - use target encoding or frequency encoding)

**Composite Features:**
- [x] Platform_Count (cross-platform score)
- [x] Language_Category (language breadth bins)
- [ ] **TODO**: Support_Level (aggregated support features)
- [ ] **TODO**: Steam_Integration_Score (Achievement + Cloud + Controller features)
- [ ] **TODO**: Genre_Diversity (number of genres per game)
- [ ] **TODO**: Category_Diversity (number of categories per game)

**Interaction Features:**
- [ ] Genre × Platform combinations
- [ ] Price × Platform combinations
- [ ] Language_Count × Platform_Count

### 2. Handle High-Cardinality Features

**Developers (8,000+ unique):**
- Option A: Target encoding (encode by average success rate)
- Option B: Frequency encoding (encode by game count)
- Option C: Group into tiers (indie / small studio / established / AAA)
- **Recommended**: Target encoding with cross-validation to avoid leakage

**Publishers (6,000+ unique):**
- Same options as Developers
- May create "self-published" binary flag

### 3. Address Multicollinearity
- Remove redundant support features (keep composite Support_Level)
- Consider PCA on platform features
- Monitor VIF (Variance Inflation Factor) during modeling

### 4. Handle Duplicates (1,439 games)
- Investigate sample of duplicates
- Decide: keep all, keep latest, merge data, or remove?
- May create "is_remaster" or "edition_count" features

### 5. Model Selection & Training

**For Regression (predict exact success_score):**
- Linear Regression (baseline)
- Ridge/Lasso Regression (handle multicollinearity)
- Random Forest Regressor
- XGBoost Regressor (likely best performer)
- LightGBM (faster alternative to XGBoost)

**For Classification (predict Low/Medium/High):**
- Logistic Regression (baseline)
- Random Forest Classifier
- XGBoost Classifier (likely best performer)
- LightGBM Classifier
- Support Vector Machine (SVM)

**Training Strategy:**
- Use stratified K-fold cross-validation (K=5)
- Consider temporal split (train on pre-2020, test on 2020+)
- Use class weights or SMOTE if needed
- Grid search or random search for hyperparameter tuning

### 6. Model Evaluation

**Regression Metrics:**
- RMSE (Root Mean Squared Error)
- MAE (Mean Absolute Error)
- R² (Coefficient of Determination)
- MAPE (Mean Absolute Percentage Error)

**Classification Metrics:**
- Accuracy (overall correctness)
- Precision, Recall, F1-Score (per class)
- Confusion Matrix (visualize misclassifications)
- ROC-AUC (if using probabilities)
- Feature importance analysis

### 7. Feature Importance Analysis
- Use SHAP (SHapley Additive exPlanations) for interpretability
- Analyze which features drive predictions
- Validate against EDA findings
- Create feature importance visualizations

---

## Questions for Discussion

### Feature Engineering:
1. **Multi-label encoding**: Should we one-hot encode all genres/categories, or use dimensionality reduction?
2. **Developer/Publisher**: Target encoding vs frequency encoding vs grouping strategy?
3. **Tags**: Very sparse (17.2% missing), high cardinality - include or exclude?

### Modeling:
4. **Model priority**: Focus on regression (exact score) or classification (Low/Medium/High)?
5. **Validation strategy**: Random split or temporal split (respecting release year)?
6. **Class imbalance**: Use SMOTE, class weights, or keep balanced thresholds as-is?

### Business Questions:
7. **Actionable insights**: Which features are most controllable by developers (price, platforms, languages)?
8. **Success definition**: Validate our 40/30/20/10 weighting with stakeholders
9. **Market trends**: How to account for market saturation in predictions for future games?

---

## Technical Notes

**Libraries Used:**
- `pandas` - Data manipulation
- `numpy` - Numerical operations
- `matplotlib` - Visualization
- `seaborn` - Statistical visualizations (correlation heatmaps)
- `sklearn.preprocessing.MinMaxScaler` - Feature normalization

**Notebook Structure:**
```
Section 1-7:   Data Preparation (COMPLETE)
Section 8-15:  Exploratory Data Analysis (COMPLETE)
Section 16+:   Feature Engineering (TODO)
Section 17+:   Model Building (TODO)
Section 18+:   Model Evaluation (TODO)
```

**File Structure:**
```
d:\PDS Project\
├── games.csv (original data)
├── success_prediction.ipynb (main notebook)
└── README.md (this file)
```

---

## Summary Statistics

**Dataset Size:**
- Original: 111,452 games
- After filtering 0-0 owners: 89,361 games  
- After removing missing values: 89,117 games

**Success Score Distribution:**
- Min: 0.032
- 25th percentile: 0.222
- Median: 0.359
- Mean: 0.322
- 75th percentile: 0.432
- Max: 0.761

**Class Distribution (Balanced):**
- Low: ~30% (26,735 games)
- Medium: ~45% (40,103 games)
- High: ~25% (22,279 games)

**Genre Statistics:**
- Total unique genres: 34
- Games with multiple genres: Majority
- Most common: Indie (70%), Action (48%), Casual (42%)

**Platform Distribution:**
- Windows: ~98% of games
- Mac: ~23% of games
- Linux: ~21% of games
- All 3 platforms: Significant minority with highest success rate

**Language Statistics:**
- English support: ~97% of games
- Single language: ~55% of games
- 10+ languages: ~8% of games (but 41% high-success rate)

**Price Statistics:**
- Free games: ~9% of total
- Median price (Low): $4.99
- Median price (Medium): $4.99
- Median price (High): $6.99
- Optimal range: $10-$30 (40-45% high-success)

**Temporal Statistics:**
- Release year range: 1997-2024
- Peak release year: 2022
- Pre-2015 high-success rate: ~49%
- 2021-2024 high-success rate: ~20%

---

## Conclusion

We have successfully completed **Phase 1 (Data Preparation)** and **Phase 2 (Exploratory Data Analysis)** of the Steam Games Success Prediction project. 

**Key Achievements:**
✅ Cleaned and prepared 89,117 game records
✅ Created multi-dimensional success score with balanced target variable
✅ Conducted comprehensive EDA across 7 major analysis areas
✅ Identified clear patterns in genres, pricing, platforms, languages, and temporal trends
✅ Discovered market saturation effects and success factor evolution

**Critical Insights Discovered:**
- Genre combinations matter more than single genres
- Steam integration features drive success
- Language localization has strong positive impact (10+ languages optimal)
- Cross-platform support significantly boosts success rates
- Mid-to-premium pricing ($10-$30) correlates with high success
- Market has become dramatically more saturated since 2014

**Ready for Phase 3:**
- Feature engineering with clear direction from EDA
- Model building with informed feature selection
- Predictions focused on controllable pre-launch factors

The project is well-positioned for the modeling phase with strong data quality, comprehensive EDA insights, and clear feature importance rankings to guide feature engineering decisions.

---

*Last Updated: November 12, 2025*
*Notebook: success_prediction.ipynb*
*Status: EDA Complete | Feature Engineering Next | 89,117 games analyzed*
#   S t e a m - G a m e - S u c c e s s - P r e d i c t i o n  
 