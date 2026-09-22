# Olist Customer Purchase Analytics

A customer analytics and machine learning project built as a
continuation of my Olist Data Quality & ETL work.

The project uses the cleaned and validated Olist e-commerce data to move
from data quality and preparation into customer-level analysis and
predictive modeling. The initial submission focuses on predicting
whether a customer will make another purchase during a defined future
observation period.

The longer-term project is planned to expand this foundation into
customer purchase propensity and behavioral segmentation.

------------------------------------------------------------------------

## 1. Project Overview

This project is the analytical continuation of my previous Olist
data-quality and ETL project.

The previous project focused on taking the raw Olist dataset through:

``` text
Raw data
    ↓
Data quality assessment
    ↓
Defensible cleaning
    ↓
Validation
    ↓
SQLite database
```

This project builds on that foundation:

``` text
Validated Olist data
    ↓
Customer-level data preparation
    ↓
Exploratory analysis
    ↓
Feature engineering
    ↓
Repeat purchase prediction
    ↓
Model evaluation
    ↓
Business interpretation
```

The purpose is not simply to train a machine-learning model. The project
is intended to demonstrate the complete reasoning process from a
business question to a measurable target, customer-level features, model
evaluation, and actionable interpretation.

------------------------------------------------------------------------

## 2. Business Problem

E-commerce businesses need to understand which customers are likely to
return after making a purchase.

The central question for the initial project is:

> Can historical customer purchasing behavior be used to predict whether
> a customer will make another purchase during a future observation
> period?

This is framed as a repeat-purchase problem rather than subscription
churn because the Olist dataset represents an e-commerce marketplace and
does not contain a contractual subscription relationship.

The prediction is therefore about observed purchasing behavior within
the available historical data.

------------------------------------------------------------------------

## 3. Project Objectives

The initial submission will focus on the following objectives:

1.  Build a customer-level analytical dataset from the validated Olist
    data.
2.  Understand customer purchasing behavior through exploratory data
    analysis.
3.  Engineer behavioral features, beginning with RFM-style measures.
4.  Define a temporally valid repeat-purchase target.
5.  Train an interpretable baseline classification model.
6.  Evaluate the model using classification metrics rather than relying
    only on accuracy.
7.  Examine the resulting predictions from a business perspective.
8.  Document limitations and possible future extensions.

### Planned future objectives

After the initial submission, the project is planned to expand to:

-   Compare the baseline model with a nonlinear model such as Random
    Forest.
-   Develop customer behavioral segmentation using RFM features and
    K-Means.
-   Use predicted probabilities as a customer purchase-propensity
    measure.
-   Examine relationships between customer segments and predicted
    purchase propensity.
-   Develop a customer intelligence dashboard.
-   Investigate business exposure and customer-level patterns in greater
    depth.

These extensions are intentionally kept outside the initial submission
scope so that the core prediction problem can be defined and validated
properly before additional complexity is introduced.

------------------------------------------------------------------------

## 4. Relationship to the Previous Olist Project

This repository is a continuation of my existing Olist Data Quality &
ETL project.

Previous project:

**Olist Data Quality & ETL Pipeline**

Repository: https://github.com/ReinSoup/OLIST-Data-Quality-ETL

That project established the data foundation by inspecting, cleaning,
validating, and structuring the nine related Olist datasets.

The current project uses that work as its starting point rather than
repeating the complete ETL process.

The overall portfolio progression is:

``` text
PROJECT 1
Olist Data Quality & ETL
        ↓
Data inspection
Data quality assessment
Cleaning
Validation
SQLite database
        ↓
PROJECT 2
Olist Customer Purchase Analytics
        ↓
Customer analysis
Feature engineering
Repeat purchase prediction
Model evaluation
        ↓
Future expansion
Purchase propensity
Customer segmentation
Dashboard
```

This separation keeps the two projects focused while showing how a
validated data foundation can support downstream analytics and machine
learning.

------------------------------------------------------------------------

## 5. Dataset

The project uses the Brazilian E-Commerce Public Dataset by Olist.

Source:

https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce

The original dataset contains approximately 100,000 orders from
2016--2018 and consists of nine related source tables.

