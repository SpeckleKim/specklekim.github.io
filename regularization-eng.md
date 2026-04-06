---
layout: default
title: Regularization (L1, L2, Dropout)
lang: en
---

# Regularization (L1, L2, Dropout)

Regularization is the primary technique for reducing variance and preventing overfitting. It works by adding constraints or penalties to the learning objective, forcing the model to find simpler solutions that generalize better.

## Why Regularization is Essential

Without regularization, optimization algorithms seek to minimize training error at all costs. On small datasets or with high-capacity models, this leads to memorization of noise rather than learning true patterns. Regularization introduces a bias toward simpler models, trading some training accuracy for much better generalization.

The regularized objective becomes:
```
Loss_total = Loss_data + λ * Loss_regularization
```

The hyperparameter λ (lambda) controls the regularization strength. Higher λ enforces stronger constraints toward simplicity.

## L2 Regularization (Ridge Regression)

L2 regularization adds a penalty proportional to the square of model weights:
```
Loss_L2 = Σ(actual - predicted)² + λ Σ(w²)
```

**How it works:**
- During optimization, the algorithm avoids large weight values
- Encourages small, diffuse weights distributed across features
- No weights are exactly zero (they're just shrunk)

**When to use:**
- When you believe all features are relevant
- When you want to balance the influence of many correlated features
- General purpose regularization that's almost always safe

**Effect on model:**
- All weights reduced proportionally
- Smoother decision boundaries in neural networks
- Computationally efficient (closed-form solutions exist for linear models)

In neural networks, L2 is also called **weight decay** because it decays weights toward zero during training.

## L1 Regularization (Lasso)

L1 regularization adds a penalty proportional to the absolute value of weights:
```
Loss_L1 = Σ(actual - predicted)² + λ Σ(|w|)
```

**How it works:**
- Creates a "diamonds" constraint region around the origin
- Forces solutions to lie on the axes/edges of constraint region
- This geometry naturally drives many weights to exactly zero

**When to use:**
- When you want feature selection (sparse models)
- When you suspect only a subset of features matter
- When interpretability is crucial

**Effect on model:**
- Some weights become exactly zero (automatic feature selection)
- Non-zero weights can be large (less shrinkage than L2)
- More aggressive in eliminating irrelevant features
- Computationally more expensive than L2

L1 regularization naturally produces **sparse models** where many weights are zero. This reduces the model to its essential features.

## L1 vs L2: The Geometric Intuition

Consider weight space with two parameters (w1, w2):

**L2 constraint:** w1² + w2² ≤ t (circle)
- Contours touch the circle at any point
- Optimal solution rarely at axis (weights stay non-zero)

**L1 constraint:** |w1| + |w2| ≤ t (diamond)
- Corners of diamond on axes
- Contours likely touch at corner (one weight = zero)

This geometric difference explains why L1 produces sparsity and L2 doesn't.

## Elastic Net: Combining L1 and L2

Elastic Net combines both penalties:
```
Loss = Σ(actual - predicted)² + λ₁ Σ(|w|) + λ₂ Σ(w²)
```

Or with a mixing parameter α (0 ≤ α ≤ 1):
```
Loss = Loss_data + λ[(1-α) Σ(w²)/2 + α Σ(|w|)]
```

**Advantages:**
- α=0 recovers pure L2; α=1 recovers pure L1
- Combines feature selection (L1) with weight shrinkage (L2)
- More stable than L1 when features are highly correlated
- Encourages similar coefficients for correlated features (grouping effect)

**When to use:**
- When you want some sparsity but fear L1's instability
- With high-dimensional data and correlated features
- Default choice when unsure between L1 and L2

## Dropout: Regularization for Neural Networks

Dropout is a stochastic regularization technique specific to neural networks:

**During training:**
- Randomly "drop" (set to zero) a fraction p of neurons in each layer
- Update weights only through surviving neurons
- Different random subset dropped each iteration

**During inference:**
- Use all neurons, but scale activations by (1-p)
- This ensures expected value matches training behavior

**Why it works:**
- Prevents co-adaptation of neurons
- Forces the network to learn redundant representations
- Equivalent to averaging many thinned networks
- Creates implicit ensemble of models

**Common dropout rates:**
- Input layer: 0.1-0.2
- Hidden layers: 0.3-0.5
- Output layer: Usually no dropout

**When to use:**
- Standard practice in all modern deep learning
- Especially important with limited data
- Most effective when network is very deep

Dropout is remarkably effective and simple—add it to nearly all neural networks.

## Early Stopping: Regularization Through Timing

Early stopping monitors validation error during training and stops when it starts increasing:

**Algorithm:**
1. Split data into train/validation
2. Monitor validation loss every few epochs
3. Stop when validation loss doesn't improve for N epochs
4. Use the best model (lowest validation loss), not final model

**Why it works:**
- Prevents long training that leads to overfitting
- Finds the sweet spot where training has learned patterns but hasn't memorized noise

**Advantages:**
- Simple to implement
- Works with any model
- Gives interpretable stopping criterion
- Computationally efficient (no extra cost)

**Disadvantages:**
- Requires separate validation set (reduces training data)
- Introduces randomness (different runs may stop at different epochs)

**Best practice:** Combine early stopping with other regularization techniques.

## Data Augmentation as Regularization

Data augmentation artificially increases dataset size by applying transformations:

**Computer Vision:**
- Rotations, flips, crops, brightness changes
- Mixup (blend two images)
- CutMix (paste patches between images)

**NLP:**
- Paraphrasing, backtranslation
- Word replacement with synonyms
- Random insertion/deletion

**Why it's regularization:**
- Each augmented version is slightly different
- Network must learn invariances rather than memorize exact pixels/words
- Effectively trains on more data, reducing variance

**Advantages:**
- Increases effective dataset size
- Often improves generalization more than L1/L2
- Can incorporate domain knowledge (what transformations are valid)

## Batch Normalization as Regularization

Batch normalization normalizes layer inputs to have zero mean and unit variance:

```
x_normalized = (x - batch_mean) / sqrt(batch_variance + ε)
```

**Regularization effect:**
- Noise from mini-batch statistics adds stochastic perturbation
- Reduces internal covariate shift
- Has implicit regularization effect (smoother loss landscape)
- Larger batch sizes reduce regularization effect

**Benefits:**
- Speeds up training (allows higher learning rates)
- Improves gradient flow in deep networks
- Acts as regularization (reduces need for dropout)

**Note:** Batch norm's regularization effect diminishes with large batch sizes, where batch statistics become stable.

## Practical Regularization Strategy

1. **Start with L2:** Safe default that rarely hurts
2. **Add dropout to neural networks:** Standard practice
3. **Use early stopping:** Always monitor validation performance
4. **Consider data augmentation:** Especially if data is limited
5. **Switch to L1 or Elastic Net for interpretability:** If you need feature selection
6. **Tune λ via cross-validation:** Regularization strength is a hyperparameter

## Typical λ Values

- **L2 in linear models:** 0.0001 to 0.1
- **L2 in neural networks:** 1e-6 to 1e-3
- **Dropout rate p:** 0.3 to 0.5 for hidden layers
- **Early stopping patience:** 10-20 epochs without improvement

## Common Mistakes

- **No regularization:** Model overfits without it
- **Too much regularization:** Model underfits (high bias)
- **Tuning λ on test set:** Makes test estimates invalid
- **Forgetting normalization:** Regularization assumes features are scaled similarly
- **Dropout at test time:** Must scale activations by (1-p) or use inverse dropout

The goal of regularization is not perfect training accuracy, but good generalization. A slightly undertrained model that generalizes well beats a perfectly trained model that doesn't.
