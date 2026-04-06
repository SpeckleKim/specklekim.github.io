---
layout: default
title: Ensemble Methods
lang: en
---

# Ensemble Methods

## Introduction

Ensemble methods combine multiple base learners to create more powerful models than any single learner alone. The core principle: diverse, moderately accurate models can collectively achieve high accuracy through aggregation. Ensemble methods are among the most practical and successful approaches in modern machine learning, powering many state-of-the-art solutions.

## Why Ensembles Work

### Bias-Variance Tradeoff

Ensembles improve performance by reducing both bias and variance:

**Averaging effect:**
```
Var(average) = Var(individual) / n (if uncorrelated)
```

Combining uncorrelated predictions reduces variance significantly.

**Diversity requirement:**
- Identical learners don't help (no variance reduction)
- Diversity comes from different training data, features, or algorithms
- Uncorrelated errors are essential

### Mathematical Intuition

If each learner achieves accuracy p > 0.5 and makes independent errors:

```
Ensemble accuracy ≈ Σ C(n,k) × p^k × (1-p)^(n-k) for k > n/2
```

With sufficient diversity, ensemble accuracy approaches 1 regardless of individual accuracy.

## Bagging (Bootstrap Aggregating)

Bagging trains multiple copies of the same base learner on different bootstrap samples of the training data.

**Process:**
1. Create B bootstrap samples with replacement
2. Train base learner independently on each sample
3. Average predictions:
   - Classification: Majority voting
   - Regression: Average predictions

**Key insight:**
- Reduces variance through averaging
- Parallelizable (independent training)
- Works best with high-variance, low-bias learners
- [Random forests](/decision-trees-eng.html) extend bagging with feature randomness

**When to use:**
- [Decision trees](/decision-trees-eng.html) (reduce overfitting)
- Complex, unstable models
- Computational resources available

## Boosting

Boosting sequentially trains learners, where each new learner focuses on examples misclassified by previous ones. This increases diversity and reduces both bias and variance.

### AdaBoost (Adaptive Boosting)

**Algorithm:**
1. Initialize equal weights for all samples
2. For each iteration:
   - Train weak learner on weighted samples
   - Compute weighted error rate
   - Calculate learner weight (lower error → higher weight)
   - Update sample weights (increase for misclassified samples)
3. Final prediction: weighted vote

**Key formula:**
```
w_m = (1/2) × ln((1 - ε_m) / ε_m)
```

Where ε_m is the weighted error of learner m.

**Advantages:**
- Reduces bias effectively
- Theoretically grounded
- Works with any weak learner

**Disadvantages:**
- Sequential (not parallelizable)
- Sensitive to noise and outliers

### Gradient Boosting

[Gradient Boosting](/gradient-boosting-eng.html) fits each new learner to residuals (errors) of previous learners using [gradient descent](/calculus-eng.html):

**Process:**
1. Initialize with average prediction
2. For each iteration:
   - Compute residuals from current predictions
   - Fit new learner to residuals
   - Add to ensemble with step size ([learning rate](/learning-rate-scheduling-eng.html))
3. Final prediction: sum of all learners

**Key advantage:**
```
F(x) = F_(m-1)(x) + η × h_m(x)
```

Where η is the [learning rate](/learning-rate-scheduling-eng.html) (0 < η ≤ 1) controlling contribution.

**Popular variants:**
- **XGBoost**: Optimized [gradient boosting](/gradient-boosting-eng.html) with [regularization](/regularization-eng.html)
- **LightGBM**: Fast, memory-efficient [gradient boosting](/gradient-boosting-eng.html)
- **CatBoost**: Handles categorical features naturally

**Advantages:**
- Powerful, often best single algorithm
- Flexible, handles various [loss functions](/loss-functions-eng.html)
- Feature importance computation
- Relatively robust to outliers

**Disadvantages:**
- Sequential (slow training)
- More hyperparameters to tune
- Risk of overfitting if not careful

## Stacking

Stacking trains a meta-learner to combine base learners' predictions optimally.

