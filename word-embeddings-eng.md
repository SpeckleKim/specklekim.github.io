---
layout: default
title: Word Embeddings (Word2Vec, GloVe)
lang: en
---

# Word Embeddings: From One-Hot to Dense Representations

Word embeddings represent words as dense vectors in continuous space. They capture semantic and syntactic relationships between words and have become fundamental to modern NLP. The evolution from sparse one-hot representations to dense embeddings transformed our ability to model language.

## The One-Hot Encoding Problem

Before embeddings, the standard approach was one-hot encoding:

```
Vocabulary: ["cat", "dog", "mouse", "runs", "sleeps"]
cat:   [1, 0, 0, 0, 0]
dog:   [0, 1, 0, 0, 0]
mouse: [0, 0, 1, 0, 0]
```

**Limitations:**
- **Sparsity**: Only one non-zero element; computationally wasteful
- **High dimensionality**: Vocabulary size determines vector dimension
- **No semantic information**: Orthogonal vectors mean no similarity signal
- **Poor generalization**: Can't represent unknown words

The one-hot encoding treats every word as completely independent, with no way to capture that "cat" and "dog" are semantically related.

## The Distributional Hypothesis

The breakthrough insight: *words that appear in similar contexts have similar meanings*.

```
"The cat sat on the mat."
"The dog sat on the mat."
→ "cat" and "dog" should have similar representations
```

This hypothesis underpins all modern word embeddings. By analyzing large corpora, we can discover that words appearing in similar contexts (surrounding words) should occupy nearby regions of embedding space.

## Word2Vec: The Game Changer

Google's 2013 Word2Vec introduced efficient methods for learning dense embeddings from large unlabeled corpora. Two architectures:

### Skip-gram Model

Given a word, predict its surrounding context words:

```
Input: "sat"
Predict: {"on", "the", "cat", "mat"} (from a window of ±2)
```

The model learns word vectors that minimize prediction error. Words with similar contexts learn similar vectors.

**Training:**
1. Take word from corpus with context window (e.g., ±5 words)
2. Feed word vector through shallow neural network
3. Predict probability distribution over context words
4. Backprop to update word vectors
5. Repeat for billions of word-context pairs

**Advantages:**
- Works well with infrequent words (each occurrence is a training example)
- Semantically meaningful: "king - man + woman ≈ queen"
- Captures both semantic and syntactic relations

### Continuous Bag of Words (CBOW)

The inverse task: given context words, predict the target word:

```
Context: {"The", "cat", "on", "the"}
Predict: "sat"
```

CBOW averages context vectors and predicts the center word.

**Comparison:**
| Skip-gram | CBOW |
|---|---|
| Better for rare words | Faster, better for frequent words |
| Each context generates training sample | Multiple contexts merged into one |
| Captures syntactic relations well | Good semantic relations |

## GloVe: Global Vectors

Stanford's GloVe combines insights from matrix factorization and local context windows. It explicitly captures global co-occurrence statistics:

**Motivation:**
- Word2Vec uses only local context windows (miss global patterns)
- Matrix factorization captures global co-occurrences (but ignores context)
- GloVe merges both approaches

**Algorithm:**
1. Construct word co-occurrence matrix from corpus
2. Model predicts word-context co-occurrence probability
3. Weighted least-squares optimization learns vectors
4. Vectors capture both global patterns and local context

**Key insight:** Weight the loss by co-occurrence frequency, focusing model on meaningful word pairs rather than rare or very common combinations.

GloVe embeddings often demonstrate superior performance on analogy tasks and show particularly good semantic organization.

## FastText: Subword Embeddings

Facebook's FastText extends Word2Vec to subword level:

```
"running" = ["<run", "run", "unni", "nning", "nning>"]
```

Each word is represented as sum of its character n-gram vectors.

**Advantages:**
- Handles morphologically rich languages
- Better for rare and out-of-vocabulary words
- Captures character-level patterns ("running" and "runner" share morphological substrings)

**Disadvantage:**
- Larger models (must store subword embeddings)
- Slower inference

## Contextual Embeddings: ELMo

Early Language Model (ELMo) introduced context-dependent embeddings. Rather than one fixed vector per word:

```
"bank" in "river bank" → different vector than
"bank" in "savings bank"
```

ELMo uses bidirectional LSTM trained on language modeling to generate representations that depend on context.

**Architecture:**
- Train bidirectional LSTM language model (token → ±context → next token)
- For downstream tasks, concatenate all LSTM layers' hidden states
- Embeddings are weighted combinations of all layers

**Significance:**
- Context changes representation dynamically
- Different tasks may benefit from different layers
- Improved performance on nearly all downstream NLP tasks
- Introduced the era of pre-trained contextualized embeddings

## Embedding Visualization

Embeddings can be visualized using dimensionality reduction (t-SNE, UMAP):

```
Semantic clusters emerge naturally:
- Countries cluster together
- Male/female profession pairs align
- Related concepts group nearby
```

This visualization reveals that meaningful semantic structure emerges from simple co-occurrence statistics.

## Analogy Tasks

Classic evaluation: solve word analogies:

```
"man" is to "woman" as "king" is to ?
→ king - man + woman ≈ queen

"Paris" is to "France" as "Tokyo" is to ?
→ tokyo - paris + france ≈ japan
```

This works because embeddings capture relational structure:

```
king - man ≈ woman - queen
(captures the gender-relation vector)
```

Analogy tasks reveal that embeddings encode:
- Semantic relations (synonymy, antonymy, hypernymy)
- Syntactic relations (singular/plural, verb tense)
- Geographic relations (capital-country)
- Cultural relations (movie-actor)

## Why Embeddings Matter

1. **Information density**: Dense vectors carry information in every dimension
2. **Similarity**: Cosine distance between embeddings correlates with semantic similarity
3. **Compositionality**: Word vectors can be combined (king - man + woman ≈ queen)
4. **Transfer**: Embeddings learned on one task often benefit other tasks
5. **Efficiency**: Sparse operations on dense vectors often faster than dealing with original sparsity

## Limitations and Evolution

Traditional word embeddings have limitations:
- Single representation ignores polysemy (multiple meanings)
- Static vectors can't adapt to context
- Rare words get poor representations
- Don't capture pragmatic meaning

These limitations motivated contextual embeddings (ELMo, BERT), which learn task and context-specific representations.

## Modern Embedding Landscape

Contemporary NLP typically doesn't use Word2Vec/GloVe directly; instead:
- BERT generates contextualized embeddings
- Sentence-BERT (SBERT) computes semantically meaningful sentence vectors
- Specialized models for specific domains (clinical BERT, scientific BERT)
- Retrieval models use embedding space for semantic search

However, Word2Vec and GloVe remain valuable:
- Lightweight alternatives for resource-constrained settings
- Good understanding of embedding fundamentals
- Efficient for large-scale similarity search
- Interpretable and well-studied

## Conclusion

Word embeddings revolutionized NLP by showing that meaning emerges from distributional patterns in large text corpora. Word2Vec's efficiency and simplicity made embeddings practical at scale, while GloVe and FastText showed how to incorporate global statistics and morphological structure. The progression from one-hot to Word2Vec to contextual embeddings shows NLP's fundamental trajectory: increasingly sophisticated representations that capture richer linguistic information. Understanding embedding-based methods provides essential foundation for modern transformer-based models.

