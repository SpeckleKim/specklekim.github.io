---
layout: default
title: Multi-Head Attention
lang: en
---

# Multi-Head Attention

While single-head self-attention is powerful, multi-head attention enables transformers to jointly attend to information from different representation subspaces. This parallelization of attention mechanisms is one of the key innovations that made transformers practical and effective.

## Why Multiple Attention Heads?

A single attention head operates in a single subspace. Multi-head attention allows the model to:

1. **Attend to different positions**: Some heads focus on immediate neighbors, others on distant tokens
2. **Attend to different features**: Syntactic heads vs. semantic heads vs. positional heads
3. **Capture diverse patterns**: No single projection captures all relevant relationships
4. **Provide redundancy**: Multiple views enable robustness against noisy attention patterns
5. **Increase representation capacity**: More parameters per layer without increasing layer depth

## The Multi-Head Mechanism

With h heads and d_model dimensions, each head operates on d_k = d_model / h dimensions:

```
head_i = Attention(Q_i, K_i, V_i)
        = softmax(Q_i * K_i^T / sqrt(d_k)) * V_i

MultiHead(Q, K, V) = Concat(head_1, ..., head_h) * W_O
```

Where:
- Q_i = Q * W_Q^i (i-th head's query projection)
- K_i = K * W_K^i (i-th head's key projection)
- V_i = V * W_V^i (i-th head's value projection)
- W_O is the output projection (d_model, d_model)

## Parallel Computation

A major advantage of multi-head attention is **parallelizability**:

- Unlike RNNs, all h heads compute simultaneously
- Matrix operations can be efficiently batched
- GPUs excel at parallel matrix multiplications
- Modern implementations use tensor operations: (batch, seq_len, h, d_k)

This parallel architecture enabled training on large-scale datasets, transforming NLP.

## Concatenation and Projection

After computing independent attention heads, outputs are concatenated:

```
concatenated = Concat(head_1, ..., head_h)  # shape: (seq_len, d_model)
output = concatenated * W_O
```

The output projection W_O:
- Maps back to d_model dimensions
- Learns relationships between different heads
- Enables downstream layers to combine multi-head information
- Acts as a learned "mixer" across head representations

## What Do Different Heads Learn?

Empirical studies show that attention heads learn specialized roles:

- **Positional heads**: Track relative token positions and distances
- **Syntactic heads**: Focus on grammatical structures (subject-verb, modifier-noun)
- **Semantic heads**: Capture long-range content relationships
- **Output heads**: Attend to previous positions (preparing for generation)
- **Rare token heads**: Specialize in infrequent but important tokens
- **Stopword heads**: Handle function words and articles

This specialization emerges without explicit supervision—it's a natural consequence of learning to minimize loss.

## Ablation Studies and Head Importance

Research has shown:

1. **Head pruning**: Many heads can be removed post-training with minimal impact
2. **Important vs. redundant**: Some heads are critical, others seem redundant
3. **Positional vs. semantic**: Different head types matter at different layers
4. **Head contribution varies**: Early layers develop more syntactic patterns, later layers semantic
5. **Redundancy as robustness**: Redundant heads provide insurance against distribution shift

The "Lottery Ticket Hypothesis" in transformers suggests many heads are redundant lottery tickets—they help during training but aren't strictly necessary at test time.

## Number of Heads: A Design Choice

The number of heads is a hyperparameter constrained by:

- **d_model divisibility**: d_model must be divisible by h
- **Memory and compute**: More heads = more computation per forward pass
- **d_k size**: Too small d_k makes each head capacity limited
- **Empirical performance**: 8-16 heads typical for base models, 12-16 for large models

Trade-offs:
- More heads → better specialization but diminishing returns
- Fewer heads → faster but less diverse pattern capture
- Very few heads (h=1) → collapses to single-head attention
- Very many heads → dilutes each head's capacity

## Cross-Head Information Flow

Multi-head attention enables "cross-head" effects:

1. **Implicit debate**: Different heads propose different interpretations
2. **Consensus**: Output projection learns to weight heads by reliability
3. **Error correction**: If one head misfires, others can compensate
4. **Ensemble effect**: Similar to boosting where multiple weak learners combine

This ensemble-like property contributes to transformers' robustness and generalization.

## Attention Patterns Visualization

Visualizing attention weights reveals head specialization:

```
Head 1: Attends to adjacent tokens (local attention)
Head 2: Attends to nouns before verbs (syntactic)
Head 3: Attends to article-noun pairs (grammatical role)
Head 4: Long-range semantic attention
...
```

These patterns persist across datasets, suggesting learned universal attention strategies.

## Computational Complexity

Multi-head attention complexity per head:
- Query/Key/Value projections: O(h * d_model * d_k)
- Attention computation: O(h * n² * d_k)
- Output projection: O(h * d_v * d_model)

While adding h heads multiplies computation by h, the parallel nature allows efficient GPU implementation, making multi-head essentially "free" compared to adding depth.

## Comparison: Single vs. Multi-Head

| Aspect | Single Head | Multi-Head |
|--------|-------------|-----------|
| Representation subspaces | 1 | h |
| Pattern diversity | Limited | Rich |
| Specialization | None | Strong |
| Robustness | Vulnerable | Redundant |
| Interpretation | Simple | Complex but insightful |

## Modern Variants

Recent work explores alternatives:

- **Grouped query attention (GQA)**: Fewer key/value heads than query heads (reduces memory)
- **Multi-query attention (MQA)**: Single key/value head shared across all query heads
- **Sparse multi-head**: Only attend to relevant heads based on content
- **Mixture of Experts**: Head selection based on input

These variants reduce memory while maintaining performance.

## Conclusion

Multi-head attention transforms a single projection space into h complementary subspaces, enabling rich pattern capture through parallel computation. The emergence of specialized heads without supervision demonstrates how gradient-based learning naturally discovers diverse attention strategies. This design choice was pivotal in making transformers practical, enabling efficient parallelization and strong empirical performance across diverse tasks.

Understanding multi-head attention's role in specialization, redundancy, and parallelization is essential for model design, optimization, and interpretability work.
