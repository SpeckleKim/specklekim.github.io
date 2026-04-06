---
layout: default
title: Mixture of Experts (MoE)
lang: en
---

# Mixture of Experts (MoE)

## From Dense to Sparse Models

Traditional [transformer](/llm-eng.html) models are **dense**: every token flows through all layers and all parameters. This means inference cost scales directly with model size. A 70B parameter dense model processes every parameter for every token, leading to predictable but high compute requirements.

**Mixture of Experts (MoE)** takes a different approach. Instead of dense layers, MoE uses sparse layers where only a subset of parameters activates for each token. This enables models to scale parameter count dramatically while keeping per-token computation relatively constant.

### Dense vs Sparse Trade-offs

**Dense Models**:
- Predictable, stable performance
- High computational cost for inference
- Every token uses all capacity
- Easier to optimize and deploy

**Sparse Models (MoE)**:
- Lower inference cost per token
- Higher peak memory during training
- Load balancing challenges
- Potential for better efficiency

## The Gating Mechanism

The core of MoE is the **gating network**. For each token, the gating function determines which experts process it. This is typically a learned [neural network](/neural-eng.html) that outputs a gate vector—usually softmax probabilities over expert choices.

### How Gating Works

For each token x at position i:
1. **Compute gate scores**: `g(x) = softmax(W_gate * x)`
2. **Select top-k experts**: Keep only the top k experts by gate score
3. **Compute expert outputs**: Expert j computes `E_j(x)`
4. **Combine results**: Weighted sum using gate scores

The result for token x is: `y = Σ g(x)_j * E_j(x)` (summed over selected experts)

This gating is fully differentiable, allowing the model to learn which experts to route tokens to during training.

## Top-k Routing

Most practical MoE systems use **top-k routing**: for each token, select the top k experts by gate score, where k is typically 2-8. This sparse routing is key to efficiency gains.

### Benefits of Top-k
- **Computational savings**: Only compute k experts instead of all E experts
- **Memory efficiency**: Reduced memory footprint compared to dense equivalents
- **Specialization**: Experts can specialize in different patterns

### The Load Balancing Problem

A challenge with learned routing: experts may become imbalanced. If tokens consistently route to a few "easy" experts, others become inactive, wasting capacity. This is called **expert collapse** or **load imbalance**.

Solutions include:
- **Auxiliary loss**: Add penalty for uneven expert utilization
- **Load balancing gates**: Modify routing to encourage balanced expert usage
- **Expert dropout**: Probabilistically drop experts during training
- **Fixed routing**: Pre-assign tokens to experts in balanced manner

## Notable MoE Architectures

### Mixtral 8x7B

Mistral's Mixtral model demonstrates practical MoE effectiveness. It has 8 experts (each 7B parameters) with top-2 routing. Despite 46.7B total parameters, it only activates 12.9B per token—enabling competitive performance with 4x fewer active parameters than a 34B dense model.

Key features:
- Trained with balanced loads through careful auxiliary loss tuning
- Outperforms denser models on many [benchmarks](/benchmarks-eng.html) with lower inference cost
- Shows that MoE can be production-ready for consumer-grade inference

### Switch Transformer

Google's Switch [Transformer](/llm-eng.html) simplified MoE by using **top-1 routing** instead of top-k. Each token routes to exactly one expert, maximizing sparsity and simplicity. Though simpler, empirical results show competitive performance compared to top-k approaches.

Key findings:
- Top-1 routing simplifies implementation and reduces communication overhead
- Enables dramatic parameter scaling with modest inference cost increases
- Works well for very large models (hundreds of billions of parameters)

### GShard

GShard demonstrated MoE at massive scale, training 600B+ parameter models on TPU clusters. Contributions include:
- **Distributed MoE**: Techniques for sharding experts across devices
- **All-to-all communication**: Routing tokens to distant experts efficiently
- **Scalability patterns**: How to train enormous sparse models

GShard showed that MoE scales to problem sizes infeasible with dense models.

## Load Balancing in Depth

Load balancing is critical because uneven expert usage directly degrades performance.

### Auxiliary Loss

The standard approach adds a [regularization](/regularization-eng.html) term:
```
L_aux = α * Σ (f_i * L_i)
```
where `f_i` is the fraction of tokens routed to expert i, and `L_i` is the loss for that expert. This penalizes deviation from balanced utilization.

However, auxiliary loss can conflict with optimization—forcing balanced routing may prevent natural specialization.

### Capacity Factors

Another approach uses capacity constraints. Each expert has a maximum capacity (e.g., 1.25x average tokens per expert). Tokens exceeding this are either dropped (risky) or rerouted.

### Expert Dropout

During training, probabilistically disable experts to encourage load balancing. This forces remaining experts to develop robust representations without relying on specific experts.

## Efficiency Gains

The primary benefit of MoE is **computational efficiency during inference**.

### Parameters vs Computation

A 100B parameter MoE model with top-2 routing and 8 experts might only activate 25B parameters per token. This is the key distinction:
- **Parameters**: 100B (memory requirement for model storage)
- **Active computation**: 25B (compute per token)

Compare to a 40B dense model:
- **Parameters**: 40B
- **Active computation**: 40B (per token)

The MoE model has more flexibility because scaling parameters doesn't directly scale per-token cost.

### Empirical Results

Studies show:
- 10-100x parameter increases with only 2-5x compute increases
- Better [scaling laws](/scaling-laws-eng.html) than dense models for fixed compute budgets
- Competitive quality despite sparse activation

However, inference frameworks must be optimized to realize these gains—naive implementations may actually be slower due to routing overhead and irregular memory access patterns.

## Challenges and Trade-offs

### Training Complexity
- Requires careful auxiliary loss tuning
- Load balancing can be unstable early in training
- More hyperparameters to tune

### Communication Overhead
- All-to-all communication between devices for token routing
- Can saturate network bandwidth on clusters
- Makes [distributed training](/distributed-training-eng.html) more complex

### Memory Requirements
- All experts must fit in cluster memory
- Can require more total cluster memory despite sparse activation
- Load balancing often requires token duplication across devices

### Hardware Limitations
- GPUs aren't optimized for sparse operations
- Routing overhead can negate theoretical speedups
- Requires careful engineering for practical gains

## When to Use MoE

**Good for:**
- Very large model capacity requirements
- Inference throughput-sensitive applications
- Scenarios with varied input types (experts specialize)
- Budget-constrained deployments (more capacity per compute dollar)

**Not ideal for:**
- Latency-critical applications (routing adds overhead)
- Single-[GPU](/gpu-hardware-eng.html) deployment (routing becomes inefficient)
- Memory-constrained scenarios (need to store all experts)

## Future Directions

Active research includes:
- **Hierarchical MoE**: Nested expert selection for better scaling
- **Adaptive top-k**: Dynamically adjust routing budget per token
- **Expert consolidation**: Training to reduce total expert count
- **Improved load balancing**: New auxiliary loss formulations
- **Hardware optimization**: CPU/[GPU](/gpu-hardware-eng.html) support for sparse MoE operations

MoE represents an important paradigm for building more efficient large models. As inference becomes economically critical, sparse models will likely play an increasingly central role in [LLM](/llm-eng.html) deployment.
