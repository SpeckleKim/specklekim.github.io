---
layout: default
title: Weight Initialization
lang: en
---

# Weight Initialization: Setting the Foundation for Deep Learning

Weight initialization is a critical yet often overlooked aspect of training deep [neural networks](/neural-eng.html). Poor initialization can lead to slow convergence, stuck gradients, or complete training failure, while proper initialization accelerates learning and improves final model performance.

## Why Initialization Matters

Training a deep network from random initialization resembles navigating a complex landscape with no prior information. Initialization determines:

1. **Speed of convergence**: Good initialization dramatically reduces training time
2. **Final model quality**: Affects whether network reaches optimal solution
3. **Stability**: Prevents gradient explosion or [vanishing](/vanishing-gradients-eng.html)
4. **Reproducibility**: Different initializations lead to different solutions for non-convex problems

Without careful initialization, networks may get stuck in poor local minima or fail to train at all.

## The Zero Initialization Problem

The most obvious initialization strategy—setting all weights to zero—fails catastrophically:

```
If W = 0, then all neurons compute identical outputs
All backprop gradients are identical
Layer learns nothing (all neurons remain identical)
```

This failure highlights why randomization is essential. However, random initialization must be carefully scaled.

## Basic Random Initialization

Simply drawing from a normal distribution N(0, 1) creates problems in deep networks:

- **Deep layers**: Activations grow or shrink exponentially through layers
- **Gradient flow**: Gradients vanish in deep networks
- **Variance mismatch**: Network output variance changes per layer

Consider a layer: y = σ(Wx + b). If W ~ N(0, 1) and x has variance 1, then y has exploding/[vanishing](/vanishing-gradients-eng.html) variance depending on layer depth.

## Xavier/Glorot Initialization

Xavier initialization maintains consistent activation variance across layers:

```
W ~ Uniform[-√(6/(n_in + n_out)), √(6/(n_in + n_out))]
or equivalently
W ~ Normal(0, √(2/(n_in + n_out)))
```

Key idea: Scale initialization variance based on fan-in (n_in) and fan-out (n_out).

**Design principle**:
- Variance of pre-activation = 1
- Maintain backward gradient variance ≈ 1

**Best for**: Sigmoid and tanh [activation functions](/activation-functions-eng.html)

**Limitation**: Assumes linear activation between layers (approximately true for sigmoid/tanh, not for ReLU)

## He/Kaiming Initialization

Developed for ReLU networks, He initialization accounts for ReLU's property:

```
W ~ Normal(0, √(2/n_in))  [He uniform version exists too]
```

ReLU kills half the neurons (negative activations → 0), so initialization variance must increase by 2× to maintain overall variance.

**Key insight**: Only half the pre-activations are active
- Xavier: assumes all neurons contribute to output
- He: accounts for ReLU's zeroing of negative values

**Scaling factor**:
```
initialization_scale = √(2/n_in)  [for ReLU]
versus
initialization_scale = √(1/n_in)  [for sigmoid/tanh via Xavier]
```

**Best for**: ReLU, LeakyReLU, ELU, and other ReLU-family activations

## Orthogonal Initialization

For recurrent and convolutional networks, orthogonal initialization uses orthogonal matrices:

```
W is initialized as orthogonal matrix (W^T W = I)
Preserves input norms through network
Prevents gradient explosion/vanishing
```

Benefits:
- All singular values = 1, perfect for gradient flow
- Particularly effective for RNNs with many layers
- Works well in combination with proper scaling

Implementation: Use QR decomposition of random Gaussian matrix.

## Layer-Sequential Unit-Variance (LSUV)

LSUV fine-tunes initialization after computing through network:

1. Initialize weights (e.g., Xavier)
2. Forward pass through network to compute output variance
3. Adjust weights so that output variance ≈ 1
4. Repeat for each layer

Advantages:
- Empirically superior to standard methods
- Addresses actual variance after nonlinearities
- Works better across different [activation functions](/activation-functions-eng.html)
- More expensive (requires forward passes during init)

## Initialization for Different Activation Functions

| Activation | Method | Scale | Reason |
|-----------|--------|-------|--------|
| Sigmoid/Tanh | Xavier | √(1/n_in) | Kills gradients beyond [-1,1], needs careful scaling |
| ReLU | He | √(2/n_in) | Half neurons inactive, needs 2× variance |
| Leaky ReLU | He variant | √(2/(n_in(1+α²))) | Accounts for leak coefficient α |
| ELU | He | √(2/n_in) | Similar to ReLU |
| SELU | SELU-specific | √(1/n_in) | Network is self-normalizing |
| Linear (output) | Xavier/He | Depends | Match previous layer |

## Residual Networks and Initialization

For networks with [skip connections](/residual-connections-eng.html) (ResNets):
- Standard He initialization still works
- [Skip connections](/residual-connections-eng.html) ameliorate [vanishing gradient](/vanishing-gradients-eng.html) problem
- Allows training of very deep networks
- But initialization still matters for early training stability

## Batch Normalization Interaction

With [batch normalization](/batch-normalization-eng.html):
- Initialization becomes less critical
- [Batch norm](/batch-normalization-eng.html) absorbs scaling effects
- Can use wider initialization ranges
- Still good practice to use He or Xavier

## Modern Deep Learning Initialization

Current best practices:
1. **Use He for ReLU networks**: Default choice for modern architectures
2. **Use proper scaling**: Don't just use N(0, 1)
3. **Validate empirically**: Check activation distributions early in training
4. **Consider architecture**: [Transformers](/llm-eng.html), [skip connections](/residual-connections-eng.html), normalization layers affect needs
5. **Don't overthink it**: Most modern frameworks default to good initialization

## Common Initialization Mistakes

- Using N(0, 1) without scaling
- Same initialization for all layers
- Not adjusting for ReLU vs sigmoid
- Ignoring [batch norm](/batch-normalization-eng.html)'s effects
- Initializing biases randomly (usually b = 0)
- Not understanding the underlying motivation

## Practical Initialization Checklist

When implementing a new architecture:
1. Choose [activation function](/activation-functions-eng.html)(s) used
2. Select appropriate initialization method
3. Run network and check first-layer activation [statistics](/probability-eng.html)
4. Verify that activations don't explode/vanish
5. Monitor gradient norms during early training
6. Adjust if necessary (scale variance, try different method)

## Verification Methods

Check if initialization is effective:
```python
# After initialization, before training:
# Check activation statistics
assert activation_mean ≈ 0
assert activation_std ≈ 0.5-1.0

# Gradient norms should be similar across layers
for layer in network:
    assert gradient_norm ≈ constant
```

## Conclusion

Weight initialization is not just a technical detail—it's foundational to successful deep learning. Xavier and He initialization provide theoretically motivated approaches that prevent common pathologies. Understanding the reasoning behind different initialization schemes—accounting for layer depth, [activation functions](/activation-functions-eng.html), and network architecture—enables practitioners to make informed choices.

Modern frameworks make good initialization the default, but understanding these principles helps debug training failures and design better architectures. The key insight remains: properly scale random weights based on network structure and [activation functions](/activation-functions-eng.html) to maintain stable forward and backward information flow.
