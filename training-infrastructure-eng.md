---
layout: default
title: "AI Training Infrastructure"
lang: en
---

# AI Training Infrastructure

## Introduction

Training large AI models requires specialized infrastructure: distributed [GPU](/gpu-hardware-eng.html) clusters, optimized networking, fault tolerance, and monitoring. Modern large models ([LLMs](/llm-eng.html), [diffusion models](/diffusion-models-eng.html)) require thousands of GPUs across multiple data centers, coordinated training, and efficient communication. Understanding infrastructure is crucial for anyone training models at scale.

## GPU Cluster Architecture

### Single Machine Setup

**Typical high-end machine**:
- 8x A100 or H100 GPUs
- NVLink interconnect between GPUs (425 GB/s per direction on A100)
- 2TB+ CPU RAM
- 100TB+ NVMe storage
- Dual 10GbE network interfaces

**Use cases**: Models up to 50-100B parameters with model parallelism.

### Multi-Machine Clusters

**Local Cluster** (same data center):
- 10-1000 machines
- Connected via InfiniBand (200-400 Gbps)
- Sub-millisecond latency between nodes
- Petabyte-scale storage systems

**Distributed Cluster** (multiple data centers):
- Machines across geographies
- Higher latency, lower bandwidth
- Requires careful synchronization
- Handles regional compute availability

## High-Speed Interconnects

### NVLink

NVIDIA's high-bandwidth [GPU](/gpu-hardware-eng.html)-to-GPU connection:
- A100: 400 GB/s (600 GB/s with NVLink 4)
- H100: 900 GB/s
- Enables all-reduce operations at scale
- Dedicated, not shared with network traffic

### InfiniBand

High-speed networking fabric:
- HDR InfiniBand: 200 Gbps per direction
- NDR InfiniBand: 400 Gbps per direction
- Sub-microsecond latency
- RDMA (Remote Direct Memory Access) support

### 400GbE Ethernet

Emerging standard for data center networking:
- 400 Gbps full duplex
- Lower cost than InfiniBand
- Slightly higher latency
- Good for less bandwidth-constrained applications

## Distributed Training Frameworks

### PyTorch Distributed Data Parallel (DDP)

Simple data parallelism across GPUs:

```python
model = MyModel()
ddp_model = DistributedDataParallel(model)
# Automatic gradient synchronization across devices
```

**How it works**:
1. Copy model to each [GPU](/gpu-hardware-eng.html)
2. Each [GPU](/gpu-hardware-eng.html) processes different batch
3. Compute gradients locally
4. All-reduce to synchronize gradients
5. Update model weights identically

**Limitations**: Model must fit in single [GPU](/gpu-hardware-eng.html) memory.

### Tensor Parallelism

Split model parameters across GPUs:

```
Layer 1: [param_split_1][param_split_2][param_split_3]
         GPU1            GPU2           GPU3
```

- Useful for models too large for single [GPU](/gpu-hardware-eng.html)
- Requires fine-grained synchronization
- Trade-off: communication overhead vs model size

### Pipeline Parallelism

Split model layers across GPUs:

```
GPU1: [Layer1, Layer2]
GPU2: [Layer3, Layer4]
GPU3: [Layer5, Layer6]
```

- Good for very deep models
- GPUs process different layers sequentially
- Requires careful batching strategy (GPipe, Megatron)
- Can lead to [GPU](/gpu-hardware-eng.html) idle time

### Mixture of Experts (MoE)

Different parameters for different inputs:

```
Input -> Router -> Expert 1 -> Output
                -> Expert 2 ->
                -> Expert 3 ->
```

- Sparse computation: activates subset of parameters per input
- Scales parameter count without proportional compute
- Successfully used in Switch [Transformers](/llm-eng.html), Grok models
- Reduces training efficiency slightly (expert imbalance)

## Communication Optimization

### Gradient Compression

Reduce communication volume during synchronization:

**Low-rank Updates**: Communicate only significant gradient updates

**Top-k Sparsification**: Send only largest gradient values

**Quantization**: Reduce precision of gradients before sending

