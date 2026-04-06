---
layout: default
title: Learning Rate Scheduling
lang: en
---

# Learning Rate Scheduling: Dynamic Optimization Strategy

Learning rate scheduling is the practice of adjusting the learning rate during training. A fixed learning rate often represents a compromise: high enough for initial progress but low enough to avoid divergence. Dynamic scheduling adapts the learning rate to different training phases, dramatically improving convergence speed and final model performance.

## The Constant Learning Rate Problem

Using a single fixed learning rate throughout training has inherent limitations:

```
High learning rate: Fast initial progress, but overshoots minima
Low learning rate: Precise optimization, but slow early convergence
Trade-off: Fixed rate is neither optimal for early nor late training
```

Early training benefits from aggressive learning (large steps), while fine-tuning requires careful small steps. No single constant rate handles both phases well.

## Step Decay

The simplest and most intuitive scheduling approach reduces learning rate at fixed intervals:

```
Every N epochs, multiply learning rate by decay_factor (e.g., 0.1)
lr(epoch) = lr_0 * decay_factor^(epoch // N)

Example: Start at 0.1, reduce to 0.01 after 30 epochs, 0.001 after 60
```

Characteristics:
- Simple to implement and understand
- Works well in practice for many tasks
- Hyperparameters: initial lr, decay factor, schedule intervals
- Discrete jumps may cause brief instability

**Common schedules**:
- Reduce by 0.1 every 30 epochs (ResNets)
- Reduce by 0.5 every 10 epochs (RNNs)
- Task-dependent and often found via grid search

## Exponential Decay

Continuous exponential decay gradually reduces learning rate:

```
lr(t) = lr_0 * exp(-λt)  where λ is decay rate
or discretely:
lr(epoch) = lr_0 * decay_rate^epoch
```

Advantages:
- Smooth continuous reduction
- No sudden jumps in learning rate
- Flexible control via decay rate parameter
- Smooth gradient flow without discontinuities

Disadvantages:
- Hyperparameter selection (decay rate) requires tuning
- May reduce too quickly or too slowly depending on schedule length

## Cosine Annealing

Inspired by annealing in statistical mechanics, cosine annealing smoothly reduces learning rate:

```
lr(t) = lr_min + (lr_max - lr_min) * (1 + cos(πt/T)) / 2
where t is current iteration, T is total training iterations
```

Key properties:
- Starts at lr_max and decays to lr_min
- Smooth cosine function (no abrupt changes)
- Completes full decay cycle in T iterations
- Mathematically elegant and well-motivated

**Advantages**:
- Smooth transition without jumps
- Reaches near-zero learning rate by end of training
- Empirically excellent performance
- Requires knowledge of total training iterations

## Warmup Strategy

Many modern training procedures use learning rate warmup—gradually increasing learning rate early in training:

```
First N_warmup steps: linear increase from 0 to lr_max
Remaining steps: decay (cosine, step, exponential)

lr(t) = (t / N_warmup) * lr_max,  for t < N_warmup
```

Why warmup helps:
- Allows network to adjust weights with small steps initially
- Prevents divergence from poor initialization
- Stabilizes training, especially with [batch norm](/batch-normalization-eng.html)
- Particularly important for [Transformers](/llm-eng.html)
- Common in deep learning with [batch normalization](/batch-normalization-eng.html)

**Warmup duration**: Typically 5-10% of total training steps

## OneCycleLR

Combines increasing and decreasing phases in a single cycle, inspired by cyclical learning rate techniques:

```
First half: increase from lr_min to lr_max (linear or exponential)
Second half: decrease from lr_max to lr_min
Total: 1 cycle over full training
```

Implementation details:
- Momentum inversely varies with learning rate
- Works with momentum-based [optimizers](/optimizers-eng.html) ([SGD](/optimizers-eng.html) with momentum, [Adam](/optimizers-eng.html) variant)
- Can use fraction parameter for uneven cycle (e.g., 30% increase, 70% decrease)

Advantages:
- Excellent empirical results
- Simple two-phase approach
- [Regularization](/regularization-eng.html) effect from learning rate variation
- Reduces need for extensive [hyperparameter tuning](/hyperparameter-tuning-eng.html)

## Cyclical Learning Rate

Goes beyond single cycle with periodic learning rate oscillations:

```
lr oscillates between lr_min and lr_max periodically
Can use triangular (linear ramp up/down), exponential, or other waveforms
Period can be fixed or adaptive
```

