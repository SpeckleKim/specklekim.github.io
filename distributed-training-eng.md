---
layout: default
title: Distributed Training
lang: en
---

# Distributed Training

## Introduction

Distributed training parallelizes [neural network](/neural-eng.html) training across multiple GPUs, TPUs, or machines to reduce training time and enable training of models too large for single-device memory. As models grow from millions to billions of parameters, distributed training has become essential for practical deep learning. This guide covers the fundamental strategies, communication patterns, and modern systems enabling efficient distributed training.

## Fundamental Training Parallelization Strategies

There are three primary approaches to distribute training, often combined:

**Data Parallelism**:
- Replicate model on each device
- Each device processes different batch of data
- Gradients aggregated across devices (AllReduce)
- Simplest approach, scales well up to ~8 GPUs
- Limited by model size fitting on single [GPU](/gpu-hardware-eng.html)

**Model Parallelism**:
- Split model across devices
- Each device holds subset of model parameters
- Activations/gradients passed between devices
- Enables training of large models exceeding single [GPU](/gpu-hardware-eng.html) memory
- Higher communication overhead, more complex

**Pipeline Parallelism**:
- Split model into stages along depth dimension
- Each device processes one stage
- Stages pipeline batches for efficiency
- Reduces bubble time (idle device waiting)
- Enables large models with asynchronous execution

## Data Parallelism in Detail

**Basic Approach**:
1. Copy model to each of N devices
2. Distribute data: each device gets batch_size/N samples
3. Each device computes forward pass and gradients independently
4. Aggregate gradients across all devices (AllReduce)
5. All devices update model with aggregated gradient

**Synchronous vs Asynchronous**:

*Synchronous*:
- All devices complete iteration before updating
- Slower device determines overall speed
- Consistent convergence properties
- Standard in production

*Asynchronous*:
- Devices update independently with latest parameters
- Faster overall throughput
- Potential staleness: some devices use outdated parameters
- Can diverge without careful tuning

**Scaling Efficiency**:
```
Speedup = min(N, efficiency_factor × N)
```

Where efficiency_factor accounts for:
- Communication overhead
- Load imbalance
- Synchronization costs

Linear scaling (efficiency = 1) is ideal but rarely achieved beyond 8 devices.

## Model Parallelism Strategies

**Tensor Parallelism**:
- Split weight matrices across devices along a dimension
- Example: For linear layer Y = XW, split W vertically (columns) or horizontally (rows)
- Requires communication for every matrix multiplication
- Fine-grained parallelism

**Sequence Parallelism** (for [Transformers](/llm-eng.html)):
- Split along sequence dimension
- Different devices process different tokens
- Reduces memory per device proportionally to sequence length
- Effective for long sequences

**Pipeline Parallelism Implementation**:
```
Device 0: Layers 1-6
Device 1: Layers 7-12
Device 2: Layers 13-18

Batch Split: [micro-batch 1, micro-batch 2, micro-batch 3, ...]

Timeline:
Device 0: F(1) → F(2) → F(3) → B(1) → B(2) → B(3)
Device 1: _ → F(1) → F(2) → F(3) → B(1) → B(2)
Device 2: _ → _ → F(1) → F(2) → F(3) → B(1)

(F=forward, B=backward, numbers indicate micro-batch)
```

Ideal: Later devices already processing next micro-batch while earlier devices [backprop](/backpropagation-eng.html).

## Modern Distributed Training Frameworks

**Fully Sharded Data Parallel (FSDP)** (PyTorch):
- Breaks model into shards, each shard on one device
- Forward pass: gather shard parameters, compute, discard after use
- Backward pass: gather shard parameters again, compute gradients, discard
- [Optimizer](/optimizers-eng.html) state per shard (not replicated)
- Reduces memory from 3N to 3+N/K (where K = number of devices)

**DeepSpeed ZeRO** (Microsoft):
- **Stage 1**: Shard [optimizer](/optimizers-eng.html) states (4x reduction)
- **Stage 2**: Shard gradients additionally (2x reduction)
- **Stage 3**: Shard parameters additionally (K× reduction)
- Combined: ~16x memory reduction with 8 devices
- Includes CPU offloading: spill to CPU RAM for further reduction

**Megatron-LM** (NVIDIA):
- Combines pipeline and tensor parallelism
- Pipeline parallelism: stages on different devices
- Tensor parallelism: within each stage, split matrices
- Extremely efficient for training [large language models](/llm-eng.html)

## Communication Patterns

