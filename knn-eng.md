---
layout: default
title: K-Nearest Neighbors
lang: en
---

# K-Nearest Neighbors

## Introduction

K-Nearest Neighbors (KNN) is one of the simplest yet effective machine learning algorithms for classification and regression. It relies on a fundamental principle: similar instances have similar labels. KNN is non-parametric and lazy, making no assumptions about underlying data distribution and deferring computation until prediction time.

## Algorithm

### Basic Mechanism

KNN operates by storing the entire training dataset and making predictions based on the K nearest neighbors:

1. **Store training data** with corresponding labels
2. **For each test instance:**
   - Calculate distance to all training examples
   - Identify the K nearest neighbors
   - Classify as majority class (classification) or average value (regression)

### Prediction Rules

**Classification:**
```
predicted_class = most_common_class(K_nearest_neighbors)
```

Voting among K neighbors determines the prediction.

**Regression:**
```
predicted_value = mean(K_nearest_neighbors_values)
```

Average of K neighbors' values gives the prediction.

## Distance Metrics

Choosing the right distance metric is crucial for KNN performance.

### Euclidean Distance

The most common metric, measuring straight-line distance:
```
d(x, y) = √(Σ(x_i - y_i)²)
```

Works well for continuous numerical features. Sensitive to scale differences.

### Manhattan Distance (L1)

Sum of absolute differences:
```
d(x, y) = Σ|x_i - y_i|
```

More robust to outliers than Euclidean. Better for high-dimensional data.

### Minkowski Distance

Generalization encompassing both metrics:
```
d(x, y) = (Σ|x_i - y_i|^p)^(1/p)
```

When p=2, becomes Euclidean; when p=1, becomes Manhattan.

### Hamming Distance

For categorical/binary data:
```
d(x, y) = count of differing positions
```

Measures the number of attributes that differ.

### Cosine Distance

For text and high-dimensional sparse data:
```
d(x, y) = 1 - (x · y) / (||x|| × ||y||)
```

Measures angle between vectors rather than magnitude.

## Choosing K

The hyperparameter K significantly affects performance:

**Small K (e.g., K=1):**
- Captures fine details, follows noise closely
- Risk of overfitting
- High variance, low bias

**Large K:**
- Smoother decision boundaries
- May oversmooth important patterns
- Low variance, high bias
- More robust to noise

**Typical practice:**
- [Cross-validation](/cross-validation-eng.html) to find optimal K
- Often K = √(n) or K = 3-5 for small datasets
- Odd K for binary classification (avoids ties)

## Weighted KNN

Standard KNN treats all neighbors equally. Weighted KNN assigns higher importance to closer neighbors:

**Uniform weights (standard):**
```
weight_i = 1/K for all neighbors
```

**Distance-based weights:**
```
weight_i = 1 / distance_i
```

**Inverse squared distance:**
```
weight_i = 1 / (distance_i²)
```

Weighted variants reduce outlier influence and improve predictions.

## KD-Tree and Spatial Indexing

For large datasets, searching all training points is inefficient. KD-trees organize data hierarchically in k-dimensional space.

**Advantages:**
- Reduces search time from O(n) to O(log n) on average
- Divides space recursively using axis-aligned splits
- Enables efficient range queries

**Ball trees** and **LSH (Locality-Sensitive Hashing)** are alternatives for very high dimensions.

## Curse of Dimensionality

KNN suffers severely in high dimensions:

**Problem:**
- Distance becomes meaningless when dimension increases
- All points become roughly equidistant
- High-dimensional volume is concentrated in corners

**Example:**
- In 2D, neighbors within distance 1 form a circle
- In 100D, the same distance sphere misses most of the space

**Solutions:**
- [Dimensionality reduction](/dimensionality-reduction-eng.html) (PCA, feature selection)
- Distance metric learning
- Use tree-based methods instead
- Careful [feature engineering](/feature-engineering-eng.html)

## Applications

**Classification:**
- Recommendation systems (item similarity)
- Medical diagnosis
- Image classification
- Handwriting recognition

**Regression:**
- Price prediction from similar examples
- Stock market analysis
- Function approximation

**Hybrid:**
- [Anomaly detection](/anomaly-detection-eng.html) (neighbors are far apart)
- Imbalanced classification (weighted votes)

## Advantages

- **Simple and intuitive**: Easy to understand and implement
- **No training**: Lazy learning reduces training time
- **Versatile**: Works for classification, regression, and ranking
- **Non-parametric**: Makes no assumptions about data distribution
- **Effective baseline**: Often provides surprisingly good results
- **Adaptive**: Naturally adapts to local data structure

## Disadvantages

- **Computational cost**: O(n) prediction time for n training samples
- **Storage**: Requires storing entire dataset
- **Feature scaling**: Sensitive to feature magnitude differences
- **Curse of dimensionality**: Performance degrades in high dimensions
- **Irrelevant features**: All features treated equally
- **Imbalanced data**: Majority class may dominate voting
- **Interpretation**: No model to explain relationships

## Practical Considerations

**Preprocessing:**
- Normalize features to same scale
- Remove irrelevant features
- Handle missing values

**Parameter tuning:**
- [Cross-validation](/cross-validation-eng.html) for optimal K
- Try different distance metrics
- Consider weighted variants for imbalanced data

**Optimization:**
- Use KD-trees for moderate dimensions
- Approximate methods (LSH) for high dimensions
- [Dimensionality reduction](/dimensionality-reduction-eng.html) before applying KNN

**When to use:**
- Small-to-medium datasets
- Non-linear decision boundaries
- As baseline for comparison
- When interpretability via examples is valuable

## Summary

K-Nearest Neighbors is a foundational algorithm that exemplifies lazy learning. Its simplicity, effectiveness, and versatility make it valuable for practitioners. However, computational cost and the curse of dimensionality limit its application to smaller datasets and lower-dimensional problems. Understanding KNN's strengths and weaknesses is essential for choosing appropriate algorithms and preprocessing techniques.
