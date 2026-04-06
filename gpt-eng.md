---
layout: default
title: GPT & Autoregressive Models
lang: en
---

# GPT & Autoregressive Models

## What is Autoregressive Generation?

Autoregressive models generate text one token at a time, where each new token is predicted based on all previously generated tokens. This sequential approach mimics how humans naturally generate language—predicting the next word based on context. The [probability](/probability-eng.html) distribution is factorized as P(x) = ∏ P(x_t | x_{<t}), meaning the model learns to predict the next token given the entire history.

This approach contrasts with other generative paradigms like masked language modeling ([BERT](/bert-eng.html)) or non-autoregressive generation (which predicts all tokens simultaneously). The autoregressive framework provides a natural training objective through language modeling: predict the next token given the previous sequence.

## The GPT Family Evolution

### GPT-1 (2018)
OpenAI's first generative pre-trained [transformer](/llm-eng.html) demonstrated that unsupervised pretraining on large text corpora followed by task-specific fine-tuning could achieve strong performance across various NLP tasks. With 117 million parameters, GPT-1 showed that language models could be effectively applied to classification, entailment, similarity, and question-answering tasks with minimal architectural changes.

### GPT-2 (2019)
GPT-2 scaled up to 1.5 billion parameters and introduced a critical insight: [large language models](/llm-eng.html) could perform impressive "zero-shot" [transfer learning](/transfer-learning-eng.html) without explicit task-specific fine-tuning. The model demonstrated genuine [emergent abilities](/emergent-abilities-eng.html) like translation, question-answering, and [summarization](/text-summarization-eng.html) when prompted appropriately. GPT-2 showcased the potential of scale as a fundamental principle in language modeling.

### GPT-3 (2020)
With 175 billion parameters, GPT-3 represented a leap in model scale and capability. It demonstrated strong few-shot learning—the ability to learn new tasks from just a handful of examples provided in the prompt. GPT-3 introduced in-context learning as a primary interaction paradigm, reducing the need for task-specific fine-tuning.

### GPT-4 (2023)
Building on the scaling paradigm, GPT-4 improved reasoning capabilities, reduced [hallucinations](/hallucination-eng.html), and better handled edge cases. It demonstrated [multimodal](/multimodal-llm-eng.html) understanding (text and images) and showed improved performance on complex reasoning tasks including mathematics and coding.

## Decoder-Only Architecture

GPT models use a decoder-only [Transformer](/llm-eng.html) architecture, consisting of:

- **Token Embeddings**: Each token is mapped to a high-dimensional vector
- **Positional Encodings**: Information about token positions is added to embeddings
- **Transformer Blocks**: Stacked layers of multi-head [self-attention](/self-attention-eng.html) and feed-forward networks
- **Causal Masking**: Attention is restricted to previous tokens only, preventing the model from "looking ahead"
- **Layer Normalization & Residual Connections**: Stabilize training of deep networks

The decoder-only design differs from encoder-decoder models ([seq2seq](/seq2seq-eng.html)) which process input sequences fully before generating output. In decoder-only models, generation is purely sequential and autoregressive.

## In-Context Learning

In-context learning refers to the ability of language models to learn new tasks from examples provided in the prompt without explicit training updates. A prompt might include:

```
Sentiment classification examples:
"Great movie!" → Positive
"Terrible experience" → Negative
"Amazing service" → Positive

"I hated it" →
```

The model learns the task definition from the context and generates the appropriate response. This mechanism appears to rely on the model's internal representations learned during pretraining, which encode many implicit task-solving strategies.

## Few-Shot and Zero-Shot Prompting

**Few-shot prompting** provides multiple examples of a task within the prompt, enabling the model to infer the pattern and apply it to new instances. This is powerful for rapid task adaptation without retraining.

**Zero-shot prompting** provides no examples—only a task description. The model must rely entirely on knowledge acquired during pretraining. While zero-shot performance is generally lower, it demonstrates the breadth of knowledge encoded in language models.

**One-shot prompting** occupies the middle ground, providing a single example to establish the task format.

## Emergent Abilities

[Large language models](/llm-eng.html) exhibit "[emergent abilities](/emergent-abilities-eng.html)"—capabilities that appear suddenly as scale increases beyond certain thresholds. Examples include:

- **Arithmetic Reasoning**: Small models fail at even simple math; large models solve multi-step problems
- **Code Generation**: Models suddenly produce working code at sufficient scale
- **Chain-of-Thought Reasoning**: Models develop internal reasoning steps when asked to "think step by step"
- **Translation**: Without explicit translation training data, models translate between languages

These abilities are often surprising and unpredictable, suggesting that scale alone can induce qualitatively new capabilities.

## The Impact of Scaling

Research has identified consistent [scaling laws](/scaling-laws-eng.html) relating model size, training data size, and compute to downstream performance. The [scaling law](/scaling-laws-eng.html) roughly follows: Loss ∝ N^(-α), where N is model size and α ≈ 0.07. This suggests that larger models with more data consistently achieve lower loss.

Key insights from scaling:
- **Larger is better**: Scaling model size, data, and compute improves performance
- **Trade-offs exist**: Compute, parameters, and data can be traded off (some flexibility in optimization)
- **Power-law behavior**: Improvements continue at scale; no clear plateau (unlike traditional machine learning)
- **Emerging capabilities**: Scaling unlocks new abilities that smaller models cannot perform

This understanding has driven the race toward ever-larger models, though with diminishing returns and increasing resource requirements.

## Conclusion

GPT models represent a paradigm shift in natural language processing, moving from task-specific models trained from scratch toward general-purpose foundation models capable of handling diverse tasks through in-context learning. The decoder-only architecture combined with massive scale has proven remarkably effective, with improvements continuing to emerge as models grow larger. Understanding autoregressive generation, [scaling laws](/scaling-laws-eng.html), and [emergent abilities](/emergent-abilities-eng.html) is essential for working with modern [large language models](/llm-eng.html).