Intuition:
- Helps escape sharp minima
- Simulates ensemble-like behavior through rate oscillations
- Can reduce overfitting
- May improve generalization

Variants:
- Triangular: Linear increase/decrease
- Triangular2: Amplitude decreases with iterations
- Exponential: Smooth exponential increase/decrease

## Adaptive Learning Rate

Some schedulers adapt based on training progress:

**Validation-based**: Monitor validation loss, reduce when plateaus
**Loss-based**: Reduce when training loss stops improving
**Gradient-based**: Adapt based on gradient [statistics](/probability-eng.html)

These require monitoring during training (not just schedule based on time).

## Warm Restarts

Learning rate restarts periodically allow escaping local minima:

```
Multiple restart cycles, each with cosine annealing
lr(t) = lr_min + (lr_max - lr_min) * (1 + cos(π(t mod T_cur)/T_cur)) / 2
Where T_cur grows with each restart
```

**Benefits**:
- Each restart represents "fresh start" with high learning rate
- Explores different regions of loss landscape
- Can improve final model quality
- Combines exploration (high lr) and exploitation (low lr)

## LR Finder

Practical technique to find good learning rate range before training:

1. Start with small learning rate
2. Gradually increase learning rate while training (1-2 epochs)
3. Record loss at each learning rate
4. Plot loss vs learning rate
5. Choose learning rate where loss is decreasing fastest

```python
# Pseudocode
best_loss = infinity
for lr in log_range(1e-5, 1):
    train_one_batch()
    if loss > 4 * best_loss:
        break  # Stop if loss explodes
    record(lr, loss)
plot(lrs, losses)
# Choose lr in steep decrease region, not minimum
```

**Advantages**:
- Principled approach to learning rate selection
- Often beats manual tuning
- Quick to run (1-2 epochs)
- Finds range, not exact value

**Limitations**:
- Works best when single cycle will be used
- May not be optimal for learning rate schedules
- Requires interpretation of the plot

## Practical Guidelines

When choosing a learning rate schedule:

1. **For standard supervised learning**: Start with step decay or cosine annealing
2. **For Transformers**: Warmup (typically 10k steps) + cosine annealing
3. **For CNNs**: Step decay with milestone epochs works well
4. **For RNNs**: Exponential decay or step decay, often with smaller starting rates
5. **For fine-tuning**: Lower learning rates and potentially different schedule
6. **When uncertain**: Use LR finder first to establish good baseline

## Comparison of Schedules

| Schedule | Smoothness | Performance | Complexity | Best For |
|----------|-----------|-------------|-----------|----------|
| Constant | - | Baseline | None | Simple cases |
| Step Decay | Discrete | Good | Low | CNNs, standard tasks |
| Exponential | Smooth | Good | Medium | Long training |
| Cosine | Smooth | Excellent | Medium | Modern deep learning |
| Warmup+Cosine | Smooth | Excellent | Medium | [Transformers](/llm-eng.html), large models |
| OneCycleLR | Smooth | Excellent | Low | Fast training, small data |
| Cyclical | Periodic | Good | Medium | Exploration needed |
| Warm Restarts | Periodic | Good | Medium | Long training, multiple goals |

## Implementation Considerations

- **Batch size interaction**: Larger batches often need larger learning rates
- **Model architecture**: Deeper models may need more careful scheduling
- **Optimizer choice**: Different [optimizers](/optimizers-eng.html) ([SGD](/optimizers-eng.html), [Adam](/optimizers-eng.html)) may need different schedules
- **Data augmentation**: Stronger augmentation may allow higher learning rates
- **Convergence monitoring**: Track both training and validation metrics

## Common Mistakes

- Starting with too high learning rate (will diverge)
- Not adjusting schedule when changing batch size
- Using schedule designed for one [optimizer](/optimizers-eng.html) with different optimizer
- Not considering warmup for complex architectures
- Forgetting that schedule affects final model quality, not just convergence
- Setting learning rate too low (underutilizes training time)

## Conclusion

Learning rate scheduling transforms training from a compromise (single rate) into an optimized process. Modern approaches like cosine annealing with warmup have become standard in state-of-the-art deep learning. The key insight is matching the learning rate to training phase: large exploratory steps early, refined smaller steps later.

Whether using classical step decay or contemporary cyclical methods, thoughtful learning rate scheduling is essential for reaching optimal solutions efficiently. Combined with good initialization and [regularization](/regularization-eng.html), it forms the foundation of successful deep learning training.
