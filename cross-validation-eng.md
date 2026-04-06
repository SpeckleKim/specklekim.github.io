---
layout: default
title: Cross-Validation & Model Selection
lang: en
---

# Cross-Validation & Model Selection

Cross-validation is a fundamental technique for assessing model performance and selecting the best model architecture. Unlike simple train-test splits, cross-validation provides more robust estimates of generalization error and reduces variance in performance estimates.

## The Problem with Simple Train-Test Split

When you split data once into training and testing sets, your estimate of model performance depends heavily on which samples end up in each set. Different random splits can yield very different performance estimates, especially with limited data. This high variance in the estimate makes it difficult to compare models reliably.

## Train-Validation-Test Split

The three-way split is a standard approach:
- **Training set** (60-70%): Used to fit model parameters
- **Validation set** (10-20%): Used to tune hyperparameters and select features
- **Test set** (10-20%): Reserved for final evaluation, touched only once

The validation set helps prevent overfitting to the test set during model development.

## k-Fold Cross-Validation

k-Fold CV repeatedly partitions the data into k equal-sized folds and trains k models:

1. Fold 1: Train on folds 2-k, validate on fold 1
2. Fold 2: Train on folds 1,3-k, validate on fold 2
3. ... and so on

Final score is the average of all k validation scores. Common values are k=5 or k=10. Benefits:
- Uses all data for both training and validation
- Reduces variance of performance estimate
- Better for small datasets where train-test split wastes data

The standard error of the CV estimate is approximately `σ / √k`, where σ is the standard deviation of fold scores.

## Stratified k-Fold Cross-Validation

For classification problems with imbalanced classes, stratified k-fold ensures each fold maintains approximately the same class distribution as the original dataset. This prevents situations where a fold accidentally contains mostly one class, which would give misleading performance estimates.

Example: If your dataset is 90% class A and 10% class B, each fold should also be roughly 90%-10% distributed.

## Leave-One-Out Cross-Validation (LOOCV)

LOOCV is an extreme case where k = n (number of samples). Train on n-1 samples, validate on 1 sample, repeat n times. Advantages:
- Uses maximum training data per iteration
- Deterministic (no randomness in fold selection)
- Gives almost unbiased estimate of generalization error

Disadvantages:
- Extremely expensive computationally (n model fits)
- High variance because folds overlap heavily
- Rarely practical except for small datasets or simple models

## Time Series Cross-Validation

Standard k-fold is inappropriate for time series because it violates temporal ordering. Time series CV uses a forward-chaining approach:

**Walk-forward validation:**
- Fold 1: Train on [1-100], test on [101-150]
- Fold 2: Train on [1-150], test on [151-200]
- Fold 3: Train on [1-200], test on [201-250]

Training set always comes before test set temporally. This respects the causal structure of time series data.

## Model Comparison Strategies

### Between-Model Comparison
Cross-validation allows fair comparison of different model architectures:
- Model A: mean CV score = 0.855, std = 0.021
- Model B: mean CV score = 0.847, std = 0.028

Use statistical tests (e.g., paired t-test on fold scores) to determine if differences are significant.

### Nested Cross-Validation
When both selecting hyperparameters AND comparing models, use nested CV to prevent data leakage:
- **Outer CV** (e.g., 5-fold): For model comparison
- **Inner CV** (e.g., 5-fold): For hyperparameter tuning within each outer fold

The outer folds are completely untouched during hyperparameter optimization, ensuring unbiased comparison.

## Information Criteria: AIC and BIC

For model selection in parametric models, information criteria balance fit and complexity:

**Akaike Information Criterion (AIC):**
```
AIC = 2k - 2 ln(L)
```
where k = number of parameters, L = maximum likelihood

**Bayesian Information Criterion (BIC):**
```
BIC = k ln(n) - 2 ln(L)
```
where n = number of observations

Lower values are better. BIC penalizes complexity more heavily than AIC (especially for large n), making it more conservative about model complexity.

AIC is better for prediction; BIC is better for finding the "true" model. Use AIC when your goal is forecasting, BIC when you want interpretable sparse models.

## Practical Recommendations

1. **Small datasets (n < 1000):** Use stratified 5-fold or 10-fold CV
2. **Medium datasets (1000 < n < 100,000):** Use 5-fold CV
3. **Large datasets (n > 100,000):** Single train-validation-test split is sufficient
4. **Imbalanced classification:** Always use stratified k-fold
5. **Time series:** Always use forward-chaining or expanding window CV
6. **Hyperparameter tuning:** Use nested CV or hold out a separate validation set

## Common Pitfalls

- **Data leakage:** Preprocessing (scaling, PCA) BEFORE splitting folds contaminates validation estimates
- **Optimizing on CV scores:** Using CV scores as hyperparameters selects for noise; use a held-out test set for final evaluation
- **Too few folds:** k < 3 gives high-variance estimates
- **Ignoring temporal order:** Using random k-fold on time series data guarantees overly optimistic estimates

Cross-validation is not optional—it's essential for reliable model selection and honest performance reporting.
