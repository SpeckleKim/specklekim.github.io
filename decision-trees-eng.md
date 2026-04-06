---
layout: default
title: Decision Trees & Random Forests
lang: en
---

# Decision Trees & Random Forests

## Introduction

Decision trees are fundamental supervised learning algorithms that recursively partition feature space to make predictions. They work by asking simple yes/no questions about features, creating a tree-like model of decisions. Random forests extend this by combining multiple trees to reduce overfitting and improve generalization.

## Decision Tree Construction

### The Growing Process

A decision tree grows from top to bottom (top-down induction). At each node, we select the feature and threshold that best splits the data into homogeneous subgroups. The algorithm recursively applies this process until reaching stopping criteria.

**Key Components:**
- **Root node**: Initial split on entire dataset
- **Internal nodes**: Feature and threshold combinations
- **Leaf nodes**: Final predictions (class labels or regression values)
- **Branches**: Represent decision outcomes

### Information Gain

Information gain measures how much a split reduces uncertainty. It's calculated as the difference between parent node entropy and weighted average of children entropy.

**Formula:**
```
Information Gain = H(Parent) - Σ(|Child|/|Parent|) × H(Child)
```

Where H is entropy: `H = -Σ p_i × log₂(p_i)`

Splits with higher information gain are preferred because they create more pure (homogeneous) child nodes.

## Gini Impurity

Gini impurity is an alternative to entropy. It measures the probability of incorrectly classifying a random sample if we assigned a label randomly according to class distribution.

**Formula:**
```
Gini = 1 - Σ p_i²
```

Where p_i is the proportion of class i. Gini impurity ranges from 0 (pure node) to 0.5 (maximum impurity in binary classification).

**Advantages over entropy:**
- Computationally faster (no logarithm required)
- Often produces similar results with better performance
- Slightly biased toward balanced partitions

## Pruning

Unpruned trees tend to overfit, capturing noise alongside patterns. Pruning removes branches that don't improve validation performance.

### Pruning Methods

**Pre-pruning (Early Stopping):**
- Stop growing when a criterion is met (minimum samples, maximum depth)
- Simple but may underfit

**Post-pruning:**
- Grow full tree, then recursively remove branches
- More computationally expensive but generally more effective
- Common approach: cost-complexity pruning using validation set

## Random Forests

Random forests combine multiple decision trees to create a robust ensemble classifier. They achieve remarkable accuracy by leveraging diversity and averaging.

### Bagging (Bootstrap Aggregating)

Random forests use bootstrap sampling: for each tree, randomly sample the training data with replacement to create different subsets. Each tree trains on its own bootstrap sample, introducing variance that improves final predictions.

**Process:**
1. Create B bootstrap samples
2. Grow a full decision tree on each sample
3. Average predictions (classification: majority vote, regression: mean)

### Feature Randomness

At each node, instead of searching all features, random forests randomly select m features from the total p features. This further increases diversity between trees and decorrelates predictions.

**Typical choices:**
- Classification: m = √p
- Regression: m = p/3

## Feature Importance

Random forests calculate feature importance by measuring how much each feature decreases impurity across all trees.

**Importance calculation:**
```
Importance(feature) = Σ(decrease in impurity) / total decreases
```

Features that frequently provide large information gains rank higher. This helps identify which variables drive predictions.

## Advantages

- **Handles non-linearity**: No linear assumptions required
- **Feature interactions**: Automatically captures interactions
- **Robustness**: Bagging reduces variance and overfitting
- **Feature importance**: Built-in feature selection
- **Speed**: Parallelizable across trees
- **Mixed features**: Handles both categorical and numerical data
- **Missing values**: Can handle missing data

## Disadvantages

- **Interpretability**: Difficult to explain individual tree predictions
- **Computational cost**: Training many trees is slower than single tree
- **Memory**: Multiple models require more storage
- **Bias**: Cannot improve bias if base learner is biased
- **Class imbalance**: May struggle with imbalanced datasets
- **Extrapolation**: Cannot predict beyond training data range (regression)

## Practical Considerations

**Hyperparameters:**
- Number of trees (B): More trees improve stability but diminish returns
- Maximum tree depth: Controls complexity
- Minimum samples per leaf: Prevents overfitting
- Feature randomness parameter (m)

**When to use:**
- Classification and regression on tabular data
- When interpretability is secondary to performance
- Large datasets where parallelization helps
- Mixed data types

**When to avoid:**
- When you need probabilistic confidence intervals
- When model interpretability is critical
- On very high-dimensional data (curse of dimensionality)

## Summary

Decision trees provide an intuitive, interpretable foundation for understanding data. Random forests dramatically improve their performance through ensemble learning, decorrelation, and bootstrap sampling. They remain among the most practical and effective algorithms for tabular data across industry applications.
