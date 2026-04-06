---
layout: default
title: KV-Cache & Inference Optimization
lang: en
---

# KV-Cache & Inference Optimization

During autoregressive generation, language models predict tokens one by one. Without optimization, this process wastes computation by recomputing attention over previous tokens repeatedly. KV-cache (key-value cache) is a critical optimization that accelerates inference by caching and reusing previously computed keys and values.

## The Autoregressive Bottleneck

Autoregressive decoding generates sequences step-by-step:

```
Step 1: Input "Hello"     → Output "world"
Step 2: Input "Hello world" → Output "is"
Step 3: Input "Hello world is" → Output "beautiful"
```

The fundamental problem: at step t, the model processes all t tokens again to predict token t+1, even though attention for tokens 1..t-1 remains identical.

Example with position sequence of length n:
```
Step 1: Compute attention for [token_1]                    (1 token)
Step 2: Compute attention for [token_1, token_2]           (2 tokens)
Step 3: Compute attention for [token_1, token_2, token_3]  (3 tokens)
...
Step n: Compute attention for [token_1, ..., token_n]      (n tokens)

Total operations: 1 + 2 + 3 + ... + n = n(n+1)/2 = O(n²)
```

This quadratic complexity makes generation extremely slow for long sequences.

## KV-Cache Mechanism

Instead of recomputing keys and values, KV-cache stores them:

```
# Initial forward pass (prefill):
inputs = [token_1, token_2, ..., token_n]
Q_new, K_all, V_all = compute(inputs)
cache = {K: K_all, V: V_all}

# Step n+1 (decode):
new_token = token_n+1
Q_new, K_new, V_new = compute([new_token])
K_cached = cache.K + K_new          # Concatenate
V_cached = cache.V + V_new
attention_output = softmax(Q_new @ K_cached.T) @ V_cached
cache.update({K: K_cached, V: V_cached})
```

Key insight: Only Q for the new token is computed; K and V for previous tokens are reused.

## Memory and Computation Trade-off

KV-cache trades memory for computation speed:

**Without cache (naive):**
- Computation: O(n²) for n-length sequence
- Memory: O(n) per forward pass (no persistence)

**With KV-cache (optimized):**
- Computation: O(n) for n-length sequence (linear!)
- Memory: O(n * d_model) persistent storage

For a 13B parameter model with sequence length 2048:
- K cache: 2048 × 5120 dimensions × 2 bytes (fp16) ≈ 20 MB per head
- V cache: 2048 × 5120 dimensions × 2 bytes ≈ 20 MB per head
- Total for 40 heads: ~1.6 GB per sequence

This explains why batch size is limited during inference—each sequence needs its own cache.

## Multi-Batch Inference and Memory Management

Managing KV-cache for multiple sequences is critical:

```python
# Batched generation
batch_size = 32
sequence_length = 2048
num_heads = 40
head_dim = 128

kv_cache_size = batch_size * sequence_length * num_heads * head_dim * 2 bytes
              = 32 * 2048 * 40 * 128 * 2
              ≈ 52 GB (fp16)
```

Modern optimization strategies:

1. **Batch-wise allocation**: Pre-allocate cache for expected batch size
2. **Dynamic allocation**: Grow cache as sequence length increases
3. **Reuse strategies**: Evict oldest cache entries for new requests
4. **Quantization**: Store cache in int8 or fp8 (reduced precision)

## PagedAttention (vLLM)

PagedAttention, introduced in vLLM, dramatically improves memory efficiency:

**Traditional approach:**
- KV-cache allocated contiguously per sequence
- Fragmentation: partial blocks wasted if sequence ends early
- Inefficient: can't share cache across sequences

**PagedAttention approach:**
- KV-cache divided into fixed "pages" (e.g., 16 tokens per page)
- Pages are virtual (mapped to physical memory)
- Sequences share physical pages (copy-on-write)
- Dynamic memory allocation like OS virtual memory

### Example with 2-token pages:
```
Sequence 1: tokens [0,1] [2,3] [4,5] ...
Sequence 2: tokens [0,1] [2,3] [4,5] ...

If both sequences use the same prefix [0,1][2,3],
they can share the same physical pages!
```

Benefits:
- **Memory efficiency**: 5-10x improvement (60-75% GPU memory savings)
- **Higher throughput**: Batch more sequences with same GPU memory
- **Reduced latency**: Faster token generation
- **Prefix sharing**: Reuse computation for common prefixes

## Speculative Decoding

Speculative decoding accelerates inference using drafts:

```
# Draft model generates k tokens quickly
draft_tokens = draft_model.generate(k=5)

# Main model verifies all k tokens in parallel
verifications = main_model.verify(draft_tokens, batch=True)

# Accept or reject each token
for i, verified in enumerate(verifications):
    if verified:
        accept draft_tokens[i]
    else:
        continue with main model output
```

Key properties:
- Uses smaller/faster "draft" model to generate candidates
- Main model verifies in parallel using batch processing
- No quality loss: final output comes from main model
- Speedup: 2-3x when draft accuracy is high

Common draft models:
- Same architecture, smaller size (e.g., 7B for 70B)
- Distilled version trained on main model outputs
- Cheaper quantized variant

## Continuous Batching

Continuous batching (dynamic batching) improves throughput:

**Static batching (traditional):**
- Wait for all sequences to complete current iteration
- Idle GPU while waiting for slowest sequence
- Inefficient for variable-length sequences

**Continuous batching (streaming):**
- As soon as one sequence completes, remove it and add a new one
- Never wait for slowest sequence
- GPU stays highly utilized

Implementation strategy:
```
Queue of pending requests: [seq_1, seq_2, seq_3, ...]
Batch at iteration t: [seq_1(t), seq_2(t), seq_3(t)]
Batch at iteration t+1: [seq_2(t+1), seq_3(t+1), seq_4(t+1)]
                        (seq_1 completed, seq_4 added)
```

Throughput improvement: 2-4x compared to static batching.

## Memory-Efficient Inference Techniques

### Multi-Query Attention (MQA)
- One key/value head shared across all query heads
- Reduces KV-cache by h times (h = number of heads)
- Trade-off: slight quality loss, but massive memory savings

### Grouped-Query Attention (GQA)
- Intermediate: fewer KV heads than Q heads (e.g., 8 KV, 40 Q)
- Balance between MQA and standard MHA
- Popular in modern models (LLaMA 2, Mistral)

### Quantized KV-Cache
- Store keys/values in int8 or lower precision
- 50% memory reduction (fp16 → int8)
- Minimal quality loss with proper quantization
- Reduces memory to bottleneck other components

## Constraints and Scaling

KV-cache constraints determine model deployment:

1. **Memory ceiling**: Limited GPU memory restricts max batch size
2. **Latency-throughput trade-off**: More batching → higher throughput but worse latency
3. **Sequence length limits**: Longer sequences need more cache
4. **Flash attention limitations**: Even with fast kernels, cache limits throughput

Modern LLM services balance these through:
- Smart batching schedules
- Request queue management
- Priority scheduling (shorter sequences first)
- Early exit strategies

## Conclusion

KV-cache transforms autoregressive decoding from O(n²) to O(n), making generation practical. Innovations like PagedAttention, continuous batching, and speculative decoding push efficiency further. Understanding cache management is essential for deploying LLMs at scale, predicting inference costs, and optimizing token throughput.

The difference between naïve and optimized inference can be orders of magnitude in throughput and latency—making KV-cache one of the highest-impact optimizations in modern LLM deployment.
