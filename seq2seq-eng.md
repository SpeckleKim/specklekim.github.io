---
layout: default
title: Sequence-to-Sequence Models
lang: en
---

# Sequence-to-Sequence Models: The Foundation of Neural Translation

Sequence-to-sequence (seq2seq) models form the foundation of many contemporary NLP applications. From [machine translation](/machine-translation-eng.html) to [summarization](/text-summarization-eng.html) to dialogue systems, seq2seq architectures have become ubiquitous. These models take variable-length input sequences and generate variable-length output sequences, making them flexible and powerful.

## The Seq2Seq Problem

Traditional machine learning assumes fixed-size inputs and outputs. Language problems don't conform to this:

```
English: "Hello, how are you?" (5 tokens)
French:  "Bonjour, comment allez-vous ?" (6 tokens)

Japanese "こんにちは、お元気ですか?" (7 tokens after tokenization)
```

**Key challenges:**
- Variable length inputs and outputs
- No one-to-one token alignment
- Complex reordering between languages
- Capturing long-range dependencies

Seq2seq models handle these by learning implicit alignments rather than explicit linguistic rules.

## Encoder-Decoder Architecture

The seq2seq framework separates processing into two components:

### Encoder: Processing the Input

The encoder processes the input sequence and produces a compact representation (context vector):

```
Input: "Hello, how are you?"
↓ (LSTM/GRU Processing)
Context Vector: [2.3, -1.1, 0.8, ..., 1.2]
```

**Encoder responsibilities:**
1. Process tokens sequentially or in parallel
2. Maintain hidden state capturing context
3. Produce single vector representing full input
4. Bridge gap between input and output domains

The encoder must compress entire sequence semantics into a single vector. For long sequences, information loss is inevitable—leading to a motivation for attention (discussed later).

### Decoder: Generating the Output

The decoder receives the context vector and generates output tokens sequentially:

```
Context Vector: [2.3, -1.1, 0.8, ..., 1.2]
↓ (Decoder initialization)
Hidden State: context vector
↓ (Token-by-token generation)
Output: "Bonjour, comment allez-vous ?"
```

**Decoder responsibilities:**
1. Initialize from encoder context
2. Generate tokens sequentially
3. Maintain hidden state across generation
4. Stop generation appropriately ([EOS] token)

The decoder is essentially a language model conditioned on the input representation.

## RNN-Based Seq2Seq

Most seq2seq models use recurrent architectures (LSTM, GRU):

**Encoder:**
```
Input: x₁, x₂, ..., xₙ
LSTM: h₁ → h₂ → ... → hₙ
Output: hₙ (context vector)
```

**Decoder:**
```
Initialize: h₀' = hₙ (encoder context)
Generate: y₁ ← p(y₁|h₀', context)
          h₁' = LSTM(y₁, h₀')
          y₂ ← p(y₂|h₁', context)
          ... repeat until [EOS]
```

**Key decision: Bidirectional encoders**

Bidirectional RNNs process input left-to-right and right-to-left, concatenating representations:

```
Forward:  → The cat sat on the mat →
Backward: ← mat the on sat cat The ←
Concat:   [forward; backward] for each token
```

Bidirectional encoders typically outperform unidirectional, as context from both directions informs representation.

## Teacher Forcing

During training, how does the decoder learn to generate?

```
Naive approach:
Input: x₁, x₂, ... (e.g., English sentence)
Target: y₁, y₂, y₃ (e.g., French sentence)
Decoder step 1: predict y₁
                if wrong, predict y₂ based on wrong y₁
                prediction compounds errors
```

**Teacher forcing solution:**

Use ground truth targets during training:

```
At training time:
Step 1: predict y₁ given context
Step 2: predict y₂ given context + ground truth y₁
Step 3: predict y₃ given context + ground truth y₁, y₂
```

**Mismatch problem:**
- Training: decoder sees ground truth tokens
- Inference: decoder sees its own predictions (possibly wrong)
- Distribution shift: training and test conditions differ

This exposure bias causes models trained with teacher forcing to make more errors at test time. Solutions include:
- **Scheduled sampling**: Gradually transition from ground truth to predictions during training
- **Beam search**: Use multiple hypotheses rather than greedy decoding
- **Data augmentation**: Train with corrupted inputs occasionally

## Attention Mechanism (Brief Introduction)

The core problem with basic seq2seq: compressing an entire sequence into one vector causes information loss.