**AllReduce Operation**:
```
Before:
Device 0: [1, 2]
Device 1: [3, 4]
Device 2: [5, 6]

After AllReduce(Sum):
Device 0: [9, 12]
Device 1: [9, 12]
Device 2: [9, 12]
```

All devices end with aggregated result. Used for:
- Averaging gradients across devices
- Synchronizing model state

**Ring AllReduce** (efficient for large clusters):
- Devices arranged in logical ring
- Each device sends to next, receives from previous
- Multiple rounds: each round transfers different data portions
- Complexity: ~2(N-1) data transfers for N devices
- Bandwidth efficient: saturates network links

```
Ring AllReduce Steps (4 devices):
Round 1: 0→1→2→3→0
Round 2: 3→0→1→2→3
Round 3: 2→3→0→1→2
```

**Tree Reduction**:
- Hierarchical structure like binary tree
- Reduce phase: combine upward to root
- Broadcast phase: distribute downward from root
- Lower latency than Ring for small data, higher for large

**Parameter Server**:
- Central server holds model parameters
- Workers fetch parameters, compute gradients, send updates
- Server applies updates
- Less scalable, less commonly used in modern systems

## Communication Efficiency Techniques

**Gradient Compression**:
- Quantize gradients to lower precision (8-bit)
- Sparsify: send only largest gradients
- Trade-off: reduced communication vs. slower convergence
- Top-k sparsification: only send k% largest gradients

**Gradient Accumulation**:
- Accumulate gradients over multiple iterations
- Periodic AllReduce instead of every iteration
- Reduces communication frequency
- Decreases synchronization overhead

**Batch Size Scaling**:
- When doubling devices, double batch size
- Maintains computation/communication ratio
- Requires adjusting [learning rate](/learning-rate-scheduling-eng.html) (learning rate scaling)
- Linear scaling rule: lr_new = lr_old × (batch_size_new / batch_size_old)

## Practical Considerations

**Load Balancing**:
- Uneven data distribution causes device idle time
- Solution: data shuffling, dynamic task assignment
- Synchronous training suffers more from imbalance

**Convergence**:
- Larger batch sizes can hurt convergence
- LARS [optimizer](/optimizers-eng.html): Layer-wise Adaptive Rate Scaling
- Adjusts [learning rate](/learning-rate-scheduling-eng.html) per layer based on gradient/weight ratio
- Enables large batch training with stable convergence

**Fault Tolerance**:
- Periodic checkpointing of model and [optimizer](/optimizers-eng.html) state
- Recovery from device failures requires resuming from last checkpoint
- Checkpoint frequency: trade-off between storage and recovery time

## Scaling Efficiency Analysis

**Weak Scaling**:
- Fix total compute per device, increase devices
- Increase problem size proportionally
- Ideal: constant time for fixed problem per device
- Example: training with 256 batch size on 8 GPUs vs 2048 batch size on 64 GPUs

**Strong Scaling**:
- Fix problem size, increase devices
- Time decreases with more devices (ideally linearly)
- Limited by communication becoming dominant
- Practical limit: ~8-16 devices per model for pure data parallelism

**Speedup Formula**:
```
Time = Computation_time/N_devices + Communication_time
Efficiency = Speedup/N_devices = T(N=1) / (N × T(N))
```

## Representative Frameworks and Tools

**PyTorch Distributed**:
- `torch.nn.parallel.DistributedDataParallel`: data parallelism
- `torch.distributed`: low-level communication (AllReduce, Send/Recv)
- FSDP for full model sharding

**TensorFlow Distribution Strategy**:
- `tf.distribute.MirroredStrategy`: data parallelism on single machine
- `tf.distribute.MultiWorkerMirroredStrategy`: multi-machine
- Built-in gradient aggregation and synchronization

**Horovod** (Uber):
- Framework-agnostic distributed training
- Ring-allreduce for communication efficiency
- Works with PyTorch, TensorFlow, MXNet

## Challenges and Limitations

- **Synchronization Overhead**: Increases with more devices
- **Network Bandwidth**: Bottleneck for large clusters
- **Complexity**: Debugging distributed systems is harder
- **Load Imbalance**: Stragglers limit overall throughput
- **Convergence**: Larger batches can hurt final accuracy if not carefully tuned

## Conclusion

Distributed training enables training of massive models and datasets by parallelizing computation across multiple devices. Modern frameworks like FSDP, DeepSpeed, and Megatron combine multiple parallelization strategies to achieve near-linear scaling on clusters of GPUs. Understanding the trade-offs between data, model, and pipeline parallelism, along with communication patterns and efficiency techniques, is essential for practitioners training state-of-the-art models at scale.