**Process:**
1. Partition training data
2. Train base learners on one part
3. Generate predictions on held-out part
4. Train meta-learner on these predictions
5. For test data: base learners predict → meta-learner combines

**Advantages:**
- Combines diverse learner strengths
- Flexible architecture
- Can improve even with similar base learners

**Disadvantages:**
- Computationally expensive
- Prone to overfitting without careful validation
- Complex to implement correctly

## Voting

Voting combines predictions from multiple learners using simple majority or weighted voting.

**Hard voting (classification):**
```
prediction = majority(predictions from all learners)
```

**Soft voting (classification):**
```
prediction = argmax(mean(probabilities from all learners))
```

**Advantages:**
- Simple to implement
- Works with different learner types
- Baseline ensemble approach

## Random Forests

[Random forests](/decision-trees-eng.html) are specialized bagging on [decision trees](/decision-trees-eng.html) with feature randomness:

**Key differences from simple bagging:**
- Feature subsampling: randomly select features at each split
- High tree depth: grow full trees (reduce bias)
- Decorrelation: feature randomness reduces tree correlations

**Why effective:**
- Bootstrap reduces variance
- Feature randomness increases diversity
- Many trees capture complex patterns
- Robust to outliers and noise

## Practical Tips

### Hyperparameter Tuning

**For bagging:**
- Number of base learners: more helps until diminishing returns
- Base learner complexity: increase to reduce bias

**For boosting:**
- [Learning rate](/learning-rate-scheduling-eng.html): lower values slower but more accurate
- Number of iterations: more improves until overfitting
- Base learner complexity: typically weak learners (shallow depth)

**For stacking:**
- Diversity in base learners critical
- [Cross-validation](/cross-validation-eng.html) folds in meta-learner training
- Meta-learner complexity: usually simple (logistic regression)

### Best Practices

**Diversity:**
- Use different algorithms (tree, [SVM](/svm-eng.html), neural net)
- Vary hyperparameters
- Different feature subsets
- Different [data preprocessing](/data-preprocessing-eng.html)

**Validation:**
- [Cross-validation](/cross-validation-eng.html) for stable estimates
- Monitor out-of-bag (OOB) error
- Separate test set for final evaluation

**Computational efficiency:**
- Parallelize independent learners (bagging)
- Cache predictions for stacking
- Use efficient implementations ([XGBoost](/gradient-boosting-eng.html), [LightGBM](/gradient-boosting-eng.html))

## Comparison

| Method | Speed | Interpretability | Parallelizable | Bias/Variance |
|--------|-------|-----------------|---|---|
| Bagging | Fast | Moderate | Yes | Reduces variance |
| AdaBoost | Moderate | Low | No | Reduces bias |
| [Gradient Boosting](/gradient-boosting-eng.html) | Slow | Low | No | Reduces both |
| Stacking | Slow | Low | No | Reduces both |
| Voting | Fast | Moderate | Yes | Reduces both |

## Ensemble in Practice

**When to use ensembles:**
- Accuracy is paramount
- Computational resources available
- Dataset is moderate-to-large
- Can tolerate increased model complexity

**Standard approaches:**
- Default: [Random Forests](/decision-trees-eng.html) (fast, effective, simple)
- For maximum accuracy: [Gradient Boosting](/gradient-boosting-eng.html) ([XGBoost](/gradient-boosting-eng.html), [LightGBM](/gradient-boosting-eng.html))
- For safety: Voting ensemble (use multiple proven methods)

**Diminishing returns:**
- Individual learner accuracy plateau first
- Ensemble improvements continue longer
- Typical sweet spot: 50-100 trees for bagging
- Typical sweet spot: 100-1000 iterations for boosting

## Summary

Ensemble methods represent a fundamental shift in how we approach prediction: instead of seeking a single perfect model, we leverage the wisdom of crowds. Through careful design of diverse base learners and intelligent combination strategies, ensembles achieve remarkable performance across classification, regression, and ranking tasks. They remain central to practical machine learning success.
