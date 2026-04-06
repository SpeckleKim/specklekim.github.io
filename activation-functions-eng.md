---
layout: default
title: Activation Functions
lang: en
---

# Activation Functions: Introducing Nonlinearity

Activation functions are nonlinear transformations applied to neuron outputs. Without them, stacking layers would produce only linear transformations, limiting neural networks to learning linear relationships. Activation functions enable networks to learn complex, nonlinear patterns.

## Why Nonlinearity Matters

A neural network without activation functions is just matrix multiplication—multiple linear transformations compose into a single linear transformation. This makes deep networks equivalent to shallow ones, unable to express the complexity needed for real-world problems.

Activation functions introduce nonlinearity, allowing networks to bend, twist, and reshape decision boundaries. This enables learning arbitrary functions.

## Sigmoid Function

The sigmoid function maps inputs to a range of (0, 1):

`σ(x) = 1 / (1 + e^(-x))`

**Characteristics**: Smooth gradient, output interpretable as probability.

**Drawbacks**: Vanishing gradient problem in deep networks. Outputs not centered at zero, which slows learning. Heavy computation due to exponential.

**Use case**: Binary classification output layers, where probability interpretation is valuable.

## Tanh Function

Tanh (hyperbolic tangent) maps inputs to (-1, 1):

`tanh(x) = (e^x - e^(-x)) / (e^x + e^(-x))`

**Characteristics**: Zero-centered output (faster convergence), stronger gradients than sigmoid.

**Drawbacks**: Still suffers from vanishing gradients in very deep networks.

**Use case**: Hidden layers, particularly in RNNs and older architectures.

## ReLU (Rectified Linear Unit)

ReLU is the most popular activation function:

`ReLU(x) = max(0, x)`

**Characteristics**: Simple, fast computation. No vanishing gradient for positive values. Trains deep networks effectively.

**Drawbacks**: "Dying ReLU" problem—neurons can get stuck outputting zero. Not differentiable at x=0 (practically ignored).

**Use case**: Default choice for hidden layers in modern deep networks.

## Leaky ReLU

Leaky ReLU addresses the dying ReLU problem:

`Leaky ReLU(x) = max(αx, x)` where α is small (typically 0.01)

**Characteristics**: Allows small negative gradients, preventing neurons from dying. Slightly more computation than ReLU.

**Drawbacks**: Hyperparameter α to tune. Minimal empirical advantage in many cases.

**Use case**: When dying ReLU is observed. Stable alternative to standard ReLU.

## ELU (Exponential Linear Unit)

ELU smoothly transitions to negative values:

`ELU(x) = x if x > 0, else α(e^x - 1)`

**Characteristics**: Negative saturation property (mean closer to zero), smoother than ReLU.

**Drawbacks**: Exponential computation, slower than ReLU. Marginal improvements.

**Use case**: When mean-zero activations are desired. Convolutional networks.

## GELU (Gaussian Error Linear Unit)

GELU weights inputs by the cumulative Gaussian distribution:

`GELU(x) = x * Φ(x)` where Φ is the cumulative normal distribution

**Characteristics**: Smooth approximation to ReLU. Popular in modern transformers and large models.

**Drawbacks**: More computational cost. Complex to implement without libraries.

**Use case**: Transformer models (BERT, GPT), state-of-the-art architectures.

## Swish Function

Swish (also called SiLU when scaled) uses sigmoid gating:

`Swish(x) = x * σ(x)`

**Characteristics**: Self-gated mechanism. Empirically outperforms ReLU in many cases. Smooth, non-monotonic.

**Drawbacks**: More computation than ReLU. Not as widely adopted.

**Use case**: Modern architectures seeking improved performance. EfficientNets and similar models.

## Mish Function

Mish is a smooth, non-monotonic activation:

`Mish(x) = x * tanh(softplus(x))` where softplus(x) = ln(1 + e^x)

**Characteristics**: Self-regularizing. Works well in modern architectures. Smooth throughout.

**Drawbacks**: Computational cost. Less established than GELU.

**Use case**: Cutting-edge models. Experiments with novel architectures.

## Softmax Function

Softmax converts a vector to a probability distribution:

`softmax(x_i) = e^(x_i) / Σ(e^(x_j))`

**Characteristics**: Outputs sum to 1, interpretable as probabilities. Differentiable.

**Drawbacks**: Not suitable for hidden layers (only output). Numerically unstable (requires log-sum-exp trick).

**Use case**: Multi-class classification output layers. Only applied to final layer.

## Comparison and Practical Guidance

**For hidden layers**:
- Start with ReLU: fast, effective, low computation
- Try Leaky ReLU if ReLU causes issues
- Use GELU in transformers
- Consider Swish for empirical improvements

**For output layers**:
- Sigmoid for binary classification
- Softmax for multi-class classification
- Linear for regression

**Modern trends**: GELU dominates in large language models and transformers. ReLU remains standard in convolutional networks. Swish and Mish gain traction in cutting-edge work.

## Summary

Choosing the right activation function impacts learning speed, convergence, and final performance. Modern networks predominantly use ReLU variants for hidden layers and problem-specific functions for outputs. GELU has become the standard in transformer-based models, reflecting the shift in deep learning paradigms.
