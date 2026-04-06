---
layout: default
title: Flow-Based Models
lang: en
---

# Flow-Based Models

## Introduction

Flow-based models represent a distinct class of generative models that learn transformations between simple distributions and complex data distributions. By using invertible transformations, these models provide tractable likelihood computation and allow exact density evaluation—a capability lacking in GANs and more expensive in VAEs.

## Normalizing Flows: Core Principle

A normalizing flow transforms a simple distribution (typically standard normal) into a complex distribution through a sequence of invertible, differentiable transformations:

```
z_0 ~ p(z_0)  (simple distribution)
z_1 = f_1(z_0)
z_2 = f_2(z_1)
...
x = z_K = f_K(f_{K-1}(...f_1(z_0)))
```

Each transformation f_i must be:
- **Invertible:** Can compute both forward and inverse transformations
- **Differentiable:** Allows gradient-based optimization
- **Efficient:** Jacobian determinant computable in reasonable time

## Change of Variables and Likelihood

The key mathematical tool is the change of variables formula. When transforming from z to x through invertible transformation f:

```
p(x) = p(z) * |det(∂f^{-1}/∂x)| = p(z) / |det(∂f/∂z)|
```

This enables computing exact log-likelihood:

```
log p(x) = log p(z) - log |det(∂f/∂z)|
```

The ability to compute exact likelihoods makes flow models valuable for tasks where density estimation matters, such as anomaly detection and scientific modeling.

## Invertible Transformations

### Linear Transformations

Simple invertible linear layers provide limited expressiveness but can be stacked effectively. The determinant is straightforward to compute from eigenvalues.

### Coupling Layers

Coupling layers split input dimensions into two groups. The first group passes through unchanged while transforming the second group based on the first:

```
x_1' = x_1
x_2' = t(x_1) ⊙ x_2 + s(x_1)
```

Where t and s are arbitrary neural networks. This maintains invertibility while allowing powerful transformations through arbitrarily complex networks. The Jacobian is triangular, making determinant computation efficient.

### Autoregressive Layers

Autoregressive flows compute each output dimension based only on previous inputs:

```
x_i' = t(x_1, ..., x_{i-1}) ⊙ x_i + s(x_1, ..., x_{i-1})
```

This guarantees invertibility and produces triangular Jacobians, but requires sequential computation during transformation.

## RealNVP: Masked Autoencoder Density Estimation

RealNVP combines masked autoencoder principles with normalizing flows. The architecture alternates between:

1. Coupling layers where different sets of dimensions are masked
2. Learnable permutations of dimensions to enable information flow

The masking ensures each layer's Jacobian is triangular while permutations prevent information from getting stuck in early dimensions. This enables efficient, parallel computation of both forward and inverse transformations.

## Glow: Improving RealNVP with Actnorm

Glow improves on RealNVP through:

**Actnorm layers:** Activation normalization using per-channel affine transformation with data-dependent initialization. Provides fast, stable training without explicit batch normalization.

**1x1 Convolutions:** Replaces fixed permutations with learnable 1x1 convolutions for improved flexibility.

**Multi-scale Architecture:** Processes different resolution levels, improving computational efficiency while maintaining expressiveness.

Glow achieves comparable image generation quality to GANs while providing exact likelihood—a unique advantage among high-quality generative models.

## Continuous Normalizing Flows

Neural ODE-based approaches model flows as continuous transformations:

```
z_T = z_0 + ∫_0^T v_θ(z_t, t) dt
```

Where v_θ is a neural network velocity function learned during training. Computing the determinant requires solving:

```
log |det(∂z_T/∂z_0)| = ∫_0^T tr(∂v_θ/∂z_t) dt
```

Continuous flows offer:
- Flexible architectures without architectural constraints
- Potential for better expressiveness through continuous modeling
- Connection to differential equations providing theoretical insights

However, they require expensive numerical integration during training and sampling.

## Comparison: Flow Models vs GAN vs VAE

**Flow Models:**
- Exact likelihood computation
- Stable training
- Slower generation than GANs
- Limited generation quality compared to diffusion models
- Architectural constraints for invertibility

**GANs:**
- Sharp image generation
- Fast sampling
- Unstable training and mode collapse
- No likelihood available
- More flexible architectures

**VAEs:**
- Interpretable latent space
- Stable training
- Blurry generation
- Tractable (though approximated) likelihood
- Moderate computational cost

**Diffusion Models:**
- State-of-the-art generation quality
- Stable training
- Slow sampling
- Allows likelihood computation with modifications
- Increasingly preferred for image generation

## Applications of Flow Models

**Density Estimation:** Computing likelihoods for scientific data analysis, anomaly detection

**Invertible Neural Networks:** Learning reversible transformations for specific applications

**Variational Inference:** Using flows to approximate complex posterior distributions

**Data Generation:** Generating synthetic samples, though typically lower quality than diffusion

**Feature Learning:** Unsupervised representation learning through density modeling

**Sampling:** Efficiently sampling from complicated distributions

## Practical Advantages and Limitations

**Advantages:**
- Exact, tractable likelihood computation
- Stable training without adversarial dynamics
- Invertible property enables both generation and inference
- Theoretically principled approach
- Deterministic encoding and decoding

**Limitations:**
- Architectural constraints reduce modeling flexibility
- Generated samples typically less sharp than GANs
- Slower generation than GANs
- Training computational cost can be significant
- Limited success on high-resolution image generation
- Jacobian computation requires careful implementation

## Current Research Directions

Recent flow research explores:
- Hybrid approaches combining flows with diffusion models
- Improved inverse transformations for faster sampling
- Scalable flow architectures for high-dimensional data
- Applications in scientific computing and physics
- Continuous flows with more efficient numerical methods

## Conclusion

Flow-based models provide a mathematically elegant approach to generative modeling with guaranteed likelihood computation. While recent diffusion models have achieved superior image generation, flows remain valuable for applications requiring exact densities, such as anomaly detection, scientific inference, and likelihood-based analysis. Understanding normalizing flows develops intuition about invertible transformations and probability, concepts increasingly relevant across deep learning.
