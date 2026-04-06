---
layout: default
title: Mixed Precision Training
lang: en
---

# Mixed Precision Training

## Introduction

Mixed precision training combines different numerical precision formats during neural network training. Instead of using 32-bit floating point (FP32) for all operations, mixed precision training strategically uses 16-bit formats (FP16 or BF16) for certain computations while maintaining FP32 for others. This approach dramatically reduces memory consumption and increases training speed with minimal impact on model accuracy.

## Precision Formats: FP32, FP16, and BF16

**FP32 (Float32)**:
- 32-bit format: 1 sign bit, 8 exponent bits, 23 mantissa bits
- Range: ±1.4×10⁻⁴⁵ to ±3.4×10³⁸
- High precision, standard format for deep learning
- Higher memory and computation cost

**FP16 (Float16)**:
- 16-bit format: 1 sign bit, 5 exponent bits, 10 mantissa bits
- Range: ±6.1×10⁻⁵ to ±65,504
- 2x memory reduction compared to FP32
- Faster computation on GPUs with tensor cores
- Limited numerical range causes overflow/underflow issues

**BF16 (Brain Float16)**:
- 16-bit format: 1 sign bit, 8 exponent bits, 7 mantissa bits
- Range: ±1.4×10⁻⁴⁵ to ±3.4×10³⁸ (same as FP32!)
- Maintains FP32 dynamic range with FP16 memory
- Developed by Google for better stability
- Superior to FP16 for many applications

## Memory and Speed Benefits

**Memory Reduction**:
- Activations stored in FP16/BF16: 50% reduction
- Gradients in FP16/BF16: 50% reduction
- Optimizer states in FP32: necessary for stability
- Overall: 20-50% memory reduction (depends on architecture)

**Computation Speed**:
- NVIDIA Tensor Cores: specialized hardware for matrix operations
- FP16 tensor operations: up to 8x faster than FP32 on modern GPUs
- Bandwidth reduction: half the memory transfers
- Actual speedup: 2-4x in practice (limited by memory bandwidth)

## The Challenge: Numerical Stability

Training with low precision creates numerical challenges:

**Gradient Underflow**: Gradients become too small to represent in FP16, losing important information

**Loss Scaling Solution**:
```
1. Compute loss in FP32
2. Scale loss by large factor (typically 2^15 or 2^16)
3. Compute gradients of scaled loss (prevents underflow)
4. Unscale gradients before optimizer step
5. Update parameters in FP32
```

**Dynamic Loss Scaling**:
- Static scaling may scale too much (overflow) or too little (ineffective)
- Dynamically adjust scaling factor based on overflow detection
- Algorithm:
  1. Attempt training step with current scale
  2. If overflow detected, reduce scale (divide by 2)
  3. If no overflow for N steps, increase scale (multiply by 2)
- PyTorch/TensorFlow implementations handle this automatically

## NVIDIA Tensor Cores

Tensor Cores are specialized hardware units for accelerating matrix multiplication:

**Architecture**:
- Each core performs: D = A×B + C in one operation
- A and B in FP16 or BF16, accumulate in FP32
- 8x4x4 operations per clock
- Modern GPUs have 64-80 Tensor Cores per SM

**Performance**:
- V100: 125 TFLOPS FP16 tensor ops vs 15.7 TFLOPS FP32
- A100: 312 TFLOPS BF16 vs 19.5 TFLOPS FP32
- Approximately 16x theoretical speedup for pure tensor operations

**Utilization Requirements**:
- Batch size and matrix dimensions should be multiples of 8
- Aligned memory access patterns
- Large matrices (>32×32) to amortize overhead

## Practical Implementation: PyTorch AMP

**Automatic Mixed Precision (AMP)**:
- PyTorch's native mixed precision support
- Two approaches: torch.cuda.amp and newer torch.amp

**Using torch.cuda.amp**:

```python
from torch.cuda.amp import autocast, GradScaler

model = MyModel().cuda()
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
scaler = GradScaler()

for epoch in range(epochs):
    for batch in dataloader:
        optimizer.zero_grad()

        # Forward pass in lower precision
        with autocast():
            outputs = model(batch['input'])
            loss = criterion(outputs, batch['target'])

        # Backward pass with loss scaling
        scaler.scale(loss).backward()
        scaler.unscale_(optimizer)
        scaler.step(optimizer)
        scaler.update()
```

