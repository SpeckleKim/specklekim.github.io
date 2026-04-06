---
layout: default
title: Attention Mechanism
lang: en
---

# Attention Mechanism: Why the Decoder Can't Look Away

The attention mechanism represents one of the most important innovations in deep learning, transforming NLP from sequence-to-sequence models to the modern transformer era. It elegantly solves a fundamental problem: how can a model focus on relevant parts of input when generating output?

## The Motivation: Information Loss

Basic seq2seq models bottleneck information through a single context vector:

```
Input: "The quick brown fox jumped over the lazy dog"
                    ↓
            [context vector]  ← All information squeezed here
                    ↓
Output: "Le rapide brun renard a sauté par-dessus le chien paresseux"
```

For short sentences, this works adequately. For longer sequences, critical information is lost. The context vector must compress 20+ token meanings into a fixed-size representation.

**The problem becomes acute with:**
- Long documents or articles
- Complex narratives requiring backward reference
- Technical content with many entity relationships
- Dialogue with extensive history

Attention solves this by allowing the decoder to dynamically focus on different parts of the input.

## Attention Concept: Weighted Aggregation

At each decoding step, rather than using only the final encoder hidden state, attention computes a distribution over all input tokens:

```
When decoding "renard" (fox):
Attention distribution over input:
- "The":     0.01
- "quick":   0.02
- "brown":   0.03
- "fox":     0.80  ← Focus here!
- "jumped":  0.05
- others:    0.09

Context = weighted sum of all input hidden states
```

The model learns to attend to the relevant input tokens, effectively asking: "Which part of the input do I need to understand for this output token?"

## Bahdanau Attention (Additive Attention)

One of the first attention mechanisms, proposed by Bahdanau et al. (2014):

**Components:**
1. Query: decoder hidden state h^d_t
2. Key: encoder hidden states h^e_i
3. Alignment function: score(h^d_t, h^e_i) = v^T tanh(W_1 h^d_t + W_2 h^e_i)

**Computation:**
```
Step 1: Compute alignment scores
        score_i = v^T tanh(W_1 h^d_t + W_2 h^e_i)

Step 2: Normalize to attention weights
        α_i = exp(score_i) / Σ_j exp(score_j)  [softmax]

Step 3: Compute context vector
        c_t = Σ_i α_i * h^e_i  (weighted sum of encoder states)

Step 4: Combine with decoder state
        h'_t = tanh(W_c [c_t; h^d_t])
```

**Intuition:** The score function measures similarity between decoder state and each encoder state. Higher scores → higher attention weight → stronger contribution to context.

**Additive vs Multiplicative:**
- Additive (Bahdanau): Uses learned weight matrices and tanh nonlinearity
- Multiplicative: Simpler score function (discussed next)

## Luong Attention (Multiplicative Attention)

Proposed later by Luong et al., simpler alternative:

**Luong attention score:** score(h^d_t, h^e_i) = h^d_t^T W h^e_i

Just a single learned matrix W. Simpler, often faster.

**Comparison:**
| Bahdanau | Luong |
|---|---|
| Additive, nonlinear | Multiplicative, linear |
| More parameters | Fewer parameters |
| Slightly better for long sequences | Slightly better for shorter sequences |
| More compute per attention step | Faster attention computation |

## Scaled Dot-Product Attention

The modern standard (used in transformers):

**Score:** score(Q, K) = (Q K^T) / √d_k

where Q is query, K is key, and d_k is dimension of keys.

**Why the scaling?** Large dot products can saturate softmax (gradients → 0). Division by √d_k normalizes variance.

**Full computation:**
```
Attention(Q, K, V) = softmax((Q K^T) / √d_k) V
```

Simple but effective. Three matrices:
- **Query (Q)**: What am I looking for? (from decoder)
- **Key (K)**: What can you tell me? (from encoder)
- **Value (V)**: What's the information content? (from encoder)

All three typically derived from the same source in practice.

## Multi-Head Attention

Different "heads" attend to different aspects:

```
Input: hidden state h (dimension d_model)

Split into multiple heads:
- Head 1: learns to focus on adjectives ("quick", "brown", "lazy")
- Head 2: learns to focus on nouns ("fox", "dog")
- Head 3: learns to focus on verbs ("jumped")
- etc.

Concatenate outputs: [head_1_output; head_2_output; head_3_output]
Project back to d_model dimension
```