The previous ETL project established the following data foundation:

  Dataset component             Result
  -------------------------- ---------
  Related source tables              9
  Orders                        99,441
  Order items                  112,650
  Product records               32,951
  Sellers                        3,095
  Cleaned geolocation rows     738,332
  SQLite tables                      9

The complete data-quality findings and cleaning decisions remain
documented in the previous ETL project.

------------------------------------------------------------------------

## 6. Data Model and ER Diagram

The Olist dataset is relational rather than a collection of independent
CSV files.

The ER diagram below represents the underlying relationships between the
nine source tables and is retained in this project because understanding
those relationships is necessary when constructing the customer-level
analytical dataset.

![Olist ER Diagram](OLIST_ER_Diagram.png)

The primary tables expected to be relevant to the initial prediction
problem are:

-   `customers`
-   `orders`
-   `order_items`

Additional tables will only be incorporated when they provide a clear
analytical purpose.

This is intentional. Joining every available table is not necessary for
a customer-level prediction problem and can introduce duplicated rows or
unclear analytical grain.

------------------------------------------------------------------------

## 7. Customer Identity

Customer-level analysis requires careful treatment of the Olist customer
identifiers.

The project will investigate and use the appropriate customer identifier
for tracking purchasing behavior across orders, with particular
attention to `customer_unique_id`.

The analytical dataset should represent:

> One row = one customer

rather than one row per order or one row per order item.

This distinction is important because Olist contains one-to-many
relationships between customers, orders, order items, payments, reviews,
and other entities.

Repeated identifiers in child tables are therefore not automatically
evidence of duplicate records.

------------------------------------------------------------------------

## 8. Analytical Grain

The final modeling dataset will use customer-level grain.

The intended structure is:

``` text
Customer
    ↓
Historical orders
    ↓
Customer behavioral features
    ↓
Future purchase behavior
    ↓
Repeat purchase target
```

The modeling table is expected to contain one record per eligible
customer.

The exact population definition will be documented after the data is
inspected and the temporal prediction framework is finalized.

**Final eligible-customer count:**\
\[To be completed\]

------------------------------------------------------------------------

## 9. Project Methodology

The project will follow a chronological workflow so that the modeling
process remains reproducible and easy to audit.

### Stage 1: Load the validated data

The project will begin with the cleaned Olist datasets produced by the
previous data-quality workflow.

The relevant tables will be loaded and inspected before any analytical
joins are performed.

### Stage 2: Understand the data

The analysis will inspect:

-   Table dimensions
-   Column names and data types
-   Order statuses
-   Purchase date range
-   Customer identifier relationships
-   Orders per customer
-   Order value distribution
-   Missing values relevant to the analysis

This stage will determine which records and fields are appropriate for
the prediction problem.

### Stage 3: Exploratory Data Analysis

EDA will focus on customer purchasing behavior rather than reproducing
the full data-quality analysis from the previous project.

Planned questions include:

-   How many customers have one order?
-   How many customers have multiple orders?
-   What does the distribution of orders per customer look like?
-   How does order value vary?
-   How does purchasing behavior change over time?
-   What proportion of eligible customers make another purchase?

Final findings will be added after the analysis is completed.

### Stage 4: Define the temporal prediction framework

The project will separate historical information from future behavior.

The intended structure is:

``` text
Historical feature period
        ↓
Prediction cutoff
        ↓
Future observation period
        ↓
Repeat purchase target
```

Customer features must be calculated using information available before
the prediction cutoff.

The future observation period will be used only to determine the target.

This separation is necessary to avoid using future information when
training the model.

**Feature period:**\
\[To be completed\]

**Prediction cutoff:**\
\[To be completed\]

**Future observation period:**\
\[To be completed\]

### Stage 5: Define a qualifying purchase

The project will inspect Olist order statuses before deciding which
orders represent a qualifying purchase for the prediction target.

The final rule will be documented here after validation.

**Qualifying purchase definition:**\
\[To be completed\]

This decision will be based on the actual structure and status values in
the dataset rather than being assumed in advance.

### Stage 6: Feature engineering

The initial feature set will focus on interpretable measures of customer
behavior.

Planned features include:

  Feature               Description
  --------------------- -----------------------------------------------------------
  Recency               Time since the customer's most recent historical purchase
  Frequency             Number of qualifying historical orders
  Monetary              Historical customer spending
  Average Order Value   Historical monetary value per qualifying order

