# Olist Customer Purchase Analytics

A customer-level analytics and machine learning project using the Olist Brazilian e-commerce dataset to predict whether an existing customer will make another delivered purchase during a future observation period.

This project continues the [Olist Data Quality & ETL Pipeline](https://github.com/ReinSoup/OLIST-Data-Quality-ETL), using its cleaned and validated data foundation.

## Table of Contents

- [Overview](#overview)
- [Business Problem](#business-problem)
- [Approach](#approach)
- [Data Foundation](#data-foundation)
- [Customer Identity and Grain](#customer-identity-and-grain)
- [Temporal Framework](#temporal-framework)
- [Feature Engineering](#feature-engineering)
- [Target](#target)
- [Exploratory Analysis](#exploratory-analysis)
- [Model](#model)
- [Evaluation](#evaluation)
- [Threshold Analysis](#threshold-analysis)
- [Key Findings](#key-findings)
- [Limitations](#limitations)
- [Future Improvements](#future-improvements)
- [Repository Structure](#repository-structure)
- [Notebooks](#notebooks)
- [Technologies](#technologies)
- [Reproducibility](#reproducibility)
- [Related Project](#related-project)

## Overview

The project asks whether historical customer purchasing behavior contains useful signal for predicting a future repeat purchase.

The problem is deliberately framed as repeat-purchase prediction rather than subscription churn because Olist is an e-commerce marketplace and does not contain a contractual subscription cancellation event.

The implemented workflow is:

```text
Cleaned Olist data
        ↓
Customer and order validation
        ↓
Temporal feature / target split
        ↓
Customer-level RFM + AOV features
        ↓
Future repeat-purchase target
        ↓
Logistic Regression baseline
        ↓
Imbalance-aware evaluation
        ↓
Threshold analysis
        ↓
Findings and limitations
```

## Business Problem

> Can historical customer purchasing behavior be used to predict whether a customer will make another qualifying purchase during a future observation period?

The goal is to demonstrate a complete analytical workflow from business question and data validation through feature engineering, predictive modeling, evaluation, and business interpretation.

The prediction is treated as an estimate of observed behavior, not a guarantee of future purchasing or a causal explanation.

## Approach

The project follows five core principles:

1. Use customer-level analytical grain for the final modeling dataset.
2. Separate historical features from future outcomes to prevent temporal leakage.
3. Aggregate order-item data before calculating customer monetary measures.
4. Establish an interpretable baseline before adding model complexity.
5. Use metrics appropriate for a severely imbalanced target rather than relying on accuracy.

## Data Foundation

The project uses the Brazilian E-Commerce Public Dataset by Olist.

The relevant tables are:

| Table | Purpose |
|---|---|
| `customers` | Customer identity mapping |
| `orders` | Order status and purchase timestamps |
| `order_items` | Price and freight values used to calculate order value |

Key sizes inspected during preparation:

| Dataset | Rows |
|---|---:|
| Customers | 99,441 |
| Orders | 99,441 |
| Order items | 112,650 |

The current project uses cleaned datasets produced by the previous ETL workflow instead of repeating the full data-quality process.

## Customer Identity and Grain

Olist contains both `customer_id` and `customer_unique_id`.

The preparation analysis verified that:

- every `customer_id` maps to exactly one `customer_unique_id`
- a `customer_unique_id` can be associated with multiple `customer_id` records
- 2,997 unique customers are associated with more than one `customer_id`

`customer_unique_id` was therefore used as the analytical customer identifier.

The final modeling dataset follows:

> One row = one customer

This prevents multiple Olist customer records belonging to the same underlying customer from being treated as separate customers.

## Qualifying Purchase

A qualifying purchase was defined as an order with:

```text
order_status == "delivered"
```

This represents completed purchasing behavior. Intermediate states such as `shipped`, `processing`, and `invoiced`, as well as `canceled` and `unavailable` orders, were excluded.

The dataset contains 96,478 delivered orders and 2,963 orders with other statuses.

The same rule was used for historical feature construction and the future target.

## Temporal Framework

The project uses a temporal split so that features represent information available before the prediction point.

| Period | Definition | Purpose |
|---|---|---|
| Historical period | Up to 2018-03-31 | Build customer features |
| Prediction cutoff | 2018-03-31 | Separate past from future |
| Future observation period | 2018-04-01 to 2018-09-30 | Define repeat purchase |

The historical qualifying-order period begins with the first available delivered purchase on 2016-09-04. October 2018 was excluded because the available dataset ends partway through that month.

The resulting split contained:

- 64,158 historical delivered orders
- 32,156 future delivered orders
- 62,145 customers with historical qualifying purchases

The leakage-control structure is:

```text
BEFORE CUTOFF
    ↓
Features
    ↓
MODEL
    ↓
AFTER CUTOFF
    ↓
Target
```

## Feature Engineering

All customer features were calculated from historical delivered orders only.

### Recency

```text
Recency = Prediction Cutoff - Last Historical Purchase
```

The result was converted from a Pandas `Timedelta` to a numeric number of days.

### Frequency

Frequency is the number of unique historical delivered orders for each customer.

The historical population was heavily concentrated at one order:

- 60,324 customers had 1 historical order
- 1,680 had 2
- 114 had 3
- maximum = 9

### Monetary

Because `order_items` is at a lower grain than `orders`, order value was calculated at the order level first:

```text
order_value = price + freight_value
```

Item-level values were summed by `order_id` before being aggregated to the customer level.

Customer monetary value is the sum of historical order values.

This avoids inflating monetary values through lower-grain joins.

### Average Order Value

```text
AOV = Monetary / Frequency
```

The final feature set was:

```text
recency
frequency
monetary
average_order_value
```

The prepared modeling dataset contains 62,145 customers with no missing values in these features.

## Target

The target identifies whether a customer made at least one qualifying purchase during the future observation period.

```text
1 = at least one future delivered purchase
0 = no future delivered purchase
```

Of the 62,145 historical customers:

- 601 made another delivered purchase
- 61,544 did not
- positive-class rate = 0.97%

This severe class imbalance strongly influenced model selection and evaluation.

## Exploratory Analysis

EDA focused on customer purchasing behavior and the variables used for modeling.

The notebook examined:

- customer order frequency
- recency, frequency, monetary value, and AOV distributions
- skewness and extreme values
- feature behavior by repeat-purchase outcome
- boxplots and log-scale monetary distributions
- class balance

Selected observations:

- Repeat purchasers had lower median historical recency than non-repeat customers.
- Repeat purchasers had somewhat higher historical frequency on average.
- Median monetary value was very similar between the two groups.
- Frequency, monetary value, and AOV were strongly right-skewed.

These are descriptive observations and are not treated as causal findings.

## Model

The implemented baseline is Logistic Regression.

It was chosen because it provides:

- an interpretable binary classification baseline
- probability estimates
- a simple benchmark for later model comparison
- compatibility with straightforward feature scaling

The model was implemented as:

```text
StandardScaler
      ↓
Logistic Regression
```

The scaler and model were placed inside a scikit-learn pipeline so preprocessing is learned from the training data only.

Because only 0.97% of observations belonged to the positive class, `class_weight="balanced"` was used to give greater importance to the minority class.

The dataset was split using an 80/20 stratified train/test split with `random_state=42`.

| Split | Customers | Repeat purchasers |
|---|---:|---:|
| Training | 49,716 | 481 |
| Test | 12,429 | 120 |

## Evaluation

Accuracy was not treated as the primary metric. With only 0.97% positive cases, a model could obtain very high accuracy while failing to identify meaningful repeat-purchase behavior.

The baseline was evaluated using:

- Precision
- Recall
- F1-score
- Average Precision
- ROC-AUC
- Confusion matrix

### Baseline Results

| Metric | Result |
|---|---:|
| Precision | 1.36% |
| Recall | 56.67% |
| F1-score | 2.65% |
| Average Precision | 2.18% |
| ROC-AUC | 0.618 |

Confusion matrix at the default 0.50 threshold:

| | Predicted 0 | Predicted 1 |
|---|---:|---:|
| Actual 0 | 7,366 | 4,943 |
| Actual 1 | 52 | 68 |

The model identified 68 of the 120 repeat purchasers in the test set, but generated 4,943 false positives. This produced very low precision despite moderate recall.

The baseline therefore shows some predictive signal, but limited separation between repeat and non-repeat customers.

## Threshold Analysis

Threshold analysis was performed because the model's predicted probabilities can be converted into binary predictions at different probability cutoffs.

| Threshold | Precision | Recall | F1 |
|---:|---:|---:|---:|
| 0.50 | 1.36% | 56.67% | 2.65% |
| 0.40 | 1.00% | 97.50% | 1.99% |
| 0.30 | 0.97% | 100.00% | 1.93% |
| 0.20 | 0.97% | 100.00% | 1.92% |
| 0.10 | 0.97% | 100.00% | 1.91% |
| 0.05 | 0.97% | 100.00% | 1.91% |
| 0.02 | 0.97% | 100.00% | 1.91% |
| 0.01 | 0.97% | 100.00% | 1.91% |

Lowering the threshold increased recall substantially, but precision remained extremely low.

At thresholds of 0.30 and below, all repeat purchasers were identified, but precision stayed close to the underlying 0.97% positive-class rate.

This indicates that threshold adjustment alone could not produce a useful precision-recall trade-off for this baseline.

## Key Findings

1. The project successfully converted transactional Olist data into a customer-level, temporally structured prediction problem.
2. RFM + AOV features contain some predictive signal, reflected in ROC-AUC of 0.618 and Average Precision of 2.18%.
3. Severe class imbalance makes accuracy misleading for this problem.
4. The baseline achieved 56.67% recall but only 1.36% precision at the default threshold.
5. Threshold tuning increased recall but did not materially improve precision.
6. The results indicate that better behavioral features and model comparison are more promising next steps than threshold tuning alone.

## Limitations

- Olist represents historical Brazilian e-commerce transactions rather than a subscription business.
- Repeat purchase is observed behavior during a defined future period, not contractual churn.
- Most historical customers have only one qualifying purchase, limiting frequency variation.
- The dataset covers a finite historical period.
- RFM and AOV do not capture every factor influencing future purchasing.
- Model predictions describe patterns in the available data and do not establish causality.
- Predicted probabilities are estimates, not guarantees.
- The baseline is not sufficiently precise for practical customer targeting without further development.

## Future Improvements

The current repository intentionally stops at the interpretable baseline stage.

Potential future work includes:

- testing log transformations for strongly skewed features
- investigating relationships and potential multicollinearity among monetary, frequency, and AOV features
- adding temporally valid features such as customer tenure, category diversity, and time between purchases
- comparing Logistic Regression with Random Forest and XGBoost
- performing deeper false-positive and false-negative analysis
- evaluating business-specific probability thresholds
- exploring customer segmentation and broader purchase-propensity analysis

These are future possibilities, not implemented components of the current project.

## Repository Structure

```text
OLIST-Customer-Purchase-Analytics/
│
├── data/
│   ├── cleaned/
│   ├── database/
│   └── processed/
│       └── modeling_data.csv
│
├── notebooks/
│   ├── 01_Customer_Data_Preparation.ipynb
│   ├── 02_Model_Training_&_Evaluation.ipynb
│   └── README.md
│
├── .gitignore
├── LICENSE
├── grain_level.png
├── OLIST_ER_Diagram.png
├── README.md
└── requirements.txt
```

## Notebooks

### [01_Customer_Data_Preparation.ipynb](https://github.com/ReinSoup/OLIST-Customer-Purchase-Analytics/blob/main/notebooks/01_Customer_Data_Preparation.ipynb)

Covers:

- loading cleaned customer, order, and order-item data
- validating customer identifiers
- examining order statuses and purchase dates
- defining qualifying purchases
- establishing the temporal framework
- calculating order-level monetary value
- constructing customer-level RFM and AOV features
- creating the future repeat-purchase target
- exploratory analysis
- exporting the modeling dataset

### [02_Model_Training_&_Evaluation.ipynb](https://github.com/ReinSoup/OLIST-Customer-Purchase-Analytics/blob/main/notebooks/02_Model_Training_%26_Evaluation.ipynb)

Covers:

- loading the prepared modeling dataset
- separating features and target
- checking class imbalance
- stratified train/test splitting
- StandardScaler + Logistic Regression
- probability generation
- classification metrics
- confusion matrix
- threshold analysis
- findings and future improvements

## Technologies

- Python
- pandas
- NumPy
- Matplotlib
- Seaborn
- scikit-learn
- SQLite
- Jupyter Notebook
- Git / GitHub

## Reproducibility

The notebooks use repository-relative paths rather than machine-specific absolute paths.

To reproduce the workflow:

1. Clone the repository.
2. Create and activate a Python virtual environment.
3. Install dependencies from `requirements.txt`.
4. Ensure the cleaned Olist data is available under `data/cleaned/`.
5. Run `01_Customer_Data_Preparation.ipynb`.
6. Run `02_Model_Training_&_Evaluation.ipynb`.

The first notebook generates:

```text
data/processed/modeling_data.csv
```

which is consumed by the modeling notebook.

## Related Project

This project builds on the [Olist Data Quality & ETL Pipeline](https://github.com/ReinSoup/OLIST-Data-Quality-ETL).

That project covers raw-data inspection, data-quality assessment, cleaning, validation, and SQLite database creation.

This repository focuses on the downstream analytical problem: using that validated foundation to construct customer-level behavioral features and predict future repeat purchases.

## Project Repository

[GitHub Repository](https://github.com/ReinSoup/OLIST-Customer-Purchase-Analytics)