**Key Components**:
- `autocast()`: Automatically casts operations to FP16/BF16
- `GradScaler()`: Handles loss scaling automatically
- Operations automatically casted based on input types

**TensorFlow Implementation**:

```python
optimizer = tf.keras.optimizers.SGD(learning_rate=0.01)
loss_scale = tf.keras.mixed_precision.LossScale()

with tf.GradientTape() as tape:
    outputs = model(inputs)
    loss = criterion(outputs, targets)
    scaled_loss = loss_scale.scale(loss)

gradients = tape.gradient(scaled_loss, model.trainable_variables)
gradients = loss_scale.unscale(gradients)
optimizer.apply_gradients(zip(gradients, model.trainable_variables))
```

## Best Practices and Considerations

**Batch Size**:
- Larger batches (128-512) often needed with mixed precision
- Memory savings allow larger batches
- Improved gradient stability with larger batches

**Learning Rate**:
- May need adjustment compared to FP32 training
- Often slightly lower learning rates work better
- Learning rate scaling techniques (linear warmup) recommended

**Gradient Clipping**:
- Norm-based clipping helps prevent overflow
- Clip after unscaling: `clip_grad_norm_(model.parameters(), max_norm=1.0)`
- Prevents extreme gradient values

**Batch Normalization**:
- Keep batch norm computations in FP32
- Helps maintain statistics accuracy
- Modern frameworks do this automatically

**Initialization**:
- Standard He initialization works fine
- No special initialization typically needed
- Some models benefit from careful initialization

## Techniques for Improved Training

**Automatic Gradient Accumulation**:
- Accumulate gradients in FP32 even with FP16 compute
- Improves convergence stability
- Built into most AMP implementations

**Master Weight Copies**:
- Keep master weights in FP32
- Forward/backward in FP16
- Optimizer updates FP32 master, copy to FP16 for next iteration

**Precision Tuning**:
- Some operations more sensitive than others
- Softmax typically requires FP32 for stability
- Layer norm often benefits from higher precision
- PyTorch's `cast_to_lowest_precision` parameter allows fine control

## Compatibility and Limitations

**Hardware Requirements**:
- FP16 tensor ops: NVIDIA GPUs with compute capability ≥ 7.0 (V100, A100)
- BF16: NVIDIA A100 and newer
- AMD GPUs: RDNA series and newer
- Intel Habana Gaudi

**Model Architecture Sensitivity**:
- Standard architectures (ResNet, Transformer, VGG): work great
- Models with very deep layers or skip connections: sometimes need extra care
- Custom operations may not have optimized FP16 kernels

**Training Dynamics**:
- Loss curves may differ slightly from FP32
- Final model accuracy typically within 0.1-0.5% of FP32
- Convergence patterns may vary

## Performance Measurement

**Actual Speedup Formula**:
```
Speedup ≈ 2.5x to 4x (real-world)
(Theoretical maximum with Tensor Cores is 8-16x, limited by memory bandwidth and other factors)
```

**Benchmarking**:
1. Measure FP32 training time per epoch
2. Measure mixed precision training time per epoch
3. Verify model accuracy within 0.5% of original
4. Calculate actual speedup ratio

## When to Use Mixed Precision Training

**Use When**:
- GPU with Tensor Cores available (V100 or newer NVIDIA)
- Training time is critical
- Memory is a bottleneck
- Model is large (BERT, ResNet-50, etc.)
- Batch sizes can be increased

**Skip When**:
- Limited precision GPUs (older NVIDIA, consumer GPUs)
- Running on CPUs (no benefit)
- Training very small models
- Precision requirements are critical (financial data)

## Conclusion

Mixed precision training is a powerful optimization technique that provides substantial speedup and memory savings with minimal accuracy loss. By leveraging specialized hardware like Tensor Cores and combining intelligent loss scaling, modern deep learning frameworks make mixed precision training both easy to use and highly effective. As models grow larger and computational resources remain constrained, mixed precision has become a standard practice in production deep learning systems.