**Advantages:**
1. **Parallel attention**: Multiple heads attend simultaneously (parallelizable)
2. **Rich representation**: Each head captures different linguistic aspects
3. **Robustness**: If one head fails, others provide backup signal
4. **Efficiency**: Often achieves better performance than single larger head

**Implementation:**
```
for each head h:
    Q_h = X W^Q_h
    K_h = X W^K_h
    V_h = X W^V_h
    head_h = Attention(Q_h, K_h, V_h)

MultiHead(Q, K, V) = Concat(head_1, ..., head_h) W^O
```

Typical architecture: 8-12 heads, each with d_model/num_heads dimensions.

## Self-Attention: Token Attends to Token

In self-attention, a token attends to all other tokens in the same sequence:

```
Input sequence: "The cat sat on the mat"

For "cat":
- Attend to "The" (article)
- Attend to "sat" (verb relationship)
- Attend to "mat" (indirect object)
- Lower attention to "on" (preposition)

Q, K, V all come from the same sequence!
```

Self-attention is bidirectional (a token sees all positions) and is the foundation of transformers.

## Cross-Attention: Query from One Source, Key/Value from Another

Cross-attention attends across sequences:

```
Encoder output: [token_1_state, token_2_state, ..., token_n_state]
Decoder state: current_position_state

Query from decoder (what do I want?)
Key and Value from encoder (where can I find it?)

Result: Decoder attends to encoder representation
```

Cross-attention is asymmetric and crucial for tasks like translation, where input and output sequences are different.

## Why Attention Revolutionized NLP

### 1. **Information Access**
Every decoder position can directly access every encoder position, bypassing the bottleneck.

### 2. **Interpretability**
Attention weights directly show which inputs influenced which outputs:

```
English: "The cat is on the mat"
↓
French:  "Le chat est sur le tapis"

Attention weights reveal: "chat" attends most to "cat", etc.
```

This interpretability aids debugging and understanding.

### 3. **Long-Range Dependencies**
Attention can easily connect distant positions:

```
"The bank executive announced the merger yesterday."
When processing "executive", can attend to "bank" (5 positions back)
when processing "merger", can attend to "bank" (7 positions back)
```

RNNs struggled with such long dependencies.

### 4. **Parallelization**
Self-attention allows parallel processing (unlike RNNs which are sequential):

```
RNN: token_1 → token_2 → token_3 (sequential, can't parallelize)
Self-Attention: All tokens processed in parallel with attention
```

Transformers achieve massive speedups through parallelization.

### 5. **Position Flexibility**
Unlike RNNs biased toward recent tokens, attention treats all positions equally. Information content matters more than position.

## Attention in Practice

### Dropout and Regularization
Attention weights can overfit to training data. Attention dropout (randomly zeroing attention weights) helps.

### Residual Connections
Modern attention includes residual connections:
```
output = Attention(input) + input
```
This helps gradient flow and training stability.

### Layer Normalization
Normalizing layer outputs improves training:
```
output = LayerNorm(Attention(LayerNorm(input)) + input)
```

## Attention Variants

**Sparse Attention**: For very long sequences, computing attention to all positions is expensive. Sparse attention only attends to a subset (e.g., local attention, strided attention).

**Relative Position Bias**: Instead of absolute positions, encode relationships between positions, improving generalization to longer sequences.

**Bucketed Attention**: Group positions into buckets, attend within and across buckets.

**Linear Attention**: Approximate softmax attention with kernel methods for O(n) rather than O(n²) complexity.

## Conclusion

Attention mechanisms transformed deep learning for NLP by eliminating the information bottleneck of fixed context vectors. By learning to focus on relevant information, models dramatically improved performance on translation, summarization, and virtually all sequence tasks.

The elegance of attention—computing a learned weighted average—hides profound implications. It shows that models can learn what to focus on, solving the hard problem of relevance. The path from seq2seq models with attention to transformers with multi-head self-attention demonstrates that this core idea—"letting models pay attention"—was so powerful that it became central to modern AI. Understanding attention is essential for understanding modern NLP and large language models.

