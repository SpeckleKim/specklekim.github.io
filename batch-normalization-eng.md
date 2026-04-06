---
layout: default
title: Batch Normalization
lang: en
---

# Batch Normalization: Stabilizing Deep Learning

Batch normalization (BatchNorm) normalizes layer inputs to have zero mean and unit variance, stabilizing training and enabling higher learning rates. It's one of the most impactful techniques in deep learning, addressing the internal covariate shift problem.

## Internal Covariate Shift

Internal covariate shift occurs when the distribution of inputs to a layer changes during training. As earlier layers update their weights, the distribution of data flowing to later layers shifts, forcing those layers to constantly adapt.

Example: If initial layer weights change significantly, the activation distribution entering the second layer transforms. The second layer must re-adapt its weights to this new distribution.

This shifting distribution slows learning and requires careful hyperparameter tuning (lower learning rates).

## Batch Normalization Algorithm

During training, batch normalization normalizes activations within each mini-batch:

1. **Compute batch statistics**:
   - `μ_batch = (1/m) * Σ x_i` (batch mean)
   - `σ²_batch = (1/m) * Σ (x_i - μ_batch)²` (batch variance)

2. **Normalize**:
   - `x̂_i = (x_i - μ_batch) / √(σ²_batch + ε)`

3. **Scale and shift** (learnable parameters):
   - `y_i = γ * x̂_i + β`

The parameters γ (scale) and β (shift) are learned during training, allowing the network to undo normalization if beneficial.

## Training vs. Inference

BatchNorm behaves differently during training and inference:

**During training**: Uses statistics from the current mini-batch.

**During inference**: Uses running statistics (exponential moving averages computed during training) to normalize:
- `x_normalized = (x - running_mean) / √(running_var + ε)`

This is critical: using batch statistics during inference would be wrong since test batches may have different distributions.

## Running Statistics

During training, exponential moving averages accumulate batch statistics:

`running_mean = momentum * running_mean + (1 - momentum) * batch_mean`
`running_var = momentum * running_var + (1 - momentum) * batch_var`

With momentum=0.99, recent batches heavily influence the running statistics while maintaining stability.

These accumulated statistics become the parameters used during inference.

## Benefits of Batch Normalization

**Faster convergence**: Reduces internal covariate shift, allowing higher learning rates without instability.

**Regularization effect**: Randomness from batch statistics acts as implicit regularization, often reducing overfitting.

**Reduces sensitivity to weight initialization**: Networks are less dependent on careful initialization since normalizing layers restabilize activations.

**Enables higher learning rates**: More stable gradient flow allows more aggressive learning rate schedules.

**Simplified hyperparameter tuning**: BatchNorm reduces the need for careful learning rate and initialization tuning.

## Batch Normalization in CNNs

In convolutional neural networks, BatchNorm is typically applied after convolutions and before activations:

`Conv → BatchNorm → ReLU → Pool`

BatchNorm operates on each channel independently, computing statistics across the spatial dimensions and batch dimension.

For a batch of shape (batch_size, height, width, channels), statistics are computed over axes (batch, height, width), leaving channel-wise statistics.

## Batch Normalization in RNNs

Applying BatchNorm to RNNs is more complex because sequential dependencies mean normalizing across time steps can harm temporal correlations.

Solutions:
- **Layer Normalization**: Normalizes across features rather than batch. Computes mean/variance per example, making it suitable for RNNs.
- **Weight Normalization**: Reparameterizes weights directly.
- **Recurrent BatchNorm**: Applies BatchNorm to recurrent connections carefully.

Layer norm has become more popular than BatchNorm in RNNs and Transformers.

## Alternatives to Batch Normalization

**Layer Normalization**: Normalizes per example across features:
- Batch-size independent (good for small batches)
- Works well in RNNs and Transformers
- Standard in modern architectures

**Group Normalization**: Normalizes across groups of channels:
- Useful when batch size is small
- Works with any batch size

**Instance Normalization**: Normalizes per instance per channel:
- Used in style transfer and image-to-image translation

**Positional Normalization**: Normalizes spatially:
- Emerging technique for specific applications

## Challenges with Batch Normalization

**Batch size dependency**: Small batch sizes lead to noisy statistics. Performance degrades with tiny batches (batch_size < 16).

**Training-inference mismatch**: The discrepancy between using batch statistics vs. running statistics can sometimes hurt performance.

**Computational overhead**: BatchNorm adds memory and computation (typically ~30% overhead).

**Not always beneficial**: In some cases (small datasets, small models), BatchNorm may not help or can hurt.

## Best Practices

1. **Always use for large networks**: Deep networks benefit significantly.

2. **Careful with small batches**: Use alternative normalization (GroupNorm, LayerNorm) if batch size < 32.

3. **Momentum value**: 0.9 or 0.99 are typical. Higher momentum keeps longer history.

4. **Remove with other regularization**: If using dropout heavily, BatchNorm's regularization effect may be redundant.

5. **Modern preference**: Many recent architectures use LayerNorm or GroupNorm instead of BatchNorm.

## Summary

Batch normalization is a powerful technique that normalizes intermediate activations, addressing internal covariate shift. It enables faster, more stable training and has become standard in modern deep learning. Understanding the distinction between training and inference modes, and knowing when alternatives like layer normalization are more appropriate, is essential for effective deep learning practice. While BatchNorm isn't universally applied anymore (modern transformers prefer LayerNorm), it remains crucial for understanding how networks learn and stabilize.
