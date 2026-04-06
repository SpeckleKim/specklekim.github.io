---
layout: default
title: Positional Encoding
lang: en
---

# Positional Encoding

Self-attention is permutation-invariant: it treats the input sequence the same regardless of token order. While this property enables parallelization, it means the model must be explicitly told about positions. Positional encoding solves this fundamental challenge by incorporating position information into the sequence representation.

## Why Position Matters

Consider two sentences with identical tokens:
- "Dog bites man"
- "Man bites dog"

Without position information, the model cannot distinguish these sentences. Self-attention alone operates on content, not structure. Position encoding ensures the model learns order-dependent relationships.

Key insights:
1. **Position information is essential**: Not just for language understanding
2. **Multiple solutions exist**: Various encoding schemes with different properties
3. **Trade-offs exist**: Between computational efficiency and expressiveness

## Sinusoidal Positional Encoding

The original transformer uses sinusoidal functions:

```
PE(pos, 2i) = sin(pos / 10000^(2i/d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
```

Where:
- pos: position in sequence (0, 1, 2, ...)
- i: dimension index (0, 1, ..., d_model/2 - 1)
- d_model: embedding dimension (512, 768, etc.)

### Intuition Behind Sinusoidal Encoding

1. **Unique position representation**: Each position has a unique encoding vector
2. **Relative position awareness**: Linear relationships between positions
3. **Generalization to longer sequences**: Frequencies extrapolate beyond training length
4. **Interpretable structure**: Different frequencies capture different distance scales

### Example (d_model=4)
```
pos=0: [sin(0),  cos(0),  sin(0),     cos(0)    ] = [0,     1,      0,      1      ]
pos=1: [sin(1),  cos(1),  sin(0.01),  cos(0.01) ] ≈ [0.841, 0.540,  0.01,   1.000  ]
pos=2: [sin(2),  cos(2),  sin(0.02),  cos(0.02) ] ≈ [0.909, -0.416, 0.02,   0.9998 ]
```

Lower dimensions (2i+1) change slowly (long-period waves), higher dimensions change rapidly (short-period waves).

## Learned Positional Embeddings

An alternative approach directly learns position embeddings:

```
pos_embed[i] ∈ ℝ^d_model   (trainable parameters)
```

Advantages:
- Direct optimization for task
- Can capture complex position patterns
- No frequency engineering needed

Disadvantages:
- Fixed vocabulary: only learn positions up to max_seq_len during training
- Cannot extrapolate: fails on longer sequences at test time
- Memory overhead: additional embedding matrix

## Relative Position Bias (ALiBi - Attention with Linear Biases)

ALiBi, introduced by Press et al., adds position-dependent biases directly to attention scores:

```
Attention(Q, K, V) = softmax(Q*K^T / sqrt(d_k) + bias_matrix) * V
```

Where bias_matrix[i,j] = -α * |i - j| for some learned or fixed α.

Key properties:
1. **No learnable embeddings**: Positions encoded in attention bias
2. **Simple linear decay**: Attention decreases linearly with distance
3. **Generalizes to longer sequences**: Bias formula works for any length
4. **Sparse and efficient**: No additional embeddings to compute
5. **Interpretable**: Clear relationship between distance and attention

ALiBi has become increasingly popular in modern LLMs because it:
- Extrapolates naturally to longer sequences
- Uses less memory
- Requires no position vocabulary

## Rotary Position Embeddings (RoPE)

RoPE, proposed by Su et al., uses 2D rotation matrices:

```
[cos(θ)  -sin(θ)]   [q_i]
[sin(θ)   cos(θ)] * [q_j]

where θ = base^(-2i/d_model) * pos
```

Key properties:
1. **Geometric interpretation**: Positions as rotations in complex plane
2. **Relative position encoding**: Attention naturally captures relative distance
3. **Generalizes well**: Interpolation techniques extend to longer sequences
4. **Efficient**: Applies directly to Q and K before attention
5. **Compositional**: Maintains inner product relationships

RoPE is now widely adopted in models like LLaMA, because:
- Excellent length extrapolation properties
- Mathematically elegant
- Efficient GPU implementation
- Strong empirical performance

## Comparing Encoding Methods

| Method | Parameters | Extrapolation | Complexity | Interpretability |
|--------|-----------|--------------|-----------|-----------------|
| Sinusoidal | 0 | Good | O(1) | High |
| Learned | O(max_seq_len) | Poor | O(1) | Low |
| ALiBi | 0 | Excellent | O(n) | Medium |
| RoPE | 0 | Good* | O(n) | High |

*with interpolation or extrapolation techniques

## Position Encoding in Practice

### During Training
- Models typically trained on fixed context length (512, 2048, 4096 tokens)
- Position encodings generated for [0, context_length)
- Gradient updates only affect context window

### During Inference
- Fixed position encoding for pre-trained models
- If sequence exceeds training length:
  - Sinusoidal: extrapolates (frequency pattern continues)
  - Learned: fails or requires fine-tuning
  - ALiBi/RoPE: extrapolates naturally

### Scaling to Longer Contexts
Modern techniques enable longer sequences:
- **Interpolation**: Increase position bucket size
- **Extrapolation**: Confidence that patterns hold
- **Low-rank adaptation**: Fine-tune for longer lengths
- **Coupled training**: Train on varying lengths

## Position Encoding in Cross-Attention

In encoder-decoder models:
- Encoder has own positional encoding
- Decoder has own positional encoding
- Cross-attention uses both (or neither)

Different strategies:
- Both positions: Absolute positions from both
- Decoder only: Only decoder position matters
- Relative: Compute relative positions between encoder/decoder

## Pitfalls and Best Practices

1. **Length mismatch**: Ensure position encoding covers actual sequence length
2. **Extrapolation failures**: Test model behavior on sequences longer than training
3. **Frequency bias**: Sinusoidal may underutilize certain frequency ranges
4. **Interaction with dropout**: Position encodings not typically dropped
5. **Multi-scale understanding**: Combine multiple position encodings for robustness

## Conclusion

Positional encoding transforms permutation-invariant self-attention into an order-aware mechanism. From sinusoidal functions to learned embeddings to modern RoPE, the field continues exploring better position representations. ALiBi and RoPE represent recent progress: eliminating learnable position embeddings while maintaining or improving extrapolation capabilities.

Choosing appropriate positional encoding is critical for model capability, especially when handling sequences longer than training context. Understanding the trade-offs between simplicity, efficiency, and expressiveness guides informed design decisions.
