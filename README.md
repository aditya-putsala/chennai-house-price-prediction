# Chennai House Price Prediction

Predicting the sale price of houses in seven Chennai localities from their size, location, building type and other features, using five regression models: Linear Regression, KNN, Decision Tree, Random Forest and XGBoost.

---

## Table of Contents

- [Results at a Glance](#results-at-a-glance)
- [Dataset](#dataset)
- [Data Cleaning and Preparation](#data-cleaning-and-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Feature Preparation and Modelling](#feature-preparation-and-modelling)
- [Model Results](#model-results)
- [Getting Started](#getting-started)
- [Repository Structure](#repository-structure)
- [Tech Stack](#tech-stack)

---

## Results at a Glance

| Item | Value |
|------|-------|
| Houses / columns | 7,109 / 22 |
| Localities | 7 (Karapakkam, Adyar, Chrompet, Velachery, KK Nagar, Anna Nagar, T Nagar) |
| Prediction target | `SALES_PRICE` + `REG_FEE` + `COMMIS` (sale price plus registration fee and commission) |
| Best model | **XGBoost: R² 0.995, MAE ₹218,131** (15 features, 60/40 split) |
| Second best | Random Forest: R² 0.987, MAE ₹357,164 |
| Baseline | Linear Regression: R² 0.925, MAE ₹818,137 |
| Reduced-feature model | XGBoost on 7 features (80/20 split): R² 0.952 |

---

## Dataset

`Chennai_House_Price.csv` has **7,109 rows and 22 columns**, one row per house sold.

| Column | Description |
|--------|-------------|
| `PRT_ID` | Unique ID for each house |
| `AREA` | Locality where the house is located |
| `INT_SQFT` | Total area of the house in square feet |
| `DATE_SALE` | Date the house was sold |
| `DIST_MAINROAD` | Distance from the main road in meters |
| `N_BEDROOM` | Number of bedrooms |
| `N_BATHROOM` | Number of bathrooms |
| `N_ROOM` | Total number of rooms |
| `SALE_COND` | Sale condition |
| `PARK_FACIL` | Whether a parking facility is available |
| `DATE_BUILD` | Date the house was built |
| `BUILDTYPE` | Type of building |
| `UTILITY_AVAIL` | Public utilities available |
| `STREET` | Type of street the house is on |
| `MZZONE` | Municipality zone |
| `QS_ROOMS`, `QS_BATHROOM`, `QS_BEDROOM`, `QS_OVERALL` | Masked score columns |
| `REG_FEE` | Total registration fee |
| `COMMIS` | Total commission paid |
| `SALES_PRICE` | Sale price of the house |

**Ranges in the data**

| Feature | Range |
|---------|-------|
| Sale price (before adding fees) | ₹2,156,875 to ₹23,667,340 |
| Registration fee | ₹71,177 to ₹983,922 |
| Commission | ₹5,055 to ₹495,405 |
| House size | 500 to 2,500 sq ft |
| Rooms / bedrooms / bathrooms | 2 to 6 / 1 to 4 / 1 to 2 |
| Distance from main road | 0 to 200 m |
| Year built | 1949 to 2010 |
| Year sold | 2004 to 2015 |

---

## Data Cleaning and Preparation

| Column | Issue in the raw data | Fix |
|--------|-----------------------|-----|
| `PRT_ID` | Unique ID per house | Dropped, since an ID can lead to overfitting |
| `SALES_PRICE` | Registration fee and commission affect the final price indirectly | Target set to `SALES_PRICE + REG_FEE + COMMIS`, then `REG_FEE` and `COMMIS` dropped. Target range becomes ₹2,253,935 to ₹24,969,043 |
| `AREA` | 17 spellings for 7 localities (for example Chrompt, Chrmpet, Chormpet, TNagar, Velchery, Karapakam, Ann Nagar) | Mapped to 7 standard names |
| `BUILDTYPE` | `Comercial`, `Other` | Mapped to `Commercial`, `Others` |
| `SALE_COND` | `Ab Normal`, `Partiall`, `PartiaLl`, `Adj Land` | Mapped to `AbNormal`, `Partial`, `AdjLand` |
| `PARK_FACIL` | `Noo` | Mapped to `No` |
| `STREET` | `Pavd`, `NoAccess` | Mapped to `Paved`, `No Access` |
| `UTILITY_AVAIL` | `All Pub` | Mapped to `AllPub` |
| `N_BEDROOM`, `N_BATHROOM` | 1 and 5 missing values, stored as decimals | Missing values filled with the most common value (1), converted to whole numbers |
| `QS_OVERALL` | 48 missing values | Filled with the column mean (3.503) |
| `DATE_SALE`, `DATE_BUILD` | Stored as text | Converted into `AGE` (sale year minus build year, range 3 to 55 years) and `BUILT_YEAR`, then the date columns dropped |

**Columns dropped after analysis.** `SALE_YEAR`, `DIST_MAINROAD`, `QS_ROOMS`, `QS_BATHROOM`, `QS_BEDROOM` and `QS_OVERALL` showed no clear relationship with sale price in the notebook's plots, so they were removed before modelling.

---

## Exploratory Data Analysis

**Houses by locality.** Chrompet has the most houses (1,702) and T Nagar the fewest (501). Average price rises from Karapakkam (lowest) through Adyar, Chrompet, Velachery, KK Nagar and Anna Nagar to T Nagar (highest). KK Nagar houses are the largest on average, while Adyar, Chrompet and Karapakkam have the smallest.

**Average sale price by category** (including fees; 1 lakh = ₹100,000)

| Feature | Average price in ₹ lakh, from lowest to highest |
|---------|--------------------------------------------------|
| `BUILDTYPE` | House 95.3, Others 102.7, Commercial 145.3 |
| `MZZONE` | A 76.6, C 84.6, I 91.7, RH 115.6, RL 123.2, RM 129.9 |
| `N_ROOM` | 2 rooms 73.1, 3 rooms 97.1, 4 rooms 122.5, 5 rooms 148.4, 6 rooms 152.8 |
| `N_BEDROOM` | 1 bedroom 102.3, 2 bedrooms 123.9, 3 bedrooms 131.4, 4 bedrooms 152.8 |
| `N_BATHROOM` | 1 bathroom 111.7, 2 bathrooms 123.1 |
| `PARK_FACIL` | No parking 108.3, Parking 119.9 |
| `STREET` | No Access 104.6, Paved 115.9, Gravel 119.9 |
| `UTILITY_AVAIL` | ELO 109.5, NoSeWa 114.2, NoSewr 114.5, AllPub 117.5 |
| `SALE_COND` | Partial 111.2, Family 112.5, AbNormal 114.4, Normal Sale 115.2, AdjLand 117.4 |

**Key observations**
- **Size and price:** Larger houses sell for more, both within each locality and overall. House size runs from 500 to 2,500 sq ft with no outliers by the IQR rule, but it is not normally distributed.
- **Rooms:** 4-room houses are the most common (36.1%). Price rises with the number of rooms, bedrooms and bathrooms.
- **Bedrooms and bathrooms:** 53.4% of houses have one bedroom and 78.6% have one bathroom.
- **Building type:** The three types are roughly equal in number (House 34.4%, Others 32.9%, Commercial 32.8%), but Commercial buildings are the most expensive.
- **Age:** Price falls as the house gets older.
- **Municipality zone:** Price increases from zone A to zone RM, with RL, RH and RM being the most common zones.
- **Streets:** Houses with no street access sell for less. Every KK Nagar house has road access.
- **Sale condition:** Average prices are similar across all five sale conditions.
- **Sale timing:** Houses were sold between 2004 and 2015, with the most sales in 2010 (1,341) and 2011 (1,300). Sale year shows no clear relationship with price.

---

## Feature Preparation and Modelling

**Encoding and scaling**
- **Ordinal (label) encoding** for `AREA`, `SALE_COND`, `PARK_FACIL`, `UTILITY_AVAIL`, `STREET` and `MZZONE`, with categories numbered in order of increasing average price.
- **One-hot encoding** for `BUILDTYPE` (`BUILDTYPE_Commercial`, `BUILDTYPE_House`, `BUILDTYPE_Others`).
- **Log transform** of `INT_SQFT`, because it does not follow a normal distribution.
- **Standard scaling** of all features (fitted on the training set only).

**Full-feature comparison.** 15 features (`AREA`, `INT_SQFT`, `N_BEDROOM`, `N_BATHROOM`, `N_ROOM`, `SALE_COND`, `PARK_FACIL`, `UTILITY_AVAIL`, `STREET`, `MZZONE`, `AGE`, `BUILT_YEAR` and the three `BUILDTYPE` columns), with a 60/40 train-test split (4,265 training and 2,844 test houses, `random_state=100`).

| Model | Settings |
|-------|----------|
| Linear Regression | Default |
| KNN Regressor | `n_neighbors=3` |
| Decision Tree Regressor | `max_depth=8` |
| Random Forest Regressor | `random_state=0` |
| XGBoost Regressor | Default |

---

## Model Results

| Model | Train R² | Test R² | MAE (₹) |
|-------|:--------:|:-------:|--------:|
| Linear Regression | 0.92 | 0.9251 | 818,137 |
| KNN | 0.97 | 0.9442 | 718,211 |
| Decision Tree | 0.97 | 0.9572 | 655,774 |
| Random Forest | 1.00 | 0.9869 | 357,164 |
| **XGBoost** | **1.00** | **0.9950** | **218,131** |

**XGBoost has the best R² and the lowest mean absolute error, followed by Random Forest.**

**Most important features**
- **Linear Regression:** number of rooms, building type and area have the largest coefficients.
- **Decision Tree and Random Forest:** `AREA` is the most important feature.

**Reduced-feature model.** A second XGBoost model uses only 7 features: `AREA`, `INT_SQFT`, `N_ROOM`, `MZZONE` and the three `BUILDTYPE` columns. It dropped `N_BATHROOM`, `AGE`, `PARK_FACIL`, `STREET`, `SALE_COND`, `N_BEDROOM`, `UTILITY_AVAIL` and `BUILT_YEAR`. With an 80/20 split (5,687 training and 1,422 test houses), it reaches an **R² of 0.9522**.

---

## Getting Started

**1. Install the libraries**
```bash
pip install pandas numpy matplotlib seaborn scikit-learn xgboost jupyter
```

**2. Place the dataset next to the notebook.** The notebook loads the data with `pd.read_csv('Chennai House Price.csv')`, so make sure the file name in that cell matches your CSV.

**3. Open the notebook**
```bash
jupyter notebook House_Price_Prediction.ipynb
```

---

## Repository Structure

```
.
├── House_Price_Prediction.ipynb   # Cleaning, EDA, encoding and model comparison
├── Chennai_House_Price.csv        # 7,109 houses x 22 columns
└── README.md
```

---

## Tech Stack

Python, pandas, NumPy, Matplotlib, Seaborn, scikit-learn (Linear Regression, KNN, Decision Tree, Random Forest, StandardScaler, metrics), XGBoost, Jupyter Notebook.
