---
layout: default
title: Bias-Variance Tradeoff
lang: en
---

# Bias-Variance Tradeoff

The bias-variance tradeoff is one of the most fundamental concepts in machine learning. It explains why simpler models sometimes generalize better than complex ones, and provides a framework for understanding the sources of prediction error.

## Bias: Systematic Error

**Bias** is the difference between the expected prediction of a model and the correct value we're trying to predict. It measures how far off our model's predictions are on average.

Mathematically, for a target value y and model prediction ŷ:
```
Bias = E[ŷ - y]
```

Bias arises from incorrect assumptions about the underlying problem:
- A linear model trained on nonlinear data has high bias
- Using too few features when the true relationship is complex causes high bias
- Underfitting (model too simple) leads to high bias

**Low-bias models** are flexible enough to capture complex relationships. Examples:
- High-degree polynomials
- Deep neural networks
- Decision trees with many splits

## Variance: Sensitivity to Data

**Variance** measures how much a model's predictions change when trained on different datasets. High variance means the model is very sensitive to the specific training data it sees.

Mathematically:
```
Variance = E[(ŷ - E[ŷ])²]
```

Variance increases when:
- Model has many parameters relative to training data
- Model memorizes specific patterns in the training set
- Overfitting (model too complex for the data)

**High-variance models** fit the training data very closely but fail on new data:
- Very deep neural networks on small datasets
- High-degree polynomial fits
- Decision trees that aren't pruned
- Models with many hyperparameters that are heavily tuned

## The Decomposition: Total Error

The expected squared error (loss) of any model can be decomposed as:

```
E[(y - ŷ)²] = Bias² + Variance + Irreducible Error
```

This is the bias-variance decomposition. Each term represents a different source of prediction error:

1. **Bias²**: Error from oversimplified assumptions
2. **Variance**: Error from sensitivity to training data fluctuations
3. **Irreducible Error** (Bayes Error): Random noise in the target, cannot be reduced

The key insight: **Bias and variance cannot both be minimized simultaneously**. There's always a tradeoff.

## Underfitting vs Overfitting

These concepts relate directly to bias and variance:

**Underfitting (High Bias, Low Variance):**
- Model is too simple for the problem
- Predictions are consistently wrong in a systematic way
- Training and validation error both high
- Low variance because the simple model doesn't adapt to noise

**Optimal Model (Balanced Bias & Variance):**
- Model complexity matched to data complexity
- Training and validation errors both reasonably low
- Minimizes total error

**Overfitting (Low Bias, High Variance):**
- Model is too complex for the data
- Fits training data perfectly but fails on new data
- Large gap between training and validation error
- High variance because model adapts to noise in training set

## The Model Complexity Curve

The relationship between model complexity and total error is visualized as a U-shaped curve:

```
        Error
          ^
          |     Overfitting
          |    /
          |   /    Validation error
      Min |  /
          | /
          |/__________
          |  \    Training error
          |   \
          |    \
          +-----------> Model Complexity
      Low                         High
```

- **Left side**: Underfitting region. As complexity increases, training and test error both decrease because bias is high
- **Bottom**: Sweet spot. Error is minimized
- **Right side**: Overfitting region. Test error increases while training error decreases because variance dominates

The optimal complexity is where validation error is minimized.

## Practical Implications

### How to Diagnose

**High Bias (Underfitting):**
- Check: training error and validation error are both high
- Both errors are similar (no gap)
- Model is too simple

**High Variance (Overfitting):**
- Check: training error is low, validation error is high
- Large gap between training and validation error
- Model is too complex

### How to Fix High Bias
- Increase model complexity
- Add more features
- Train longer (for neural networks)
- Use a more expressive model class

### How to Fix High Variance
- Reduce model complexity
- Collect more training data
- Apply regularization (L1, L2, dropout)
- Use ensemble methods
- Feature selection to reduce dimensionality

## The Data Size Effect

Model complexity should scale with dataset size. With limited data, even complex models have high variance because they overfit. With abundant data, complex models can be justified.

This is why transfer learning works: pretraining on large datasets followed by fine-tuning on small datasets reduces variance by starting from low-variance learned representations.

## Example: Polynomial Fitting

Fitting polynomials of increasing degree to data illustrates the bias-variance tradeoff:

- **Degree 1** (linear): High bias, low variance. Underfits even linear data if there's nonlinearity
- **Degree 3**: Balanced. Captures true patterns without memorizing noise
- **Degree 10**: Low bias, high variance. Passes through every point, overfits to noise

The degree-3 polynomial often generalizes best despite not fitting the training data as well as degree 10.

## Cross-Validation and the Tradeoff

Cross-validation helps navigate the bias-variance tradeoff by:
- **Estimating variance**: If CV fold scores vary widely, your model has high variance
- **Detecting overfitting**: If training accuracy >> CV accuracy, overfitting (high variance) is occurring
- **Comparing models**: CV helps identify the complexity level that minimizes total error

## Key Takeaways

1. Every error source comes from bias, variance, or irreducible noise
2. Adding complexity reduces bias but increases variance
3. The optimal model is rarely the one that fits training data best
4. Validation/test performance is more important than training performance
5. Match model complexity to data quantity and problem complexity
6. Regularization is the primary tool for reducing variance without reducing model capacity

The bias-variance tradeoff explains why "more complex = better" is false in machine learning. The best models balance simplicity and expressiveness.