```
English: "The quick brown fox jumped over the lazy dog"
French:  "Le rapide brun renard a sauté par-dessus le chien paresseux"

Fixed context vector loses alignment information.
Which output token aligns with which input token?
```

Attention computes a weighted distribution over input tokens:

```
When generating French "renard" (fox):
Attention focuses on English "fox"
When generating French "rapide" (quick):
Attention focuses on English "quick"
```

Rather than using only final encoder state, attention allows decoder to "look back" at input tokens weighted by relevance. This will be fully explored in the [attention mechanism](/attention-mechanism-eng.html) article.

## Beam Search Decoding

At test time, greedy decoding (always selecting highest [probability](/probability-eng.html) token) is suboptimal:

```
Greedy might select: "The cat is..."
Better solution:     "A cat is..."
But greedy locked in "The" at step 1
```

[Beam search](/decoding-strategies-eng.html) maintains multiple hypotheses:

```
Step 1: Keep top-k=3 hypotheses
        "The" (prob=0.4), "A" (prob=0.3), "One" (prob=0.2)
Step 2: Expand each, keep top-k overall
        "The cat" (0.4*0.5), "A cat" (0.3*0.6), "The dog" (0.4*0.3)
Step 3: Continue until [EOS]
```

Beam size tradeoff:
- k=1: greedy, fastest, lower quality
- k=3-5: good balance of quality and speed
- k>5: marginally better quality, slower

## Applications of Seq2Seq

### Machine Translation

The original motivation for seq2seq. Neural [machine translation](/machine-translation-eng.html) (NMT) dramatically outperformed phrase-based statistical approaches.

```
Input (English):  "The meeting is scheduled for tomorrow."
Output (Spanish): "La reunión está programada para mañana."
```

### Abstractive Summarization

[Summarization](/text-summarization-eng.html) as sequence-to-sequence task:

```
Input: Full article (hundreds of tokens)
Output: Summary (50-100 tokens)
```

Abstractive (generating new text) vs extractive (selecting sentences). Seq2seq enables abstractive approaches.

### Question Answering

Reading comprehension as seq2seq:

```
Input: "John went to the store. He bought milk. He left."
Question: "What did John buy?"
Output: "Milk."
```

Or more complex seq2seq QA with context and question as input.

### Dialogue and Chatbots

Conversational AI as seq2seq:

```
Input: "Hi, how are you?"
Output: "I'm doing well, thanks for asking!"
```

Encoder processes dialogue history or context, decoder generates response.

## Seq2Seq Extensions

Modern variants improve basic seq2seq:

1. **Bidirectional decoding**: Generate backwards to capture future context
2. **Coverage mechanism**: Track which input tokens have been translated
3. **Multi-task learning**: Combine translation with related tasks
4. **Back-translation**: Generate synthetic training data
5. **Layer-wise attention**: Different decoder layers attend to different input levels

## Limitations of Basic Seq2Seq

1. **Information bottleneck**: Single vector represents entire input
2. **Long sequence performance**: RNNs struggle with very long sequences
3. **Rigid alignment**: Cannot handle many-to-many or complex reorderings
4. **Training inefficiency**: Cannot parallelize at input level (sequential RNN processing)
5. **Context fragility**: Small changes in input can dramatically alter output

These limitations motivated [attention mechanisms](/attention-mechanism-eng.html) and eventually led to [transformer](/llm-eng.html) architectures.

## Historical Impact

Seq2seq models (with attention) dominated NLP from 2014-2018. They achieved:
- Breakthrough performance on [machine translation](/machine-translation-eng.html)
- State-of-the-art results on [summarization](/text-summarization-eng.html)
- Strong baselines for [question answering](/question-answering-eng.html)
- Foundation for modern dialogue systems

The architecture proved so flexible that it inspired [transformer](/llm-eng.html)-based variants and remains foundational to understanding modern NLP architectures.

## Conclusion

Sequence-to-sequence models solved a fundamental problem: how to process variable-length inputs to variable-length outputs using [neural networks](/neural-eng.html). The encoder-decoder framework, while challenged by [transformers](/llm-eng.html) for some tasks, remains conceptually central to understanding generative NLP. Teacher forcing, [beam search](/decoding-strategies-eng.html), and [attention mechanisms](/attention-mechanism-eng.html) are crucial techniques emerging from seq2seq research. Contemporary models build on these foundations, even if specific RNN-based seq2seq architectures are less common in 2024. Understanding seq2seq is essential for grasping modern NLP and generative AI.

