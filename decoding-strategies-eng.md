---
layout: default
title: Decoding Strategies
lang: en
---

# Decoding Strategies

## The Decoding Problem

Once a language model produces a probability distribution over the vocabulary for the next token, we face a choice: **how do we select which token to generate?** This seemingly simple decision dramatically impacts output quality, diversity, and computational cost. The wrong strategy produces repetitive, incoherent, or overly generic text.

Decoding is the algorithm that converts model probabilities into actual text. Even with a perfect language model, poor decoding ruins quality. Conversely, clever decoding can work around model limitations.

## Greedy Decoding

The simplest approach: **always pick the token with the highest probability**.

```
next_token = argmax(p(vocab))
```

### Advantages
- Fastest computation (single argmax per step)
- Deterministic and reproducible
- Works well for factual tasks

### Disadvantages
- No diversity: same input always produces identical output
- Gets stuck in loops easily
- Produces boring, repetitive text
- Doesn't explore less likely but valid continuations

Greedy decoding is a baseline, rarely the best choice for generative tasks.

## Beam Search

**Beam search** maintains multiple hypotheses (sequences) simultaneously, expanding the k most promising ones at each step.

### How Beam Search Works

1. Start with k (beam width) hypotheses, each with score = log p(prefix)
2. For each hypothesis, compute probabilities for all next tokens
3. Select top k*|vocab| candidates globally
4. Keep top k candidates, reset for next step
5. Continue until all beams reach end-of-sequence token

### Key Concept: Length Normalization

Raw log probabilities bias toward shorter sequences (fewer multiplications). **Length normalization** divides scores by sequence length to prefer longer, more detailed outputs:

```
score = (1/length^α) * log(p(sequence))
```

where α typically ranges 0.5-1.0.

### Advantages
- Explores multiple hypotheses in parallel
- Finds locally optimal sequences
- Better than greedy for translation/summarization

### Disadvantages
- Computationally expensive (k times slower than greedy)
- Still favors probable paths, misses creative alternatives
- Length normalization requires tuning
- High beam widths (k=5+) have diminishing returns

Beam search works well for translation and structured generation but can feel generic for open-ended tasks.

## Sampling-Based Methods

Sampling introduces **stochasticity**—we randomly select from the distribution rather than always picking the highest probability.

### Pure Sampling

Sample the next token directly from the model's probability distribution:

```
next_token ~ p(vocab)
```

This produces diversity but can include very low-probability, nonsensical tokens. Most models have long tails of terrible completions.

## Temperature Scaling

**Temperature** controls the sharpness of the probability distribution. It's a simple but powerful parameter.

The distribution becomes:
```
p_t(token) = exp(logits/T) / Σ exp(logits/T)
```

- **T < 1 (cold)**: Distribution sharpens, makes high-probability tokens even more likely
- **T = 1 (normal)**: Original probabilities
- **T > 1 (hot)**: Distribution flattens, gives low-probability tokens more chance

### Practical Use
- Lower temperature (0.3-0.7) for factual, consistent outputs
- Higher temperature (1.0-2.0) for creative, diverse outputs
- Temperature is model-dependent; tune empirically

## Top-k Sampling

Only sample from the **top k highest probability tokens**, then renormalize.

```
p_filtered(token) = {
  p(token) / Z,  if token in top-k
  0,             otherwise
}
```

where Z is the normalization constant.

### Advantages
- Eliminates unreasonable tokens (long tail)
- Preserves diversity in top candidates
- Simple to implement
- Works well empirically

### Disadvantages
- k is a hard cutoff—reasonable tokens just outside top-k are dropped
- Requires tuning k per use case
- Different tokens have different tail distributions

## Top-p (Nucleus) Sampling

A smarter alternative to top-k: sample from a **dynamic subset** that contains p fraction of the probability mass.

### How It Works

1. Sort tokens by descending probability
2. Select just enough tokens so their cumulative probability ≥ p
3. Renormalize and sample from this subset

Example: If p=0.9, keep sampling tokens until cumulative probability reaches 90%.

### Advantages
- Adaptive: naturally handles varying distributions
- Works well for diverse, fluent text
- Good default choice for most applications
- Less hyperparameter tuning than top-k

### Disadvantages
- Slightly more computation than top-k
- Still requires p tuning (typically 0.85-0.95)

Top-p has become the standard for many applications and is often the default in modern LLM APIs.

## Repetition Penalty

Models tend to repeat tokens or phrases. **Repetition penalty** discounts the probability of tokens that already appeared.

For tokens that appeared n times in the sequence:
```
p_penalized(token) = p(token) / (1 + n * penalty)
```

Common penalty values: 1.0-2.0

### When to Use
- Creative writing (avoid dull repetition)
- Long-form generation
- Chat and dialogue

### Caution
- Too high penalties can make repetition impossible (even when grammatical)
- Can disrupt patterns like numbered lists (1, 2, 3...)

## Min-p Sampling

A recent technique that **filters based on relative probability**, not absolute count.

For minimum probability threshold p_min:
```
keep tokens where: p(token) ≥ p_min * max(p(vocab))
```

So if the top token has probability 0.8, min-p=0.1 keeps tokens down to probability 0.08.

### Advantages
- Relative filtering is more principled than top-k's hard cutoff
- Good balance between quality and diversity
- Reduces "dead hand" problem where no valid continuations exist

## Typical Sampling

Based on information-theoretic principles. Sample tokens within a certain **entropy range** of the distribution.

The idea: keep tokens whose information content is near the distribution's average. This avoids both extreme (too certain) and tail (too unlikely) regions.

### Practical Effect
- Produces less repetitive, more fluent text than top-k
- Better handles varying entropy across contexts
- Good for zero-shot performance without tuning

## Constrained Decoding

For many applications, we want to **restrict outputs to specific patterns**:
- JSON format
- Function calls
- SQL queries
- Regular expressions

### Approaches

**Grammar-based**: Enforce outputs against a formal grammar (EBNF). Only allow continuations that can lead to valid completions.

**Token masking**: Maintain valid next tokens and mask invalid ones before sampling. Typically faster than grammar-based.

**Guided decoding**: Adjust probabilities to steer toward desired outputs while maintaining diversity.

### Trade-offs
- Guarantees valid format but may reduce quality
- Can make some outputs impossible (if grammar doesn't cover them)
- Adds modest computational overhead

## Combining Strategies

In practice, most systems combine multiple strategies:

```
1. Temperature = 0.8 (moderate heat)
2. Top-p = 0.95 (nucleus sampling)
3. Repetition penalty = 1.1 (avoid repetition)
4. Min length = 10 (avoid short answers)
5. Max length = 500 (avoid infinite generation)
```

This balanced approach provides diversity while maintaining coherence and avoiding obvious failure modes.

## Choosing the Right Strategy

**For factual/consistent outputs**:
- Low temperature (0.3-0.5)
- Beam search or greedy
- No sampling

**For creative/diverse outputs**:
- Higher temperature (1.0-1.5)
- Top-p sampling
- Repetition penalty

**For structured outputs**:
- Constrained decoding
- Potentially lower temperature for consistency

**For balanced use**:
- Temperature 0.7-0.9
- Top-p 0.9-0.95
- Repetition penalty 1.0-1.2

The best strategy depends on your task, model, and user expectations. Experimentation and measuring quality (via metrics or human evaluation) remains essential.
