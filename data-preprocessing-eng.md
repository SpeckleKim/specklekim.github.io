---
layout: default
title: "Data Preprocessing & Pipelines"
lang: en
---

# Data Preprocessing & Pipelines

## Introduction

Data preprocessing transforms raw, messy data into clean, structured formats suitable for ML. The adage "garbage in, garbage out" reflects reality: even sophisticated models fail on poor-quality data. Preprocessing often comprises 60-80% of a data scientist's time, yet it's essential for model performance and robustness.

## Data Cleaning Fundamentals

### Handling Missing Values

Missing values (NULL, NaN) are ubiquitous in real-world datasets:

**Strategies**:
- **Deletion**: Remove rows/columns with missing values (simple but loses information)
- **Imputation**: Replace with statistical values (mean, median, mode)
- **Forward fill/Backward fill**: Use previous/next values (for time series)
- **Advanced imputation**: KNN, regression, matrix factorization (preserve relationships)
- **Multiple imputation**: Generate multiple complete datasets, analyze all, combine results

**Considerations**:
- Missing completely at random (MCAR) vs missing at random (MAR)
- Domain knowledge: Some missing values have meaning (sensor failure, no answer)
- Impact on downstream analysis

### Outlier Detection and Treatment

Outliers are extreme values that deviate from the rest:

**Detection Methods**:
- **Statistical**: Z-score (|z| > 3), IQR (values outside 1.5×IQR)
- **Distance-based**: Isolation Forest, Local Outlier Factor
- **Model-based**: Autoencoders, Gaussian mixture models

**Treatment**:
- **Deletion**: Remove outliers (risky if they're informative)
- **Capping**: Replace with percentile values (95th, 99th)
- **Transformation**: Log/Box-Cox to compress range
- **Separate modeling**: Create separate models for outliers
- **Investigation**: Understand root cause; may indicate data quality issues

## Data Normalization and Encoding

### Normalization

Scaling features to comparable ranges improves gradient descent and regularization:

**Methods**:
- **Min-Max Scaling**: X_scaled = (X - min) / (max - min), range [0, 1]
- **Z-score Normalization**: X_scaled = (X - mean) / std, range approximately [-3, 3]
- **Robust Scaling**: Uses median and IQR, resistant to outliers
- **Log Scaling**: For skewed distributions, apply log(1 + X)

**When to normalize**:
- Always for: gradient-based methods (neural networks, SVM, logistic regression)
- Optional for: tree-based models (they're scale-invariant)
- Critical for: distance metrics (KNN, K-means)

### Categorical Encoding

Converting categorical variables to numerical:

**One-Hot Encoding**: Create binary column per category (for tree models, small cardinality)

**Label Encoding**: Map categories to integers 0, 1, 2... (for tree models only)

**Target Encoding**: Replace category with target variable mean (requires cross-validation to avoid leakage)

**Embedding**: Learn dense vectors for categories (for neural networks)

**Ordinal Encoding**: For ordered categories (small, medium, large → 1, 2, 3)

## Data Pipelines

### Airflow

Apache Airflow orchestrates complex data workflows:

```python
from airflow import DAG
from airflow.operators.bash import BashOperator

dag = DAG('data_pipeline', default_args=default_args)
extract = BashOperator(task_id='extract', bash_command='python extract.py')
transform = BashOperator(task_id='transform', bash_command='python transform.py')
extract >> transform
```

Features: Directed acyclic graphs (DAGs), scheduling, monitoring, retries, backfill.

### Apache Spark

Distributed data processing framework:
- Handles datasets larger than single machine memory
- RDDs and DataFrames for structured processing
- Supports SQL, machine learning, streaming
- 10-100x faster than Hadoop MapReduce

### Great Expectations

Data validation framework ensuring quality:
- Define expectations: "column X should be non-null", "Y should be in range [0, 100]"
- Validate data against expectations
- Generate data docs for documentation
- Integrates with orchestration tools

## Data Versioning

Tracking data versions like code versions ensures reproducibility:

**Tools**:
- **DVC (Data Version Control)**: Track large data files, separate from code
- **Delta Lake**: ACID transactions for data lakes
- **Git LFS**: Large file storage for Git

**Benefits**:
- Reproduce previous model versions with exact same data
- Understand what changed between versions
- Roll back to previous data if needed
- Collaborate without storage conflicts

## Feature Engineering

Converting raw data into meaningful features:

**Approaches**:
- **Domain knowledge**: Create features from business logic
- **Statistical**: Aggregate functions (sum, mean, std)
- **Temporal**: Lag features, rolling windows for time series
- **Interaction**: Multiply or combine features
- **Polynomial**: Create squared, cubic features

**Best practices**:
- Start simple, add complexity if needed
- Validate feature importance
- Avoid leakage: don't use future information
- Document feature definitions for reproducibility

## Data Quality Metrics

**Completeness**: Percentage of non-missing values

**Consistency**: Data type agreement, format standardization

**Accuracy**: Correctness relative to source of truth

**Timeliness**: How current is the data

**Uniqueness**: Duplicate record rate

## Best Practices

- **Automate**: Scripts > manual Excel work
- **Document**: Record assumptions, transformations, thresholds
- **Version**: Track data and preprocessing code changes
- **Test**: Validate data against expectations
- **Monitor**: Alert on data quality issues in production
- **Iterate**: Quality issues discovered in modeling → pipeline improvements

## Conclusion

Data preprocessing is unglamorous but critical work. Investing in robust pipelines, validation, and versioning pays dividends in model reliability and team productivity. The 80-20 rule applies: 80% of value comes from clean, well-engineered data; 20% from fancy algorithms.