Additional features may be introduced if they provide clear business
value and can be calculated without leakage.

Potential extensions include:

-   Customer tenure
-   Number of product categories purchased
-   Average time between orders

The final feature list will be recorded after feature engineering is
completed.

**Final feature list:**\
\[To be completed\]

### Stage 7: Define the target

The initial target will be binary:

``` text
1 = customer makes another qualifying purchase
0 = customer does not make another qualifying purchase
```

The target will be determined exclusively from the future observation
period.

**Final target definition:**\
\[To be completed\]

**Positive class count:**\
\[To be completed\]

**Negative class count:**\
\[To be completed\]

**Positive class proportion:**\
\[To be completed\]

### Stage 8: Prepare the modeling data

Before training, the project will check:

-   Missing values
-   Duplicate customer records
-   Feature distributions
-   Extreme values
-   Class balance
-   Data types
-   Potential target leakage

Numerical features used by Logistic Regression will be appropriately
scaled using information from the training data only.

### Stage 9: Baseline model

The initial submission will use Logistic Regression as the baseline
classification model.

The model is selected because it:

-   Provides an interpretable baseline.
-   Works naturally with binary classification.
-   Produces probability estimates.
-   Allows the relationship between features and the predicted outcome
    to be examined.
-   Provides a useful benchmark for later model comparisons.

The initial submission will deliberately avoid a large collection of
models.

### Stage 10: Model evaluation

The model will be evaluated using:

-   Confusion matrix
-   Precision
-   Recall
-   F1-score
-   ROC-AUC

Accuracy may also be reported for completeness, but it will not be
treated as the sole measure of model quality.

The appropriate interpretation of each metric will be discussed in the
context of the repeat-purchase problem.

**Final model metrics:**\
\[To be completed\]

### Stage 11: Business interpretation

The final stage of the submission will translate model results into
customer-level observations.

The analysis will investigate questions such as:

-   Which historical customer behaviors are associated with repeat
    purchasing?
-   Which customers receive higher predicted probabilities?
-   Where does the model make incorrect predictions?
-   What limitations exist in using historical Olist behavior to predict
    future purchases?

**Key findings:**\
\[To be completed\]

------------------------------------------------------------------------

## 10. Leakage Prevention

Temporal leakage is a major methodological concern in this project.

The model must not use information that occurs after the prediction
cutoff when calculating its features.

The intended separation is:

