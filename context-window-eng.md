---
layout: default
title: Context Window & Long Context
lang: en
---

# Context Window & Long Context

## What is Context Window?

The context window, or context length, defines the maximum number of tokens an LLM can process at once. This is one of the most fundamental constraints in modern language models. Early transformers like BERT had context windows of 512 tokens, while GPT-2 introduced 1,024 tokens. Today's models range from 4K to over 1M tokens, but this limitation remains a critical design consideration.

### Why Context Matters

A larger context window enables:
- **Document-level understanding**: Process full research papers or books without truncation
- **Long conversations**: Maintain coherent dialogue over many turns
- **In-context learning**: Provide numerous examples for few-shot prompting
- **Better summarization**: Capture nuance across longer sequences
- **Complex reasoning**: Hold more intermediate steps in working memory

However, context size has computational costs. Attention mechanisms scale quadratically (O(n²)) with sequence length, making large context windows expensive during both training and inference.

## The Challenge: Why Not Just Increase Context?

### Computational Complexity

Standard transformer attention requires computing similarity scores between all pairs of tokens. For a 1M token context, this means roughly 10^12 operations per attention head—computationally prohibitive on consumer hardware. This O(n²) scaling is the fundamental bottleneck.

### Training Constraints

Most models train with shorter contexts (2K-4K tokens) due to memory and time limitations. Evaluating on longer contexts at inference time without training exposure creates a distributional mismatch that hurts performance.

### Position Interpolation Problem

Transformers learn absolute positional embeddings during training. When you try to extend context beyond training length, models encounter positions they've never seen. Naively extending causes catastrophic performance drops.

## Scaling Techniques

### RoPE (Rotary Position Embedding)

RoPE encodes positions as rotation angles in complex space, allowing smoother extrapolation beyond training length. By adjusting the frequency base (theta), models can generalize to longer sequences. This is why many modern models (LLaMA, Qwen) use RoPE—it enables better length generalization with simpler modifications.

### Attention Variants

**Linear Attention**: Approximates standard attention with linear complexity (O(n)) by using kernelized functions or recurrent formulations. Models like Mamba and Liquid Transformers explore this direction.

**Sparse Attention**: Only compute attention between nearby tokens and global tokens (Longformer, BigBird). A token at position i attends to positions i-w to i+w (local window) plus select global tokens.

**Ring Attention**: Distributes long sequences across multiple GPUs, computing attention in rings to reduce memory per device. Enables 4M+ token training on clusters.

### Landmark Attention

This technique selects a small set of "landmark" tokens (summary tokens) and only computes full attention with these landmarks, while other attention is approximated. This reduces O(n²) to O(nk) where k is small.

## Long Context Models

### 128K-1M Token Models

Recent breakthroughs enable:
- **Claude 3 (200K context)**: Uses advanced attention techniques and careful training procedures
- **GPT-4 Turbo (128K context)**: Manages 25x ChatGPT's original context
- **LLaMA 2 variants (100K+)**: Through position interpolation and continued pretraining
- **Gemini Ultra (1M tokens)**: Pushes boundaries on context length

These models implement combinations of techniques: optimized attention patterns, position embedding scaling, and often continued pretraining on longer sequences.

## The "Lost in the Middle" Problem

Empirical findings show that even with large context windows, models struggle with information in the middle of their input. When processing 20+ documents, models may ignore relevant information placed in the middle, focusing disproportionately on beginning and end.

This happens because:
- **Attention patterns**: Models learn to attend primarily to recent and early tokens during pretraining
- **Optimization dynamics**: Critical information is more salient at boundaries
- **Effective context**: The "effective" usable context is much smaller than nominal length

Mitigation strategies:
- Place most important information first or last
- Use retrieval augmentation (RAG) to select only relevant documents
- Implement hierarchical processing with summaries
- Tune attention through instruction following

## Practical Implications

### For Developers

When using long-context models:
- Longer context doesn't equal better performance; relevance matters more
- Position of information within context affects quality
- Temperature and decoding parameters may need adjustment
- Monitor cost—longer contexts multiply token expenses

### Trade-offs

- **Inference speed**: Long contexts slow down token generation linearly or worse
- **Cost**: Pricing proportional to context length makes long contexts expensive
- **Hardware requirements**: Even inference needs substantial GPU memory for multi-100K contexts

## Future Directions

Research continues on:
- **Efficient attention**: Reducing O(n²) scaling through algorithmic innovation
- **Hierarchical processing**: Multi-level summarization for massive documents
- **Adaptive context**: Models that determine optimal context length per task
- **Hybrid approaches**: Combining dense and sparse attention intelligently

The path forward likely involves combinations of these techniques rather than simply scaling context length indefinitely. The goal is usable, effective context—not just token count.
