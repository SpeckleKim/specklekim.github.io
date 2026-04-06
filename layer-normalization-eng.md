---
layout: default
title: Layer Normalization
lang: en
---

# Layer Normalization: The Foundation of Transformer Architecture

Layer Normalization (LayerNorm) has become one of the most important components in modern deep learning, especially in [Transformer](/llm-eng.html) models. Unlike [batch normalization](/batch-normalization-eng.html) which normalizes across the batch dimension, layer normalization normalizes across the feature dimension for each instance independently.

## The Problem with Batch Normalization

[Batch normalization](/batch-normalization-eng.html) computes [statistics](/probability-eng.html) (mean and variance) across the batch dimension, making it problematic in several scenarios:

- **Variable batch sizes**: At inference, we typically process single examples, breaking the assumption of batch [statistics](/probability-eng.html)
- **RNNs and sequence models**: [Batch norm](/batch-normalization-eng.html) doesn't work well with variable sequence lengths
- **NLP applications**: Different sequence lengths make batch [statistics](/probability-eng.html) unreliable
- **GPU memory constraints**: Larger batches needed for stable [statistics](/probability-eng.html)

These limitations motivated the development of alternative normalization schemes.

## How Layer Normalization Works

LayerNorm computes mean and variance across the feature dimension (d) for each example independently:

```
μ_i = (1/d) * Σ(x_ij)  for sample i across all features j
σ_i² = (1/d) * Σ(x_ij - μ_i)²

y_ij = γ * (x_ij - μ_i) / √(σ_i² + ε) + β
```

Key characteristics:
- Operates independently on each instance
- Normalizes across the feature dimension, not the batch dimension
- Learnable affine transformation with parameters γ (scale) and β (shift)
- Same computation at training and inference time
- No dependency on batch size or other examples

## Why Transformers Use Layer Normalization

[Transformers](/llm-eng.html) adopted LayerNorm as the default normalization method for several compelling reasons:

1. **Batch-size independence**: Works identically regardless of batch size, crucial for variable-length sequences
2. **Stability in training**: Provides consistent gradient flow without batch [statistics](/probability-eng.html) variability
3. **Sequence model compatibility**: Works seamlessly with [attention mechanisms](/attention-mechanism-eng.html) and variable-length inputs
4. **Post-LN vs Pre-LN**: Enables different architectural variants for improved training dynamics
5. **Distributed training**: Simpler to implement in distributed settings without synchronization overhead

## Variants and Alternatives

**RMSNorm** (Root Mean Square Normalization):
- Normalizes using only variance, removing the centering step
- Simpler computation and slightly better efficiency
- Adopted in recent models like LLaMA
- Formula: y = x / √(mean(x²) + ε) * γ

**Group Normalization**:
- Divides channels into groups and normalizes within each group
- Addresses [batch norm](/batch-normalization-eng.html)'s batch size dependency
- Useful when batch sizes are constrained
- Balance between LayerNorm and InstanceNorm properties

**Instance Normalization**:
- Normalizes each channel independently for each sample
- Commonly used in style transfer and [image generation](/image-generation-eng.html)
- Removes style information while preserving content
- Can be too aggressive for discriminative tasks

## Layer Normalization in Practice

When implementing LayerNorm:

1. **Numerical stability**: Always add small epsilon (ε ≈ 1e-6) to variance to avoid division by zero
2. **Initialization**: Initialize γ = 1 and β = 0 (identity function)
3. **Placement**: Pre-LN (applied before sublayer) vs Post-LN (applied after) affects optimization
4. **Integration**: Typically applied in [Transformer](/llm-eng.html) blocks after attention and feedforward layers

## Pre-LN vs Post-LN

**Post-LN** (original [Transformer](/llm-eng.html)):
- Applied after [residual connections](/residual-connections-eng.html)
- Can cause gradient explosion in deep networks
- Requires more careful [learning rate](/learning-rate-scheduling-eng.html) tuning
- Better performance with proper warmup

**Pre-LN**:
- Applied before sublayers
- More stable gradients, easier to train
- Allows deeper networks without explosion
- Standard in recent architectures

## Comparison with Other Normalizations

| Method | Independence | Speed | Batch Sensitivity | Use Case |
|--------|-------------|-------|------------------|----------|
| [Batch Norm](/batch-normalization-eng.html) | Batch-dependent | Fast | High | CNNs, large batches |
| Layer Norm | Instance-dependent | Slower | None | [Transformers](/llm-eng.html), RNNs |
| Group Norm | Group-dependent | Medium | Low | Small batches, images |
| RMSNorm | Instance-dependent | Fastest | None | [Large language models](/llm-eng.html) |
| Instance Norm | Fully independent | Medium | None | Style transfer |

## Mathematical Intuition

LayerNorm stabilizes training by:
1. Reducing internal covariate shift (distribution changes of neuron inputs)
2. Smoothing the loss landscape for easier optimization
3. Providing [regularization](/regularization-eng.html) effect through normalization
4. Enabling higher learning rates without divergence

## Common Implementation Issues

- **Epsilon value**: Too small causes instability, too large removes normalization effect
- **Gradient flow**: Pre-LN generally has better gradient properties than Post-LN
- **Learned affine parameters**: γ and β are crucial; don't freeze them
- **Mixed precision training**: May need larger epsilon with float16 operations

## Conclusion

Layer Normalization has proven to be the normalization method of choice for modern architectures, particularly [Transformers](/llm-eng.html). Its batch-size independence and stability properties make it superior to [batch normalization](/batch-normalization-eng.html) for sequential and variable-length data. Understanding LayerNorm's mechanics—normalizing across features rather than batches—provides insight into why it works so well in these domains.

The evolution toward RMSNorm and other variants continues to improve efficiency and performance, but LayerNorm remains the foundational technique that enabled the [Transformer](/llm-eng.html) revolution in NLP.