**Benefits**: 5-10x communication reduction with minimal accuracy loss.

### Overlapping Computation and Communication

Hide communication latency:

```
While gradients for layer N are being communicated,
compute gradients for layer N+1
```

Modern frameworks (PyTorch, JAX) support overlapping automatically.

### Ring All-Reduce

Efficient collective operation for many-node clusters:

```
Node 0 -> Node 1 -> Node 2 -> ... -> Node N -> Node 0
Send local gradient segment, receive and aggregate
```

- More efficient than star topology
- Scales better with cluster size

## Checkpointing and Fault Tolerance

### Gradient Checkpointing

Trade computation for memory:

```
Forward pass: compute only checkpoints, discard intermediate activations
Backward pass: recompute intermediate activations from checkpoints
```

- Reduces memory by 50% with minimal slowdown
- Essential for large models

### Model Checkpointing

Periodically save model state:
- At each epoch or fixed interval
- Enables resuming from checkpoint if failure
- Disk space can become bottleneck

**Strategies**:
- Keep only last N checkpoints
- Store on fast storage (NVMe) initially, move to slower archive
- Async writes to avoid blocking training

### Asynchronous Checkpointing

Save while training continues:
- Write to CPU memory, transfer to disk in background
- Reduces training disruption
- Requires sufficient RAM for buffering

### Distributed Checkpointing

Save model state across multiple machines:
- Avoid single point of failure in storage
- Faster save/restore with parallel writes
- Requires coordinated recovery

## Cost Optimization

### Spot Instances

Cheaper preemptible VMs (70-80% discount):
- Risk: instance can be reclaimed anytime
- Mitigation: frequent checkpointing
- Good for non-critical experiments

### Mixed Instance Types

Use different [GPU](/gpu-hardware-eng.html) types:
- Older A100s for less demanding work
- Newer H100s for frontline research
- Reduces per-hour costs

### Batch Training Schedule

Train during off-peak hours:
- 30-50% cost savings in some regions
- Requires flexible training schedule
- Good for non-time-critical work

### Auto-scaling

Dynamically adjust cluster size:
- Start with 100 GPUs, scale to 1000 during peak
- Reduce when demand drops
- Requires infrastructure support (Kubernetes, Ray)

## Monitoring and Debugging

### GPU Utilization

Track actual [GPU](/gpu-hardware-eng.html) computation:
- Ideal: 85-95% utilization (always computing)
- Low utilization: bottleneck in data loading or synchronization
- Tools: nvidia-smi, PyTorch profiler

### Network Bandwidth

Monitor communication patterns:
- All-reduce causing bottleneck? Try communication overlapping
- Uneven traffic? May indicate skewed model parallelism
- Tools: Netstat, custom logging

### Thermal Management

GPUs generate heat:
- Monitor temperature
- Reduce clock speeds if overheating
- Ensure adequate cooling infrastructure
- Training at 80-85°C optimal

### Training Convergence

Track loss curves across checkpoints:
- Anomalies may indicate bugs or hyperparameter issues
- Compare across different training runs
- Use experiment tracking (MLflow, W&B)

## Popular Cloud Platforms

**AWS**: EC2 instances with A100/H100, good spot pricing

**Google Cloud**: TPU clusters and [GPU](/gpu-hardware-eng.html) options, integrates with JAX/TensorFlow

**Azure**: [GPU](/gpu-hardware-eng.html) clusters, integrates with PyTorch ecosystem

**Lambda Labs**: [GPU](/gpu-hardware-eng.html) rental, simple interface, good for experimentation

## Future Directions

- **Photonic interconnects**: Replace electrical InfiniBand with optical
- **Advanced cooling**: Liquid cooling enables higher density
- **Software-defined networks**: More flexible communication patterns
- **Heterogeneous compute**: Mix of GPUs, TPUs, CPUs optimized by task

## Conclusion

Modern AI training requires thinking about infrastructure as seriously as algorithms. Successful training at scale demands understanding [distributed training](/distributed-training-eng.html) patterns, communication optimization, and fault tolerance. As models grow larger, infrastructure becomes increasingly important for feasibility and cost-effectiveness.
