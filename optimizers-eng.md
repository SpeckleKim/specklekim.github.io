---
layout: default
title: Optimizers (SGD to AdamW)
lang: en
---

# Optimizers: Gradient-Based Learning Algorithms

Optimizers update model parameters based on gradients to minimize the loss function. Choosing the right optimizer affects convergence speed, stability, and generalization. Understanding their properties helps match algorithms to problems.

## Stochastic Gradient Descent (SGD)

SGD is the foundational optimizer, updating parameters in the opposite direction of the gradient:

`W_new = W_old - learning_rate * dL/dW`

**Characteristics**: Simple, robust, works well with careful tuning.

**Advantages**: Interpretable updates. Generalization often better than adaptive methods. Universal baseline.

**Disadvantages**: Slow convergence. Requires careful learning rate tuning. High sensitivity to learning rate.

**Use case**: Classical baseline. Problems with sufficient data.

## SGD with Momentum

Momentum accumulates gradient information over time, accelerating convergence:

`v = β*v + (1-β)*dL/dW` (velocity accumulation)
`W_new = W_old - learning_rate * v`

**Characteristics**: Smooths noisy gradients. Accelerates along consistent directions.

**Advantages**: Faster convergence than vanilla SGD. Better with momentum=0.9.

**Disadvantages**: Additional hyperparameter β. Still requires learning rate tuning.

**Use case**: Most problems. Standard for classical deep learning.

## Nesterov Accelerated Gradient

Nesterov improves momentum by looking ahead:

`W_lookahead = W_old - learning_rate * v`
`v = β*v + (1-β)*dL/dW|_{W_lookahead}`

**Characteristics**: Momentum with foresight. Better convergence properties.

**Advantages**: Faster than standard momentum. Theoretical guarantees.

**Disadvantages**: Slightly complex implementation. Minimal practical improvement over momentum.

**Use case**: When momentum is insufficient. Some computer vision tasks.

## Adagrad

Adagrad adapts the learning rate per parameter based on gradient history:

`g_accumulated = g_accumulated + (dL/dW)²`
`W_new = W_old - (learning_rate / √g_accumulated) * dL/dW`

**Characteristics**: Learning rate decays over time. Parameter-specific learning rates.

**Advantages**: Works well with sparse gradients. Less tuning needed.

**Disadvantages**: Learning rate monotonically decreases. May become too small.

**Use case**: Sparse data. NLP with sparse word embeddings.

## RMSprop

RMSprop addresses Adagrad's decay by using exponential moving average of squared gradients:

`v = β*v + (1-β)*(dL/dW)²`
`W_new = W_old - (learning_rate / √v) * dL/dW`

**Characteristics**: Adaptive learning rate without monotonic decay.

**Advantages**: Works well in practice. Handles non-stationary problems.

**Disadvantages**: Hyperparameter β to tune. Less theoretically justified.

**Use case**: Recurrent neural networks. General-purpose adaptive optimization.

## Adam (Adaptive Moment Estimation)

Adam combines momentum and RMSprop, maintaining first and second moment estimates:

`m = β₁*m + (1-β₁)*dL/dW` (first moment)
`v = β₂*v + (1-β₂)*(dL/dW)²` (second moment)
`m_corrected = m / (1-β₁^t)` (bias correction)
`v_corrected = v / (1-β₂^t)` (bias correction)
`W_new = W_old - learning_rate * m_corrected / (√v_corrected + ε)`

**Characteristics**: Combines advantages of momentum and RMSprop. Bias-corrected.

**Advantages**: Fast convergence. Works across diverse problems. Minimal tuning needed.

**Disadvantages**: May generalize worse than SGD. Higher memory usage.

**Use case**: Default optimizer for many tasks. Good for deep learning.

## AdamW (Adam with Decoupled Weight Decay)

AdamW decouples weight decay from gradient-based updates:

`W_new = W_old - learning_rate * m_corrected / (√v_corrected + ε) - learning_rate * λ * W_old`

**Characteristics**: Proper L2 regularization. Fixes issues with Adam + weight decay.

**Advantages**: Better generalization than Adam. Standard in modern deep learning.

**Disadvantages**: Hyperparameter λ (weight decay). Slightly more complex.

**Use case**: Modern deep networks. State-of-the-art models. Recommended over Adam.

## LAMB (Layer-wise Adaptive Moments optimizer for Batch training)

LAMB extends AdamW with layer-wise learning rate adaptation:

`update = m_corrected / (√v_corrected + ε)`
`trust_ratio = ||W|| / ||update||` (layer-wise adaptation)
`W_new = W_old - learning_rate * trust_ratio * update`

**Characteristics**: Normalizes updates layer-wise. Better for large batch training.

**Advantages**: Enables large batch training. Scales to massive datasets.

**Disadvantages**: Complex. Layer-wise statistics overhead.

**Use case**: Large-batch distributed training. Natural language processing at scale.

## Learning Rate and Convergence

Learning rate is critical across all optimizers:
- **Too high**: Divergence, oscillations, instability
- **Too low**: Slow convergence, getting stuck in local minima
- **Sweet spot**: Fast convergence with stability

**Learning rate schedules** reduce learning rate over time:
- **Step decay**: Reduce by factor periodically
- **Exponential decay**: Exponential reduction
- **Cosine annealing**: Smooth cosine-based reduction
- **Warm-up**: Increase early then decay

Most modern frameworks support these automatically.

## Practical Comparison

| Optimizer | Speed | Stability | Generalization | Tuning |
|-----------|-------|-----------|-----------------|--------|
| SGD       | Slow  | Very high | Very good       | Hard   |
| Momentum  | Medium| High      | Very good       | Medium |
| Adam      | Fast  | Medium    | Good            | Easy   |
| AdamW     | Fast  | High      | Very good       | Easy   |
| LAMB      | Fast  | High      | Very good       | Hard   |

## Choosing an Optimizer

**General deep learning**: Start with AdamW. It balances speed and stability.

**If convergence is slow**: Try SGD with momentum for better generalization.

**Large batch training**: Use LAMB to maintain stability.

**Sparse gradients or NLP**: RMSprop or Adam works well.

**When in doubt**: AdamW is a safe, practical default that works across domains.

## Summary

Modern optimization has evolved from simple SGD to adaptive methods like Adam and AdamW. Each optimizer trades off between convergence speed, stability, and generalization. For most modern deep learning, AdamW provides an excellent balance. Understanding the characteristics of different optimizers allows informed choice based on problem characteristics and computational constraints.
