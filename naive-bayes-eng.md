---
layout: default
title: Naive Bayes Classifier
lang: en
---

# Naive Bayes Classifier

## Introduction

The Naive Bayes classifier is a probabilistic algorithm based on Bayes' theorem with a simplifying assumption: features are conditionally independent given the class label. Despite its oversimplified assumption, Naive Bayes is remarkably effective for classification tasks, particularly in [text classification](/text-classification-eng.html) and spam filtering. It's fast, interpretable, and requires modest computational resources.

## Bayes Theorem

Naive Bayes applies Bayes' theorem to compute posterior probabilities:

**Formula:**
```
P(Class | Features) = P(Features | Class) × P(Class) / P(Features)
```

Where:
- **P(Class | Features)**: Posterior [probability](/probability-eng.html) (what we want to compute)
- **P(Features | Class)**: Likelihood ([probability](/probability-eng.html) of features given class)
- **P(Class)**: Prior [probability](/probability-eng.html) (class frequency in training data)
- **P(Features)**: Evidence (constant for comparison)

**For classification:**
```
Predicted_Class = argmax_c P(c) × ∏ P(f_i | c)
```

We choose the class with highest posterior [probability](/probability-eng.html).

## The Naive Independence Assumption

The core simplifying assumption: features are conditionally independent given the class:

```
P(x₁, x₂, ..., xₙ | c) = ∏ P(x_i | c)
```

This dramatically simplifies computation but is rarely true in reality. Despite this oversimplification, Naive Bayes often performs well due to **probability cancellation** and **robustness to violation** of independence assumptions.

## Variants and Probability Models

Different [probability](/probability-eng.html) models suit different feature types and data distributions.

### Gaussian Naive Bayes

For continuous numerical features, assumes Gaussian (normal) distribution:

```
P(x_i | c) = (1/√(2πσ_c²)) × exp(-(x_i - μ_c)² / (2σ_c²))
```

**Parameters:**
- μ_c: Mean of feature i in class c
- σ_c²: Variance of feature i in class c

Estimated from training data.

**Use cases:**
- Real-valued features from continuous distributions
- Physical measurements and sensor data

### Multinomial Naive Bayes

For discrete count data, particularly useful for [text classification](/text-classification-eng.html):

```
P(x_i | c) = (count_i + 1) / (total_count + vocabulary_size)
```

Where count_i is the occurrence count of feature i in class c.

**Key properties:**
- Models word frequencies in documents
- Natural fit for document classification
- Widely used in NLP applications

### Bernoulli Naive Bayes

For binary features (presence/absence), common in document classification:

```
P(x_i | c) = P_i^(x_i) × (1 - P_i)^(1 - x_i)
```

Where P_i is the [probability](/probability-eng.html) of feature i in class c.

**Use cases:**
- Binary feature vectors
- Document classification with binary word indicators

## Laplace Smoothing

Smoothing prevents zero probabilities when a feature value never appears in a class during training.

**Problem:**
If P(x_i | c) = 0 for any feature, the entire product becomes 0.

**Laplace smoothing solution:**
```
P(x_i | c) = (count_i + α) / (total_count + α × K)
```

Where:
- α: Smoothing parameter (typically α = 1)
- K: Number of possible values for feature i

This ensures all probabilities remain non-zero.

## Text Classification

Naive Bayes excels at [text classification](/text-classification-eng.html) due to high-dimensional sparse features and probabilistic modeling of word distributions.

**Process:**
1. Tokenize documents into words
2. Build word frequency counts for each class
3. Apply Laplace smoothing
4. Classify new documents by comparing class probabilities

**Why Naive Bayes works for text:**
- Words' high dimensionality vs. modest sample size
- Independence assumption captures word significance
- Fast computation suitable for large text datasets
- Interpretable: can inspect learned feature probabilities

## Spam Filtering

Naive Bayes became famous for effective email spam filtering:

**Training:**
- Learn word probabilities in spam and legitimate emails
- Store P(word | spam) and P(word | legitimate)

**Prediction:**
- Compute P(spam | words_in_email) vs. P(legitimate | words_in_email)
- Classify based on which is higher

**Advantages for spam:**
- Adapts as spam evolves (retrainable)
- Fast, enabling real-time filtering
- Graceful degradation with unknown words (smoothing)

## Advantages

- **Speed**: Training and prediction are computationally fast
- **Simplicity**: Easy to understand and implement
- **Probabilistic**: Provides [probability](/probability-eng.html) estimates for predictions
- **Interpretable**: Can examine learned feature-class relationships
- **Small datasets**: Works well with limited training data
- **High dimensions**: Handles many features effectively
- **Robust**: Performs well despite independence assumption violations
- **Text**: Excellent for [text classification](/text-classification-eng.html) and NLP tasks

## Disadvantages

- **Oversimplified assumption**: Feature independence rarely holds in reality
- **Feature correlation**: Doesn't capture feature relationships
- **Zero frequency problem**: Requires smoothing for unseen data
- **Imbalanced data**: May struggle with highly imbalanced classes
- **Continuous features**: Gaussian assumption may not fit all distributions
- **Posterior probability**: Not well-calibrated, especially with many features
- **Rare events**: Extreme class imbalance causes problems
- **Feature selection**: Cannot leverage feature interactions

## Practical Considerations

**Preprocessing:**
- Feature scaling not required
- Handle missing values beforehand
- Text: [Tokenization](/tokenization-eng.html), lowercasing, stop word removal

**Parameter tuning:**
- Smoothing parameter α (typically 1.0)
- Feature selection to reduce noise
- Class weights for imbalanced data

**Model selection:**
- Choose variant based on feature type (Gaussian/Multinomial/Bernoulli)
- [Cross-validation](/cross-validation-eng.html) for performance assessment
- Compare [probability](/probability-eng.html) calibration with other methods

**When to use:**
- [Text classification](/text-classification-eng.html) (primary use case)
- Spam and email filtering
- [Sentiment analysis](/sentiment-analysis-eng.html)
- Document categorization
- Medical diagnosis with [probability](/probability-eng.html) output

**When to avoid:**
- When feature dependencies are critical
- With highly imbalanced data without adjustment
- When well-calibrated probabilities are essential
- If interpretability is not valued

## Comparison with Other Classifiers

**vs. Logistic Regression:**
- Naive Bayes: Probabilistic generative model
- Logistic Regression: Discriminative, better for dependent features
- Naive Bayes often faster to train

**vs. SVM:**
- [SVM](/svm-eng.html): Better for complex decision boundaries
- Naive Bayes: Better for high-dimensional text
- Naive Bayes: More interpretable

**vs. Random Forest:**
- [Random Forest](/decision-trees-eng.html): Captures feature interactions better
- Naive Bayes: Faster, more interpretable
- Naive Bayes: Better with limited data

## Summary

Naive Bayes remains a cornerstone algorithm for classification, particularly in text processing and spam filtering. Its combination of simplicity, speed, interpretability, and surprising effectiveness makes it invaluable as both a practical tool and educational foundation. Understanding its assumptions and limitations helps practitioners use it appropriately alongside modern deep learning approaches.
