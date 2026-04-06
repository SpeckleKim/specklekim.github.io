---
layout: default
title: Tokenization
lang: en
---

# Tokenization: Breaking Text into Digestible Pieces

Tokenization is the foundational process of converting raw text into discrete units (tokens) that machine learning models can process. It sits at the intersection of natural language processing and deep learning, determining how models perceive and understand language.

## The Problem with Raw Text

[Neural networks](/neural-eng.html) cannot directly process raw text strings. They require numerical representations. Before we can embed or transform text, we must first decide what constitutes a meaningful unit: Is it a word? A character? A subword? This decision has cascading effects on model performance, [vocabulary](/tokenizer-design-eng.html) size, and multilingual capability.

## Word-Level Tokenization

The simplest approach splits text at whitespace and punctuation:

```
"Hello, world!" → ["Hello", ",", "world", "!"]
```

**Advantages:**
- Intuitive and human-readable
- Preserves semantic meaning of words
- Works well for well-formatted English text

**Disadvantages:**
- Large [vocabulary](/tokenizer-design-eng.html) size (100,000+ tokens for English)
- Cannot handle out-of-[vocabulary](/tokenizer-design-eng.html) (OOV) words
- Treats morphological variants as separate tokens ("running," "run," "runs")
- Poor for languages without clear word boundaries (Chinese, Japanese)

## Character-Level Tokenization

This approach treats every character as a token:

```
"Hello" → ["H", "e", "l", "l", "o"]
```

**Advantages:**
- Minimal [vocabulary](/tokenizer-design-eng.html) (typically 100-200 tokens)
- Can represent any text without OOV issues
- Language-agnostic

**Disadvantages:**
- Sequences become very long, straining model capacity
- Loses semantic structure of words
- Inefficient memory usage
- Difficult to learn meaningful representations at character level

## Subword Tokenization: The Modern Approach

Subword tokenization finds a middle ground between word and character levels, splitting words into meaningful subword units. Three dominant algorithms emerged:

### Byte Pair Encoding (BPE)

BPE iteratively merges the most frequent adjacent character/token pairs:

```
Initial: "hello" → ["h", "e", "l", "l", "o"]
Merge most frequent pair ("l", "l"): ["h", "e", "ll", "o"]
Merge next frequent pair ("e", "ll"): ["h", "ell", "o"]
Final: ["h", "ell", "o"]
```

Used in [GPT](/gpt-eng.html) models. Creates a compact [vocabulary](/tokenizer-design-eng.html) (typically 50K tokens) while maintaining reasonable sequence length.

### WordPiece

Google's WordPiece algorithm greedily constructs the longest token that exists in a pre-built [vocabulary](/tokenizer-design-eng.html):

```
"playing" → ["play", "##ing"]
"unseeded" → ["un", "##seed", "##ed"]
```

The "##" prefix indicates subword continuations. Used in [BERT](/bert-eng.html), which is why BERT vocabularies contain many "##" tokens. [Vocabulary](/tokenizer-design-eng.html) size typically 30K tokens.

### Unigram Language Model

This probabilistic approach treats tokenization as a maximum likelihood problem:

```
P("hello" → ["h", "e", "l", "l", "o"]) vs P("hello" → ["he", "llo"])
```

Selects the tokenization that maximizes overall [probability](/probability-eng.html) under a unigram language model.

### SentencePiece

A language-independent tokenization method that works directly on bytes, allowing seamless handling of Unicode, spaces, and special characters:

```
"hello world" → ["▁hello", "▁world"]
"こんにちは" (Japanese) → ["▁", "こ", "ん", "に", "ちは"]
```

Used in T5, mBART, and XLNet. Particularly effective for multilingual models.

## Vocabulary Size Tradeoffs

Choosing [vocabulary](/tokenizer-design-eng.html) size requires balancing several factors:

| [Vocabulary](/tokenizer-design-eng.html) Size | Impact |
|---|---|
| **Small (10K)** | Shorter sequences, smaller embeddings, but frequent OOV and information loss |
| **Medium (30-50K)** | Sweet spot for most models; reasonable sequence length and memory usage |
| **Large (100K+)** | Longer sequences, larger embeddings, better preservation of rare words but memory overhead |

## Multilingual Tokenization Challenges

Different languages have vastly different optimal tokenization strategies:

**Morphologically Rich Languages** (Turkish, Finnish, Hungarian): Require subword tokenization to handle complex affixation

**No Explicit Word Boundaries** (Chinese, Japanese): Character-based or SentencePiece approaches often superior

**Combining Scripts** (Transliteration, code-mixing): Standard tokenizers fail; SentencePiece's byte-level approach handles this better

Multilingual models typically use unified vocabularies (e.g., mBERT's 119K [vocabulary](/tokenizer-design-eng.html) covers 104 languages).

## Tokenization and Model Performance

Tokenization choice influences downstream performance:

1. **Information Loss**: Aggressive subword tokenization may lose morphological signals
2. **Sequence Length**: Longer tokenized sequences consume more memory and compute
3. **Embedding Quality**: Word-level vocabularies allow high-quality embeddings; subword vocabularies split semantic meaning
4. **Fine-tuning**: Some tokenization choices benefit specific downstream tasks
5. **Rare Words**: Subword tokenization better handles domain-specific terminology (medical, technical)

Research shows that optimal [vocabulary](/tokenizer-design-eng.html) size varies by language and task: English benefits from 30-50K, Japanese from 50-100K, and morphologically rich languages from smaller vocabularies (16-32K) with more aggressive subword splitting.

## Modern Tokenization Strategies

Contemporary models employ hybrid approaches:

- **Prefix Tokenization**: Separate treatment for special tokens and prompt structure (important in instruction-tuned models)
- **Anchored Tokenization**: Reserving tokens for specific functions (reasoning tokens, retrieval tokens)
- **Vocabulary Expansion**: Dynamically adding domain-specific tokens during fine-tuning

## Practical Implications

When building NLP systems:

1. Match tokenizer to model: Use the exact tokenizer released with pretrained models
2. Monitor OOV rates: High OOV suggests [vocabulary](/tokenizer-design-eng.html) mismatch
3. Consider downstream task: Translation benefits from morphological awareness; classification may tolerate aggressive subword splitting
4. Evaluate on real data: Tokenization performance varies dramatically across domains
5. Handle special tokens: Proper handling of `[CLS]`, `[SEP]`, `[PAD]`, etc. is crucial

## Conclusion

Tokenization appears simple but shapes everything downstream. The shift from word-level to subword tokenization was crucial to modern NLP's success, enabling handling of morphological complexity, rare words, and multilingual contexts. Understanding tokenization strategies helps explain why certain models excel at specific tasks and informs decisions about model selection and architecture design for new applications.

