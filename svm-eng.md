---
layout: default
title: Support Vector Machines
lang: en
---

# Support Vector Machines

## Introduction

Support Vector Machines (SVMs) are powerful [supervised learning](/supervised-learning-eng.html) algorithms for classification and regression. They find the optimal hyperplane that maximizes the margin between classes, making them particularly effective for binary and multi-class problems. SVMs are versatile, handling both linearly and non-linearly separable data through kernel methods.

## Linear SVM

### The Margin Concept

The fundamental idea behind SVMs is the **maximum margin principle**: the best decision boundary is the one that maximizes the distance (margin) between the hyperplane and the nearest data points from each class.

**Mathematical formulation:**
```
Maximize: margin = 2 / ||w||
Subject to: y_i(w·x_i + b) ≥ 1 for all i
```

Where w is the weight vector, b is the bias, and y_i ∈ {-1, +1} are class labels. The constraint ensures correct classification with maximum margin.

### Support Vectors

Support vectors are the training points closest to the decision boundary. Only these points matter for defining the hyperplane; all other points could be removed without changing the solution. This property makes SVMs efficient and robust.

**Key insight:** The decision function depends only on support vectors:
```
f(x) = sign(Σ α_i × y_i × (x_i · x) + b)
```

Where α_i are non-zero only for support vectors.

## Soft Margin

Real-world data is rarely perfectly separable. The soft margin formulation allows some misclassification to handle noise and overlapping classes.

**Objective function:**
```
Minimize: (1/2)||w||² + C × Σ ξ_i
```

Where ξ_i are slack variables allowing violations, and C is a [regularization](/regularization-eng.html) parameter:
- Large C: Penalizes misclassification heavily (less tolerance for errors)
- Small C: Allows more violations (more tolerance)

## The Kernel Trick

The kernel trick enables SVMs to handle non-linear decision boundaries without explicitly computing high-dimensional feature transformations.

### Kernel Functions

**Linear kernel:**
```
K(x_i, x_j) = x_i · x_j
```
Simple, fast, best for linearly separable data.

**Polynomial kernel:**
```
K(x_i, x_j) = (x_i · x_j + c)^d
```
Creates polynomial decision boundaries. Parameter d controls polynomial degree.

**Radial Basis Function (RBF) kernel:**
```
K(x_i, x_j) = exp(-γ ||x_i - x_j||²)
```
Most versatile, handles complex non-linear patterns. Parameter γ controls influence range.

**Sigmoid kernel:**
```
K(x_i, x_j) = tanh(k × (x_i · x_j) + c)
```
Similar behavior to [neural networks](/neural-eng.html).

### How Kernels Work

Kernels implicitly map data into high-dimensional spaces where linear separation becomes possible, avoiding expensive explicit feature computation. This is the "kernel trick"—computing similarity without coordinates.

## SVM for Classification

### Binary Classification

SVMs naturally handle binary classification. The hyperplane separates the two classes while maximizing margin. Decision function gives distance from boundary (useful for confidence scores).

### Multi-class Classification

For problems with more than two classes, common strategies include:
- **One-vs-Rest (OvR)**: Train k classifiers for k classes
- **One-vs-One (OvO)**: Train k(k-1)/2 pairwise classifiers
- **Structured SVM**: Direct multi-class formulation

## SVM for Regression

Support Vector Regression (SVR) extends SVMs to continuous output. Instead of maximizing margin between classes, SVR fits a tube around predictions with specified width ε.

**Objective:**
```
Minimize: (1/2)||w||² + C × Σ max(0, |y_i - (w·x_i + b)| - ε)
```

The ε-insensitive loss ignores small errors within the tube, reducing sensitivity to outliers.

## Advantages

- **Effective in high dimensions**: Good for problems with many features
- **Memory efficient**: Decision depends only on support vectors
- **Versatile**: Handles classification, regression, and ranking
- **Strong theoretical foundation**: [Convex optimization](/convex-optimization-eng.html) with global optimum
- **Robust to outliers**: Soft margin and support vector mechanism
- **Non-linear capability**: Kernel trick handles complex relationships
- **Unique solution**: No local minima, always finds global optimum

## Disadvantages

- **Computational complexity**: O(n²) or O(n³) training time for n samples
- **Parameter tuning**: Requires careful C and kernel parameter selection
- **Scalability**: Challenging for very large datasets
- **Black box**: Difficult to interpret feature contributions
- **Class imbalance**: May require adjustment of class weights
- **Probability estimates**: Not natural to SVM; require post-processing
- **Kernel selection**: Choosing appropriate kernel needs domain knowledge

## Comparison with Other Methods

**vs. Logistic Regression:**
- SVM better for non-linear, high-dimensional problems
- Logistic regression more interpretable and faster

**vs. Neural Networks:**
- SVM better for small-to-medium datasets
- [Neural networks](/neural-eng.html) scale better, more flexible
- SVM less prone to local minima

**vs. Random Forests:**
- SVM better for high dimensions
- [Random Forests](/decision-trees-eng.html) more interpretable, handle mixed features
- Comparable accuracy on tabular data

## Practical Tips

**Preprocessing:**
- Standardize features (critical for SVM)
- Handle missing values beforehand

**Parameter tuning:**
- Grid search or randomized search for C and γ
- [Cross-validation](/cross-validation-eng.html) to prevent overfitting

**Class imbalance:**
- Adjust class weights: `class_weight = 'balanced'`
- Use appropriate sampling strategies

**Kernel selection:**
- Start with RBF (most versatile)
- Try polynomial for polynomial-looking boundaries
- Use linear for interpretability and speed

## Summary

Support Vector Machines remain one of the most elegant and theoretically grounded machine learning algorithms. Their ability to handle non-linearity through kernels, combined with strong generalization guarantees, makes them invaluable for classification and regression. While computationally intensive for massive datasets, SVMs excel on moderate-sized problems with complex decision boundaries.
