# Notebooks

This folder contains the two Jupyter notebooks that implement the customer purchase prediction workflow.

The notebooks are intentionally separated into two stages:

```text
01_Customer_Data_Preparation
            ↓
    modeling_data.csv
            ↓
02_Model_Training_&_Evaluation
```

The first notebook prepares and validates the customer-level modeling dataset. The second notebook uses that dataset to train and evaluate the baseline prediction model.

## Notebook Index

| Notebook | Purpose |
|---|---|
| [01_Customer_Data_Preparation.ipynb](https://github.com/ReinSoup/OLIST-Customer-Purchase-Analytics/blob/main/notebooks/01_Customer_Data_Preparation.ipynb) | Customer-level data preparation, temporal split, feature engineering, target creation, and EDA |
| [02_Model_Training_&_Evaluation.ipynb](https://github.com/ReinSoup/OLIST-Customer-Purchase-Analytics/blob/main/notebooks/02_Model_Training_%26_Evaluation.ipynb) | Logistic Regression training, evaluation, and threshold analysis |

---

## 01. Customer Data Preparation

[Open notebook](https://github.com/ReinSoup/OLIST-Customer-Purchase-Analytics/blob/main/notebooks/01_Customer_Data_Preparation.ipynb)

### Purpose

This notebook transforms the cleaned Olist transactional data into a customer-level dataset suitable for predictive modeling.

The main objective is to make sure that customer behavior is defined correctly before any machine learning is performed.

### Workflow

```text
Cleaned Olist tables
        ↓
Data inspection
        ↓
Customer identity validation
        ↓
Qualifying purchase definition
        ↓
Temporal split
        ↓
Order-level monetary value
        ↓
Customer-level RFM + AOV
        ↓
Future repeat-purchase target
        ↓
EDA and validation
        ↓
modeling_data.csv
```

### Main Steps

#### 1. Load the cleaned data

The notebook loads:

- `customers`
- `orders`
- `order_items`

from the cleaned Olist data.

#### 2. Validate customer identity

The notebook examines the relationship between:

- `customer_id`
- `customer_unique_id`

`customer_unique_id` is selected as the customer-level identifier because multiple `customer_id` records can belong to the same underlying customer.

#### 3. Define a qualifying purchase

Only orders with:

```text
order_status == "delivered"
```

are treated as qualifying purchases.

This keeps the analysis focused on completed purchasing behavior.

#### 4. Establish the temporal framework

The prediction cutoff is:

```text
2018-03-31
```

Historical purchases are used to construct customer features.

Future delivered purchases from:

```text
2018-04-01 → 2018-09-30
```

are used to create the repeat-purchase target.

This separation prevents future purchasing behavior from leaking into the historical features.

#### 5. Calculate order value

Because `order_items` contains multiple rows per order, item-level monetary values are first aggregated to the order level:

```text
order_value = price + freight_value
```

The resulting order value is then joined to the historical orders.

This preserves the correct data grain before customer-level aggregation.

#### 6. Build customer-level features

The notebook constructs:

- Recency
- Frequency
- Monetary
- Average Order Value

The final feature table has:

```text
1 row = 1 customer
```

#### 7. Create the target

The target is:

```text
1 = customer made at least one future delivered purchase
0 = customer did not
```

The resulting dataset contains 62,145 customers, of which 601 are repeat purchasers.

#### 8. Perform exploratory analysis

The notebook examines:

- feature distributions
- descriptive statistics
- skewness
- class balance
- feature behavior by repeat-purchase outcome
- boxplots for RFM and AOV variables

#### 9. Export the modeling dataset

The final customer-level dataset is saved as:

```text
data/processed/modeling_data.csv
```

This file is the input to the second notebook.

---

## 02. Model Training & Evaluation

[Open notebook](https://github.com/ReinSoup/OLIST-Customer-Purchase-Analytics/blob/main/notebooks/02_Model_Training_%26_Evaluation.ipynb)

### Purpose

This notebook takes the prepared customer-level dataset and establishes an interpretable baseline for predicting future repeat purchases.

The notebook focuses on model training, evaluation, and understanding the effect of classification thresholds.

### Workflow

```text
modeling_data.csv
        ↓
Feature / target separation
        ↓
Class imbalance check
        ↓
Stratified train/test split
        ↓
StandardScaler
        ↓
Logistic Regression
        ↓
Predictions + probabilities
        ↓
Classification metrics
        ↓
Confusion matrix
        ↓
Threshold analysis
```

### Main Steps

#### 1. Define features and target

The model uses:

```text
recency
frequency
monetary
average_order_value
```

The target is:

```text
repeat_purchase
```

The target is kept outside the feature matrix to avoid target leakage.

#### 2. Check class imbalance

The dataset contains:

- 61,544 non-repeat customers
- 601 repeat customers

The positive class therefore represents approximately 0.97% of the modeling population.

This makes class imbalance a central modeling consideration.

#### 3. Create the train/test split

An 80/20 split is used with stratification so that the minority class remains represented in both datasets.

```text
Training: 49,716 customers
Test:     12,429 customers
```

#### 4. Train the baseline model

The baseline is:

```text
StandardScaler
      ↓
Logistic Regression
```

`class_weight="balanced"` is used so the minority class receives greater importance during training.

The preprocessing and model are combined into a scikit-learn pipeline.

#### 5. Generate predictions and probabilities

The notebook generates:

- binary predictions
- predicted probabilities for the repeat-purchase class

The probabilities are required for threshold analysis and ranking-based evaluation.

#### 6. Evaluate model performance

The notebook calculates:

- Precision
- Recall
- F1-score
- Average Precision
- ROC-AUC
- Confusion matrix

Accuracy is not used as the main evaluation metric because of the extreme class imbalance.

#### 7. Analyze classification thresholds

The notebook evaluates thresholds from:

```text
0.50
0.40
0.30
0.20
0.10
0.05
0.02
0.01
```

For each threshold, precision, recall, and F1-score are compared.

The analysis shows that lowering the threshold increases recall substantially, but precision remains extremely low.

#### 8. Document model limitations

The notebook concludes that threshold adjustment alone does not solve the baseline model's limitations.

It identifies future directions including:

- transformations for skewed features
- additional behavioral features
- investigation of relationships between monetary variables
- nonlinear models such as Random Forest and XGBoost
- deeper error analysis
- business-specific threshold selection

---

## Notebook Dependency

The notebooks are designed to be run in order.

### Step 1

Run:

`01_Customer_Data_Preparation.ipynb`

This creates:

```text
data/processed/modeling_data.csv
```

### Step 2

Run:

`02_Model_Training_&_Evaluation.ipynb`

This loads the prepared modeling dataset and performs the modeling and evaluation workflow.

Running the second notebook before the first may fail if `modeling_data.csv` has not already been created.

## Design Principles

The notebooks prioritize:

- correct data grain
- temporal validity
- prevention of data leakage
- interpretable feature engineering
- simple, explainable modeling
- evaluation appropriate for class imbalance
- reproducible repository-relative paths

The notebooks intentionally stop at a Logistic Regression baseline rather than introducing model complexity without first establishing whether the underlying feature set contains useful predictive information.

## Related Project

The cleaned data used by this project originates from the:

[Olist Data Quality & ETL Pipeline](https://github.com/ReinSoup/OLIST-Data-Quality-ETL)

That project focuses on data inspection, cleaning, validation, and database preparation.

These notebooks build the downstream customer analytics and predictive modeling layer on top of that foundation.