``` text
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

For example, a customer's future purchase cannot be used when
calculating their historical frequency or monetary value.

The notebook will explicitly document the cutoff and the fields used in
each period.

Any feature found to contain future information will be excluded or
redesigned.

------------------------------------------------------------------------

## 11. Model Outputs

The initial model is expected to produce:

-   Binary repeat-purchase predictions
-   Repeat-purchase probabilities
-   Evaluation metrics
-   Confusion matrix
-   Customer-level prediction results

A planned prediction table is:

  ----------------------------------------------------------------------------------------------------------------------------------
  customer_unique_id      recency   frequency   monetary   avg_order_value   repeat_purchase_probability   predicted_repeat_purchase
  -------------------- ---------- ----------- ---------- ----------------- ----------------------------- ---------------------------
  \[Customer\]              \[ \]       \[ \]      \[ \]             \[ \]                         \[ \]                       \[ \]

  ----------------------------------------------------------------------------------------------------------------------------------

This table will be populated after the model has been trained and
evaluated.

------------------------------------------------------------------------

## 12. Results

This section will be completed after the analysis and model have been
run.

### Dataset

**Eligible customers:** \[To be completed\]

**Historical orders:** \[To be completed\]

**Future qualifying purchases:** \[To be completed\]

### Target distribution

  Class                  Count   Percentage
  -------------------- ------- ------------
  Repeat purchase        \[ \]        \[ \]
  No repeat purchase     \[ \]        \[ \]

### Model performance

  Metric        Logistic Regression
  ----------- ---------------------
  Precision                   \[ \]
  Recall                      \[ \]
  F1-score                    \[ \]
  ROC-AUC                     \[ \]
  Accuracy                    \[ \]

### Key findings

1.  \[To be completed\]
2.  \[To be completed\]
3.  \[To be completed\]

No model performance values will be added to this README until they have
been produced from the final notebook.

------------------------------------------------------------------------

## 13. Repository Structure

The repository will be organized around the analytical workflow rather
than duplicating the entire previous ETL project.

Planned structure:

``` text
OLIST-Customer-Purchase-Analytics/
│
├── data/
│   ├── raw/
│   ├── cleaned/
│   └── database/
│
├── notebooks/
│   ├── 01_Customer_Data_Preparation.ipynb
│   ├── 02_Customer_EDA.ipynb
│   └── 03_Repeat_Purchase_Prediction.ipynb
│
├── OLIST_ER_Diagram.png
├── .gitignore
├── LICENSE
├── requirements.txt
└── README.md
```

The exact notebook structure may be adjusted if a simpler organization
improves reproducibility.

The previous ETL notebooks are maintained in the original Olist
data-quality repository rather than being reproduced as part of this
project's analytical workflow.

------------------------------------------------------------------------

## 14. Planned Notebook Responsibilities

### `01_Customer_Data_Preparation.ipynb`

Planned responsibilities:

-   Load validated Olist data.
-   Inspect relevant tables.
-   Confirm customer identifier relationships.
-   Inspect order statuses and dates.
-   Select the appropriate records.
-   Establish the analytical grain.
-   Build the customer-level dataset.
-   Define the temporal feature and target periods.
-   Engineer initial customer features.
-   Save the modeling dataset if required.

### `02_Customer_EDA.ipynb`

Planned responsibilities:

-   Analyze customer order frequency.
-   Examine repeat-purchase behavior.
-   Analyze recency, frequency, and monetary distributions.
-   Examine order value and customer behavior over time.
-   Investigate class balance.
-   Identify patterns relevant to feature engineering.

### `03_Repeat_Purchase_Prediction.ipynb`

Planned responsibilities:

-   Load the prepared customer-level dataset.
-   Separate features and target.
-   Perform final preprocessing.
-   Split the modeling data appropriately.
-   Train Logistic Regression.
-   Generate predictions and probabilities.
-   Evaluate the model.
-   Analyze errors.
-   Produce customer-level prediction outputs.
-   Record final findings.

------------------------------------------------------------------------

## 15. Data Quality Foundation

The analytical project inherits its data-quality foundation from the
previous Olist ETL project.

The previous workflow identified and documented issues including:

-   261,831 exact duplicate geolocation rows.
-   8 delivered orders missing customer delivery dates.
-   789 repeated `review_id` values across 1,603 rows.
-   610 products missing category information.
-   2 products missing all four physical dimensions.
-   Potential geographic inconsistencies.
-   Legitimate one-to-many repetition across order-related tables.

The previous project deliberately preserved ambiguous records when a
reliable correction could not be justified.

This principle continues into the current project:

> An unusual value is not automatically a bad value, and a predictive
> model should not compensate for an undefined data problem.

The current project will therefore validate the analytical assumptions
it needs rather than assuming that every cleaned record is automatically
suitable for modeling.

------------------------------------------------------------------------

## 16. Important Data Modeling Considerations

The Olist dataset contains multiple one-to-many relationships.

For example, one order can contain multiple:

-   Order items
-   Payment records
-   Review records

Therefore, joining tables without first considering their grain can
multiply rows and distort customer-level metrics.

The project will explicitly control the analytical grain before
calculating:

-   Frequency
-   Monetary value
-   Average order value
-   Other customer-level features

Where necessary, data will be aggregated to the appropriate level before
being joined to another table.

This is especially important for monetary calculations because joining
multiple child tables directly can unintentionally duplicate transaction
values.

------------------------------------------------------------------------

## 17. Technologies

The initial project is planned to use:

-   Python
-   pandas
-   NumPy
-   Matplotlib
-   Seaborn
-   scikit-learn
-   SQLite
-   Jupyter Notebook
-   Git / GitHub

The implementation will prioritize readable, reproducible code over
unnecessary abstraction.

------------------------------------------------------------------------

## 18. Reproducibility

The intended workflow is:

1.  Obtain the Olist dataset.
2.  Use the cleaned and validated data foundation from the previous ETL
    project.
3.  Run the customer data preparation notebook.
4.  Run the customer EDA notebook.
5.  Run the repeat-purchase prediction notebook.
6.  Review the generated customer-level features and predictions.
7.  Record the final results and findings in this README.

The final notebook sequence and required files will be updated if the
implementation changes during development.

------------------------------------------------------------------------

## 19. Limitations

The following limitations will be considered when interpreting the
results:

-   Olist represents historical Brazilian e-commerce transactions rather
    than a subscription business.
-   A repeat purchase target represents observed purchasing behavior and
    should not be interpreted as contractual customer churn.
-   The dataset covers a finite historical period.
-   Many customers may have limited purchasing history.
-   A customer's historical behavior may not fully explain future
    purchasing decisions.
-   Model predictions describe patterns in the available dataset and do
    not automatically establish causal relationships.
-   A predicted probability is an estimate from the trained model, not a
    guarantee that a customer will purchase.

Additional limitations identified during analysis will be documented
here.

**Additional limitations:**\
\[To be completed\]

------------------------------------------------------------------------

## 20. Future Development

The initial submission intentionally focuses on a single, interpretable
prediction problem.

After the submission, the project is planned to evolve into a broader
customer intelligence analysis.

### Planned Phase 2: Model comparison

A nonlinear model such as Random Forest may be introduced and compared
with the Logistic Regression baseline.

The purpose will be to determine whether additional model complexity
produces a meaningful improvement rather than adding models simply for
variety.

### Planned Phase 3: Customer segmentation

RFM and behavioral features will be used to explore customer segments
with K-Means clustering.

The clustering process will be evaluated using appropriate measures and
interpreted based on the resulting customer behavior rather than
assigning arbitrary customer labels beforehand.

### Planned Phase 4: Purchase propensity

The repeat-purchase probability produced by the classification model
will become the foundation for a broader purchase-propensity analysis.

The analysis will examine how predicted purchase probability varies
across customer behaviors and segments.

### Planned Phase 5: Customer intelligence dashboard

A dashboard may be developed to present:

-   Customer and order overview
-   Repeat-purchase behavior
-   Customer segments
-   Purchase propensity
-   RFM characteristics
-   Customer-level predictions
-   Business observations

These extensions will only be added after the underlying analytical
definitions and model results have been validated.

------------------------------------------------------------------------

## 21. Project Status

### Initial submission scope

  Component                    Status
  ---------------------------- -------------------------------------
  Repository setup             In progress
  Data foundation              Available from previous ETL project
  Customer-level dataset       Planned
  Customer EDA                 Planned
  RFM feature engineering      Planned
  Temporal target definition   Planned
  Repeat purchase target       Planned
  Logistic Regression          Planned
  Model evaluation             Planned
  Business interpretation      Planned
  Final results                Pending

### Future scope

  Component                           Status
  ----------------------------------- ---------
  Random Forest comparison            Planned
  K-Means segmentation                Planned
  Purchase propensity analysis        Planned
  Customer intelligence dashboard     Planned
  Extended business impact analysis   Planned

------------------------------------------------------------------------

## 22. Final Project Outcome

The completed project is intended to demonstrate a progression from
reliable data preparation to customer analytics and predictive modeling.

The final workflow should be:

``` text
Olist raw data
      ↓
Previous ETL and data-quality project
      ↓
Cleaned and validated data
      ↓
Customer-level analytical dataset
      ↓
Exploratory analysis
      ↓
Historical behavioral features
      ↓
Temporal prediction framework
      ↓
Repeat purchase target
      ↓
Logistic Regression
      ↓
Evaluation and error analysis
      ↓
Business interpretation
```

The longer-term project will extend this into:

``` text
Repeat purchase prediction
        +
Customer behavioral segmentation
        +
Purchase propensity
        ↓
Customer Intelligence
```

The objective throughout the project is to keep the analytical reasoning
visible: define the business question, understand the data, control the
grain, prevent leakage, build interpretable features, evaluate the model
appropriately, and only then add additional complexity.

------------------------------------------------------------------------

## 23. Related Project

The data-quality and ETL foundation for this project is documented
separately:

**Olist Data Quality & ETL Pipeline**

https://github.com/ReinSoup/OLIST-Data-Quality-ETL

That project covers the original raw-data inspection, data-quality
assessment, cleaning, validation, and SQLite database creation.

This repository focuses on what comes next: using that validated data to
understand and predict customer purchasing behavior.
